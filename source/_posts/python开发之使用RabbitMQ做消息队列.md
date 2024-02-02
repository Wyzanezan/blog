---
title: python开发之使用RabbitMQ做消息队列
date: 2020-05-30 12:03:39
tags: RabbitMQ
categories: Python
---

最近在研究redis做消息队列时，顺便看了一下RabbitMQ做消息队列的实现。以下是总结的RabbitMQ中三种exchange模式的实现，分别是fanout, direct和topic。

<!--more-->

# 实例化MQ

base.py：

```python
import pika


# 获取认证对象，参数是用户名、密码。远程连接时需要认证
credentials = pika.PlainCredentials("admin", "admin")

# BlockingConnection(): 实例化连接对象
# ConnectionParameters(): 实例化链接参数对象
connection = pika.BlockingConnection(pika.ConnectionParameters(
    "192.168.0.102", 5672, "/", credentials))

# 创建新的channel(通道)
channel = connection.channel()
```



# fanout模式

fanout模式会向绑定到指定exchange的queue中发送消息，消费者从queue中取出数据，类似于广播模式、发布订阅模式。
绑定方式: 在接收端channel.queue_bind(exchange=”logs”, queue=queue_name)
代码：
publisher.py：

```python
from base import channel, connection

# 声明exchange, 不声明queue
channel.exchange_declare(exchange="logs", exchange_type="fanout")  # 广播
message = "hello fanout"
channel.basic_publish(
    exchange="logs",
    routing_key="",
    body=message
)
connection.close()
```



consumer.py：

```python
from base import channel, connection

# 声明exchange
channel.exchange_declare(exchange="logs", exchange_type="fanout")

# 不指定queue名字, rabbitmq会随机分配一个名字, 消息处理完成后queue会自动删除
result = channel.queue_declare(exclusive=True)  

# 获取queue名字
queue_name = result.method.queue

# 绑定exchange和queue
channel.queue_bind(exchange="logs", queue=queue_name)


def callback(ch, method, properties, body):
    print("body:%s" % body)


channel.basic_consume(
    callback,
    queue=queue_name
)


channel.start_consuming()
```



# direct模式

direct模式中发送端会绑定一个routing_key1, queue中绑定若干个routing_key2, 若key1与key2相等，或者key1在key2中，则消息就会发送到这个queue中，再由相应的消费者去queue中取数据。
publisher.py：

```python
from base import channel, connection


channel.exchange_declare(exchange="direct_test", exchange_type="direct")

message = "hello"

channel.basic_publish(
    exchange="direct_test",
    routing_key="info",  # 绑定key
    body=message
)
connection.close()
```

consumer01.py：

```python
from base import channel, connection


channel.exchange_declare(exchange="direct_test", exchange_type="direct")
result = channel.queue_declare(exclusive=True)
queue_name = result.method.queue


channel.queue_bind(
    exchange="direct_test",
    queue=queue_name,
    # 绑定的key,与publisher中的相同
    routing_key="info"  
)


def callback(ch, method, properties, body):
    print("body:%s" % body)


channel.basic_consume(
    callback,
    queue=queue_name
)


channel.start_consuming()
```

consumer02.py：

```python
from base import channel, connection


channel.exchange_declare(exchange="direct_test", exchange_type="direct")
result = channel.queue_declare(exclusive=True)
queue_name = result.method.queue


channel.queue_bind(
    exchange="direct_test",
    queue=queue_name,
    # 绑定的key
    routing_key="error"   
)


def callback(ch, method, properties, bosy):
    print("body:%s" % body)


channel.basic_consume(
    callback,
    queue=queue_name
)


channel.start_consuming()
```

consumer03.py：

```python
from base import channel, connection


channel.exchange_declare(exchange="direct_test", exchange_type="direct")
result = channel.queue_declare(exclusive=True)
queue_name = result.method.queue


key_list = ["info", "warning"]
for key in key_list:
    channel.queue_bind(
        exchange="direct_test",
        queue=queue_name,
        # 一个queue同时绑定多个key，有一个key满足条件时就可以收到数据
        routing_key=key  
    )


def callback(ch, method, properties, body):
    print("body:%s" % body)


channel.basic_consume(
    callback,
    queue=queue_name
)


channel.start_consuming()
```

执行：

```python
python producer.py
python consumer01.py
python consumer02.py
python consumer03.py
```

打印结果如下：

```python
consumer01.py: body:b'hello'
consumer02.py没收到结果
consumer03.py: body:b'hello'
```



# topic模式

topic模式不是太好理解，我的理解如下:
对于发送端绑定的routing_key1，queue绑定若干个routing_key2；若routing_key1满足任意一个routing_key2，则该消息就会通过exchange发送到这个queue中，然后由接收端从queue中取出其实就是direct模式的扩展。

发送端绑定：

```python
channel.basic_publish(
    exchange="topic_logs",
    routing_key=routing_key,
    body=message
)
```

接收端绑定：

```python
channel.queue_bind(
    exchange="topic_logs",
    queue=queue_name,
    routing_key=binding_key
)
```

publisher.py：

```python
import sys
from base import channel, connection


# 声明exchange
channel.exchange_declare(exchange="topic_test", exchange_type="topic")

# 待发送消息
message = " ".join(sys.argv[1:]) or "hello topic"

# 发布消息
channel.basic_publish(
    exchange="topic_test",
    routing_key="mysql.error",   # 绑定的routing_key
    body=message
)
connection.close()
```

consumer01.py：

```python
from base import channel, connection


channel.exchange_declare(exchange="topic_test", exchange_type="topic")
result = channel.queue_declare(exclusive=True)
queue_name = result.method.queue


channel.queue_bind(
    exchange="topic_test",
    queue=queue_name,
    routing_key="*.error"    # 绑定的routing_key
)


def callback(ch, method, properties, body):
    print("body:%s" % body)


channel.basic_consume(
    callback,
    queue=queue_name,
    no_ack=True
)


channel.start_consuming()
```

consumer02.py：

```python
from base import channel, connection


channel.exchange_declare(exchange="topic_test", exchange_type="topic")
result = channel.queue_declare(exclusive=True)
queue_name = result.method.queue


channel.queue_bind(
    exchange="topic_test",
    queue=queue_name,
    routing_key="mysql.*"    # 绑定的routing_key
)


def callback(ch, method, properties, body):
    print("body:%s" % body)


channel.basic_consume(
    callback,
    queue=queue_name,
    no_ack=True
)


channel.start_consuming()
```

执行：

```python
python publisher02.py "this is a topic test"
python consumer01.py
python consumer02.py
```

打印结果：

```python
consumer01.py的结果: body:b'this is a topic test'
consumer02.py的结果: body:b'this is a topic test'
```

说明通过绑定相应的routing_key，两个消费者都收到了消息

将publisher.py的routing_key改成”mysql.info”
再次执行：

```python
python publisher02.py "this is a topic test"
python consumer01.py
python consumer02.py
```

结果：

```python
consumer01.py没收到结果
consumer02.py的结果: body:b'this is a topic test'
```

通过这个例子我们就能明白topic的运行方式了。



RabbitMQ的三种exchange模式就介绍到这里。