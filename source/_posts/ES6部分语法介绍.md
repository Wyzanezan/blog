---
title: ES6部分语法介绍
date: 2020-05-17 08:52:38
tags:
categories: Nodejs
---



ES6是 ECMAScript 6 的简称，它是JavaScript语言的下一代标准，已经在2015年6月正式发布了。今天介绍下ES6中有哪些新语法。

<!--more-->

## 开发环境搭建

```txt
新建如下目录结构：
ES6test
	dist
	src
		index.js
	index.html
```

index.js中的内容如下：

```txt
let a = 1;
console.log(a);
```

在终端执行 npm init -y 后，会生成 package.json 文件，文件内容如下：

```json
{
  "name": "ES6test",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "", 
  "license": "ISC"
}
```



此时，我们还需要把此处的 ES6 语法的代码转换成 ES5 才能执行，所以需要一个转换的插件（例如 webpack）。这里我们使用 Babel 进行转换。全局安装 Babel：

```txt
npm install -g babel-cli

此外，还需要安装以下插件
npm install --save-dev babel-preset-es2015 babel-cli
```

安装完成后，package.json 文件中多了以下内容：

```json
"devDependencies": {
   "babel-cli": "^6.26.0",
   "babel-preset-es2015": "^6.24.1"
}
```



在根目录下创建 .babelrc 文件，并输入一下内容：

```json
{
    "presets": [
        "es2015"
    ],
    "plugins": []
}
```



最后就可以执行转换命令把  es6 的代码转换成 es5 的代码：

```txt
babel src/index.js -o dist/index.js
```

执行完成后，dist/index.js 文件的内容为（转换成了es5的写法）：

```js
"use strict";

var a = 1;
console.log(a);
```



可以配置 npm 命令来简化转换操作，修改 package.json 文件，添加如下内容：

```json
"scripts": {
	"build": "babel src/index.js -o dist/index.js"
},
```

此时，执行 npm run build 命令就可以进行打包。



## 变量声明

```txt
声明方式：
var 全局声明
let 局部声明
const 常量
```

例子：

```js
var a = "wyzane";

{
    var a = "wz";
}

console.log(a);  // wz
```

```js
var a = "wyzane";

{
    let a = "wz";
}

console.log(a);  // wyzane
```

```js
let a = "wyzane";

{
    let a = "wz";
}

console.log(a);  // wyzane
```

```js
let a = "wyzane";

{
    a = "wz";
}

console.log(a);  // wz
```

```js
{
    let a = "wz";
}

console.log(a);  // 抛异常 a is not defined
```



使用 var 时，数据容易被污染

```js
for (var i=0; i<10; i++) {
    console.log("inner: " + i);
}
console.log("outer: " + i);

/**输出如下
inner: 0
index.js:4 inner: 1
index.js:4 inner: 2
index.js:4 inner: 3
index.js:4 inner: 4
index.js:4 inner: 5
index.js:4 inner: 6
index.js:4 inner: 7
index.js:4 inner: 8
index.js:4 inner: 9
index.js:6 outer: 10

此时循环体外的变量容易被污染
**/
```

使用 let 时：

```js
for (let i=0; i<10; i++) {
    console.log("inner: " + i);
}
console.log("outer: " + i);

/**输出为：
inner: 0
index.js:4 inner: 1
index.js:4 inner: 2
index.js:4 inner: 3
index.js:4 inner: 4
index.js:4 inner: 5
index.js:4 inner: 6
index.js:4 inner: 7
index.js:4 inner: 8
index.js:4 inner: 9
index.js:6 Uncaught ReferenceError: i is not defined
    at index.js:6
**/
```



## 变量解构赋值

数组解构：

```js
let [a, b, c] = [1, 2, 3];
console.log(a);

解构的时候定义默认值：
let [a, b, c=5] = [1, 2];
console.log(a);  // 1
console.log(c);  // 5

let [a=0, b, c] = [1, 2];
console.log(a); // 1
console.log(c); // undefined


// undefined 与 null 在解构时的区别
let [a, b, c=6] = [1, 2, undefined];
console.log(a); // 1
console.log(c); // 6

let [a, b, c=6] = [1, 2, null];
console.log(a); // 1
console.log(c); // null
```

对象解构：

```js
对象的解构是根据属性名称进行对应的
let {name1, name2} = {'name2': 'wyzane', 'name1': 'wz'};
console.log(name1); // wz
console.log(name2); // wyzane


一个小坑：
let foo;
{foo} = {foo: 'wyzane'};
console.log(foo);
上面的解构方式会报错，应该使用下面的方式：
let foo;
({foo} = {foo: 'wyzane'});
console.log(foo);
```

字符串解构：

```js
let [a, b, c] = 'nihao';
console.log(a); // n
console.log(c); // h
```



## 对象扩展运算符

对象扩展运算符使用 ... 表示

```js
// 对象扩展运算符用在函数不确定参数个数的时候，例子如下：
function test(...args) {
    console.log(args[0]);  // 1
    console.log(args[1]);  // 2
    console.log(args[2]);  // 3
    console.log(args[3]);  // undefined
}

test(1, 2, 3);


let a1 = ['1', '2', '3'];
let a2 = a1;
console.log(a1); // ['1', '2', '3']
a2.push('4');
// a1的值已改变
console.log(a1); // ['1', '2', '3', '4']
// 下面使用 ... 解决上面的问题
let a1 = ['1', '2', '3'];
let a2 = [...a1];
console.log(a1);  // ['1', '2', '3']
a2.push('4');
console.log(a1);  // ['1', '2', '3']
```



## rest运算符

与对象扩展运算符类似

```js
// 当函数参数前几个确定，后面的不确定时，可以使用rest运算符
function test(f, ...args) {
    console.log(args.length);  // 6
    console.log(args[0]);  // 1
    
    for (let i=0; i<args.length; i++) {
        console.log(args[i]);
    }
    
    // es6中的 for ... of ... 循环
    for (let val of args) {
        console.log(val);
    }
}

test(0, 1, 2, 3, 4, 5, 6);
```



## 字符串操作

创建一个node项目的步骤：

```txt
1. 执行 npm init 初始化项目
2. 执行 cnpm install -g live-server 安装live-server作为前端服务器
```



字符串模板的使用：

```js
let name = 'wyuzane';
// es5字符串拼接写法
let info = '你们好呀，我是' + name + '，欢迎来到学习园地';
// es6语法，字符串中可以含有html标签
let info = `你们好呀，我是${name}，欢迎来到学习园地`;
let info = `你们好呀，我是${name}，<br>欢迎来到学习园地`;
document.write(info);


// 字符串模板支持简单运算：
let a = 1;
let b = 2;
let c = `${a+b}`;
document.write(c);


// 字符串查找操作：
let name = 'wyzane';
let info = '你们好呀，我是wyzane，欢迎来到学习园地';
// es5中查找字符串
document.write(info.indexOf(name)); // 7
// es6中判断子字符串是否存在
document.write(info.includes(name)); // true
// 还有 startsWith()、endsWith() 等方法


// 字符串复制操作：
document.write('wyzane | '.repeat(3));  // wyzane | wyzane | wyzane |
```





## 数字操作

```js
// 二进制声明
let binary = 0B010101;
console.log(binary);  // 21

// 八进制声明
let octal = 0O666;
console.log(octal);  // 438

// 数字判断，判断变量的值是否是数字
let a = 11/4;
console.log(Number.isFinite(a));  // true
console.log(Number.isFinite(NaN)); // false
console.log(Number.isFinite('wyzane'));  // false

// 判断是否为 NaN
console.log(Number.isNaN(NaN));  // true
console.log(Number.isNaN('wyzane'));  // false

// 判断是否是整数
let a = 1.1;
let b = 2;
console.log(Number.isInteger(a));  // false
console.log(Number.isInteger(b));  // true

// 数字转换
let a = 2.531;
let b = 2;
console.log(Number.parseInt(a));  // 2
console.log(Number.parseFloat(b));  // 2


// es6中提供了一个最大安全整数
console.log(Number.MAX_SAFE_INTEGER);  // 9007199254740991  即 2的53次方减1
console.log(Math.pow(2, 53) - 1);  // 9007199254740991

// 最小安全整数
console.log(Number.MIN_SAFE_INTEGER);  // -9007199254740991

// 判断整数是否超过最大范围
console.log(Number.isSafeInteger(123));   // true
```



## 数组1

```js
// 将json数组转换成数组
// json数组
let json = {
    '0': 'wyzane1',
    '1': 'wyzane2',
    '2': 'wyzane3',
    length: 3
}
// from() 方法将json数组转换成数组
let arr = Array.from(json);
console.log(arr);  // ["wyzane1", "wyzane2", "wyzane3"]


// of() 方法
let arr = Array.of(3,4,5,6);
console.log(arr);  // [3, 4, 5, 6]
let arr = Array.of('qw', 'er', 'rt');
console.log(arr);  // ["qw", "er", "rt"]


// es5中将字符串转换为数组
let a = '[3, 4, 5, 6]';
console.log(eval(a));  // [3, 4, 5, 6]


// find() 实例方法，该方法的参数是一个匿名函数，查找到元素后将不再查找，直接返回
find(function(val, idx, arr) {
	return val > 5;
})

let arr = [1, 2, 3, 4, 5];
console.log(arr.find(function(val, index, arr){
    return val > 3;
}));  // 4

let arr = ['wyzane', 'nihao', 'hello'];
console.log(arr.find(function(val, index, arr){
    return val == 'hello';
}));  // hello

let arr = ['wyzane', 'nihao', 'hello'];
console.log(arr.find(function(val, index, arr){
    return val == 'he';
}));  // undefined
```



## 数组2

```js
// fill() 实例方法，用于数组元素的替换 fill(带替换元素, 起始位置，结束位置)
let arr = ['1', '2', '3', '4', '5'];
arr.fill('wyzane', 1, 4);  // 替换第2,3,4个元素为wyzane
console.log(arr);  //  ["1", "wyzane", "wyzane", "wyzane", "5"]

// 数组循环 for ... of ... 循环
let arr = ['1', '2', '3', '4', '5'];
// 数组元素
for (let val of arr) {
    console.log(val);   
}
// 输出下标
for (let val of arr.keys()) {
    console.log(val);   
}
// 输出下标和元素
for (let val of arr.entries()) {
    console.log(val);   
}
// 同时获取索引和值
for (let [idx, val] of arr.entries()) {
    console.log(val);
    console.log(idx);
}

// entries()方法
let arr = ['1', '2', '3', '4', '5'];
let arr2 = arr.entries()
console.log(arr2);  // Array Iterator {}
console.log(arr2.next().value); // [0, "1"]



// 数组遍历 forEach的使用
let json = ['hello', 'nihao', 'hihi']
json.forEach((idx, val) => console.log(idx, val));

// 数组遍历 filter的使用
let json = ['hello', 'nihao', 'hihi']
json.filter(x => console.log(x));  // 仅仅输出元素

// 数组遍历 some的使用
let json = ['hello', 'nihao', 'hihi']
json.some(x => console.log(x)); // 仅仅输出元素


// 数组遍历 map的使用 有替换效果
let json = ['hello', 'nihao', 'hihi']
console.log(json.map(x => 'web'));  // ["web", "web", "web"]


// 数组转换成字符串
let json = ['hello', 'nihao', 'hihi']
console.log(json.toString());  // hello,nihao,hihi

let json = ['hello', 'nihao', 'hihi']
console.log(json.join('|'));  // hello|nihao|hihi
```



## 函数的扩展

```js
// es5中定义函数
function add(a, b) {
    console.log(a + b);
}

add(1, 2);


// es6中给参数添加默认值
function add(a, b=4) {
    console.log(a + b);
}

add(1);


// 主动抛出异常
function add(a, b=4) {
    if (a==0) {
        throw new Error('a ia error');
    }
    console.log(a + b);
}

add(0);
/*
index.js:3 Uncaught Error: a ia error
    at add (index.js:3)
    at index.js:8
*/


// 获得函数需要传递的参数个数
function add(a, b=4) {
    console.log(a + b); 
}

function add2(a, b) {
    console.log(a + b);
}

console.log(add.length);  // 1
console.log(add2.length);  // 2



// 箭头函数
let add = (a, b=1) => a + b;
console.log(add(1));  // 2

let add = (a, b=1) => {
    console.log('wyzane');  // wyzane
    return a + b;
}
console.log(add(1));  // 2


// 对象的函数解构
let json = {
    a: 'wyzane1',
    b: 'wyzane2'
}

function func({a, b='nihao'}) {
    console.log(a, b);  // wyzane1 wyzane2
}

func(json);

// 数组解构
let json = ['nihao', 'hi', 'hello'];

function func([a, b, c]) {
    console.log(a, b, c);  // nihao hi hello
}

func(json);


let json = ['nihao', 'hi', 'hello'];

function func(a, b, c) {
    console.log(a, b, c);  // nihao hi hello
}

func(...json);



// in的使用
let json = {
    a: 'nihao',
    b: 'hi'
}

console.log('a' in json);  // true
```



## 对象

```js
// 对象的赋值
// es5中的赋值
let name = 'wyzane';
let age = 18;
let wz = {name: name, age: age};
console.log(wz);  // {name: "wyzane", age: 18}


// es6中允许直接使用变量赋值
let name = 'wyzane';
let age = 18;
let wz = {name, age};
console.log(wz);  // {name: "wyzane", age: 18}



// key值的构建， 动态的创建key值
let key = 'name';
let wz = {
    [key]: 'wyzane'
}
console.log(wz);  // {name: "wyzane"}



// 自定义对象方法
let wz = {
    add: function(a, b) {
        return a + b;
    }
}
console.log(wz.add(1, 2));  // 3



// is() 方法，用于比较两个对象
let obj1 = {name: 'wyzane'};
let obj2 = {name: 'wyzane'};

console.log(obj1.name === obj2.name);  // true
console.log(Object.is(obj1.name, obj2.name));  //true

console.log(+0 === -0);  // true
console.log(NaN === NaN);  //false

console.log(Object.is(+0, -0));  // false
console.log(Object.is(NaN, NaN));  //true


/*
===: 表示同值相等
is(): 严格相等
*/



// assign() 方法用于合并对象
let a = {name: 'wyzane'}
let b = {age: 18}
let c = {salary: 1}
let d = Object.assign(a, b, c);
console.log(d);  // {name: "wyzane", age: 18, salary: 1}
```



## symbol在对象中的作用

```js
let a = new String;
let b = new Array;
let c = new Number;
let d = new Boolean;
let e = new Object;

let f = Symbol();
console.log(typeof(f));  // symbol


let a = Symbol('wyzane');
console.log(a);  // Symbol(wyzane)
console.log(a.toString());  // Symbol(wyzane)


// symbol在对象中的使用
let name = Symbol();
let obj = {
    [name]: 'wyzane'
}
console.log(obj.name);   // undefined  key为Symbol对象时，不能使用.的方式访问key
console.log(obj[name]);  // wyzane
obj[name] = 'wz'
console.log(obj[name]);  // wz


// symbol的应用，起到隐藏是有属性的作用
let person = {name: 'wyzane', skill: 'web'}
let age = Symbol()
person[age] = 18
console.log(person);  // index.js:4 {name: "wyzane", skill: "web", Symbol(): 18}

for (let item in person) {
    console.log(person[item]);
}
/*
wyzane
web
*/
```



## Set与WeakSet

```js
// set类似于数组，但是不能有重复元素
let setArr = new Set(['wyzane1', 'wyzane2', 'wyzane']);
console.log(setArr);  // {"wyzane1", "wyzane2", "wyzane"}
setArr.add('nihao');  // 添加元素，数组中使用push添加元素
console.log(setArr);  //  {"wyzane1", "wyzane2", "wyzane", "nihao"}

console.log(setArr.has('wyzane'));  // 查找元素

setArr.delete('wyzane');  // 删除元素

setArr.clear();  // 清空元素


// for-of 输遍历出
for (let item of setArr) {
    console.log(item);
}

// foreach 遍历输出
setArr.forEach((val) => console.log(val));

// 获取set中元素的个数
console.log(setArr.size);
```

```js
// 由于set只能存放元素，不能存放对象，所以就出现了WeakSet。WeakSet可以存放对象
// WeakSet的使用
let setArr = new WeakSet();
let obj = {name: 'wyzane1', age: 13};
setArr.add(obj);
console.log(setArr);
```



## map数据结构

```js
// map中的key可以是任意类型
let json = {
    name: 'wyzane',
    age: 12
}

let map = new Map();
map.set(json, 'im');  // set(key, val): 向map中添加元素
map.set('salary', 13);
console.log(map);  // Map(2) {{…} => "im", "salary" => 13}

console.log(map.get('salary'));  // 取值

map.delete(json);  // delete(key): 删除指定key的元素

map.clear();  // 删除全部元素

map.size;  // 获取元素个数

map.has(json);  // has(key)：根据key查找元素
```



## proxy的使用

```js
// proxy 用于增强对象和函数，执行预处理操作
let obj = {
    add: function(val) {
        return val + 100;
    },
    name: 'wyzane'
}
console.log(obj.add(100));  // 200
console.log(obj.name);  // wyzane


// proxy-get方法，获取对象属性时使用
// 当需要在调用obj之前处理其他事情时，可以使用 proxy
let obj = new Proxy({
    add: function(val) {
        return val + 100;
    },
    name: 'wyzane'
}, {
    get: function(target, key, property){
        console.log("预处理");
    }
})
console.log(obj.name); // 在访问属性name前，会先执行get方法，但是此时obj.name为 undefined，get方法修改为：
let obj = new Proxy({
    add: function(val) {
        return val + 100;
    },
    name: 'wyzane'
}, {
    get: function(target, key, property){
        console.log("预处理");
        console.log(target);  // {add: ƒ, name: "wyzane"}  target就是原对象
        return target[key];
    }
})
console.log(obj.name);  // wyzane



// proxy-set方法，修改对象属性时使用
let obj = new Proxy({
    add: function(val) {
        return val + 100;
    },
    name: 'wyzane'
}, {
    get: function(target, key, property){
        console.log("预处理");
        console.log(target);
        return target[key];
    },
    set: function(target, key, value, receiver) {
        console.log(`setting ${key} = ${value}`);  // setting name = nihao
        target[key] = value;
    }
})

console.log(obj.name);
obj.name = 'nihao';
console.log(obj.name);  // nihao set中不加target[key] = value时，属性值不会被修改



// proxy-apply方法，用于对方法的预处理  apply的使用方式与上面的get和set不同
let target = function() {
    return 'nihao';
}
let handler = {
    apply(target, ctx, args) {
        console.log('apply func');  // apply func
    }
}

let obj = new Proxy(target, handler);

console.log(obj());  // undefined
// 上面的写法会有问题，apply方法中没有return

let target = function() {
    return 'nihao';
}
let handler = {
    apply(target, ctx, args) {
        console.log('apply func');  // apply func
        return Reflect.apply(...arguments);
    }
}

let obj = new Proxy(target, handler);

console.log(obj());  // nihao

```

## async与await的使用

async与await应该算是 ES7 中的语法，下面介绍下它的使用。

```txt
async 可以使一个方法变为异步方法
await 用于获取一个异步方法中的返回值，可以等待异步方法的执行完成并返回（await需要在异步方法中使用）
```

简单例子：

```js
async function getData() {
    return '你好';
}

console.log(getData());  // Promise { '你好' }
```

可以看到，async方法返回的是一个 Promise 对象，想要获取函数的返回值，可以使用下面的方式：

方式一：使用 then 方法

```js
async function getData() {
    return '你好';
}

let p = getData();

p.then((data)=>{
    console.log(data);
})
```

方式二：使用 await ,await只能在异步方法中使用

```js
async function getData() {
    return '你好';
}

async function test() {
    let ret = await getData();
    console.log(ret);
}

test()
```





## Promise的使用

Promise 是异步编程的一种解决方案，我们可以把将来发生的事件定义到 Promise 中。

下面是 Promise 使用的例子

```js
// Promise可以优雅的执行多重回调函数。es5中使用嵌套执行多重回调，不优雅
let state = 1;

let step1 = function(resolve, reject) {
    console.log('1-开始洗菜做饭');
    if (state == 1){
        resolve('洗菜做饭-完成');
    }else{
        reject('洗菜做饭-出错');
    }
}

let step2 = function(resolve, reject) {
    console.log('2.吃饭');
    if (state==1){
        resolve('吃饭-完成');
    }else{
        reject('吃饭-出错');
    }
}

let step3 = function(resolve, reject){
    console.log('3.洗碗');
    if (state==1) {
        resolve('洗碗-完成');
    }else{
        reject('洗碗-出错');
    }
}

new Promise(step1).then(function(val){
    console.log(val);
    return new Promise(step2);
}).then(function(val){
    console.log(val);
    return new Promise(step3);
}).then(function(val){
    console.log(val);
});

/*
1-开始洗菜做饭
洗菜做饭-完成
2.吃饭
吃饭-完成
3.洗碗
洗碗-完成
*/


// 出错时，程序会停止
let state = 1;

let step1 = function(resolve, reject) {
    console.log('1-开始洗菜做饭');
    if (state == 1){
        resolve('洗菜做饭-完成');
    }else{
        reject('洗菜做饭-出错');
    }
}

let step2 = function(resolve, reject) {
    state = 0;
    console.log('2.吃饭');
    if (state==1){
        resolve('吃饭-完成');
    }else{
        reject('吃饭-出错');
    }
}

let step3 = function(resolve, reject){
    console.log('3.洗碗');
    if (state==1) {
        resolve('洗碗-完成');
    }else{
        reject('洗碗-出错');
    }
}

new Promise(step1).then(function(val){
    console.log(val);
    return new Promise(step2);
}).then(function(val){
    console.log(val);
    return new Promise(step3);
}).then(function(val){
    console.log(val);
});

```

还可以在 async 方法中调用含有 Promise 的方法：

```js
function getData() {
    return new Promise((resolve, reject)=> {
        setTimeout(()=>{
            let username = 'zhangsan';
            resolve(username);
        }, 1000)
    })
}

async function test() {
    let ret = await getData();
    console.log(ret);
}

test()
```



## es6中类的使用

```js
// 类的定义
class Coder {
    name (val) {
        console.log(val);
    }
    skill (val) {
        console.log(val);
    }
}

let wyzane = new Coder; 
wyzane.name('nihao');   // nihao
wyzane.skill('nihaoya');  // nihaoya


// 类里面的成员之间互相访问使用this
class Coder {
    name (val) {
        console.log('name method is called');  // name method is called
        console.log(val);  // nihao
    }
    skill (val) {
        this.name(val)
    }
}

let wyzane = new Coder;
wyzane.name('nihao');


// 类的构造方法和静态方法
class Person {
    // 构造方法
    constructor (name, age) {
        this._name = name;
        this._age = age;
    }

    // 实例方法
    run () {
        console.log(this._name);
    }

    // 静态方法
    static work () {
        console.log('静态方法');
    }
}

let person = new Person('zhangsan', 20);
person.run();  // 调用实例方法
Person.work(); // 调用静态方法



// 类的继承
class Xiaoming extends Person {
    constructor (name, age, gender) {
        super(name, age);  // 通过super给父类传参
        this.gender = gender;
    }

    eat () {
        console.log('小明喜欢吃糖');
    }
}

let xm = new Xiaoming('xiaoming', 20, '男');
xm.run()
xm.eat();
Xiaoming.work()


// 单例类的实现
class Client {
    constructor () {
        console.log('执行构造方法');
    }

    static getInstance () {
        if (!Client.instance) {
            Client.instance = new Client();
        }

        return Client.instance
    }
};


// 多次获取对象时，只会执行一次构造方法
let client1 = Client.getInstance();
let client2 = Client.getInstance();
```

作为对比，es5中类的定义如下：

```js
// es5中定义类和静态方法
function Person (name, age) {
    // 这个是构造函数
    this.name = name;
    this.age = age;
    this.run = function () {
        console.log(this.name);
    }
}

// 原型链上的属性和方法（也属于实例方法）
/*原型链上的属性和方法与构造函数中的属性和方法的区别：
    1. 原型链上的属性和方法可以被多个实例共享
*/
Person.prototype.gender = '男';
Person.prototype.work = function () {
    console.log(this.name);
}

// 静态方法
Person.setName = function () {
    console.log('静态方法');
}

var p = new Person('zhansan', 20); //实例方法通过实例化来调用
// 调用实例方法
p.run();
p.work();

// 调用静态方法
Person.setName();


// es5中的继承：原型链继承和对象冒充继承
// 对象冒充继承(该继承方式不能继承原型链上的属性和方法，所以要结合原型链继承方式)
function Web (name, age) {
    Person.call(this, name, age);
}

// 原型链继承(该继承方式实例化子类的时候不能给父类传参，即不能将属性传递给父类，所以要结合对象冒充继承)
Web.prototype = new Person();

var web = Web('lisi', 20);
web.run();
```





## 模块化操作

```js
// export-输出   import-输入

temp.js:
export let name = 'wyzane';

index.js:
import {name} from './temp'
console.log(name);



// 导出多个变量
let a = 'a';
let b = 'b';
let c = 'c';
export {a, b, c};



// 函数的模块化输出
export function add(a, b) {
    return a + b;
}


// 导出时语义化处理，相当于引用的时候可以使用别名
let a = 'a';
let b = 'b';
let c = 'c';
export {
	name as a,
    cname as b,
    skill as c
}


// export default 与 export 的区别：一个文件中可以有很多export，但是只能有一个export default
/*
export导出的对象只能使用{}import
export default 导出的对象可以不使用{}import，并且可以自定义名称
*/

temp.js:
export default var a = 'nihao';

index.js
import 自定义名称 from './temp'
```

