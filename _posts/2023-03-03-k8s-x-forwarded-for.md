---
layout: post
title: k8s nginx ingress x-forwarded-for
date: 2023-03-03
tags: k8s nginx
---
x-forwarded-for

nginx-ingress ConfigMap 的data: 加
```
  compute-full-forwarded-for: "true"
  use-forwarded-headers: "true"
```

我的機器 nginx-ingress ConfigMap

default_configmaps_nginx-ingress-ingress-nginx-controller.yaml

```
apiVersion: v1
data:
  allow-snippet-annotations: "true"
  compute-full-forwarded-for: "true"
  use-forwarded-headers: "true"
kind: ConfigMap
metadata:
  annotations:
    meta.helm.sh/release-name: nginx-ingress
    meta.helm.sh/release-namespace: default
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: nginx-ingress
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.5.1
    helm.sh/chart: ingress-nginx-4.4.2
  name: nginx-ingress-ingress-nginx-controller
  namespace: default
```

看 log 最後20(持續 tail -f )
```
kubectl logs -f --tail=20 $(kubectl get pods -A|grep ingress-nginx-controller|awk '{print $2}') -n $(kubectl get pods -A|grep ingress-nginx-controller|awk '{print $1}')
```

```
34.87.99.99 - - [03/Mar/2023:04:15:19 +0000] "GET /api/status HTTP/1.1" 200 246 "-" "-" 305 0.210 [-] [] 10.15.4.212:8907 246 0.210 200 a27b91ba763850d3f1d0a0b17b67729f
34.87.99.99 - - [03/Mar/2023:04:15:41 +0000] "GET /api/status HTTP/1.1" 200 246 "-" "Zabbix" 325 0.200 [-] [] 10.15.4.212:8907 246 0.200 200 3b2c30725a952c011ef1046706e7f3e4
34.87.99.99 - - [03/Mar/2023:04:16:42 +0000] "GET /api/status HTTP/1.1" 200 246 "-" "Zabbix" 325 0.201 [-] [] 10.15.4.212:8907 246 0.200 200 b032a97689eaa95e0e50a24b8d5192dd
34.87.99.99 - - [03/Mar/2023:04:16:42 +0000] "GET /api/status HTTP/1.1" 200 246 "-" "-" 305 0.220 [-] [] 10.15.4.212:8907 246 0.220 200 fe0dcab535e4e57f5307e9ed9edcc0e2
34.87.99.99 - - [03/Mar/2023:04:17:42 +0000] "GET /api/status HTTP/1.1" 200 246 "-" "Zabbix" 325 0.244 [-] [] 10.15.4.212:8907 246 0.243 200 b30c3257b659f0cd0a8933e3308a34fa
```

可建立一個 nginx pod 去 curl 內部 pod 測試
```
kubectl run nginx --image=nginx --restart=Never
kubectl exec --stdin --tty nginx -- /bin/bash
kubectl delete pod nginx
```
