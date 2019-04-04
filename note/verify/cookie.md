# 5.1 cookies

作为一名程序猿必然知晓，HTTP 是一个无状态协议。为了能在服务器辨识请求的用户，cookies 和 session 机制就诞生了~ 更具体的知识，大家 Google 一下，看看 Wiki，我就不瞎说了。

下面我们来看看 koa 中是如何操作 cookies 的。在 koa 中，它提供了操作 cookies 的接口：`ctx.cookies.get(name, options)` 和 `ctx.cookies.set(name, value, options)`(其实 koa 直接调用了 [cookies](https://github.com/pillarjs/cookies)，所以二者的用法是一样的~)。

还是直接实践一下。我们修改 index.html，引入 axios:

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>{{ title }}</title>

  <script src="https://cdn.bootcss.com/axios/0.18.0/axios.min.js"></script>
</head>
<body>
  <h1>{{ title }}</h1>
  <button id="btn">Click me!</button>
</body>

<link rel="stylesheet" type="text/css" href="css/index.css">
<script src="js/index.js"></script>
</html>
```

修改 index.js，完成设置 cookie 和请求 cookie 的操作：

```javascript
// index.js
document.getElementById('btn').onclick = () => {
  axios
    .get('/set-cookies')
    .then((res) => {
      window.location.href = '/get-cookies';
    })
    .catch((err) => {
      alert(err);
    });
};
```

最后，修改 app.js，处理请求：

```javascript
// app.js
const koa = require('koa');
const router = require('koa-router')();
const static = require('koa-static');
const nunjucks = require('nunjucks');

const path = require('path');

const app = new koa();

let env = nunjucks.configure('views'); // path to model file folder

router.get('/', (ctx, next) => {
  let cookieValue = ctx.cookies.get('cookie-sample');
  ctx.response.body = env.render('index.html', {
    title: cookieValue === undefined ? 'index' : cookieValue
  });
});

router.get('/set-cookies', (ctx, next) => {
  ctx.cookies.set('cookie-sample', 'sample', { overwrite: true });
  ctx.response.body = 'Success';
});

router.get('/get-cookies', (ctx, next) => {
  let cookieValue = ctx.cookies.get('cookie-sample');
  ctx.response.body = env.render('index.html', {
    title: cookieValue
  });
});

app.use(router.routes());

app.use(static(path.join( __dirname,  './static')));

app.listen(3000, () => {
  console.log('Koa running at port 3000...');
});
```

可以看到，未点击按钮之前，我们的标题是 index，点击按钮之后，我们的页面就根据我们设置的 cookie 值改变了：

![cookies](../../assets/image/cookies.jpg)