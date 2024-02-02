---
title: django开发之在docker中部署项目
date: 2020-07-26 15:51:27
tags: django, docker
categories: Python
---

今天整理了一下如何在docker中部署django项目。

<!--more-->

# 项目环境

```txt
python3.6 django2.0.5 nginx mysql5.7 gunicorn
```

# 项目文件

项目主目录为 blog 目录，需要编写的文件包括（省去了其他django文件）

```
blog/Dockerfile, blog/gunicorn.conf, blog/start.sh, nginx/Doickerfile, nginx/nginx.conf, docker-conpose.yml
```

# 项目配置

## Docker配置

blog/Dockerfile 内容如下：

```txt
FROM python:3.6    # 选择基础镜像,这里的基础镜像也可以选择ubuntu,centos等，但是下面的配置就会发生变化

# 创建工作目录
RUN mkdir /blog  

#设置工作目录
WORKDIR /blog

#将当前目录加入到工作目录中
ADD . /blog
RUN pip3 install -r requirements.txt
#对外暴露端口
EXPOSE 80 8080 8000 5000
#设置环境变量
ENV SPIDER=/blog
```

上面基础镜像使用的是python:3.6,而不是ubuntu、centos。如果是ubuntu、cenos，Dockerfile文件中需要配置python环境

nginx/Doickerfile 内容如下：

```txt
FROM nginx    # nginx镜像，最好是先拉取到本地


# 对外暴露端口
EXPOSE 80 8000

RUN rm /etc/nginx/conf.d/default.conf  # 删除原有配置文件
ADD nginx.conf  /etc/nginx/conf.d/   # 添加配置文件
RUN mkdir -p /usr/share/nginx/html/static  # 创建静态资源文件夹
```

docker-compose.yml 内容如下：

```txt
version: "3"

services:


db:
    image: mysql:5.7  # mysql镜像，最好先拉取到本地
    environment:
      - MYSQL_HOST=localhost
      - MYSQL_DATABASE=docker
      - MYSQL_USER=root
      - MYSQL_PASSWORD=wyzane
      - MYSQL_ROOT_PASSWORD=wyzane
    volumes:
      - /home/wyzane/pyprojects/db:/var/lib/mysql  # 将宿主机与容器中的文件映射
    restart: always  # 若容器运行出现问题，会自动重启容器

  web:
    build: ./blog
    ports:
    - "8000:8000"
    volumes:
    - ./blog:/blog
    - /tmp/logs:/tmp
    command: bash start.sh  # 执行命令，有多种格式
    links:
    - db
    depends_on:
    - db
    restart: always

nginx:
    build: ./nginx
    ports:
    - "80:80"
    volumes:
    - ./blog/static:/usr/share/nginx/html/static:ro
    links:
    - web
    depends_on:
    - web
    restart: always
```



## gunicorn配置

blog/gunicorn.conf 的配置内容如下：

```txt
workers=2
bind='0.0.0.0:8000'
proc_name='blog'
```

## start文件配置

blog/start.sh 文件内容如下，用于启动项目：

```txt
#!/bin/bash
python manage.py collectstatic --noinput&&
python manage.py makemigrations&&
python manage.py migrate &&
gunicornblog.wsgi:application -c gunicorn.conf
```

## nginx配置

nginx/nginx.conf 文件配置如下：

```txt
server {
    listen      80;
    server_name localhost;

    location /static {
        alias /usr/share/nginx/html/static;
    }

    location / {
        proxy_pass http://web:8000;
        proxy_set_header Host $host;
        proxy_redirect off;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

}
```

# 项目启动

配置完成后，依次执行以下命令：

```txt
docker-compose build
docker-compose up -d
```

执行完成后，通过 docker images 命令可以查看新增了2个镜像，docker ps 命令可以查看启动了3个容器。

多执行几次docker ps，当容器的STATUS是以Restarting开头时，表示这个容器运行时发生了错误。执行docker logs CONTAINERID可以查看容器出错的具体原因。

若上述容器都成功运行，则在浏览器中输入http://127.0.0.1:80/api/v1/articles/info/时（这是django中的一个视图，在这里没有写出来，不同的小伙伴可以替换成不同的接口），视图会返回相应的结果。
以交互方式进入容器：

```txt
docker exec -it CONTAINERID /bin/bash
```

后，进入mysql数据库，会看到在数据库中生成了相应的表。

# 遇到问题

在运行3个容器后，web容器一直报错，通过 docker logs CONTAINERID查看主要错误信息如下：

```txt
django.db.utils.OperationalError: (2003, 'Can\'t connect to MySQL server on \'mariadb55\' (111 "Connection refused")')
```

解决方案在这里：

```txt
https://stackoverflow.com/questions/47979270/django-cannot-connect-mysql-in-docker-compose
```

主要是在settings.py中，将database配置中的HOST值改成db,而不是127.0.0.1，指向docker-compose.yml中的db服务。

