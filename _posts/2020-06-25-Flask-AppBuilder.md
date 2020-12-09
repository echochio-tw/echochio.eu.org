---
layout: post
title: Flask AppBuilder
date: 2020-06-25
tags: Python Flask
---

Flask-AppBuilder .. MVC ... 還不錯以後來用

與安裝 Flask 一樣參考

https://echochio-tw.github.io/2019/09/flask+uwsgi/

在 source env/bin/activate 後安裝相關套件
```
python -m pip install flask-appbuilder
```
或是 ...  有 requirements.txt ...

run.py 我改成這樣看一下是不是與 wsgi.py 很像

```
from app import app

app.run(host="0.0.0.0")

```
wsgi.py 這樣設會直接帶起flask-AppBuilder沒問題

```
from app import app as application

if __name__ == "__main__":
    application.run()
```

這邊就不紀錄 flask + uwsgi 的方式了

紀錄一下 root 測試環境

建立工作目錄
```
flask fab create-app
```

改資料庫為 mysql 編輯 config.py 

```
SQLALCHEMY_DATABASE_URI = 'mysql+pymysql://root:password@localhost/message'
```
其中 
```
帳號 : root
密碼 : password
主機 : locqlhost
資料庫 : message
```

建立 admin 帳密
```
flask fab create-admin
```

執行
```
python3 run.py
```

這就不截圖了

改名 ... config.py 的 
```
APP_NAME = "短信APP"
```
修改首页 /app/index.py
```
from flask_appbuilder import IndexView
class MyIndexView(IndexView):
    index_template = 'new_index.html'
```

templates中new_index.html 改所想要的首页

修改app/__iniy__.py 在AppBuilder初始化的时候，指定indexview的值
```
from app.index import MyIndexView
appbuilder = AppBuilder(app, db.session, indexview=MyIndexView)
```

加 Helloword ... app/views.py 不寫了 ... google 都有

好了這邊紀錄一下 API 部分  app/views.py (這個與 flask 相近只比他寫在一個 class 就可)

要 import 那些就沒詳查了 ....
```
from flask import render_template
from flask_appbuilder.models.sqla.interface import SQLAInterface
from flask_appbuilder import ModelView, ModelRestApi
from . import appbuilder, db

from flask_appbuilder import ModelView,AppBuilder,expose,BaseView,has_access
from .models import Member,Message_list

from flask_appbuilder.api import BaseApi, expose
from flask import jsonify
from flask import request

def telegram_send(bot_message,bot_chatID = '-341334440',bot_token = '917680011:AAFpNGEoiebe1yXbkHDVbS11uKA12DSJLjs'):
    send_text = 'https://api.telegram.org/bot' + bot_token + '/sendMessage?chat_id=' + bot_chatID + '&parse_mode=Markdown&text=' + bot_message
    response = requests.get(send_text)
    return response.json()

class ApiView(BaseView):
    route_base = "/api"
    
    @expose("/telegram/<message>", methods=['GET'])
    def telegramdo(self,message):
        j = telegram_send(message) # 這邊呼叫 def 要放 class 外面,.. 這邊是 telegram 簡易跳轉程式
        return j
      
    @expose("/list", methods=['GET'])
    def list(self):
        return jsonify({'res':'OK','apis':['/apiv1/devices','/apiv1/accesses','/apiv1/accounts']})

    @expose("/do/<p>", methods=['GET'])
    def alist(self,p):
        return jsonify({p:'OK'})
        
    @expose('/method', methods=['GET'])
    def method(self):
        dic = request.args
        return jsonify(dic)
        
    @expose('/dns/<function>', methods = ['POST'])
    def dns(self,function):
        if (request.is_json):
            content = request.get_json()
        else:
            content = ''
        print (function)
        return content


    @expose("/id/<p>/<s>", methods=['GET'])
    def alist(self,p,s):
        a = db.session.query(Member).filter_by(user = p,signature = s).first()
        if(a is None):
            d = []
        else:
            d = [{'user':a.user,'nickname':a.nickname,'user_password':a.user_password,'signature':a.signature,'point':a.point,'account':a.account,'password':a.password,'extno':a.extno}]
        return jsonify(d)

    @expose('/get', methods=['GET'])
    def method(self):
        dic = request.args
        if ('user' not in dic) and ('signature' not in dic):
            d =[]
        else:
            p = dic['user']
            s = dic['signature']
            a = db.session.query(Member).filter_by(user = p,signature = s).first()
            if(a is None):
                d = []
            else:
                d = [{'user':a.user,'nickname':a.nickname,'user_password':a.user_password,'signature':a.signature,'point':a.point,'account':a.account,'password':a.password,'extno':a.extno}]
        return jsonify(d)
        
    @expose("/wri/<u>/<s>/<p>", methods=['GET'])
    def alist(self,u,s,p):
        a = db.session.query(Member).filter_by(user = u,signature = s).first()
        if(a is None):
            d = []
        else:
            # update point new value
            a.point = p 
            db.session.commit()
            d = [{'user':a.user,'nickname':a.nickname,'user_password':a.user_password,'signature':a.signature,'point':a.point,'account':a.account,'password':a.password,'extno':a.extno}]
            dt=datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
            #wirte to other table (add new one)
            todo = Message_list(message='test '+user,send_request = dt, returnlog = 'test '+signature, point = 10)
            db.session.add(todo)
            db.session.commit()
        return jsonify(d)   

appbuilder.add_api(ApiView)
```

呼叫方式 
```
http://{domain}/api/{resource}/{params}
```

測試結果看一下
```
curl 'http://192.168.8.121:5000/api/list'
輸出 : {"apis":["/apiv1/devices","/apiv1/accesses","/apiv1/accounts"],"res":"OK"}

curl 'http://192.168.8.121:5000/api/do/hello'
輸出 : {"hello":"OK"}

curl 'http://192.168.8.121:5000/api/method?data=user'
輸出 : {"data":"user"}

curl 'http://192.168.8.121:5000/api/dns/set' -X POST -H "Content-Type:application/json" -d '{"domain" : "test.com"}'
輸出 : {"domain":"test.com"}

```


資料庫的部分 拿我亂寫的

app/models.py
```
from flask_appbuilder import Model
from sqlalchemy.dialects.mysql import LONGTEXT
from sqlalchemy.orm import relationship
from sqlalchemy import Column, Integer, String, DateTime, Text

class Member(Model):
    id = Column(Integer, primary_key=True)
    user =  Column(String(30), unique = True, nullable=False)
    nickname =  Column(String(50))
    user_password = Column(String(30))
    signature = Column(String(50))
    point = Column(Integer)
    account = Column(String(50))
    password = Column(String(50))
    extno = Column(String(50))
    def __repr__(self):
        return self.name

class Message_list(Model):
    id = Column(Integer, primary_key=True)
    message =  Column(Text)
    send_request =  Column(DateTime)
    returnlog = Column(LONGTEXT)
    point = Column(Integer)
    def __repr__(self):
        return self.message

```

app/views.py
```
from flask import render_template
from flask_appbuilder.models.sqla.interface import SQLAInterface
from flask_appbuilder import ModelView, ModelRestApi
from . import appbuilder, db

from flask_appbuilder import ModelView,AppBuilder,expose,BaseView,has_access
from .models import Member,Message_list

from flask_appbuilder.api import BaseApi, expose
from flask import jsonify

class MemberModelView(ModelView):
    datamodel = SQLAInterface(Member)
    #修改列名称
    label_columns = {'user':'帳號','nickname':'帳戶名稱','password':'密碼','signature':'客戶金鑰','point':'點數','account':'供應商帳號'}
    #定义列显示名称
    list_columns = ['user','nickname','password','signature','point','account'] #定义视图中要显示的字段
    show_fieldsets = [
        (
            'Summary',
            {'fields':['user','nickname','password','signature','point','account']}
        ),
        ]

class MessagelistModelView(ModelView):
    datamodel = SQLAInterface(Message_list)
    #只能列表
    base_permissions = ['can_list','can_show']
    #一頁五筆
    page_size=5
    #修改列名称
    label_columns = {'message':'傳送資訊','send_request':'傳送時間','returnlog':'回應資訊','point':'剩餘點數'}
    #定义列显示名称
    list_columns = ['send_request','message','returnlog','point'] #定义视图中要显示的字段
    show_fieldsets = [
        (
            'Summary',
            {'fields':['send_request','message','returnlog','point']}
        ),
        ]

# 首先使用db.create_all()根据数据库模型创建表，然后再将视图添加到菜单。
db.create_all()
appbuilder.add_view(MemberModelView,'會員列表',icon = 'fa-address-card-o',category = '會員資訊')
appbuilder.add_view(MessagelistModelView,'紀錄列表',icon = 'fa-address-card-o',category = '紀錄')
```

這個還沒試應該是OK的
```
#todo = Todo(content='hacking')
#db.session.add(todo)
#db.session.commit()
#todo = Todo.query.filter_by(contant='hacking').update({'content': 'reading'})
#db.session.commit()
# todo = Todo.query.filter_by(content='reading').first()
# db.session.delete(todo)
#db.session.commit()
```
詳細在這裡 : 全會還真難

https://flask-appbuilder.readthedocs.io/en/latest/



