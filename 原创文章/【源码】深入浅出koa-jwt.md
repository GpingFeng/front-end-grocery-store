## 前言
之前发了一篇文章，使用 [【Node】使用 koa2 实现一个简单JWT鉴权](https://juejin.cn/post/6921493257578872845)，先来回顾一下 `JWT` 鉴权的整体流程

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/606f394383b84198af2c08a23034019a~tplv-k3u1fbpfcp-zoom-1.image)

首先，用户登录后服务端根据用户信息生成并返回 `token` 给到客户端，前端在下次请求中把 `token` 带给服务器，服务器验证有效后，返回数据。无效的话，返回 `401` 状态码。

这里验证是否有效的时候用到 [koa-jwt](https://github.com/koajs/jwt) 这个库，今天我们就来看看这个库做了什么东西吧

## koa-jwt 的基础使用

来看下官方的简单使用示例

```js
var Koa = require('koa');
var jwt = require('koa-jwt');

var app = new Koa();

// Middleware below this line is only reached if JWT token is valid
// unless the URL starts with '/public'
app.use(jwt({ secret: 'shared-secret' }).unless({ path: [/^\/public/] }));

// Unprotected middleware
app.use(function(ctx, next){
  if (ctx.url.match(/^\/public/)) {
    ctx.body = 'unprotected\n';
  } else {
    return next();
  }
});

// Protected middleware
app.use(function(ctx){
  if (ctx.url.match(/^\/api/)) {
    ctx.body = 'protected\n';
  }
});

app.listen(3000);
```


- 获取 `token` 的多种方法，优先队列的实现
- 将 `token` 设置为 `ctx.state` 的值，方便获取
- `isRevoked` 方法检测是否有效
- 验证异常的时候