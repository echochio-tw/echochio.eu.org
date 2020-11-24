---
layout: post
title: python string multi replace
date: 2020-11-24
tags: python replace
---

python string multi replace

小程式記錄一下

```
s = "The quick brown fox jumps over the lazy dog"
for r in (("brown", "red"), ("lazy", "quick")):
    s = s.replace(*r)

#output will be:  The quick red fox jumps over the quick dog
```
