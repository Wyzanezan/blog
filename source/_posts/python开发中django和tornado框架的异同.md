---
title: python开发中django和tornado框架的异同
date: 2020-05-30 12:30:35
tags: django
categories: Python
---

python中常用的几个web框架有django, tornado, flask等，今天来总结一下django和tornado的不同。

<!--more-->

工作中django和tornado都用过，使用django相对更多一些。个人感觉django虽然好用，有搭建项目快、自带ORM、自动生成路由、自带管理后台等优势；但若实际工作中选择，我还是会偏向于使用tornado框架，因为torndo使用更加灵活，并且支持websocket,tcp等通信协议,最重要的是tornado是异步非阻塞的web框架；而在django中要实现websocket、异步非阻塞等功能则需要引入dwebsocket、celery等第三方模块。

本文使用的环境是python3.6, django2.0, tornado5.1。

下面主要从以下几个方面介绍一下这两个框架的不同：
1.创建项目的方式
2.数据库连接
3.异步非阻塞请求
4.websocket的使用

# 项目创建方式

## django

django主要是通过下面两个命令创建项目：

```python
django-admin startproject Test  # 创建项目，名称为Test
django-admin startpapp Test01   # 创建app，名称为Test01
```

执行完成后，会生成如下的目录结构：

```txt
D:.
│  manage.py
│  test.txt
│  
├─.idea
│  │  misc.xml
│  │  modules.xml
│  │  Test.iml
│  │  workspace.xml
│  │  
│  └─inspectionProfiles
│          profiles_settings.xml
│          
├─Test
│      settings.py
│      urls.py
│      wsgi.py
│      __init__.py
│      
└─Test01
    │  admin.py
    │  apps.py
    │  models.py
    │  tests.py
    │  views.py
    │  __init__.py
    │  
    └─migrations
            __init__.py
```

主要是manage.py,Test,Test01这几个文件和文件夹，
manage.py是管理项目的文件，通过它运行django的一些内置命令，如模型迁移、启动项目等；
Test/settings.py是配置文件，项目配置存放在这里
Test/urls.py是路由文件，负责分发http请求
Test01/models.py是模型文件，Test01下创建的模型就放在这里，模型负责将表结构映射到数据库中
Test01/views.py是视图文件，django中的视图在这里定义
Test01/migrations目录中存放迁移后生成的迁移文件。
django项目的基本结构就是这样。



## tornado

tornado项目的创建比较灵活，没有什么项目名称和app的概念，全靠自己组织项目，就是创建一个个python文件和python package。可以像下面一样来组织tornado项目：

```txt
├── App
│   ├── __init__.py
│   ├── Shop
│   │   ├── __init__.py
│   │   └── views.py
│   └── User
│       ├── __init__.py
│       └── views.py
├── application.py
├── Config
│   ├── config_base.py
│   ├── config_db.conf
│   ├── config_db_get.py
│   ├── config_engine.py
│   ├── __init__.py
├── Models
│   ├── __init__.py
│   ├── Shop
│   │   └── __init__.py
│   └── User
│       ├── BaseClass.py
│       ├── __init__.py
│       └── UserModel.py
├── server.py
├── static
│   └── __init__.py
├── templates
│   └── __init__.py
├── test.py
└── Urls
    ├── __init__.py
    ├── Shop.py
    └── User.py
```

这里有几个主要文件App, Config, Models, Urls, static, templates, application.py, server.py。
项目的app可以集中放在App目录中，与数据库对应的模型文件可以放在Models中，http路由可以放在Urls中，项目配置信息可以放在Config目录中，静态文件和模板分别放在static和templates中。application.py文件可以加载路由信息和项目配置信息，server.py文件负责启动项目。
项目的基本配置信息可以放在Config/config_base.py中，如下：

```python
# coding=utf-8


import os


BASE_DIR = os.path.dirname(__file__)

# 参数
options = {
    "port": 8001,
}


# 基本配置信息
settings = {
    "debug": True,
    "static_path": os.path.join(BASE_DIR, "static"),
    "template_path": os.path.join(BASE_DIR, "templates")
}
```

路由信息可以放在Urls/User.py中，如下：

```python
# coding=utf-8


from App.UserInfo import views


user_urls = [
    (r'/user/', views.IndexHandler),
]
```

application.py中加载路由信息和配置信息：

```python
# coding=utf-8


from tornado import web

from Config.config_base import settings
from Urls.UserInfo import user_urls
from Urls.Shop import shop_urls


"""
路由配置
"""


class Application(web.Application):

    def __init__(self):
        urls = user_urls + shop_urls
        super(Application, self).__init__(urls, **settings)
```

最后在server.py中启动项目：

```python
# coding=utf-8


from tornado import ioloop, httpserver

from application import Application
from Config import config_base


if __name__ == '__main__':
    app = Application()
    http_server = httpserver.HTTPServer(app)
    http_server.listen(config_base.options.get("port"))

    ioloop.IOLoop.current().start()
```



# 数据库连接

## django

django中使用数据库时，首先要在settings.py中配置数据库信息：

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',   # 数据库引擎
        'NAME': 'django_test',                  # 你要存储数据的库名，事先要创建之
        'USER': 'root',                         # 数据库用户名
        'PASSWORD': 'test',                     # 密码
        'HOST': 'localhost',                    # 主机
        'PORT': '3306',                         # 数据库使用的端口
    }
}
```

然后在每个app下编写完models.py后，执行以下两个命令后，就可以使用数据库了：

```python
python manage.py makemigrations
python manage.py migrate
```

可以调用模型管理器对象objects的相应方法，执行增删改查等操作。

## tornado

这里说一下在tornado中使用sqlalchemy连接数据库，需要安装sqlalchemy和pymysql。

### 首先在Config/config_db.conf中配置数据库信息

```ini
[db_user]
name = db_tornado03
port = 3306
user = root
host = 127.0.0.1
pass = test
pool_size = 3
```

### 然后在Config/config_engine.py中配置engine

```python
# coding=utf-8


from sqlalchemy import create_engine

from Config.config_db_get import ConfigDBUser


# 数据库配置信息 可以配置多个engine, 每个数据库对应一个engine
db_user = ConfigDBUser("db_user")
engine_user = create_engine(
    "mysql+pymysql://%s:%s@%s:%d/%s" % (
        db_user.get_db_user(),
        db_user.get_db_pass(),
        db_user.get_db_host(),
        db_user.get_db_port(),
        db_user.get_db_database()
    ),
    encoding='utf-8',
    echo=True,
    pool_size=20,
    pool_recycle=100,
    connect_args={"charset": 'utf8mb4'}
)
```

create_engine用来初始化数据库连接。

### 在Models/UserInfo/BaseClass.py中配置连接数据库的session信息

```python
# coding=utf-8


from sqlalchemy.orm import scoped_session, sessionmaker

from Config.config_engine import engine_user


class BaseClass:
    def __init__(self):
        # 创建session对象，并且用scoped_session维护session对象
        # 数据库的增删改查通过session对象来完成
        self.engine_user = scoped_session(
            sessionmaker(
                bind=engine_user,
                autocommit=False,
                autoflush=True,
                expire_on_commit=False
            )
        )
```

### 在Models/UserInfo/UserModel.py中配置模型信息，用于映射到数据库中对应的表

```python
# coding=utf-8


from sqlalchemy import Table, MetaData
from sqlalchemy.ext.declarative import declarative_base


from Config.config_engine import engine_user


BaseModel = declarative_base()


def user_model(table_name):
    class UserModel(BaseModel):
        __tablename__ = table_name
        metadata = MetaData(engine_user)

        Table(__tablename__, metadata, autoload=True)
    return UserModel
```

配置模型信息前，需要在数据库中把表创建好，这是就需要写sql语句创建表了。对于熟练sql的同学，写sql语句应该不算什么；对应不熟悉sql的同学，可能更习惯于django中那种创建表的方式。

### 在视图中使用

App/UserInfo/views.py：

```python
# coding=utf-8


from tornado import web
from Models.UserInfo.BaseClass import BaseClass
from Models.UserInfo.UserModel import user_model


class UserInfoHandler(web.RequestHandler, BaseClass):
    def get(self):
        """
        获取用户信息
        :return:
        """
        # user_model中的参数对应数据库中的表名
        user_info = user_model("user_info")
        # 获取参数
        user_id = self.get_query_argument("id")

        # self.engine_user其实就是一个session对象；query()方法会返回一个query.Query对象，通过这个对象查询数据库
        user_info_obj = self.engine_user.query(user_info).filter(user_info.id==user_id).first()
        self.write(user_info_obj.name)
        self.finish()
```

### 最后配置路由

```python
Urls/UserInfo.py：
# coding=utf-8


from App.UserInfo import views


user_urls = [
    (r'/userinfo', views.UserInfoHandler),
]

application.py:
# coding=utf-8


from tornado import web

from Config.config_base import settings
from Urls.UserInfo import user_urls
from Urls.Shop import shop_urls


"""
路由配置
"""


class Application(web.Application):

    def __init__(self):
        urls = user_urls + shop_urls
        super(Application, self).__init__(urls, **settings)
```

启动服务后，就可以访问了。



# 异步非阻塞请求

## django

django中可以通过celery来实现异步任务，也可以使用asyncio和aiohttp实现异步。下面讲一下celery的使用:
首先需要安装 celery和 django-celery，使用pip安装就行了；
然后在zsettings.py中进行如下配置：

```python
在INSTALLED_APPS中加入djcelery。
import djcelery
# Celery便会去查看INSTALLD_APPS下包含的所有app目录中的tasks.py文件，找到标记为task的方法，将它们注册为celery task
djcelery.setup_loader()
BROKER_URL = 'redis://127.0.0.1:6379/2'
CELERY_RESULT_BACKEND = 'redis://127.0.0.1:6379/3'  
# 或者使用rabbitmq: 
BROKER_URL = 'amqp://test:test@192.168.173.1:5672/testhost'
CELERY_RESULT_BACKEND = 'amqp://test:test@192.168.173.1:5672/testhost'
```

在需要使用异步的app中创建tasks.py文件，然后编辑该文件：

```python
# coding=utf-8
import time
from celery import task
@task
def test(data):
    """
    预处理
    :param data:
    :return:
    """
    time.sleep(3)
    return data
```

耗时的任务就可以放在使用@task修饰的函数中
在views.py中调用tasks.py中的函数：

```python
from rest_framework.response import Response
from .tasks import test


class CeleryTrainView(APIView):
    def get(self, request):
        try:
            for i in range(0, 5):
                ret = test.delay(str(i))
                print("ret:", ret)
        except Exception as e:
            return Response(dict(msg=str(e), code=10001))
        return Response(dict(msg="OK", code=10000))
```

上面的结果ret是一个AsyncResult对象，可以通过这个对象拿到保存在CELERY_RESULT_BACKEND中的结果。如果想立即得到结果，可以直接调用get()方法，但是这样就会阻塞其他请求，直到结果返回：

```python
from rest_framework.response import Response
from .tasks import test


class CeleryTrainView(APIView):
    def get(self, request):
        try:
            for i in range(0, 5):
                ret = test.delay(str(i))
                print("ret:", ret.get())
        except Exception as e:
            return Response(dict(msg=str(e), code=10001))
        return Response(dict(msg="OK", code=10000))
```

最后启动celery：

```python
#先启动服务器
python manage.py runserver
#再启动worker 
python manage.py celery worker
```



## tornado

tornado中实现异步有回调和协程这两种方式，这里只举一个协程实现异步的例子：

```python
from tornado import web
from tornado import gen
from tornado.httpclient import AsyncHTTPClient


class AsyncHandler(web.RequestHandler):
    @gen.coroutine
    def get(self, *args, **kwargs):
        client = AsyncHTTPClient()
        url = 'http://ip.taobao.com/service/getIpInfo.php?ip=14.130.112.24'
        # 根据ip地址获取相关信息
        resp = yield client.fetch(url)
        data = str(resp.body, encoding="utf-8")
        print("data:", data)
        self.write(data)
        self.finish()
```

或者像下面这样，把获取ip信息的部分封装成一个函数：

```python
from tornado import web
from tornado import gen
from tornado.httpclient import AsyncHTTPClient


class AsyncHandler(web.RequestHandler):
    @gen.coroutine
    def get(self, *args, **kwargs):
        ip_info = yield self.get_ip_info()
        self.write(ip_info)
        self.finish()

    @gen.coroutine
    def get_ip_info(self):
        client = AsyncHTTPClient()
        url = 'http://ip.taobao.com/service/getIpInfo.php?ip=14.130.112.24'
        resp = yield client.fetch(url)
        data = str(resp.body, encoding="utf-8")
        return data
```

也可以同时发起多个异步请求：

```python
from tornado import web
from tornado import gen
from tornado.httpclient import AsyncHTTPClient


class AsyncHandler(web.RequestHandler):
    @gen.coroutine
    def get(self, *args, **kwargs):
        ips = [
            "14.130.112.24",
            "14.130.112.23",
            "14.130.112.22"
        ]
        info1, info2, info3 = yield [self.get_ip_info(ips[0]), self.get_ip_info(ips[1]), self.get_ip_info(ips[2])]
        self.write(info1)
        self.write(info2)
        self.write(info3)
        self.finish()

    @gen.coroutine
    def get_ip_info(self, ip):
        client = AsyncHTTPClient()
        url = 'http://ip.taobao.com/service/getIpInfo.php?ip=' + ip
        resp = yield client.fetch(url)
        data = str(resp.body, encoding="utf-8")
        return data
```

AsyncHTTPClient的fetch()方法有两种调用方式，一种是像上面那样只传入一个url的字符串，另一种是接收一个HTTPRequest对象作为参数，像下面这样：

```python
@gen.coroutine
def get_ip_info(self, ip):
    client = AsyncHTTPClient()
    url = 'http://ip.taobao.com/service/getIpInfo.php?ip=' + ip
    header = {'Accept': 'application/json;charset=utf-8',
              'Content-Type': 'application/x-www-form-urlencoded;charset=utf-8'}
    param1 = 'test'
    http_request = HTTPRequest(url=url,
                               method='POST',
                               headers=header,
                               body=urlencode({'param1': param1}))
    resp = yield client.fetch(http_request)
    data = str(resp.body, encoding="utf-8")
    return data
```

# websocket的使用

## django

django中使用websocket需要安装第三方包dwebsocket。

## tornado

tornado中实现websocket功能需要用到tornado.websocket模块，主要有以下几个方法：open(), write_message(), on_message(), on_close()

```python
open(): 当websocket客户端连接时所做的操作
write_message(): 使用这个方法向客户端发送消息
on_message(): 接收并处理客户端的消息
on_close(): websocket关闭连接时所作的操作
```

下面看一个例子：

```python
views.py:

from tornado import websocket


class IndexHandler(web.RequestHandler):
    def get(self, *args, **kwargs):
        self.render("chat.html")

class ChatHandler(websocket.WebSocketHandler):
    clients = set()

    def open(self, *args, **kwargs):
        self.clients.add(self)
        for client in self.clients:
            client.write_message("%s上线了" % self.request.remote_ip)

    def on_message(self, message):
        for client in self.clients:
            client.write_message("%s: %s" % (self.request.remote_ip, message))

    def on_close(self):
        self.clients.remove(self)
        for client in self.clients:
            client.write_message("%s下线了" % self.request.remote_ip)

    def check_origin(self, origin):
        """
        用于处理跨域问题
        :param origin:
        :return:
        """
        return True
```

路由：

```python
# coding=utf-8


from App.UserInfo import views


user_urls = [
    (r'/index', views.IndexHandler),
    (r'/chat', views.ChatHandler),
]
```

chat.html：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>聊天室</title>
</head>
<body>
    <div id="content" style="height: 500px;overflow: auto;"></div>
    <div>
        <textarea id="msg"></textarea>
        <a href="javascript:;" onclick="sendMsg()">发送</a>
    </div>
    <script src="{{ static_url('js/jquery.min.js') }}"></script>
    <script type="text/javascript">
        var ws = new WebSocket("ws://192.168.1.104:8001/chat");
        ws.onmessage = function (data) {
            $("#content").append("<p>"+ data.data +"</p>")
        };
        function sendMsg() {
            var msg = $("#msg").val();
            if (msg) {
                ws.send(msg);
            }
        }
    </script>
</body>
</html>
```

上面一个例子通过websocket实现了简单的聊天室功能。

以上就简单的比较了django和tornado几个方面的不同，它们各有优缺点，实际工作中可以根据不同的需求选择不同的框架进行开发。