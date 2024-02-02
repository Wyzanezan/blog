---
layout: “项
title: 分享一个项目重启的shell脚本
date: 2020-05-24 15:15:03
tags: linux
categories: Linux
---

今天分享一个重启 Django 项目的 shell 脚本，当然，修改一下也可以重启其他项目。

<!--more-->

脚本如下：

```shell
#!/bin/sh


# 获取项目进程号
pids=$(ps -ef | grep 8000 | awk '{if($9=="manage.py") {print $2}}')
echo "pid-"${pids}
# kill项目进程
kill -9 ${pids}
if [ $? -eq 0 ];then
    echo "kill success"
    # 启动项目
    nohup python manage.py runserver 127.0.0.1:8000 &amp;
    if [ $? -eq 0 ];then
        echo "restart success"
    else
        echo "restart failure"
    fi
else
    echo "kill failure"
fi
echo "end!!!"

```

