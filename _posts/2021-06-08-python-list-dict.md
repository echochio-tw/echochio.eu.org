---
layout: post
title: python list to dict
date: 2021-06-08
tags: python list dict
---

計算水果賣幾個...

我的解法
```
fruits = ['apple', 'banana', 'cherry','banana', 'peach', 'pear','peach', 'cherry' ]
d = {item:fruits.count(item) for item in fruits}
```

範例解法
```
fruits = ['apple', 'banana', 'cherry','banana', 'peach', 'pear','peach', 'cherry' ]
d = {}
fruits_set = set(fruits)
for item in fruits_set:
    d[item] = 0
    for i in range(len(fruits)):
        if item == fruits[i]:
            d[item] += 1
```		

第二解法
```
fruits = ['apple', 'banana', 'cherry','banana', 'peach', 'pear','peach', 'cherry' ]
d = {}
for item in fruits:
    d[item] = d.get(item, 0) + 1
```
