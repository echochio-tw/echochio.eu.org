---
layout: post
title: GKE 到 ELK APM 測試
date: 2024-07-09
tags: ELK
---

```
cat > elk_flask.py << EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pythonapp
  labels:
    app: python
spec:
  replicas: 1
  selector:
    matchLabels:
      app: python
  template:
    metadata:
      labels:
        app: python
      annotations:
        dapr.io/enabled: "true"
        dapr.io/app-id: "pythonapp"
        dapr.io/enable-api-logging: "true"
    spec:
      containers:
      - name: python
        image: ghcr.io/dapr/samples/hello-k8s-python:latest
EOF
kubectl apply -f python.yaml -n major
```

用dashboard 到 pod shell 內
```
apk add curl
python -m pip install elastic-apm
python -m pip install Flask --root-user-action=ignore
python -m pip install elastic-apm --root-user-action=ignore
python -m pip install blinker --root-user-action=ignore

cat > elk_flask.py << EOF
from flask import Flask
from elasticapm.contrib.flask import ElasticAPM
app=Flask(__name__)
app.config['ELASTIC_APM']={'SERVICE_NAME': 'flask','SECRET_TOKEN': '','SERVER_URL': 'http://10.28.0.8:8200','ENABLED': True,'ENVIRONMENT': 'production','LOG_LEVEL': "debug"}
apm=ElasticAPM(app)
@app.route('/')
def hello():
        return 'hello world!'
if __name__ == "__main__":
    app.run(host='0.0.0.0')
EOF

python elk_flask.py &

curl http://127.0.0.1:5000
```

看看 ELK 是否有 timeout 連不到，連不到去處理 GCP 防火牆。
