---
title: zeroc ice在python中的使用
date: 2020-07-11 09:54:26
tags: python
categories: Python

---

zeroc ice 指的是 zeroc 公司开发的一款网络通讯引擎 ice。ice 是一个面向对象的 RPC 框架，可以搭建分布式应用。最主要的一点，它是跨语言的，不管你使用 Python、Java 还是 C++、Ruby、C# 等开发语言，它都支持。还有一点，ice 提供了其他功能，包括：IceStorm（一种订阅服务，类似于消息队列）、IceGrid 等。

<!--more-->

下面从安装、使用等几个方面介绍下 ice 在 python 中的使用。

# 安装

安装环境是 python3.6.9，ubuntu18.04。

## 安装方式一

安装步骤如下：

```txt
1. sudo apt-get install openssl libssl-dev libbz2-dev
    第一步安装不成功时，可以使用 aptitude 来安装需要的包：
	sudo apt-get install aptitude 
	sudo aptitude install xxxxx
2. 安装 ice for python: pip install zeroc-ice
3. 安装 ice for linux
	1）将源码克隆到本地: git clone https://github.com/zeroc-ice/ice.git
		若网速太慢，可以先在码云上创建项目并将ice.git克隆过去，再从码云上克隆
	2）克隆完成后，切换分支到3.7: git checkout 3.7（安装3.7版本）
	3）编译：make supported-languages='cpp python' （只编译c++和python，还可以编译其他语言）
	4）安装：make install supported-languages='cpp python' prefix=/opt/zeroc-ice 
	5）配置环境变量：在 .zshrc 中添加配置: PATH=$PATH:/opt/zeroc-ice/bin
	6）执行命令: icegridnode -v，会输出响应版本
4. 安装完成后，可以从 github 上克隆 ice-demos 来学习。
```



## 安装方式二

或者，可以按照 zeroc 官方文档进行安装，官方文档如下：

```txt
https://zeroc.com/downloads/ice/3.7/python
```



# 基础介绍

首先简单介绍下 ice 中相关概念和一些基础知识。

## slice语言

Slice (Specification Language for Ice) 是实现 Ice 协议的开发语言，它可以把 Ice 接口的实现翻译成不同的开发语言版本。对于不同的应用开发语言，有不同的映射规则。

Slice 提供一个基本的抽象机制用于分离接口和他们的实现。Slice 用于描述接口，建立客户端和服务端之间的关系。Slice的接口描述与具体实现语言无关，即它可以让你定义 client 和 server 之间的交互而不用关心具体的开发语言，例如 C++，Java、Python等。通过编译，可以把 Slice对接口的描述转换成不同的开发语言。



## client 与 server

在 ice 中，client 和 server 不是指一个应用系统的特定部分，它们是在一个持续请求中扮演不同角色，具有不同的功能。

client 会主动向 server 发送服务请求，而 server 端只能被动的接收请求，并针对请求，为 client 端提供不同的服务。通常，client 会主动向 server 端发起请求，也可以接收 server 端的回调通知，这样看来，client 既有 client 端属性也有 server 端属性。

client 和 server 的结构如下：

![Ice_Client_and_Server_Structure](Ice_Client_and_Server_Structure.gif)

上面的结构中，有以下几部分。

###  Ice Core

ice core 包含了 client 和 server 的运行时，用于支持远程通信。这部分包含了网络通信、线程、字节序及其他网络相关的代码实现，并与应用层代码分离开来。ice core 提供了一系列库供 client 和 server 端调用。

### Ice API

ice 中通过 ice api 来访问 ice core，使用 ice api 来做一些基础工作，例如初始化和资源回收。ice api 对于 client 和 server 端来说没有什么区别。

### Proxy Code

proxy code 部分是根据 slice 文件中定义的内容生成的，它规定了对象和数据的类型。proxy code 主要有两个功能：

1. 为 client 端提供向下调用的接口，在 proxy API 调用函数最终会向 server 端发送一个 RPC 消息，然后在服务端执行响应的目标函数
2. 提供组包和解包代码，组包是序列化复杂数据结构的过程，例如为了在网络上传送，序列化一个序列或者字典。组包会将数据转换成一种标准的数据传输格式，并且不受字节序的影响。解包就是相反的过程。

### Skeleton Code

skeleton code 与 proxy code 功能相似，只是它在 server 端发挥作用，proxy code 是在 client 端的。它提供了向上调用的接口，并且允许 ice 运行时将控制线程转换为应用代码。skeleton 中也包含组包和解包的功能，以便 server 端能接收 client 端传入的参数，并且向 client 端返回结果或者异常。

### Object Adapter

object adapter 是 ice api 的一部分，它只在 server 端使用，它由如下几个功能：

1. 将客户端请求与编程语言中的特定方法映射，即 object adapter 会找到内存中具有特定标识的对象
2. object adapter 与传输协议有关，如果一个 object adapter 对应多个传输协议，server 就能提供多种服务
3. object adapter 可以生成 proxy（proxy 会被发送到 client 端）。object adapter 中记录了每个对象的类型、名称、传输信息。当 server 端想要创建 proxy 时，object adapter 能创建 proxy 而不需要知道具体细节。



## Ice Objects

简单来说，ice object 对象中定义了一些接口，通过调用这些接口，client 能向服务器发送请求。我的理解是（以 Python 为例），ice object 中定义了一些方法，这些方法实现了请求创建、数据传递、数据接收等功能；我们在 slice 文件中定义的接口，经过转换后是 ice object 的子类；server 端需要继承 ice object 的子类来实现具体的功能。

## Proxies

通过 proxy，client 能与 ice object 关联起来，proxy 就类似于 ice object 的一个使者。当 client 想调用 ice object 中的某个功能时（其实就是某个方法），需要通过 proxy ，这时会执行以下流程：

```txt
1. 定位 ice object
2. 激活 ice object
3. 激活 server 端的 ice object
4. 传递入参到 ice object 中
5. 等待操作完成
6. 把出参和结果返回给 client 端
```

proxy 信息可以用一个字符串表示：

```txt
SimplePrinter:default -p 10000
```

这种字符串形式的 proxy 更加易于理解和存储。

proxy 还分为 Direct Proxy 和 Indirect Proxy。Direct Proxy 通常包括 proxy 对象名称和服务器地址，包括以下两类：

```txt
1. a protocol identifier (such TCP/IP or UDP)
2. a protocol-specific address (such as a host name and port number)
```

Indirect Proxy 有两种形式，一种是仅仅提供对象名称（如：SimplePrinter），另一种是提供对象名称和对象适配器名称（如：SimplePrinter@PrinterAdapter）。

## Ice Protocol

ice能提供 RPC 协议，这些 RPC 协议可以使用多种底层协议，最常用的是TCP和UDP，但是ice也支持 Websocket, Bluetooth 和 Apple's iAP。Ice还可以使用 SSL 协议对传输数据加密。

Ice 协议中定义了以下内容：

```txt
1. 定义了一些消息类型，例如请求\应答的消息类型
2. 定义了状态机，确定客户端和服务端如何交换数据
3. 定义编码规则，确定怎样表示数据类型
4. 定义了消息类型的头部，包括消息类型、消息大小、使用的协议等。
```

Ice也支持数据压缩，当客户端与服务端传输的数据量非常大时，这很有用。
Ice 协议非常适合创建高效的事件分发机制，因为它允许我们分发消息而不用关心消息内部的实现。
Ice 协议也支持双向的操作，即：如果服务端传输数据到客户端提供的回调对象上，回调过程可以通过客户端最初创建的连接来完成。

## Ice Service

对于开发分布式应用程序，Ice Core提供了一个复杂的 client-server 平台，然而，实际开发中不仅仅需要远程处理能力，也需要按需提供服务、向客户端分发代理、分发异步任务、向应用程序分发补丁等等。Ice 提供了以下服务来实现上面的功能：

```txt
1. IceGrid
2. IceStorm
3. IcePatch2
4. Glacier2
5. IceBridge
```



## 总结

从上面的介绍中，我们可以发现，开发 ice 的 client 和 server 时，我们需要以下几个部分：

```txt
1. slice 文件
2. ice object
3. proxy code
4. object adapter
```

上面这些都会在下面的代码中体现出来，可以回想一下它们的功能分别是什么。

# 简单使用

安装完成后，首先看第一个使用的例子。

## 编写 Slice File

开发时，首先要编写 slice 文件，slice 文件是与开发语言无关的。编写完成后，需要使用编译器将其转换成不同的开发语言，例如转换成 Python 时，就需要使用 slice2py。slice 文件里面的内容其实就是一些接口，server 端开发时需要编写 Python 类来实现这些接口。

Printer.ice：

```txt
module Demo
{
    interface Printer
    {
        void printString(string s);

        int addMun(int num1, int num2);
    }
};
```

其中， Demo 是模块名称，Printer 是接口名称，接口里面是方法的定义。

编写完成后，执行：

```txt
slice2py Printer.ice
```

执行完成后，会生成 Printer_ice.py 文件和 Demo 文件夹。Demo文件夹下的内容如下：

```txt
├── __init__.py
└── __pycache__
    └── __init__.cpython-36.pyc
    


Demo/__init__.py 内容如下：
# Generated by slice2py - DO NOT EDIT!
#

import Ice
Ice.updateModule("Demo")

# Modules:
import Printer_ice

# Submodules:
```

Printer_ice.py 文件中的内容如下：

```python
# -*- coding: utf-8 -*-
#
# Copyright (c) ZeroC, Inc. All rights reserved.
#
#
# Ice version 3.7.4
#
# <auto-generated>
#
# Generated from file `Printer.ice'
#
# Warning: do not edit this file.
#
# </auto-generated>
#

from sys import version_info as _version_info_
import Ice, IcePy

# Start of module Demo
_M_Demo = Ice.openModule('Demo')
__name__ = 'Demo'

_M_Demo._t_Printer = IcePy.defineValue('::Demo::Printer', Ice.Value, -1, (), False, True, None, ())

if 'PrinterPrx' not in _M_Demo.__dict__:
    _M_Demo.PrinterPrx = Ice.createTempClass()
    class PrinterPrx(Ice.ObjectPrx):

        def printString(self, s, context=None):
            return _M_Demo.Printer._op_printString.invoke(self, ((s, ), context))

        def printStringAsync(self, s, context=None):
            return _M_Demo.Printer._op_printString.invokeAsync(self, ((s, ), context))

        def begin_printString(self, s, _response=None, _ex=None, _sent=None, context=None):
            return _M_Demo.Printer._op_printString.begin(self, ((s, ), _response, _ex, _sent, context))

        def end_printString(self, _r):
            return _M_Demo.Printer._op_printString.end(self, _r)

        def addMun(self, num1, num2, context=None):
            return _M_Demo.Printer._op_addMun.invoke(self, ((num1, num2), context))

        def addMunAsync(self, num1, num2, context=None):
            return _M_Demo.Printer._op_addMun.invokeAsync(self, ((num1, num2), context))

        def begin_addMun(self, num1, num2, _response=None, _ex=None, _sent=None, context=None):
            return _M_Demo.Printer._op_addMun.begin(self, ((num1, num2), _response, _ex, _sent, context))

        def end_addMun(self, _r):
            return _M_Demo.Printer._op_addMun.end(self, _r)

        @staticmethod
        def checkedCast(proxy, facetOrContext=None, context=None):
            return _M_Demo.PrinterPrx.ice_checkedCast(proxy, '::Demo::Printer', facetOrContext, context)

        @staticmethod
        def uncheckedCast(proxy, facet=None):
            return _M_Demo.PrinterPrx.ice_uncheckedCast(proxy, facet)

        @staticmethod
        def ice_staticId():
            return '::Demo::Printer'
    _M_Demo._t_PrinterPrx = IcePy.defineProxy('::Demo::Printer', PrinterPrx)

    _M_Demo.PrinterPrx = PrinterPrx
    del PrinterPrx

    _M_Demo.Printer = Ice.createTempClass()
    class Printer(Ice.Object):

        def ice_ids(self, current=None):
            return ('::Demo::Printer', '::Ice::Object')

        def ice_id(self, current=None):
            return '::Demo::Printer'

        @staticmethod
        def ice_staticId():
            return '::Demo::Printer'

        def printString(self, s, current=None):
            raise NotImplementedError("servant method 'printString' not implemented")

        def addMun(self, num1, num2, current=None):
            raise NotImplementedError("servant method 'addMun' not implemented")

        def __str__(self):
            return IcePy.stringify(self, _M_Demo._t_PrinterDisp)

        __repr__ = __str__

    _M_Demo._t_PrinterDisp = IcePy.defineClass('::Demo::Printer', Printer, (), None, ())
    Printer._ice_type = _M_Demo._t_PrinterDisp

    Printer._op_printString = IcePy.Operation('printString', Ice.OperationMode.Normal, Ice.OperationMode.Normal, False, None, (), (((), IcePy._t_string, False, 0),), (), None, ())
    Printer._op_addMun = IcePy.Operation('addMun', Ice.OperationMode.Normal, Ice.OperationMode.Normal, False, None, (), (((), IcePy._t_int, False, 0), ((), IcePy._t_int, False, 0)), (), ((), IcePy._t_int, False, 0), ())

    _M_Demo.Printer = Printer
    del Printer

# End of module Demo
```

从上面的内容中可以看出，Printer_ice.py 中主要有两个 Python 类 PrinterPrx 和 Printer。



## 编写 Server 端

```python
import sys, Ice
import Demo
 
class PrinterI(Demo.Printer):
    def printString(self, s, current=None):
        print(s)

    def addMun(self, n1, n2, current=None):
        ret = n1 + n2
        print(ret)
        return ret
 
with Ice.initialize(sys.argv) as communicator:
    adapter = communicator.createObjectAdapterWithEndpoints("SimplePrinterAdapter", "default -p 10000")
    object = PrinterI()
    adapter.add(object, communicator.stringToIdentity("SimplePrinter"))
    adapter.activate()
    communicator.waitForShutdown()
```

server 端的代码中主要有两部分：一个 Python 类 PrinterI，继承了 Demo.Printer 并实现了其中的方法；with 代码块。

with 代码块中的含义如下：

```txt
1. Ice.initialize()初始化 Ice 运行环境，返回一个Ice.Communicator对象，它是 Ice 运行时的主要对象
2. communicator.createObjectAdapterWithEndpoints()创建一个 Object Adapter 对象，
   "SimplePrinterAdapter"表示 Adapter 对象名称，"default -p 10000" 表示使用 TCP/IP协议并监听10000端口
3. object = PrinterI() 表示实例化 Printer 接口的子类
4. 调用 Adapter 对象的 add 方法，将实例化的对象绑定到 Adapter 中。add() 方法的第一个参数就是 PrinterI 类的对    象，第二个参数是给 PrinterI 对象指定一个名称，如果有多个实例化的对象时，每个对象的名称都不能相同
5. adapter.activate() 表示启用 Adapter 
6. 最后，调用 communicator.waitForShutdown() 会阻塞直到服务关闭
```



##  编写 Client 端

```python
import sys, Ice
import Demo
 
with Ice.initialize(sys.argv) as communicator:
    base = communicator.stringToProxy("SimplePrinter:default -p 10000")
    printer = Demo.PrinterPrx.checkedCast(base)
    if not printer:
        raise RuntimeError("Invalid proxy")
 
    printer.printString("Hello World!")
    ret = printer.addMun(1, 2)
    print("client:", ret)
```

client 中只有一个 with 代码块，其含义如下：

```txt
1. Ice.initialize()：初始化 Ice 运行环境
2. communicator.stringToProxy()：获得远端 printer 的一个代理对象，参数 "SimplePrinter:default -p 10000"    表示在服务端指定的 printer 对象名称和监听端口
3. stringToProxy()方法返回的是一个Ice.ObjectPrx类型，它是其他所有接口的父类。但是，实际上我们是需要一个          Demo.Printer 类型，所以需要类型向下的转换。使用 Demo.PrinterPrx.checkedCast(base) 可以向服务器发送消息，    确认代理是否是 Demo.Printer类型，如果是，就返回一个 Demo.PrinterPrx 类型的代理，否则返回None
4. printer.printString("Hello World!") 调用方法。
```



编写完成后，执行 Python 文件就行了：

```txt
python server.py
python client.py
```

