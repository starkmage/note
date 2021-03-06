`beforeCreate`：是new Vue()之后触发的第一个钩子，在当前阶段data、methods、computed以及watch上的数据和方法都不能被访问。

`created`：在实例创建完成后发生，当前阶段已经完成了数据观测，也就是可以使用数据，更改数据，在这里更改数据不会触发updated函数。可以做一些初始数据的获取，在当前阶段无法与Dom进行交互，如果非要想，可以通过vm.$nextTick来访问Dom。

`beforeMount`：发生在挂载之前，在这之前template模板已导入渲染函数编译。而当前阶段render函数已经创建，即将开始渲染。在此时也可以对数据进行更改，不会触发updated。

`mounted`：在挂载完成后发生，在当前阶段，真实的Dom挂载完毕，数据完成双向绑定，可以访问到Dom节点，使用$refs属性对Dom进行操作。

`beforeUpdate`：发生在更新之前，也就是响应式数据发生更新，虚拟dom重新渲染之前被触发，你可以在当前阶段进行更改数据，不会造成重渲染。

`updated`：发生在更新完成之后，当前阶段组件Dom已完成更新。要注意的是避免在此期间更改数据，因为这可能会导致无限循环的更新。

`beforeDestroy`：发生在实例销毁之前，在当前阶段实例完全可以被使用，我们可以在这时进行善后收尾工作，比如清除计时器。

`destroyed`：发生在实例销毁之后，这个时候只剩下了dom空壳。组件已被拆解，数据绑定被卸除，监听被移出，子实例也统统被销毁。

另外，只有当组件设置 keep-alive 的时候，还有两个特殊的生命周期：

activated 和 deactivated，注意，在组件初次加载的时候，也会触发 activated ，在 mounted 之后触发

![](http://img.stark.pub/20200921134915.png)

组件的调用顺序都是`先父后子`,渲染完成的顺序是`先子后父`。

组件的销毁操作是`先父后子`，销毁完成的顺序是`先子后父`。

### 加载渲染过程

```
父beforeCreate->父created->父beforeMount->子beforeCreate->子created->子beforeMount- >子mounted->父mounted
```

### 子组件更新过程

```
父beforeUpdate->子beforeUpdate->子updated->父updated
```

### 父组件更新过程

```
父 beforeUpdate -> 父 updated
```

### 销毁过程

```
父beforeDestroy->子beforeDestroy->子destroyed->父destroyed
```

参考文章：

[Vue 的生命周期之间到底做了什么事情？](https://juejin.im/post/6844904114879463437#heading-1)