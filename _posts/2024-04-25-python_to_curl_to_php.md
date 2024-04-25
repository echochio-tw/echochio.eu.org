---
layout: post
title: python to curl to php .......
date: 2024-04-25
tags: python curl
---
python 轉 curl 然後再轉至其他語言

curl to php .......

https://curlconverter.com/

這邊來 python to curl (當然要先安裝 requests_to_curl)
```
import requests_to_curl
import requests
to = 'test@gmail.com'
body = 'juest test'
subject = 'test_mail'
headers = { 'Authorization': 'Authorization_passwd' }
payload = {'from': 'testf@gmail.com'+' <'+'nickname'+'>',
'to': to,
'subject': subject,
'text': body}
requests_to_curl.parse(requests.post('https://api.mailgun.net/v3/'+'name'+'/messages', headers=headers, data=payload))
```
顯示
```
curl -X POST -H 'Accept: */*' -H 'Accept-Encoding: gzip, deflate' -H 'Authorization: Authorization_passwd' -H 'Connection: keep-alive' -H 'Content-Type: application/x-www-form-urlencoded' -H 'User-Agent: python-requests/2.23.0' -d 'from=testf%40gmail.com+%3Cnickname%3E&to=test%40gmail.com&subject=test_mail&text=juest+test' https://api.mailgun.net:443/v3/name/messages
```

把這個去 https://curlconverter.com/ 轉php(這邊用GuzzleHttp)
```
<?php
require 'vendor/autoload.php';

use GuzzleHttp\Client;

$client = new Client();

$response = $client->post('https://api.mailgun.net:443/v3/name/messages', [
    'headers' => [
        'Accept'          => '*/*',
        'Accept-Encoding' => 'gzip, deflate',
        'Authorization'   => 'Authorization_passwd',
        'Connection'      => 'keep-alive',
        'Content-Type'    => 'application/x-www-form-urlencoded',
        'User-Agent'      => 'python-requests/2.23.0'
    ],
    'form_params' => [
        'from' => 'testf@gmail.com <nickname>',
        'to' => 'test@gmail.com',
        'subject' => 'test_mail',
        'text' => 'juest test'
    ]
]);
```
用 cURL for php
```
<?php
$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, 'https://api.mailgun.net:443/v3/name/messages');
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'POST');
curl_setopt($ch, CURLOPT_HTTPHEADER, [
    'Accept: */*',
    'Accept-Encoding: gzip, deflate',
    'Authorization: Authorization_passwd',
    'Connection: keep-alive',
    'Content-Type: application/x-www-form-urlencoded',
    'User-Agent: python-requests/2.23.0',
]);
curl_setopt($ch, CURLOPT_POSTFIELDS, 'from=testf%40gmail.com+%3Cnickname%3E&to=test%40gmail.com&subject=test_mail&text=juest+test');

$response = curl_exec($ch);

curl_close($ch);
```







```
