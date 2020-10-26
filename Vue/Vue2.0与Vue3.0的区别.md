## 区别概述

1. 重构响应式系统，使用Proxy替换Object.defineProperty，使用Proxy优势：
   * 可直接监听数组类型的数据变化
   * 监听的目标为对象本身，不需要像Object.defineProperty一样遍历每个属性，有一定的性能提升
   * 可拦截apply、ownKeys、has等13种方法，而Object.defineProperty不行
   * 直接实现对象属性的新增/删除
   
2. 新增Composition API，更好的逻辑复用和代码组织

   * 在Vue2.x中，组件的主要逻辑是通过一些配置项来编写，包括一些内置的生命周期方法或者组件方法，这种基于配置的组件写法称为Options API（配置式API）

   * Vue3的一大核心特性是引入了Composition API（组合式API），**这使得组件的大部分内容都可以通过setup() 方法进行配置**

3. 重构 Virtual DOM
   * 模板编译时的优化，将一些静态节点编译成常量，更新时只`diff`动态的部分
   * slot优化，将slot编译为lazy函数，将slot的渲染的决定权交给子组件
   * 模板中内联事件的提取并重用（原本每次渲染都重新生成内联函数）

4. 代码结构调整，更便于Tree shaking，使得体积更小

   * Tree shaking是一个术语，通常用于描述移除 JavaScript 上下文中的未引用代码

   - 简而言之： 不会把所有的都打包进来，只会打包你用到的`api`
   - 很大程度的减少了开发中的冗余代码，提升编译速度

5. 在Vue3的源码结构层面，使用Typescript替换Flow

6. 移除了一些 API，比如在`Vue2.x`中可以通过`EventBus`的方法来实现组件通信，这种用法在`Vue3`中就不行了，在`Vue3`中移除了` $on,$off`等方法，而是推荐使用`mitt`方案来代替：

   ``` js
   // Vue2.x
   var EventBus = new Vue()
   Vue.prototype.$EventBus = EventBus
   ...
   this.$EventBus.$on()  this.$EventBus.$emit()
   
   // Vue3
   import mitt from 'mitt'
   const emitter = mitt()
   emitter.on('foo', e => console.log('foo', e) )
   emitter.emit('foo', { a: 'b' })
   ```

## Vue3 的响应式区别

**`Proxy` 的代理是针对整个对象的，而不是对象的某个属性**，因此不同于 `Object.defineProperty` 的必须遍历对象每个属性，`Proxy` 只需要做一层代理就可以监听同级结构下的所有属性变化，当然对于深层结构，递归还是需要进行的。此外`Proxy`也支持代理数组的变化。

Vue2 中，对于给定的 data，如 `{ count: 1 }`，是需要根据具体的 key 也就是 `count`，去对「修改 data.count 」 和 「读取 data.count」进行拦截，也就是必须预先知道要拦截的 key 是什么，这也就是为什么 Vue2 里对于对象上的新增属性无能为力。

```js
Object.defineProperty(data, 'count', {
  get() {},
  set() {},
})
```

而 Vue3 所使用的 Proxy，则是这样拦截的：

```js
new Proxy(data, {
  get(data, key) { },
  set(data, key, value) { },
})
```

可以看到，根本不需要关心具体的 key，它去拦截的是 「修改 data 上的任意 key」 和 「读取 data 上的任意 key」。

所以，不管是已有的 key  还是新增的 key，都逃不过它的魔爪。

但是 Proxy 更加强大的地方还在于 Proxy 除了 get 和 set，还可以拦截更多的操作符。

> Proxy只会代理对象的第一层，那么Vue3又是怎样处理这个问题的呢？

判断当前Reflect.get的返回值是否为Object，如果是则再通过`reactive`方法做代理， 这样就实现了深度观测。

> 监测数组的时候可能触发多次get/set，那么如何防止触发多次呢？

我们可以判断key是否为当前被代理对象target自身属性，也可以判断旧值与新值是否相等，只有满足以上两个条件之一时，才有可能执行trigger。

## 为什么要新增Composition API，它能解决什么问题

Vue2.0中，随着功能的增加，组件变得越来越复杂，越来越难维护，而难以维护的根本原因是Vue的API设计迫使开发者使用watch，computed，methods选项组织代码，而不是实际的业务逻辑。

另外Vue2.0缺少一种较为简洁的低成本的机制来完成逻辑复用，虽然可以mixin完成逻辑复用，但是当mixin变多的时候，会使得难以找到对应的data、computed或者method来源于哪个mixin，使得类型推断难以进行。

所以Composition API的出现，主要是也是为了解决Option API带来的问题，第一个是代码组织问题，Compostion API可以让开发者根据业务逻辑组织自己的代码，让代码具备更好的可读性和可扩展性，也就是说当下一个开发者接触这一段不是他自己写的代码时，他可以更好的利用代码的组织反推出实际的业务逻辑，或者根据业务逻辑更好的理解代码。

第二个是实现代码的逻辑提取与复用，当然mixin也可以实现逻辑提取与复用，但是像前面所说的，多个mixin作用在同一个组件时，很难看出property是来源于哪个mixin，来源不清楚，另外，多个mixin的property存在变量命名冲突的风险。而Composition API刚好解决了这两个问题。

## 参考文章

[快速进阶Vue3.0](https://segmentfault.com/a/1190000020709962)

[Vue3 的响应式和以前有什么区别，Proxy 无敌？](https://juejin.im/post/6844904122479542285#heading-0)

[Vue3.0 进阶、环境搭建、相关API的使用](https://juejin.im/post/6877832502590111757)

[最全的Vue3.0升级指南](https://mp.weixin.qq.com/s?__biz=MjM5MTA1MjAxMQ==&mid=2651239020&idx=1&sn=80c6b6f2828be6f4a1c4bf9922380ba8&chksm=bd496be88a3ee2fe9cdb022081d2a68c91e4d95a7f483c9c5f1a8187b7a05a2dc0d0d20b5837&mpshare=1&scene=24&srcid=0820y73hmLnpKrfVHoWvoqGg&sharer_sharetime=1597883501832&sharer_shareid=50ec90ef8a78d43d2aeefdb38f1cb3a1#rd)