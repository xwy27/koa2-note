# 2.3 模板引擎

我们到现在的页面，都是直接在 JavaScript 中输出的，如 hello world。这并不是一个网页应该有的样子，比如我们需要根据不同登陆用户，显示他们的用户名，那么单纯一个字符串，是不能满足需求的。这时候我们就需要模板引擎了。

那么模板引擎是什么呢？模板引擎的作用，就是配合数据构造出字符串输出。比如，下面的函数就可以称为模板引擎：

```javascript
function modelEngine(name) {
  return `Hello: ${name}`;
}
```

那么，Javascript 就带有字符串模板，我们为什么还要模板引擎呢？原因很简单，Javascript 的字符串模板只能写在 JavaScript 代码中。我们的网页，常常是由 html 渲染的，而且长度都不短，如果都写在 JavaScript 中，那么 JavaScript 代码长度将很恐怖。这个时候，我们就可以用模板引擎来帮忙了。

这里介绍一个模板引擎：[nunjucks](https://mozilla.github.io/nunjucks/)。Nunjucks 由 Mozilla 开发的，可以运行在 node 环境。关于更具体的用法，大家自行查阅官方文档，这里给出一个简短的例子。

1. 创建 views 文件夹
2. 在 views 文件夹新建 index.html
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
      {% if index %}
      <h1>{{ title }}</h1>
      {% else %}
      <p>{{ title }}</p>
      {% endif %}
    </body>
    </html>
    ```
3. 重新编写 app.js
    ```javascript
    const koa = require('koa');
    const router= require('koa-router')();
    const nunjucks = require('nunjucks');

    const app = new koa();

    let env = nunjucks.configure('views'); // 模板文件路径

    router.get('/', (ctx, next) => {
      ctx.response.body = env.render('index.html'， {
        title: 'Index',
        index: true
      });
    });

    app.use(router.routes());

    app.listen(3000, () => {
      console.log('Koa running at port 3000...');
    });
    ```

再次运行，我们看看结果:
![nunjucks](../../assets/image/nunjuck.jpg)

当然，模板引擎是 SSR(Sever Side Render) 的使用策略。