---
title: nginx中https的配置
date: 2020-06-06 10:42:00
tags:
categories: Nginx
---

今天聊一下如何在nginx中配置https

在web应用开发中，为保证前端访问后端服务器的安全，需要使用https连接，现在来聊一下如何在nginx中配置https.

<!--more-->

# 申请ssl证书

在阿里云，腾讯云，华为云等云服务提供商的网站一般都会有免费ssl证书，申请一个即可；下面配置以华为云为例展开。

# 下载证书

证书申请后，下载它，会得到server.key和server.crt两个文件；在与nginx.conf同目录下创建ssl文件夹(名字任意), 把这两个证书放入刚创建的文件夹中；

# 配置

在nginx.conf的server中增加如下配置：

```nginx
server {
    listen                 443;
    server_name            www.test.com # 域名         
    ssl                    on;  # 启用ssl功能            
    ssl_certificate        ssl/server.crt;           
    ssl_certificate_key    ssl/server.key;           
    ssl_session_timeout    5m;       # 客户端可以重用会话参数的时间
    ssl_protocols          TLSv1 TLSv1.1 TLSv1.2;    # 使用的协议     
    ssl_ciphers            ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;   # 配置加密套件    
    ssl_prefer_server_ciphers     on;

       ...  
}
```

在配置443端口之前，需要先打开防火墙和443端口，下面介绍如何打开防火墙，以Centos7为例：

```txt
1) 开启防火墙: systemctl start firewalld
	查看防火墙状态: systemctl status firewalld
2) 查看开通了哪些端口: firewall-cmd –list-ports
3) 开通443端口: firewall-cmd –zone=public –add-port=443/tcp –permanent
4) 重新加载下防火墙配置: firewall-cmd –reload
	注意: 如果还有其它应用在运行，开启防火墙后，需要开通相应的端口，否则不能访问。
```

打开防火墙后，需要将80端口重定向至443端口，在nginx.conf的server上面增加如下的server：

```nginx
server {
    listen 80;
    server_name www.test.com;
    rewrite ^(.*)$ https://www.test.com$1 permanent; 
}
```



# 验证

执行 /usr/sbin/nginx -t，返回如下信息则表示配置成功：

```txt
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

重启nginx：

```txt
systemctl restart nginx
```



nginx中配置 https 就介绍到这里。