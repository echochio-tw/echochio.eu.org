---
layout: post
title: Line bot 用 google apps script
date: 2021-01-07
tags: line bot gas google apps script
---

進到 LINE Developers 的頁面：https://developers.line.biz/zh-hant/

登入 LINE 帳號後，進到後台，選擇要建立的Providers

進到 Provider 後，點 Create a new channel，類型的部份選擇「Messaging API」：

Webhook URL (這個後來會用到)

在 LINE 的後台，機器人的頁面中，頁籤上選「Messaging API」：

```
LINE Official Account features -> Allow bot to join group chats -> Enabled
Greeting messages -> Disabled
```

管理頁面中有一項是「聊天」，把「加入群組或多人聊天室」的選項改為接受

加好友用 QR code 加 比較簡單(Bot basic ID)

程式API用 Channel access token

Google Apps Script (取得ID)
```
var token = '??????????????????????????????????????0DW78tL/bQqjZxgxH3Mvwp/H8Xh8i7TnKCCwdB04t89/1O/w1cDnyilFU=';

function doPost(e) {
  var message = JSON.parse(e.postData.contents);
  var targetID = "";
  var replayToken = message.events[0].replyToken;
  if ( message.events[0].source.type == "user" )
  { targetID = "使用者ID:" + message.events[0].source.userId; 
  }
  else if ( message.events[0].source.type == "room" )
  { targetID = "房间ID:" + message.events[0].source.roomId;
    targetID = targetID + " 使用者ID:" + message.events[0].source.userId; 
  }
  else
  { targetID = "群組ID:" + message.events[0].source.groupId;
    targetID = targetID + " 使用者ID:" + message.events[0].source.userId;
  }
  var data = {
    replyToken: replayToken,
    messages: [
      {
        "type": "text",
        "text": targetID
      },
    ]
  };
  var option = {
    method: 'post',
    headers: { Authorization: 'Bearer ' + token },
    contentType: 'application/json',
    payload: JSON.stringify(data)
  };
  
  UrlFetchApp.fetch('https://api.line.me/v2/bot/message/reply', option);
}
```

寫好取得網址

放到 Webhook URL 然后 Use webhook 打開


特定 user 或 group 或 room
```
replyToken: replayToken,
改成
'to':  '你的 user ID',
```

usrl 改成 push 
```
https://api.line.me/v2/bot/message/push
```
