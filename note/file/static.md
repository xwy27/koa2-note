# 4.1 静态文件

我们利用 HTTP 请求一个 HTML 网页，网页通常还会包含 CSS 和 JavaScript 文件。HTML 通常通过路径引用这些文件，当我们请求这个 HTML 网页，浏览器会自动帮我们通过 URL 向服务器请求这些文件。这些文件均属于服务器的静态资源，在用户请求时，服务器应该将文件发送给用户，否则我们的网页就会变成一塌糊涂。

## 自制 Static-Middleware

我们编写一个 static-file 中间件来处理静态文件请求。

首先安装 mime，因为我们的 response 需要说明返回的数据类型，mime 可以根据文件信息返回 response type。

```bash
npm install mime
```

接着，我们完成 middleware 的编写。原理很简单，根据 URL 判断是否为请求静态资源，不是调用后面的中间件，是的话，查找文件并返回。

```javascript
// static-file.js
const path = require('path');
const mime = require('mime');
const fs = require('fs');

/**
 * Static file middleware
 * @param {string} url: request url pattern, like '/static/'
 * @param {string} dir: static file folder, like '__dirname + /static/'
 */
function staticFiles(url, dir) {
  return async (ctx, next) => {
    let reqPath = ctx.request.path;
    // if url matches the static file request url
    if (reqPath.startsWith(url)) {
      let filePath = path.join(dir, reqPath.substring(url.length));
      // if file exists
      if (fs.existsSync(filePath)) {
        ctx.response.type = mime.getType(reqPath);
        ctx.response.body = fs.readFileSync(filePath, 'binary');
      } else {
        ctx.response.status = 404;
      }
    } else {
      await next();
    }
  };
}

module.exports = staticFiles;
```

接着我们修改一下上次的 index.html 文件，让它引用一些静态文件。

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>{{ title }}</title>
</head>
<body>
  <h1>{{ title }}</h1>
  <button id="btn">Click me!</button>
</body>

<link rel="stylesheet" type="text/css" href="/static/css/index.css">
<script src="/static/js/index.js"></script>
</html>
```

接着，我们在根目录添加一个 static 文件夹，我们的静态文件都会放在这。它包含 css 和 js 两个文件夹。
然后分别将 index.css 和 index.js 放到对应文件夹下。代码如下：

```css
button {
  background-color: aqua;
}
```

```javascript
document.getElementById('btn').onclick = () => {
  alert('You clicked');
};
```

现在，我们的 sample 文件目录如下:

```
├─static
│  ├─css
│  |  └─index.css
│  └─js
│     └─index.css
├─views
│  └─index.html
├─staticFile.js
└─app.js
```

我们修改 app.js，完成测试代码：

```javascript
const koa = require('koa');
const static = require('./staticFiles');
const router= require('koa-router')();
const nunjucks = require('nunjucks');

const app = new koa();

let env = nunjucks.configure('views'); // path to model file folder

app.use(static('/static/', __dirname + '/static'));

router.get('/', (ctx, next) => {
  ctx.response.body = env.render('index.html', {
    title: 'Index'
  });
});

app.use(router.routes());

app.listen(3000, () => {
  console.log('Koa running at port 3000...');
});
```

测试结果

![static](../../assets/image/static.jpg)
![static](../../assets/image/static2.jpg)

## koa-static

每次都自己写一个 static middleware，是一件很繁琐的事情。这个时候，我们就不用造轮子了，用别人写好的库：`koa-static`。**千万注意 koa-static 要在 koa-router 后面使用**，因为 koa-static 会拦截路由，这样你写的路由很可能会被 koa-static 处理而出现和你想象不符的情况。在 koa-static 源码中，如如下代码：
```javascript
// koa-static index.js
if (ctx.method !== 'HEAD' && ctx.method !== 'GET') return
// response is already handled
if (ctx.body != null || ctx.status !== 404) return
```
非请求和已处理的请求会被它略过，所以，我们先处理路由，再交给 koa-static 是很安全的。

直接更改 app.js：

```javascript
const koa = require('koa');
const router = require('koa-router')();
const static = require('koa-static');
const path = require('path');
const nunjucks = require('nunjucks');

const app = new koa();

let env = nunjucks.configure('views'); // path to model file folder

router.get('/', (ctx, next) => {
  ctx.response.body = env.render('index.html', {
    title: 'Index'
  });
});

app.use(router.routes());

app.use(static(path.join( __dirname,  './static')));


app.listen(3000, () => {
  console.log('Koa running at port 3000...');
});
```

效果和前面是一样的。