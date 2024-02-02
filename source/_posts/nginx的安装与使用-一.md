---
title: nginx的安装与使用(一)
date: 2020-04-19 15:39:32
tags:
categories: Nginx
---

今天主要介绍下 nginx 的安装与简单使用。

<!--more-->

# 安装

```txt
安装步骤如下（ubuntu18.04）：
1. 安装依赖项：
	sudo apt-get update
	sudo apt-get install build-essential zlib1g-dev libpcre3 libpcre3-dev libssl-dev    			libxml2-dev libgeoip-dev libgoogle-perftools-dev libperl-dev libtool
	sudo apt-get install openssl
2. 下载 nginx 源码包并解压
	wget http://nginx.org/download/nginx-1.16.1.tar.gz
3. 执行安装命令 （ngx_http_proxy_connect_module是一个正向代理的模块，如果有需求，还可以安装其他模块）
	./configure --prefix=/usr/local/nginx --with-http_ssl_module --with-stream 
	 --add-module=/home/wyzane/softwares/ngx_http_proxy_connect_module
	sudo make
	sudo make install
安装完成后，就可以启动了

4. 创建软链接
	sudo ln -s /usr/local/nginx/sbin/nginx /usr/local/bin
5. 查看nginx是否启动成功
	浏览器中输入：http://127.0.0.1:8081/
	出现nginx的欢迎页即表明nginx启动成功
	
	
安装步骤如下（centos8.1）：
1. 安装依赖
	yum update
	yum install gcc-c++
	yum install -y pcre pcre-devel
	yum install -y zlib zlib-devel
	yum install -y openssl openssl-devel
2. 下载 nginx 源码包并解压
	wget http://nginx.org/download/nginx-1.16.1.tar.gz
3. 安装patch命令（对于nginx-1.16，ngx_http_proxy_connect_module需要打补丁）
	yum -y install patch
4. 给ngx_http_proxy_connect_module打补丁
	patch -p1 < /home/wz/softwares/ngx_http_proxy_connect_module/patch/proxy_connect_rewrite_101504.patch
5. 执行命令
	./configure --prefix=/usr/local/nginx --with-http_ssl_module --with-stream 
		--add-module=/home/wz/softwares/ngx_http_proxy_connect_module
	make
	make install
	
	make clean：重新编译安装
6. 创建软链接
	sudo ln -s /usr/local/nginx/sbin/nginx /usr/local/bin


可能的问题：
1）centos编译安装Nginx提示 bash: make: command not found
	解决：yum -y install gcc automake autoconf libtool make
```



```txt
ubuntu上安装net工具：
sudo apt-get install net-tools
安装后可以使用 netstat 命令
```



下面的操作都是在 ubuntu18.04 上完成的。

# 设置系统服务

```txt
将nginx添加到系统服务的步骤如下：
1. 新增nginx.service文件
	sudo vim /lib/systemd/system/nginx.service
2. 添加如下内容：
	[Unit]
    Description=nginx - high performance web server
    After=network.target remote-fs.target nss-lookup.target

    [Service]
    Type=forking
    ExecStart=/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
    ExecReload=/usr/local/nginx/sbin/nginx -s reload
    ExecStop=/usr/local/nginx/sbin/nginx -s stop

    [Install]
    WantedBy=multi-user.target
    
    配置说明：
        [Unit]部分
        Description:描述服务
        After:依赖，当依赖的服务启动之后再启动自定义的服务

        [Service]部分
        Type=forking是后台运行的形式
        ExecStart为服务的具体运行命令(需要根据路径适配)
        ExecReload为重启命令(需要根据路径适配)
        ExecStop为停止命令(需要根据路径适配)
        PrivateTmp=True表示给服务分配独立的临时空间
        
        [Install]部分
		服务安装的相关设置，可设置为多用户
3. 配置完成后，常用命令如下：
	# 设置了自启动后，任意目录下执行
    systemctl enable nginx.service
    # 启动nginx服务
    systemctl start nginx.service
    # 设置开机自动启动
    systemctl enable nginx.service
    # 停止开机自动启动
    systemctl disable nginx.service
    # 查看状态
    systemctl status nginx.service
    # 重启服务
    systemctl restart nginx.service
    # 查看所有服务
    systemctl list-units --type=service
```



# 常用命令

```txt
-c: 指定开启nginx时使用的配置文件

nginx -s: 发送信号
nginx -s stop
nginx -s quit  优雅的停止服务
nginx -t: 修改配置文件后，先测试
nginx -v：查看nginx版本

./nginx -s reload 不停止Nginx服务的方式重启nginx
重启命令还有如下方式：
kill -SIGHUP masterpid  与reload相同，都是重启nginx服务（主进程重新生成子进程）

此外，还有一个命令：
kill -SIGTERM childpid  向子进程发送退出命令，并告知主进程，主进程会产生新的子进程
```



# 目录结构

```txt
解压后nginx的目录结构如下：
auto: 文件夹
CHANGES: 文件，介绍nginx每个版本的特性
conf: 配置文件所在文件夹
configure: 脚本，在编译前执行，生成中间文件
contrib:
html: 提供两个标准html文件: 50x.html,index.html
man: linux对nginx的帮助文件  man ./nginx.8
src: nginx的源代码
```

# 组成部分

```txt
nginx 主要由二进制文件（定义nginx的功能）、conf配置文件（控制nginx的行为）、access.log（记录nginx的执行信息）、error.log（记录nginx的错误信息） 这四部分组成。
```



# 热部署方式

```txt
热部署nginx：不停止当前nginx的情况下升级nginx，只需要更换nginx二进制文件
步骤如下：
1.备份当前nginx二进制文件： cp nginx nginx.old
2.用编译后的最新版本的nginx二进制文件替换掉现在的nginx文件
3.给当前nginx的master进程发送信号：kill -USR2 pid
	此时会使用新的二进制文件生成新的master进程，老的进程也在使用，两者会平滑的过度；老的进程不会再监听80或者443端口
4.向老的master进程发送信号，优雅的关闭所有进程：kill -WINCH pid。此时老的master进程已经没有了子进程。
	若再想恢复老的版本，可以向master进程发起reload命令（用于做版本回退）

日志切割：
1.备份原先的日志文件 mv old.log bak.log
2.重新生成日志文件 ./nginx -s reopen  会重新生成一个old.log日志文件
	（可以每天或者每周定时执行日志切割）
```



# 搭建静态资源服务器

```txt
搭建静态资源服务器时，可以将下面的内容放在http块下：
# 使用了很多变量，日志格式命名，便于区分不同域名下的日志格式
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';

access_log  logs/access.log  main;

server {
        listen       8081;
        server_name  192.168.0.105;

        #charset koi8-r;

        index  index.html index.htm;
        access_log  logs/host.access.log  main;

        location / {
            alias  docs/;
            autoindex  on;  # 显示docs目录结构
        	set $limit_rate 1k; # limit_rate是内置变量，用于限制访问速度，每秒传输多少字节到浏览器中 
            # index  index.html index.htm;
        }

        #error_page  404              /404.html;
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
    
上面的docs就表示静态资源文件夹
```



