---
title: python开发之使用Redis实现发布订阅功能
date: 2020-05-30 12:17:20
tags: Redis
categories: Python
---

redis中的发布/订阅模型是一种消息通信模式，今天聊一下在python中实现简单的发布订阅功能。

<!--more-->

# 实现方式一

redis_helper.py：封装发布订阅方法

```python
import redis


class RedisHelper(object):

    def __init__(self):
        self.__conn = redis.Redis(host="localhost")
        # 订阅频道
        self.chan_sub = "fm104.5"

    def public(self, msg):
        """
        在指定频道上发布消息
        :param msg:
        :return:
        """
        # publish(): 在指定频道上发布消息，返回订阅者的数量
        self.__conn.publish(self.chan_sub, msg)
        return True

    def subscribe(self):
        # 返回发布订阅对象，通过这个对象你能1）订阅频道 2）监听频道中的消息
        pub = self.__conn.pubsub()
        # 订阅频道，与publish()中指定的频道一样。消息会发布到这个频道中
        pub.subscribe(self.chan_sub)
        ret = pub.parse_response()  # [b'subscribe', b'fm86', 1]
        print("ret:%s" % ret)
        return pub
```



redis_pub.py： 发布者

```python
from redis_helper import RedisHelper


obj = RedisHelper()
for i in range(5):
    obj.public("hello_%s" % i)
```

redis_sub.py： 订阅者

```python
from redis_helper import RedisHelper


obj = RedisHelper()
redis_sub = obj.subscribe()
while True:
    msg = redis_sub.parse_response()
    print(msg)
```

# 实现方式二

redis_helper.py： 封装发布订阅方法

```python
import redis


class RedisHelper(object):

    def __init__(self):
        self.__conn = redis.Redis(host="localhost")
        # 频道名称
        self.chan_sub = "orders"

    def public(self, msg):
        """
        在指定频道上发布消息
        :param msg:
        :return:
        """
        # publish(): 在指定频道上发布消息，返回订阅者的数量
        self.__conn.publish(self.chan_sub, msg)
        return True

    def subscribe(self):
        # 返回发布订阅对象，通过这个对象你能1）订阅频道 2）监听频道中的消息
        pub = self.__conn.pubsub()
        # 订阅某个频道，与publish()中指定的频道一样。消息会发布到这个频道中
        pub.subscribe(self.chan_sub)
        return pub
```

redis_pub.py：

```python
from redis_helper import RedisHelper


obj = RedisHelper()
for i in range(5):
    obj.public("hello_%s" % i)
```

redis_sub.py：

```python
from redis_helper import RedisHelper

obj = RedisHelper()
redis_sub = obj.subscribe()
while True:
    # listen()函数封装了parse_response()函数
    msg = redis_sub.listen()
    for i in msg:
        if i["type"] == "message":
            print(str(i["channel"], encoding="utf-8") + ":" + str(i["data"], encoding="utf-8"))
        elif i["type"] == "subscrube":
            print(str(i["chennel"], encoding="utf-8"))
```

以上两种方式的不同之处在于，方式一使用发布订阅对象的parse_response()方法获取订阅信息，方式二使用发布订阅对象的listen()方法获取订阅信息。listen()方法是对parse_response()方法的封装，加入了阻塞，并将parse_response()返回的结果进行了处理，使结果更加简单。