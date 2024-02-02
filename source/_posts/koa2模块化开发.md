---
title: koa2模块化开发
date: 2020-06-12 23:40:16
tags:
categories: Nodejs
---

模块化开发可以使我们的项目结构更加清晰，协作开发更加方便，后期维护更加容易。下面介绍下 koa2 中的模块化开发。

<!--more-->

# koa-generator

模块化开发就是将接口、视图等归类，按照业务功能分别存放在不同的目录下，koa2 中我们可以使用脚手架 koa-generator 来生成项目结构，它的使用步骤如下：

```txt
1. 安装 npm install -g koa-generator
2. 创建项目 koa koa_demo
3. cd koa_demo
4. 安装依赖 npm install
5. 启动项目 npm start
```

使用 koa-generator 生成的项目目录结构大致如下：

```txt
├─bin
│  ├─www
├─node_modules
├─public
│  ├─images
│  ├─javascripts
│  └─stylesheets
├─routes
│  ├─admin.js
│  ├─users.js
│  ├─goods.js
└─views
├─app.js
└─package.json
```

上面自动生成的代码中，将路由和视图模块化了，不同的业务场景路由放在不同的文件下。然后将路由文件引入到 app.js 中统一调用。



# 路由模块化例子

admin.js：

```js
const Router = require('koa-router');

const router = new Router();


router.get("/auth", (ctx)=>{
    ctx.body = '这是权限管理'
});

router.get("/base", (ctx)=>{
    ctx.body = '这是基本信息'
});


module.exports = router;
```

user.js：

```js
const Router = require('koa-router');

const router = new Router();


router.get("/list", (ctx)=>{
    ctx.body = '这是用户列表'
});

router.get("/detail", (ctx)=>{
    ctx.body = '这是用户详情'
});


module.exports = router;
```

app.js中的代码如下：

```js
const Koa = require('koa');
const Router = require('koa-router');

const app = new Koa();
const router = new Router();

// 引入子路由
const admin = require('./routes/admin.js');
const user = require('./routes/user.js');

// 配置子路由
router.use('/admin', admin.routes());
router.use('/user', user.routes());

app.use(router.routes()).use(router.allowedMethods());

app.listen(9091, ()=>{
    console.log('server start ...');
})
```



# 视图模块化

视图模块化中，我们使用 art-template 模板引擎为例子。

目录结构如下：

```txt
F:.
├─module
├─node_modules
├─public
│  ├─images
│  ├─javascripts
│  └─stylesheets
├─routes
│  ├─admin.js
│  ├─users.js
│  ├─goods.js
├─statics
└─views
    ├─goods
    	├─add.html
      	└─list.html
    └─user
      	├─add.html
      	│─detail.html
      	└─list.html
```



app.js：

```js
const Koa = require('koa');
const Router = require('koa-router');
const render = require('koa-art-template');
const path = require('path');

const app = new Koa();
const router = new Router();

// 配置art-template模板引擎
render(app, {
    root: path.join(__dirname, 'views'),  // 指定模板的路径
    extname: '.html',                     // 指定模板后缀
    debug: process.env.NODE_ENV != 'production'
})

// 引入子路由
const admin = require('./routes/admin.js');
const user = require('./routes/user.js');
const goods = require('./routes/goods.js');

router.use('/admin', admin.routes());
router.use('/user', user.routes());
router.use('/goods', goods.routes());

app.use(router.routes()).use(router.allowedMethods());

app.listen(9091, ()=>{
    console.log('server start ...');
})
```

user.js：

```js
const Router = require('koa-router');

const router = new Router();


router.get("/list", async (ctx)=>{
    // render()中的第一个参数指定模板的路径
    await ctx.render('user/list');
});

router.get("/detail", async (ctx)=>{
    await ctx.render('user/detail');
});

router.get("/add", async (ctx)=>{
    await ctx.render('user/add');
});


module.exports = router;
```

goods.js：

```js
const Router = require('koa-router');

const router = new Router();


router.get("/list", async (ctx)=>{
    await ctx.render('goods/list');
});

router.get("/detail", async (ctx)=>{
    ctx.body = '这是商品详情'
});

router.get("/add", async (ctx)=>{
    await ctx.render('goods/add');
});


module.exports = router;
```

上面就是模块化的一个例子。