---
title: Python标准库之argparse
date: 2019-12-29 21:36:20
tags: python
categories: Python
---

python中用来解析命令行的工具有两个：sys.argv 和 argparse。对于简单的命令行传参，使用 sys.argv 就可以解析了，如果需要解析复杂的命令行参数，那么就可以使用 argparse。下面介绍下 argparse 的用法。

<!--more-->

第一个简单的例子：

```python
import argparse


# 实例化参数解析器
parser = argparse.ArgumentParser(description="a argument parser")

# 添加参数
parser.add_argument('--host',
                    action="store",
                    default='127.0.0.1')
parser.add_argument('--port',
                    action="store", type=int,
                    default=8000)

print(parser.parse_args())
"""
在命令行执行：
python 01_test.py --host 192.168.0.102 --port 8001
输出结果：
Namespace(host='192.168.0.102', port=8001)
"""
```

上述步骤中，首先使用 parser = argparse.ArgumentParser() 实例化参数解析器，然后使用 parser.add_argument() 添加需要解析的参数信息，最后使用 parser.parse_args() 获取命令行的参数信息。



### add_argument() 

parser.add_argument() 函数用来添加需要解析的参数信息，它的入参有下面一些：

```txt
参数1：命令行中参数的名称
action：传参时需要执行的功能，有以下几个：
	store：保存值，并转换为指定类型。默认行为
	store_const：定义参数时就设置默认值，但是参数不能是 Boolean 类型
	store_true/store_false：与 store_const 类似，保存 Boolean 类型的值
	append：向list中添加数据
	append_const：向list中添加常量值
type：指定参数值的类型，默认类型str。当传入的参数不能被转换为指定类型时，会抛出异常。
default：参数的默认值
dest：访问参数时，参数的别名
```

在 add_argument()  中，还可以为命令行参数指定简短的参数名，像下面这样：

```python
parser.add_argument('--host',
                    '-ho',
                    dest='h',
                    action="store",
                    default='127.0.0.1')
parser.add_argument('--port',
                    '-po',
                    dest='p',
                    action="store", type=int,
                    default=8000)
"""
命令行上传输参数时，有下面两种方式：
1）python 01_test.py --host 192.168.0.102 --port 8002
2）python 01_test.py -ho 192.168.0.102 -po 8002
"""
```



### parse_args()

函数 parser.parse_args() 用来获取命令行的参数信息，它的返回值的是一个 `Namespace`  对象，该对象的属性名就是传入的参数名，属性值是对应传入参数的值。如下：

```python
parser_args = parser.parse_args()
print(parser_args)  # Namespace(host='192.168.0.102', port=8001)
print(parser_args.host)  # 192.168.0.102
```

当在 add_argument() 函数中指定dest参数时，可以用过 dest 指定的参数访问属性值，像下面这样：

```python
import argparse


# 实例化参数解析器
parser = argparse.ArgumentParser(description="a argument parser")

# 添加参数
parser.add_argument('--host',
                    dest='h',
                    action="store",
                    default='127.0.0.1')
parser.add_argument('--port',
                    dest='p',
                    action="store", type=int,
                    default=8000)

parser_args = parser.parse_args()
print(parser_args)  # Namespace(h='192.168.0.102', p=8001)
print(parser_args.h)  # 192.168.0.102
print((parser_args.p))  # 8001
```





### action参数

上面例子中，action 参数的值都是 "store"，下面介绍下其它参数值的使用。

#### action_const

action_const表示传入指定参数名时，参数值是常量，不会改变。

```python
import argparse


parser = argparse.ArgumentParser()

parser.add_argument("-t", action='store_const',
                    dest='const',
                    const="const value")


arg_parser = parser.parse_args()
print(arg_parser.const)

"""
执行：python test.py -t
则 arg_parser.const 的值为 "const value"

此时如果执行 python test.py -t "const value" 
试图向常量中传入参数，则会抛出异常
"""
```

#### action_true

action_true或者action_false，这两个值与action_const类似，只是常量是 Boolean 类型。

例子如下：

```python
# @Author : WZ 
# @Time : 2019/12/29 10:37
# @Intro :


"""action='store_const'的使用
"""


import argparse


parser = argparse.ArgumentParser()

parser.add_argument("-t", action='store_true',
                    default=False,
                    dest='bool1')

parser.add_argument("-b", action='store_false',
                    default=True,
                    dest='bool2')


arg_parser = parser.parse_args()
print(arg_parser.bool1)
print(arg_parser.bool2)

"""
执行 python test.py -t 时，arg_parser.bool1 的值是 True
执行 python test.py 时，arg_parser.bool1 的值是False

执行 python test.py -b 时，arg_parser.bool2 的值是 False
执行 python test.py 时，arg_parser.bool2 的值是 True
"""
```

#### append

append功能是向list中追加数据，使用例子如下：

```python
import argparse


parser = argparse.ArgumentParser()

parser.add_argument("-a", action='append',
                    default=[],
                    dest='add')


arg_parser = parser.parse_args()
print(arg_parser.add)


"""
执行 python test.py，输出为 []
执行 python test.py -a one -a two -a second，输出为 ['one', 'two', 'second']
"""
```

#### append_const

```python
import argparse


parser = argparse.ArgumentParser()

parser.add_argument("-a", action='append_const',
                    const="const1",
                    default=[],
                    dest='add')
parser.add_argument("-b", action='append_const',
                    const="const2",
                    default=[],
                    dest='add')

arg_parser = parser.parse_args()
print(arg_parser.add)

"""
执行 python test.py，输出为 []
执行 python test.py -a，输出为 ['const1']
执行 python test.py -b，输出为 ['const2']
执行 python test.py -a -b，输出为 ['const1', 'const2']
"""
```



### 修改前缀

命令行参数默认前缀是 "-"，也可以使用 prefix_chars 指定其他前缀。

```python
import argparse


# 实例化参数解析器
parser = argparse.ArgumentParser(
    description="a argument parser",
    prefix_chars='-+/')


# 添加参数
parser.add_argument('+host',
                    dest='h',
                    action="store",
                    default='127.0.0.1')

parser_args = parser.parse_args()
print(parser_args.h)

"""
使用 prefix_chars 参数可以同时指定多个前缀
传参时，可以执行 python test.py +host 192.168.0.102
"""
```



此外，默认可以使用 -h 参数显式脚本支持的命令行参数信息。

```python
import argparse


# 实例化参数解析器
parser = argparse.ArgumentParser(
    description="a argument parser",
    prefix_chars='-+/')


# 添加参数
parser.add_argument('-host',
                    dest='h',
                    action="store",
                    default='127.0.0.1',
                    help="server host")

parser.add_argument('-port',
                    dest='p',
                    action="store",
                    default=8001,
                    help="server port")

"""
执行 python test.py -h
会在终端输出以下信息：
usage: 01_test.py [-h] [-host H] [-port P]

a argument parser

optional arguments:
  -h, --help  show this help message and exit
  -host H     server host
  -port P     server port
"""
```



以上就是 argparse 的简单用法。