## 模板转换成视图的过程

* Vue.js通过编译将template 模板转换成 with 语句封装好的渲染函数（render）

* 调用`Vue`原型上`vm._render`方法，将 `render` 函数转换成 `Virtual DOM`，更具体的说，是`Vnode` 节点组成的树结构。

``` js
Vue.prototype._render = function() {
    var ref = vm.$options;
    var render = ref.render;
    ···
    try {
        // 这句代码是核心
        vnode = render.call(vm._renderProxy, vm.$createElement);
    } catch (e) {
        ···
    }
    ···
    return vnode
}
```

* 调用`Vue`原型上`vm._update`方法，将 `Vnode` 节点渲染成真正的 `DOM`，即把虚拟`DOM`映射成为真实的`DOM`。
  * `_update` 的核心是 `patch`，`patch`（也叫做`patching`算法），这是虚拟`DOM`最核心的部分。
  * 它可以将 `vnode` 渲染成真实的`DOM`，通过调用`createElm`方法进行`DOM`操作，创建节点，插入子节点，递归创建一个完整的`DOM`树并插入到`Body`中。
  * 在这个过程中，会利用 `diff` 算法对比新旧 `Vnode` 之间有哪些不同，然后根据对比结果找出需要更新的的节点进行更新，以求最小化改变真实 `DOM`。这点我们从单词含义就可以看出， `patch`本身就有补丁、修补的意思，其实际作用是在现有`DOM`上进行修改来实现更新视图的目的。

![](https://camo.githubusercontent.com/ecb8e60110bd61ee799c1ab28caa8d0cdce57b5c/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333137343730312d303964383937356166343365643663642e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

参考文章：

[vue源码阅读之数据渲染过程](https://juejin.im/post/6844904030997577741)

[Vue渲染过程浅析](https://segmentfault.com/a/1190000018495383)

[深入剖析Vue源码 - 完整渲染过程](https://juejin.im/post/6844903845282185229)

## Vue模板编译原理

简单说，Vue的编译过程就是将`template`转化为`render`函数的过程。会经历以下阶段：

1. 解析（parse）：使用大量的正则表达式对template字符串进行解析，将标签、指令、属性等转化为**抽象语法树 AST**
2. 优化（optimize)：Vue的数据是响应式的，但其实模板中并不是所有的数据都是响应式的。有一些数据首次渲染后就不会再变化，对应的DOM也不会变化。那么优化过程就是深度遍历AST树，按照相关条件对树节点进行标记，这些被标记的节点（静态节点）在后续 patch 的时候我们就可以跳过对它们的比对
3. generate（生成）：根据 AST 结构拼接生成 render 函数的字符串

### temeplate 的预编译

对于 Vue 组件来说，模板编译只会在组件实例化的时候编译一次，生成渲染函数之后再也不会进行编译。

因此，runtime 需要是仅仅是 render 函数，编译对组件的 runtime 是一种性能损耗。而模板编译的目的仅仅是将`template`转化为`render` 函数，而这个过程，正好可以在项目构建的过程中完成。这样可以让实际组件在 runtime 时直接跳过模板渲染，进而提升性能，这个在项目构建的编译template的过程，就是预编译。

比如`webpack`的`vue-loader`依赖了`vue-template-compiler`模块，在 webpack 构建过程中，将`template`预编译成 render 函数，在 runtime 可直接跳过模板编译过程。

Vue-cli 初始化项目时选择 runtime-compiler 和 runtime-only，看之前的[总结](http://stark.pub/article/5f488e19e612bf22264c8a35)。

## Virtual DOM是什么

**Virtual DOM 其实就是一棵以 JavaScript 对象( `VNode` 节点)作为基础的树，用对象属性来描述节点，实际上它只是一层对真实 DOM 的抽象。最终可以通过一系列操作使这棵树映射到真实环境上。**

**简单来说，可以把Virtual DOM 理解为一个简单的JS对象，并且最少包含标签名(`ela`)、属性(`attrs`)和子元素对象(`children`)三个属性。不同的框架对这三个属性的命名会有点差别。**

对于虚拟DOM，咱们来看一个简单的实例，就是下图所示的这个，详细的阐述了`模板 → 渲染函数 → 虚拟DOM树 → 真实DOM`的一个过程。

![](https://camo.githubusercontent.com/221b74e0795b9bfcd09d2ae7d46db2f4f114436f/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333137343730312d363763646536623766653264633233622e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

## Virtual DOM的作用

**虚拟DOM的最终目标是将虚拟节点渲染到浏览器视图上**。但是如果直接使用虚拟节点覆盖旧节点的话，会有很多不必要的DOM操作。例如，一个 ul 标签下很多个 li 标签，其中只有一个 li 有变化，这种情况下如果使用新的 ul 去替代旧的 ul，因为这些不必要的 DOM 操作而造成了性能上的浪费。

为了避免不必要的 DOM 操作，虚拟 DOM 在`VNode`虚拟节点映射到视图的过程中，将虚拟节点与上一次渲染视图所使用的旧虚拟节点（`oldVnode`）利用 `diff` 算法做对比，找出真正需要更新的节点来进行`DOM`操作，从而避免操作其他无需改动的`DOM`。

**其实虚拟DOM在Vue.js主要做了两件事：**

- **提供与真实DOM节点所对应的虚拟节点`VNode`**
- **将虚拟节点`VNode`和旧虚拟节点`oldVnode`进行对比，然后更新视图**

## 为什么需要 Virtual DOM

- 具备跨平台的优势

  由于 Virtual DOM 是以 JavaScript 对象为基础而不依赖真实平台环境，所以使它具有了跨平台的能力，比如说浏览器平台、Node 等。

- 因为操作 DOM 慢，而 JS 运行效率高，所以我们可以将DOM对比操作放在JS层，提高效率。

  因为 DOM 操作的执行速度远不如 JavaScript 的运算速度快，因此，把大量的DOM操作搬运到 JavaScript 中，运用 patching 算法来计算出真正需要更新的节点，最大限度地减少DOM操作，从而显著提高性能。

  Virtual DOM 本质上就是在 JS 和 DOM 之间做了一个缓存。可以类比 CPU 和硬盘，既然硬盘这么慢，我们就在它们之间加个缓存：既然 DOM 这么慢，我们就在它们 JS 和 DOM 之间加个缓存。CPU（JS）只操作内存（Virtual DOM），最后的时候再把变更写入硬盘（DOM）

- 提升渲染性能

  **Virtual DOM的优势不在于单次的操作，而是在大量、频繁的数据更新下，能够对视图进行合理、高效的更新**

为了实现高效的DOM操作，一套高效的虚拟DOM diff算法显得很有必要。**我们通过 patch 的核心----diff 算法，找出本次DOM需要更新的节点来更新，其他的不更新**。

缺点：

- 无法进行极致优化： 虽然虚拟 DOM + 合理的优化，足以应对绝大部分应用的性能需求，但在一些性能要求极高的应用中虚拟 DOM 无法进行针对性的极致优化。
- 首次渲染大量DOM时，由于多了一层虚拟DOM的计算，会比innerHTML插入慢。

## diff 算法

Vue.js 的 `diff` 算法**仅在同级的 `VNode` 间做 `diff`，递归地进行同级 `VNode` 的 `diff`，最终实现整个`DOM`树的更新**。因为跨层级的操作是非常少的，忽略不计，这样时间复杂度就从`O(n^3)`变成`O(n)`。

参考文章：

[为什么 Vue 中不要用 index 作为 key？（diff 算法详解）](https://juejin.im/post/6844904113587634184)

[Vue 虚拟dom diff原理详解](https://segmentfault.com/a/1190000020663531)

[详解vue的diff算法](https://juejin.im/post/6844903607913938951)

[解析vue2.0的diff算法](https://segmentfault.com/a/1190000008782928)

通过例子来更好的理解 diff 算法

以这样一个列表为例：

```html
<ul>
  <li>1</li>
  <li>2</li>
</ul>
```

那么它的 `vnode` 也就是虚拟 DOM 节点大概是这样的。

```js
{
  tag: 'ul',
  children: [
    { tag: 'li', children: [ { vnode: { text: '1' }}]  },
    { tag: 'li', children: [ { vnode: { text: '2' }}]  },
  ]
}
```

假设更新以后，我们把子节点的顺序调换了一下：

```js
{
  tag: 'ul',
  children: [
+   { tag: 'li', children: [ { vnode: { text: '2' }}]  },
+   { tag: 'li', children: [ { vnode: { text: '1' }}]  },
  ]
}
```

首先响应式数据更新后，触发了 `渲染 Watcher`  的回调函数 `vm._update(vm._render())`去驱动视图更新，

`vm._render()` 其实就是通过调用 `_render` 去执行 `render` 函数生成 `vnode`，而 `vm._update` 就会带着新的 `vnode` 去走触发 `__patch__` 过程。

我们直接进入 `ul` 这个 `vnode` 的 `patch` 过程。

``` js
function patch (oldVnode, vnode) {
    // some code
    if (sameVnode(oldVnode, vnode)) {
        patchVnode(oldVnode, vnode)
    } else {
        const oEl = oldVnode.el // 当前oldVnode对应的真实元素节点
        let parentEle = api.parentNode(oEl)  // 父元素
        createEle(vnode)  // 根据Vnode生成新元素
        if (parentEle !== null) {
          api.insertBefore(parentEle, vnode.el, api.nextSibling(oEl)) // 将新元素添加进父元素
          api.removeChild(parentEle, oldVnode.el)  // 移除以前的旧元素节点
          oldVnode = null
        }
    }
    // some code
    return vnode
}
```

对比新旧节点是否是相同类型的节点（判断两个节点是否值得比较）：

### 不是相同类型节点

`sameNode`为false的话，直接销毁旧的 `vnode`，渲染新的 `vnode`。**这也解释了为什么 diff 是同层对比。**

### 是相同类型节点(比如都是 ul 节点)，进入下一层

调用`patchVNode`方法

``` js
patchVnode (oldVnode, vnode) {
    const el = vnode.el = oldVnode.el
    let i, oldCh = oldVnode.children, ch = vnode.children
    // 相同，直接返回
    if (oldVnode === vnode) return
    // 文字
    if (oldVnode.text !== null && vnode.text !== null && oldVnode.text !== vnode.text) {
        api.setTextContent(el, vnode.text)
    }else {	//不是文字	
        updateEle(el, vnode, oldVnode)
        // 如果新旧 children 都存在，核心！
        if (oldCh && ch && oldCh !== ch) {
            updateChildren(el, oldCh, ch)
        }else if (ch){	// 新增
            createEle(vnode) //create el's children dom
        }else if (oldCh){	// 删除
            api.removeChildren(el)
        }
    }
}
```

#### 如果新旧完全相同，直接返回

#### 如果新的子 vnode 是文字 vnode

就直接调用浏览器的 `dom api` 把节点的直接替换掉文字内容就好

#### 如果新的子 vnode 不是文字 vnode

那么就要开始对子节点 `children` 进行对比了。（可以类比 `ul` 中的 `li` 子元素）。

##### 如果有新 children 而没有旧 children

说明是新增 children，直接 `addVnodes` 添加新子节点。

##### 如果有旧 children 而没有新 children

说明是删除 children，直接 `removeVnodes` 删除旧子节点。

##### 如果新旧 children 都存在（都存在 `li 子节点列表`，进行对比，核心！）

那么就是我们 `diff算法` 想要考察的最核心的点了，`updateChildren`，也就是新旧节点的 `diff` 过程。

1. 在讲对比过程之前，要讲一个比较重要的函数：`sameVnode`

   ```js
   function sameVnode (a, b) {
       return (
           a.key === b.key &&  // key值
           a.tag === b.tag &&  // 标签名
           a.isComment === b.isComment &&  // 是否为注释节点
           // 是否都定义了data，data包含一些具体信息，例如onclick , style
           isDef(a.data) === isDef(b.data) &&
           sameInputType(a, b) // 当标签是<input>的时候，type必须相同
       )
   }
   ```

   它是用来判断节点是否可用的关键函数，可以看到，判断是否是 `sameVnode`，传递给节点的 `key` 是关键。不设置 key 的时候，key 都是 undefined，肯定相等。

2. 首先，设置指针记录 diff 过程，这些变量分别指向`旧节点的首尾`、`新节点的首尾`。

   ```js
     // 旧首节点
     let oldStartIdx = 0
     // 新首节点
     let newStartIdx = 0
     // 旧尾节点
     let oldEndIdx = oldCh.length - 1
     // 新尾节点
     let newEndIdx = newCh.length - 1
   ```

   根据这些指针，在一个 `while` 循环中**不停的对新旧节点的两端的进行对比，然后把两端的指针向不断内部收缩，直到没有节点可以对比**。这其实就是在不断的patch。

3. 每一轮都是同样的对比：

   * 旧首节点和新首节点用 `sameNode` 对比

   * 旧尾节点和新尾节点用 `sameNode` 对比

   * 旧首节点和新尾节点用 `sameNode` 对比

   * 旧尾节点和新首节点用 `sameNode` 对比

   * 如果以上逻辑都匹配不到，再把所有旧子节点的 `key` 做一个映射到旧节点下标的 `key -> index` 表，然后用新 `vnode` 的 `key` 去找出在旧节点中可以复用的位置。这也是设置key和不设置key的区别：**不设key，新老 VNode 节点只会进行头尾两端的相互比较，设key后，除了头尾两端的比较外，还会从用旧节点的key生成的哈希表中寻找新节点的key以更高效的利用 DOM**

4. 其中某一项命中了，就递归的进入 `patchVnode` 针对单个 `vnode` 进行的过程（如果这个 `vnode` 又有 `children`，那么还会来到这个 `diff children` 的过程 ）
5. 不停的把匹配到的指针向内部收缩，直到新旧节点有一端的指针相遇（说明这个端的节点都被patch过了）

6. 在指针相遇以后，还有两种比较特殊的情况：

   * 有新节点需要加入。 如果更新完以后，`oldStartIdx > oldEndIdx`，说明旧节点都被 `patch` 完了，但是有可能还有新的节点没有被处理到。接着会去判断是否要新增子节点。
* 有旧节点需要删除。 如果新节点先patch完了，那么此时会走 `newStartIdx > newEndIdx`  的逻辑，那么就会去删除多余的旧子节点

![](http://img.stark.pub/20200921171514.gif)

### 为什么不要用 index 作为 key

举个很简单的例子：

一个列表组件使用 v-for 进行展示，data 是 ['a', 'b', 'c']，使用 index 作为 key，当 data 进行反转，变成 ['c', 'b', 'a']。

发现什么问题没有？key的顺序没变，传入的值完全变了。这会导致一个什么问题？

本来按照最合理的逻辑来说，旧的第一个 `vnode` 是应该直接完全复用 新的第三个`vnode`的，因为它们本来就应该是同一个 `vnode`，自然所有的属性都是相同的。

但是在进行子节点的 `diff` 过程中，会在 旧首节点和新首节点用`sameNode` 对比。 这一步命中逻辑，因为现在`新旧两次首部节点` 的 `key` 都是 `0`了。

然后把旧的节点中的第一个 `vnode` 和 新的节点中的第一个 `vnode` 进行 `patchVnode` 操作。

这会发生什么？这不就会在 `patchVnode` 函数里发现两个节点的文本不一样，然后重新进行 DOM 的渲染。当然，对于举的这个例子，只有文本的渲染可能觉得这工作很简单。。。

但是，本来是可以直接复用之前的最后一个 DOM 的，现在没法用！导致所有的优化都失效了，和不写 key 的效果是一样的了。

### 为什么不要用随机数作 key

然官方要求一个 `唯一的key`，是不是可以用 `Math.random()` 作为 `key` 来偷懒？这是一个很鸡贼的想法，看看会发生什么吧。

还是上面那个例子，数据变化以后，`key` 变成了完全全新的 3 个随机数。

上面说到，`diff` 子节点的首尾对比如果都没有命中，就会进入 `key` 的详细对比过程，简单来说，就是利用旧节点的 `key -> index` 的关系建立一个 `map` 映射表，然后用新节点的 `key` 去匹配，如果没找到的话，就会调用 `createElm` 方法 **重新建立** 一个新节点。

具体代码在这：

```js
// 建立旧节点的 key -> index 映射表
oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx);

// 去映射表里找可以复用的 index
idxInOld = findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx);
// 一定是找不到的，因为新节点的 key 是随机生成的。
if (isUndef(idxInOld)) {
  // 完全通过 vnode 新建一个真实的子节点
  createElm();
}
```

这。。。完全重建了，谈何优化。

