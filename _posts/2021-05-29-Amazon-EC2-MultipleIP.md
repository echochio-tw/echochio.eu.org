---
layout: post
title: Amazon EC2 MultipleIP
date: 2021-05-27
tags: AWS Amazon EC2 Multiple IP
---

最近常遇到紀錄一下

AWS EC2綁多網卡

```
先搞定單網卡可上網再處裡第二張網卡
```

```
內網加第二張網卡

1.點右上方：Create Network Interface
2.Description:
3.Subnet: (選VPC的名字)
4.Private IP (自選)
5.Security Groups(VPC default, 或.....)
6.create
7.Attach(附加到Instance)
8.in-use(啟用)
```

第二張網卡分配IP公網IP

```
1.Elastic IPs 点进去
2.Allocate New Address (選VPC)
3.Associate Address  (把IP给instance 選私有IP)
```
