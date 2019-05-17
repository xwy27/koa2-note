# 5.3 JWT

我们知道如何在 koa 中使用 cookies 和 session。那么我们再来看看近些年开始流行的 token 机制。正是 token 驱使我写下 koa2 教程(~~感觉自己当时学了很久，不写点东西对不起自己~~)。

## 随便谈谈

用我浅显的见识，来聊一聊 token 把。如果有错，欢迎大佬们指正。

为什么会有 token 呢？这个和 session 有关。session 解决了 HTTP 无状态但服务器又希望识别此次访问的用户的问题。对于登陆后的用户，服务器为其生成唯一的 session_id 并存进数据库，同时将其返回给用户浏览器。浏览器将它存入 cookie，之后每次访问都将 cookie 带上，服务器访问 cookie，寻找存入的 session_id，就能识别当前的用户。但是，我们的服务器现在基本上是分布式的。这就带来了一个很致命的问题：session 一致性。
考虑这么一个情况：一个用户，原本已经登陆，拥有了 session_id，该地区的服务器是已知的。接着，在 session 有效期内，用户换了个地方，由另一个服务器为它服务。理论上，新服务器应该能够鉴别识别这个用户。但如何做到呢？有一个很笨的解决方案，专门运行一个存储 session_id 的数据库服务器。所有服务器都向这个数据库进行访问。
听起来，一切问题都解决了。但真的是这样吗？如果，我们的用户数据十分庞大，比如谷歌这些 IT 巨头，集中式的数据库，就很难满足需求了。
这个时候，我们开始思考，为什么我们要把用户的 session_id 存起来？他们自己存不行吗？是的，让用户自己存着就好了。但如果直接让用户存一个 session_id，被人伪造了怎么办？那就加一个鉴别功能。这就是 token，一个身份令牌。它可以完成像 session_id 一样的识别用户的功能。用户登陆后，服务器颁发一个经过加密的身份令牌给用户。用户把这个身份令牌存起来，以后每次向服务器发起请求的时候，都带上这个身份令牌，我们的服务器就能够鉴别是否是合法用户了。同时，由于所有服务器运行的加密 token 算法是一致的，所以不存在不同服务器无法鉴别的情况。而且，token 的存储是在客户端，而不是服务器，大大降低了服务成本。这就是基本的 token 原理。

## JWT

JWT 是 Json 形式的 token，在 web 上的常用 token 形式。下面，我们来看看 JWT(JsonWebToken) 的构成和工作原理。

```javascript
{
  header: {
    typ: 'jwt',
    alg: 'HS256'
  },
  payload: {
    exp: 14483,
    user: 'username'
  },
  signature: // encrypted string for header and payload
}
```

JWT 由三个部分组成，header，payload 和 signature。

- header
  
  头部，存储 token 的相关信息，比如类型和加密算法。

- payload

  有效负荷，存储签发者，签发时间，过期时间等相关信息和服务器需要放入的信息

- signature

  签名，将 header 和 payload 使用 64 进制编码后的字符串一同进行加密后的结果。

这就是一个 JWT 的构成。服务器将生成的 token 进行编码后的字符串，发送给用户。以后用户每次都将这个 token 字符串写进 Authorization Header(*注意一下写入格式，作为小 trick 给大家自己去查一下*)，服务器进行解析，并且重新计算 header 和 payload 加密结果，看是否与 signature 匹配，以此决定用户是否合法。**用 CPU 计算时间换取了存储空间**，并且解决了服务期间同步 session 的问题。**敏感信息不要放入 JWT，除非经过加密**

## koa-jwt

接着，我们实践一下，用 JWT 来完成和 session 一样的，限制访问页面的功能。

安装两个库：koa-jwt 和 jsonwebtoken。第一个是拦截请求，解析 jwt 的库，第二个是签发 jwt 的库。

```bash
npm install koa-jwt
npm install jsonwebtoken
```

我们直接修改上一节的 app.js：

```javascript
const koa = require('koa');
const router = require('koa-router')();
const static = require('koa-static');
const koaJwt = require('koa-jwt');
const jwt = require('jsonwebtoken');
const nunjucks = require('nunjucks');

const path = require('path');

const app = new koa();

let env = nunjucks.configure('views'); // path to model file folder

// Add jwt verification middleware
app.use(koaJwt({secret: 'secret', cookie: 'jwt'}) // cookie makes token also be searched in cookies with certain name, too
  .unless({path: [/^\//, /^\/get-token/, /^\/js\/*/, /^\/css\/*/]})); // the url need not to be verified

router.get('/', (ctx, next) => {
  ctx.response.body = 'Unprotected';
});

router.get('/get-token', (ctx, next) => {
  // Token generate
  const token = jwt.sign({ user: 'sample' }, 'secret', { expiresIn: '1h' });
  ctx.status = 200;
  ctx.cookies.set('token', token);
  ctx.redirect('/index');
});

router.get('/index', (ctx, next) => {
  ctx.response.body = env.render('index.html', {
    title: 'Protected'
  });
});

app.use(router.routes());

app.use(static(path.join( __dirname,  './static')));

app.listen(3000, () => {
  console.log('Koa running at port 3000...');
});
```

试一下直接访问 http://localhost:3000/ ，显示是 Unprotected。接着访问 http://localhost:3000/index ，服务器会给出 Authentication Error 的信息，因为我们无法通过 jwt 验证 (未登陆)。接着，访问 http://localhost:3000/get-token ，然后我们就可以正常访问 http://localhost:3000/index 了。浏览器里也留下了我们的 token：

![token](../../assets/image/token.jpg)

### 代码解析

用起来很简单。接着稍微说一下，刚刚 koa-jwt 和 jsonwebtoken 调用时传入参数的作用。

- koa-jwt

  ```javascript
  app.use(koaJwt({secret: 'secret', cookie: 'jwt'})
    .unless({path: [/^\//, /^\/get-token/, /^\/js\/*/, /^\/css\/*/]}));
  ```

  secret 就是我们的加密密钥，是不能随便泄露的东西，而且要和 jsonwebtoken 的加密密钥匹配。
  
  后面的 cookie 的作用，是告诉 koa-jwt，你不仅要在 Authorization Header 检查 jwt，还要把 cookie 里键值为 jwt 的值作为 jwt 进行检查。

  之前说过，我们是要把 JWT 写入 Authorization Header 来检查的，为什么突然又写到 cookie 里了？这是为前端开发提供的一个便利。我们的页面跳转，如 `window.loaction.href = '/index'`，是由浏览器自动执行的。该请求头是通用头，里面是不会包含 Authorization Header 的。这样一来，原本登陆的用户，却无法跳转页面，必然要背锅的。所以，直接写进 cookie，一了百了，每次请求必然带有 token。

  unless 是指明哪些 URL 不需要进行 jwt 鉴权。

  **很重要的一点，调用 koa-jwt 中间件的代码以下的中间件，只有 token 合法才会运行**。这就是 koa-jwt 对路由进行的拦截了。这也再次说明了要用好 koa，必须注意中间件的顺序。

- jsonwebtoken

  ```javascript
  const token = jwt.sign({ user: 'sample' }, 'secret', { expiresIn: '1h' });
  ```

  第一个参数是你希望放入的信息，第二个是加密密钥，第三个是配置信息。
  
这只是一个例子，更具体的使用要看库的文档：[koa-jwt](https://www.npmjs.com/package/koa-jwt) 和[jsonwebtoken](https://www.npmjs.com/package/jsonwebtoken)