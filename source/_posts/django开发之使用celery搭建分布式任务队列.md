---
layout: 'n'
title: django开发之使用celery搭建分布式任务队列
date: 2020-05-24 15:35:27
tags: django
categories: Python
---

今天介绍一下如何在django项目中使用celery搭建一个有两个节点的任务队列(一个主节点一个子节点；主节点发布任务，子节点收到任务并执行。搭建3个或者以上的节点就类似了)，使用到了celery,rabbitmq。这里不会单独介绍celery和rabbitmq中的知识了。

<!--more-->



# 基础环境

两个ubuntu18.04虚拟机、python3.6.5、django2.0.4、celery3.1.26post2

# 项目结构

```txt
proj
 - celery1
	 - admin.py
	 - apps.py
	 - models.py
	 - tasks.py
	 - tests.py
	 - urls.py
	 - views.py
 - proj
	 - celery.py
	 - settings.py
	 - urls.py
	 - wsgi.py
 - manage,py
```

# celery的配置

settings.py中关于celery的配置：

```python
import djcelery
# 此处的Queue和Exchange都涉及到RabbitMQ中的概念，这里不做介绍
from kombu import Queue, Exchange
djcelery.setup_loader()
BROKER_URL = 'amqp://test:test@192.168.43.6:5672/testhost'
CELERY_RESULT_BACKEND = 'amqp://test:test@192.168.43.6:5672/testhost'

CELERY_TASK_RESULT_EXPIRES=3600
CELERY_TASK_SERIALIZER='json'
CELERY_RESULT_SERIALIZER='json'
# CELERY_ACCEPT_CONTENT = ['json', 'pickle', 'msgpack', 'yaml']

CELERY_DEFAULT_EXCHANGE = 'train'
CELERY_DEFAULT_EXCHANGE_TYPE = 'direct'

CELERY_IMPORTS = ("proj.celery1.tasks", )

CELERY_QUEUES = (
  Queue('train', routing_key='train'),
  Queue('predict', routing_key='predict'),
)
```

celery.py中的配置：

```python
# coding:utf8
from __future__ import absolute_import

import os

from celery import Celery
from django.conf import settings

# set the default Django settings module for the 'celery' program.

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'proj.settings')

app = Celery('proj')

# Using a string here means the worker will not have to
# pickle the object when using Windows.
app.config_from_object('django.conf:settings')
# app.autodiscover_tasks(lambda: settings.INSTALLED_APPS)
app.autodiscover_tasks(settings.INSTALLED_APPS)


@app.task(bind=True)
def debug_task(self):
    print('Request: {0!r}'.format(self.request))
```

proj/**init**.py中的配置：

```python
from __future__ import absolute_import
from .celery import app as celery_app
```

celery1/tasks.py：（主节点中的任务不会执行，只执行子节点中的任务）

```python
from __future__ import absolute_import
from celery import task


@task
def do_train(x, y):
    return x + y
```

celery1/views.py：

```python
from .tasks import do_train
class Test1View(APIView):
    def get(self, request):
        try:
            # 这里的queue和routing_key也涉及到RabiitMQ中的知识
            # 关键，在这里控制向哪个queue中发送任务，子节点通过这个执行对应queue中的任务
            ret = do_train.apply_async(args=[4, 2], queue="train", routing_key="train")
            # 获取结果
            data = ret.get()
        except Exception as e:
            return Response(dict(msg=str(e), code=10001))
        return Response(dict(msg="OK", code=10000, data=data))
```



# 子节点信息

## 目录结构

```txt
celery1
 - celery.py
 - config.py
 - tasks.py
```



## 配置

子节点中celery1/celery.py：

```python
from __future__ import absolute_import
from celery import Celery
CELERY_IMPORTS = ("celery1.tasks", )
app = Celery('myapp',
             # 此处涉及到RabbitMQ的知识,RabbitMQ是主节点上的
             broker='amqp://test:test@192.168.43.6:5672/testhost',
             backend='amqp://test:test@192.168.43.6:5672/testhost',
             include=['celery1.tasks'])

app.config_from_object('celery1.config')

if __name__ == '__main__':
  app.start()
```

子节点中celery1/config.py：

```python
from __future__ import absolute_import
from kombu import Queue,Exchange
from datetime import timedelta

CELERY_TASK_RESULT_EXPIRES=3600
CELERY_TASK_SERIALIZER='json'
CELERY_RESULT_SERIALIZER='json'
CELERY_ACCEPT_CONTENT = ['json','pickle','msgpack','yaml']

CELERY_DEFAULT_EXCHANGE = 'train'
# exchange type可以看RabbitMQ中的相关内容
CELERY_DEFAULT_EXCHANGE_TYPE = 'direct'

CELERT_QUEUES =  (
  Queue('train',exchange='train',routing_key='train'),
)
```

子节点celery1/tasks.py：(这个是要真正执行的task，每个节点可以不同)

```python
from __future__ import absolute_import
from celery1.celery import app


import time
from celery import task


@task
def do_train(x, y):
    """
    训练
    :param data:
    :return:
    """
    time.sleep(3)
    return dict(data=str(x+y),msg="train")
```

# 项目启动

启动子节点：其中celery1是项目，-Q train表示从train这个queue中接收任务

```python
celery -A celery1 worker -l info -Q train
```

启动主节点：

```python
python manage.py runserver
```



使用Postman请求对应的view：

```python
请求url：http://127.0.0.1:8000/api/v1/celery1/test/
返回的结果是:
{
    "msg": "OK",
    "code": 10000,
    "data": {
        "data": "6",
        "msg": "train"
    }
}
```



# 可能遇到的问题

1）celery队列报错: AttributeError: ‘str’ object has no attribute ‘items’
解决：将redis库从3.0回退到了2.10，pip install redis==2.10
解决方法参考链接：https://stackoverflow.com/questions/53322425/celery-critical-mainprocess-unrecoverable-error-attributeerrorfloat-object

