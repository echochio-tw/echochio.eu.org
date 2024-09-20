---
layout: post
title: MYSQL cloud sql with compute engine slave
date: 2022-12-26
tags: GCP
---
GCP Cloud SQL MYSQL 5.7.40 

Ubuntu 18.04 slave (GCP compute engine)

下載 https://dev.mysql.com/downloads/mysql/5.7.html
```
wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-server_5.7.40-1ubuntu18.04_amd64.deb-bundle.tar
```

安裝時出現 

```
couldn't be accessed by user '_apt'. - pkgAcquire::Run (13: Permission denied)
```
請 apt update 再裝一次

```
tar xvf mysql-server_5.7.40-1ubuntu18.04_amd64.deb-bundle.tar
apt-get install ./libmysql*
apt-get install libmecab2
apt-get install ./mysql-community-client_5.7.40-1ubuntu18.04_amd64.deb 
apt-get install ./mysql-client_5.7.40-1ubuntu18.04_amd64.deb 
apt-get install ./mysql-community-server_5.7.40-1ubuntu18.04_amd64.deb
apt-get install ./mysql-server_5.7.40-1ubuntu18.04_amd64.deb 
```

啟動
```
systemctl start mysql.service
systemctl enable mysql.service
```

MYSQL root 密碼是 test1222

cloud SQL root 密碼是 12212bhdsf

cloud SQL rep 密碼是 密碼是 9999

cloud SQL 內網IP 10.3.0.3

才能只行下面 sell (不同自行修改 用 sed 換 10.3.0.3, test1222, 12212bhdsf, 9999)

```
#!/bin/bash
USER="root"
PASSWORD="12212bhdsf"
PASSWORD2="test1222"

cat <<EOF > /etc/mysql/mysql.conf.d/mysqld.cnf
[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
log-error       = /var/log/mysql/error.log
# By default we only accept connections from localhost
bind-address    = 127.0.0.1
server-id=2
gtid_mode=ON
enforce_gtid_consistency=ON
log_slave_updates=ON
replicate-ignore-db=mysql
binlog-format=ROW
log_bin=mysql-bin
expire_logs_days=1
read_only=ON
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
EOF

systemctl stop mysql
systemctl start mysql

mysql -uroot -p$PASSWORD2 -e "show databases" | grep -v Database | grep -v mysql| grep -v performance_schema |grep -v information_schema | rep -v sys | gawk '{print "drop database `" $1 "`;select sleep(0.1);"}' | mysql -uroot -p$PASSWORD2

# 下面兩行是刪除 rep user 與建立 rep user 如有了請自行註解或 自行改 rep 密碼
mysql -u $USER -p$PASSWORD -h 10.3.0.3 -e "DROP USER 'rep'@'%';"
mysql -u $USER -p$PASSWORD -h 10.3.0.3 -e "CREATE USER 'rep'@'%' IDENTIFIED BY '9999';GRANT REPLICATION SLAVE ON *.* TO 'rep'@'%';"

databases=`mysql -u $USER -p$PASSWORD -h 10.3.0.3 -e "SHOW DATABASES;" | tr -d "| " | grep -v Database`
mysql -u $USER -p$PASSWORD -h 10.3.0.3 -e "FLUSH TABLES WITH READ LOCK;show master status;"
for db in $databases; do
    if [[ "$db" != "information_schema" ]] && [[ "$db" != "performance_schema" ]] && [[ "$db" != "sys" ]] && [[ "$db" != "mysql" ]] && [[ "$db" != _* ]] ; then
        mysqldump -u $USER --set-gtid-purged=off -p$PASSWORD -h 10.3.0.3 --databases $db | mysql -u $USER -p$PASSWORD2
    fi
done
lock=`mysql -u $USER -p$PASSWORD -h 10.3.0.3 -e "show master status;" | grep -v File |awk '{print $(NF)'}`
# 下面這行是請自行改 rep 密碼
mysql -u $USER -p$PASSWORD2 -e "CHANGE MASTER TO MASTER_HOST='10.3.0.3', MASTER_USER='rep',MASTER_PASSWORD='9999', MASTER_AUTO_POSITION=1;"
mysql -u $USER -p$PASSWORD2 -e "stop slave;"
mysql -u $USER -p$PASSWORD2 -e "reset master;"
mysql -u $USER -p$PASSWORD2 -e "set global gtid_purged='$lock';"
mysql -u $USER -p$PASSWORD2 -e "start slave;"
mysql -u $USER -p$PASSWORD -h 10.3.0.3 -e "UNLOCK TABLES;"
mysql -u $USER -p$PASSWORD2 -e "show slave status\G"

```

發生同步失敗察看是timezone 問題
```
mysql> select * from performance_schema.replication_applier_status_by_worker\G;
*************************** 1. row ***************************
                                           CHANNEL_NAME:
                                              WORKER_ID: 1
                                              THREAD_ID: NULL
                                          SERVICE_STATE: OFF
                                      LAST_ERROR_NUMBER: 1298
                                     LAST_ERROR_MESSAGE: Worker 1 failed executing transaction 'c697f0d8-569d-11ee-bbbb-42010a030002:2406437' at source log mysql-bin.000358, end_log_pos 1957492; Error 'Unknown or incorrect time zone: 'Asia/Taipei'' on query. Default database: 'activity_system'. Query: 'BEGIN'
                                   LAST_ERROR_TIMESTAMP: 2024-09-20 03:54:42.618283
                               LAST_APPLIED_TRANSACTION:
     LAST_APPLIED_TRANSACTION_ORIGINAL_COMMIT_TIMESTAMP: 0000-00-00 00:00:00.000000
    LAST_APPLIED_TRANSACTION_IMMEDIATE_COMMIT_TIMESTAMP: 0000-00-00 00:00:00.000000
         LAST_APPLIED_TRANSACTION_START_APPLY_TIMESTAMP: 0000-00-00 00:00:00.000000
           LAST_APPLIED_TRANSACTION_END_APPLY_TIMESTAMP: 0000-00-00 00:00:00.000000
                                   APPLYING_TRANSACTION: c697f0d8-569d-11ee-bbbb-42010a030002:2406437
         APPLYING_TRANSACTION_ORIGINAL_COMMIT_TIMESTAMP: 2024-09-19 16:00:34.307776
        APPLYING_TRANSACTION_IMMEDIATE_COMMIT_TIMESTAMP: 2024-09-19 16:00:34.307776
             APPLYING_TRANSACTION_START_APPLY_TIMESTAMP: 2024-09-20 03:54:42.615626
                 LAST_APPLIED_TRANSACTION_RETRIES_COUNT: 0
   LAST_APPLIED_TRANSACTION_LAST_TRANSIENT_ERROR_NUMBER: 0
  LAST_APPLIED_TRANSACTION_LAST_TRANSIENT_ERROR_MESSAGE:
LAST_APPLIED_TRANSACTION_LAST_TRANSIENT_ERROR_TIMESTAMP: 0000-00-00 00:00:00.000000
```

把timezone 匯入
```
＃　mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -uroot mysql -p
```

設定全區 time_zone
```
mysql> SET GLOBAL time_zone = 'Asia/Taipei';
```
