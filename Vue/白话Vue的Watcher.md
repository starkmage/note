最近看了很多关于Vue响应式原理以及Watcher的文章，大部分文章都是贴一大段源码然后进行解析，可能因为自己比较笨吧，看完之后总还是迷迷糊糊的，不能系统的说出来这到底是怎么一回事，这其中的原理还是非常复杂的，下面，我就个人学习时候的一些总结，简单白话一下Vue中的响应式到底是怎么实现的，如有错误，恳请指正。

---

## Vue中的几个Watcher

我们都知道 Vue.js 有响应式的特点，常见的场景有以下几种：

* 数据变——>使用数据的视图变
* 数据变——>使用数据的计算属性变——>使用计算属性的视图变
* 数据变——>我们自己注册的 watch 执行回调函数

上面三个场景，分别对应三种 Watcher：

* render-watcher

  负责更新视图，每一个 Vue 组件实例都对应一个 render-watcher

* computed-watcher

  computed 里每一个属性都有一个 computed-watcher

* normal-watcher

  我们在 watch 中定义的，只有监听的数据变了，就执行回调函数

这三种 watcher 也有固定的执行顺序：

computed-render -> normal-watcher -> render-watcher

---

## render-watcher

每一个组件都会有一个 render-watcher, `function () {↵ vm._update(vm._render(), hydrating);↵ }`, 当响应式属性改变的时候，会调用该 render-watcher 来更新组件的视图，其实就是 render-watcher 去调用 `vm._update(vm._render())` 这个函数，重新根据 render 函数生成的 vnode 去渲染视图：`vm._render` 其实就是调用 `render` 函数生成 `vnode`，而 `vm._update` 方法则会对这个 `vnode` 进行 `patch` 操作，帮我们把 `vnode` 通过 `createElm`函数创建新节点并且渲染到 `dom节点` 中。

如何才能建立视图渲染与属性值之间的联系呢，主要是两个方面：

* 谁用了这个数据
* 数据变了之后怎么办

### 数据劫持

这一步是通过 `Object.defineProperty()` 来实现，也称为 Observe ，为每一个属性设置访问器属性，通过访问器中的 get 和 set ，我们就可以获取数据被使用了和数据被修改了的消息，即数据通过这一步的转换，变成了可以观测的数据。

因此，我们需要遍历 data 和 Vuex 中 store 里的所有数据，为每一个属性设置 get 和 set：

* 使用这个数据——触发 getter
* 修改这个数据——触发 setter

### 依赖收集

所谓依赖收集，简单说就是收集某个组件所依赖的数据。

那么由谁来管理数据与 watcher 的联系呢，答案是 Dep，每一个属性在设置 get 和 set 的时候，都会生成一个 Dep 类实例（**即每一个属性对应一个 Dep 实例**)，负责**记录以及通知**使用了这个属性的 watcher。

举个例子：

* 由于 Vue 执行一个组件的 render 函数是由 render-watcher 去代理执行的，render-watcher 在执行前会把 render-watcher 自身先赋值给 Dep.target 这个全局变量，等待响应式属性去收集
* 当组件 A 实例模板中使用了属性值 a ，会触发属性 a 的 getter，利用闭包的特性将这个 render-watcher 添加到 dep 的 subs 数组中去

* 同样的，当执行组件 B 的 render 时，Dep.target 被设置为 B 的 Watcher，如果 B 组件实例模板中也使用了属性值 a，那么组件实例 B 也会被添加到属性 a 的 dep 当中去

因此，我们可以记住以下几个结论：

* 依赖收集是由 getter 实现的依赖收集
* 我们可以把 watcher 称为观察者，把 dep 称为观察者目标，也就是观察者模式（虽然有的地方也说发布者-订阅者模式，但并不是完全相同的）

关于观察者模式和发布-订阅模式的区别，可以参考这两篇文章：

https://www.zhihu.com/question/23486749

https://juejin.im/post/6844903513009422343

### 派发更新

* 当属性值改变的时候，会触发 setter，调用 dep 的 notify 方法
* notify() 通知 dep 中 subs 数组中所有的 watcher
* render-watcher 执行回调 (render函数)，更新页面
* **每当 render 函数执行一次，也就是触发属性值的 getter 时，观察者（render-watcher）会存储一份新的依赖集合。对比新旧依赖集合，找出已经不再依赖的旧dep，然后将 render-watcher 从旧的 dep 的观察者队列中删除。这样就不会通知到当前的观察者了（render-watcher）**

### 补充

* 实际上，在 Vue1.x 系列的时候，组件中每一个响应式变量都会有一个 render-watcher ，后来开发者发现这样的粒度太细了，于是在 Vue2.x 的时候，就变成了更高粒度的划分：一个 Vue 组件实例对应一个 render-watcher
* 在一个Vue组件实例中，render-watcher 只有一个，而实例中的每一个属性都会有一个 dep ，于是，一个组件中的 render-watcher 与 dep 之间的关系，是一对多的关系
* 而Vue组件肯定不止一个，组件内部还会嵌套组件，属性有可能会与多个组件产生关联，于是，在这个层面上，一个 dep 会对应多个 render-watcher
* 综上，render-watcher 与 dep之间，是多对多的关系

---

## computed-watcher

两个问题：

- data 属性值变了，computed属性怎么知道？
- computed 的属性值变了，render 函数怎么知道？

解决这两个问题的大致思路是：

* 为每一个 computed 属性创建一个 computed-watcher
* computed 属性值的变化是由 data 属性变化引起的,因此若想模板也跟着变化，render-watcher 就得添加到 computed 属性的相关 data 属性的 dep 里

### 依赖收集

#### computed 属性的初始化

首先要对 computed 属性进行初始化。

当完成所有 data 属性值的数据劫持之后，便进入了 computed 属性的初始化流程：

- 遍历 computed 属性，为每个computed 属性都创建一个 computed-watcher，重写 get 和 set
- 遍历 computed 属性，对每个 computed 属性进行 get 劫持，下面详细介绍这里

#### computed 属性的 get 劫持

* 当 render-watcher 在执行 render 函数的时候，如果在模板中有 computed 属性值，则会触发 computed 属性的 getter
* 如果 dirty === true（最开始的时候一定为 true），执行 computed 属性函数，将函数返回值保存起来，注意，执行 computed 函数的过程中，data 属性值被 get 了
* 所以 data 属性值把当前的 computed 属性的 computed-watcher 添加到 dep 的 subs 数组里
* data 属性值还会把 render-watcher 添加到 dep 的 subs 数组里
* 返回保存起来的computed函数执行结果

注意 dep 中 subs 数组里的顺序，**computed-watcher先，render-watcher后**

### 连携反应

#### 模板直接引用 computed 属性

当 data 属性值发生修改，会带来什么反应呢？

* data 的属性值被 set 了
* data 属性值的 dep 通知 subs 里的所有 watcher，即执行 notify()
* 在 computed-watcher 的回调中，我们将 computed 属性标记为“待更新“，即 dirty = true，注意，此时并没有执行 computed 属性函数
* 在 render-watcher 执行 render 函数的时候，computed 属性值被 get 时，看有没有”待更新“标记，有就执行 computed 属性函数，执行完了就取消掉“待更新”标记，没有就直接返回上次执行后保存下来的结果

这也就是说，computed 属性值依赖 data 属性值，但是当 data 属性值改变的时候，computed 属性值并不会立即重新计算，只有当其它地方引起 computed 属性的 getter 时，computed 属性函数才会重新执行，这也就是所谓的 lazy(懒计算) 特性。

补充：

“待更新”(dirty)这样巧妙的设计，避免了computed 函数执行两次： computed-watcher 回调执行了一次 computed 函数，render-watcher 回调 render 函数，会触发 computed 属性的 get 进而又执行了 computed 函数

#### 模板嵌套引用 computed 属性

若出现当前 computed 计算属性嵌套其他 computed 计算属性时，先进行其他的依赖收集。

举个例子：

有这样一个引用链：模板——> computed 属性1——> computed 属性2——> data 属性值

此时 data 属性值的 dep 为：data属性值——> [computed-watcher1, computed-watcher2, render-watcher]

当 data 属性值被 set 的时候，computed-watcher1、computed-watcher2 相继被标记“待更新”，待到 render-watcher 更新时，就会依序执行：

- render 函数
- computed1 属性被get
- computed1 属性函数执行
- computed2 属性被get
- computed2 属性函数执行
- 缓存并返回 computed2 结果给 computed1
- 缓存并返回 computed1 结果给render函数
- render 函数将 computed1 新值渲染到视图中
- 页面更新

---

## normal-watcher

基本的原理也是类似的，遍历传入 watch 中的属性值，会在对应属性值的 dep 的 subs 数组中添加一个 normal-watcher，当属性值发生变化的时候，会通知 normal-watcher 去执行回调函数。

**这里的 normal-watcher 有一个特点，最初绑定的时候是不会执行的，只有在属性值发生变化的时候，才会执行，如果想一开始就执行一次，那么需要去更改为 handle 的写法（其实本来也会被编译成 handle），添加 immediate: true 参数。**

normal-watcher 还有一个特点，对于对象，默认只会监听对象的引用地址是否发生改变，而不会深入监听对象内属性值的变化，比如如果在 watch 内监听了 obj，则在 obj.a 改变时，watch 是不会知道的，如果想要深入监听，则需要添加 deep: true，deep 的意思就是深入观察，监听器会一层层的往下遍历，给对象的所有属性都加上这个 normal-watcher，但是这样性能开销就会非常大了，任何修改 obj 里面任何一个属性都会触发这个监听器里的 handler 执行**。所以，在回答这个地方的时候，一定要考虑到 `递归收集依赖` 对性能上的损耗和权衡。**

---

## computed-watcher 和 normal-watcher 的区别

1. 用 lazy 为 true 标示为它是一个 computed-watcher 
2.  computed-watcher 的 get 和 set 是在初始化(initComputed)时经过 defineComputed() 方法重写了的 
3. 当它所依赖的属性发生改变时虽然也会调用 computed-watcher 的 update()，但是因为它的 lazy 属性为 true，所以只执行把 dirty 设置为 true 这一个操作，并不会像 noraml-watcher 一样立即执行回调
4. 当有用到这个 computed-watcher 的时候，例如视图渲染时调用了它时，才会触发 computed-watcher 的 get，但又由于这个 get 在初始化时被重写了，其内部会判断 dirty 的值是否为 true 来决定是否需要执行 evaluate() 重新计算 
5. 因此有这么一句话：当计算属性所依赖的属性发生变化时并不会马上重新计算(只是将 dirty 设置为了 true 而已)，而是要等到其它地方读取这个计算属性的时候(会触发重写的 get )时才重新计算，因此它具备懒计算特性
6. **计算属性不能执行异步任务，计算属性必须同步执行**。如果遇到异步任务，就交给侦听属性

## 关于 Object.defineProprety() 的缺点

很多地方对此有这样类似的描述:

"无法监控到数组下标的变化，导致直接通过数组的下标给数组设置值，不能实时响应"

虽然 Vue.js 官方文档也是这么说的“由于 JavaScript 的限制，Vue 不能检测。。。”，但是数组下标本质上也就是对象的一个属性，其实也是可以设置监听的，对此尤老师本人在github上给出的回复是：考虑性能和用户体验的权衡。

关于此，详细可参考[这篇文章](https://segmentfault.com/a/1190000015783546)。

另外，还**无法检测到对象属性的添加或删除**。

对于数组的 push、pop 等操作，Vue 是通过内部重写数组原型的方式解决的。

使用了函数劫持的方式，重写了数组的方法，Vue将data中的数组进行了原型链重写，指向了自己定义的数组原型方法。这样当调用数组api时，可以通知依赖更新。如果数组中包含着引用类型，会对数组中的引用类型再次递归遍历进行监控。这样就实现了监测数组变化。

---

参考文章

[深入解析Vue依赖收集原理](https://www.ruphi.cn/archives/336/#anchor3)

[vue源码阅读（二）：数据响应式与实现](https://www.zybuluo.com/frank-shaw/note/1633762)

[Vue 响应式原理白话版]([https://www.njleonzhang.com/2018/09/26/vue-reactive.html#%E5%AE%98%E6%96%B9%E7%9A%84%E7%AE%80%E6%98%8E%E8%A7%A3%E9%87%8A](https://www.njleonzhang.com/2018/09/26/vue-reactive.html#官方的简明解释))

[Vue 的计算属性真的会缓存吗？（保姆级教学，原理深入揭秘）](https://juejin.im/post/6844904120290131982)

[聊聊 vue 中的 watcher](https://juejin.im/post/6844903518457823245)

[说说Vue的几个watcher（一）](https://juejin.im/post/6844904128435470350)

[说说Vue的几个watcher（二）](https://juejin.im/post/6844904128435453966)

[Vue.js中 watch 的高级用法](https://juejin.im/post/6844903600737484808)

