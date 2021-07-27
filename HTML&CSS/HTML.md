## 标准模式和兼容模式

DOCTYPE是document type的简写，它并不是 HTML 标签，也没有结束标签，它是一种标记语言的文档类型声明，**即告诉浏览器当前 HTML 是用什么版本编写的**。DOCTYPE的声明必须是 HTML 文档的第一行，位于html标签之前，对大小写不敏感。

标准模式又称为严格模式，兼容模式又称为怪异模式、混杂模式，都是 HTML4.01中的模式。声明引用DTD,因为HTML4.01基于SGML。DTD规定了标记语言的规则，这样浏览器才能正确的呈现内容。

html5不基于SGMl,所以不需要引用DTD。**html5没有这两种模式，它只有自己的html标准，是向后兼容的。所以说，第一行如果写`<!DOCTYPE html>`就是声明用的html5的标准。**

``` html
html5
<!DOCTYPE httml>

html4.01触发标准模式
<!-- HTML 4.01 严格型 -->strict
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"  "http://www.w3.org/TR/html4/strict.dtd"> 
触发兼容模式
 <!-- HTML 4.01 过渡型 -->Transitional
<!DOCTYPE HTML PUBLIC  "-//W3C//DTD HTML 4.01 Transitional//EN"  "http://www.w3.org/TR/html4/loose.dtd"> 
 
<!-- HTML 4.01 框架集型 --> Frameset
<!DOCTYPE HTML PUBLIC  "-//W3C//DTD HTML 4.01 Frameset//EN"  "http://www.w3.org/TR/html4/frameset.dtd"> 
```

**如果不写文档DOCTYPE声明，浏览器将无法获知HTML或XHTML文档的类型，会进入怪异模式；还有在IE6以下版本永远进入怪异模式**

**若文档为标准模式，则该文档的排版与JS运作模式都是以该浏览器支持的最高标准运行；兼容模式中，页面以宽松的向后兼容的方式显示，模拟老式浏览器的行为以防止站点无法工作。如果浏览器进入兼容模式，就会按自己的方式解析渲染页面。那么，在不同的浏览器下，显示的样式效果会不一致。**

参考文章：

[html中doctype的作用及几种类型详解](https://www.liudaima.com/a/45.html)

[HTML文件里开头的!Doctype有什么作用？](https://www.jianshu.com/p/ce48b13a4e1e)

[严格模式与混杂模式-如何触发这两种模式，区分它们有何意义](https://blog.csdn.net/binglingnew/article/details/17301433)

## HTML5 的新特性

[参考链接](https://www.cnblogs.com/ainyi/p/9777841.html)

1. 语义化标签:header、footer、section、nav、aside、article
2. 增强型表单：input的多个type(color, date, datetime, email, month, number, range, search, tel, time, url, week)
3. 新增表单元素：datalist、keygen、output
4. 新增表单属性：placehoder, required, min和max, step, height 和 width, autofocus, multiple
5. 音频视频：audio、video
6. canvas
7. 地理定位：
8. 拖拽：drag
9. 本地存储：localStorage、sessionStorage
10. 新事件：onresize、ondrag、onscroll、onmousewheel、onerror、onplay、onpause
11. WebSocket

## HTML5 和 HTML4.01 的主要区别

### 声明方面

1. HTML5 文件类型声明（<!DOCTYPE>）变成下面的形式：

```html
<!DOCTYPE html>
```

声明该 HTML 采用的是 HTML5 标准

### 标准方面

2. HTML5的文档解析不再基于SGML(标准通用标记语言)标准，而是形成了自己的一套标准。

因此 DOCTYPE 不需要对DTD进行引用，所以HTML5 也就没有严格模式与混杂模式的区别，HTML5 有相对宽松的语法，实现时，已经尽可能大的实现了向后兼容。

### 标签方面

3. 新增语义标签，其中包括

```html
<header>、<footer>、<section>、<article>、<nav>、<hgroup>、<aside>、<figure>
```

语言化解读：[IFE-NOTE：页面结构语义化](https://rainylog.com/post/ife-note-1/)

4. 废除一些网页美化方面的标签，使样式与结构分离更加彻底，包括

```html
<big>、<u>、<font>、<basefont>、<center>、<s>、<tt>
```

5. 通过增加了`<audio>、<video>`两个标签来实现对多媒体中的音频、视频使用的支持。
6. 增加`<canvas>`绘图标签

### 属性方面

6. 增加了一些表单属性, 主要是其中的input属性的增强

```html
<!-- 此类型要求输入格式正确的email地址 -->
<input type=email >
<!-- 要求输入格式正确的URL地址  -->
<input type=url >
<!-- 要求输入格式数字，默认会有上下两个按钮 -->
<input type=number >
<!-- 时间系列，但目前只有 Opera和Chrome支持 -->
<input type=date >
<input type=time >
<input type=datetime >
<input type=datetime-local >
<input type=month >
<input type=week >
<!-- 默认占位文字 -->
<input type=text placeholder="your message" >
<!-- 默认聚焦属性 -->
<input type=text autofocus="true" >
```

7. 其他标签新增了一些属性,

```html
<!-- meta标签增加charset属性 -->
<meta charset="utf-8">
<!-- script标签增加async属性 -->
<script async></script>
```

8. 使部分属性名默认具有boolean属性

```html
<!-- 只写属性名默认为true -->
<input type="checkbox"  checked/>
<!-- 属性名="属性名"也为true -->
<input type="checkbox"  checked="checked"/>
```

### 存储方面

9. 新增WebStorage, 包括localStorage和sessionStorage

10. 引入了IndexedDB和Web SQL，允许在浏览器端创建数据库表并存储数据, 两者的区别在于IndexedDB更像是一个NoSQL数据库，而WebSQL更像是关系型数据库。W3C已经不再支持WebSQL。

11. 引入了应用程序缓存器(application cache)，可对web进行缓存，在没有网络的情况下使用，通过创建cache manifest文件,创建应用缓存，为PWA(Progressive Web App)提供了底层的技术支持。

## meta标签属性

常用于定义页面的说明，关键字，最后修改日期，和其它的元数据。这些元数据将服务于浏览器（如何布局或重载页面），搜索引擎和其它网络服务。

### charset属性

```html
<!-- 定义网页文档的字符集 -->
<meta charset="utf-8" />
```

### name + content属性

```html
<!-- 网页作者 -->
<meta name="author" content="开源技术团队"/>
<!-- 网页地址 -->
<meta name="website" content="https://sanyuan0704.github.io/frontend_daily_question/"/>
<!-- 网页版权信息 -->
<meta name="copyright" content="2018-2019 demo.com"/>
<!-- 网页关键字, 用于SEO -->
<meta name="keywords" content="meta,html"/>
<!-- 网页描述 -->
<meta name="description" content="网页描述"/>
<!-- 搜索引擎索引方式，一般为all，不用深究 -->
<meta name="robots" content="all" />
```

尤其是设置移动端窗口，最常用的

``` html
<!-- 移动端常用视口设置 -->
<meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0, user-scalable=no"/>
<!-- 
  viewport参数详解：
  width：宽度（数值 / device-width）（默认为980 像素）
  height：高度（数值 / device-height）
  initial-scale：初始的缩放比例 （范围从>0 到10）
  minimum-scale：允许用户缩放到的最小比例
  maximum-scale：允许用户缩放到的最大比例
  user-scalable：用户是否可以手动缩 (no,yes)
 -->
```

### http-equiv属性

```html
<!-- expires指定网页的过期时间。一旦网页过期，必须从服务器上下载。 -->
<meta http-equiv="expires" content="Fri, 12 Jan 2020 18:18:18 GMT"/>
<!-- 等待一定的时间刷新或跳转到其他url。下面1表示1秒 -->
<meta http-equiv="refresh" content="1; url=https://www.baidu.com"/>
<!-- 禁止浏览器从本地缓存中读取网页，即浏览器一旦离开网页在无法连接网络的情况下就无法访问到页面。 -->
<meta http-equiv="pragma" content="no-cache"/>
<!-- 也是设置cookie的一种方式，并且可以指定过期时间 -->
<meta http-equiv="set-cookie" content="name=value expires=Fri, 12 Jan 2001 18:18:18 GMT,path=/"/>
<!-- 使用浏览器版本 -->
<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
<!-- 针对WebApp全屏模式，隐藏状态栏/设置状态栏颜色，content的值为default | black | black-translucent -->
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />
```

## link和@import的区别

1. link 属于 HTML 标签，不只可以加载 CSS 文件，而 @import 是 CSS 提供的，只有导入样式表的作用

2. 导入的语法不同

   * link（链接式）语法为：

   ``` html
   <link rel="stylesheet" href="style.css" type="text/css" />
   ```

   * @import（导入式）语法为：

   ``` html
   <style>
   	@import url('style.css')
   </style>
   ```

3. 加载页面时，link 标签引入的 CSS 被同时加载，并且多个 link 标签引入的 CSS 文件可以并行下载，而 @import 引入的 CSS 将在页面加载完毕后才被加载

4. link 作为 HTML 标签，不存在兼容性问题，而 @import 是 CSS2.1 才有的语法，故只可以在 IE5+ 才可以识别（当然，现在谁还适配 IE5 ）

5. 可以通过 JS 操作 DOM 插入 link 标签来改变样式，而 @import 是不能通过 JS 操作的

6. @import 一定要放在样式表的前面，不管是内部样式还是外部样式，如果放在末尾就会被浏览器忽略

## src 和 href 的区别

### 定义区别

href是Hypertext Reference的简写，表示超文本引用，指向网络资源所在位置。

常见场景:

```html
<a href="http://www.baidu.com"></a> 
<link type="text/css" rel="stylesheet" href="common.css"> 
```

src是source的简写，目的是要把文件下载到html页面中去。

常见场景:

```html
<img src="img/girl.jpg"></img> 
<iframe src="top.html"> 
<script src="show.js"> 
```

### 作用结果区别

1. href 用于在当前文档和引用资源之间确立联系
2. src 用于替换当前内容

### 浏览器解析方式

1. 当浏览器遇到href会并行下载资源并且不会停止对当前文档的处理。(同时也是为什么建议使用 link 方式加载 CSS，而不是使用 @import 方式)
2. 当浏览器解析到 src ，会暂停其他资源的下载和处理，直到将该资源加载或执行完毕。(这也是script标签为什么放在底部而不是头部的原因)（补充一点：Chrome会做一个优化，如果遇到`script`脚本，会快速的查看后边有没有需要下载其他资源的，如果有的话，会先下载那些资源，然后再进行下载`script`所对应的资源，这样能够节省一部分下载的时间）
3. 遇到 img 标签中的 src，是会异步并行下载的，不会阻塞

## Canvas 和 SVG的区别

### Canvas

* Canvas 是 HTML5 新增的标签，通过 JavaScript 来绘制 2D 图形

* Canvas 依赖分辨率，是逐像素进行渲染的

* 在 Canvas 中，一旦图形被绘制完成，它就不会继续得到浏览器的关注。如果其位置发生变化，那么整个场景也需要重新绘制，包括任何或许已被图形覆盖的对象

- 不支持事件处理器
- 弱的文本渲染能力
- 能够以 .png 或 .jpg 格式保存结果图像

### SVG

* SVG 历史更悠久，是一种使用 XML 描述 2D 图形的语言
* 不依赖分辨率
* 在 SVG 中，每个被绘制的图形均被视为对象。如果 SVG 对象的属性发生变化，那么浏览器能够自动重现图形
* 支持事件处理器
* 最适合带有大型渲染区域的应用程序（比如谷歌地图）
* 不适合游戏应用

## WebWorker

Web Worker 是HTML5标准的一部分，这一规范定义了一套 API，它允许一段 JavaScript 程序运行在主线程之外的另外一个线程中。 web worker 在后台运行，独立于其他脚本，不会影响页面的性能。

值得注意的是， Web Worker 规范中定义了两类工作线程，分别是专用线程Dedicated Worker和共享线程 Shared Worker，其中，Dedicated Worker只能为一个页面所使用，而Shared Worker则可以被多个页面所共享。

所有主流浏览器均支持 web worker，除了 Internet Explorer。

简单的使用方法：

1. 创建 web worker 文件

   创建一个计数脚本。该脚本存储于 "demo_workers.js" 文件中：

   ```js
   var i = 0;
   
   function timedCount() {
     i = i + 1;
     postMessage(i);
     setTimeout("timedCount()", 500);
   }
   
   timedCount();
   ```

   以上代码中重要的部分是 `postMessage`() 方法 - 它用于向 HTML 页面传回一段消息。

2. 创建 Web Worker 对象

   我们已经有了 web worker 文件，现在我们需要从 HTML 页面调用它。

   下面的代码检测是否存在 worker，如果不存在，它会创建一个新的 web worker 对象，然后运行 "demo_workers.js" 中的代码：

   ``` js
   var w;
   // 开始计时
   function startWorker()
   {
     // 浏览器是否支持 Web Worker
     if(typeof(Worker) !== "undefined") {
       if(typeof(w) == "undefined") {
         w = new Worker("demo_workers.js");
       }
       // 监听消息
       w.onmessage = function (event) {
         document.getElementById("result").innerHTML=event.data;
       };
     }
     else {
      document.getElementById("result").innerHTML = "您的浏览器不支持 Web Worker";
     }
   }
   // 停止计时
   function stopWorker() {
     // 终止 Web Worker
   	w.terminate();
   }
   ```

**Web Worker 有两个特点：**

1. **只能服务于新建它的页面，不同页面之间不能共享同一个 Web Worker。**
2. **当页面关闭时，该页面新建的 Web Worker 也会随之关闭，不会常驻在浏览器中。**

## PWA（Progressive Web Apps，渐进式网络应用程序）

### 概念

PWA相对于传统Web应用，主要在以下几个方面变得更强：

- 观感方面：在手机上，可以添加Web应用到桌面，并可提供类似于Native应用的沉浸式体验（也就是可以隐藏浏览器的脑门）。这部分背后的技术是**manifest**。
- 性能方面：由于本文主角Service Worker具有拦截浏览器HTTP请求的超能力，搭配CacheStorage，PWA可以提升Web应用在网络条件不佳甚至**离线时**的用户体验和性能。
- 其它方面：**推送通知**、后台同步等可选的高级功能，这些功能也是利用**Service Worker**来实现的。

### service worker

#### 简介

- `Service Worker`是浏览器在后台独立于网页运行的、用JavaScript编写的脚本。
- **Service Worker 不是服务于某个特定页面的，而是服务于多个页面的。（按照同源策略）**
- Service Worker 会**常驻在浏览器中**，即便注册它的页面已经关闭，Service Worker 也不会停止。本质上它是一个后台线程，只有你主动终结，或者浏览器回收，这个线程才会结束。
- 本质上充当Web应用程序与浏览器之间的代理服务器，也可以在网络可用时作为浏览器和网络间的代理。
- 它们旨在（除其他之外）使得能够创建有效的离线体验，拦截网络请求并基于网络是否可用以及更新的资源是否驻留在服务器上来采取适当的动作。他们还允许访问推送通知和后台同步`API`。

```javascript
// 不起眼的一行if，除了防止报错之外，也无意间解释了PWA的P：
// 如果浏览器不支持Service Worker，那就当什么都没有发生过
if ('serviceWorker' in navigator) {
    window.addEventListener('load', function () {
        // 所以Service Worker只是一个挂在navigator对象上的HTML5 API而已
        navigator.serviceWorker.register('/service-worker.js').then(function (registration) {
            console.log('我注册成功了666');
        }, function (err) {
            console.log('我注册失败了');
        });
    });
}
```

以下代码，可以拦截网页上所有png图片的请求，返回你的支付宝收款码图片，只要用户够多，总会有人给你打钱的。

```javascript
// service-worker.js
// 虽然可以在里边为所欲为地写任何js代码，或者也可以什么都不写，
// 都不妨碍这是一个Service Worker，但还是举一个微小的例子：
self.addEventListener('fetch', function (event) {
    if (/\.png$/.test(event.request.url)) {
        event.respondWith(fetch('/images/支付宝收款码.png'));
    }
});
```

#### 生命周期

Service Worker生命周期：安装中、安装后、激活中、激活后、已卸载。

- 首次导航到网站时，会下载、解析并执行Service Worker文件，触发install事件，尝试安装Service Worker。
- 如果install事件回调函数中的操作都执行成功，标志Service Worker安装成功，此时进入waiting状态。注意这时的Service Worker只是准备好了而已，并没有生效。
- 当用户二次进入网站时，才会激活Service Worker，此时会触发activate事件，标志Service Worker正式启动，开始响应fetch、post、sync等事件。

#### 主要事件

- **install：Service Worker安装时触发，通常在这个时机缓存文件。**
- activate：Service Worker激活时触发，通常在这个时机做一些重置的操作，例如处理旧版本Service Worker的缓存。
- **fetch：浏览器发起HTTP请求时触发，通常在这个事件的回调函数中匹配缓存，是最常用的事件。**
- push：和推送通知功能相关。
- sync：和后台同步功能相关。

#### 应用

- 缓存静态资源：可以利用CacheStorage API来缓存js、css、字体、图片等静态文件。
- 离线体验：如果我们首页index.html缓存下来，那我们的网页甚至可以支持离线浏览。
- 消息推送

## 前端实现下载功能的方式

只需要知道文件在服务器上的地址，就可以通过a标签实现下载

``` html
<a href="https://.../158ac1e6917445a4aa384a2a7209445a.xlsx" download="test">下载文件</a>
```

download属性存放下载文件的名称，此属性为必须

若文件地址为异步获取，即点击下载/导出按钮时才会从接口拿，则可以通过js插入a标签来实现。demo如下：

异步获取文件路径之后执行以下代码即可自动下载

```js
// 创建a标签
let a = document.createElement('a')
// 定义下载名称
a.download = '文件名称'
// 隐藏标签
a.style.display = 'none'
// 设置文件路径
a.href = 'https://.../158ac1e6917445a4aa384a2a7209445a.xlsx'
// 将创建的标签插入dom
document.body.appendChild(a)
// 点击标签，执行下载
a.click()
// 将标签从dom移除
document.body.removeChild(a)
```

缺点：此方式只适用于非图片和非pdf格式的文件下载，当文件为图片或pdf时，浏览器会打开预览，而非下载

https://blog.csdn.net/hfhwfw161226/article/details/105700504