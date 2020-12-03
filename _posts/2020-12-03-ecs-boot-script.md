---
layout: post
title: gcp ecs 開 root ssh
date: 2020-12-3
tags: gcp ecs
---

gcp 開 ecs

紀錄一下 shell script in create

<img src="/images/posts/google-doc/7.png">

<img src="/images/posts/google-doc/8.png">

<img src="/images/posts/google-doc/9.png">


放這個然後 root 密碼是 abcd1234
```
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
sed -i 's/PermitRootLogin no/PermitRootLogin yes/g' /etc/ssh/sshd_config
service sshd restart
echo -e "abcd1234\nabcd1234" | passwd root
```

建造下去 .....
