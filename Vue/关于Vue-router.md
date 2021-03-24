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
   **query可以用path来引入，也可以用name引入，params要用name来引入。**在前端接收参数都是类似的，分别是this.$route.query.name和this.$route.params.name。（在Node.js 中，如果params用了花括号形式传参，需要用query接收）

2.  url地址显示
    query更加类似于我们ajax中get传参，params则类似于post，说的再简单一点，前者在浏览器地址栏中显示参数，后者则不显示，需要在 router 里配置 /:key  才会显示在url里，而且是以路径的形式

   ```text
   // query
   http://localhost:8080/workorder/newApply?type=BOX_DEPLOY&typeDesc=%E5%B0%8F%E7%99%BD%E7%9B%92%E9%83%A8%E7%BD%B2
   
   // params
   http://localhost:8080/workorder/newApply
   或者配置了 /:type 的话，配置哪个显示哪个（没配置typeDesc就不显示）
http://localhost:8080/workorder/newApply/BOX_DEPLOY
   ```
   
3. 注意点
    query刷新不会丢失query里面的数据， params刷新会丢失 params里面的数据

这篇文章[容易混淆-论query和params的使用区别](https://juejin.im/post/6844903872545161224)说的非常好

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

## hash模式分享网页，#后被截断

使用 vue 框架开发的应用，分享出去的链接会被截断：

正常链接：[https://hxkj.vip/#/article?article_id=8](https://links.jianshu.com/go?to=https%3A%2F%2Fhxkj.vip%2F%23%2Farticle%3Farticle_id%3D8)
分享出去的链接被打开之后变成了：[https://hxkj.vip/?from=singlemessage&isappinstalled=0](https://links.jianshu.com/go?to=https%3A%2F%2Fhxkj.vip%2F%3Ffrom%3Dsinglemessage%26isappinstalled%3D0)
不仅路由被切掉了，参数也没了。。。。。。

### 一、全局路由里拦截链接

#### 1、在 # 号前面加上 ? 号

经过试验发现，只要在路由的 # 号前面加上 ？号，微信分隔链接的时候只会在域名与参数之间加上 `?from=singlemessage&isappinstalled=0`，后面还是会携带原本的参数的，不会被完全切掉。所以，加上之后：

```js
let shareLink = 'https://hxkj.vip/?#/article?article_id=8';
shareLink = shareLink.replace('/#/', '/?#/');
```

> 待分享的链接变成了：[https://hxkj.vip/?#/article?article_id=8](https://links.jianshu.com/go?to=https%3A%2F%2Fhxkj.vip%2F%3F%23%2Farticle%3Farticle_id%3D8)
>  分享出去之后，链接变成了这个：[https://hxkj.vip/?from=singlemessage&isappinstalled=0#/article?article_id=8](https://links.jianshu.com/go?to=https%3A%2F%2Fhxkj.vip%2F%3Ffrom%3Dsinglemessage%26isappinstalled%3D0%23%2Farticle%3Farticle_id%3D8)

发现区别了吧，这次虽然被插入了 `?from=singlemessage&isappinstalled=0` 这一串东西，但是最起码路由和参数还保留着，接下来我们就要对这一段链接进行处理了。

#### 2、正则替换

这一步需要在 vue 全局路由里完成，操作如下：

```js
// router.js
router.beforeEach((to, from, next) => {
    let url = window.location.href;
    if (url.includes('?from=')) { // 判断是否携带了 from 参数，这一步灵活变通，只要能判断出是从微信分享链接进来的就 OK
        url = url.replace(/vip.+.#/, 'vip/#'); // 利用正则表达式去掉微信携带的 ?from=singlemessage&isappinstalled=0 这串参数，如果这串参数对于你当前的页面有用处的话，可以重新拼接到你正常的链接后面去
        window.location.href = url; // 重定向到正常链接
    }
})
```

上面这段代码的核心在于正则替换 `url = url.replace(/vip.+.#/, 'vip/#')`，这并不是吃饱了没事干，非要写正则。而是微信分享到每个渠道（微信单人聊天、微信群聊、朋友圈、QQ...）所携带的 `from` 参数是不一样的，所以需要从域名后缀那里开始往后匹配，直到 # 号为止。替换之后，就相当于把微信添加上去的那一串参数给删除了！

以上步骤操作正确的话：

> 待分享的链接为：[https://hxkj.vip/?#/article?article_id=8](https://links.jianshu.com/go?to=https%3A%2F%2Fhxkj.vip%2F%3F%23%2Farticle%3Farticle_id%3D8)
>  分享出去之后，再次打开分享链接。由于路由那里做了处理，链接变为为正常状态：[https://hxkj.vip/#/article?article_id=8](https://links.jianshu.com/go?to=https%3A%2F%2Fhxkj.vip%2F%23%2Farticle%3Farticle_id%3D8)

### 二、前端页面中转，重定向

#### 1、新建中转页

在 `public` 文件夹里新建一个 `html` 页面（与项目中 `index.html` 同级），命名为 `redirect.html`，文件内容如下：

```html
<script>
    let url = location.href.split('?')
    let params = url[1].split('&')
    let data = {}
    params.forEach((n, i) => {
        let p = n.split('=')
        data[p[0]] = p[1]
    })
    if (!!data.shareRedirect) {
        window.location.href = decodeURIComponent(data.shareRedirect)
    }
</script>
```

因为只作为跳转使用，所以不需要其他的东西，只需要写 js 就可以了

#### 2、组装分享链接

把要分享的链接，设置为中间页面的路径

```js
let shareLink = 'https://hxkj.vip/#/article?article_id=8';
shareLink = window.location.href.split('#')[0] + 'redirect.html?shareRedirect=' + encodeURIComponent(shareLink);
```

这个方法，比第一个方法多了个中间页，总体来说，还是比较方便的。

以上步骤操作正确的话：

> 待分享的链接为：[https://hxkj.vip/redirect.html?shareRedirect=https%3A%2F%2Fhxkj.vip%2F%3F%23%2Farticle%3Farticle_id%3D8](https://links.jianshu.com/go?to=https%3A%2F%2Fhxkj.vip%2Fredirect.html%3FshareRedirect%3Dhttps%3A%2F%2Fhxkj.vip%2F%3F%23%2Farticle%3Farticle_id%3D8)
>  分享出去之后，再次打开分享链接。由于中间页面做了重定向处理，链接变为为正常状态：[https://hxkj.vip/#/article?article_id=8](https://links.jianshu.com/go?to=https%3A%2F%2Fhxkj.vip%2F%23%2Farticle%3Farticle_id%3D8)

参考文章：

[vue hash模式下微信分享后打开首页，三种完美解决方案](https://www.jianshu.com/p/97729dd2c94d)

## encodeURI和encodeURIComponent

这俩都是编码URL，可以把字符串作为 URI 组件进行编码，唯一区别就是编码的字符范围，其中：

* encodeURI方法***不会***对下列字符编码  **ASCII字母  数字  ~!@#$&\*()=:/,;?+'**

* encodeURIComponent方法***不会***对下列字符编码 **ASCII字母  数字  ~!\*()'**

* 所以encodeURIComponent比encodeURI编码的范围更大。

实际例子来说，encodeURIComponent会把 http://  编码成  http%3A%2F%2F 而encodeURI却不会。

**什么场合应该用什么方法**

如果需要编码整个URL，然后需要使用这个URL，那么用encodeURI

比如

```js
encodeURI("http://www.cnblogs.com/season-huang/some other thing");
```

编码后会变为

```js
"http://www.cnblogs.com/season-huang/some%20other%20thing";
```

其中，空格被编码成了%20。但是如果你用了encodeURIComponent，那么结果变为

```js
"http%3A%2F%2Fwww.cnblogs.com%2Fseason-huang%2Fsome%20other%20thing"
```

连 "/" 都被编码了，整个URL已经没法用了。

**当你需要编码URL中的参数的时候，那么encodeURIComponent是最好方法。**

```js
var param = "http://www.cnblogs.com/season-huang/"; //param为参数
param = encodeURIComponent(param);
var url = "http://www.cnblogs.com?next=" + param;
console.log(url) //"http://www.cnblogs.com?next=http%3A%2F%2Fwww.cnblogs.com%2Fseason-huang%2F"
```

参数中的 "/" 可以编码，如果用encodeURI肯定要出问题，因为后面的/是需要编码的。

参考文章：

[escape,encodeURI,encodeURIComponent有什么区别?](https://www.zhihu.com/question/21861899)

## 路由懒加载的3种方式

### Vue异步组件技术

结合vue的异步组件和Webpack的代码分割功能可以实现路由组件的懒加载。打包后，每一个组件生成一个js文件。举例如下:

```js
{
  path: '/Demo',
  name: 'Demo',
  //打包后，每个组件单独生成一个chunk文件
  component: reslove => require(['../views/Demo'], resolve)
}
```

### 动态import语法

这种方法很常用

``` js
//默认将每个组件，单独打包成一个js文件
const Demo = () => import('../views/Demo')
const Demo1 = () => import('../views/Demo1')
```

有时我们想把某个路由下的所有组件都打包在同一个异步块(chunk)中。需要使用命名chunk一个特殊的注释语法来提供chunk name

``` js
//指定了相同的webpackChunkName,会合并打包成一个文件。
const Demo = () => import(/*webpackChunkName:'Home'*/ '../views/Demo')
const Demo1 = () => import(/*webpackChunkName:'Home'*/ '../views/Demo1')
```

### require.ensure()

使用webpacke的require.ensure也可以实现按需加载

- require.ensure()是webpack特有的，现在已经被import()取代
- 这个特性依赖于内置的promise，低版本浏览器使用require.ensure需要考虑是否支持es6

```js
//这种情况下，多个路由指定相同的chunkName，会打包成一个文件
const Demo = r => require.ensure([], () => r(require('../views/Demo')), 'Demo')
const Demo1 = r => require.ensure([], () => r(require('../views/Demo1')), 'Demo')
```