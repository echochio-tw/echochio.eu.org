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


<img src="/images/posts/google-doc/21.png">
<img src="/images/posts/google-doc/22.png">
<img src="/images/posts/google-doc/23.png">
<img src="/images/posts/google-doc/24.png">
