## HTMLCollection 与 NodeList

### 共同点

1. 都是类数组对象，都有`length`属性

2. 都有共同的方法：`item`，可以通过`item(index)`或者`item(id)`来访问返回结果中的元素


### 区别

1. `HTMLCollection`只包含元素节点（elementNode），elementNode就是HTML中的标签；`NodeList`可以包含任何节点类型，换行符也被当作文本节点
2. `HTMLCollection`比`NodeList`多一项方法：`namedItem`，可以通过传递id或name属性来获取节点信息
3. querySelectorAll 返回的是一个 Static Node List，而 getElementsBy 系列的返回的是一个 Live Node List。

### 获取方法

`getElementsByClassName` 之类得到的是 `HTMLCollection`，`queryselectorAll` 得到的是`NodeList`

``` html
<body>
  <div class="demo">
    内容1
    <p>内容2</p>
    <span>内容3</span>
  </div>
  
  <script>
    let d1 = document.getElementsByClassName('demo')
    let d2 = document.querySelectorAll('.demo')
    console.log(d1);
    console.log(d2);
  </script>
</body>
```

![](http://img.stark.pub/20201029195736.png)

`elm.children` 得到的是 `HTMLCollection`，而 `elm.childrenNodes` 得到的是`NodeList`

``` js
let d = document.querySelector('.demo')

console.log(d.children);
console.log(d.childNodes);
```

![](http://img.stark.pub/20201029200144.png)

## DOM API中append和appendChild的三个不同点

append和appendChild是两个常用的方法，用于将元素添加到文档对象模型（DOM）中。它们经常可以互换使用，没有太多麻烦，但如果它们是一样的，那么为什么要出现两个API呢？……它们只是相似，但不是一样。

### .append()

此方法用于以Node对象或DOMString(基本上是文本)的形式添加元素。

#### 插入一个Node对象

```js
const parent = document.createElement('div');
const child = document.createElement('p');
parent.append(child);
// 这会将子元素追加到div元素
// 然后div看起来像这样<div> <p> </ p> </ div>
```

这会将子元素追加到 `div` 元素，然后 `div` 看起来像这样

```html
<div> <p> </ p> </ div>
```

#### 插入DOMString

```js
const parent = document.createElement('div');
parent.append('附加文本');
```

然后 `div` 看起来像这样的

```html
<div>附加文本</ div>
```

### .appendChild()

与 `.append` 方法类似，该方法用于DOM中的元素，但在这种情况下，只接受一个Node对象。

#### 插入一个Node对象

```js
const parent = document.createElement('div');
const child = document.createElement('p');
parent.appendChild(child);
```

这会将子元素追加到 `div` 元素，然后 `div` 看起来像这样

```html
<div> <p> </ p> </ div>
```

#### 插入DOMString

```js
const parent = document.createElement('div');
parent.appendChild('Appending Text');
// Uncaught TypeError: Failed to execute 'appendChild' on 'Node': parameter 1 is not of type 'Node'
```

### 不同点

`.append` 接受Node对象和DOMString，而 `.appendChild` 只接受Node对象。

```js
const parent = document.createElement('div');
const child = document.createElement('p');
// 追加节点对象
parent.append(child) // 工作正常
parent.appendChild(child) // 工作正常
// 追加DOMStrings
parent.append('Hello world') // 工作正常
parent.appendChild('Hello world') // 抛出错误
```

`.append` 没有返回值，而 `.appendChild` 返回附加的Node对象。

```js
const parent = document.createElement('div');
const child = document.createElement('p');
const appendValue = parent.append(child);
console.log(appendValue) // undefined
const appendChildValue = parent.appendChild(child);
console.log(appendChildValue) // <p></p>
```

`.append` 允许您添加多个项目，而 `.appendChild` 仅允许单个项目。

```js
const parent = document.createElement('div');
const child = document.createElement('p');
const childTwo = document.createElement('p');
parent.append(child, childTwo, 'Hello world'); // 工作正常
parent.appendChild(child, childTwo, 'Hello world');
// 工作正常，但添加第一个元素，而忽略其余元素
```

### 总结

在可以使用 `.appendChild` 的情况下，可以使用 `.append`，但反过来不行。

## nodeValue、value 和 innerText、innerHTML、textContent

1. innerHTML将内容解析为HTML，因此需要更长时间
2. nodeValue使用直接文本，不解析HTML，并且速度更快，任意Node节点都可以使用
3. textContent使用直接文本，不解析HTML，速度更快（推荐使用），任意Node节点都可以使用
4. innerText考虑到样式，例如，它不会获取隐藏文本，只有HTML元素才可以调用
5. value一般为input里面输入的值

nodeValue 和 textContent 的区别：

``` html
<p id="demo">
  内容
</p>

<script>
  let p = document.getELementById('demo')
  console.log(p.nodeValue)	// null
  console.log(p.textContent) // 内容
  let t = p.childNodes[0]	// t 即为 #text 节点
  console.log(t.nodeValue) // 内容
  console.log(p.textContent) // 内容
</script>
```

``` html
<div class="demo">
  内容1
  <p>内容2</p>
  <span>内容3</span>
</div>

<script>
  let d = document.querySelector('.demo')
  console.log(d.innerHTML);
  console.log(d.innerText);
  console.log(d.textContent);
</script>
```

![](http://img.stark.pub/20201104224139.png)

## 对于js拖拽功能的实现的理解，具体的实现方式是什么

思路：

1. 鼠标按下时，纪录一下鼠标位置距离移动元素的边的距离。

2. 鼠标移动时，实时更新元素位置
   * 具体计算以垂直方向为例：鼠标距离浏览器垂直方向距离，减去刚刚纪录的鼠标距离元素的垂直方向距离，就是元素定位top属性的值
3. 鼠标松开时，完成元素位移。

``` html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <title>Title</title>
  <style>
    div {
      width: 300px;
      height: 200px;
      background-color: pink;
      position: fixed;
      top: 0;
      left: 0;
    }
  </style>
</head>

<body>
  <div></div>
  <script>
    var divEle = document.getElementsByTagName('div')[0]
    var mouse = {
      mouseLock: false,
      mouseEleClient: {
        t: 0,
        l: 0
      }
    }
    divEle.onmousedown = function (e) { //当鼠标在divEle按下时触发事件
      e = e || window.event
      var l = e.clientX
      var t = e.clientY
      mouse.mouseEleClient.t = t - this.offsetTop
      mouse.mouseEleClient.l = l - this.offsetLeft
      mouse.mouseLock = true
    }
    divEle.onmouseup = function (e) {//当鼠标松开时触发事件
      mouse.mouseLock = false
    }
    window.onmousemove = function (e) { //当鼠标移动时持续触发
      if (mouse.mouseLock) {
        e = e || window.event
        var l = e.clientX
        var t = e.clientY
        var divT = t - mouse.mouseEleClient.t
        var divL = l - mouse.mouseEleClient.l
        divEle.style.top = divT + 'px'
        divEle.style.left = divL + 'px'
      }
    }
  </script>
</body>

</html>
```

