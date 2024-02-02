---
title: Koa2的简单使用
date: 2020-05-17 10:03:41
tags:
categories: Nodejs
---



Koa2 是一个基于 Nodejs 的web 服务端框架，它是一个插件式框架，很多功能都可以i通过安装插件的方式扩展。下面介绍下它的简单使用。

<!--more-->

## 开发环境搭建

首先要安装 node 和 npm 环境，然后再执行以下操作：

```txt
1. 初始化项目（生成package.json文件）
	npm init -y
2. 安装koa
	npm install --save koa （--save表示安装到生产环境 --save-dev表示安装到开发环境）
3. 运行koa服务
	node index.js
```



环境搭建完成后，可以写一个简单的demo：

```js
// demo.js
// 引入Koa
const Koa = require('koa');

// 实例化 koa
const app = new Koa();

// ctx: request和response对象都保存在ctx里面
// => 箭头函数
app.use(async(ctx) => {
    ctx.body = 'hello koa'
})

// 设置监听端口
app.listen(8090);

console.log('koa server starting ...');

/*
或者可以这样写：
app.listen(8090, ()=>{
    console.log('koa server starting ...');
});
*/
```

写完后，执行 node demo.js，然后再浏览器上输入 localhost:8090，可以看到，返回了 "hello koa"。



## 定义get请求

```js
// query：返回的是格式化好的参数对象
// querystring:  返回的是请求字符串

const Koa = require('koa');
const app = new Koa();

app.use(async(ctx)=>{
    // 请求url
    let url = ctx.url;
    // 获取request对象，并通过request对象接收请求信息
    let request = ctx.request;
    let query = request.query;
    let queryString = request.querystring;
    
    // 从上下文ctx中直接获取请求信息
    // let ctx_query = ctx.query;
    // let ctx_querystring = ctx.querystring;

    ctx.body = {
        url,
        query,
        queryString
    }
});

app.listen(8090, ()=>{
    console.log('koa server starting ...');
});
```

上面的服务启动后，使用 postman 访问，结果如下：

```txt
postman 访问的url为：localhost:8090/?name=wyzane&age=18
postman 接收到的结果为：
{
    "url": "/?name=wyzane&age=18",
    "query": {
        "name": "wyzane",
        "age": "18"
    },
    "queryString": "name=wyzane&age=18"
}
```



##  定义post请求

```js
const Koa = require('koa');
const app = new Koa();
app.use(async(ctx)=>{
    if(ctx.url==='/' && ctx.method==='GET'){
        // 显示表单页面
        let html = `
            <h1>Koa2 Class</h1>
            <form method="POST" action="/">
                <p>username:</p> <input name="userName" /><br/>
                <p>age:</p> <input name="userAge" /><br/>
                <p>website:</p> <input name="website" /><br/>
                <button type="submit">sibmit</submit>
            </form>
        `;
        ctx.body = html;
    }else if(ctx.url==='/' && ctx.method==='POST'){
        let postParams = await parse(ctx);
        ctx.body = postParams;
    }else{
        ctx.body = '<h1>404!</h1>';
    }
});

function parse(ctx){
    return new Promise((resolve, reject)=>{
        try{
            let postParams = '';
            ctx.req.addListener('data', (data)=>{
                postParams += data;
            })
            ctx.req.on('end', function(){
                let postData = parseQuery(postParams);
                resolve(postData);
            })
        }catch(error){
            reject(error);
        }
    })
}

function parseQuery(queryStr){
    /* 解析 post 请求参数 */
    let queryData = {};
    let queryStrList = queryStr.split('&');
    console.log(queryStrList);
    for(let itemStr of queryStrList){
        let item = itemStr.split('=');
        queryData[item[0]] = decodeURIComponent(item[1]);
    }
    return queryData;
}

app.listen(8091, ()=>{
    console.log('server starting ...');
});
```

```txt
ctx.request与ctx.req的区别
ctx.request: 是koa2中的context经过封装的请求对象，它用起来更直观和简单
ctx.req: 是context提供的nodejs原生http请求对象


解析Post请求的步骤
1. 解析上下文ctx中的原生nodejs对象req
2. 将post表单数据解析成query string字符串（例如user=wyzane&age=18）
3. 将字符串转换成json格式
```

上面的代码中，解析 post 请求参数时，是通过直接解析字符串的方式，对于处理 post 请求，下面还会介绍一些中间件用来处理 post 请求的参数。



## 路由的使用

路由是服务端开发很重要的一个方面，Koa2 中我们可以使用 koa-router 插件来实现路由。

安装：

```txt
npm install --save koa-router
```

下面看一个简单的例子：

```js
const Koa = require('koa');
const Router = require('koa-router');

const app = new Koa();

const router = new Router()

// 接收get请求
router.get('/', (ctx, next)=>{
    ctx.body = 'hello wyzane'
});
router.get('/get', (ctx, next)=>{
    ctx.body = 'hello get'
});

// routes()表示装载路由，allowedMethods()表示只接收某种方法，不接收其他方法
app.use(router.routes()).use(router.allowedMethods());

app.listen(8093, ()=>{
    console.log('server starting ...');
});
```

启动上面的服务后，在浏览器上请求 http://localhost:8093/ 或者 http://localhost:8093/get，回得到不同的响应信息。



我们还可以给路由加上前缀：

```js
const Koa = require('koa');
const Router = require('koa-router');

const app = new Koa();

// 实例化路由时指定前缀
const router = new Router({
    prefix: '/wyzane'
});

// 接收gei请求
router.get('/', (ctx, next)=>{
    ctx.body = 'hello wyzane'
});
router.get('/get', (ctx, next)=>{
    ctx.body = 'hello get'
});

// routes()表示装载路由，allowedMethods()表示只接收某种方法，不接收其他方法
app.use(router.routes()).use(router.allowedMethods());

app.listen(8093, ()=>{
    console.log('server starting ...');
});
```

启动上面的服务后，在浏览器上请求 http://localhost:8093/wyzane 或者 http://localhost:8093/wyzane/get，回得到不同的响应信息。



除了上面给路由添加前缀，还可以定义父子路由：

```js
const Koa = require('koa');
const Router = require('koa-router');

const app = new Koa();

// 子路由
const home = new Router();
home.get('/info', async(ctx)=>{
    ctx.body = 'hello info';
}).get('/todo', async(ctx)=>{
    ctx.body = 'hello todo';
})


const interest = new Router();
interest.get('/basket', async(ctx)=>{
    ctx.body = 'hello basket';
}).get('/soccer', async(ctx)=>{
    ctx.body = 'hello soccer';
})

// 父级路由
const router = new Router();

// 装载子路由到父路由
router.use('/home', home.routes(), home.allowedMethods());
router.use('/interest', interest.routes(), interest.allowedMethods());


// routes()表示装载路由，allowedMethods()表示只接收某种方法，不接收其他方法
app.use(router.routes()).use(router.allowedMethods());

app.listen(8093, ()=>{
    console.log('server starting ...');
});
```

上面的代码中，有以下路由

```txt
http://localhost:8093/home/info
http://localhost:8093/home/todo
http://localhost:8093/interest/basket
http://localhost:8093/interest/soccer
```



## 请求体解析

解析请求体有很多插件，下面介绍几种。

```txt
koa-body：可以解析 x-www-form-urlencoded和json类型的请求体
	安装：npm install --save koa-body

koa-bodyparse：可以解析 x-www-form-urlencoded和json类型的请求体
	安装：npm install --save koa-bodyparse

multy: 上面的中间件都不能解析form-data类型的请求体，该中间件则可以解析form-data类型的请求体(只能解析该类型)
	安装：npm install --save multy
```



下面分别看以下它们使用的例子。

koa-body 的使用：

```js
const Koa = require('koa');
const bodyParser = require('koa-body');

const app = new Koa();

app.use(bodyParser());

app.use(async(ctx) => {
    console.log(ctx.request.body);
    ctx.body = ctx.request.body;
})

app.listen(8099, ()=>{
    console.log('server starting ...');
});
```



koa-bodyparser 的使用：

```js
const Koa = require('koa');
const bodyParser = require('koa-bodyparser');

const app = new Koa();

app.use(bodyParser());

app.use(async(ctx) => {
    console.log(ctx.request.body);
    ctx.body = ctx.request.body;
})

app.listen(8099, ()=>{
    console.log('server starting ...');
});
```



multy 的使用：

```js
const Koa = require('koa');
const Multy = require('multy');

const app = new Koa();

app.use(Multy());

app.use(async(ctx) => {
    console.log(ctx.request.body);
    ctx.body = ctx.request.body;
})

app.listen(8099, ()=>{
    console.log('server starting ...');
});
```



查看请求体信息时，都可以在 ctx.request.body 中查看。



Koa2 是一个小巧、灵活、强大的 web 服务端框架，对于经常写 python 或者 java 的小伙伴来说，它的代码风格、开发方式都会吸引我们去探讨一番。