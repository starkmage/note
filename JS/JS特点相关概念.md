## JS为什么是单线程

这主要和 JS 的用途有关，JS 是作为浏览器的脚本语言，主要是实现用户与浏览器的交互，以及操作 DOM。这决定了它只能是单线程，否则会带来很复杂的同步问题。 

举个例子：假如 JS 被设计了多线程，如果有一个线程要修改一个 DOM 元素，另一个线程要删除这个 DOM 元素，你让浏览器咋办？所以，为了避免复杂性，从一诞生，JavaScript 就是单线程，这已经成了这门语言的核心特征。

## Node.js的两大特点

### Node.js的非阻塞I/O、异步I/O

#### 什么是I/O

I/O 即Input/Output, 输入和输出的意思。在浏览器端，只有一种 I/O，那就是利用 Ajax 发送网络请求，然后读取返回的内容，这属于`网络I/O`。

回到 Node.js 中，其实这种的 I/O 的场景就更加广泛了，主要分为两种:

- `文件 I/O`，比如用 `fs` 模块对文件进行读写操作
- `网络 I/O`，比如 `http` 模块发起网络请求

#### 阻塞、非阻塞I/O

`阻塞`和`非阻塞 I/O` 其实是针对操作系统内核而言的，而不是 Node.js 本身。`阻塞 I/O` 的特点就是一定要**等到操作系统完成所有操作后才表示调用结束**，进程会被阻塞，而 `非阻塞 I/O` 是调用后立马返回，不用等操作系统内核完成操作。

对前者而言，在操作系统进行 I/O 的操作的过程中，我们的应用程序其实是一直处于等待状态的，什么都做不了。那如果换成`非阻塞I/O`，调用返回后我们的 Node.js 应用程序可以完成其他的事情，而操作系统同时也在进行 I/O。这样就把等待的时间充分利用了起来，提高了执行效率，但是同时又会产生一个问题，Node.js 应用程序怎么知道操作系统已经完成了 I/O 操作呢？

为了让 Node.js 知道操作系统已经做完 I/O 操作，需要重复地去操作系统那里判断一下是否完成，这种重复判断的方式就是`轮询`。而对于轮询，CPU要么重复检查I/O，要么重复检查文件描述符，要么休眠，都得不到很好的利用。我们想要的理想情况是这样的：

Node.js 应用程序发起 I/O 调用后可以直接去执行别的逻辑，操作系统默默地做完 I/O 之后给 Node.js 发一个完成信号，Node.js 执行回调操作。

这就是异步 I/O 的效果。

#### 异步I/O

异步 I/O 是非阻塞 I/O 的具体实现方式。

Node.js 中的异步 I/O 采用多线程的方式，由 `EventLoop`、`I/O 观察者`，`请求对象`、`线程池`四大要素相互配合，共同实现。

那么`同步I/O`和`异步I/O`又有什么区别么？是不是只要做到`非阻塞IO`就可以实现`异步I/O`呢？

不是的。

- `同步I/O(synchronous I/O)`做`I/O operation`的时候会将process阻塞,所以`阻塞I/O`，`非阻塞I/O`，`IO多路复用I/O`都是`同步I/O`。
- `异步I/O(asynchronous I/O)`做`I/O opertaion`的时候将不会造成任何的阻塞。

`非阻塞I/O`都不阻塞了为什么不是`异步I/O`呢？其实当`非阻塞I/O`准备好数据以后还是要阻塞住进程去内核拿数据的。所以算不上`异步I/O`。

https://www.zhihu.com/question/19732473

### Node.js事件驱动

Node.js 的 **事件驱动（Event-Driven）** 模型是其高并发能力的核心设计，它基于 **事件循环（Event Loop）** 和 **观察者模式**，通过异步非阻塞的方式处理 I/O 操作。以下是详细解析：

------

#### 1. **事件驱动模型的核心概念**

- **事件（Event）**：由系统或用户触发的动作（如网络请求完成、文件读取结束、定时器到期等）。
- **事件触发器（Event Emitter）**：Node.js 中通过 `EventEmitter` 类（`events` 模块）实现事件的发布和监听。
- **事件循环（Event Loop）**：持续检查事件队列，按顺序执行回调的机制。

------

#### 2. **事件驱动的工作流程**

1. **初始化阶段**：
   - Node.js 启动时初始化事件循环，加载用户代码。
   - 遇到异步操作（如 `fs.readFile`、`setTimeout`）时，将其交给底层系统（Libuv）处理，**不阻塞主线程**。
2. **事件触发阶段**：
   - 当异步操作完成（如文件读取完毕），系统将对应的回调函数放入 **事件队列**（Event Queue）。
3. **事件循环处理阶段**：
   - 事件循环不断检查队列，按优先级分阶段执行回调（如先处理定时器，再处理 I/O 回调）。

## JS面向对象的三大特征

* 封装

  封装就是把抽象出来的数据和对数据的操作封装在一起，数据被保护在内部，程序的其它部分只有通过被授权的操作(成员方法)，才能对数据进行操作。

   JS封装只有两种状态，一种是公开的，一种是私有的。

  ``` js
  function Person(name, age){
    // 公开
    this.name = name
    // 私有
    var age = age
  }
  var p1 = new Person('zs', 20)
  ```

* 继承

  在ES5中，用的最多的就是组合继承和寄生组合继承，而ES6中则是extends

* 多态

  多态其实就是把做的内容和谁去做分开。同一操作作用于不同的对象，可以有不同的解释，产生不同的执行结果。

  因为js是动态语言，多态性本身就有。

  下面这个例子就说明了，一个动物能否实现叫声，只取决于makeSound，不针对某种类型的对象。

  ``` js
   var makeSound=function (animal) {
     animal.sound();
   }
   var Duck=function () {
   }
   var Dog=function () {
   }
   Duck.prototype.sound=function () {
     console.log("嘎嘎嘎")
   }
  Dog.prototype.sound=function () {
    console.log("旺旺旺")
  }
  makeSound(new Duck());
  makeSound(new Dog());
  ```

参考文章：

[JavaScript面向对象的三大特征](https://segmentfault.com/a/1190000008321085)

## JS 的严格模式

进入"严格模式"的标志，是下面这行语句：

``` js
'use strict'
```

老版本的浏览器会把它当作一行普通字符串，加以忽略。

* 针对整个脚本文件

  将"use strict"放在脚本文件的第一行，则整个脚本都将以"严格模式"运行。如果这行语句不在第一行，则无效，整个脚本以"正常模式"运行。

* 针对单个函数

  将"use strict"放在函数体的第一行，则整个函数以"严格模式"运行。

**ES6 的模块自动采用严格模式，不管你有没有在模块头部加上`"use strict"`，ES6模块中的this是undefined**

使用严格模式的一些限制：

- 变量必须声明后再使用

- 全局函数的 this 不再指向 Window，而是 undefined，用 window.fn 调用的时候，还是指向 window

  ``` js
  'use strict'
  
  console.log(this)	// Window
  
  // 但是在ES6模块里，直接打印this，就是undefined
  
  function fn() {
    console.log(this);
  }
  fn()	// undefined
  window.fn() // window
  
  // 但是在ES6模块里，直接就报错，因为你定义的fn属于模块，不属于window
  
  let fn2 = () => {
    console.log(this);
  }
  fn2()	// Window
  
  setTimeout(function() {
    console.log(this);
  }, 0)
  // Window，因为setTimeout是window的属性，必然是由window调用，相当于 window.setTimeout
  ```

* 函数的参数不能有同名属性，对象不能有重名的属性，否则报错

* 不能对只读属性赋值，否则报错

* 无法删除变量。只有configurable设置为true的对象属性，才能被删除

* 禁止使用arguments.callee

* arguments不再追踪参数的变化

  ``` js
  function f(a) {
    a = 2;
    return [a, arguments[0]];
  }
  f(1); // 正常模式为[2,2]
  
  function f(a) {
    "use strict";
    a = 2;
    return [a, arguments[0]];
  }
  f(1); // 严格模式为[2,1]
  ```

* 新增了一些保留字

参考文章：

[JavaScript 严格模式下this的几种指向](https://segmentfault.com/a/1190000010108912)

[Javascript 严格模式详解](https://www.ruanyifeng.com/blog/2013/01/javascript_strict_mode.html)

[ES6入门——Module 的语法](https://es6.ruanyifeng.com/#docs/module#%E6%A6%82%E8%BF%B0)

## JS传值调用

在传值调用中，传递给函数参数是函数被调用时所传实参的拷贝。在传值调用中实际参数被求值，其值被绑定到函数中对应的变量上（通常是把值复制到新内存区域）。

在传引用调用调用中，传递给函数的是它的实际参数的隐式引用而不是实参的拷贝。通常函数能够修改这些参数（比如赋值），而且改变对于调用者是可见的。

还有一种求值策略叫做传共享调用，传共享调用和传引用调用的不同之处是，该求值策略传递给函数的参数是对象的引用的拷贝，即对象变量指针的拷贝。

对于 JS 来说：

- **基本类型是传值调用**
- **引用类型传共享调用**

传值调用本质上传递的是变量的值的拷贝。

传共享调用本质上是传递对象的指针的拷贝，其指针也是变量的值。所以传共享调用也可以说是传值调用。

通过一个例子理解上面的内容：

``` js
var num = 10;
var obj1 = {item: "unchanged"};
var obj2 = {item: "unchanged"};

function changeStuff(a, b, c) {
  a = a * 10;
	b.item = "changed";
	c = {item: "changed"};
}

changeStuff(num, obj1, obj2);
```

<img src="https://raw.githubusercontent.com/nodejh/nodejh.github.io/master/images/Is-JavaScript-a-pass-by-reference-or-pass-by-value-language-3.png" style="zoom: 50%;" />

<img src="https://raw.githubusercontent.com/nodejh/nodejh.github.io/master/images/Is-JavaScript-a-pass-by-reference-or-pass-by-value-language-4.png" style="zoom:50%;" />

如图所示，变量 `a` 的值的改变，并不会影响变量 `num`。

而 `b` 因为和 `obj1` 是指向同一个对象，所以使用 `b.item = "changed";` 修改对象的值，会造成 `obj1` 的值也随之改变。

由于是对 `c` 重新赋值了，所以修改 `c` 的对象的值，并不会影响到 `obj2`。

参考文章：

[JavaScript 是传值调用还是传引用调用？](https://github.com/nodejh/nodejh.github.io/issues/32)