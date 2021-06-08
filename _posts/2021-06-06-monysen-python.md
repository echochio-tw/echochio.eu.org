---
layout: post
title: monysen python
date: 2021-06-06
tags: monysen python lambda
---

【尋找第6個默尼森數】利用python找出第n個默尼森數

半抄半寫 
```
import math
#質數
prime = lambda num: not [num%i for i in range(2,int(math.sqrt(num))+1) if num%i == 0]
p=2
i=1
while i<=9:
    if prime(p):
        m=2**p-1
        print((i-1),'monysen is',m)
        i+=1
    p+=1
```
