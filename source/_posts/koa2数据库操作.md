---
title: koa2数据库操作
date: 2020-06-13 21:34:25
tags:
categories: Nodejs
---

今天以 postgresql数据库为例，介绍下 koa2 中如何操作数据库。

<!--more-->

# 所需中间件

下面使用到了以下中间件：

```txt
koa-pg
koa-router
koa-bodyparser

其中，koa-pg 作为koa连接pg数据库的中间件，koa-router作为路由中间件，koa-bodyparser作为解析post请求中json数据的中间件。
```

# 代码示例

## 使用客户端连接数据库

postgre.js：

```js
const pg = require('pg');

const conString = "postgres://postgres:wyzane@localhost:5432/test_koa2";

const client = new pg.Client(conString);

module.exports = client;
```



app.js：

```js
const Koa = require('koa');
const Router = require('koa-router');
const bodyparser = require('koa-bodyparser');

const app = new Koa();
const router = new Router();

const client =  require('./dbs/postgres.js');

app.use(bodyparser());

var respData = {
    'code': '0000',
    'msg': '成功',
    'data': null
}

function query (sql, values) {
    return new Promise((resolve, reject) => {
        client.connect(async function(err) {
            if (err) {
                reject(err);
            }
        
            client.query(sql, values, function(err, results) {
                if (err) {
                    reject(err);
                }
                resolve(results);
            });

        });
    })       
}

// 查询用户列表
router.get('/user/list', async (ctx, next)=>{
    await query('SELECT * FROM tb_user', []).then((data) => {
        respData.data = data.rows;
        ctx.body = respData;
    }).catch((err) => {
        respData.code = '10001';
        respData.msg = '查询用户列表失败';

        ctx.body = respData;
    })
});

// 查询用户详情
router.post('/user/detail', async (ctx, next)=>{
    let postParams = ctx.request.body;

    userId = postParams.userId;

    await query('SELECT * FROM tb_user WHERE user_id = $1', [userId]).then((data) => {
        respData.data = data.rows;
        ctx.body = respData;
    }).catch((err) => {
        respData.code = '10002';
        respData.msg = '查询用户详情失败';

        ctx.body = respData;
    })
});

// 更新用户数据
router.post('/user/update', async (ctx, next)=>{
    let postParams = ctx.request.body;

    userId = postParams.userId;
    username = postParams.username;

    await query('UPDATE tb_user set username = $1 WHERE user_id = $2', [username, userId]).then((data) => {
        console.log('更新数据:', data);
        ctx.body = respData;
    }).catch((err) => {
        respData.code = '10003';
        respData.msg = '更新用户信息失败';

        ctx.body = respData;
    })
});

// 删除用户数据
router.post('/user/delete', async (ctx, next)=>{
    let postParams = ctx.request.body;

    userId = postParams.userId;

    await query('DELETE FROM tb_user WHERE user_id = $1', [userId]).then((data) => {
        console.log('删除数据:', data);
        
        ctx.body = respData;
    }).catch((err) => {
        console.log('delete data error!');
        rollback(client);

        respData.code = '10004';
        respData.msg = '删除用户信息失败';

        ctx.body = respData;
    })
});

// 添加数据
router.post('/user/add', async (ctx, next)=>{
    let postParams = ctx.request.body;

    username = postParams.username;
    password = postParams.password;
    phone = postParams.phone;

    await query('INSERT INTO tb_user(username, password, phone, add_time) VALUES($1, $2, $3, now())', 
                [username, password, phone]).then((data) => {
        console.log('添加数据:', data);
        ctx.body = respData;
    }).catch((err) => {
        console.log('add data error!');
        rollback(client);

        respData.code = '10005';
        respData.msg = '添加用户信息失败';

        ctx.body = respData;
    })
});

// 定义回滚函数
let rollback = function (client) {
    client.query('ROLLBACK', function () {
        client.end();
    })
}

app.use(router.routes()).use(router.allowedMethods());

app.listen(9090, ()=>{
    console.log('server start ...');
})
```

上面的代码中，postgre.js 文件中实例化了一个 postgresql 数据库客户端，并导出。app.js 中分别实现了增、删、改、查方法。

其中，client.query(sql, values, func) 是主要方法，参数1表示需要执行的 sql 语句，参数2表示 sql 语句中对应的值，参数3是一个回调函数，用于 sql 执行完后的回调工作。



上面执行的 sql 中，返回的 data 内容分别如下，它们都是 Result 对象：

查询列表返回的结果：

```json
Result {
  command: 'SELECT',
  rowCount: 9,
  oid: null,
  rows: [
    {
      user_id: 2,
      username: 'guangtouqiang',
      password: '123456',
      phone: '123456',
      add_time: 2020-05-16T16:00:00.000Z
    },
    {
      user_id: 3,
      username: 'xiongda',
      password: '123456',
      phone: '123456',
      add_time: 2020-05-16T16:00:00.000Z
    }
  ],
  fields: [
    Field {
      name: 'user_id',
      tableID: 41294,
      columnID: 1,
      dataTypeID: 23,
      dataTypeSize: 4,
      dataTypeModifier: -1,
      format: 'text'
    },
    Field {
      name: 'username',
      tableID: 41294,
      columnID: 2,
      dataTypeID: 1043,
      dataTypeSize: -1,
      dataTypeModifier: 104,
      format: 'text'
    },
    Field {
      name: 'password',
      tableID: 41294,
      columnID: 3,
      dataTypeID: 1043,
      dataTypeSize: -1,
      dataTypeModifier: 44,
      format: 'text'
    },
    Field {
      name: 'phone',
      tableID: 41294,
      columnID: 4,
      dataTypeID: 1043,
      dataTypeSize: -1,
      dataTypeModifier: 15,
      format: 'text'
    },
    Field {
      name: 'add_time',
      tableID: 41294,
      columnID: 5,
      dataTypeID: 1082,
      dataTypeSize: 4,
      dataTypeModifier: -1,
      format: 'text'
    }
  ],
  _parsers: [
    [Function: parseInteger],
    [Function: noParse],
    [Function: noParse],
    [Function: noParse],
    [Function: parseDate]
  ],
  _types: TypeOverrides {
    _types: {
      getTypeParser: [Function: getTypeParser],
      setTypeParser: [Function: setTypeParser],
      arrayParser: [Object],
      builtins: [Object]
    },
    text: {},
    binary: {}
  },
  RowCtor: null,
  rowAsArray: false
```

查询详情返回的结果：

```json
Result {
  command: 'SELECT',
  rowCount: 1,
  oid: null,
  rows: [
    {
      user_id: 6,
      username: 'chunzhang',
      password: '12456',
      phone: '123456',
      add_time: 2020-05-16T16:00:00.000Z
    }
  ],
  fields: [
    Field {
      name: 'user_id',
      tableID: 41294,
      columnID: 1,
      dataTypeID: 23,
      dataTypeSize: 4,
      dataTypeModifier: -1,
      format: 'text'
    },
    Field {
      name: 'username',
      tableID: 41294,
      columnID: 2,
      dataTypeID: 1043,
      dataTypeSize: -1,
      dataTypeModifier: 104,
      format: 'text'
    },
    Field {
      name: 'password',
      tableID: 41294,
      columnID: 3,
      dataTypeID: 1043,
      dataTypeSize: -1,
      dataTypeModifier: 44,
      format: 'text'
    },
    Field {
      name: 'phone',
      tableID: 41294,
      columnID: 4,
      dataTypeID: 1043,
      dataTypeSize: -1,
      dataTypeModifier: 15,
      format: 'text'
    },
    Field {
      name: 'add_time',
      tableID: 41294,
      columnID: 5,
      dataTypeID: 1082,
      dataTypeSize: 4,
      dataTypeModifier: -1,
      format: 'text'
    }
  ],
  _parsers: [
    [Function: parseInteger],
    [Function: noParse],
    [Function: noParse],
    [Function: noParse],
    [Function: parseDate]
  ],
  _types: TypeOverrides {
    _types: {
      getTypeParser: [Function: getTypeParser],
      setTypeParser: [Function: setTypeParser],
      arrayParser: [Object],
      builtins: [Object]
    },
    text: {},
    binary: {}
  },
  RowCtor: null,
  rowAsArray: false
}
```

执行更新操作返回的结果：

```json
Result {
  command: 'UPDATE',
  rowCount: 1,
  oid: null,
  rows: [],
  fields: [],
  _parsers: undefined,
  _types: TypeOverrides {
    _types: {
      getTypeParser: [Function: getTypeParser],
      setTypeParser: [Function: setTypeParser],
      arrayParser: [Object],
      builtins: [Object]
    },
    text: {},
    binary: {}
  },
  RowCtor: null,
  rowAsArray: false
}
```

执行删除操作返回的结果：

```json
Result {
  command: 'DELETE',
  rowCount: 0,
  oid: null,
  rows: [],
  fields: [],
  _parsers: undefined,
  _types: TypeOverrides {
    _types: {
      getTypeParser: [Function: getTypeParser],
      setTypeParser: [Function: setTypeParser],
      arrayParser: [Object],
      builtins: [Object]
    },
    text: {},
    binary: {}
  },
  RowCtor: null,
  rowAsArray: false
}
```

执行添加操作返回的结果：

```json
Result {
  command: 'INSERT',
  rowCount: 1,
  oid: 0,
  rows: [],
  fields: [],
  _parsers: undefined,
  _types: TypeOverrides {
    _types: {
      getTypeParser: [Function: getTypeParser],
      setTypeParser: [Function: setTypeParser],
      arrayParser: [Object],
      builtins: [Object]
    },
    text: {},
    binary: {}
  },
  RowCtor: null,
  rowAsArray: false
}
```



## 使用连接池连接数据库

舒勇连接池连接数据库时，postgre.js 文件的配置如下：

```js
const pg = require('pg');

const pgConfig = {
    user: 'postgres',
    database: 'test_koa2',
    password: 'wyzane',
    host: 'localhost',
    port: '5432'
}

const pool = new pg.Pool(pgConfig);

module.exports = pool;
```

app.js 中查询方法定义如下：

```js
function query (sql, values) {
    return new Promise((resolve, reject) => {
        pool.connect(function(err, client, done) {
            if (err) {
                console.log('连接失败：', err);
                reject(err);
            }

            client.query(sql, values, function(err, results) {
                if (err) {
                    reject(err);
                }
                resolve(results);
            })
        })
    })
}
```

其余的 sql 操作代码都与使用 client 连接数据库一样。



## 把连接操作封装成对象

实际开发中，我们可以把数据库连接方法和增、删、改、查等操作数据库的方法封装在一个对象中，像下面这样。

postgres.js：

```js
const pg = require('pg');

const pgConfig = {
    user: 'postgres',
    database: 'test_koa2',
    password: 'wyzane',
    host: 'localhost',
    port: '5432'
}

const pool = new pg.Pool(pgConfig);


class Postgres {

    // 连接数据库方法
    connect () {
        return new Promise((resolve, reject) => {
            pool.connect((err, client, done) => {
                if (err) {
                    reject(err)
                }

                resolve(client);
            })
        })
    }

    // 查询列表
    list (sql, values) {
        return new Promise((resolve, reject) => {
            this.connect().then((client) => {
                client.query(sql, values, (err, results) => {
                    if (err) {
                        reject(err);
                    }
                    resolve(results);
                })
            })
        })
    }
}

let postgres = new Postgres();

module.exports = postgres;
```

app.js：

```js
const Koa = require('koa');
const Router = require('koa-router');
const bodyparser = require('koa-bodyparser');

const app = new Koa();
const router = new Router();

// 引入数据库实例
const postgres =  require('./dbs/postgres.js');

app.use(bodyparser());

var respData = {
    'code': '0000',
    'msg': '成功',
    'data': null
}


// 查询用户列表
router.get('/user/list', async (ctx, next)=>{
    let sql = 'SELECT * FROM tb_user';
    let values = [];
    
    await postgres.list(sql, values).then((data) => {     
        respData.data = data.rows;
        ctx.body = respData;
    }).catch((err) => {
        respData.code = '10001';
        respData.msg = '查询用户列表失败';

        ctx.body = respData;
    })
});


app.use(router.routes()).use(router.allowedMethods());

app.listen(9090, ()=>{
    console.log('server start ...');
})
```



## 使用单例模式进一步优化

对于数据库连接对象，我们可以使用单例模式做进一步优化，例子如下：

postgre.js：

```js
const pg = require('pg');

const pgConfig = {
    user: 'postgres',
    database: 'test_koa2',
    password: 'wyzane',
    host: 'localhost',
    port: '5432'
}

const pool = new pg.Pool(pgConfig);


class Postgres {

    constructor () {
        this.dbClient = '';
        this.connect()
    }

    connect () {
        let _that = this;
        return new Promise((resolve, reject) => {
            if (!_that.dbClient) {
                pool.connect((err, client, done) => {
                    if (err) {
                        reject(err)
                    }

                    _that.dbClient = client;
                    resolve(_that.client);
                })
            } else {
                resolve(_that.dbClient);
            }
        })
    }

    list (sql, values) {
        return new Promise((resolve, reject) => {
            this.connect().then((client) => {
                client.query(sql, values, (err, results) => {
                    if (err) {
                        reject(err);
                    }
                    resolve(results);
                })
            })
        })
    }
}

let postgres = new Postgres();

module.exports = postgres;
```

上面的例子中，使用了一个 dbClient 属性，当 dbClient 存在时，直接返回，而不是再次连接数据库，这样节省了连接数据库所耗费的时间。

