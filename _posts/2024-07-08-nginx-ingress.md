---
layout: post
title: nginx-ingress 快速安裝 與測試
date: 2024-07-08
tags: nginx-ingress
---

安裝 nginx-ingress (這邊安裝到 major 的 namespace)
```
helm repo add nginx-stable https://helm.nginx.com/stable
helm repo update
helm install nginx-ingress nginx-stable/nginx-ingress -n major --set rbac.create=true
```

whoami 的 deploy 與 svc
```
kubectl create deployment -n major whoami --image=nginxdemos/hello
kubectl expose deployment whoami --port=80 --target-port=80 -n major
```

whoami 的 Ingress
```
kubectl create ingress hello -n major --class=nginx  --rule="hello.luck.com/=whoami:80"
```

kubectl get ingress -A
```
NAMESPACE   NAME    CLASS   HOSTS            ADDRESS         PORTS   AGE
major       hello   nginx   hello.luck.com   35.18.26.186    80      14m
``` 
