---
title: django开发中signals信号的使用
date: 2020-05-05 10:30:56
tags: django
categories: Python
---

django 中提供了信号机制，它类似于数据库中的触发器，也类似于编程中的回调函数。

<!--more-->

在 signals 中，我们可以自定义需要执行的函数，当某个动作（如model对象被保存）发生时，可以触发自定义函数的执行。

django中有内置的信号，我们也可以自定义信号。
django中有以下常用内置信号：

```txt
django.db.models.signals.pre_save 在某个Model保存之前调用
django.db.models.signals.post_save 在某个Model保存之后调用
django.db.models.signals.pre_delete 在某个Model删除之前调用
django.db.models.signals.post_delete 在某个Model删除之后调用
django.core.signals.request_started 在建立Http请求时发送
django.core.signals.request_finished 在关闭Http请求时发送
```

其他内置信号可以查看官方文档：
https://docs.djangoproject.com/en/2.1/topics/signals/

下面介绍下内置信号和自定义信号的使用。



# 内置信号

下面举一个post_save的例子，test01 是 app 名称。

首先在 test01/signals/handler.py 中定义signal函数：

```python
from django.db.models.signals import post_save
from django.dispatch import receiver
from ..models import StudentModel


@receiver(post_save)
def save_signal(sender, **kwargs):
    """
    所有Model被保存后，都会触发该函数的执行
    :param sender:
    :param kwargs:
    :return:
    """
    print("save_signal sender:", sender)


# sender用于指定哪个model被保存后触发该函数的执行
@receiver(post_save, sender=StudentModel)
def student_signal(sender, **kwargs):
    """
    StudentModel被保存后，触发该函数的执行
    :param sender:
    :param kwargs:
    :return:
    """
    print("student_signal sender:", sender)
```

在 test01/views.py 中定义视图：

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from .models import UserModel, StudentModel


class UserView(APIView):

    def post(self, request):
        print("-- user view --")
        UserModel.objects.create(username="wyzane", password="123456")
        return Response("OK")


class StudentView(APIView):

    def post(self, request):
        print("-- student view --")
        StudentModel.objects.create(name="wyzane", age=16)
        return Response("OK")
```

配置 test01/apps.py：

```python
from django.apps import AppConfig


class Test01AppConfig(AppConfig):
    name = "test01"

    print("-- Test01AppConfig --")

    def ready(self):
        from .signals import handler
```

配置 **test01/**init.py：

```python
default_app_config = 'test01.apps.Test01AppConfig'
```

此时在postman中分别请求下面两个url：
http://127.0.0.1:8000/test01/user/
http://127.0.0.1:8000/test01/student/
打印的结果分别是：

```txt
-- user view --
save_signal sender: &lt;class 'test01.models.UserModel';

-- student view --
save_signal sender: &lt;class 'test01.models.StudentModel';
student_signal sender: &lt;class 'test01.models.StudentModel';
```

通过上面的结果，我们也可以看出信号中使用和不使用sender参数区别。



# 自定义信号

下面介绍下自定义信号的使用。



定义信号 test01/signals/signals.py：

```python
import django.dispatch

signal01 = django.dispatch.Signal(providing_args=["p1", "p2"])
```

在视图中触发自定义信号 test01/views.py：

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from .signals.signals import signal01


class SignalView(APIView):

    def post(self, request):
        # 触发自定义信号
        signal01.send(sender="signal_view", p1="wyzane", p2=18)
        return Response("OK")
```

编写处理函数，即收到信号后要做哪些操作 test01/signals/handler.py：

```python
from django.dispatch import receiver
from .signals import signal01


@receiver(signal01)
def my_signal(sender, **kwargs):
    """
    信号触发后执行的操作
    :param sender: 
    :param kwargs: 
    :return: 
    """
    print("my signal:", sender, kwargs)
```

下一步就是配置，这里的配置与内置信号中的配置相同，就不介绍了。

配置完成后，请求下面的url
http://127.0.0.1:8000/test01/signal/
看到打印的结果如下：

```txt
my signal: signal_view {‘signal’: <django.dispatch.dispatcher.Signal object at 0x000001F57FCA1160>, ‘p1’: ‘wyzane’, ‘p2’: 18}
```



上面的配置中使用了 AppConfig，AppConfig 是用来设置 django中 每个 app 属性的，上面的配置中定义了一个Test01AppConfig 类来继承 AppConfig，并在 init.py 中使用 default_app_config 来加载这个类。
重写在ready方法中的代码会在django运行的时候执行。
感兴趣的小伙伴可以看下官方文档的介绍：
https://docs.djangoproject.com/en/2.1/ref/applications/。

signals信号的知识就介绍到这里。