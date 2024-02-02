---
title: koa2中cookie和session的使用
date: 2020-06-07 11:09:40
tags:
categories: Nodejs
---

cookie 和 session 都是保存用户状态的机制，今天介绍下它们在 koa 中的使用。

<!--more-->

# cookie

## cookie的使用

cookie可以记录用户的状态信息，在koa2中设置 cookie 的方法如下：

```txt
ctx.cookies.set(key, value, [options])
```

其中 options 中的常用参数如下：

```txt
maxAge: cookie过期时间
expires：cookie的过期日期
path: cookie路径，默认是 /
domain: cookie的域名
secure: 安全 cookie ，默认为 false,设置成true时表示只有https可以访问
httpOnly： true-表示只有服务器端可以访问，默认true(不能在浏览器中使用js访问)
overwrite: 是否覆盖之前设置的同名 cookie，默认false
```



获取cookie中的数据：

```txt
ctx.cookies.get(key)
```



demo.js：

```js
const Koa = require('koa');
const Router = require('koa-router');

const app = new Koa();
const router = new Router();


router.get('/', async (ctx, next)=>{
    ctx.cookies.set(
        'userinfo', 'hello cookies', {
            domain: '127.0.0.1',  // 设置访问域名
            path: '/get',   // 设置允许访问cookie的路由，其他路由则不能访问
            maxAge: 60 * 60  * 100,  // 设置超时时间
            expires: new Date('2020-07-04'), 
            httpOnly: false,  // 是否仅仅允许htpp请求
            overwrite: false,  // 是否允许重写
        }
    );
    ctx.body = 'cookies is ok';
});
router.get('/get', async (ctx, next)=>{
    let userinfo = ctx.cookies.get('userinfo');
    console.log(userinfo);
    ctx.body = userinfo;
});

app.use(router.routes()).use(router.allowedMethods());

app.listen(8095, ()=>{
    console.log('server starting ...');
});
```

当使用下面的url访问服务时，会分别设置cookie和获取cookie：

```txt
http://127.0.0.1:8095/
http://127.0.0.1:8095/get
```

设置 cookie 后，可以在浏览器的 Application 里面看到信息。

## cookie中设置中文

使用 ctx.cookies.set() 设置中文cookie值时，会抛出异常：

```txt
TypeError: argument value is invalid
```

此时，可以先将中文转换成 base64 编码，再把编码后的中文设置到 cookie 中即可，取 cookie 数据的时候需要解码。

```js
const Koa = require('koa');
const Router = require('koa-router');

const app = new Koa();
const router = new Router();


router.get('/', async (ctx, next)=>{
    let userinfo = new Buffer('中文cookie来啦').toString('base64');
    ctx.cookies.set(
        'userinfo', userinfo, {
            domain: '127.0.0.1',  // 设置域名
            path: '/get',   // 设置访问路径
            maxAge: 60 * 60  * 100,  // 设置超时时间
            expires: new Date('2020-07-04'), 
            httpOnly: false,  // 是否仅仅允许htpp请求
            overwrite: false,  // 是否允许重写
        }
    );
    ctx.body = 'cookies is ok';
});
router.get('/get', async (ctx, next)=>{
    let userinfo = ctx.cookies.get('userinfo');
    let info = new Buffer(userinfo, 'base64').toString();
    console.log(info);
    ctx.body = info;
});

app.use(router.routes()).use(router.allowedMethods());

app.listen(8095, ()=>{
    console.log('server starting ...');
});
```

# session

session是保存在服务端的一种记录用户状态信息的机制（cookie保存在客户端）。下面看一下它在 koa 中的使用。

## session的使用

koa 中使用 session 需要安装 koa-session 模块：

```txt
npm install --save koa-session
```

使用例子如下：

```js
const Koa = require('koa');
const Session = require('koa-session');
const Router = require('koa-router');

const app = new Koa();
const router = new Router();

// 配置 session 中间件
app.keys = ['some secret hurr'];
 
let CONFIG = {
  key: 'koa.sess', // 对应的cookie名称
  maxAge: 86400000,  // session过期时间(单位:ms)
  autoCommit: true, // 自动提交headers，默认true
  overwrite: true, // 覆盖之前key同名的session值，默认true
  httpOnly: true, // true-表示只有服务器端可以访问，默认true(不能在浏览器中使用js访问)
  signed: true, // 是否使用签名，默认true
  rolling: false, // 每次访问都重设过期时间 maxAge，默认false
  renew: true // 当session快过期时自动更新
};
 
router.get('/', async (ctx, next)=>{
    // 设置 session
    ctx.session.name = 'koa session'
    ctx.body = 'session is ok';
});
router.get('/get', async (ctx, next)=>{
    // 获取 session
    let name = ctx.session.name;
    console.log(name);
    ctx.body = name;
});

app.use(Session(CONFIG, app));

app.use(router.routes()).use(router.allowedMethods());

app.listen(8095, ()=>{
    console.log('server starting ...');
});
```



cookie 与 session 的使用就介绍到这里，感兴趣的小伙伴可以深入研究一下。