---
title: 为已安装的nginx动态添加模块
date: 2020-07-25 17:47:09
tags:
categories: Nginx
---

今天介绍下如何为已安装的 nginx 动态添加模块。

<!--more-->

首先，我们下载需要编译进 nginx 模块，或者使用 nginx 自带的模块如：realip 模块。

然后，执行 nginx -V，查看nginx信息和之前的安装记录：

```txt
➜  conf nginx -V                                  
nginx version: nginx/1.16.1
built by gcc 7.5.0 (Ubuntu 7.5.0-3ubuntu1~18.04) 
built with OpenSSL 1.1.1  11 Sep 2018
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx --with-http_ssl_module --with-stream --with-mail=dynamic
```

可以看到，我们之前编译 nginx 时使用的参数是：

```txt
--prefix=/usr/local/nginx --with-http_ssl_module --with-stream --with-mail=dynamic
```

那么，我们这次编译时，如果想把 nginx 自带的模块编译进去，如 realip，则执行如下命令：

```txt
./configure --prefix=/usr/local/nginx --with-http_ssl_module --with-stream --with-mail=dynamic --with-http_realip_module
```

如果想把第三方模块编译进去，则可以执行：

```txt
./configure --prefix=/usr/local/nginx --with-http_ssl_module --with-stream --with-mail=dynamic -–add-module=第三方模块所在目录
```

上述命令执行完成后，再执行 make 命令（不需要指定 make install命令）就行了。

最后，将现有的 nginx 二进制文件备份（例如：备份文件 /usr/local/nginx/sbin/nginx），备份完成后，再使用刚刚生成的 nginx 二进制文件替换现有的 nginx 二进制文件既可。

