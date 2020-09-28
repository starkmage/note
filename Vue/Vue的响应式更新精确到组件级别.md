参考文章：

[为什么说 Vue 的响应式更新精确到组件级别？](https://juejin.im/post/6844904113432444942#heading-0)

Vue 对于响应式属性的更新，只会精确更新依赖收集的当前组件，也就是说一个组件实例只有一个 render-watcher，而不会递归的去更新子组件，这也是它性能强大的原因之一。

举例来说，这样的一个组件：

```vue
<template>
   <div>
      {{ msg }}
      <ChildComponent />
   </div>
</template>
```

我们在触发 `this.msg = 'Hello, Changed~'`的时候，会触发组件的更新，视图的重新渲染。

但是 `<ChildComponent />` 这个组件其实是不会重新渲染的，这是 Vue 刻意而为之的。

那么，Vue 这种精确的更新是怎么做的呢？我们知道每个组件都有自己的`渲染 watcher`，它掌管了当前组件的视图更新，但是并不会掌管 `ChildComponent` 的更新。

具体到源码中，是怎么样实现的呢？

在 `patch`  的过程中，当组件更新到`ChildComponent`的时候，会走到 `patchVnode`，那么这个方法大致做了哪些事情呢？

## patchVnode

### 执行 `vnode` 的 `prepatch` 钩子。

注意，只有 `组件vnode` 才会有 `prepatch` 这个生命周期，这里会走到`updateChildComponent`方法，这个 `child` 具体指什么呢？

```js
prepatch (oldVnode: MountedComponentVNode, vnode: MountedComponentVNode) {
  const options = vnode.componentOptions
  // 注意 这个child就是ChildComponent组件的 vm 实例，也就是咱们平常用的 this
  const child = vnode.componentInstance = oldVnode.componentInstance
  updateChildComponent(
  child,
  options.propsData, // updated props
  options.listeners, // updated listeners
  vnode, // new parent vnode
  options.children // new children
  )
}
```

其实看传入的参数也能猜到大概了，就是做了：

1. 更新props
2. 更新绑定事件
3. 对于slot做一些更新

也就是说，只会对 `childComponent` 上声明的 `props`、`listeners`等属性进行更新，**而不会深入到组件内部进行更新**。

## props的更新如何触发重渲染

如果不会递归的去对子组件更新，如果我们把 `msg` 这个响应式元素通过props传给 `ChildComponent`，此时它怎么更新呢？

其实，`msg` 在传给子组件的时候，会被保存在子组件实例的 `_props` 上，并且被定义成了`响应式属性`，而子组件的模板中对于 `msg` 的访问其实是被代理到 `_props.msg` 上去的，所以自然也能精确的收集到依赖，只要 `ChildComponent` 在模板里也读取了这个属性。

上面说过，父组件发生重渲染的时候，是会重新计算子组件的 `props` 的，`msg` 的变化通过 `_props` 的响应式能力，也让子组件重新渲染了，到目前为止，都只有真的用到了 `msg` 的组件被重新渲染了。

## slot怎么更新

假设我们有父组件`parent-comp`：

```vue
<div>
  <slot-comp>
     <span>{{ msg }}</span>
  </slot-comp>
</div>
```

子组件 `slot-comp`：

```vue
<div>
   <slot></slot>
</div>
```

组件中含有 `slot`的更新 ，是属于比较特殊的场景。

这里的 `msg` 属性在进行依赖收集的时候，**收集到的是 `parent-comp` 的渲染watcher。**（至于为什么，你看一下它所在的渲染上下文就懂了。）

那么我们想象 `msg` 此时更新了，

```vue
<div>
  <slot-comp>
     <span>{{ msg }}</span>
  </slot-comp>
</div>
```

这个组件在更新的时候，遇到了一个子组件 `slot-comp`，按照 Vue 的精确更新策略来说，子组件是不会重新渲染的。

但是在源码内部，它做了一个判断，在执行 `slot-comp` 的 `prepatch` 这个钩子函数的时候，会执行 `updateChildComponent` 逻辑，在这个函数内部会发现它有 `slot` 元素。

总结来说，这次 `msg` 的更新不光触发了 `parent-comp` 的重渲染，也进一步的触发了拥有slot的子组件 `slot-comp` 的重渲染。

它也只是触发了两层渲染，如果 `slot-comp` 内部又渲染了其他组件 `slot-child`，那么此时它是不会进行递归更新的。（只要 `slot-child` 组件不要再有 slot 了）。

### Vue2.6的优化

简单来说，利用

```vue
<slot-comp>
  <template v-slot:foo>
    {{ msg }}
  </template>
</slot-comp>
```

这种语法生成的插槽，会统一被编译成函数，在子组件的上下文中执行，所以父组件不会再收集到它内部的依赖，如果父组件中没有用到 `msg`，更新只会影响到子组件本身。而不再是从通过父组件修改 `_props` 来通知子组件更新了，是直接去触发子组件的重新渲染的。

也就是说，2.6以后版本插槽渲染上下文是子组件，那么依赖收集自然也就是收集到子组件的依赖了（是不是可以理解成直接收集的是子组件的 render-watcher ）。所以在 msg 更新后，就会直接去触发子组件响应式依赖，进行重新渲染了。