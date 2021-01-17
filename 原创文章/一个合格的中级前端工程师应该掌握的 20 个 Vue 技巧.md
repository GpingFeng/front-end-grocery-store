
## 初衷

之前写过一篇文章，关于 `Vue` 属性透传的，文章中我列举了很多种方法去实现属性透传。其中包括直接设置 `props`，`v-bind="$attrs"`，`render function` 等方式。感兴趣，详情看 [【Vue进阶】——如何实现组件属性透传？
](https://juejin.im/post/6865451649817640968)

不过后来有个掘友给我留言，其实 `v-bind="obj"` 的方式就可以直接传递展开后对象的所有属性的，试了一下，是可以的

这让我意识到了自己对一些 `Vue` 技巧掌握不足，所有有了这篇文章。**通过这篇文章，我总结很多能够帮助我们提高开发效率的 `Vue` 技巧，同时也指出这些技巧的使用场景以及使用注意事项，我坚信对于 `Vue` 开发者有一定的帮助**

## 1.通过 v-bind="\$props" 以及v-bind="$attrs" 实现属性透传

很多时候，我们会写一些嵌套组件，比如 A 的子组件是 B，B 的子组件是 C。这个时候如果 A 传递 `props` 给 B，B 又得传递 `props` 给 C，我们经常在 B 传给 C 的时候这么写

```html
<template>
  <child-component :someprop1="someprop1"
                   :someprop2="someprop2"
                   :someprop3="someprop3"
                   :someprop4="someprop4"
                   ...
  />
</template>
```

这样是很不优雅的，其实你可以直接使用 `v-bind: $props`

```html
<template>
  <child-component v-bind="$props"/>
</template>
```

这里我们利用 `v-bind` 可以传入一个对象的所有 `property`，类似 `v-bind="Obj"`。例如，对于一个给定的对象 post

```js
post: {
  id: 1,
  title: 'My Journey with Vue'
}
```

下面的模板：

```html
<blog-post v-bind="post"></blog-post>
```

等价于：

```html
<blog-post
  v-bind:id="post.id"
  v-bind:title="post.title"
></blog-post>
```


这个配合 `v-bind="$attrs"` 在封装一些组件的时候非常有用，比如实现属性透传。

`vm.$attrs` 包含了父作用域中不作为 `prop` 被识别 (且获取) 的 `attribute` 绑定 (class 和 style 除外)。当一个组件没有声明任何 prop 时，这里会包含所有父作用域的绑定 (class 和 style 除外)，并且可以通过 `v-bind="$attrs"` 传入内部组件——在创建高级别的组件时非常有用。

比如将上面传递进来的 `props` 全部绑定到 `el-input` 中,我们可以在子组件中这么写：

```html
<template>
  <div>
    <el-input v-bind="$attrs" ></el-input>
  </div>
</template>
```


详情具体可以看[【Vue进阶】——如何实现组件属性透传？](https://juejin.im/post/6865451649817640968)


> 有同学可能想到了 `provide` 和 `inject`，确实也是可以的传递 `props`，却做不到属性透传，而且 `provide` 和 `inject` 绑定并不是可响应的，这一点需要额外注意一下。这是刻意为之的。然而，如果你传入了一个可监听的对象，那么其对象的属性还是可响应的。

## 2.两个 component 接受同样的 props
上面例子中，要求组件 B 和组件 C 接受同样的的 `props`，其中 B 组件写法如下（C 组件类似）：

```html
<template>
  <child-component v-bind="$props"/>
</template>

<script>
  import ChildComponent from '@/components/ChildComponent'
  
  export default {
    props:{
      someProp1: String,
      someProp2: String,
      someProp3: String,
      // and so on
    }
  }
</script>
```

但这样有个问题，就是 C 组件修改了 `props`，那么 B 组件得一起修改，这样做的一个坏处有可能会有遗漏。另外就是代码冗余，看起来不精简。其实我们可以在 B 组件中这么写

```html
<template>
  <child-component v-bind="$props"/>
</template>

<script>
  import ChildComponent from '@/components/ChildComponent'
  
  export default {
    props:{
      ...ChildComponent.options.props
    }
  }
</script>
``` 

## 3.Props  校验

由于 `Javascript` 是弱类型语言，在写 `props` 的时候，最佳实践是对 `props` 使用 `type` 指定类型以及设定默认值，如下：

```js
Vue.component('my-component', {
    // 带有默认值的对象
    propA: {
      type: Object,
      // 对象或数组默认值必须从一个工厂函数获取
      default: function () {
        return { message: 'hello' }
      }
    }
})
```

但是有可能我们不知道的是，`props` 可以自定义验证函数
```js
Vue.component('my-component', {
    // 自定义验证函数
    propF: {
      validator: function (value) {
        // 这个值必须匹配下列字符串中的一个
        return ['success', 'warning', 'danger'].indexOf(value) !== -1
      }
    }
})
```

甚至上面的 `type` 除了可以设置 `String` `Number` `Boolean` `Array` `Object` `Date` `Function` `Symbol` 之外，还可以自定义构造函数，其底层实现原理是通过 `instanceof` 去判断

```js
function Person (firstName, lastName) {
  this.firstName = firstName
  this.lastName = lastName
}

Vue.component('blog-post', {
  props: {
    author: Person
  }
})
```

这样来验证 `author prop` 的值是否是通过 `new Person` 创建的

## 4.透传所有事件监听

有时候，我们需要对一些开源库的表单组件，比如 `elementUI` 的 `form`  进行一层包装，让它更好的为我们的业务服务，但是一旦这么包装，就出现一个问题，调用的时候如何监听到内部 `form` 组件暴露出来的所有事件呢？总不能一个个的绑定吧

这个时候，可以在内部 `form` 组件设置 `v-on="$listeners"`

`$listeners` 包含了父作用域中的 (不含 `.native` 修饰器的)  `v-on` 事件监听器。它可以通过 `v-on="$listeners"` 传入内部组件——在创建更高层次的组件时非常有用，上面的例子可以很好解决如下：

```html
<template>
  <div>
    ...
    <el-form v-on="$listeners"/>
    ...
  </div>
</template>
```

当然如果子组件是父组件的根元素，则不需要这么写，`Vue` 默认已经做了这一层的操作了

## 5.作用域插槽实现 UI 和业务逻辑的分离

很多时候，我们想复用一个组件的业务逻辑，但是不想使用该组件的 `UI`，那么可以使用作用域插槽实现 `UI` 和业务逻辑的分离。

作用域插槽大致的思路是将 `DOM` 结构交给调用方去决定，组件内部只关注业务逻辑，最后将数据和事件等通过 `:item ="item"` 的方式传递给父组件去处理和调用，实现 UI 和业务逻辑的分离。再结合渲染函数，就可以实现无渲染组件的效果

具体可以看我的另一篇文章 [【Vue 进阶】从 slot 到无渲染组件](https://juejin.im/post/6869537683736100871)

其中父组件调用的时候可以类似这样，其中 `#row` 是 `v-slot:row` 的缩写

```html
<!-- 父组件 -->
<template>
  ...
  <my-table>
    <template #row="{ item }">
      /* some content here. You can freely use 'item' here */
    </template>
  </my-table>
  ...
</template>
```

```html
<!-- 子组件 -->
<span>
  <!-- 使用类似 v-bind:item="item"，将子组件中的事件或者data传递给父组件-->
  <slot v-bind:item="item">
    {{ item.lastName }}
  </slot>
</span>
```

> 需要注意，Vue 2.6 之前使用的是 `slot` 和 `slot-scope`，后面使用的是 `v-slot`

## 6.动态的指令参数
在 `Vue 2.6` 中提供了这样一个特性：可以动态的将指令参数传递给组件。假设你有一个组件 ` <my-button>`，有时候你需要绑定一个点击事件 `click`，有时候需要绑定一个双击事件 `dblclick`，这个时候你可以这么写

```html
<template>
  ...
  <my-button @[someEvent]="handleSomeEvent()"/>
  ...
</template>

<script>
  ...
  data(){
    return{
      ...
      someEvent: someCondition ? "click" : "dblclick"
    }
  },
  
  methods:{
    handleSomeEvent(){
      // do something
    }
  }
  ...
</script>
```

## 7.hookEvent 的使用

在 `Vue 2.X` 当中，`hooks` 可以作为一种 `Event`，在 `Vue` 的源码当中，称之为 `hookEvent`。 利用它，我们可以模板声明式的监听子组件的生命周期钩子，从而可以给第三方组件添加生命周期处理函数

比如，我们调用了一个很耗费性能的第三方组件 `List`，这个组件可能需要渲染很久，为了更好的用户体验，我们想在 `List ` 组件进行更新的时候添加一个 `loading` 的动画效果

这个时候，我们可能会想到直接修改这个组件的源码，利用 `beforeUpdate` 和 `updated` 来显示 `loading`，但是这种办法非常不好。第一修改成本比较大，第二无法享受开源社区对这个组件的升级与维护，你需要自己手动维护

这个时候就可以通过 `hookEvent` 模板声明式的注入声明周期钩子函数，类似如下：

```html
<List @hook:updated="handleTableUpdated"></List >
```

另外，我们还可以通过下面的方式给一个 `Vue` 组件添加生命周期处理函数

```
vm.$on('hooks:created', cb)
vm.$once('hooks:created', cb)
```

比如我们想在组件销毁的时候，去销毁之前调用的组件，一般我们会这么做：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f5e3d035a10248d9be6dc06815151306~tplv-k3u1fbpfcp-zoom-1.image)

但是这么做代码的可读性是不好的，我们可以修改成以下的方式，优化我们代码的可读性，精简我们的代码，这也是 `Vue 3` 在致力解决的问题

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/14a19fd037b247d09ae4368a7f6c6d33~tplv-k3u1fbpfcp-zoom-1.image)


## 8.key 值的使用

在 `Vue` 中，使用 `v-for`，官方建议带上 `key` 值，因为如果不使用 `key`，`Vue` 默认会使用一种“就地复用”的策略进行更新。在一些情况下，很有可能会导致渲染不正确，之前总结过一篇 [使用 key 不当踩坑的经历](https://juejin.im/post/6844903865930743815)，感兴趣可以看下

除了 `v-for`， 在使用 `Vue-router` 做项目时，会遇到如 `/path/:id` 这样只改变 `id` 号的场景，但渲染不同的组件。由于 `router-view` 是复用的，单纯的改变 `id` 号并不会刷新 `router-view`，这并不是我们所期望的结果

这个时候，我们可以给每个 `router-view` 添加一个不相同 `key` 值，让 `Vue` 每次切换路由参数的时候，认为是不同的组件，从而得到更新

```html
<router-view :key="key"></router-view>
```

实际上对于所有的 `DOM`，`Vue` 都有可能采取就地复用的策略，所以如果遇到了渲染顺序不正确的问题，可以往 `key` 值设置的方向考虑


## 9.自定义组件使用 v-model

我们知道，`v-model` 是 `v-bind` 以及 `v-on` 配合使用的语法糖，以下的两者的实现是一致的:

```html
<input v-model="value" />
<input v-bind:value="value" v-on:input="value= $event.target.value" />
```

一个组件上的 `v-model` 默认会利用名为 `value` 的 prop 和名为 `input` 的事件，但是像单选框、复选框等类型的输入控件可能会将 `value` attribute 用于不同的目的。

其中 `v-model` 在内部为不同的输入元素使用不同的 `property` 并抛出不同的事件：

- text 和 textarea 元素使用 value property 和 input 事件
- checkbox 和 radio 使用 checked property 和 change 事件
- select 字段将 value 作为 prop 并将 change 作为事件

以上的情况，我们在自定义组件中使用的时候，就需要使用 `model` 选项了，按照官方的示例，写了个 [demo](https://codepen.io/gpingfeng/pen/MWyGryd?editors=1011)

![](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/66685be0287342cf99a4e0e7ffc8bf5c~tplv-k3u1fbpfcp-zoom-1.image)

这里的 `lovingVue` 的值将会传入这个名为 `checked` 的 `prop`。同时当 `<base-checkbox>` 触发一个 `change` 事件并附带一个新的值的时候，这个 `lovingVue` 的 `property` 将会被更新。

## 10.CSS scoded 和深度作用选择器
在 `Vue-loader` 中，当 `<style>` 标签有 `scoped` 属性时，它的 `CSS` 只作用于当前组件中的元素，它通过使用 `PostCSS` 来实现以下转换，可以注意到 `.example` 后面添加了专属于该元素的 [data-v-f3f3eg9]

```html
<style scoped>
.example {
  color: red;
}
</style>

<template>
  <div class="example">hi</div>
</template>
```

```html
<style>
.example[data-v-f3f3eg9] {
  color: red;
}
</style>

<template>
  <!-- 留意data-v-f3f3eg9 -->
  <div class="example" data-v-f3f3eg9>hi</div>
</template>
```

但有时候，我们想设置混用本地和全局样式，可以在一个组件中同时使用有 scoped 和非 scoped 样式

```html
<style>
/* 全局样式 */
</style>

<style scoped>
/* 本地样式 */
</style>
```

但这样全局的样式就有可能产生一些副作用，我们很多时候这么设置，其实只是想设置父组件中子组件的样式。因为父组件设置了 `scoped ` 之后，父组件的样式将不会渗透到子组件中

这个时候，我们可以直接通过深度作用选择器去影响子组件，如下

```html
<style scoped>
.a >>> .b { /* ... */ }
</style>
```
会编译成如下:
```
.a[data-v-f3f3eg9] .b { /* ... */ }
```

有些像 `Sass` 之类的预处理器无法正确解析 `>>>`。这种情况下你可以使用 `/deep/` 或 `::v-deep` 操作符取而代之——两者都是 `>>>` 的别名，同样可以正常工作

## 11.watch immediate
有时候我们想在一个组件中 `watch` 一个值，进行一些初始化页面和更新页面的操作，比如 `this.getDetails()`

类似如下：
```
watch: {
  id: {
    handler(newValue) {
      this.getDetails(newValue);
    }
  }
}
```

这里 `watch` 的一个特点是，最初绑定的时候是不会执行的，要等到 id 改变时才执行监听计算。这可能导致我们页面第一次渲染出错

想要在 `watch` 中声明了 `id` 后立即先执行 `handler` 方法，可以加上 `immediate: true`

```
watch: {
  id: {
    handler(newValue) {
      this.getDetails(newValue);
    },
    // 代表在wacth里声明了id后这个方法之后立即先去执行handler方法
    immediate: true
  }
}
```

## 12.在 Vue 中使用 JSX

`JSX` 是一种 `Javascript` 的语法扩展，JSX = Javascript + XML，即在 `Javascript` 里面写 `XML`，因为 `JSX` 的这个特性，所以他即具备了 `Javascript` 的灵活性，同时又兼具 `html` 的语义化和直观性

有时候，我们使用渲染函数（`render function`）来抽象组件，而渲染函数有时候写起来是非常痛苦的，这个时候我们可以在渲染函数中使用 `JSX` 简化我们的代码。在 Vue 中使用 JSX，需要使用 Babel 插件，它可以让我们回到更接近于模板的语法上，详情可以看我之前总结的一篇文章[【Vue进阶】手把手教你在 Vue 中使用 JSX](https://juejin.im/post/6870480188086419470)

比如常见的指令可以书写如下：

```js
render() {
  {/* 指令 */}
  {/* v-model */}
  <div><input vModel={this.newTodoText} /></div>
  {/* v-model 以及修饰符 */}
  <div><input vModel_trim={this.tirmData} /></div>
  {/* v-on 监听事件 */}
  <div><input vOn:input={this.inputText} /></div>
  {/* v-on 监听事件以及修饰符 */}
  <div><input vOn:click_stop_prevent={this.inputText} /></div>
  {/* v-html */}
  <p domPropsInnerHTML={html} />
}
```

## 13.v-cloak 解决页面闪烁问题
很多时候，我们页面模板中的数据是异步获取的，在网络不好的情况下，渲染页面的时候会出现页面闪烁的效果，影响用户体验，`v-cloak` 指令保持在元素上直到关联实例结束编译，利用它的特性，结合 `CSS` 的规则 `[v-cloak] { display: none }` 一起使用就可以隐藏掉未编译好的 `Mustache` 标签，直到实例准备完毕

```html
// template 中
<div class="#app" v-cloak>
    <p>{{value.name}}</p>
</div>

// css 中
[v-cloak] {
    display: none;
}
```

> 需要注意，虽然解决了闪烁的问题，但这段时间内如果什么都不处理的话，会直接白屏，这并不是我们想要的效果，我们应该加一个 `loading` 或者骨架屏的效果，提升用户体验

## 14.v-once 和 v-pre 提升性能
我们知道 `Vue` 的性能优化很大部分在编译这一块，`Vue` 源码就有类似标记静态节点的操作，以在 `patch` 的过程中跳过编译，从而提升性能。另外，`Vue` 提供了 `v-pre` 给我们去决定要不要跳过这个元素和它的子元素的编译过程。可以用来显示原始 `Mustache` 标签。跳过大量没有指令的节点会加快编译。

```html
<span v-pre>{{ this will not be compiled }}</span>   显示的是{{ this will not be compiled }}
<span v-pre>{{msg}}</span>     即使data里面定义了msg这里仍然是显示的{{msg}}
```

另外，如果只渲染元素和组件一次。随后的重新渲染，元素/组件及其所有的子节点将被视为静态内容并跳过。可以使用 `v-once`

```html
<!-- 单个元素 -->
<span v-once>This will never change: {{msg}}</span>
<!-- 有子元素 -->
<div v-once>
  <h1>comment</h1>
  <p>{{msg}}</p>
</div>
<!-- 组件 -->
<my-component v-once :comment="msg"></my-component>
<!-- `v-for` 指令-->
<ul>
  <li v-for="i in list" v-once>{{i}}</li>
</ul>
```

## 15.函数式组件

函数式组件指的是无状态,无法实例化，内部没有任何生命周期处理方法的组件。因为函数式组件只是函数，所以渲染开销也低很多。可以通过声明 `functional: true`，表明它是一个函数式组件

在作为包装组件的时候，它们是非常有用的

- 程序化地在多个组件中选择一个来代为渲染
- 在将 `children`、`props`、`data` 传递给子组件之前操作它们

看官方的一个 demo，留意注释

```js
// 根据不同的情况渲染不同的组件
var EmptyList = { /* ... */ }
var TableList = { /* ... */ }
var OrderedList = { /* ... */ }
var UnorderedList = { /* ... */ }

Vue.component('smart-list', {
  functional: true, // 声明 functional: true，表明它是一个函数式组件
  props: {
    items: {
      type: Array,
      required: true
    },
    isOrdered: Boolean
  },
  // 为了弥补缺少的实例
  // 提供第二个参数作为上下文
  render: function (createElement, context) { // 组件中所有的一切都是通过 context 传递的
  	// 根据不同的情况渲染不同的组件
    function appropriateListComponent () {
      var items = context.props.items

      if (items.length === 0)           return EmptyList
      if (typeof items[0] === 'object') return TableList
      if (context.props.isOrdered)      return OrderedList

      return UnorderedList
    }

    return createElement(
      appropriateListComponent(),
      context.data,	// 传递给组件的整个数据对象
      context.children // `VNode` 子节点的数组
    )
  }
})
```

留意下，组件中所有的一切都是通过 `context` 传递的(`render` 函数的第二个参数)，比如上面通过 `context.data` `context.children` 分别代表传递给组件的整个数据对象，作为 `createElement` 的第二个参数传入组件和`VNode` 子节点的数组，详细的 `context` 中包含的内容见[官网](https://cn.vuejs.org/v2/guide/render-function.html#%E5%87%BD%E6%95%B0%E5%BC%8F%E7%BB%84%E4%BB%B6)

## 16.使用 Vue.observable 实现状态共享
众所众知，`Vuex` 就是专门用来解决多组件状态共享的情况，不过就像 `Vuex` 官方文档所说的，如果应用不够大，为避免代码繁琐冗余，最好不要使用它

`Vue.observable( object )` 让一个对象可响应。`Vue` 内部会用它来处理 `data` 函数返回的对象。返回的对象可以直接用于渲染函数和计算属性内，并且会在发生变更时触发相应的更新。也可以作为最小化的跨组件状态存储器，用于简单的场景，有点类似小型的 `Vuex`

我们新建一个文件 `store.js`

```js
import Vue from "vue";

// 创建一个小型的 store，里面的数据可以实现多组件共享
export const store = Vue.observable({ count: 0 });

// 模糊VueX 的 mutation
export const mutations = {
  setCount(count) {
    store.count = count;
  }
};
```

在组件中使用 `store` 中的值以及更新 `store` 中的值，具体的可以看 [demo](https://codesandbox.io/s/frosty-driscoll-0pjv9?file=/src/App.vue:0-516)

```html
<template>
  <div id="app">
    <p>count:{{count}}</p>
    <button @click="setCount(count+1)">+1</button>
    <button @click="setCount(count-1)">-1</button>
  </div>
</template>

<script>
import HelloWorld from "./components/HelloWorld";
import { store, mutations } from "./store";

export default {
  name: "App",
  components: {
    HelloWorld
  },
  // 获取 store 中的数据
  computed: {
    count() {
      return store.count;
    }
  },
  // 更改 store 中的数据
  methods: {
    setCount: mutations.setCount
  }
};
</script>
```

## 17.表单输入控制——表单修饰符/change事件/filter/指令
我们经常遇到控制表单输入内容的需求，比如输入框内一定是是数字，不能有特殊字符等等。这里我提供一些自己的一些思路，供大家选择使用

### 表单修饰符

如果是简单的控制输入一定是数字或者去掉用户输入的收尾空白符，可以直接使用 `Vue` 提供的表单修饰符 `.number` 和 `.trim`

如果想自动将用户的输入值转为数值类型，可以给 `v-model` 添加 `number` 修饰符：

```html
<input v-model.number="age" type="number">
```

如果要自动过滤用户输入的首尾空白字符，可以给 `v-model` 添加 `trim` 修饰符：

```html
<input v-model.trim="msg">
```

### change事件
给表单绑定事件，在事件处理中进行表单输入控制

```html
<input v-model="value2" type="text" @change="inputChange(value2)" />
```

```js
methods: {
  inputChange: function(val) {
    if (!val) return ''
    val = val.toString()
    this.value2 = val.charAt(0).toUpperCase() + val.slice(1)
  }
}
```

### filter
还可以通过过滤器 `filter` 进行

```
<input v-model="value1"  type="text" />
```

```js
Vue.filter('capitalize', function (value) {
  if (!value) return ''
  value = value.toString()
  return value.charAt(0).toUpperCase() + value.slice(1)
})
```

```
watch: {
  value1(val) {
     this.value1 = this.$options.filters.capitalize(val);
  }
}
```

详情可以看这个 [demo](https://codepen.io/gpingfeng/pen/XWdqyPO?editors=1111)

### 指令

声明一个全局的指令

```js
// 只能输入正整数,0-9的数字
Vue.directive('enterIntNumber', {
  inserted: function (el) {
    let trigger = (el, type) => {
      const e = document.createEvent('HTMLEvents')
      e.initEvent(type, true, true)
      el.dispatchEvent(e)
    }
    el.addEventListener("keyup", function (e) {
      let input = e.target;
      let reg = new RegExp('^\\d{1}\\d*$');  //正则验证是否是数字
      let correctReg = new RegExp('\\d{1}\\d*');  //正则获取是数字的部分
      let matchRes = input.value.match(reg);
      if (matchRes === null) {
        // 若不是纯数字 把纯数字部分用正则获取出来替换掉
        let correctMatchRes = input.value.match(correctReg);
        if (correctMatchRes) {
          input.value = correctMatchRes[0];
        } else {
          input.value = "";
        }
      }
      trigger(input, 'input')
    });
  }
});
```

使用指令控制

```html
<!--限制输入正整数-->
<input v-enterIntNumber placeholder="0" type="number">
```


## 18.事件：特殊变量 $event



**原生事件**

解决场景：有时候，我们绑定事件后，想传入除了原生事件对象之外的其他参数

在监听原生 `DOM` 事件时，方法以原生事件对象为唯一的参数（默认值）。很多时候，我们想要在内联处理器中访问原始的 `DOM` 事件（而且同时想传其他参数），可以使用 `$event` 把它传入。

```html
<!-- 注意这里调用的方法有两个参数 -->
<input v-model="value1" @change="inputChange('hello', $event)">
```

```
methods: {
  inputChange(msg, e) {
    console.log(msg, e);
  }
}
```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b0a3577e4ee94b9f811d822e160cfb88~tplv-k3u1fbpfcp-zoom-1.image)

**自定义事件**

在自定义事件中，`$event` 是从其子组件中捕获的值

场景：你想监听 `el-input` 的传递过来的值的同时，传递其他的参数。

```html
<el-input
      v-model="value2"
      @change="change($event, 'hello')"
      placeholder="Input something here"
    />
```

```
methods: {
  change(e, val) {
    console.log("event is " + e); // el-input 输入的值
    console.log(val); // hello
  }
}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/092e695b64c2459eaaca5d95f28b000f~tplv-k3u1fbpfcp-zoom-1.image)

详情可看这个 [demo](https://codesandbox.io/s/patient-moon-vuh0o?file=/src/components/HelloWorld.vue:128-248)

## 19.调试 template
很多时候，我们会遇到 `template` 模板中变量报错的问题，这个时候，我们很想通过 `console.log` 打印到控制台，看它的值是什么

```js
// 这里最好是判断一下，只有在测试环境中才使用
// main.js
Vue.prototype.$log = window.console.log;

// 组件内部
<div>{{$log(info)}}</div>
```
可见 [demo](https://codepen.io/gpingfeng/pen/BaKxeEN?editors=1011)

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/14dbbd655ac04d18b8905f5cbe2cef7e~tplv-k3u1fbpfcp-zoom-1.image)


## 20.开发调试 Vue 必备的工具——vue-devtools
`vue-dev-tool` 是 `Vue` 官方开源的调试神器，利用它可以很清楚的查看组件层级，组件数据变化，`Vuex` 数据以及事件等等。特别是对于一些复杂应用，决定能够提升你的工作效率

可以科学上网的见[地址1](https://chrome.google.com/webstore/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd?hl=zh-CN)

不能的见[地址2](https://github.com/vuejs/vue-devtools) 里面有手动安装教程

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1e7e7a2f9cb244de999b34b217361aaf~tplv-k3u1fbpfcp-zoom-1.image)

另外，`vscode` 的插件 `Vetur` 也一起推荐一下，支持语法高亮，代码块等等功能，个人觉得使用 `vscode` 的 `Vue` 开发者必备



## 总结
`Vue` 的学习门槛相对比较低，我们掌握一些常见的用法就能解决我们日常大部分的问题。但掌握上面全部的 `Vue` 技巧，你肯定会有一种全新的体验

“原来 Vue 还可以这么玩”，如果你有这样的感觉，恭喜你，尝试使用这些技巧去解决你的一些问题吧，肯定可以达到事半功倍的效果

## 往期优秀文章推荐

- [【Vue进阶】——如何实现组件属性透传？](https://juejin.im/post/6865451649817640968)
- [前端应该知道的 HTTP 知识【金九银十必备】](https://juejin.im/post/6864119706500988935)
- [最强大的 CSS 布局 —— Grid 布局](https://juejin.im/post/6854573220306255880)
- [如何用 Typescript 写一个完整的 Vue 应用程序](https://juejin.im/post/6860703641037340686)
- [前端应该知道的web调试工具——whistle](https://juejin.im/post/6861882596927504392)


## 参考
- [hookEvent of Vue](https://juejin.im/post/6844903974999425037)

- [10 Tips and Tricks to Make You a Better Vue.js Developer](https://medium.com/better-programming/10-tips-and-tricks-to-make-you-a-better-vue-js-developer-afc74acaf388)

- [7个有用的Vue开发技巧](https://juejin.im/post/6844903848050589704#heading-1)