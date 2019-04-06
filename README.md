# koa2-note

## 序言

这是记录我学习使用 koa2 的笔记，也希望可以帮助更多想要学习 koa2 的人。

## 前言

什么是 koa2? *Next generation web framework for node.js*，这是 koa 团队对它的定位，下一代扎根于 node.js 的 web 框架。为什么是下一代呢？因为之前有 express。但 koa 的开发团队，也就是开发 express 的团队。所以你也可以将 koa 看作是 express 的进化版。

原来的 express 用户可能会问，koa 相较于 express，究竟好在哪里？按我的观点来看，它好在两个方面。第一，koa 利用 async 代替了回调，避免了 express 可能会带来的回调地狱。你当然可以说，自己再包装或者使用第三方库可以解决这个问题。但有已经写好而且使用起来习惯相同的框架，为什么不用呢？第二，koa 摒弃了 express 的冗余，它不捆绑任何中间件，只提供一套它自己包装的方法，让整个框架变得十分精简。据说，这也是 koa 出现的原因，大家受够了笨重，想要简单轻便。

并没有人要求你放弃其他框架，只拥抱 koa。不过，只要感兴趣，学习 koa 肯定是件好事。技多不压身，况且学起来很简单。不过，koa 官方的文档，对于入门者而言，会难以上手。这也是这本笔记诞生的初衷之一。

在完成过程中，参考过别人对于 koa 的上手文章。有很多地方其实都差不多，也存在差异，没有谁对谁错，看个人习惯而已。在开始动手之前，很大一部分原因就是为了写下 [5.3 token](note/verify/token.md) ，[7.1 项目结构](note/struct/struct.md) 和 [8.1 Mocha/Chai/Supertest](note/test/test.md) 这三个小节。这三个小节的内容，是我在给实验室使用 koa2 搭建一个简易项目时，学习感触(~~BUG~~)最深(~~多~~)的地方。这也驱使我将他们都记录下来。

## 目录

* [koa2-note](README.md)
* 1.快速上手
    * [1.1 Hello World](note/start/helloworld.md)
    * [1.2 代码初探](note/start/codeInspect.md)
* 2.路由
    * [2.1 自制路由](note/router/custom.md)
    * [2.2 koa-router](note/router/koa-router.md)
    * [2.3 模板引擎](note/router/nunjucks.md)
* 3.数据请求
    * [3.1 GET](note/data/get.md)
    * [3.2 POST](note/data/post.md)
* 4.文件服务
    * [4.1 静态文件](note/file/static.md)
    * [4.2 文件上传/下载](note/file/upload.md)
* 5.用户认证
    * [5.1 cookie](note/verify/cookie.md)
    * [5.2 session](note/verify/session.md)
    * [5.3 token](note/verify/token.md)
* 6.数据库
    * [6.1 MongoDB](note/database/mongodb.md)
    * [6.2 MySQL](note/database/mysql.md)
* 7.项目重构
    * [7.1 项目结构](note/struct/struct.md)
* 8.单元测试
    * [8.1 Mocha/Chai/Supertest](note/test/test.md)

## 参考内容

- koajs，[koa doc](https://koajs.com/)
- 廖雪峰，[koa 入门](https://www.liaoxuefeng.com/wiki/001434446689867b27157e896e74d51a89c25cc8b43bdb3000/001434501579966ab03decb0dd246e1a6799dd653a15e1b000)
- 大深海，[koa-note](https://chenshenhai.github.io/koa2-note/)

## 关于作者

<img style="max-width: 50px; max-height: 50px;" src="https://avatars0.githubusercontent.com/u/27471274?s=400&u=a6ce2f0e709385117969efe4a11e0be1f2ae44f8&v=4">

[xwy27](https://github.com/xwy27)
