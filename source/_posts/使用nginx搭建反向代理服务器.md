---
title: 使用nginx搭建反向代理服务器
date: 2020-07-20 21:56:31
tags:
categories: Nginx
---

今天分享下，如何使用 nginx 搭建具有反向代理功能的服务器。

首先把 nginx 官方文档地址贴出来，有问题可以随时查阅：

```txt
http://nginx.org/en/docs/
```

<!--more-->

# 反向代理

首先介绍下正向代理与反向代理的区别。

正向代理是发送请求时，隐藏了真正的客户端，即服务端不知道请求的客户端信息，客户端的请求都被代理服务器代替来请求。正向代理的常用场景就是在爬虫系统中。
反向代理是客户端不知道将要请求的服务器的信息，而是请求一个代理服务器，代理服务器再去请求上游服务器，并把上游服务器的响应返回给客户端。

其实，正向代理和反向代理的请求流程都是一样的，只是被代理对象不同。正向代理代理的对象是客户端，反向代理代理的对象是服务端，也就是说，谁被代理就隐藏了谁。

## 反向代理服务器搭建

下面看一个 nginx 配置反向代理的例子。

使用 nginx 搭建一个反向代理，上游服务器是一个静态资源 web 服务，配置如下：

nginx.conf：

```nginx
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';


    #access_log  logs/access.log  main;

    sendfile        on;
    keepalive_timeout  65;

    #gzip  on;
    
    server {
        listen       127.0.0.1:8081;

        index  index.html index.htm;
        access_log  logs/host.access.log  main;

        location / {
            alias  docs/;   # 指定静态资源目录
            autoindex  on;  # 这个设置会显示docs目录结构
            set $limit_rate 1k;  # 限制速率
        }
    }

}
```

反向代理服务器配置如下：

nginx2.conf：

```nginx
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    include       ../confs/*.conf;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';


    #access_log  logs/access.log  main;

    sendfile        on;

    keepalive_timeout  65;

    #gzip  on;
    
    # 指定上游服务,这里可以配置负载均衡算法
    upstream local {
        server  127.0.0.1:8081;
    }
    
    server {
        listen       8082;
        server_name  192.168.0.105;

        access_log  logs/host.access.log  main;

        location / {
            proxy_set_header  Host $host;
            proxy_set_header  X-Real-IP $remote_addr;
            proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;

        	# 将请求转发到指定的上游服务器
            proxy_pass  http://local;

        }
    }

}
```

分别启动两个服务器：

```txt
sudo nginx -c /usr/local/nginx/conf/nginx.conf
sudo nginx -c /usr/local/nginx/conf/nginx2.conf
```

启动后，再浏览器地址栏输入 http://192.168.0.105:8082/ 便可以访问静态资源了。

## 反向代理服务加入缓存功能

如果想提高响应速率，可以在 nginx 反向代理服务器上增加缓存功能。

反向代理增加缓存后，客户端的请求会首先从nginx缓存中获取数据，并返回给客户端，当没有缓存数据或者缓存数据过期时，才会请求上游服务器。

```nginx
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    include       ../confs/*.conf;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';

    # proxy_cache_path 参数用于设置缓存路径，并且可以指定其他参数
    # levels=1:2 参数指定缓存等级，值从1到3（暂时没弄明白这里的意思）
    # keys_zone  参数指定共享内存大小和名称
    # inactive 参数指定数据缓存的时长，默认10分钟
    # max_size 参数指定缓存数据的最大值
    # use_temp_path 指定是否使用临时文件存放缓存数据
    指定保存缓存数据的路径, keys_zone表示开了一个10m的共享内存用于存放key，my_cache是共享内存名称 
    proxy_cache_path  /tmp/nginx_cache levels=1:2 keys_zone=my_cache:10m max_size=10g 
                      inactive=60m use_temp_path=off;
   

    #access_log  logs/access.log  main;

    sendfile        on;

    keepalive_timeout  65;

    #gzip  on;
    
    # 指定上游服务,这里可以配置负载均衡算法
    upstream local {
        server  127.0.0.1:8081;
    }
    
    server {
        listen       8082;
        server_name  192.168.0.105;

        access_log  logs/host.access.log  main;

        location / {
            proxy_set_header  Host $host;
            proxy_set_header  X-Real-IP $remote_addr;
            proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;


            # 指定刚刚开辟的共享内存
            proxy_cache  my_cache;

            # 定义缓存数据对应的 key 由什么组成
            proxy_cache_key  $host$uri$is_args$args;
            
            # 给指定的想用窗台吗设置缓存过期时间，最后一个值时过期时间，可以是小时、分钟等单位
            proxy_cache_valid 200 304 302 1d;            
           
            proxy_pass  http://local;

        }
    }
}
```

上面的配置中也介绍了一些常用的缓存参数，详细参数信息可以参考 nginx 的官方文档，文档地址为：

```txt
http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_path
```

# 负载均衡

既然提到了反向代理，那么负载均衡是一定要说一下的。负载均衡的大意就是反向代理服务器把客户端的请求均匀的，或者按照一定的规则分发给上游服务器，从而保证服务可用性。

实现负载均衡有以下几种方式：

```txt
1. 水平扩展：基于 Round-Robin或者least-connect算法进行请求分发
2. 按功能扩展：根据请求URL，按照功能分发请求到上游服务器
3. 基于用户信息扩展：根据请求客户端ip或者其他信息分发客户端请求（基于hash的某些算法）
```

上面的方式可以组合起来使用，并不是只能单独使用。

nginx支持多种协议的反向代理，即客户端使用http协议请求服务时，nginx可以将其转换成fastcgi、uwsgi、rpc、websocket等协议，再向上游服务器发起请求。

## 负载均衡策略

nginx 中负责与上游服务器交互的模块是 upstream 模块，该模块中提供了一个基本的负载均衡算法 Round-Robin。

upstream 模块中，指定上游服务器的指令是 upstream，上面的例子中已经见过，该指令的作用域是 http 上下文。在 upstream 中，使用 server 指令指定上游服务器地址信息。

Round-Robin负载均衡算法介绍：

```txt
该算法功能：以轮询的方式访问 server 指令指定的上游服务器，该算法集成在 Nginx 的 upstream 模块中。
```

该算法提供以下指定：

```txt
wieght: 上游服务器权重，默认1
max_conns：上游服务器的最大并发连接数，仅用于单 worker 进程，默认0，表示没有限制
max_fails: 与fail_timeout配合使用。在fail_timeout时间段内，最大的失败次数。当达到最大失败次数时，在fail_timeout时间内，该上游服务器不再被选择
fail_timeout:单位秒，默认为10，与max_fails配合使用，有两个功能：，指定一段时间内最大的失败次数max_fails；当到达max_fails后，该上游服务器不能再被访问
```



## 反向代理服务中加入负载均衡

### 基于  Round-Robin 算法的负载均衡

#### 反向代理服务配置

nginx.conf：

```nginx
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
  
  
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';


    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
    
    server {
        listen       127.0.0.1:8081;
        # server_name  192.168.0.103;

        #charset koi8-r;

        index  index.html index.htm;
        access_log  logs/access.log  main;
        error_log   logs/error.log;

        location / {
            alias  docs/;
            autoindex  on;  # 显示docs目录结构
            set $limit_rate 1k;
            # index  index.html index.htm;
        }
    }
    # 指定反向代理的配置文件，配置文件中加入了负载均衡
    include reverse_proxy.conf;
}
```

reverse_proxy.conf：

```nginx
upstream proxy {
    server 127.0.0.1:8011 weight=2 max_conns=2 max_fails=2 fail_timeout=5;
    server 127.0.0.1:8012;
}


server {
    listen 8091;
    error_log  logs/reverse_proxy.err.log;

    location / {
        proxy_pass  http://proxy;
    }
}
```

#### 上游服务配置

nginx2.conf：

```nginx
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
  
  
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';


    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
    
    server {
        listen       127.0.0.1:8082;
        # server_name  192.168.0.103;

        #charset koi8-r;

        index  index.html index.htm;
        access_log  logs/access.log  main;
        error_log   logs/error.log;

        location / {
            alias  docs/;
            autoindex  on;  # 显示docs目录结构
            set $limit_rate 1k;
            # index  index.html index.htm;
        }
    }
    # 指定上游服务配置文件
    include upstream.conf;
}
```

upstream.conf：

```nginx
server {
    # 监听 8011 端口
	listen 8011;
	default_type text/plain;
	return 200 '8011 server response';
}

server {
    # 监听 8012 端口
	listen 8012;
	default_type text/plain;
	return 200 '8012 server response';
}
```

上面的服务配置好以后，可以分别启动服务：

```txt
启动反向代理服务：sudo nginx -c nginx.conf
启动上游服务：sudo nginx -c nginx2.conf
```

服务启动后，在终端多次执行 curl localhost:8091，可以发现，响应结果中，'8011 server response' 和 '8012 server response' 的比例大致为 2:1，这说明我们配置的负载均衡已经生效了。



### 基于 ip hash 的负载均衡

基于 ip 地址的 hash 算法可以根据客户端 ip 的不同，将请求转发到不同的上游服务器，它的配置如下（仅仅修改了 reverse_proxy.conf文件）：

reverse_proxy.conf：

```txt
upstream proxy {
    ip_hash;
    server 127.0.0.1:8011 weight=2 max_conns=2 max_fails=2 fail_timeout=5;
    server 127.0.0.1:8012 weight=1;
}


server {
    set_real_ip_from  192.168.0.105;
    real_ip_recursive on;
    real_ip_header X-Forwarded-For;

    listen 8091;
    error_log  logs/reverse_proxy.err.log;

    location / {
        proxy_pass  http://proxy;
    }
}
```

需要注意的是，上面使用了 realip 模块的一些指令，现在的 nginx 版本中，realip模块默认没有编译进 nginx，如果要使用 该模块，需要将 realip 模块编译进 nginx 中。如何把某个模块编译进已安装的 nginx 中，可以查看我的另一篇博客。

上面指定的大致意思如下：

```txt
ip_hash：根据用户的 ip 地址分发客户端的请求到上游服务器
set_real_ip_from：设置可信任的地址
real_ip_recursive on：表示原始客户端 ip 地址会被请求头中的某个非信任 ip 地址（由real_ip_header决定）替换
real_ip_header X-Forwarded-For：表示以 X-Forwarded-For 中的最后一个 ip 作为 ip_hash 对应的 ip 地址
```

具体解释可以查看官方文档：

```txt
http://nginx.org/en/docs/http/ngx_http_realip_module.html#set_real_ip_from
```

配置完成重启后，我们可以在终端执行以下命令来测试：

```txt
curl -H "X-Forwarded-For: 10.112.23.45" localhost:8091
```

可以更换 ip ，查看返回的结果。

### 基于关键字  hash 的负载均衡

使用关键字 hash 时，reverse_proxy.conf的配置如下：

```txt
upstream proxy {
    # 使用特定的字符串作为hash值
    hash user_$arg_username;
    server 127.0.0.1:8011 weight=2 max_conns=2 max_fails=2 fail_timeout=5;
    server 127.0.0.1:8012 weight=1;
}


server {
    set_real_ip_from  192.168.0.105;
    real_ip_recursive on;
    real_ip_header X-Forwarded-For;

    listen 8091;
    error_log  logs/reverse_proxy.err.log;

    location / {
        proxy_pass  http://proxy;
    }
}
```

请求服务时时，传入不同的 username 参数，会请求不同的上游服务器。

### hash算法优化

当有上游服务器宕机或者需要扩容时，hash 算法会导致负载均衡的路由发生变化，这样会导致一系列问题（例如缓存失效）。一致性 hash 算法可以解决这个问题。

nginx 中配置一致性 hash 比较简单，配置如下：

reverse_proxy.conf：

```txt
upstream proxy {
    # 使用特定的字符串作为hash值
    hash user_$arg_username consistent;
    server 127.0.0.1:8011 weight=2 max_conns=2 max_fails=2 fail_timeout=5;
    server 127.0.0.1:8012 weight=1;
}


server {
    # set_real_ip_from  192.168.0.105;
    # real_ip_recursive on;
    real_ip_header X-Forwarded-For;

    listen 8091;
    error_log  logs/reverse_proxy.err.log;

    location / {
        proxy_pass  http://proxy;
    }
}
```

在原来的 hash key 后面加上 consistent 即可实现一致性 hash，具体介绍可以查看官方文档：

```txt
http://nginx.org/en/docs/http/ngx_http_upstream_module.html#hash
```

### 基于最少连接数算法的负载均衡

最少连接数算法中，nginx 反向代理服务器会将客户端请求转发到连接数最少的上游服务器中。当有多个上游服务器最少连接数相同时，会按照 Roubd-Robin 算法将请求发送到这几个服务器中的一个。最少连接数算法的配置如下：

reverse_proxy.conf：

```txt
upstream proxy {
    least_conn;
    server 127.0.0.1:8011 weight=2 max_conns=2 max_fails=2 fail_timeout=5;
    server 127.0.0.1:8012 weight=1;
}


server {
    # set_real_ip_from  192.168.0.105;
    # real_ip_recursive on;
    real_ip_header X-Forwarded-For;

    listen 8091;
    error_log  logs/reverse_proxy.err.log;

    location / {
        proxy_pass  http://proxy;
    }
}
```

在 upstream 上下文中，加入 least_conn 指令即可配置最少连接数算法。

官方文档如下：

```txt
http://nginx.org/en/docs/http/ngx_http_upstream_module.html#least_conn
```

