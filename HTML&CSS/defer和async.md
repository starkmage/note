[js的位置是否会影响首屏的加载时间](https://blog.csdn.net/weixin_37722222/article/details/90021406)

[无线性能优化：页面可见时间与异步加载](https://fed.taobao.org/blog/taofed/do71ct/mobile-wpo-pageshow-async/)

[浅谈script标签中的async和defer](https://www.cnblogs.com/jiasm/p/7683930.html)

[渲染流水线：CSS如何影响首次加载时的白屏时间？](http://interview.poetries.top/browser/part5/lesson23.html#%E9%82%A3%E6%B8%B2%E6%9F%93%E6%B5%81%E6%B0%B4%E7%BA%BF%E4%B8%BA%E4%BB%80%E4%B9%88%E9%9C%80%E8%A6%81-cssom-%E5%91%A2%EF%BC%9F)

[DOMContentLoaded与load的区别](https://www.cnblogs.com/caizhenbo/p/6679478.html)

[浏览器资源的加载与解析的影响与原因](https://www.jianshu.com/p/2bc93efe0958)

[页面js脚本与img等资源的下载顺序问题](https://www.cnblogs.com/wuguanglin/p/JSAndImgLoadOrder.html)

[深入浅出浏览器渲染原理](https://github.com/ljianshu/Blog/issues/51)

### 普通script标签

文档解析的过程中，如果遇到 script 脚本，就会停止页面的解析进行下载（**但是Chrome会做一个优化，如果遇到script 脚本，会快速的查看后边有没有需要下载其他资源的，如果有的话，会先下载那些资源，然后再进行下载script 所对应的资源，这样能够节省一部分下载的时间** ）。
资源的下载是在解析过程中进行的，虽说 script1 脚本会很快的加载完毕，但是他前边的 script2 并没有加载&执行，所以他只能处于一个挂起的状态，等待 script2 执行完毕后再执行。
当这两个脚本都执行完毕后，才会继续解析页面。

<img src="https://user-images.githubusercontent.com/9568094/31619989-a874ae42-b25b-11e7-9a80-e0f644f27849.png" style="zoom:33%;" />

<img src="https://user-images.githubusercontent.com/9568094/31621391-39849b1a-b25f-11e7-9301-641b1bc07155.png" style="zoom:33%;" />

### defer

HTML 4.0 规范，其作用是，告诉浏览器，等到 DOM+CSSOM 渲染完成，再执行指定脚本

```html
  <script defer src="xx.js"></script>
```

> - 浏览器开始解析 HTML 网页
>
> - 解析过程中，发现带有 defer 属性的 script 标签
>
> - 浏览器继续往下解析 HTML 网页，解析完就渲染到页面上，同时并行下载 script 标签中的外部脚本
>
> - 浏览器完成解析 HTML 网页，此时再执行下载的脚本，完成后触发 DOMContentLoaded

下载的脚本文件在 DOMContentLoaded 事件触发前执行（即刚刚读取完</html>标签），而且可以保证执行顺序就是它们在页面上出现的顺序。所以添加 defer 属性后，domReady 的时间并没有提前，但它可以让页面更快显示出来。

将放在页面上方的 script 加 defer，在 PC Chrome 下其效果相当于把这个 script 放在底部，页面会先显示。 但对 iOS Safari 和 iOS WebView 加 defer 和 script 放底部一样都是长时间白屏。

<img src="https://user-images.githubusercontent.com/9568094/31621324-046d4a44-b25f-11e7-9d15-fe4d6a5726ae.png" style="zoom:33%;" />

### async

HTML 5 规范，其作用是，使用另一个进程下载脚本，下载时不会阻塞渲染，并且下载完成后立刻执行

```html
  <script async src="yy.js"></script>
```

> - 浏览器开始解析 HTML 网页
>
> - 解析过程中，发现带有 async 属性的 script 标签
>
> - 浏览器继续往下解析 HTML 网页，同时并行下载 script 标签中的外部脚本，如果解析完先显示页面则触发 DOMContentLoaded，也就是说，DOMContentLoaded 事件的触发并不受 async 脚本加载的影响，可能在脚本加载完之前，就已经触发了 DOMContentLoaded，但一定会在load事件之前执行
>
> - 脚本下载完成，浏览器暂停解析 HTML 网页，开始执行下载的脚本
>
> - 脚本执行完毕，浏览器恢复解析 HTML 网页

async 属性可以保证脚本下载的同时，浏览器继续渲染。但是 async 无法保证脚本的执行顺序，哪个脚本先下载结束，就先执行那个脚本。

<img src="https://user-images.githubusercontent.com/9568094/31621170-b4cc0ef8-b25e-11e7-9980-99feeb9f5042.png" style="zoom:33%;" />

<img src="https://user-images.githubusercontent.com/9568094/31622216-6c37db9c-b261-11e7-8bd3-79e5d4ddd4d0.png" style="zoom:33%;" />

### 如何选择 defer 和 async

- defer 可以保证执行顺序，async 不行

- async 可以提前触发 domReady（即DOMContentLoaded），defer 不行

- defer 在 iOS 和部分 Android 下依然阻塞渲染，白屏时间长

- 当 script 同时加 async 和 defer 属性时，后者不起作用，浏览器行为由 async 属性决定

### DOMCOntentLoaded 与 load

DOMContentLoaded：DOMContentLoaded 事件在 html 文档加载完毕，并且 html 所引用的内联 JS、以及外链 JS 的同步代码都执行完毕后触发，但是页面的有些资源比如说图片资源还无法看到。

load：当页面 DOM 结构中的所有资源都加载完成之后，包括 async 标签的 js脚本，才会触发 load 事件。video、audio、flash 不会影响 load 事件触发。

### 关于白屏时间

- 首、白屏时间和 DomContentLoad 事件没有必然的先后关系，因为首屏图片加载完成才算首屏时间，所以与DOMContendLoad 不一定谁先谁后

- 所有 CSS 尽早加载是减少首屏时间的最关键

- script 标签放在 body 底部，做与不做 async 或者 defer 处理，都不会影响首屏时间，但影响DomContentLoad 和 load 的时间，进而影响依赖他们的代码的执行的开始时间

- **白屏时间：从浏览器输入地址并回车后到页面开始有内容的时间；**

  - ```js
    (window.chrome.loadTimes().firstPaintTime - window.chrome.loadTimes().startLoadTime)*1000
    ```

- **首屏时间：从浏览器输入地址并回车后到首屏内容渲染完毕的时间；**

- **用户可操作时间节点：domready触发节点，点击事件有反应；**

- **总下载时间：window.onload的触发节点。**

### 关于阻塞

* CSS 不会阻塞 DOM 树的解析（因为 CSSOM 树和 DOM 树的构建在 GUI 渲染线程是并行的呀）
* CSS 会阻塞页面渲染
* CSS 会阻塞后面 JS 语句的执行，因为 JS 的执行必须在 CSSOM 树构建之后，JS 可能会操作 CSS，间接影响了 DOMContentLoaded 事件的触发
* JS 为什么会阻塞 DOM 的解析？最直接准确的答案**：因为 JS 引擎线程和 GUI 渲染线程是互斥的，必然是阻塞的，与其说是阻塞，不如说是 JS 引擎线程夺走了渲染进程内的控制权**
* **JS文件不只是阻塞DOM的构建，它会导致CSSOM也阻塞DOM的构建**，原本DOM和CSSOM的构建是互不影响，井水不犯河水，但是一旦引入了JavaScript，CSSOM也开始阻塞DOM的构建，就变了，因为JavaScript不只是可以改DOM，它还可以更改样式，也就是它可以更改CSSOM。因为不完整的CSSOM是无法使用的，如果JavaScript想访问CSSOM并更改它，那么在执行JavaScript时，必须要能拿到完整的CSSOM。所以就导致了一个现象，如果浏览器尚未完成CSSOM的下载和构建，而我们却想在此时运行脚本，那么浏览器将延迟脚本执行和DOM构建，直至其完成CSSOM的下载和构建。也就是说，**在这种情况下，浏览器会先下载和构建CSSOM，然后再执行JavaScript，最后在继续构建DOM**。

### 为什么要强调CSS要放在header里，js放在尾部？

构建布局树需要 DOM 和 CSSOM，所以 HTML 和 CSS 都会阻塞渲染，所以需要让 CSS 尽早加载（如：放在头部），以缩短首次渲染的时间。

除此之外，由于 CSS 不会阻塞文档的解析，但是会阻塞文档渲染。把 CSS 放在头部可以尽快生成 CSSOM 树，后续渲染 DOM 的时候，可以一次性构建布局树，只需要渲染一次；如果把 CSS 放在后面，会先解析一次 DOM，加载 CSS 之后，会重新渲染之前的 DOM，需要两次渲染。

至于 JS 放在尾部，就是前面说的 JS 会阻塞解析和渲染嘛。

### First-Paint

经常会有人在回答页面的优化中提到将 JS 放到 body 标签底部，原因是因为浏览器生成 DOM 树的时候是一行一行读 HTML 代码的，script 标签放在最后面就不会影响前面的页面的渲染。那么问题来了，既然 DOM 树完全生成好后页面才能渲染出来，浏览器又必须读完全部 HTML 才能生成完整的 DOM 树，script 标签不放在 body 底部是不是也一样，因为 DOM 树的生成需要整个文档解析完毕。

![img](https://images2015.cnblogs.com/blog/746387/201704/746387-20170407181912191-1031407943.png)

我们再来看一下 chrome 在页面渲染过程中的，绿色标志线是 First Paint 的时间。为什么会出现 First Paint，页面的 paint 不是在渲染树生成之后吗？其实现代浏览器为了更好的用户体验,渲染引擎将尝试尽快在屏幕上显示的内容。它不会等到所有 HTML 解析之前开始构建和布局渲染树，部分的内容将被解析并显示。**也就是说浏览器能够渲染不完整的 DOM 树和 CSSOM，尽快的减少白屏的时间**。假如我们将 JS 放在 header，JS 将阻塞解析 DOM，DOM 的内容会影响到 First Paint，导致 First Paint 延后。所以说我们会将 JS 放在后面，以减少First Paint的时间，但是不会减少 DOMContentLoaded 被触发的时间。

在 body 中第一个 script 资源下载完成之前，浏览器会进行首次渲染，将该 script 标签前面的 DOM 树和 CSSOM 合并成一棵 Render 树，渲染到页面中。**这是页面从白屏到首次渲染的时间节点，比较关键**。

### 浏览器对同一域名下的资源并发下载线程数，chrome为6个。

浏览器对**同一域名**下的下载并发不超过 6 个。超过 6 个的话，剩余的将会在队列中等待，这就是为什么我们要将资源收敛到不同的域名下，也是为了充分利用该机制，最大程度的并发下载所需资源，尽快的完成页面的渲染。

