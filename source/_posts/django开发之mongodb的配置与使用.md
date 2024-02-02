---
title: django开发之mongodb的配置与使用
date: 2020-06-06 10:20:28
tags: django
categories: Python
---

今天整理了一下在django项目中如何使用mongodb, 环境如下：
ubuntu18.04, django2.0.5, drf3.9, mongoengine0.16

<!--more-->

# 配置mongodb

在settings.py中配置mongodb和mysql,配置如下(可以同时使用mysql和mongodb)：这里使用的第三方库是 mongoengine，需要使用 pip 进行安装。

```txt
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',   # 数据库引擎
        'NAME': 'django_test2',                  # 你要存储数据的库名，事先要创建之
        'USER': 'root',                         # 数据库用户名
        'PASSWORD': 'wyzane',                     # 密码
        'HOST': 'localhost',                    # 主机
        'PORT': '3306',                         # 数据库使用的端口
    },
    'mongotest': {
        'ENGINE': None,
    }
}

import mongoengine
# 连接mongodb中数据库名称为mongotest5的数据库
conn = mongoengine.connect("mongotest")
```



# mongodb使用

## 添加数据

### 插入 json 类型的数据

代码如下：

```python
# models.py:
import mongoengine
class StudentModel(mongoengine.Document):
    name = mongoengine.StringField(max_length=32)
    age = mongoengine.IntField()
    password = mongoengine.StringField(max_length=32)

# views.py:
from rest_framework.views import APIView
class FirstMongoView(APIView):
    def post(self, request):
        name = request.data["name"]
        age = request.data["age"]
        password = request.data["password"]
        StudentModel.objects.create(name=name, age=age, password=password)
        return Response(dict(msg="OK", code=10000))
```

请求数据格式如下：

```txt
{
    "name": "nihao",
    "age": 18,
    "password": "123456"
}
```



### 插入含有list的json数据

代码如下：

```python
# models.py:
import mongoengine
class Student2Model(mongoengine.Document):
    name = mongoengine.StringField(max_length=32)
    # 用于存储list类型的数据
    score = mongoengine.ListField()

# views.py:
from rest_framework.views import APIView
class FirstMongo2View(APIView):
    def post(self, request):
        name = request.data["name"]
        score = request.data["score"]
        Student2Model.objects.create(name=name, score=score)
        return Response(dict(msg="OK", code=10000))
```

请求数据格式如下：

```txt
 {
     "name": "test",
     "score": [12, 13]
 }
```



### 插入含有list和dict的json数据

代码如下：

```python
# models.py:
import mongoengine
class Student3Model(mongoengine.Document):
    name = mongoengine.StringField(max_length=32)
    # DictField用于存储字典类型的数据
    score = mongoengine.DictField()
# views.py:
from rest_framework.views import APIView
class FirstMongo3View(APIView):
    def post(self, request):
        name = request.data["name"]
        score = request.data["score"]
        Student3Model.objects.create(name=name, score=score)
        return Response(dict(msg="OK", code=10000))
```

请求数据格式为：

```txt
{
    "name": "test",
    "score": {"xiaoming": 12, "xiaoli": 13}
}
或者:
{
    "name": "test",
    "score": {"xiaoming": 12, "xiaoli": {"xiaozhao": 14}}
}
或者:
{
"name": "test",
"score": {"xiaoming": 12, "xiaoli": {"xiaozhao": {"xiaoliu": 12, "xiaojian": 18}}}
}
或者:
{
"name": "test",
"score": {"xiaoming": 12, "xiaoli": {"xiaozhao": {"xiaoliu": 12, "xiaojian": [12,13,14]}}}
}
```



## 查询数据

### 查询并序列化复杂json数据

代码为：

```python
# serializers.py:
class StudentSerializer(serializers.Serializer):
    name = serializers.CharField()
    score = serializers.DictField()  # 序列化复杂的json数据
    # DictField与EmbeddedDocumentField类似，但是比EmbeddedDocumentField更灵活
# views.py:
class FirstMongo4View(APIView):
    def get(self, request):
        student_info = Student3Model.objects.all()
        # 增加过滤条件
        # student_info = Student3Model.objects.filter(name="test1")
        ser = StudentSerializer(instance=student_info, many=True)
        return Response(dict(msg="OK", code="10000", data=ser.data))
```



### 序列化mongodb中含有嵌套关系的两个document

代码为：

```python
# models.py:
class AuthorModel(mongoengine.EmbeddedDocument):
    author_name = mongoengine.StringField(max_length=32)
    age = mongoengine.IntField()


class BookModel(mongoengine.Document):
    book_name = mongoengine.StringField(max_length=64)
    publish = mongoengine.DateTimeField(default=datetime.datetime.utcnow())
    words = mongoengine.IntField()
    author = mongoengine.EmbeddedDocumentField(AuthorModel)

# serializers.py: 序列化时注意与rest_framework的序列化中DictField()的区别
from rest_framework_mongoengine import serializers as s1
class AuthorSerializer(s1.DocumentSerializer):  
    # DocumentSerializer继承自drf中的ModelSerializer，用于代替ModelSerializer序列化mongodb中的document.
    # 具体可以到官网上查看
    class Meta:
        model = AuthorModel
        fields = ('author_name', 'age')


class BookSerializer(s1.DocumentSerializer):
    author = AuthorSerializer()

    class Meta:
        model = BookModel
        fields = ('book_name', 'publish', 'words', 'author')

# AuthorSerializer还可以这样写:
class AuthorSerializer(s1.EmbeddedDocumentSerializer):
    # EmbeddedDocumentSerializer继承了DocumentSerializer
    class Meta:
        model = AuthorModel
        fields = ('author_name', 'age')

# views.py:
class BookView(APIView):
    def get(self, request):
        """查询数据
        :param request:
        :return:
        """
        books = BookModel.objects.all()
        ser = BookSerializer(instance=books, many=True)
        return Response(dict(msg="OK", code="10000", data=ser.data))

```

需要注意的是：序列化mongodb中相关联的两个表时，如果序列化器继承自rest_framework中的Serializer和ModelSerializer，会抛出如下异常：

```txt
Django serialization to JSON error: 'MetaDict' object has no attribute 'concrete_model'
```

此时，序列化器需要继承自rest_framework_mongoengine的类，具体可以查看官网：
http://umutbozkurt.github.io/django-rest-framework-mongoengine/



mongodb 在 django 中的使用就介绍到这里。