## XMLHTTPRequest对象

现代浏览器，最开始与服务器交换数据，都是通过`XMLHttpRequest` 对象。它可以使用JSON、XML、HTML和text文本等格式发送和接收数据。

**它给我们带来了很多好处**。

1. 不重新加载页面的情况下更新网页
2. 在页面已加载后从服务器请求/接收数据
3. 在后台向服务器发送数据。

**但是，它也有一些缺点**：

1. 使用起来也比较繁琐，需要设置很多值。
2. 早期的IE浏览器有自己的实现，这样需要写兼容代码。

readyState 的 5 个值的含义：

| 状态值 | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| 0      | 初始化状态。XMLHttpRequest 对象已创建或已被 abort() 方法重置。 |
| 1      | open() 方法已调用，但是 send() 方法未调用。请求还没有被发送。 |
| 2      | Send() 方法已调用，HTTP 请求已发送到 Web 服务器。未接收到响应。 |
| 3      | 所有响应头部都已经接收到。响应体开始接收但未完成。           |
| 4      | HTTP 响应已经完全接收。                                      |

在手写里我也分别用原生和Promise封装过 Ajax，可去参考

## jQuery的Ajax

jQuery 的 Ajax 就是对 `XMLHttpRequest` 对象的封装，兼容了各浏览器。

``` js
$.ajax({
  type: 'POST',
  url: url, 
  data: data,
  dataType: dataType,
  success: function () {},
  error: function () {}
})
```

**优点**：

1. 对原生`XHR`的封装，做了兼容处理，简化了使用
2. 增加了对`JSONP`的支持，可以简单处理部分跨域

**缺点**：

1. 如果有多个请求，并且有依赖关系的话，容易形成回调地狱
2. 本身是针对MVC的编程，不符合现在前端MVVM的浪潮
3. ajax是jQuery中的一个方法。如果只是要使用ajax却要引入整个jQuery非常的不合理

## axios

这个好，它是一个基于`promise`的`HTTP`库，可以用在浏览器和 `node.js` 中。它本质也是对原生`XMLHttpRequest`的封装，只不过它是Promise的实现版本，符合最新的ES规范。

``` js
axios({
    method: 'post',
    url: '/user/12345',
    data: {
      firstName: 'z',
      lastName: 's'
    }
  })
  .then(res => console.log(res))
  .catch(err => console.log(err))
```

**优点**：

1. 从浏览器中创建`XMLHttpRequests`
2. 从 `node.js` 创建 `http` 请求
3. 支持 `Promise` API
4. 拦截请求和响应
5. 转换请求数据和响应数据
6. 取消请求
7. 自动转换 `JSON` 数据
8. 客户端支持防御 `XSRF`

**缺点**：

1. 只持现代代浏览器

**原理是什么？**

**其核心只有两点：**

**1、request方法，**Axios外部方法其实都是在调用这一个方法

**2、方法内部创建一个Promise链式调用，**常用的功能，拦截器，数据修改器，http请求，就是在这个Promise链式调用中逐步被执行。**request**方法返回Promise链。我们用的就是这个返回的Promise，执行结果就在这个Promise中的状态值中。

并发请求：

配合axios.all()、axios.spread()同时发送多个请求

参考文章：

[Axios基本原理深度解析](https://juejin.im/post/6844904199302430733#heading-0)

[axios执行原理了解一下！](https://juejin.im/post/6844903685068161038#heading-1)

[axios.all()解决并发请求](https://segmentfault.com/a/1190000019882188)

## fetch

`Fetch API`提供了一个 `JavaScript` 接口，用于访问和操作`HTTP`管道的部分，例如请求和响应。它还提供了一个全局`fetch()`方法，该方法提供了一种简单，合理的方式来跨网络异步获取资源。
`fetch`是低层次的API，代替`XHR`，可以轻松处理各种格式，非文本化格式。可以很容易的被其他技术使用，例如`Service Workers`。但是想要很好的使用`fetch`，需要做一些封装处理。

**fetch不是ajax的进一步封装，而是原生js，没有使用XMLHttpRequest对象**。

``` js
fetch('http://example.com/movies.json')
  .then(function(response) {
    return response.json();
  })
  .then(function(myJson) {
    console.log(myJson);
  });
```

**优势：跨域的处理**
在配置中，添加`mode： 'no-cors'`就可以跨域了

```js
fetch('/users.json', {
    method: 'post', 
    mode: 'no-cors',
    data: {}
}).then(function() { /* handle response */ });
```

**`fetch`目前遇到的问题**：

1. `fetch`只对网络请求报错，对`400`，`500`都当做成功的请求，需要封装去处理。只有网络错误这些导致请求不能完成时，fetch 才会被 reject
2. `fetch`默认不会带`cookie`，需要添加配置项：fetch(url, {credentials: 'include'})
3. `fetch`不支持`abort`（终止），不支持超时控制，使用`setTimeout`及`Promise.reject`的实现超时控制并不能阻止请求过程继续在后台运行，造成了流量的浪费
4. `fetch`没有办法原生监测请求的进度，而`XHR`可以

**`fetch`规范与`jQuery.ajax()`主要有两种方式的不同，牢记：**

1. 当接收到一个代表错误的 `HTTP 状态码`时，从 `fetch()`返回的 `Promise` **不会被标记为 reject**， 即使该 HTTP 响应的状态码是 `404` 或 `500`。相反，它会将 `Promise 状态`标记为 `resolve` （但是会将 `resolve`的返回值的 `ok` 属性设置为 `false` ），仅当网络故障时或请求被阻止时，才会标记为 `reject`

2. 默认情况下，`fetch` **不会从服务端发送或接收任何 cookies**, 如果站点依赖于用户 `session`，则会导致未经认证的请求（要发送 `cookies`，必须设置 `credentials` 选项）