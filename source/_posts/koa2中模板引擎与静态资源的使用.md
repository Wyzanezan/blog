---
title: koa2中模板引擎与静态资源的使用
date: 2020-05-31 16:01:49
tags:
categories: Nodejs
---

今天介绍下 koa2 中模板引擎与静态资源的使用。

<!--more-->

# 模板引擎

适用于 koa2 的模板引擎有很多，比如ejs，ant-template，jade，今天介绍下前两个模板引擎的使用。

## ejs

官方文档：

```txt
https://ejs.bootcss.com/
```



使用esj模板引擎需要安装如下中间件：

```txt
安装模板中间件：npm install --save koa-views
安装ejs模板引擎：cnpm install --save ejs
```

### 渲染单个数据

渲染单个数据的例子如下。

index.js

```js
const Koa = require('koa');
const views = require('koa-views');

// node原生模块
const path = require('path');

const app = new Koa();

// 配置模板引擎方式一：模板为ejs后缀
// __dirname表示项目根目录, extension: 'ejs'表示指定模板引擎
// app.use(views(path.join(__dirname, './view/'), {
//     extension: 'ejs'
// }));

// 配置模板引擎方式二：模板为html后缀
app.use(views(path.join(__dirname, './view'), {
    map: {
        html: 'ejs'
    }
}));

app.use(async(ctx)=>{
    let title = 'hello wyzane';
    // 渲染 index 模板，对应的是view目录下的index.ejs或者index.html文件
    await ctx.render('index', {title});
})

app.listen(8090, ()=>{
    console.log('server starting ...');
})
```

view/index.ejs：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <!-- 从后端接收title的值并显示 -->
    <title><%= title %></title>
</head>
<body>
    <h1><%= title %></h1>
    <p>EJS welcome to <%= title %></p>
</body>
</html>
```

view/index.html：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <!-- 从后端接收title的值并显示 -->
    <title><%= title %></title>
</head>
<body>
    <h1><%= title %></h1>
    <p>EJS welcome to <%= title %></p>
</body>
</html>
```

### 渲染for循环

index.js：

```js
const Koa = require('koa');
const views = require('koa-views');

// node原生模块
const path = require('path');

const app = new Koa();

// 配置模板引擎方式一：模板为ejs后缀
// __dirname表示项目根目录, extension: 'ejs'表示指定模板引擎
app.use(views(path.join(__dirname, './view/'), {
    extension: 'ejs'
}));

// 配置模板引擎方式二：模板为html后缀
// app.use(views(path.join(__dirname, './view'), {
//     map: {
//         html: 'ejs'
//     }
// }))

app.use(async(ctx)=>{
    let names = ['喜洋洋', '吉吉国王', '光头强'];
    await ctx.render('news', {names});
})

app.listen(8090, ()=>{
    console.log('server starting ...');
})
```

view/index.ejs：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <!-- 从后端接收title的值并显示 -->
</head>
<body>
    <%for(var i=0;i<names.length;i++){%>
        <li><%= names[i]%></li>
    <%}%>
</body>
</html>
```

### 渲染if语句

index.js：

```js
const Koa = require('koa');
const views = require('koa-views');

// node原生模块
const path = require('path');

const app = new Koa();

app.use(views(path.join(__dirname, './view/'), {
    extension: 'ejs'
}));

app.use(async(ctx)=>{
    let num = 4;
    await ctx.render('news', {num});
})

app.listen(8090, ()=>{
    console.log('server starting ...');
})
```

view/index.ejs：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <!-- 从后端接收title的值并显示 -->
</head>
<body>
    <%if (num>10) {%>
        大于10
    <%}else{%>
        小于10
    <%}%>
</body>
</html>
```



## ant-template

### 官方文档

```txt
https://aui.github.io/art-template/
```

下面介绍一些 art-template 的基本语法，感兴趣的同学可以学习官方文档。

### 安装

```txt
npm install --save art-template
npm install --save koa-art-template
```

### 使用

index.js：

```js
const Koa = require('koa');
const Router = require('koa-router');
const render = require('koa-art-template');
const path = require('path');

const app = new Koa();
const router = new Router();

// 配置art-template模板引擎
render(app, {
    root: path.join(__dirname, 'view'),  // 模板位置
    extends: '.art',  // 模板后缀名
    debug: process.env.NODE_ENV !== 'production'  // 是否开启调试模式
})

router.get("/", async (ctx, next)=>{
    // ctx.body = "首页";
    await ctx.render('index');
})

router.get("/news", async (ctx, next)=>{
    ctx.body = "新闻";
})

app.use(router.routes());
app.use(router.allowedMethods());


app.listen(8099, ()=>{
    console.log("start ...");
})
```

view/index.art：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title></title>
</head>
<body>
    <h1>art-template模板引擎使用</h1>
</body>
</html>
```

art-template种支持两种语法：原始语法和标准语法。原始语法与ejs的语法类似。下面只介绍标准语法。

### 渲染单个数据

index.js：

```js
const Koa = require('koa');
const Router = require('koa-router');
const render = require('koa-art-template');
const path = require('path');

const app = new Koa();
const router = new Router();

// 配置art-template模板引擎
render(app, {
    root: path.join(__dirname, 'view'),  // 模板位置
    extends: '.art',  // 模板后缀名
    debug: process.env.NODE_ENV !== 'production'  // 是否开启调试模式
})

router.get("/", async (ctx, next)=>{
    // ctx.body = "首页";
    let title = '首页';
    // 把数据传入模板中
    await ctx.render('index', {title});
})

router.get("/news", async (ctx, next)=>{
    ctx.body = "新闻";
})

app.use(router.routes());
app.use(router.allowedMethods());


app.listen(8099, ()=>{
    console.log("start ...");
})
```

view/index.art：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title></title>
</head>
<body>
    <h1>art-template模板引擎使用</h1>
    <!-- 接收传入的数据 -->
    <h2>{{ title }}</h2>
</body>
</html>
```

### 渲染for循环

```js
const Koa = require('koa');
const Router = require('koa-router');
const render = require('koa-art-template');
const path = require('path');

const app = new Koa();
const router = new Router();

// 配置art-template模板引擎
render(app, {
    root: path.join(__dirname, 'view'),  // 模板位置
    extends: '.art',  // 模板后缀名
    debug: process.env.NODE_ENV !== 'production'  // 是否开启调试模式
})

router.get("/", async (ctx, next)=>{
    // ctx.body = "首页";
    let names = ['光头强', '喜洋洋', '吉吉国王'];
    await ctx.render('index', {names});
})

router.get("/news", async (ctx, next)=>{
    ctx.body = "新闻";
})

app.use(router.routes());
app.use(router.allowedMethods());


app.listen(8099, ()=>{
    console.log("start ...");
})
```

view/index.art：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title></title>
</head>
<body>
    <h1>art-template模板引擎使用</h1>
    <!-- 渲染for循环 -->
    {{each names}}
    {{$index}} {{$value}}
    {{/each}}
</body>
</html>
```

### 渲染if

index.js：

```js
const Koa = require('koa');
const Router = require('koa-router');
const render = require('koa-art-template');
const path = require('path');

const app = new Koa();
const router = new Router();

// 配置art-template模板引擎
render(app, {
    root: path.join(__dirname, 'view'),  // 模板位置
    extends: '.art',  // 模板后缀名
    debug: process.env.NODE_ENV !== 'production'  // 是否开启调试模式
})

router.get("/", async (ctx, next)=>{
    // ctx.body = "首页";
    let num = 10;
    await ctx.render('index', {
        num: num
    });
})

router.get("/news", async (ctx, next)=>{
    ctx.body = "新闻";
})

app.use(router.routes());
app.use(router.allowedMethods());


app.listen(8099, ()=>{
    console.log("start ...");
})
```

view/index/art：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title></title>
</head>
<body>
    <h1>art-template模板引擎使用</h1>
    {{ if num > 10}}
        <h2>大于</h2>
    {{ else }}
        <h2>小于</h2>   
    {{ /if }}
</body>
</html>
```

### 子模板

在一个模板中引入另外一个模板

index.art：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title></title>
</head>
<body>
    <h1>art-template模板引擎使用</h1>
    {{ if num > 10}}
        <h2>大于</h2>
    {{ else }}
        <h2>小于</h2>   
    {{ /if }}

    <!-- 引入子模板 -->
    {{ include './footer.art'}}
</body>
</html>
```

footer.art：

```html
<h1>Footer</h1>
```



# 静态资源的访问

静态资源在项目中非常重要，在koa2中，需要使用中间件 koa-static 来访问静态资源。

## 安装中间件

```txt
cnpm install --save koa-static
```

## 在模板中使用静态资源

模板以 ejs 为例

view/index.ejs：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <!-- 从后端接收title的值并显示 -->
    <title><%= title %></title>
    <!-- 引入静态文件 -->
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <h1 class='title'><%= title %></h1>
    <p>EJS welcome to <%= title %></p>
    <!-- 引入图片静态资源 -->
    <img src="koa.jpg" alt="">
</body>
</html>
```

static/style.css：

```css
.title {
    width: 100%;
    color: red;
    text-align: center;
}
```

index.js：

```js
const Koa = require('koa');
const static = require('koa-static');
const views = require('koa-views');

const path = require('path');

const app = new Koa();

const viewPath = './view/'
const staticPath = './static';

// 指定模板引擎
app.use(views(path.join(__dirname, viewPath), {
    extension: 'ejs'
}));

// 指定静态资源目录
// app.use(static(path.join(__dirname, staticPath)));  // 方式一
app.use(static(staticPath));  // 方式二

app.use(async(ctx)=>{
    let title = "你好"
    await ctx.render('index', {title});
});

app.listen(8091, ()=>{
    console.log('server starting ...');
})
```



以上就是 koa2 中模板引擎的使用和静态资源的访问方式，感兴趣的同学可以深入学习。