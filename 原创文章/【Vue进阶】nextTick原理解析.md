## nextTick 官方解释

> 语法：Vue.nextTick( [callback, context] )
- `{Function} [callback]`
- `{Object} [context]`

> 在下次 `DOM` 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 `DOM`

```js
// 修改数据
vm.msg = 'Hello'
// DOM 还没有更新
Vue.nextTick(function () {
  // DOM 更新了
})

// 作为一个 Promise 使用 (2.1.0 起新增，详见接下来的提示)
Vue.nextTick()
  .then(function () {
    // DOM 更新了
  })
```



## nextTick 应用场景

## nextTick 源码解析