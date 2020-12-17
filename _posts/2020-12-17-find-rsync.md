---
layout: post
title: rsync one file type
date: 2020-12-17
tags: find rsync
---

rsync one file type

```
rsync -avz --remove-sent-files --files-from=<(find . -type f -name '*.ipa') source dest
```
