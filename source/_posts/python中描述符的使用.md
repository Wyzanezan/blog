---
title: python中描述符的使用
date: 2019-11-28 18:39:46
tags: python
categories: Python
---

描述符的官方文档如下，有问题可以查看官方文档。官方文档：https://docs.python.org/3.7/howto/descriptor.html?highlight=descriptor

<!--more-->

## 描述符

### 描述符介绍

```python
	描述符（descriptor）实际是一种类，但是这个类实现了__set__ __get__ __delete__ 这3个方法中一个或多个。 其中__get__是必须的，其他2个可选。
	如果实现中实现了__set__ __get__那么称之为数据描述符
	如果只实现了 __get__ 那么称之为非数据描述符
    
    __get__():调用一个属性时,触发
    __set__():为一个属性赋值时,触发
    __delete__():采用del删除属性时,触发
    
    描述符作用：实现对属性的代理；在对属性获取，设置，删除时，可以进行额外的操作

    数据描述符与非数据描述符的区别：
    访问相同属性时，数据描述符优先于instance dictionary;instance dictionary优先于非数据描述符
    
    当然，我们也可以定义只读的数据描述符：
    在类中同时定义__get__和__set__方法，并且在调用__set__方法时，抛出 AttributeError 异常即可。
    
    属性的调用顺序：
    对象属性的调用顺序：
    	在object.__getattribute__()中，会将 obj.x 转换成 type(obj).__dict__['x'].__get__(obj, type(obj))，再按照 数据描述符 > instance variables > 非数据描述符 >  __getattr__() 的优先级顺序调用
    类属性的调用顺序：
    	type.__getattribute__()中 会将 class.x 转换成 B.__dict__['x'].__get__(None, B)，类似于下面这种调用：
        	def __getattribute__(self, key):
                v = object.__getattribute__(self, key)
                if hasattr(v, '__get__'):
                    return v.__get__(None, self)
                return v
            
    通过上面的介绍，可以总结出下面的几点：
    1. 描述符会在 __getattribute__() 中被调用
    2. 重写 __getattribute__() 方法会阻止描述符的调用
    3. 数据描述符会重写 instance dictionary
    4. 非数据描述符可能会被 instance dictionary 重写
```



### 描述符应用场景

```python
1. 访问属性时进行验证
```



### property的使用

```python
# property是一个描述符类，用于访问属性

# 使用方式一：
class Test:

    def __init__(self, name, age):
        self._name = name
        self._age = age

    @property
    def name(self):
        print("property get")
        return self._name

    @name.setter
    def name(self, name):
        print("property setter")
        self._name = name
        
if __name__ == '__main__':
    t = Test("小明", 12)
    print(t.name)
    t.name = "小刘"
    print(t.name)

"""
_get_name
小明
===============
_set_name
_get_name
小刘
"""



# 使用方式二：
调用 property()方法 是创建数据描述符的一种简洁的方法
class Test:

    def __init__(self, name, age):
        self._name = name
        self._age = age

    def _set_name(self, name):
        print("_set_name")
        self._name = name

    def _get_name(self):
        print("_get_name")
        return self._name

    # 使用property描述符
    name = property(_get_name, _set_name)
    
if __name__ == '__main__':
    t = Test("小明", 12)
    print(t.name)

    print("===============")
    t.name = "小刘"
    print(t.name)

"""
_get_name
小明
===============
_set_name
_get_name
小刘
"""
```

对于 property() 如何实现的描述符协议，类似下面这段代码：

```python
class Property(object):
    "Emulate PyProperty_Type() in Objects/descrobject.c"

    def __init__(self, fget=None, fset=None, fdel=None, doc=None):
        self.fget = fget
        self.fset = fset
        self.fdel = fdel
        if doc is None and fget is not None:
            doc = fget.__doc__
        self.__doc__ = doc

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        if self.fget is None:
            raise AttributeError("unreadable attribute")
        return self.fget(obj)

    def __set__(self, obj, value):
        if self.fset is None:
            raise AttributeError("can't set attribute")
        self.fset(obj, value)

    def __delete__(self, obj):
        if self.fdel is None:
            raise AttributeError("can't delete attribute")
        self.fdel(obj)

    def getter(self, fget):
        return type(self)(fget, self.fset, self.fdel, self.__doc__)

    def setter(self, fset):
        return type(self)(self.fget, fset, self.fdel, self.__doc__)

    def deleter(self, fdel):
        return type(self)(self.fget, self.fset, fdel, self.__doc__)
```



### 描述符的使用

```python
class SimpleDescriptor:

    def __init__(self, name):
        self.name = name

    def __get__(self, instance, owner):
        """
        Args:
            instance: 类对象的实例
            owner: 类对象
        """
        if not instance:
            return self
        return instance.__dict__[self.name]

    def __set__(self, instance, value):
        """
        Args:
            instance: 类对象实例
            value: 属性值
        """
        name = self.name
        if name == "age" and value < 18:
            raise ValueError("age less than 18")
        instance.__dict__[self.name] = value

    def __delete__(self, instance):
        del instance.__dict__[self.name]


class Test:

    name = SimpleDescriptor("name")
    age = SimpleDescriptor("age")

    def __init__(self, name, age):
        self._name = name
        self._age = age


if __name__ == '__main__':
    t = Test("小明", 12)
    # print(t.name)

    print("=============")
    t.name = "小刘"
    t.age = 16
    print(t.name)
    print(t.age)
```



### 描述符的应用

```python
# 1. 应用在 staticmethod 中 ，staticmethod是一个非数据描述符。下面两段代码片段功能相同
# 代码1：
class E:

    @staticmethod
    def f(x):
        print(x)

if __name__ == '__main__':
    E.f(1)
    E().f(2)
    
  
# 代码2：
class E:

    def f(x):
        print(x)

    f = staticmethod(f)


if __name__ == '__main__':
    E.f(1)
    E().f(2)
    
    
# 同样的， classmethod 也是一个非数据描述符，下面两段代码功能相同
# 代码1：
class E:

    @classmethod
    def f(cls, x):
        print(x)


if __name__ == '__main__':
    E.f(1)
    E().f(2)
    
    
# 代码2：
class E:

    def f(cls, x):
        print(x)

    f = classmethod(f)


if __name__ == '__main__':
    E.f(1)
    E().f(2)
```



以上就是python中描述符的介绍！