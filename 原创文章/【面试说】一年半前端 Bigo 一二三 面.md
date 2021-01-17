## 前言
笔者其实是三月份就面的 `Bigo`，当时工作经验算一年半多。之所以现在才发，其实是之前虽然总结了，但多半是自己总结归纳复盘用，有点粗糙，现在重新整理，希望对大家有所帮助

## 一面
### 说说 `Javascript` 的数据类型

最新的 `ECMAScript`标准 定义了 8 种数据类型

原始数据类型：Boolean、Null、Undefined、Number、BigInt、String、Symbol 

引用类型：对象（`Object`）。其中对象类型包括：数组（`Array`）、函数（`Function`）、还有两个特殊的对象：正则（`RegExp`）和日期（`Date`）

加分项：`BigInt` 和 `Symbol` 类型

`BigInt` 是一种数字类型的数据，它可以表示任意精度格式的整数。由于 `Number` 类型的局限性。Number 类型的局限性（JavaScript 中的 `Number` 是双精度浮点型，这意味着精度有限，如下所示）

```
const max = Number.MAX_SAFE_INTEGER; // 9007199254740991
max + 1 // 9007199254740992
max + 2 // 9007199254740992
```

注意 `max + 1 === max + 2`，这是不对的

`BigInt` 就是解决此类问题




### `Symbol` 类型的使用场景

`Symbol`: 表示独一无二的值，通过 `Symbol` 函数生成，接收一个字符串作为参数，表示对 `Symbol` 实例的描述，主要是为了在控制台显示

应用场景：`Symbol` 的目的就是为了实现一个唯一不重复不可变的值，任何一个 `Symbol` 都是唯一的，不会和其他任何 `Symbol` 相等
  - 对象中保证不同的属性名
      - 注意：使用 `Symbol` 值定义属性的时候，必须放在方括号中
      - 读取的时候也是不能使用点运算符
  - 定义一组常量，保证这组常量都是不相等的
  - 使用 Symbol 定义类的私有属性/方法
  ```js
  const bar = Symbol('bar');
  const snaf = Symbol('snaf');

  export default class myClass{

    // 公有方法
    foo(baz) {
      this[bar](baz);
    }

    // 私有方法
    [bar](baz) {
      return this[snaf] = baz; //私有属性
    }

    // ...
  };
  ```
  - `Vue` 中的 project 和 inject
  
其他应该留意的知识点：
  - 注意属性名的遍历
    - 遍历对象的时候，该属性不会出现在 for...in...、for...of循环中，也不会被Object.keys()、Object.getOwnPropertyNames()、JSON.stringify()返回
    - 可以通过 Object. getOwnPropertySymbols()

  - Symbol.hasInstance
    - 指向一个内部方法。当其他对象使用 instanceof 运算符，判断是否为该对象的实例的时候，会调用这个方法
    - 比如，foo instanceof Foo在语言内部，实际调用的是Foo[Symbol.hasInstance]\(foo\)，有点类似劫持了 instanceof 方法

```js
class MyClass {
  [Symbol.hasInstance](foo) {
    return foo instanceof Array;
  }
}
[1, 2, 3] instanceof new MyClass() // true
```

### `null` 和 `undefined` 的区别

`null` 类型代表着空值，代表着一个空指针对象，`typeof` `null` 会是得到 `'object'` 所以可以认为它是一个特殊的对象值。`undefined` 当你声明一个变量未初始化的时候，得到的就是 `undefined`

   - `typeof` 的值不一样
  ```js
  console.log(typeof undefined); //undefined
  console.log(typeof null); //object
  ```
  - 转为数值时，值不一样
  ```js
  console.log(Number(undefined)); //NaN
  console.log(undefined + 10);//NaN
  console.log(Number(null)); //0
  console.log(null + 10); //10
  ```
  - === 运算符可区分 `null` 和 `undefined`
  - `null` 使用的场景
   	- 作为对象原型链的终点
 `
 Object.getPrototypeOf(Object.prototype)
  // null
`
   - `undefined` 的典型用法 【变量，函数参数，函数返回，对象属性】
### 常见的页面性能优化
### HTTP 上有哪些优化手段
### 重排和重绘的概念，以及如何避免重排和重绘
### TCP 三次握手和 TCP 四次握手的区别

  关于TCP的握手机制，一定不要死记硬背，要理解为什么这么设计，也就很容易记住了

三次握手：

- 在客户端和服务器之间建立正常的TCP网络连接时，客户端首先会发出一个 `SYN` 消息，服务器使用 `SYN+ACK` 应答表示已经接收到这个消息，最后客户端再以 `ACK` 消息响应。这样在客户端和服务器之间才能建立起可靠的 `TCP` 连接，数据才可以在客户端和服务器之间传递

- 建立连接时，客户端发送 `SYN` 包到服务器，等待服务器响应。（`SYN` 同步序列编号，是建立连接时使用的握手信号）

- 服务器收到 `SYN` 包，使用 `ACK` 包进行确认应答，同时自己也会发送一个 `SYN` 包，即发送 `SYN+ACK` 包。
客户端收到服务器的 `SYN` 包，向服务器发送确认包 `ACK`。此包发送完毕，代表 `TCP` 连接完成，完成了三次握手

四次挥手：四次挥手是释放 `TCP` 连接的握手过程

- 客户端向服务端发送释放连接报文 `FIN`，等待服务端确认，并停止发送数据
- 服务器收到连接释放请求后，发送 `ACK` 包表示确认。（此状态下，表示客户端到服务器的连接已经释放，不再接受客户端发的数据了，但是服务器要是还发送数据，客户端依然接收）
- 服务器将最后的数据发送完毕后，就向客户端发送连接释放报文 `FIN`，等待客户端确认。
- 客户端收到服务器连接释放报文后，发出 `ACK` 包表示确认。此时客户端会进入 `TIME_WAIT` 状态，该状态将持续 `2MSL`（最大报文段生存时间，指报文段在网络中生存的时间，超时将被抛弃）时间，若该时间段内没有服务器重发请求的话，就进入关闭状态，当服务端接收到 `ACK` 应答后，立即进入关闭状态

![](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/408e7d0bae984fa68af221606a3b7f13~tplv-k3u1fbpfcp-zoom-1.image)

### webpack 性能优化怎么做的？
### 小程序的数据更新是怎样的？
### 小程序有哪些线程？

![](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d337fa9d9fad4ba799ade4808373182f~tplv-k3u1fbpfcp-zoom-1.image)

   - 小程序的渲染层和逻辑层分别由 2 个线程管理：渲染层的界面使用了 `WebView` 进行渲染，逻辑层采用 JsCore 线程运行 JS 脚本
  - 逻辑层：创建一个单独的线程去执行 JavaScript，在这个环境下执行的都是有关小程序业务逻辑的代码，由于 js 不跑在 `WebView` 里，就不能直接操纵 DOM 和 BOM，这就是小程序没有 window  全局变量的原因
  - 渲染层：界面渲染相关的任务全都在 `WebView` 线程里执行，通过逻辑层代码去控制渲染哪些界面。一个小程序存在多个界面，所以渲染层存在多个 `WebView` 线程
   - 双线程通信【Virtual DOM 相信大家都已有了解，大概是这么个过程：用JS对象模拟DOM树 -> 比较两棵虚拟DOM树的差异 -> 把差异应用到真正的DOM树上。】
   
    1、在渲染层把 WXML 转化成对应的 JS 对象
    
    2、在逻辑层发生数据变更的时候，通过宿主环境提供的 setData 方法把数据从逻辑层传递到 Native，再转发到渲染层
    
    3、经过对比前后差异，把差异应用在原来的 DOM 树上，更新界面

   
![](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2fc2074d1e0240fe8f91bfd288cecca6~tplv-k3u1fbpfcp-zoom-1.image)
  

### 小程序多次设置很多数据的时候会对性能产生很大的影响么？怎么优化
有，可以统一一次设置，多次尽量最后一次设置

### `Vue 2.x` 的实现原理?
回答了双向数据绑定的相关原理

### Vue 的 diff 算法

### Vue 的 compile 过程
  - 主要是三个过程 `parse`，`optimize`，`generate`
  - compile 的作用是解析模板，生成渲染模板的 render。而 render 的作用，也是为了生成跟模板节点一一对应的 Vnode
  - parse: 接收 template 原始模板，按照模板的节点 和数据 生成对应的 ast【通过大量的正则匹配去实现对字符串的解析】
  - Optimize:遍历递归每一个ast节点，标记静态的节点（没有绑定任何动态数据），这样就知道那部分不会变化，于是在页面需要更新时，减少去比对这部分DOM。从而达到性能优化的目的。【为什么要有优化过程，因为我们知道 Vue 是数据驱动，是响应式的，但是我们的模板并不是所有数据都是响应式的，也有很多数据是首次渲染后就永远不会变化的，那么这部分数据生成的 DOM 也不会变化，我们可以在 patch 的过程跳过对他们的比对。】
  - Generate：把前两步生成完善的 ast 组装成 render 字符串（这个 render 变成函数后是可执行的函数，不过现在是字符串的形态，后面会转成函数）

### Vuex简单的原理实现【同时考察了发布订阅的模式】

参考：[面试题：谈谈你对对vuex的理解](https://www.cnblogs.com/LittleStar-/p/9982606.html)

### 移动端有哪些兼容的场景【重点回顾】
  - 1px 问题解决：https://www.jianshu.com/p/3a262758abcf
  - 防止手机中页面放大和缩小
  ```html
  <meta name="viewport" content="width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=1,user-scalable=no" />
  ```
  - 上下拉动滚动条时卡顿、慢。Android3+和iOSi5+支持CSS3的新属性为overflow-scrolling
  ```css
    body {
      -webkit-overflow-scrolling:touch;
      overflow-scrolling:touch;
  }
  ```
     - ios和android下触摸元素时出现半透明灰色遮罩
  ```css
  -webkit-tap-highlight-color:rgba(255,255,255,0);
  ```
  - click 的 `300ms` 延迟问题
  
  问题：在移动端中，click事件是生效的，但是，点击之后会有300ms的延迟响应
  
原因：safari是最早做出这个机制的，因为在移动端里，浏览器需要等待一段时间来判断此次用户操作是单击还是双击，所以就有click 300ms 的延迟机制



方案一：禁用缩放

当 `HTML`文档头部包含如下 `meta` 标签时：表明这个页面是不可缩放的，那双击缩放的功能就没有意义了，此时浏览器可以禁用默认的双击缩放行为并且去掉 `300ms` 的点击延迟

```html
<meta name="viewport" content="user-scalable=no">
<meta name="viewport" content="initial-scale=1,maximum-scale=1">
```
方案二：更改默认的视口宽度

```html
<meta name="viewport" content="width=device-width">
```
如果设置了上述meta标签，那浏览器就可以认为该网站已经对移动端做过了适配和优化，就无需双击缩放操作了
  这个方案相比方案一的好处在于，它没有完全禁用缩放，而只是禁用了浏览器默认的双击缩放行为，但用户仍然可以通过双指缩放操作来缩放页面。

  
方案三：引入 `fastclick` 库来解决

`FastClick` 的实现原理是在检测到 `touchend` 事件的时候，会通过 `DOM` 自定义事件立即出发模拟一个click事件，并把浏览器在300ms之后的click事件阻止掉。


参考：[https://www.jianshu.com/p/b5c103a9bed0](https://www.jianshu.com/p/b5c103a9bed0)


- 水平居中和垂直居中

- node有使用过么

## 二面

### HTTP 中的 content-encoding

`Content-Encoding` 是一个实体消息首部，用于对特定媒体类型的数据进行压缩。当这个首部出现的时候，它的值表示消息主体进行了何种方式的内容编码转换。这个消息首部用来告知客户端应该怎样解码才能获取在 `Content-Type` 中标示的媒体类型内容。

```js
Content-Encoding: gzip
Content-Encoding: compress
Content-Encoding: deflate
Content-Encoding: identity
Content-Encoding: br
```

参考：[https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Encoding](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Encoding)

### `HTTP` 中的长连接

`HTTP` 协议采用“请求-应答”模式，当使用普通模式，即非 `KeepAlive` 模式时，每个请求/应答客户和服务器都要新建一个连接，完成 之后立即断开连接（`HTTP` 协议为无连接的协议）；当使用 `Keep-Alive` 模式（又称持久连接、连接重用）时，`Keep-Alive` 功能使客户端到服 务器端的连接持续有效，当出现对服务器的后继请求时，`Keep-Alive` 功能避免了建立或者重新建立连接

`http 1.0` 中默认是关闭的，需要在 `http` 头加入 `"Connection: Keep-Alive"`，才能启用 `Keep-Aliv`e；`http 1.1` 中默认启用 `Keep-Alive`，如果加入 "Connection: close "，才关闭。目前大部分浏览器都是用 http1.1 协议，也就是说默认都会发起 Keep-Alive 的连接请求了，所以是否能完成一个完整的 `Keep- Alive` 连接就看服务器设置情况

参考：[https://blog.csdn.net/gt11799/article/details/41147933](https://blog.csdn.net/gt11799/article/details/41147933)
[https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Keep-Alive](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Keep-Alive)


### 说下 JS 中的事件循环

### 强缓存和协商缓存

### 启发缓存有了解过么
- 没有任何关于缓存的字段 —— 不设置任何缓存策略
- 常会取响应头中的 `Date` 减去 `Last-Modified` 值的 10% 作为缓存时间

参考：
- [浏览器缓存策略](https://youyingjie114.github.io/2019/11/08/FE-basic/knowledge-network/%E6%B5%8F%E8%A7%88%E5%99%A8%E7%BC%93%E5%AD%98%E6%9C%BA%E5%88%B6/#%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E6%9C%89Etag)



### V8 中的垃圾回收机制
参考：[【Web技术】737- 深入理解 Chrome V8垃圾回收机制](https://mp.weixin.qq.com/s/8GKiSNPBe_FluNc4Ii4PeQ)
### node 的全局变量有哪些

`JavaScript` 中有一个特殊的对象，称为全局对象（`Global Object`），它及其所有属性都可以在程序的任何地方访问，即全局变量

在浏览器 JavaScript 中，通常 window 是全局对象， 而 Node.js 中的全局对象是 `global`，所有全局变量（除了 global 本身以外）都是 `global` 对象的属性。

在 Node.js 我们可以直接访问到 global 的属性，而不需要在应用中包含它。
global，process，\__filename，\__dirname

参考：[Node.js 全局对象](https://www.runoob.com/nodejs/nodejs-global-object.html)

### node 了解程度

### ES6 有了解哪些？

### set 和 map 有了解么？

### map 的实现原理是什么？

`Map` 利用链表，`hash` 的思想来实现。

首先，Map可以实现删除，而且删除的数据可以是中间的值。而链表的优势就是在中间的任意位置添加，删除元素都非常快，不需要移动其他元素，直接改变指针的指向就可以。而在存储数据很多的情况下，会导致链条过长，导致查找效率慢，所以我们可以创建一个桶（存储对象的容器），根据 `hash`（把散列的值通过算法变成固定的某值）来平局分配数据，防止链条过长。

参考：
- [ES6 Map 原理分析](https://www.php.cn/js-tutorial-436743.html)
- [ES6 Map 原理](https://www.cnblogs.com/jiaobaba/p/11918975.html)



### 为什么要想着离职呢？

### 算法题

输入[1,3,1,3,2]，输出数组中唯一一个只存在一项的值，比如如上就是 2

![](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b4642f035a72422fb1b64a772dcd7ea7~tplv-k3u1fbpfcp-zoom-1.image)

![](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8308bb9e51c54ec7986a58f2054c883b~tplv-k3u1fbpfcp-zoom-1.image)

![](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3131385892a349778addb7e2f923942d~tplv-k3u1fbpfcp-zoom-1.image)


## 三面

### 谈谈你的项目【其中针对项目进行了提问】
### 说说 webpack  打包优化
### 从一个数组中拿到前三个最小的值
### 一道编程题
```
[1,2,3,1,2,4,1,3,2,1] [1,2] [3,4,5] =>[3,4,5,3,3,4,5,4,1,3,2,1]

说明：第一个输入是原始数组，第二个输入是一个条件：要在原始数组中的连续数组，第三个输入是要替换掉原数组符合参数二条件的数据

能够使用的api：
1. 数组的长度获取。
2. 数组某个下标对应的数字。
3. 往数组里push元素。
```


## 总结

归纳总结了 Bigo 一二三面的知识，有一些问题总结回答建议，有一些没有，希望对大家有帮助

## 参考

- [JavaScript 中的表示任意精度的 BigInt](https://juejin.im/post/6844903795797786632)

- [小程序的线程架构](https://www.cnblogs.com/idreamo/p/10853965.html)

- [小程序学习笔记-双线程](https://www.jianshu.com/p/fb331438e223)
- [【Vue原理】Compile - 白话版](https://blog.csdn.net/qq_27460969/article/details/98947331)
- [移动端页面兼容性问题解决方案整理（一）](https://www.cnblogs.com/changningios/p/6486610.html)
- [关于移动端适配，你必须要知道的](https://juejin.im/post/5cddf289f265da038f77696c#heading-10)
- [移动端webApp兼容问题解决](https://blog.csdn.net/quanyuejie/article/details/53422081)



