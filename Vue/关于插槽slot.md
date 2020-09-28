Vue实现了一套遵循 [`Web Components 规范草案`](https://github.com/w3c/webcomponents/blob/gh-pages/proposals/Slots-Proposal.md) 的内容分发系统，即将`<slot>`元素作为承载分发内容的出口。

插槽slot，也是组件的一块HTML模板，这一块模板显示不显示、以及怎样显示由父组件来决定，插槽显示的位置却由子组件自身决定，slot写在组件template的什么位置，父组件传过来的模板将来就显示在什么位置。实际上，**一个slot最核心的两个问题在这里就点出来了，是显示不显示和怎样显示。**

插槽又分默认插槽、具名插槽、作用域插槽。

参考文章：

[深入理解vue中的slot与slot-scope](https://juejin.im/post/6844903555837493256?utm_source=gold_browser_extension)

[面试官：聊聊对Vue.js框架的理解](https://github.com/yacan8/blog/issues/26)

## Vue 2.6.0 之前

### 默认插槽

又名单个插槽、匿名插槽，与具名插槽相对，这类插槽没有具体名字，一个组件只能有一个该类插槽。

如：

```vue
<template>
<!-- 父组件 parent.vue -->
<div class="parent">
    <h1>父容器</h1>
    <child>
        <div class="tmpl">
            <span>菜单1</span>
        </div>
    </child>
</div>
</template>

<template>
<!-- 子组件 child.vue -->
<div class="child">
    <h1>子组件</h1>
    <slot></slot>
</div>
</template>
```

如上，渲染时子组件的`slot`标签会被父组件传入的`div.tmpl`替换。

### 具名插槽

匿名插槽没有name属性，所以叫匿名插槽。那么，插槽加了name属性，就变成了具名插槽。具名插槽可以在一个组件中出现N次，出现在不同的位置，只需要使用不同的name属性区分即可。

如：

```vue
<template>
<!-- 父组件 parent.vue -->
<div class="parent">
    <h1>父容器</h1>
    <child>
        <div class="tmpl" slot="up">
            <span>菜单up-1</span>
        </div>
        <div class="tmpl" slot="down">
            <span>菜单down-1</span>
        </div>
        <div class="tmpl">
            <span>菜单->1</span>
        </div>
    </child>
</div>
</template>

<template>
    <div class="child">
        <!-- 具名插槽 -->
        <slot name="up"></slot>
        <h3>这里是子组件</h3>
        <!-- 具名插槽 -->
        <slot name="down"></slot>
        <!-- 匿名插槽 -->
        <slot></slot>
    </div>
</template>
```

如上，slot 标签会根据父容器给 child 标签内传入的内容的 slot 属性值，替换对应的内容。

其实，默认插槽也有 name 属性值，为`default`，同样指定 slot 的 name 值为 default，一样可以显示父组件中传入的没有指定slot的内容。

### 作用域插槽

作用域插槽可以是默认插槽，也可以是具名插槽，不一样的地方是，作用域插槽可以为 slot 标签绑定数据，让其父组件可以获取到子组件的数据。

如：

```vue
<template>
    <!-- parent.vue -->
    <div class="parent">
        <h1>这是父组件</h1>
        <child>
            <template slot="default" slot-scope="slotProps">
                {{ slotProps.user.name }}
            </template>
        </child>
    </div>
</template>

<template>
    <!-- child.vue -->
    <div class="child">
        <h1>这是子组件</h1>
        <slot :user="user"></slot>
    </div>
</template>
<script>
export default {
    data() {
        return {
            user: {
                name: '小赵'
            }
        }
    }
}
</script>
```

如上例子，子组件 child 在渲染默认插槽 slot 的时候，将数据 user 传递给了 slot 标签，在渲染过程中，父组件可以通过`slot-scope`属性获取到 user 数据并渲染视图。

slot 实现原理：当子组件`vm`实例化时，获取到父组件传入的 slot 标签的内容，存放在`vm.$slot`中，默认插槽为`vm.$slot.default`，具名插槽为`vm.$slot.xxx`，xxx 为插槽名，当组件执行渲染函数时候，遇到`<slot>`标签，使用`$slot`中的内容进行替换，此时可以为插槽传递数据，若存在数据，则可曾该插槽为作用域插槽。

## Vue 2.6.0 之后

上面的用法，已经被废弃，但是还没有被移除，Vue 2.6.0 之后推出了新的用法：为**具名插槽和作用域插槽**引入了一个新的统一语法，即 **`v-slot`**。它取代了`slot`和`slot-scope`

参考文章：

[vue 插槽 slot 和 slot-scope 已被废弃](https://segmentfault.com/a/1190000019683759)

[Vue新指令：v-slot](https://juejin.im/entry/6844903785035366407)

### 默认插槽

``` vue
<!-- SlotDemo.vue -->
<template>
    <div class="slot-demo">
        <slot>我是一个slot</slot>
    </div>
</template>

<!-- App.vue -->
<SlotDemo>
    <template v-slot:default>
        <h3>v-slot:default匿名插槽</h3>
    </template>
</SlotDemo>
```

这里有一个特殊之处，**当被提供的内容只有默认的插槽时，组件的标签上才可以被当作插槽的模板来使用，也就是说`v-slot`可以直接使用在组件上**。正如上面的示例所示：

``` vue
<!-- App.vue -->
<SlotDemo v-slot:default>
    <p>v-slot:default 的使用</p>
</SlotDemo>
```

### 具名插槽

用一个 BaseLayout 组件当示例

``` vue
<!-- BaseLayout.vue -->
<template>
    <div class="container">
        <header>
            <slot name="header">Header Content</slot>
        </header>
        <main>
            <slot>Main Content</slot>
        </main>
        <footer>
            <slot name="footer">Footer Content</slot>
        </footer>
    </div>
</template>

<script>
    export default {
        name: 'BaseLayout'
    }
</script>
```

``` vue
<!-- V2.6版本之前 具名插槽的使用 -->
<BaseLayout>
    <h1 slot="header">Vue 2.6之前具名插槽</h1>
    <p>我是页面的主内容</p>
    <p slot="footer">©w3cplus</p>
</BaseLayout>

<!-- V2.6之后 具名插槽的使用 -->
<BaseLayout>
    <template v-slot:header>
        <h1>Vue 2.6之后具名插槽</h1>
    </template>
    <template v-slot:default>
        <p>我是页面的主内容</p>
    </template>
    <template v-slot:footer>
        <p>©w3cplus</p>
    </template>
</BaseLayout>
```

`v-slot`和 `v-on`和`v-bind`类似，也可以缩写，即 **把参数之前的所有内容（`v-slot:`）替换为字符 `#`**

### 作用域插槽

``` vue
<span>
  <slot :user="user">
    {{ user.name }}
  </slot>
</span>
```

``` vue
<!-- 之前 -->
<children>
  <template slot="default" slot-scope="slotProps">
    {{ slotProps.user.name }}
  </template>
</children>

<!-- 2.6.0之后 -->
<children>
  <template v-slot:default="slotProps">
    {{ slotProps.user.name }}
  </template>
</children>
```

slot 有 name 的时候把 default 换成 name 就行了

