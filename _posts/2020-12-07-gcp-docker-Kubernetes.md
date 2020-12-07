---
layout: post
title: gcp Cloud docker Kubernetes
date: 2020-12-07
tags: gcp Cloud docker Kubernetes Container
---

gcp Container clusters 紀錄一下 

大概就是

1. 開 Container 

2. deploy nginx

3. nginx 設成 NLB

4. replicas 成 3 台


首先 enabled:

Kubernetes Engine API
Container Registry API


在 Cloud Shell.
```
export MY_ZONE=us-central1-a
gcloud container clusters create webfrontend --zone $MY_ZONE --num-nodes 2
kubectl version
kubectl create deploy nginx --image=nginx:1.17.10
kubectl get pods
kubectl expose deployment nginx --port 80 --type LoadBalancer
kubectl get services
kubectl scale deployment nginx --replicas 3
kubectl get pods
kubectl get services
```


<img src="/images/posts/google-doc/21.png">
<img src="/images/posts/google-doc/22.png">
<img src="/images/posts/google-doc/23.png">
<img src="/images/posts/google-doc/24.png">
