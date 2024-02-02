---
title: python开发中locust库的使用
date: 2020-05-05 17:02:06
tags: python
categories: Python
---

locust是一个易于使用的、分布式的压测工具，今天介绍下python开发中locust的使用。

<!--more-->

# locust基础

安装locust：

```txt
pip install locust

# 查看是否安装成功
locust --help
```



locust中的常用类如下：

```txt
Locust
代表请求待测系统的一个用户，使用task_set属性指定用户的行为，task_set属性的值是一个TaskSet类。如果压测http接口，可以使用下面的HttpLocust类。

HttpLocust
代表请求待测系统的一个http用户，使用task_set属性指定用户的行为，task_set属性的值是一个TaskSet类
HttpLocust的属性：
client：一个HttpSession实例，支持cookie，可以在http请求间保持对话。

TaskSet
定义一些任务供locust用户调用，TaskSet有下面一些属性：
client: Locust实例中的client属性
tasks: 一些待执行任务的集合，当它是一个列表时，列表中的任务会随机执行；是元祖或者字典时（(callable,int)或者{callable:int}，int是执行的权重）,会根据权重执行任务
max_wait: 任务执行的最大时间间隔
min_wait: 任务执行的最小时间间隔
locust: 当TaskSet类被实例化后，该参数表示一个locust类实例
```



# 压测脚本例子

```python
import json

from locust import HttpLocust, TaskSet, task


class HttpTest(TaskSet):

    def on_start(self):
        """初始化
        """
        pass

    @task(1)
    def hobby_detail(self):
        data = json.dumps(self.locust.hobby_detail_data)

        resp = self.client.post("/api/v1/hobby/detail",
                                data=data,
                                headers=self.locust.headers)
        content = json.loads(resp.text, encoding="utf-8")
        print("content:", content)

    # @task(1)  # task参数表示执行权重，值越大，执行概率越高
    # def hobby_list(self):
    #     self.client.post("/api/v1/hobby/list")


class HttpUser(HttpLocust):

    # locust -f test.py --host=http://192/168/0/102:8001 --port=8090

    # 待请求地址，也可以在命令行中使用--host参数指定
    host = "http://192.168.0.102:8002"

    # 指定TaskSet类
    task_set = HttpTest

    # 请求头
    headers = {"Content-Type": "application/json"}

    # 请求参数
    hobby_detail_data = {
        "hobbyId": 57
    }

    # 每个用户执行两个任务的时间间隔(单位: 毫秒)，
    # 上下限，具体值随机取，默认间隔位固定值1s
    min_wait = 1000
    max_wait = 3000
```



# 命令行测试

在命令行中输入以下命令进行测试：

```txt
locust -f test.py --host=http://192.168.0.102:8001 --port=8089
```

在命令行中测试时，还可以提供以下参数：

```txt
locust -f test.py --csv=foobar --no-web -c2 -t10s

# --no-web：不使用web界面测试
# --cvs: 测试结果的位置
# -c: 设置虚拟用户数
# -r: 设置每秒启动用户数
# -t: 设置运行时间
```



# 浏览器测试

我们也可以在 locust 为我们提供的管理控制台上进行测试。 在浏览器中访问地址 http://localhost:8089/ 可以进入 locust 的界面，填入Number of users to simulate和Hatch rate参数（分别表示虚拟用户数量和每秒产生的用户数量），然后点击Start swarming，即可开始测试工作。





# 分布式测试

locust也支持分布式测试。主从机中必须运行相同的测试代码（把主机中代码复制一份到多个从机中），主机负责收集测试数据，从机进行施压测试；

在主机终端中执行：

```txt
locust -f test.py --master
```

从机终端执行：

```txt
locust -f  test.py --slave --master-host=master ip
```

开始测试：

```txt
locust -f test.py  --csv=foobartt --no-web -c2 -t10s --master
```

