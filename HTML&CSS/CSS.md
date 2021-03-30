## 块元素、内联元素、内联块元素

### 块元素-block

div、p、h、ul、ol、li等

1. 每个块级元素都从新的一行开始（独占一行）
2. 元素的高度、宽度、行高以及顶和底边距都可设置
3. 元素宽度在不设置的情况下，是它本身父容器的100%（和父元素的宽度一致），除非设定一个宽度

### 内联元素（行内元素）-inline

a、span、code、label、i等

1. 和其他元素都在一行上
2. 元素的高度、宽度及**顶部和底部**边距**不可**设置
3. 元素的宽度就是它包含的文字或图片的宽度，不可改变

**absolute/fixed 和 float 只能作用于 block，作用于 inline 时，inline 会被自动转化为 block**

### 行内块元素-inline-block

img、input等

1. 和其他元素都在一行上
2. 元素的高度、宽度、行高以及顶和底边距都可设置

## BFC

块级格式上下文，是一个容器，用于管理块级元素

### 触发条件

1. 根元素
2. position 为 absolute 或者 fixed
3. 浮动元素（float 不为 none）
4. overflow 不为 visible 默认值
5. 行内块元素 inline-block
6. 表格元素
7. 弹性元素（display为flex的子元素）
8. 网格元素（display为 grid 或 inline-grid 元素的直接子元素）

### 渲染规则

1. 计算BFC高度的时候浮动元素也会参与计算
2. BFC是一个独立的容器，外面的元素不会影响里面的元素
3. BFC垂直方向边距重叠（**就是同属于一个BFC时，两个元素才有可能发生垂直margin的重叠**）
4. BFC的区域不会与浮动元素的box重叠

### 应用场景

1. 防止浮动导致父元素高度塌陷，清除浮动
2. 避免外边距折叠

## 脱离文档流的方式

- float
- position: absolute
- position: fixed

## z-index的默认值

默认值为auto，0在auto的上面，z-index:0 的会创建一个新的层叠上下文

## 圣杯布局和双飞翼布局对比

- 两种布局方式都是把主列放在文档流最前面，使主列优先加载。
- 两种布局方式在实现上也有相同之处，都是让三列浮动，然后通过负外边距形成三列布局。
- 两种布局方式的不同之处在于如何处理中间主列的位置：
  **圣杯布局是利用父容器的左、右内边距+两个从列相对定位**；
  **双飞翼布局是把主列嵌套在一个新的父级块中利用主列的左、右外边距进行布局调整**

##  margin的负值

- 非浮动

  - ```
    position: static
    ```

    - `margin-top/margin-left`: 元素本身向左/向上移动。
    - `margin-bottom/margin-right`: 元素本身不移动，元素后面的其他元素会向该元素的方向移动相应的距离，并且覆盖该元素。

  - ```
    position: relative
    ```

    - `margin-top/margin-left`: 元素本身向左/向上移动。
    - `margin-bottom/margin-right`: 元素本身不移动，元素后面的其他元素会向该元素的方向移动相应的距离，但是会被该元素覆盖。

  - ```
    position: absolute
    ```

    - `margin-top/margin-left`: 元素本身向左/向上移动。
    - `margin-bottom/margin-right`: 元素本身不移动，元素后面的其他元素会向该元素的方向移动相应的距离，对后面的元素不影响。（因为已经脱离了文档流）

- 浮动

  - 如果设置的 margin 的方向与浮动的方向相同：元素会往对应的方向移动对应的距离。
  - 如果设置的 margin 的方向与浮动的方向相反：则元素本身不动，元素之前或者之后的元素会向该元素的方向移动相应的距离。

## position 定位相对于谁

### absolute 元素

absolute 元素的包含块规则：

1. 最近的 `position` 值为 `fixed`、`absolute`、`relative` 或 `sticky` 的祖先元素（即 `position` 值不为 `static`）。
2. 是相对于上述祖先元素的 `padding` 区域（padding box）定位的。

### fixe 元素

fixed 元素的包含块规则：

- 相对于视口（viewport）定位的。 

### 都有的规则

`absolute` 或 `fixed` 元素还有共有的包含块规则。**满足下列条件之一的祖先元素，也是它们的包含块（相对于 padding 区域）**:

1. `transform` 的属性值不为 `none`。
2. `perspective` 的属性值不为 `none`。
3. `will-change` 的属性值为 `transform` 或 `perspective；`还有在属性值为 `filter` 的时候，不过仅在 Firefox 浏览器中有效。
4. `filter` 属性值不为 `none`。
5. `contain` 的属性值为 `paint`，即 `contain: paint`。

## position:sticky

### 用法

- position:sticky 被称为粘性定位元素（stickily positioned element）是计算后位置属性为 sticky 的元素
- 简单的理解就是：在目标区域以内，它的行为就像 position:relative;在滑动过程中，某个元素距离其父元素的距离达到sticky粘性定位的要求时(比如top：100px)；position:sticky这时的效果相当于fixed定位，固定到适当位置
- 可以说是相对定位relative和固定定位fixed的结合
- 元素固定的**相对偏移是相对于离它最近的具有滚动框的祖先元素**，如果祖先元素都不可以滚动，那么是相对于viewport来计算元素的偏移量

### position:sticky 使用条件

1. 父元素不能overflow:hidden或者overflow:auto属性
2. 必须指定top、bottom、left、right4个值之一，否则只会处于相对定位
3. 父元素的高度不能低于sticky元素的高度

4. sticky元素仅在其父元素内生效
5. 不会触发BFC

### 问题

兼容性差

## 浏览器是怎样解析CSS选择器的？

CSS**选择器**的解析是从右向左解析的。

若从左向右的匹配，发现不符合规则，需要进行回溯，会损失很多性能。

若从右向左匹配，先找到所有的最右节点，对于每一个节点，向上寻找其父节点直到找到根元素或满足条件的匹配规则，则结束这个分支的遍历。

两种匹配规则的性能差别很大，是因为从右向左的匹配在第一步就筛选掉了大量的不符合条件的最右节点（叶子节点），而从左向右的匹配规则的性能都浪费在了失败的查找上面。

## **怎么让Chrome支持小于12px 的文字？**

```
p {
  font-size:10px;
  -webkit-transform:scale(0.8);
} //0.8是缩放比例
```

## 使用图片 base64 编码的优点和缺点

base64编码是一种图片处理格式，通过特定的算法将图片编码成一长串字符串，在页面上显示的时候，可以用该字符串来代替图片的
url属性。

使用base64的优点是：

1. 减少一个图片的HTTP请求

使用base64的缺点是：

1. 根据base64的编码原理，编码后的大小会比原文件大小大1/3，如果把大图片编码到html/css中，不仅会造成文件体
   积的增加，影响文件的加载速度，还会增加浏览器对html或css文件解析渲染的时间。

2. 使用base64无法直接缓存，要缓存只能缓存包含base64的文件，比如HTML或者CSS，这相比直接缓存图片的效果要
   差很多。

3. 兼容性的问题，ie8以前的浏览器不支持。

一般一些网站的小图标可以使用base64图片来引入。

## ::before 和:after 中有什么区别

在css3中使用单冒号来表示伪类，用双冒号来表示伪元素。但是为了兼容已有的伪元素的写法，在一些浏览器中也可以使用单冒号
来表示伪元素。

伪类一般匹配的是元素的一些特殊状态，如hover、link等，而伪元素一般匹配的特殊的位置，比如after、before等。

## 伪类与伪元素的区别

css引入伪类和伪元素概念是为了格式化文档树以外的信息。也就是说，伪类和伪元素是用来修饰不在文档树中的部分，比如，一句
话中的第一个字母，或者是列表中的第一个元素。

伪类用于当已有的元素处于某个状态时，为其添加对应的样式，这个状态是根据用户行为而动态变化的。比如说，当用户悬停在指定的
元素时，我们可以通过:hover来描述这个元素的状态。

伪元素用于创建一些不在文档树中的元素，并为其添加样式。它们允许我们为元素的某些部分设置样式。比如说，我们可以通过::be
fore来在一个元素前增加一些文本，并为这些文本添加样式。虽然用户可以看到这些文本，但是这些文本实际上不在文档树中。

有时你会发现伪元素使用了两个冒号（::）而不是一个冒号（:）。这是CSS3的一部分，并尝试区分伪类和伪元素。大多数浏览
器都支持这两个值。按照规则应该使用（::）而不是（:），从而区分伪类和伪元素。但是，由于在旧版本的W3C规范并未对此进行
特别区分，因此目前绝大多数的浏览器都支持使用这两种方式表示伪元素。

## 为什么 img 是 inline 还可以设置宽高

img 是可替换元素。

在 CSS 中，可替换元素（replaced element）的展现效果不是由 CSS 来控制的。这些元素是一种外部对象，它们外观的渲染，是独立于 CSS 的。
简单来说，它们的内容不受当前文档的样式的影响。CSS 可以影响可替换元素的位置，但不会影响到可替换元素自身的内容。

## 遇到的浏览器兼容问题

1. 浏览器默认的margin和padding不同
   解决方案：加一个全局的*{margin:0;padding:0;}来统一

2. Chrome中文界面下默认会将小于12px的文本强制按照12px显示

   解决方案：前面提了

3. 超链接访问过后hover样式就不出现了，被点击访问过的超链接样式不再具有hover和active了

   解决方法：改变CSS属性的排列顺序L-V-H-A

4. IE 模式下的一些问题

## 阐述一下 CSSSprites（雪碧图）

将一个页面涉及到的所有图片都包含到一张大图中去，然后利用CSS的background-image，background-repeat，background-origin，background-position的组合进行背景定位。
利用CSSSprites能很好地减少网页的http请求，从而很好的提高页面的性能；CSSSprites能减少图片的字节。

优点：

1. 减少HTTP请求数，极大地提高页面加载速度
2. 增加图片信息重复度，提高压缩比，减少图片大小
3. 更换风格方便，只需在一张或几张图片上修改颜色或样式即可实现

缺点：

1. 图片合并麻烦
2. 维护麻烦，修改一个图片可能需要重新布局整个图片，样式

## 如何实现单行／多行文本溢出的省略（...）

```css
/*单行文本溢出*/
p {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}


/*多行文本溢出*/
p {
  position: relative;
  line-height: 1.5em;
  /*高度为需要显示的行数*行高，比如这里我们显示两行，则为3*/
  height: 3em;
  overflow: hidden;
}

p:after {
  content: "...";
  position: absolute;
  bottom: 0;
  right: 0;
  background-color: #fff;
}
```

## text-align 和 vertical-align

### text-align

text-aligin 有如下的选项 （这就相当于word文件的对齐方式一样的 有左对齐、右对齐、居中对齐）

* left：居左对齐 
* right：右对齐 
* center：居中 
* start：如果方向是左向右（ltf）的话=left end：同上 
* justify：两端对齐，最后一行无效 
* justify-all：强制最后一行两端对齐 
* match-parent：和inherit类似，区别在于start和end的值根据父元素的direction确定，并被替换为恰当的left或者right。

**要在块级元素容器下使用，并不控制块元素自己的对齐，只控制它的内部行内元素、行内块元素和文字的对齐。如果在 span 这种标签里面使用，是不会进行水平居中的。**

``` html
div {
  height: 100px;
  width: 100px;
  background-color: yellowgreen;
  text-align: center;
}

span {
	background-color: pink;
}

<div>
  <span>hello</span>
</div>
```

![](http://img.stark.pub/20201104135400.png)

对内部的块级元素不起作用，内部的div继承了外部div的text-align，所以它内部的文字是水平居中的，而整体相对父元素并不是水平居中的

``` html
#d {
  height: 100px;
  width: 100px;
  background-color: yellowgreen;
  text-align: center;
}

#dd {
  width: 80px;
  background-color: pink;
}

<div id="d">
  <div id="dd">hello</div>
</div>
```

![](http://img.stark.pub/20201104140158.png)

### line-height

“行高”顾名思意指一行文字的高度。具体来说是**指两行文字间基线之间的距离**。基线实在英文字母中用到的一个概念，我们刚学英语的时使用的那个英语本子每行有四条线，其中底部第二条线就是基线。

百分比或者不带单位的数字都是相对于行内内容高度的

分析一个情况：

有一个空的`div`，`<div></div>`，如果没有设置至少大于`1`像素高度`height`值时，该`div`的高度就是个`0`。如果该`div`里面打入了一个空格或是文字，则此`div`就会有一个高度。为什么`div`里面有文字后就会有高度呢？

这是个看上去很简单的问题，是理解`line-height`非常重要的一个问题。可能有人会认为是：文字撑开的！文字占据空间，自然将`div`撑开。但是事实上，深入理解`inline`模型后，会发现，根本不是文字撑开了`div`的高度，而是`line-height`！

``` html
#help {
  height: 100px;
  background-color: yellow;
}

#d1 {
  font-size: 30px;
  line-height: 0;
  background-color: green;
}

#d2 {
  font-size: 0;
  line-height: 30px;
  background-color: pink;
}

<div id="help"></div>
<div id="d1">test1</div>
<div id="d2">test2</div>
```

![](http://img.stark.pub/20201104144217.png)

看效果吧，div1是，没有高度的，但是文字显示出来了，div2没显示文字，但是依然有高度

网上都是这么说的，把`line-height`值设置为`height`一样大小的值可以实现单行文字的垂直居中。这句话确实是正确的，但其实也是有问题的。问题在于`height`，看我的表述：“把line-height设置为您需要的box的大小可以实现单行文字的垂直居中”，差别在于我把`height`去掉了，这个`height`是多余的，您不信您可以自己试试。——张鑫旭：[css行高line-height的一些深入理解及应用](https://www.zhangxinxu.com/wordpress/2009/11/css%E8%A1%8C%E9%AB%98line-height%E7%9A%84%E4%B8%80%E4%BA%9B%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3%E5%8F%8A%E5%BA%94%E7%94%A8/)

### vertical-align

`vertical-align`的百分比数值，是相对于此标签继承的`line-height`值决定的

只有一个元素属于`inline`或是`inline-block`，其身上的`vertical-align`属性才会起作用。

这部分内容非常多，详细参考张鑫旭大神的文章：[我对CSS vertical-align的一些理解与认识（一）](https://www.zhangxinxu.com/wordpress/2010/05/%E6%88%91%E5%AF%B9css-vertical-align%E7%9A%84%E4%B8%80%E4%BA%9B%E7%90%86%E8%A7%A3%E4%B8%8E%E8%AE%A4%E8%AF%86%EF%BC%88%E4%B8%80%EF%BC%89/?shrink=1)

实现水平垂直居中的一个例子

``` html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <style>
    html,
    body {
      margin: 0;
      padding: 0;
      width: 100%;
    }

    .text-container {
      height: 150px;
      text-align: center;
      background-color: pink;
      border: 3px solid greenyellow;
    }

    .text-container:after {
      content: "";
      display: inline-block;
      width: 0;
      height: 100%;
      vertical-align: middle;
    }

    span {
      vertical-align: middle;
      display: inline-block;
      max-width: 90%;
      max-height: 100px;
      overflow: hidden;
    }
  </style>
</head>

<body>
  <div class="text-container">
    <span>我是单行文本我是单行文本</span>
  </div>
  <div class="text-container">
    <span>我是多行文本我是多行文本我是多行文本我是多行文本我是多行文本我是多行文本</span>
  </div>
</body>

</html>
```

vertical-align 实现的是近似垂直居中

当设置元素的对齐方式为middle时，指的是：**元素的垂直中心线与父级元素基线的位置往上二分之一x高度所在线对齐**。换句话说，就是图中的黄色线与红色字母x的中心处对齐。但是文字具有下沉特性，原本x的中心点应该与图中的白色线对齐，但是文字的下沉特性使字母往下掉了一点。从而导致黄色线无法绝对与白色线对齐。这个下沉的大小与文字的大小和字体有关。当文字大小足够小时，我们可以忽略，近似的，白色线就与黄色线对齐，实现居中效果。但是文字大小很大时，就不能很好的实现了。

<img src="https://user-gold-cdn.xitu.io/2018/2/9/16179fb93a9ff285?imageView2/0/w/1280/h/960/format/webp/ignore-error/1" style="zoom:50%;" />

## 常见的布局方式

* flex布局
* 流式布局（百分比布局）
* 浮动布局（float）
* 定位布局（position）
* rem布局
* vw、vh布局
* 响应式布局

## 分析比较 opacity: 0、visibility: hidden、display: none 优劣和适用场景

- `display: none;`

1. **DOM 结构**：浏览器不会渲染 `display` 属性为 `none` 的元素，不占据空间；
2. **事件监听**：无法进行 DOM 事件监听；
3. **性能**：动态改变此属性时会引起回流，性能较差；
4. **继承**：不会被子元素继承，毕竟子类也不会被渲染；
5. **transition**：`transition` 不支持 `display`。

- `visibility: hidden;`

1. **DOM 结构**：元素被隐藏，但是会被渲染不会消失，占据空间；
2. **事件监听**：无法进行 DOM 事件监听；
3. **性能**：动态改变此属性时会引起重绘，性能较高；
4. **继承**：会被子元素继承，子元素可以通过设置 `visibility: visible;` 来取消隐藏；
5. **transition**：`transition` 不支持 `display`。

- `opacity: 0;`

1. **DOM 结构**：透明度为 100%，元素隐藏，占据空间；
2. **事件监听**：可以进行 DOM 事件监听；
3. **性能**：提升为合成层，不会触发重绘，性能较高；
4. **继承**：会被子元素继承,且，子元素**并不能**通过 `opacity: 1` 来取消隐藏；
5. **transition**：`transition` 支持 `opacity`。

## CSS:line-height:150%与line-height:1.5的真正区别是什么？

150%是根据父元素的字体大小计算出行高，并且子元素依然沿用这个计算后的行高。而1.5则是根据子元素自己字体的大小去乘以1.5来计算行高。另，1.5em等也是按照150%的情况来算的。

有单位时，子元素继承了父元素计算得出的行距；无单位时继承了系数，子元素会分别计算各自行距（推荐使用

## 去除inline-block元素间间距

[去除inline-block元素间间距的N种方法](https://www.zhangxinxu.com/wordpress/2012/04/inline-block-space-remove-%E5%8E%BB%E9%99%A4%E9%97%B4%E8%B7%9D/)