---
layout: 'n'
title: django开发之重写objects模型管理器
date: 2020-05-24 15:31:06
tags: django
categories: Python
---

在django中，与数据库相关的操作都封装在了objects这个对象中。objects是models.Manager的一个实例，重写Manager中的filter、all等方法可以提高开发效率，是代码看起来更加简洁。

<!--more-->

下面是一个重写filter和all方法的例子：
models.py：

```python
from django.db import models


class BaseModel(models.Model):
    """
    所有模型的共有字段
    """
    create_time = models.DateTimeField(auto_now_add=True)
    is_delete = models.IntegerField(default=1)

    class Meta:
        abstract = True


class BaseManager(models.Manager):
    def all(self):
        """
        重写all方法
        :return:
        """
        students = super(BaseManager, self).all()
        student = students.filter(is_delete=1)
        return student

    def filter(self, *args, **kwargs):
        """
        重写filter方法
        :param args:
        :param kwargs:
        :return:
        """
        if not kwargs.get("is_delete", 0):
            kwargs["is_delete"] = 1
        return super(BaseManager, self).filter(*args, **kwargs)


class StudentsModel(BaseModel):
    name = models.CharField(max_length=32)
    age = models.IntegerField(default=0)
    gender = models.IntegerField(default=1)
    objects = BaseManager()

    class Meta:
        db_table = "test01_students"
        verbose_name = "学生表"

```

views.py：

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from .models import StudentsModel


class StudentView(APIView):
    def get(self, request):
        """
        获取学生信息
        :param request:
        :return:
        """
        students = StudentsModel.objects.filter()
        # students = StudentsModel.objects.all()
        print("students:", students.query)
        for student in students:
            print("student info:", (student.name, student.age))
        return Response(dict(msg="OK", code=10000))

    def post(self, request):
        """
        新增学生信息
        :param request:
        :return:
        """
        name = request.data.get("name")
        age = request.data.get("age")
        gender = request.data.get("gender")
        StudentsModel.objects.create(**{"name": name, "age": age, "gender": gender})
        return Response(dict(msg="OK", code=10000))

```

当访问视图的get方法时，students.query打印出的结果如下：

```python
SELECT `test01_students`.`id`, `test01_students`.`create_time`, `test01_students`.`is_delete`, `test01_students`.`name`, `test01_students`.`age`, `test01_students`.`gender` FROM `test01_students` WHERE `test01_students`.`is_delete` = 1
```

可以看到，我们省去了is_delete=1。调用all()方法时也是一样的。

上面的models.py中，模型类StudentModel继承了BaseModel,省去了我们写公共字段的麻烦；类中的属性objects是BaseManager的实例，StudentsModel.objects.filter()就会调用BaseManager中重写filter方法。