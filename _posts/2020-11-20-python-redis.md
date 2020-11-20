---
layout: post
title: python redis 記憶體快取
date: 2020-11-20
tags: python redis
---

之前都用memcache 現在改由 redis記憶體快取 或 SQL

redis 與 php 用法一樣

安裝 redis
```
yum install http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
yum --enablerepo=remi install redis
systemctl enable redis
systemctl start redis
redis-cli info server
```

python 改由 redis
```
#!/usr/bin/env python3
import pyodbc
import requests
import redis   # 导入redis 模块

def telegram_send(bot_message):
    url = 'https://api.telegram.org/bot61xxxxxxx:xxxxxxxxxxxxxxxxxxxxxxxxxxxxx/sendMessage'
    data = {'chat_id': '-45xxxxx',  'text': bot_message}
    requests.post(url, data).json()
    return

def write_info(infoout):
    r.set('gameinfo',infoout) # 设定新值
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

r = redis.Redis(host='localhost', port=6379, decode_responses=True)   #連 redis , 加 decode_responses 这个抓回来的不必再 decode
tinfo = r.get('gameinfo') # 取值 .... 取不到不會 error 只會是空值
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

