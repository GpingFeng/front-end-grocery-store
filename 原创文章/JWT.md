## JWT 简介

> 本部分参考了阮一峰老师的文章——[JSON Web Token 入门教程](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)

### 什么是 JWT

全称 `JSON Web Token` 是目前最流行的跨域认证解决方案。基本的实现是服务端认证后，生成一个 `JSON` 对象，发回给用户。用户与服务端通信的时候，都要发回这个 `JSON` 对象。

该 `JSON` 类似如下：

```json
{
  "姓名": "张三",
  "角色": "管理员",
  "到期时间": "2018年7月1日0点0分"
}
```

### 为什么需要 JWT
先看下一般的认证流程，基于 `session_id` 和 `Cookie` 的模式

>1、用户向服务器发送用户名和密码。
2、服务器验证通过后，在当前对话（session）里面保存相关数据，比如用户角色、登录时间等等。
3、服务器向用户返回一个 session_id，写入用户的 Cookie。
4、用户随后的每一次请求，都会通过 Cookie，将 session_id 传回服务器。
5、服务器收到 session_id，找到前期保存的数据，由此得知用户的身份。

但是这里有一个大的问题，**假如是服务器集群，则要求 session 数据共享，每台服务器都能够读取 session**

而 `JWT` 是将数据返回给前端的，也就是服务器是无状态的，所以更加容易拓展。


### JWT 的数据结构

`JWT` 的三个部分依次如下:

- Header（头部），类似如下
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```
`alg` 属性表示签名的算法（`algorithm`），默认是 `HMAC SHA256`（写成 `HS256`）。`typ` 属性表示这个令牌（token）的类型（type），JWT 令牌统一写为 `JWT`。
- Payload（负载）。也是一个 JSON，用来存放实际需要传递的数据。JWT 规定了7个官方字段。也可以自定义私有字段。**注意，JWT 默认是不加密的，任何人都可以读到，所以不要把秘密信息放在这个部分。**
>iss (issuer)：签发人
exp (expiration time)：过期时间
sub (subject)：主题
aud (audience)：受众
nbf (Not Before)：生效时间
iat (Issued At)：签发时间
jti (JWT ID)：编号

- Signature（签名）。Signature 部分是对前两部分的签名，防止数据篡改。首先，需要指定一个密钥（`secret`）。这个密钥只有服务器才知道，不能泄露给用户。然后，使用 `Header` 里面指定的签名算法（默认是 `HMAC SHA256`），按照下面的公式产生签名。
```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

算出签名以后，把 `Header`、`Payload`、`Signature` 三个部分拼成一个字符串，每个部分之间用"点"（.）分隔，就可以返回给用户。

![](https://upload-images.jianshu.io/upload_images/1784460-a3be07be9b718511.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### JWT 的特点

（1）`JWT` 默认是不加密，但也是可以加密的。生成原始 `Token` 以后，可以用密钥再加密一次。

（2）`JWT` 不加密的情况下，不能将秘密数据写入 `JWT`。

（3）`JWT` 不仅可以用于认证，也可以用于交换信息。有效使用 `JWT`，可以降低服务器查询数据库的次数。

（4）`JWT` 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 `JWT` 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。

（5）`JWT` 本身包含了认证信息，一旦泄露，任何人都可以获得该令牌的所有权限。为了减少盗用，`JWT` 的有效期应该设置得比较短。对于一些比较重要的权限，使用时应该再次对用户进行认证。

（6）为了减少盗用，`JWT` 不应该使用 `HTTP` 协议明码传输，要使用 `HTTPS` 协议传输。



## Node 简单demo—— Koa JWT 的实现
说完理论知识，我们来看下 Node 如何实现 `JWT`。

![](https://upload-images.jianshu.io/upload_images/1784460-5ed8ba72c6e616d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

首先，用户登录后返回 `token` 给到客户端，前端在下次请求中把 `token` 带给服务器，服务器验证有效后，返回数据。无效的话，返回 `401`

主要技术：
- [jsonwebtoken](https://www.npmjs.com/package/jsonwebtoken)。可以生成 `token`
- [koa-jwt 中间件](https://www.npmjs.com/package/koa-jwt) 对 `jsonwebtoken` 进一步的封装，主要用来校验 `Token`
### 生成 Token
为了演示方便，我这里直接使用 `userList`，存储用户的信息，真实应该是存放在数据库中的。

```js
const crypto = require("crypto"),
  jwt = require("jsonwebtoken");
// TODO:使用数据库
// 这里应该是用数据库存储，这里只是演示用
let userList = [];

class UserController {
  // 用户登录
  static async login(ctx) {
    const data = ctx.request.body;
    if (!data.name || !data.password) {
      return ctx.body = {
        code: "000002", 
        message: "参数不合法"
      }
    }
    const result = userList.find(item => item.name === data.name && item.password === crypto.createHash('md5').update(data.password).digest('hex'))
    if (result) {
      const token = jwt.sign(
        {
          name: result.name
        },
        "Gopal_token", // secret
        { expiresIn: 60 * 60 } // 60 * 60 s
      );
      return ctx.body = {
        code: "0",
        message: "登录成功",
        data: {
          token
        }
      };
    } else {
      return ctx.body = {
        code: "000002",
        message: "用户名或密码错误"
      };
    }
  }
}

module.exports = UserController;
```

通过 `jsonwebtoken` 的 `sign` 方法生成一个 `token`。第一个参数指的是 `Payload`（负载），用于编码后存储在 `token` 中的数据，也是校验 `token` 后可以拿到的数据。第二个是秘钥，服务端特有，注意校验的时候要相同才能解码。第三个参数是 `option`

### 客户端获取 token
前端登录获取到 `token` 后可以存储到 `cookie` 中也可以存放在 `localStorage` 中。

```js
login() {
  this.$axios
    .post("/api/login", {
      ...this.ruleForm,
    })
    .then(res => {
      if (res.code === "0") {
        this.$message.success('登录成功');
        localStorage.setItem("token", res.data.token);
        this.$router.push("/");
      } else {
        this.$message(res.message);
      }
    });
}
```

封装 axios 的拦截器，每次请求的时候把 token 带在请求头发送给服务器进行验证。

```js
// axios 请求拦截器处理请求数据
axios.interceptors.request.use(config => {
  const token = localStorage.getItem('token');
  config.headers.common['Authorization'] = 'Bearer ' + token; // 留意这里的 Authorization
  return config;
})
```


### 校验 token
使用 koa-jwt 中间件进行验证，注意几点
- secret 必须和 sign 时候保持一致
- 可以通过 `unless` 配置接口白名单，也就是哪些 URL 可以不用经过校验，想登陆/注册都可以不用校验。
- 校验的中间件需要放在需要校验的路由前面，要不然无法生效
```js
// 错误处理
app.use((ctx, next) => {
  return next().catch((err) => {
      if(err.status === 401){
          ctx.status = 401;
        ctx.body = 'Protected resource, use Authorization header to get access\n';
      }else{
          throw err;
      }
  })
})

// 注意：放在路由前面
app.use(koajwt({
  secret: 'Gopal_token'
}).unless({ // 配置白名单
  path: [/\/api\/register/, /\/api\/login/]
}))

// routes
app.use(index.routes(), index.allowedMethods())
app.use(users.routes(), users.allowedMethods())
```

### 演示
- 先注册，后登录，不然会提示用户名或者密码错误

![](https://upload-images.jianshu.io/upload_images/1784460-eefcb53675aa986e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 登录后带上 `Authorization`，可以正常访问

![](https://upload-images.jianshu.io/upload_images/1784460-c5eca08f08a32605.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 如果直接访问，则会 `401`

![](https://upload-images.jianshu.io/upload_images/1784460-8acdffa2d33bc46d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 总结
总结了关于 JWT 鉴权相关的知识，并提供了一个 `koa2` 实现的 `demo`，希望对大家有所帮助。

本文 `demo` 地址: [Client](https://github.com/GpingFeng/jwt-client) 和 [Server](https://github.com/GpingFeng/jwt-koa2-demo)



## 参考
- [JSON Web Token 入门教程](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)
- [Node.js 应用：Koa2 使用 JWT 进行鉴权](https://github.com/lin-xin/blog/issues/28)
