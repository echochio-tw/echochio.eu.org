---
layout: post
title: 免費的web , smtp , pop3 多網域認證 key 申請 , 及自動換KEY
date: 2016-11-14
tags: letsencrypt ssl
---

我家的 mail 是 Centos 7 + iredmail + 16 個網域

就簡單寫一下 :

```
1.
   安裝  git bc wget 套件
2. 
   git clone https://github.com/letsencrypt/letsencrypt /opt/letsencrypt (安裝程式到 /opt/letsencrypt)
3. 
   systemctl stop nginx (程式會模擬 web 給外面 ssl發行的網站確認用,所以要停 web) ....就是停掉 web 服務
4. 
   cd /opt/letsencrypt (切換目錄)
5. 
   執行
./letsencrypt-auto certonly --standalone --email chio@test1.com \
--agree-tos \
-d ms1.test1.com \
-d ms1. ms1.test1.com \
-d ms1. ms1.test2.com  \
-d ms1. ms1.test3.com \
-d mail. test1.com \
-d ms1. ms1.test4.com \
-d ms1. ms1.test5.com \
-d ms1. ms1.test6.com \
-d ms1. ms1.test7.com \
-d test.com \
-d ms1. test8.com \


(取認證 , 我家 mail 上有 16 個網域耶 ....哈)

6. 
   ls /etc/letsencrypt/live/ms1.test1.com/ (確認取到 4 個檔案...看日期及時間)

7.  
   cat /etc/nginx/conf.d/default.conf |grep pem (nginx 依設定放入正確位置)
   ssl_certificate /etc/letsencrypt/live/ms1.test1.com/fullchain.pem;
   ssl_certificate_key /etc/letsencrypt/live/ms1.test1.com/privkey.pem;
   ssl_dhparam /etc/pki/tls/dhparams.pem;
   
8.  
   cat /etc/postfix/main.cf |grep pem (postfix 依設定放入正確位置)
   smtpd_tls_dh1024_param_file = /etc/pki/tls/dhparams.pem
   smtpd_tls_key_file = /etc/letsencrypt/live/ms1.test1.com/privkey.pem
   smtpd_tls_cert_file = /etc/letsencrypt/live/ms1.test1.com/fullchain.pem
   smtpd_tls_CAfile = /etc/letsencrypt/live/ms1.test1.com/fullchain.pem

9. 
   cat /etc/dovecot/dovecot.conf |grep pem
   ssl_cert = </etc/letsencrypt/live/ms1.test1.com/fullchain.pem
   ssl_key = </etc/letsencrypt/live/ms1.test1.com/privkey.pem

10. 
   nginx -t  (檢查 nginx 語法 )
11. 
   postfix check ; postconf (檢查 postfix 語法 )
12. 
   systemctl restart nginx ; systemctl restart postfix ;systemctl restart dovecot (重啟)

```


自動更新方式

```
1. 
   NGINX config檔中加 『/.well-known』設定 
   location ~ /.well-known {
        allow all;
    }

2. 
   nginx -t (檢查 nginx 語法 )
3. 
   systemctl restart nginx (重啟)
4. 
   cd /opt/letsencrypt ;  cp ./certbot/examples/cli.ini /usr/local/etc/le-renew-webroot.ini
5. 
   vi /usr/local/etc/le-renew-webroot.ini

   email = chio@test.com
   domains = ms1.test1.com, ms1.test2.com, ms1.test3.com .............
   webroot-path = /var/www/html
  

6. 
   手動取憑證看看
   ./letsencrypt-auto certonly -a webroot --agree-tos --renew-by-default --config /usr/local/etc/le-renew-webroot.ini

7. 
   curl -L -o /usr/local/sbin/le-renew-webroot              https://gist.githubusercontent.com/thisismitch/e1b603165523df66d5cc/raw/fbffbf358e96110d5566f13677d9bd5f4f65794c/le-renew-webroot (抓程式)
8. 
   chmod +x /usr/local/sbin/le-renew-webroot  (改可執行)
9. 
   /usr/local/sbin/le-renew-webroot (手動取憑證看看)
10. 
   加入 cron
   0 3   2 /usr/local/sbin/le-renew-webroot >> /var/log/le-renewal.log;systemctl restart nginx;systemctl restart postfix;systemctl restart dovecot 2>&1 (每周二晚上三點執行)
```

後來故障了 ....XD 
   發現加 log 通知 比較好 .... 寫個 python .. 放入 cron
```
#! /usr/bin/python
with open('/var/log/le-renewal.log', 'r') as myfile:
        data=myfile.read().replace('\n', '<br>')
import smtplib
from email.mime.text import MIMEText
from email.header import Header
smtpserver='192.168.0.100'
user='echohio'
password='echochio123'
sender='chio@echochio-tw.github.io'
receiver='chio@echochio-tw.github.io'
subject='Python email le-renewal.log'
msg=MIMEText(data,'html','utf-8')
msg['Subject']=Header(subject,'utf-8')
smtp=smtplib.SMTP()
smtp.connect(smtpserver)
smtp.login(user, password)
smtp.sendmail(sender, receiver, msg.as_string())
smtp.quit()
```

 CentOS nginx 還有其他方法  
```
yum install epel-release
yum install python-certbot-nginx
certbot --nginx -d ms1.test.com
```
/lib/systemd/system/certbot-renew.service
```
ExecStart 后面 
--renew-hook "nginx -s reload"
```
  
