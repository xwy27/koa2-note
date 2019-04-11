# 6.2 MySQL

MySQL 是关系型数据库，它是成熟度较高的数据库，适用范围很广，但是在面对海量数据时，性能有些不足。

## 安装 MySQL

首先我们安装 MySQL：https://www.mysql.com/downloads/ 。当然，我们依然可以使用 Docker 来安装 MySQL：`docker pull mysql`。

## 安装 mysql 模块

```bash
npm install mysql
```

## 代码样例

我们利用 MySQL 在数据库中初始增加一个用户，然后访问查询并返回结果。

```javascript
const koa = require('koa');
const router = require('koa-router')();
const koaBody = require('koa-body');
const mysql = require('mysql');

const app = new koa();

function customSync(promise) {
  return promise
    .then(data => {
    return [null, data];
  })
    .catch(err => [err]);
}

// database
const pool = mysql.createPool({
  host     : '127.0.0.1',
  user     : 'root',
  password : '123456',
  database : 'test_db'
});

let query = function( sql, values ) {
  return new Promise(( resolve, reject ) => {
    pool.getConnection(function(err, connection) {
      if (err) {
        reject( err );
      } else {
        connection.query(sql, values, (err, rows) => {
          if ( err ) {
            reject( err );
          } else {
            resolve( rows );
          }
          connection.release();
        })
      }
    })
  });
}

// initial table
query(`
  CREATE TABLE IF NOT EXISTS User
  (username VARCHAR(200) NOT NULL,
   password TEXT
   PRIMARY KEY(username))`)
  .then(result => {
    console.log('Create user table.')
  })
  .catch(error => {
    console.error('Error create table.')
    console.error(error);
  });
query(`
  INSERT INTO User (username, password)
  VALUES('test', 'test')`))
  .then(result => {
    console.log('Add test user.');
  })
  .catch(err => {
    console.error('Error add test user');
    console.error(error);
  });

// request data parser
app.use(koaBody());

// routers
router.post('/api/query-user', async (ctx, next) => {
  let username = ctx.request.body.username;
  let password = ctx.request.body.password;
  const u = new user({
    username: username,
    password: password,
  });
  let [err, temp] = await customAsync(query(
    `SELECT username FROM User
    WHERE username='${username}' AND password='${password}'`
  ));
  if (err) {
    console.error(`Error query ${username}`);
    console.error(err);
    ctx.response.body = 'Error';
    return;
  }
  ctx.response.body = temp;
});

app.use(router.routes());

app.listen(3000, () => {
  console.log('Koa running at port 3000...');
});
```

同样可以使用 curl 测试查询结果。
`curl http://localhost:3000/api/query-user -X POST "username=test&password=test"`
`curl http://localhost:3000/api/query-user -X POST "username=t&password=test"`