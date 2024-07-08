---
layout: post
title: kubernetes 安装 kubernetes-dashboard 7.x
date: 2024-07-08
tags: kubernetes-dashboard 
---

```
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/

# 默认参数安装
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kube-system

# 我的集群使用默认参数安装 kubernetes-dashboard-kong 出现异常 8444 端口占用
# 使用下面的命令进行安装，在安装时关闭kong.tls功能
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --namespace kube-system --set kong.admin.tls.enabled=false
```

確認都正常
```
kubectl get pod -A
kubectl get svc -A
```

GCP 可以直接指定外部IP
```
kubectl patch svc kubernetes-dashboard-kong-proxy -n kube-system -p '{"spec": {"type": "LoadBalancer"}}'
kubectl get svc kubernetes-dashboard -n kube-system
```

一般 kubernetes 可指定 IP 
```
kubectl patch svc kubernetes-dashboard -n kubernetes-dashboard -p '{"spec": {"type": "LoadBalancer", "externalIPs":["192.168.0.144"]}}'
```


等到顯示
```
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
kubernetes-dashboard   LoadBalancer   10.16.130.180   34.81.100.122   443:30706/TCP   5``
```

瀏覽 : https://34.81.100.122

創建臨時token
```
cat > dashboard-user.yaml << EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
EOF

kubectl  apply -f dashboard-user.yaml

# 创建token
kubectl -n kube-system create token admin-user
```

創建長期token
```
cat > dashboard-user-token.yaml << EOF
apiVersion: v1
kind: Secret
metadata:
  name: admin-user
  namespace: kube-system
  annotations:
    kubernetes.io/service-account.name: "admin-user"   
type: kubernetes.io/service-account-token  
EOF

kubectl  apply -f dashboard-user-token.yaml

# 查看密码
kubectl get secret admin-user -n kube-system -o jsonpath={".data.token"} | base64 -d
```

READ-ONLY user
```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: read-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: read-user
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["get", "watch", "list"]        
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  name: read-user
  kind: ClusterRole
subjects:
  - kind: ServiceAccount
    name: read-user
    namespace: kube-system
---
apiVersion: v1
kind: Secret
metadata:
  name: read-user
  annotations:
    kubernetes.io/service-account.name: "read-user"
type: kubernetes.io/service-account-token
EOF

# 创建token
kubectl -n kube-system create token read-user
```

READ-ONLY 創建長期token
```
cat > dashboard-user-token.yaml << EOF
apiVersion: v1
kind: Secret
metadata:
  name: admin-user
  namespace: kube-system
  annotations:
    kubernetes.io/service-account.name: "read-user"   
type: kubernetes.io/service-account-token  
EOF

kubectl  apply -f dashboard-user-token.yaml

# 查看密码
kubectl get secret read-user -n kube-system -o jsonpath={".data.token"} | base64 -d
```
