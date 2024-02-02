---
title: ice配合tornado搭建http服务
date: 2020-07-19 21:23:40
tags: tornado
categories: Python
---

tornado 是 python 中的一个 http 框架，以异步高性能著称，zeroc-ice 是一个分布式的 rpc 框架，将两者结合使用，可以搭建高性能的 web 应用服务。

下面写了一个小 demo，演示了如何在 tornado 中使用 ice。

<!--more-->

## demo结构

demo 的结构如下：

```txt
.
├── app.py
├── handlers.py
├── Test
│   └── __init__.py
├── Test.ice
├── Test_ice.py
└── user
    ├── client.ini
    ├── client.py
    ├── __init__.py
    ├── server.ini
    └── server.py

```

其中，Test.ice 是 slice 文件，其内容如下：

```txt
module Test
{
    dictionary<string, string> Params;

    interface User
    {
        void printStr(string s);

        int addMun(int num1, int num2);

        string invoke(string fid, string params);
    }
};
```

执行 slice2py 后，生成了 Test 目录和 Test_ics.py 文件。

handlers.py 和 app.py 是 tornado 文件，内容分别如下：

handlers.py：

```txt
# @Author : WZ 
# @Time : 2020/7/19 13:48 
# @Intro :


import json
import tornado.web

from user.client import IceCommunicator


class UserHandler(tornado.web.RequestHandler):

    def post(self, url):
        print("url:", url)
        # print("params:", self.request.body.decode("utf-8"))
        print("params:", self.request.arguments)

        path = "user/" + url
        params = self.request.arguments
        params = {k: params.get(k)[0].decode("utf-8") for k in params.keys()}

        resp = self.invoke(path, params)
        self.write(resp)
        self.finish()

    def invoke(self, path, params):
        communicator = IceCommunicator()
        user = communicator.init()
        params = json.dumps(params)
        resp = user.invoke(path, params)
        return resp


class GoodsHandler(tornado.web.RequestHandler):

    def post(self):
        pass
```

app.py：

```txt
# @Author : WZ 
# @Time : 2020/7/19 13:46 
# @Intro :


import tornado.web
import tornado.ioloop
import tornado.options
import tornado.httpserver
from tornado.options import define, options

from handlers import UserHandler, GoodsHandler

define("port", default=8000, help="run on the given port", type=int)


def run():
    handlers = [
        (r"/user/(.*)", UserHandler),
        (r"/goods/(.*)", GoodsHandler)
    ]
    app = tornado.web.Application(handlers=handlers)
    http_server = tornado.httpserver.HTTPServer(app)
    http_server.listen(options.port)
    tornado.ioloop.IOLoop.instance().start()


if __name__ == '__main__':
    run()
```

user目录下是 ice 相关的文件和配置，内容分别如下：

server.py：

```txt
# @Author : WZ 
# @Time : 2020/7/19 10:00 
# @Intro :

import sys
import Ice
import json

Ice.loadSlice("../Test.ice")
import Test


def get_resp(status=None, msg=None, data=None):
    """获取响应信息
    """
    resp = dict()
    resp["status"] = status if status else '10000'
    resp["msg"] = msg if msg else 'ok'
    resp["data"] = data if data else {}
    return resp


class UserI(Test.User):

    def printStr(self, s, current=None):
        print("hello ice")

    def addMun(self, n1, n2, current=None):
        ret = n1 + n2
        print("addMun result is:", ret)
        return ret

    def invoke(self, path, params, current=None):
        """调用具体业务逻辑
        """
        user = ViewUser()

        params = json.loads(params)

        map = {
            "user/list": user.list,
            "user/detail": user.detail,
            "user/add": user.add
        }

        resp = map.get(path, user.others)(params)
        return resp


class ViewUser():

    def list(self, params):
        resp = get_resp(data={"url": "user/list"})
        print("user list:", params)
        return json.dumps(resp)

    def detail(self, params):
        resp = get_resp(data={"url": "user/detail"})
        print("user detail:", params)
        return json.dumps(resp)

    def add(self, params):
        resp = get_resp(data={"url": "user/add"})
        print("user add:", params)
        return json.dumps(resp)

    def others(self, params):
        resp = get_resp('10001', 'not found')
        return json.dumps(resp)


with Ice.initialize(sys.argv, "server.ini") as communicator:
    adapter = communicator.createObjectAdapter("User")
    adapter.add(UserI(), Ice.stringToIdentity("user"))
    adapter.activate()
    communicator.waitForShutdown()
```

client.py：

```txt
# @Author : WZ 
# @Time : 2020/7/19 10:00 
# @Intro :

import sys
import Ice

Ice.loadSlice("Test.ice")
import Test


class IceCommunicator:

    def init(self):
        communicator = Ice.initialize(sys.argv, 'user/client.ini')
        user = Test.UserPrx.checkedCast(communicator.propertyToProxy('User.Proxy')
                                        .ice_twoway()
                                        .ice_secure(False))
        if not user:
            print("invalid proxy")
            sys.exit(1)

        return user
```

server.ini：

```txt
User.Endpoints=tcp -p 10000
Ice.Default.Host=localhost
Ice.Warn.Connections=1
```

client.ini：

```txt
User.Proxy=user:tcp -p 10000
Ice.Default.Host=localhost
Ice.Warn.Connections=1
```



## demo执行流程

执行流程如下：

```txt
1. app.py 中接受 http 请求，并将请求转发到对应的 handler
2. 在 hendler 中调用 ice 客户端向 ice 服务端发送请求，请求 ice 服务端时会传入请求 path 和参数
3. ice 服务端根据不同的请求 path 执行不同的操作，并返回响应结果
```



上面仅仅是一个小的 demo，后面还会更近一步研究两者的结合使用。