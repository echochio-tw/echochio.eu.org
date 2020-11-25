---
layout: post
title: mysql table 備份清空
date: 2020-11-25
tags: mysql table
---

mysql table message_list 備份清空

```
RENAME TABLE message_list TO message_list_20201125;
CREATE TABLE message_list LIKE message_list_20201125;
```
