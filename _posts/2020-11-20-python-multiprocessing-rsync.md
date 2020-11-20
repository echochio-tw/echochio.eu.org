---
layout: post
title: python multi processing rsync 多序多工
date: 2020-11-20
tags: python multiprocessing rsync
---

沒有仔細寫 ... 就寫出來交卷就這樣 ....
```
#!/usr/bin/env python3
from multiprocessing import Pool
import subprocess
import os
def run(task):
    subprocess.call(["rsync", "-arq", dsrc[task], dest[task]])
    print ("rsync", "-arq", dsrc[task], dest[task])
    print("Handling {}".format(task))
if __name__ == "__main__":
    for x in os.walk('data/prod/'):
        x[0].replace(" ", "\ ")
        d = x[0].replace('data/prod/','data/prod_backup/')
        if not os.path.exists(d):
             os.mkdir(d)
    dsrc=[]
    dest=[]
    for dirPath, dirNames, fileNames in os.walk("data/prod/"):
        for f in fileNames:
            f.replace(" ", "\ ")
            s = os.path.join(dirPath, f)
            dsrc.append(s)
            dest.append(s.replace("data/prod/", "data/prod_backup/"))
    tasks = [i for i in range(len(dsrc))]
    p = Pool(len(tasks))
    p.map(run, tasks)
```
