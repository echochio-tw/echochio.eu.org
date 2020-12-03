---
layout: post
title: gcp Cloud Storage 開外部可下載
date: 2020-12-3
tags: gcp Cloud Storage
---

gcp 開 Cloud Storage 開外部可下載

紀錄一下 Cloud Shell. (其中 acl 開全限讓外部可下載)
```
gcloud config set project qwiklabs-gcp-00-7717f40b97e5
export LOCATION=ASIA
gsutil mb -l $LOCATION gs://$DEVSHELL_PROJECT_ID
gsutil cp gs://cloud-training/gcpfci/my-excellent-blog.png my-excellent-blog.png
gsutil cp my-excellent-blog.png gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png
gsutil acl ch -u allUsers:R gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png
```

<img src="/images/posts/google-doc/13.png">

可瀏覽了
```
https://storage.googleapis.com/qwiklabs-gcp-00-7717f40b97e5/my-excellent-blog.png
```

<img src="/images/posts/google-doc/14.png">
<img src="/images/posts/google-doc/15.png">
<img src="/images/posts/google-doc/16.png">
<img src="/images/posts/google-doc/17.png">

不用 Cloud Shell
這邊設定Bucket 全開放 read

<img src="/images/posts/google-doc/10.png">
<img src="/images/posts/google-doc/11.png">
<img src="/images/posts/google-doc/12.png">
