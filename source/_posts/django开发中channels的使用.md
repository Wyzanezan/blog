---
title: django开发中channels的使用
date: 2020-05-05 10:02:12
tags: django
categories: Python
---

今天介绍下在django中使用channels实现websocket的步骤。

<!--more-->

首先介绍下channels。
channels官方文档连接如下：
https://channels.readthedocs.io/en/latest/

channels这个第三方包可以帮助django在http之上扩展很多功能，可以使django实现websocket，聊天协议等。channels可以使django运行在同步模式下，但是异步的处理connections和socket。

下面介绍在django中，使用channels实现websocket功能。下面DjangoChannel是项目名称，test01是app名称。

# 环境

python3.6 channels2.1.7 channels-redis2.3.3 django2.0.5 drf3.9.2

# 前端页面

index.html：

```html
<!DOCTYPE html>
<html>
<head>
    <title>cchannels实现websocket</title>
    <script src="http://code.jquery.com/jquery-1.11.1.min.js"></script>
    <script type="text/javascript">//<![CDATA[
    $(function () {
        $('#sendMsg').click(function () {
            var socket = new WebSocket("ws://127.0.0.1:8000/ws/status/1")
            socket.onopen = function () {
                console.log('WebSocket open') //成功连接上Websocket
                socket.send($('#msg').val())  //发送数据到服务端
            }
            socket.onmessage = function (ret) {
                var data = JSON.parse(ret.data)
                var msg = data["message"]
                console.log('message: ' + msg)//打印服务端返回的数据
                $('#retMsg').prepend('<p>' + msg + '</p>')
            }
        })
    })
    //]]></script>
</head>
<body>
    <br>
    <input type="text" id="msg" value="Hello, World!"/>
        <button type="button" id="sendMsg">发送</button>
    <h1>Received Messages</h1>
    <div id="retMsg"></div>
</body>
</html>
```

# routing配置

test01/routing.py：

```python
from django.urls import path
from . import consumers


websocket_urlpatterns = [
    path('ws/status/&lt;client_id&gt;', consumers.ServiceConsumer),
]
```

DjangoChannel/routing.py：

```python
from channels.auth import AuthMiddlewareStack
from channels.routing import ProtocolTypeRouter, URLRouter
import test01.routing


application = ProtocolTypeRouter({
    'websocket': AuthMiddlewareStack(
        URLRouter(
            test01.routing.websocket_urlpatterns
        )
    )
})
```

# consumer的编写

```python
import json
from channels.generic.websocket import AsyncWebsocketConsumer


class ServiceConsumer(AsyncWebsocketConsumer):
    async def connect(self):
        """
        连接成功后执行该函数
        :return:
        """
        # self.scope中保存了请求头、请求体等信息
        self.client_id = self.scope['url_route']['kwargs']['client_id']
        self.room_group_name = 'group_%s' % self.client_id

        print("scope:", self.scope)
        print("client id:", self.client_id)  # client id: 1
        print("channel layer:", self.channel_layer)  # channel layer: RedisChannelLayer(hosts=[{'address': ('127.0.0.1', 6379)}])
        print("channel name:", self.channel_name)  # channel name: specific.VgcSONTt!RxtpUIegoTzX
        print("room group name:", self.room_group_name)  # room group name: group_1

        await self.channel_layer.group_add(
            self.room_group_name,
            self.channel_name
        )
        print("channel layer:", self.channel_layer)  # channel layer: RedisChannelLayer(hosts=[{'address': ('127.0.0.1', 6379)}])
        await self.accept()

    async def disconnect(self, close_code):
        """
        与客户端断开连接
        :param close_code:
        :return:
        """
        await self.channel_layer.group_discard(
            self.room_group_name,
            self.channel_name
        )

    async def receive(self, text_data):
        """
        接受客户端websocket发送过来的信息
        :param text_data:
        :return:
        """
        print("test data:", text_data)  # test data: Hello, World!
        print("group name:", self.room_group_name)  # group name: group_1

        message = text_data

        # 向指定room group中发送一个event信息，chat_message为对应处理函数
        # 会调用下面的chat_message()方法
        await self.channel_layer.group_send(
            self.room_group_name,
            {
                'type': 'chat_message',
                'message': message,
            }
        )

    async def chat_message(self, event):
        """
        接收room group中传过来的信息，并把信息发送给websocket，发送给前端
        :param event:
        :return:
        """
        message = event['message']
        print("event:", event)  # event: {'type': 'chat_message', 'message': 'Hello, World!'}

        # 向前端发送消息
        await self.send(text_data=json.dumps({
            'message': message
        }))
```

# settings配置

DjangoChannel/asgi.py：

```python
import os
import django
from channels.routing import get_default_application


os.environ.setdefault("DJANGO_SETTINGS_MODULE", "DjangoChannels2.settings")
django.setup()
application = get_default_application()
```

DjangoChannel/settings.py：

```python
INSTALLED_APPS = [
    ...
    'channels',
]

CHANNEL_LAYERS = {
    'default': {
    'BACKEND': 'channels_redis.core.RedisChannelLayer',
    'CONFIG':
        {
            "hosts": [('127.0.0.1', 6379)],
        }
    }
}

ASGI_APPLICATION = "DjangoChannel.routing.application"
```

到这里，整个项目就完成了。当访问index.html后，在输入框中输入消息，点击发送按钮后，后端就会接收到消息，并将消息又发回到页面上。