---
title: openrestry的安装
date: 2020-08-30 09:18:00
tags:
categories: Nginx
---

OpenResty 是一个基于 Nginx 的可伸缩的 Web 平台，提供了很多高质量的第三方模块；它也是一个强大的 Web 应用服务器，开发人员可以使用 Lua 脚本语言调用 Nginx 支持的各种 C 以及 Lua 模块。

今天介绍下 OpenResty 在 Ubuntu 1804 上的安装。

<!--more-->

# 安装依赖

安装依赖时，执行以下命令：

```txt
apt-get install libpcre3-dev libssl-dev perl make build-essential curl
```

需要注意的是，执行上面的命令时，可能会出现以下错误：

```txt
E: py3compile:243: Requested versions are not installed
```

这是因为我们安装的 python3 版本与 py3compile 不一致引起的，py3compile 是属于 python3-minimal 的，我们只需要安装对应的 python3-minimal 即可。python3-minimal 的下载地址为：

```txt
https://mirrors.edge.kernel.org/ubuntu/pool/main/p/python3-defaults/
```

下载完成后，执行以下命令安装：

```txt
sudo dpkg -i python3-minimal_3.6.7-1~18.04_amd64.deb
```

安装 python3-minimal 可能还会出现以下错误：

```txt
/var/lib/dpkg/info/python3-minimal.postinst: py3compile: not found
```

这是因为安装过程中，python3-minimal 会寻找 python3.6（根据安装版本不同而不同），我的系统中 /usr/bin 下只有python3，这时候添加一个 python3.6 的软链接即可。



# 下载及安装

依赖安装完成后，可以到以下地址下载相应版本的 OpenResty ：

```txt
http://openresty.org/cn/download.html
```

下载完成后，解压缩：

```txt
tar -xzvf openresty-1.17.8.2.tar.gz
```

执行配置及安装命令：

```txt
sudo ./configure --prefix=/usr/local/openresty --with-luajit
sudo make
sudo make install
```

命令执行完成后，在 /usr/local 下会生成一个 openrestry 目录，里面就行 openrestry 的一些源码文件、配置文件和可执行文件。

其它系统的安装步骤可以参考：

```txt
http://openresty.org/cn/installation.html
```

# 简单使用

## hello world的例子

我们可以在 /usr/local/openresty 目录下新建一个文件 conf/openrestry.conf 配置文件，文件内容如下：

```nginx
worker_processes  1;
error_log logs/error.log;
events {
    worker_connections 1024;
}
http {
    server {
        listen 8080;
        location / {
            default_type text/html;
            content_by_lua_block {
                ngx.say("<p>hello, world</p>")
            }
        }
    }
}
```

然后启动 openrestry 服务：

```txt
sudo ./nginx -c /usr/local/openresty/conf/openrestry.conf
```

执行命令，请求 openrestry 服务：

```txt
curl http://localhost:8080/
```

请求后，会返回下面的信息：

```txt
<p>hello, world</p>
```



## 接收请求参数

openresty 中接收客户端请求参数的配置为：

conf/params.conf：

```nginx
events {
    worker_connections 1024;
}
http {
    server {
        listen 8080;

        location /test {
            default_type text/html;
            content_by_lua '
            
                local params = ngx.req.get_uri_args();
                ngx.print(params.name);
                ngx.print(params.age);
            ';
        }

    }
}
```

在终端请求 http://localhost:8080/test/?name=aaa&age=12 时，返回的内容为：

```txt
aaa12
```

ngx.req.get_uri_args 的用法如下：

```txt
args, err = ngx.req.get_uri_args()，用于获取客户端请求 uri 中的参数。

它的说明文档为：https://github.com/openresty/lua-nginx-module#ngxreqget_uri_args
```

其中，ngx.print() 函数的功能是将参数作为响应体返回给调用的客户端，未返回响应头时，首先返回响应头，再返回响应体。

还有一个 ngx.say() 方法，它与 ngx.print() 方法的功能一样，但是 ngx.say() 最后会返回一个换行符。

上面的方法 ngx.req.get_uri_args() 是接收 get 请求的参数，接收 post 请求参数的配置为：

```nginx
events {
    worker_connections 1024;
}
http {
    server {
        listen 8080;

        location /test {
            default_type text/html;
            content_by_lua_block {
                ngx.req.read_body();
                local params = ngx.req.get_post_args();
                ngx.say(params.name);
                ngx.say(params.age);
           }
        }

    }
}
```

在终端执行以下命令：

```txt
curl http://localhost:8080/test -X POST -d 'name=aaa&age=12'
```

得到的结果为：

```txt
aaa
12
```

接收 post 请求使用的是 ngx.req.get_post_args()。，在使用该方法之前，需要调用方法：ngx.req.read_body()。ngx.req.read_body() 的作用是：在不阻塞 nginx event loop 的请求下，异步读取客户端的请求体数据。



## 内部调用

在 openresty 中，可以进行内部 location 之间的调用，配置例子如下：

conf/inner.conf：

```nginx
events {
    worker_connections 1024;
}
http {
    server {
        listen 8080;

        location /test {
            default_type text/html;
            content_by_lua '
                local res = ngx.location.capture(
                    "/getname", {args={name=ngx.var["arg_name"], age=18}}
                )
                ngx.say(ngx.var["arg_name"]);
                ngx.say("status:", res.status, " resp:", res.body)
            ';
        }

        location /getname {
            content_by_lua '
                local params = ngx.req.get_uri_args();
                ngx.print(params.name);
            ';
        }
    }
}
```

上面的例子中，当访问 http://localhost:8080/test/?name=xxx 时，会打印出以下内容：

```txt
xxx
status:200 resp:aaa
```

配置文件中，ngx.location.capture 的作用如下：

```txt
功能：在 nginx 内部调用其它 location 块
语法：res = ngx.location.capture(uri, options)
上面的配置文件中，它的作用域是 content_by_lua 开头的模块

ngx.location.capture 会返回一个 lua table 对象，有以下4个属性：res.status, res.header, res.body, and res.truncated

ngx.location.capture 的说明文档为：https://github.com/openresty/lua-nginx-module#ngxlocationcapture
```



## 指定lua脚本文件

在 openresty 的 location 中，可以指定 lua 脚本来执行，配置例子为：

```nginx
events {
    worker_connections 1024;
}
http {
    server {
        listen 8080;

        location = /test {
            default_type text/html;
	
            content_by_lua_file /usr/local/openresty/lua_code/test.lua;
        }
    }
}
```

上面的例子中，使用 content_by_lua_file 来指定请求进来时需要运行的 lua 脚本文件。

test.lua 的内容为：

```txt
ngx.say("lua test file");
```

在终端执行  curl http://localhost:8080/test 后，会返回 lua test file。

## 连接redis

openresty 中通过 lua 连接 redis 的例子如下：redis.lua：

```txt
-- redis use

local resty_redis = require('resty.redis')
local redis = resty_redis:new()

redis:settimeout(1000)

local ok, err = redis:connect('127.0.0.1', 6379)
if not ok then
        ngx.say(err)
        ngx.eof()
end

-- 连接 redis 后进行密码验证
local res, err = redis:auth('wyzane')
if not res then
        ngx.say("failed to auth:", err)
        return
end


local res, err = redis:get("name")
if not res then
        ngx.say("failed to get name:", err)
        return
end


ngx.say(res)

local ok, err = redis:set_keepalive(10000, 100)
if not ok then
        ngx.say("failed to set keeplive:", err)
        return
end
```

redis.conf：

```txt
events {
    worker_connections 1024;
}
http {
    server {
        listen 8080;

        location = /test {
            default_type text/html;
	
            content_by_lua_file /home/wyzane/lua_code/redis.lua;
        }
    }
}
```

## 连接mysql

openresty 中使用 lua 连接 mysql 的例子如下：mysql.lua：

```lua
-- mysql use

local resty_mysql = require("resty.mysql")
local cjson = require("cjson")

local db, err = resty_mysql:new()

if not db then
        ngx.say('init mysql failed', err)
        return
end

db:set_timeout(1000)


local ok, err, errcode, sqlstate = db:connect({
    host = "127.0.0.1",
    post = 3306,
    database = "test_koa",
    user = "root",
    password = "wyzane",
})

if not ok then
        ngx.say("failed to connect mysql:", err)
        return
end


res, err, errcode, sqlstate = db:query("select * from tb_user")
if not res then
        nga.say("query failed:", err)
        return
end

db:set_keepalive(0, 100)
ngx.say(cjson.encode(res))
```

mysql.conf：

```nginx
events {
    worker_connections 1024;
}
http {
    server {
        listen 8080;

        location = /test {
            default_type text/html;
	
            content_by_lua_file /home/wyzane/lua_code/mysql.lua;
        }
    }
}
```





Openresty 官网：

```txt
http://openresty.org/cn/
```

Openresty的 github 地址为：

```txt
https://github.com/openresty
```

Openresty最佳实践：

```txt
https://www.kancloud.cn/allanyu/openresty-best-practices/82658
```

