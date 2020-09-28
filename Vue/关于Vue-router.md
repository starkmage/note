## 前端路由与后端路由

之前已经总结过一篇文章《前端渲染与后端渲染》

## vue-router 是什么

vue-router是Vue.js官方的路由插件，它和vue.js是深度集成的，适合用于构建单页面应用。vue的单页面应用是基于路由和组件的，路由用于设定访问路径，并将路径和组件映射起来。传统的页面应用，是用一些超链接来实现页面切换和跳转的。在vue-router单页面应用中，则是路径之间的切换，也就是组件的切换。**路由模块的本质就是建立起url和组件之间的映射关系**。

## vue-router 实现方式

**单页面应用(SPA)的核心之一是: 更新视图而不重新请求页面**。vue-router在实现单页面前端路由时，提供了两种方式：Hash模式和History模式；根据mode参数来决定采用哪一种方式。

### hash 模式

**vue-router 默认 hash 模式 —— 使用 URL 的 hash 来模拟一个完整的 URL，于是当 URL 改变时，页面不会重新加载。** 

hash（#）是URL 的锚点，代表的是网页中的一个位置，单单改变#后的部分，浏览器只会滚动到相应位置（或者说加载相应的组件），不会重新加载网页，也就是说**hash 出现在 URL 中，但不会被包含在 http 请求中，对后端完全没有影响，因此改变 hash 不会重新加载页面**。

同时每一次改变#后的部分，都会在浏览器的访问历史中增加一个记录，使用”后退”按钮，就可以回到上一个位置；所以说**Hash模式通过锚点值的改变，根据不同的值，渲染指定DOM位置的不同数据。**

**hash 模式的原理是 onhashchange 事件(监测hash值变化)，可以在 window 对象上监听这个事件**。

### history 模式

只需要在配置路由规则时，加入"mode: 'history'"。

**这种模式充分利用了html5 history interface 中新增的 pushState() 和 replaceState() 方法。这两个方法应用于浏览器记录栈，在当前已有的 back、forward、go 基础之上，它们提供了对历史记录修改的功能。只是当它们执行修改时，虽然改变了当前的 URL ，但浏览器不会立即向后端发送请求**。

但是，当刷新页面的时候，则在 http 请求中会带上整个 url 去向服务端请求，所以，这种模式还需要服务端有相应的配置，比如设置如果URL输入错误或者是URL 匹配不到任何静态资源，就自动跳到到Home页面。

### 跳转方式

- 方式1：直接修改地址栏
- 方式2：this.$router.push(‘路由地址’)
- 方式3：`<router-link to="路由地址"></router-link>`

## vue-router 参数传递

声明式的导航`<router-link :to="...">`和编程式的导航`router.push(...)`都可以传参，本文主要介绍前者的传参方法，同样的规则也适用于编程式的导航。

### 1. 使用 name + params 传参

这种传参方法的基本语法：

```vue
<router-link :to="{name:xxx,params:{key:value}}">valueString</router-link>
```

比如先在src/App.vue文件中

```vue
<router-link :to="{name:'hi1',params:{username:'jspang',id:'555'}}">Hi页面1</router-link>
```

然后把src/router/index.js文件里给hi1配置的路由起个name,就叫hi1.

```js
{path:'/hi1',name:'hi1',component:Hi1}
```

最后在模板里(src/components/Hi1.vue)用`$route.params.username`进行接收.

```js
{{$route.params.username}}-{{$route.params.id}}
```

### 2. 使用 name/path + query 来传递参数

```vue
<router-link :to="{ path: '/query', query: { queryId: status }}" >
     router-link跳转Query
</router-link>

<router-link :to="{ name: 'Query', query: { queryId: status }}" >
     router-link跳转Query
</router-link>
```

对应路由配置：

```js
   {
     path: '/query',
     name: 'Query',
     component: Query
   }
```

于是我们可以获取参数：

```js
this.$route.query.queryId
```

### 3 利用url传递参数----在配置文件里以冒号的形式设置参数

我们在/src/router/index.js文件里配置路由

```
{
    path:'/params/:newsId/:newsTitle',
    component:Params
}
```

我们需要传递参数是新闻ID（newsId）和新闻标题（newsTitle）.所以我们在路由配置文件里制定了这两个值。

在src/components目录下建立我们params.vue组件，也可以说是页面。我们在页面里输出了url传递的的新闻ID和新闻标题。

```vue
<template>
    <div>
        <h2>{{ msg }}</h2>
        <p>新闻ID：{{ $route.params.newsId}}</p>
        <p>新闻标题：{{ $route.params.newsTitle}}</p>
    </div>
</template>
<script>
export default {
  name: 'params',
  data () {
    return {
      msg: 'params page'
    }
  }
}
</script>
```

在App.vue文件里加入我们的`<router-view>`标签。这时候我们可以直接利用url传值了

```vue
<router-link to="/params/198/jspang website is very good">params</router-link>
```

### query 和 params 的区别

1. 用法
   **query可以用path来引入，也可以用name引入，params要用name来引入。**接收参数都是类似的，分别是this.$route.query.name和this.$route.params.name。

2.  url地址显示
    query更加类似于我们ajax中get传参，params则类似于post，说的再简单一点，前者在浏览器地址栏中显示参数，后者则不显示。

   ```text
   // query
   http://localhost:8080/workorder/newApply?type=BOX_DEPLOY&typeDesc=%E5%B0%8F%E7%99%BD%E7%9B%92%E9%83%A8%E7%BD%B2
   
   // params
   http://localhost:8080/workorder/newApply
   ```

3. 注意点
    query刷新不会丢失query里面的数据， params刷新会丢失 params里面的数据

## 三类导航守卫

### 全局守卫

* beforeEach
* beforeResolve
* afterEach

### 路由独享守卫

* beforeEnter

### 组件内守卫

* beforeRouterLeave
* beforeRouterEnter
* beforeRouterUpdate

### 执行顺序

beforeRouterLeave——>beforeEach——>beforeEnter——>beforeRouterEnter——>beforeResolve

——>afterEach——>**beforeCreate-->created-->beforeMount-->mounted-->**beforeRouterEnter的next回调

加粗的是Vue的生命周期钩子函数

参考文章：[vue-router导航守卫，不懂的来](https://zhuanlan.zhihu.com/p/54112006)