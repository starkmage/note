## Standard Mode and Compatibility Mode

The term "DOCTYPE" is an abbreviation for "document type," and it is not an HTML tag with an opening or closing tag. It serves as a document type declaration in markup languages, specifically indicating the version in which the HTML document is written. The DOCTYPE declaration must be the first line of an HTML document, preceding the opening `<html>` tag, and it is case-insensitive.

Standard mode, also known as strict mode, and compatibility mode, alternatively referred to as quirks mode or mixed mode, are both modes defined in HTML4.01. The declaration references the Document Type Definition (DTD) since HTML4.01 is based on SGML. The DTD specifies the rules of the markup language, allowing browsers to accurately render content.

In contrast, HTML5 is not based on SGML, eliminating the need for DTD references. **HTML5 does not have distinct standard and compatibility modes; instead, it adheres to its own HTML standard. It is backward-compatible, meaning that specifying `<!DOCTYPE html>` as the first line of the document declares the use of the HTML5 standard.**

``` html
html5
<!DOCTYPE html>

html4.01触发标准模式
<!-- HTML 4.01 严格型 -->strict
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"  "http://www.w3.org/TR/html4/strict.dtd"> 
触发兼容模式
 <!-- HTML 4.01 过渡型 -->Transitional
<!DOCTYPE HTML PUBLIC  "-//W3C//DTD HTML 4.01 Transitional//EN"  "http://www.w3.org/TR/html4/loose.dtd"> 
 
<!-- HTML 4.01 框架集型 --> Frameset
<!DOCTYPE HTML PUBLIC  "-//W3C//DTD HTML 4.01 Frameset//EN"  "http://www.w3.org/TR/html4/frameset.dtd"> 
```

**If the document lacks a DOCTYPE declaration, the browser will be unable to determine the type of the HTML or XHTML document, leading it to enter quirks mode. Additionally, in versions prior to IE6, the browser will always default to quirks mode.**

**In standard mode, the document's layout and JavaScript operations adhere to the highest standards supported by the browser. In compatibility mode, the page is displayed in a lenient, backward-compatible manner, emulating behaviors of older browsers to prevent site malfunctions. If the browser enters compatibility mode, it interprets and renders the page in its own way. Consequently, across different browsers, the displayed styles and effects may vary.**

参考文章：

[html中doctype的作用及几种类型详解](https://www.liudaima.com/a/45.html)

[HTML文件里开头的!Doctype有什么作用？](https://www.jianshu.com/p/ce48b13a4e1e)

[严格模式与混杂模式-如何触发这两种模式，区分它们有何意义](https://blog.csdn.net/binglingnew/article/details/17301433)

## New features in HTML 5

[参考链接](https://www.cnblogs.com/ainyi/p/9777841.html)

1. Semantic Markup: header、footer、section、nav、aside、article
2. Enhanced Forms: input starts to support some types(color, date, datetime, email, month, number, range, search, tel, time, url, week)
3. New Form Types: datalist、keygen、output
4. New Form Attributes: placehoder, required, min和max, step, height 和 width, autofocus, multiple
5. audio、video
6. canvas
7. location：
8. drag
9. storage：localStorage、sessionStorage
10. new events：onresize、ondrag、onscroll、onmousewheel、onerror、onplay、onpause
11. WebSocket

## Differences between HTML5 and HTML4.01

### Declaration

1. HTML5 use the declaration as below

```html
<!DOCTYPE html>
```

### Standard

2. HTML5's document parsing is no longer based on the SGML (Standard Generalized Markup Language) standard; instead, it has its own set of standards.

As a result, there is no need for a DOCTYPE to reference a Document Type Definition (DTD) in HTML5. Consequently, HTML5 eliminates the distinction between strict mode and quirks mode. HTML5 adopts a relatively lenient syntax, and in its implementation, efforts have been made to achieve maximum backward compatibility.

### Tag

3. New Semantic Markup

```html
<header>、<footer>、<section>、<article>、<nav>、<hgroup>、<aside>、<figure>
```

语言化解读：[IFE-NOTE：页面结构语义化](https://rainylog.com/post/ife-note-1/)

4. Deprecated embellishment tags to achieve a more thorough separation of style and structure. 

```html
<big>、<u>、<font>、<basefont>、<center>、<s>、<tt>
```

5. Add `<audio>、<video>`
6. Add `<canvas>`

### Attribute

6. New Form Features

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

7. New attribute in the other tags

```html
<!-- meta标签增加charset属性 -->
<meta charset="utf-8">
<!-- script标签增加async属性 -->
<script async></script>
```

8. Enable certain attribute names to default to boolean properties.

```html
<!-- 只写属性名默认为true -->
<input type="checkbox"  checked/>
<!-- 属性名="属性名"也为true -->
<input type="checkbox"  checked="checked"/>
```

### Storage

9. Introduced WebStorage, including localStorage and sessionStorage.

10. Introduced IndexedDB and Web SQL, allowing the creation of database tables and storage of data on the client side.
11. Introduced the Application Cache, enabling web caching for offline use. This is achieved by creating a cache manifest file to establish application caching, providing foundational technical support for Progressive Web Apps (PWAs).

## meta tag attribute

Commonly used to define page descriptions, keywords, last modification date, and other metadata. This metadata serves browsers (for page layout or reloading), search engines, and other web services.

### charset

Define the character set for a web document.

```html
<!-- 定义网页文档的字符集 -->
<meta charset="utf-8" />
```

### name + content

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

Especially for setting the viewport on mobile devices, the most commonly used.

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
  user-scalable：用户是否可以Manually scale (no,yes)
 -->
```

### http-equiv

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

## Differences between link and @import

1. `<link>` is an HTML tag that not only loads CSS files but also has various use cases,  like icon and font, `@import` is provided by CSS and serves the only purpose of importing style sheets.

2.  different importation code:

   * link

   ``` html
   <link rel="stylesheet" href="style.css" type="text/css" />
   ```

   * @import

   ``` html
   <style>
   	@import url('style.css')
   </style>
   ```

3. When the page is loaded, CSS linked with the `<link>` tag is loaded at the same time, and multiple CSS files linked with multiple `<link>` tags can be downloaded in parallel. In contrast, CSS imported with `@import` is loaded only after the page has finished loading.

4. `<link>` is an HTML tag and doesn't pose compatibility issues, whereas `@import` is a CSS2.1 syntax and is recognized only by IE5 and later versions (though, adapting to IE5 is practically obsolete now).

5. The style applied by `<link>` can be dynamically changed by operating the DOM with JavaScript, while `@import` cannot be operated using JavaScript.

6. It is important for `@import` to be placed at the beginning of the style sheet, whether it is an internal or external style sheet. If placed at the end, the browser will ignore it.


## Differences between src and href

### Definition

`href` is an abbreviation for Hypertext Reference, indicating a hyperlink reference that points to the location of a network resource.

```html
<a href="http://www.baidu.com"></a> 
<link type="text/css" rel="stylesheet" href="common.css"> 
```

 `src` is a shortened form of source, serving the purpose of downloading a file into an HTML document.

```html
<img src="img/girl.jpg"></img> 
<iframe src="top.html"> 
<script src="show.js"> 
```

### Different effects

1. `href` is used to set a link between the current document and the referenced resource.
2. `src` is used to replace the current content.

### Browser parsing method

1. When the browser encounters `href`, it will initiate parallel downloading of resources without interrupting the processing of the current document. (This is also why using the `link` method for loading CSS is recommended over the `@import` method.)
2. When the browser encounters `src`, it will pause the downloading and processing of other resources until the specified resource is fully loaded or executed. (This is the reason why the `script` tag is often placed at the bottom rather than the head of the document.) (Additional note: Chrome optimizes this process by quickly checking if there are other resources that need to be downloaded after encountering a `script` tag. If so, it will download those resources first before proceeding with the resources associated with the `script`, thereby saving some download time.)
3. When encountering the `src` attribute within an `img` tag, asynchronous parallel downloading occurs without blocking.

## Differences between Canvas and  SVG

### Canvas

* `Canvas` is a new HTML5 tag used to draw 2D graphics through JavaScript.

* `Canvas` relies on resolution and renders pixel by pixel.

* In `Canvas`, once a graphic is drawn, it no longer receives attention from the browser. If its position changes, the entire scene, including any objects that may have been covered by the graphic, needs to be redrawn.

* Does not support event handlers.

* Limited text rendering capabilities.

* Ability to save the resulting image in .png or .jpg format.


### SVG

* `SVG` has a longer history and is a language that uses XML to describe 2D graphics.
* Does not rely on resolution.
* In `SVG`, each drawn graphic is treated as an object. If the properties of an SVG object change, the browser can automatically redraw the graphic.
* Supports event handlers.
* Most suitable for applications with large rendering areas (e.g., Google Maps).
* Not suitable for gaming applications.

## Web Worker

`Web Worker` is part of the HTML5 standard, which defines a set of APIs allowing a JavaScript program to run in a separate thread outside the main thread. `Web Worker` operates independently in the background, unaffected by other scripts, and does not impact the performance of the page.

It's worth noting that the `Web Worker` has two types of worker threads: `Dedicated Worker` and `Shared Worker`. A `Dedicated Worker` is exclusive to a single page, whereas a `Shared Worker` can be shared among multiple pages.

All major browsers support `Web Worker`, with the exception of Internet Explorer.

Simple Usage:

1.**Create a Web Worker File:**

Create a counting script stored in the "demo_workers.js" file:

```js
var i = 0;

function timedCount() {
  i = i + 1;
  postMessage(i); // Send count to the main thread
  setTimeout("timedCount()", 500);
}

timedCount();
```

2.**Create the Web Worker Object:**

Now that we have the Web Worker file, let's invoke it from the HTML page.

The following code checks for the existence of a worker. If it doesn't exist, it creates a new Web Worker object and runs the code from "demo_workers.js":

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

**Web Worker has two characteristics:**

1. **Can only serve the page that creates it; it cannot be shared among different pages.**
2. **When a page is closed, the Web Worker created by that page will also be closed and will not persist in the browser.**

## PWA（Progressive Web Apps）

### Definition

Compared to traditional web applications, PWA excels in the following aspects:

- **User Experience (UX):** On mobile devices, PWAs can be added to the home screen, providing an immersive experience similar to native applications (i.e., the ability to hide the browser's address bar). The underlying technology enabling this is the **manifest**.
- **Performance:** With the central role played by Service Worker, equipped with the ability to intercept browser HTTP requests and coupled with CacheStorage, PWAs can enhance the user experience and performance of web applications, particularly in challenging network conditions and even when **offline**.
- **Additional Features:** Optional advanced features such as **push notifications** and background synchronization are also made possible by leveraging **Service Worker**.

### service worker

#### Introduction

- `Service Worker` is a JavaScript-scripted script running in the background independently of web pages within the browser.
- **Service Worker does not serve a specific page but to multiple pages (subject to the same-origin policy).**
- Service Worker remains **live in the browser** even if the page that registered it is closed; it does not stop. Essentially, it functions as a background thread that only terminates if actively terminated or when the browser recycle it.
- Actually acts as a proxy server between a web application and the browser, serving as a proxy between the browser and the network when available.
- They are designed, among other things, to enable the creation of effective offline experiences, intercept network requests. They also allow access to push notifications and the background sync `API`.

```javascript
// 不起眼的一行if，除了防止报错之外，也无意间解释了PWA的P：
// 如果浏览器不支持Service Worker，那就当什么都没有发生过
// Navigator 对象包含有关浏览器的信息
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

The following code can intercept requests for all PNG images on a webpage and respond with a specified image:

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

#### Lifecycle

Service Worker Lifecycle: Installing, Installed, Activating, Activated, and Uninstalled.

- When a user first navigates to a website, the Service Worker file is downloaded, parsed, and executed, triggering the `install` event to attempt the installation of the Service Worker.
- If the operations in the `install` event callback are successful, the Service Worker is marked as installed, entering the waiting state. At this point, the Service Worker is prepared but not yet active.
- When the user revisits the website, the Service Worker is activated, triggering the `activate` event. This marks the formal start of the Service Worker, allowing it to respond to events such as `fetch`, `post`, and `sync`.

#### Key Events

- **install:** Triggered when the Service Worker is being installed. Typically used to cache files.
- activate: Triggered when the Service Worker is activated. Commonly used for reset operations, such as handling the cache of the old version of the Service Worker.
- **fetch:** Triggered when the browser makes an HTTP request. Often used to match cached resources; it is the most commonly used event.
- push: Related to push notification functionality.
- sync: Related to background sync functionality.

#### Applications

- Caching static resources: Utilize the CacheStorage API to cache static files such as JavaScript, CSS, fonts, and images.
- Offline experience: By caching the homepage (index.html), a website can support offline browsing.
- Push notifications

## Download Functionality in Front-end

To achieve file download, you can use the `<a>` tag by simply knowing the file's server address. 

``` html
<a href="https://.../158ac1e6917445a4aa384a2a7209445a.xlsx" download="test">下载文件</a>
```

The `download` attribute holds the filename for download.

If the file address is obtained asynchronously, meaning it is fetched from an API only when the download/export button is clicked, you can use JavaScript to dynamically insert the `<a>` tag. The demo is as follows:

After async getting the file path, execute the following code to trigger automatic download.

```js
// 创建a标签
let a = document.createElement('a')
// 定义下载名称
a.download = '文件名称'
// hide the tag
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

Drawback: This method is suitable only for downloading non-image and non-PDF files. When dealing with image or PDF files, the browser will open a preview instead of initiating the download.

https://blog.csdn.net/hfhwfw161226/article/details/105700504
