---
title: django开发中drf视图的使用
date: 2020-05-05 15:37:47
tags: django
categories: Python
---

使用django开发前后端分离项目时，django rest framework（以下简称drf）是常用的扩展库，今天主要介绍下drf中几个视图的使用。

<!--more-->

下面例子中涉及的model对象为

```python
class BooksModel(models.Model):
    name = models.CharField(max_length=32)
    author = models.CharField(max_length=32)
    star = models.IntegerField()

    class Meta:
        db_table = "books"
```

序列化类：

```python
class BookInfoSerializer(serializers.ModelSerializer):
    class Meta:
        model = models.BooksModel
        fields = '__all__'
```

# APIView

APIView继承自django的View，APIView中对request进行了封装。
在继承自APIView的视图中，你可以使用request.data和request.query_params来接收请求数据，类似于标准django视图中的request.POST和request.GET。例子如下：

```python
def get(self, request):
    # get请求中接收参数
    id = request.query_params.get("id")
    ...
    return Response(...)

def post(self, request):
    # post方法中接收参数
    name = request.data.get("name")
    author = request.data.get("author")
    star = request.data.get("star")
    ...
    return Response(...)
```

在继承自APIView的视图中，可以添加authentication_classes、permission_classes 类，分别用于身份认证和权限认证。可以使用自带的认证类，也可以自定义认证类。

# GenericAPIView

继承自APIView，主要是增加了操作序列化器和数据库查询的方法。
GenericAPIView的视图中可以指定以下属性：

```python
queryset 列表视图的查询集
serializer_class 视图使用的序列化器
pagination_class 分页控制类
filter_backends 过滤控制后端
lookup_field：查询单一数据库对象时使用的条件字段，默认为pk
```

可以使用下面的方法：

```python
get_queryset : 获取视图对应的查询集，是列表视图和详细视图获取数据的基础；默认返回的是queryset 的属性，可重写
get_serializer_class : 获取序列化器类，默认返回的是serializer_class，可重写；
get_serializer(self, args, *kwargs)：获取序列化器对象，这一步相对于APIView来说，就免去了创建序列化对象；
get_serializer_context(self)：这个是给序列化器返回的一个context属性，context属性里面有‘request’，‘format’，‘view’值可以在序列化器类中使用。
get_object(self) : 返回详情视图所需的模型类数据对象，默认使用lookup_field参数来过滤queryset。 在试图中可以调用该方法获取详情信息的模型类对象
```

例子如下：

```python
urls.py:
from . import views
from django.urls import path

urlpatterns = [
    path('book/&lt;pk&gt;', views.BookView.as_view())
]

views.py:
from . import models
from rest_framework.response import Response
from rest_framework.generics import GenericAPIView
from rest_framework import serializers


class BookView(GenericAPIView):
    # 查询集
    queryset = models.BooksModel.objects.all()
    # 指定序列化类
    serializer_class = BookInfoSerializer

    def get(self, request, pk):
        “”” 查询某个书籍的具体信息 ”””
        book = self.get_object()
        ser = self.get_serializer(book)
        return Response(ser.data)
```

# ViewSetMixin

在定义路由时，这种视图可以实现将http方法名与自定义的方法名称映射，下面可以下例子：

两个get请求：

```python
urls.py:
from .views import BookView
from django.urls import path

urlpatterns = [
    path("book/", BookView.as_view({'get': 'list'})),
    path("book/id", BookView.as_view({'get': 'retrive'})),
]

views.py:
class BookView(ViewSetMixin, APIView):
    def list(self, request, *args, **kwargs):

        ret = {"code": 10000, "data": None}
        try:
            book_objs = models.BooksModel.objects.all()
            book_ser = BookInfoSerializer(instance=book_objs, many=True)
            ret["data"] = book_ser.data
        except Exception as e:
            ret["code"] = 10001
            ret["error"] = "获取数据失败"
        return Response(ret)

    def retrive(self, request, *args, **kwargs):
        ret = {"code": 10000, "data": None}
        try:
            pk = kwargs.get("id")
            # 查询课程详细表
            book_objs = models.BooksModel.objects.filter(course_id=pk).first()
            book_ser = CourseInfoSerializer(instance=book_objs, many=False)
            ret["data"] = book_ser.data
        except Exception as e:
            print("e:", e)
            ret["code"] = 10001
            ret["error"] = "获取数据失败"
        return Response(ret)
```



get和post请求：

```python
urls.py:
from . import views
from django.urls import path

urlpatterns = [
    path('book2/', views.Book2View.as_view({"get": 'list', "post": 'create'})),
]

views.py:
from . import models
from rest_framework.response import Response
from rest_framework.views import APIView
from rest_framework.viewsets import ViewSetMixin
from rest_framework.generics import GenericAPIView
from rest_framework import serializers


class Book2View(ViewSetMixin, APIView):

    def list(self, request):
        books = models.BooksModel.objects.all()
        ser = BookInfoSerializer(instance=books, many=True)
        return Response(ser.data)

    def create(self, request):
        name = request.data.get("name")
        author = request.data.get("author")
        star = request.data.get("star")
        book_info = {
            "name": name,
            "author": author,
            "star": star
        }
        models.BooksModel.objects.create(**book_info)
        return Response(dict(msg="OK"))
```



# 视图集ViewSet

继承自APIView与ViewSetMixin，可以实现与APIView类似的功能，如身份认证、权限校验、流量管理等；也可以实现在调用as_view()时传入字典（如{‘get’:’list’}）的映射处理工作。

例子：

```python
urls.py:
urlpatterns = [
    path('book5/', views.Book5View.as_view({"get": 'list', "post": 'create'})),
]

views.py
class Book5View(ViewSet):
    """ 同ViewSetMixin + APIView """

    def list(self, request):
        books = models.BooksModel.objects.all()
        ser = BookInfoSerializer(instance=books, many=True)
        return Response(ser.data)

    def create(self, request):
        name = request.data.get("name")
        author = request.data.get("author")
        star = request.data.get("star")
        book_info = {
            "name": name,
            "author": author,
            "star": star
        }
        models.BooksModel.objects.create(**book_info)
        return Response(dict(msg="OK"))
```



# GenericViewSet

继承自GenericAPIView+ViewSetMixin，比上面讲到的ViewSet更方便，在实现了调用as_view()时传入字典（如{‘get’:’list’}）的映射处理工作的同时，还提供了GenericAPIView提供的基础方法，不用再编写ViewSet中的list()、create()等方法。
例子：

```python
urls.py:
urlpatterns = [
    path('book6/', views.Book6View.as_view({'get': 'list'})),
    path('book6/&lt;pk&gt;', views.Book6View.as_view({'get': 'retrive'})),
]

views.py
class Book6View(GenericViewSet):
    """ 同GenericAPIView + ViewSetMixin """
    # 查询集
    queryset = models.BooksModel.objects.all()
    # 指定序列化类
    serializer_class = BookInfoSerializer

    def list(self, request):
        book = self.get_queryset()
        ser = self.get_serializer(instance=book, many=True)
        return Response(ser.data)

    def retrive(self, request, pk):
        book = self.get_object()
        ser = self.get_serializer(book)
        return Response(ser.data)
```

