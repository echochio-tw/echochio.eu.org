---
layout: post
title: gcp App Engine
date: 2020-12-07
tags: gcp App Engine
---

gcp App Engine 紀錄一下 


在 Cloud Shell.
```
gcloud auth list
gcloud config list project
gcloud app create --project=$DEVSHELL_PROJECT_ID
sudo apt-get update
sudo apt-get install virtualenv -y
git clone https://github.com/GoogleCloudPlatform/python-docs-samples.git
cd python-docs-samples/appengine/standard_python3/django
virtualenv -p python3 venv
source venv/bin/activate
python3 main.py
gcloud app deploy
gcloud app browse
```

flask 有測出來 

Flask AppBuilder 沒搞定 ......
