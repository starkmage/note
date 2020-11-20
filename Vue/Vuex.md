https://github.com/ljianshu/Blog/issues/36

### 如何理解getters

**getters从表面是获得的意思，可以把他看作在获取数据之前进行的一种再编辑,相当于对数据的一个过滤和加工**。getters就像计算属性一样，getter 的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。

它和 computed 的区别就是使用场景不同而已。