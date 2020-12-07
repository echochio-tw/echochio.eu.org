---
layout: post
title: gcp Deployment Manager and Cloud Monitoring
date: 2020-12-07
tags: gcp Deployment Manager and Cloud Monitoring
---

Deployment Manager 在 Cloud Shell 建立 ECS
```
export MY_ZONE=us-central1-a
gsutil cp gs://cloud-training/gcpfcoreinfra/mydeploy.yaml mydeploy.yaml
sed -i -e "s/PROJECT_ID/$DEVSHELL_PROJECT_ID/" mydeploy.yaml
sed -i -e "s/ZONE/$MY_ZONE/" mydeploy.yaml
gcloud deployment-manager deployments create my-first-depl --config mydeploy.yaml
```

reconfig ECS
```
sed -i 's/apt-get update/apt-get update; apt-get install nginx-light -y/g'  mydeploy.yaml
gcloud deployment-manager deployments update my-first-depl --config mydeploy.yaml

```

Cloud Monitoring


<img src="/images/posts/google-doc/25.png">
<img src="/images/posts/google-doc/26.png">
<img src="/images/posts/google-doc/27.png">
<img src="/images/posts/google-doc/28.png">
<img src="/images/posts/google-doc/29.png">
<img src="/images/posts/google-doc/30.png">

```
curl -sSO https://dl.google.com/cloudagents/install-monitoring-agent.sh
sudo bash install-monitoring-agent.sh

curl -sSO https://dl.google.com/cloudagents/install-logging-agent.sh
sudo bash install-logging-agent.sh
```
<img src="/images/posts/google-doc/31.png">
