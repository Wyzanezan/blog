---
title: Python3中如何使用Fabric
date: 2019-12-22 21:54:21
tags: python
categories: Python
---

Fabric是基于Python实现的SSH命令行工具，简化了SSH的应用程序部署及系统管理任务，它提供了系统基础的操作组件，可以实现本地或远程shell命令，包括：命令执行、文件上传、下载及完整执行日志输出等功能。Fabric在Paramiko的基础上做了更高一层的封装，操作起来会更加简单。

<!--more-->

## Fabric的使用

Fabric中，fabric和fabric2是官方版本，fabric3是从fabric中fork出来的非官方版本。

今天介绍下 python3 中 Fabric 的使用，即 fabric3 的使用。

### 安装

```txt
pip install fabric3
```



### 初次使用

使用 fabric3 时，先创建一个 fabric 文件（文件名为 fabfile.py 或者其他名称）

```python
# fabfile.py:
def hello():
    print("hello fabric")
```

然后再执行 fab hello，结果如下：

```txt
hello fabric

Done.
```

执行fab命令时，若文件名是 fabfile.py，则可以不用指定。文件名不是 fabfile.py 时，需要使用 -f 指定文件名。

```python
# test.py:
def hello():
    print("hi fabric")
    
def hello2(name):
    print("hello:", name)
```

执行 fab -f test hello，结果如下：

```txt
hi fabric

Done.
```

执行函数时添加参数：fab -f test hello2:name=wyzane，结果如下：

```txt
hello: wyzane

Done.
```



### 再次使用



在 fabric 中，在本地执行命令可以使用 fabric.api 中的 local 方法。

#### 启动Django项目

使用 fabric 启动 Django 项目。新建 Django 项目，目录结构如下：

```txt
DjangoFabric
	- DjangoFabric
	- templates
	- manage.py
	- fabfile.py
```

fabfile.py：

```python
from fabric.api import local

def startup():
    local("python manage.py runserver")

```

执行 fab startup，会启动 Django 项目。



#### 执行Git命令

在 git 仓库下，创建 fabfile.py 文件，文件内容如下：

```python
from fabric.api import local

def git_pull():
    local("git pull origin master")

def git_push():
    local("git add . && git commit -m 'test' && git push")

```

执行 fab git_pull 和 fab git_push 后，会进行代码的拉取和推送操作，这样看起来确实很方便。



#### 执行Linux命令

远程登陆到服务器，然后执行 Linux 命令

```python
from fabric.api import *


env.hosts = ['192.168.172.128']
env.user = 'wyzane'
env.password = 'wyzane'


def cmd():
    # run：执行 Linux 命令
    run('touch /home/wyzane/tmp.txt')

```

执行 fab cmd，就会在服务器上创建 tmp.txt 文件。



#### 上传文件

```txt
fabric.operations.put((local_path=None, remote_path=None, use_sudo=False, mirror_local_mode=False, mode=None, use_glob=True, temp_dir=''))
```

使用上面的函数进行文件上传，或者使用 fabric.api 中的函数

```python
from fabric.operations import put
from fabric.operations import env
# 或者使用 from fabric.api import put, env


env.hosts = ['192.168.172.128']
env.user = 'wyzane'
env.password = 'wyzane'


def upload():
    # put：上传文件到远程服务器
    put('mem.py', '/home/wyzane/')
```

执行 fab upload，就会将文件上传到服务器。



#### 下载文件

```txt
fabric.operations.get(remote_path, local_path=None, use_sudo=False, temp_dir="") 
```

使用上面的函数进行文件下载，可以下载单个文件，也可以下载一个文件夹。返回值是一个列表，列表中的元素是下载文件在本地的路径。

```python
from fabric.operations import get
from fabric.operations import env
# 或者使用 from fabric.api import get, env


env.hosts = ['192.168.172.128']
env.user = 'wyzane'
env.password = 'wyzane'


def download():
    results = get("/home/wyzane/test")
    print(results)
    
"""返回结果如下：
['D:\\pyprojects\\DjangoFabric\\192.168.172.128\\test\\tmp2.txt', 'D:\\pyprojects\\DjangoFabric\\192.168.172.128\\test\\tmp1.txt']
"""
```



#### 常用API

下面介绍下 fabric 中的常用 API，下面的 API 可以从 fabric.operations 引入：

```txt
1. get(remote_path, local_path=None, use_sudo=False, temp_dir='')：从远程服务器上下载一个或者多个文件
2. local(command, capture=False, shell=None)：在本地运行命令，像上面的 git 命令
3. put(local_path=None, remote_path=None, use_sudo=False, mirror_local_mode=False, mode=None, use_glob=True, temp_dir='')：上传一个或者多个文件到远程服务器
4. reboot(wait=120, command='reboot', use_sudo=True)：重启远程服务器
5. run(command, shell=True, pty=True, combine_stderr=None, quiet=False, warn_only=False, stdout=None, stderr=None, timeout=None, shell_escape=None, capture_buffer_size=None)：在远程服务器上运行shell命令
6. sudo(command, shell=True, pty=True, combine_stderr=None, user=None, quiet=False, warn_only=False, stdout=None, stderr=None, group=None, timeout=None, shell_escape=None, capture_buffer_size=None)：以超级用户权限在远程服务器上运行shell命令
```

fabric 文档地址为：[http://docs.fabfile.org/en/1.14/](http://docs.fabfile.org/en/1.14/)