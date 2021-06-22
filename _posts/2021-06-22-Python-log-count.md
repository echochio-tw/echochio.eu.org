---
layout: post
title: Python log count by hour
date: 2021-06-22
tags: python Pandas
---

log count by hour

```
import pandas as pd
df = pd.read_csv('20210619.log', header=None)
for i in range (0,24):
    p = r'22/Jun/2021:'+str(i).zfill(2)+':.'
    print ('22/Jun/2021:'+str(i).zfill(2)+':00:00 - 22/Jun/2021:'+str(i).zfill(2)+':59:59 -->', df[0].str.contains(p).sum(),'條')
```
