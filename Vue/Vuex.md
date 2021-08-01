https://github.com/ljianshu/Blog/issues/36

## Vuex的原理

**将数据存放到全局的store，再将store挂载到每个vue实例组件中，利用Vue.js的细粒度数据响应机制来进行高效的状态更新。**

两个疑问：

- **vuex的store是如何挂载注入到组件中呢？**

1、在vue项目中先安装vuex，核心代码如下：

```js
import Vuex from 'vuex';
Vue.use(vuex);// vue的插件机制
```

2、利用vue的[插件机制](https://cn.vuejs.org/v2/guide/plugins.html)，使用Vue.use(vuex)时，会调用vuex的install方法，装载vuex，install方法的代码如下：

```js
export function install (_Vue) {
  if (Vue && _Vue === Vue) {
    if (process.env.NODE_ENV !== 'production') {
      console.error(
        '[vuex] already installed. Vue.use(Vuex) should be called only once.'
      )
    }
    return
  }
  Vue = _Vue
  applyMixin(Vue)
}
```

3、applyMixin方法使用vue[混入机制](https://cn.vuejs.org/v2/guide/mixins.html)，vue的生命周期beforeCreate钩子函数前混入vuexInit方法，核心代码如下：

```js
Vue.mixin({ beforeCreate: vuexInit });
 
function vuexInit () {
    const options = this.$options
    // store injection
    if (options.store) {
      this.$store = typeof options.store === 'function'
        ? options.store()
        : options.store
    } else if (options.parent && options.parent.$store) {
      this.$store = options.parent.$store
    }
}
```

分析源码，我们知道了vuex是利用vue的mixin混入机制，在beforeCreate钩子前混入vuexInit方法，vuexInit方法实现了store注入vue组件实例，并注册了vuex store的引用属性$store。

- **vuex的state和getters是如何映射到各个组件实例中响应式更新状态呢？**

Vuex的state状态是响应式，是借助vue的data是响应式，将state存入vue实例组件的data中；Vuex的getters则是借助vue的计算属性computed实现数据实时监听。

参考文章：

https://www.cnblogs.com/lguow/p/13753900.html

## 如何理解getters

**getters从表面是获得的意思，可以把他看作在获取数据之前进行的一种再编辑,相当于对数据的一个过滤和加工**。getters就像计算属性一样，getter 的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。

它和 computed 的区别就是使用场景不同而已。

## 为什么 Vuex 的 mutation 中不能做异步操作 

vuex中在mutation中使用异步，其实也不会报错，只是人为规定不能在mutation中使用异步：

**因为更改state的函数必须是纯函数，纯函数既是统一输入就会统一输出，没有任何副作用；如果是异步则会引入额外的副作用，导致更改后的state不可预测。**

## 为什么区分mutation和action

在 vuex 里面 actions 只是一个架构性的概念，并不是必须的，说到底只是一个函数，你在里面想干嘛都可以，只要最后触发 mutation 就行。vuex 真正限制你的只有 mutation 必须是同步的这一点。

同步的意义在于这样每一个 mutation 执行完成后都可以对应到一个新的状态（和 reducer 一样），这样 devtools 就可以打个 snapshot 存下来，然后就可以随便 time-travel 了。如果你开着 devtool 调用一个异步的 action，你可以清楚地看到它所调用的 mutation 是何时被记录下来的，并且可以立刻查看它们对应的状态。

## 解决Vuex与v-model的冲突

当在严格模式中使用 Vuex 时，在属于 Vuex 的 state 上使用 `v-model` 会比较棘手：

```html
<input v-model="obj.message">
```

假设这里的 `obj` 是在计算属性中返回的一个属于 Vuex store 的对象，在用户输入时，`v-model` 会试图直接修改 `obj.message`。在严格模式中，由于这个修改不是在 mutation 函数中执行的, 这里会抛出一个错误。

下面是 mutation 函数：

```js
// ...
mutations: {
  updateMessage (state, message) {
    state.obj.message = message
  }
}
```

解决方法1

使用带有 setter 的双向绑定计算属性：

```html
<input v-model="message">
```

``` js
computed: {
  message: {
    get () {
      return this.$store.state.obj.message
    },
    set (value) {
      this.$store.commit('updateMessage', value)
    }
  }
}
```

解决方法2

不使用v-model这种语法糖了

```html
<input :value="message" @input="updateMessage">
```

``` js
computed: {
  ...mapState({
    message: state => state.obj.message
  })
},
methods: {
  updateMessage (e) {
    this.$store.commit('updateMessage', e.target.value)
  }
}
```

## Vuex缓存

**为什么页面刷新后 vuex 的内容会清空呢？**

因为 store 里的数据是保存在运行内存中的，当页面刷新时，页面会重新加载 Vue 实例，store 里面的数据就会被重新赋值初始化。

**刷新的时候缓存下来数据**

可以存到sessionStorage里

在顶层app.vue文件

``` vue
methods:{
  //将store中的数据存放到sessionStorage
  saveState() {
  	sessionStorage.setItem("state", JSON.stringify(this.$store.state));
  }
},
created() {
	//向window添加unload事件
	window.addEventListener("unload", this.saveState);
}
```

初始化state的时候

``` js
 const state = sessionStorage.getItem('state') ? JSON.parse(sessionStorage.getItem('state'))：{}
```