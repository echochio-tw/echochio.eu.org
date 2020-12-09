---
layout: post
title: GCP 用 App Engine 安装 flask-appbuilder
date: 2020-12-09
tags: App Engine flask-appbuilder
---
GCP 用 App Engine 安装 flask-appbuilder 試了好久搞通了

原來沒加 uwsgi 進去 python 與 app.yaml 設定用 uwsgi 開


開 gconsole 
```
gcloud auth list
gcloud config list project
gcloud app create --project=$DEVSHELL_PROJECT_ID
mkdir flask-appbuilder
cd flask-appbuilder
sudo apt-get update
sudo apt-get install virtualenv
virtualenv -p python3 venv
source venv/bin/activate
pip install flask-appbuilder
flask fab create-app
(输入 app)
cd app
flask fab create-admin (測試用 sqllite 要改外部 sql 也行改完config.py 再建 admin)
cd ..
mv app/config.py . (如外部資料庫去改一下, 這要做一下測試不知連線IP 是啥, sql 要鎖來源IP)
mv app/app.db . (先用 sqllite 吧 ...)
pip install uwsgi==2.0.19.1 (參考 google 範例加 uwsgi 才OK的)
pip freeze > requirements.txt
```

app.yaml: (參考 google 範例加 uwsgi 才OK的)
```
# [START gae_python3_custom_runtime]
runtime: python38
entrypoint: uwsgi --http-socket :8080 --wsgi-file main.py --callable app --master --processes 1 --threads 4
# [END gae_python3_custom_runtime]
```


main.py (這個當然抄寫 google 範例去改的):
```
from flask import Flask
from app.app import app as app
#app = Flask(__name__)

@app.route('/')
def main():
    return 'Hello, World!'

if __name__ == '__main__':
    app.run(debug=True)
```

測試的 run.py (這只是測試, 當開發用的, 正式是用main.py)
```
rom app.app import app as application

if __name__ == "__main__":
    application.run(host="127.0.0.1", port=8080, debug=True)
```

測試 (測試開發用)

```
python run.py
```

push 上去
```
gcloud app deploy
gcloud app browse
```

<img src="/images/posts/google-doc/40.png">
<img src="/images/posts/google-doc/41.png">
