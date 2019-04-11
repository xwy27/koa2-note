# 6.1 MongoDB

[mongoDB](https://www.mongodb.com/) 是非关系型数据库，属于文档型数据库，是最近新型的数据库，还在发展当中。文档型数据库，即可以存放xml、json等类型系的数据。这些数据具备自述性(self-describing)，呈现分层的树状数据结构。数据结构由键值(key=>value)对组成。

文档型数据将数据存储在硬盘中，但对于经常访问的数据，会 load 进内存，以此加快访问速度。这种热数据优化的数据库，对于小数据项目，具有很好的支持度。

## 安装 MongoDB

为了让我们连上数据库，我们要先[下载 mongoDB](https://docs.mongodb.com/manual/administration/install-community/)。我们也可以通过 Docker 直接 `docker pull mongo`，避免安装出错的尴尬。

## 安装 mongoose

怎么连接数据库呢？依旧有第三方库可以帮忙：mongoose

```bash
npm install mongoose
```

## 代码样例

我们修改一下 app.js，让我们可以在数据库中增加一个用户，然后我们从数据库中查询新加入的用户并且返回。

```javascript
const koa = require('koa');
const router = require('koa-router')();
const koaBody = require('koa-body');
const mongoose = require('mongoose');

const app = new koa();

function customSync(promise) {
  return promise
    .then(data => {
    return [null, data];
  })
    .catch(err => [err]);
}

// database
const dbConfig = {
  ip: '192.168.1.100',
  port: 27017,
  name: 'mongo_db'
}
const dbURL = 'mongodb://' + dbConfig.ip + ':' + dbConfig.port + '/' + dbConfig.name;
mongoose.connect(dbURL, { useCreateIndex: true, useNewUrlParser: true });
let db = mongoose.connection;
db.on('error', (err) => {console.log(`[Database] [Fail] Database Error: ${err}`);});
db.once('open', () => {console.log(`[Database] [Success] Database online`);});

// define the user schema in database
const userSchema = new db.Schema({
  username: { type: String, unique: true, dropDups: true },
  password: String
});

// model the schema
let user = db.model('user', userSchema);

// request data parser
app.use(koaBody());

// routers
router.post('/api/add-user', (ctx, next) => {
  let username = ctx.request.body.username;
  let password = ctx.request.body.password;
  const u = new user({
    username: username,
    password: password,
  });
  let [err, temp] = await customAsync(u.save().exec());
  if (err) {
    console.error(`Error query ${username}`);
    console.error(err);
    ctx.response.body = 'Error';
    return;
  }

  let [err, temp] = await customAsync(user.findOne({ username: username }).exec());
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

我们可以使用 `curl http://localhost:3000/api/add-user -X POST "username=test&password=test"` 直接在 Terminal 进行测试。

这里只是简单的展示了一下数据库的连接和增查操作，mongoose 提供了很多操作函数，想使用 mongodb 的可以查阅[文档](https://mongoosejs.com/docs/index.html)。值得一提的是，mongoose 的数据库操作是异步的，它可以返回一个 Promise，所以代码中我封装了一个 `customAsync` 函数来处理 Promise，保证数据库操作的同步进行。