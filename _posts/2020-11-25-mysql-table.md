---
layout: post
title: mysql table 備份清空
date: 2020-11-25
tags: mysql table
---

mysql table message_list 備份清空

```
CREATE TABLE message_list_20201125_t LIKE message_list;
RENAME TABLE message_list TO message_list_20201125;
RENAME TABLE message_list_20201125_t TO message_list;
```
