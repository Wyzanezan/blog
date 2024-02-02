---
title: Django3.0新功能介绍
date: 2019-12-08 12:03:21
tags: django
categories: Python
---

最近，Django框架发布了3.0版本，同时为我们带来了一些新功能，其中最引人注目的一个新功能就是对异步的原生支持。

<!--more-->

Django3.0版本发布说明：[https://docs.djangoproject.com/en/3.0/releases/3.0/](https://docs.djangoproject.com/en/3.0/releases/3.0/)



Django3.0仅支持Python3.6及以上的版本，不再支持Python3.5及以下的Python版本。



下面，介绍下Django3.0中的新功能。



### 支持MariaDB

Django官方开始支持MariaDB10.1（MariaDB数据库管理系统是MySQL的一个分支，是MySQL的开源版本）及其更高版本。



### 支持ASGI协议

通常我们开发Django应用时，使用WSGI协议，它是Python的Web服务器和Web应用程序或框架之间的一种简单而通用的接口。而ASGI协议是对WSGI的扩展，它在WSGI的基础上定义了实现异步通信的方式，遵循ASGI协议的Web程序可以实现异步通信。

以前我们在Django中实现异步通信,需要借助channels等第三方包。现在，Django3.0可以运行为一个ASGI应用程序，从而实现对异步的支持；即应用程序运行在ASGI协议下时，可以使用异步特性，运行在其他协议下则不能使用这个特性。

支持ASGI协议可能也会有负面影响，当你的代码中有异步非安全的的操作时（例如ORM操作），Django会阻塞代码的执行。当代码中抛出 **SynchronousOnlyOperation** 异常时，你应该检查异步代码并移除其中的数据库操作部分。



### 支持PostgreSQL建立排他约束

ExclusionConstraint类可以在pg数据库上添加排他约束，将约束添加到Meta.constraints中，像下面这样：

```python
from django.db import models

class Customer(models.Model):
    age = models.IntegerField()

    class Meta:
        constraints = [
            models.CheckConstraint(check=models.Q(age__gte=18), name='age_gte_18'),
        ]
```



### field choice支持枚举类型

 **TextChoices**， **IntegerChoices**, 和 **Choices**等枚举类型可以用来定义**Field.choise**。下面是具体使用方式：

```python
# TextChoices
class Student(models.Model):

    class YearInSchool(models.TextChoices):
        FRESHMAN = 'FR', _('Freshman')
        SOPHOMORE = 'SO', _('Sophomore')
        JUNIOR = 'JR', _('Junior')
        SENIOR = 'SR', _('Senior')
        GRADUATE = 'GR', _('Graduate')

    year_in_school = models.CharField(
        max_length=2,
        choices=YearInSchool.choices,
        default=YearInSchool.FRESHMAN,
    )
```

```python
# IntegerChoices
class Card(models.Model):

    class Suit(models.IntegerChoices):
        DIAMOND = 1
        SPADE = 2
        HEART = 3
        CLUB = 4

    suit = models.IntegerField(choices=Suit.choices)
```



Django3.0中主要新增主要特性就是上面这些，主要特性具体信息、新增次要特性及向后不兼容的功能（不再支持pg9.4,和oracle12.1等）可以点击上面的链接查看。

### 