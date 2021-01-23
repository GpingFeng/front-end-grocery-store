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

而 `JWT` 是将数据返回给前端的，也就是服务器是无状态的，所以更加容易拓展，也不会存在上面的问题


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


## Node 简单实现——使用 JWT 进行鉴权
说完理论知识，我们来实现一个 `JWT`