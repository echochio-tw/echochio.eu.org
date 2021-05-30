---
layout: post
title: python 雜記 骰子問題
date: 2021-05-26
tags: python random enumerate pandas numpy
---

#Python 骰子問題

看到不錯記錄起來

https://ithelp.ithome.com.tw/questions/10203556

加上了註解
```
import random
#宣告 亂數

r = [0 for _ in range(6)]
# r是六個 0 的 list  -> [0, 0, 0, 0, 0, 0]

for _ in range(100):
# 跑100 次
    r[random.randint(0, 5)] += 1
    # 每次一個亂數 0 到 5 (相對 1 到 6) , 
	# 每次一個亂數放到 list 內 相對位置 
	# 如 r[0] = r[0]+1, r[1] = r[1]+1 .....


for i, j in enumerate(r): 
# 0,r[0] 1,r[1] .. 的 loop 
    print(f"{i + 1}: 出現 {j} 次") 
    # format 輸出 (1, r[0]) , (2, r[1]) ....

print(f"{' '.join([str(i+1) for i, x in enumerate(r) if x == max(r)])} 出現最多次，是{max(r)}次")
#format 輸出x,及max(r), 
# 回圈內 x 為 list 中最大值(if 取得）ｉ是list 第幾個 , 輸出 str(1+1)
# join 處理多個 max 時顯示
print(f"{' '.join([a = a + 1 for i, x in enumerate(r) if x == 1])} 出現最少次，是{r}次")
# 同 max 改成 min
```

用 pandas
```
import pandas as pd
import random

def change(n):
	ex = [0] * 6
	ex[n] = 1
	return ex

data = [change(random.randint(0, 5)) for x in range(100)]
df = pd.DataFrame(data, columns=[x + 1 for x in range(6)])
print(df.sum())
```

我用 pandas 加上 numpy
```
import pandas, numpy, random
r = pandas.DataFrame(numpy.random.randint(1,7, size=(1,100))).values.tolist()[0]
for j in range(1,7):
    print(f"出現 {j} 的 {int((sum([x for x in r if x == j]))/j)} 次")

```
亂搞
```
import numpy, random
r = numpy.random.randint(1,7,100, numpy.int)
for j in range(1,7):
    print(f"出現 {j} 的 {len([x for x in r if x == j])} 次")
```

改成用 choice 比較好懂
```
import numpy as np
r = np.random.choice(np.arange(1, 7), 100)
for j in range(1,7):
    print(f"出現 {j} 的 {len([x for x in r if x == j])} 次")
```

用 collections的Counter

```
import random
from collections import Counter

numbers  = [random.randint(1,6) for i in range(100)]
counted = list(Counter(numbers).items())
counted_sort_by_key = sorted(counted, key=lambda x: x[0])
counted_sort_by_value = sorted(counted, key=lambda x: x[1])

[print("點數{0}出現{1}次".format(i[0], i[1])) for i in counted_sort_by_key]

print("點數{0}出現最少次，共{1}次".format(counted_sort_by_value[0][0], counted_sort_by_value[0][1]))
print("點數{0}出現最多次，共{1}次".format(counted_sort_by_value[-1][0], counted_sort_by_value[-1][1]))
```
