---
layout: post
title: python memcache 記憶體快取
date: 2020-11-20
tags: python memcache
---

之前都用檔案快取現在改由 記憶體快取 或 SQL

memcached 與 php 用法一樣, 記憶體與檔案比用起來快多了 (其實沒差啦)

安裝 memcached
```
yum install memcached
```

/etc/sysconfig/memcached
```
PORT="11211"
USER="memcached"
MAXCONN="1024"
CACHESIZE="128MB" # 看需求 ....
OPTIONS=""
```
```
systemctl enable memcached
systemctl restart memcached
```

python 改由 memcache
```
#!/usr/bin/env python3
import pyodbc
import requests
import memcache # import memcache 進來

def telegram_send(bot_message):
    url = 'https://api.telegram.org/bot61xxxxxxx:xxxxxxxxxxxxxxxxxxxxxxxxxxxxx/sendMessage'
    data = {'chat_id': '-45xxxxx',  'text': bot_message}
    requests.post(url, data).json()
    return

def write_info(infoout):
    conn.delete('gameinfo') #清除
    conn.set('gameinfo',infoout) #重設
    telegram_send(infoout)
    return

server = '192.168.0.99,63600'
database = 'master'
username = 'sa'
password = 'aaaaaaaaa'
driver= '{ODBC Driver 13 for SQL Server}'
conn = pyodbc.connect('DRIVER='+driver+';SERVER='+server+';DATABASE='+database+';UID='+username+';PWD='+ password)
cursor = conn.cursor()
cursor.execute("""
SELECT b.ID,b.Name FROM [TDB].[dbo].[GDB] as a
left join [QDB].[dbo].[AInfo] as b on a.UID = b.UID
 where a.[EMachine] != ''
""")
rows = cursor.fetchall()
tout = ''
pout = 0

conn = memcache.Client(['127.0.0.1:11211']) #連 memcache
tinfo = conn.get('gameinfo') # 取值 .... 取不到不會 error 只會是空值
if rows != []:
    for row in rows:
        pout = pout + 1
        out = 'ID:{} Name:{}\n'.format(row[0],row[1])
        tout = tout + out
    if tinfo != tout:
        write_info(tout)
else:
    if tinfo != '人數：0':
        write_info('人數：0')
print (pout)

```

memcache的一些限制：
```
the maxiumum key length? (250 bytes) (一百多個中文字)
the limits on setting expire time 30 day (30天)
maximum data size you can store 1 megabyte (1MB)
```
