## What's the difference between using setTimeout() to simulate setInterval() and using setInterval() directly?

Each task produced by setTimeout is directly pushed to the task queue;

Whereas setInterval checks (to see if the previous task is still in the queue) before pushing each task to the task queue. Only when there are no other instances of this timer's task in the queue will the current round's task be added to the task queue.

Therefore, using setInterval may result in some intervals being skipped, and the total execution time may be less than expected.

Using setTimeout to simulate setInterval solves this problem. No new timer code will be inserted into the queue before the previous timer code has finished executing, ensuring that there are no missing intervals. Moreover, it guarantees that at least the specified time interval must be waited before the next timer code executes.

## Differences between animation and transition

1. transition is a transition effect that **focuses on CSS property changes**. It's the process of style value changes with only a start and end state; animation, also known as keyframes, emphasizes process and control, and can set intermediate frame states through combination with keyframes;

2. In other words, animation can set many frames in combination with keyframe, while transition only has two frames;

3. animation with @keyframe can trigger the process without time triggers, while transition needs to be triggered through hover or js events;

4. animation can set many properties, such as loop count, animation end state, etc., while transition can only be triggered once;

## Image Lazy Loading and Preloading

* Lazy Loading

**Principle**

For img elements in the page, if there's no src attribute, the browser won't send requests to download images. Only when the image path is set through JS will the browser send the request.
The principle of lazy loading is to first use a placeholder image for all images in the page, store the real path in the element's "data-url" attribute (you can name it something memorable), and when needed, retrieve it and set it through JS.

**Implementation Steps**

1. First, don't put the image address in the src attribute, but in another attribute (data-url).
2. After the page loads, determine if the image is in the user's viewport based on scrollTop. If it is, take the value from the data-original attribute and put it in the src attribute.
3. In the scroll event, repeatedly check if images enter the viewport. If they do, set the value from the data-url attribute to the src attribute through JS.

I've written this by hand before...

**Advantages**

Faster page loading speed, can reduce server pressure, saves bandwidth, better user experience

* Preloading

Load images in advance, so they can be rendered directly from local cache when users need to view them.

Preloading can be said to sacrifice front-end server performance in exchange for better user experience, allowing user operations to get the fastest response.

Simple implementation:

```html
<div class="hidden">  
    <script type="text/javascript">   
            var images = new Array()  
            function preload() {  
                for (i = 0; i < preload.arguments.length; i++) {  
                    images[i] = new Image()  
                    images[i].src = preload.arguments[i]  
                }  
            }  
            preload(  
                "http://domain.tld/gallery/image-001.jpg",  
                "http://domain.tld/gallery/image-002.jpg",  
                "http://domain.tld/gallery/image-003.jpg"  
            )  
    </script>  
</div>
```

* Resource Preloading

Preloading is actually a declarative `fetch`, forcing the browser to request resources without blocking the `onload` event. You can use the following code to enable preloading

```html
<link rel="preload" href="http://example.com">
```

> Preloading can reduce first screen loading time to some extent because it can delay loading some important files that don't affect the first screen. The only drawback is poor compatibility

* Prerendering

**Pre-render downloaded files in the background** through prerendering. You can use the following code to enable prerendering

```html
<link rel="prerender" href="http://example.com">
```

Although prerendering can improve page loading speed, you need to **ensure that the page is likely to be opened by users later**, otherwise it's just wasting resources on rendering.

Reference articles:

https://juejin.im/entry/6844903512514494472

https://www.jianshu.com/p/4876a4fe7731

## Differences between JS Animation and CSS Animation

1. In terms of code complexity, for simple animations, CSS implementation will be simpler, JS more complex. For complex animations, CSS code becomes verbose, while JS implementation is better. That is, CSS is more suitable for implementing simple animations, JS for complex animations.

2. In terms of control over the animation during runtime, JS is more flexible, able to control animation pause, cancel, terminate, etc., while CSS animations cannot add events and can only set fixed nodes for what kind of transition animation.

3. In terms of compatibility, CSS has browser compatibility issues, while JS mostly doesn't.

4. In terms of performance, CSS animations are relatively better, as CSS animations are parsed directly through the GUI rendering thread, while JS animations need to be parsed through the JS engine thread code before GUI parsing and rendering.

Additional:

**`window.requestAnimationFrame(callback)`** tells the browser that you want to perform an animation and requests that the browser call a specified callback function to update the animation before the next repaint. The method needs to be passed a callback function as a parameter, which will be executed before the browser's next repaint.

## Advantages of requestAnimationFrame for Implementing Animations

The window.requestAnimationFrame() method tells the browser that you wish to perform an animation and requests that the browser **call a specified function to update the animation before the next repaint**. The method takes a callback function as a parameter, which will be **called before the browser repaints**.

1. Compared to setTimeout, requestAnimationFrame's biggest advantage is that the system decides when to execute the callback function. More specifically, if the screen refresh rate is 60Hz, then the callback function is executed every 16.7ms, if the refresh rate is 75Hz, then this time interval becomes 1000/75=13.3ms. In other words, requestAnimationFrame's pace follows the system's refresh pace. It can ensure the callback function is executed only once in each screen refresh interval, which won't cause frame dropping phenomena or animation stuttering issues.

2. CPU energy saving: For animations implemented with setTimeout, when the page is hidden or minimized, setTimeout still executes animation tasks in the background. Since the page is in an invisible or unavailable state at this time, refreshing the animation is meaningless and just wastes CPU resources. However, requestAnimationFrame is completely different. When the page is in an inactive state, the screen refresh task for that page will also be suspended by the system, so requestAnimationFrame that follows the system's pace will also stop rendering. When the page is activated, the animation continues from where it last stopped, effectively saving CPU overhead.

3. Function throttling: In high-frequency events (resize, scroll, etc.), to prevent multiple function executions within one refresh interval, using requestAnimationFrame can ensure the function is executed only once within each refresh interval. This not only ensures smoothness but also better saves function execution overhead. Multiple function executions within one refresh interval are meaningless because the display refreshes every 16.7ms, and multiple drawings won't be reflected on the screen.

Reference articles:

[Advantages of requestAnimationFrame for Implementing Animations](https://juejin.im/post/6844903933970743309)

[What You Know About requestAnimationFrame [From 0 to 0.1]](https://juejin.im/post/6844903761102536718#heading-0)

## Key Issues with requestAnimationFrame

* **Does each round of Event Loop come with rendering?**

Not every round of event loop necessarily corresponds to a browser render, it depends on the screen refresh rate, page performance, and whether the page is running in the background.

According to some conventional understanding, rendering should be interspersed between macro tasks, and timer tasks are a typical macro task. Let's look at the following code:

```js
setTimeout(() => {
  console.log("2")
  queueMicrotask(() => console.log("3"))
  requestAnimationFrame(() => console.log("5"))
})
setTimeout(() => {
  console.log("4")
  requestAnimationFrame(() => console.log("6"))
})

queueMicrotask(() => console.log("1"))
```

Intuitively, the order should be:

```text
1,2,3,5,4,6
```

That is, each macro task is followed immediately by a render.

In reality, it won't be like that, the browser will merge these two timer tasks:

```text
1,2,3,4,5,6
```

* **In which phase does `requestAnimationFrame` execute, before or after rendering? Before or after `microTask`?**

The callbacks of `requestAnimationFrame` have two characteristics:

1. Called before rerendering.
2. Very likely not to be called after macro tasks.

Why call before rerendering? Because `rAF` is the officially recommended API for creating smooth animations, and animations inevitably involve changing the DOM. If DOM changes were made after rendering, they could only be drawn in the next rendering opportunity, which would obviously be unreasonable.

`rAF` gives you one last chance to change DOM properties before the browser decides to render, and then quickly helps you present it in the following paint, so this is the best choice for smooth animations.

**Browser frame execution order (not all steps are necessarily executed): 1 macro task, all micro tasks, requestAnimationCallback, render, requestIdleCallback**

https://zhuanlan.zhihu.com/p/142742003

## queueMicrotask()

To allow third-party libraries, frameworks, and polyfills to use microtasks, [`Window`](https://developer.mozilla.org/en-US/docs/Web/API/Window) exposes the [`queueMicrotask()`](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/queueMicrotask) method.

```js
queueMicrotask(() => {
  /* Code to run in microtask */
});
```

## Differences between onclick and addEventListener in JS

https://blog.csdn.net/weixin_42881768/article/details/104856558

1. onclick can only register one event for an element. If there are multiple events, later events will override previous ones;

2. addEventListener allows registering multiple listener monitors for one event, and added events won't override existing ones;
3. addEventListener can control the listener's trigger phase (capture/bubble). For multiple identical event handlers, they won't trigger repeatedly, and don't need to be manually cleared using removeEventListener;
4. IE8 and below, Opera 7.0 and earlier versions can use attachEvent(eventName, handler) and detachEvent(eventName, handler). Note: event names include 'on'.

## Why can string, boolean values, etc. in JS use Object prototype methods like toString?

a is a string, equivalent to an instance object of String (see the reference section for details). The String object has the toString method, so a inherits String's prototype methods:

```jsx
typeof a === 'string'
// true
a.hasOwnProperty('toString')
// false
a.__proto__ === String.prototype
// true
String.prototype.hasOwnProperty('toString')
// true
a.__proto__.hasOwnProperty('toString')
// true
```

```js
let s = '1'
console.log(s.__proto__ === String.prototype);	// true
console.log(typeof String);	// function
console.log(String.prototype.__proto__ === Object.prototype);	// true
console.log(typeof String.__proto__);	// function
console.log(String.__proto__ === Function.__proto__);	// true
console.log(String.__proto__ === Function.prototype);	// true
console.log(Function.__proto__ === Function.prototype);	// true
console.log(Function.prototype.__proto__ === Object.prototype);	// true
console.log(Object.__proto__ === Function.prototype);	// true
```

String is an instance object of Function, Function is an instance object of itself, and Function's prototype object's **__proto__** ultimately points to Object's prototype object

## Differences between String() and toString()

1. String() is a global constructor, while toString() is a method on Object's prototype

2. Different handling of null and undefined

   ```js
   String(null) 	// null
   String(undefined) // undefined
   null.toString()	// Uncaught TypeError: Cannot read property 'toString' of null
   null.toString()	// Uncaught TypeError: Cannot read property 'toString' of undefined
   ```

3. toString() can convert numbers to strings in specified bases

## Differences between String() and new String()

String() constructs a primitive string type, while new creates an object

```js
let s1 = String('a')
let s2 = new String('a')
console.log('s1':s1);
console.log('s2':s2);
/*
s1:a
s2:String {"a"}
	0: "a"
	length: 1
	__proto__: String
  [[PrimitiveValue]]: "a"
*/
```

Here we need to introduce the concept of wrapper objects, which are the three native objects `Number`, `String`, and `Boolean` that correspond to numbers, strings, and boolean values respectively. They can wrap primitive values into objects.

The main **purposes** of wrapper objects are: first, to ensure that JavaScript objects cover all values, and second, to allow primitive types to conveniently call certain methods.

**When these three objects are used as constructors (with new), they can convert primitive types into objects; when used as regular functions (without new), they can convert any type of value into primitive type values.**

Primitive type values can automatically be treated as objects when calling various object methods and parameters. At this time, the JavaScript engine will automatically convert the primitive type value into a wrapper object instance, and destroy the instance immediately after use.

```dart
var str = 'abc';
str.length // 3
// equivalent to
var strObj = new String(str)
// String {
//   0: "a", 1: "b", 2: "c", length: 3, [[PrimitiveValue]]: "abc"
// }
strObj.length // 3
strObj = null
```

## WebWorker, JS Multi-threading?

The JS engine is single-threaded, and long JS execution times can block the page. So is JS really powerless when it comes to CPU-intensive computations?

That's why HTML5 later supported `Web Worker`.

MDN's official explanation is:

```
Web Workers provide a simple means for web content to run scripts in background threads. The worker thread can perform tasks without interfering with the user interface.

A worker is created using a constructor (e.g. Worker()) that runs a named JavaScript file.

This file contains code that will run in the worker thread; workers run in another global context that's different from the current window.

Thus, using the window shortcut to get the current global scope (instead of self) will return an error in a Worker.
```

Understand it this way:

- When creating a Worker, the JS engine requests the browser to open a child thread (the child thread is opened by the browser, completely controlled by the main thread, and cannot manipulate the DOM)
- The JS engine thread communicates with the worker thread in a specific way (postMessage API, needs to interact with specific data through serialized objects)

So, if there is very time-consuming work, please open a separate Worker thread. No matter how the work turns upside down inside, it won't affect the JS engine's main thread.
Just wait until the result is calculated, then communicate the result to the main thread, perfect!

And note that **the JS engine is single-threaded**, this essential point remains unchanged. Worker can be understood as an external plugin opened by the browser for the JS engine, specifically used to solve those massive computation problems.

## JS Object Keys

In JavaScript, all object keys are strings (unless the object is a Symbol). Even though we may not define them as strings, they are always converted to strings under the hood.

```js
let map = {
  0: 'zzzzz'
}
let s = Object.keys(map)
console.log(map[0])	// zzzzz
console.log(map['0'])	// zzzzz
console.log(typeof s[0])	// string
```

## The Third Parameter of setTimeout

The third and subsequent parameters of setTimeout serve as parameters for the first function of setTimeout

```js
for (var i = 0; i < 3; i++) {
  setTimeout(j => {
    console.log(j)
  }, i * 1000, i)
}
// 0 1 2
```

```js
function f(a, b) {
  console.log(a + b)
}

setTimeout(f, 0, 2, 3)
// 5
```

## Macrotasks and Microtasks

* Macrotasks

| #                       | Browser | Node |
| :---------------------- | :-----: | :--: |
| `I/O`                   |    ✅    |  ✅   |
| `setTimeout`            |    ✅    |  ✅   |
| `setInterval`           |    ✅    |  ✅   |
| `setImmediate`          |    ❌    |  ✅   |
| `requestAnimationFrame` |    ✅    |  ❌   |

*Some places will list `UI Rendering` as a macrotask, but after reading the [HTML specification document](https://html.spec.whatwg.org/multipage/webappapis.html#event-loop-processing-model), it's clearly a parallel operation step with microtasks*
*Let's consider `requestAnimationFrame` as a macrotask. According to [MDN's definition](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestAnimationFrame), it's an operation executed before the next page repaint, and repaint exists as a step of macrotask, and this step is executed after microtasks*

* Microtasks

| #                            | Browser | Node |
| :--------------------------- | :-----: | :--: |
| `process.nextTick`           |    ❌    |  ✅   |
| `MutationObserver`           |    ✅    |  ❌   |
| `Promise.then catch finally` |    ✅    |  ✅   |

## What SEO Aspects Should Frontend Developers Pay Attention To?

1. Appropriate title, description, keywords: Search engines' weight for these three items gradually decreases. The title should emphasize key points, important keywords should appear no more than twice and should be placed near the front. The description should highly summarize the page content, be of appropriate length, not excessively stack keywords, and be different for different pages. Keywords should just list out important keywords.
2. Semantic HTML code to make it easier for search engines to understand the webpage.
3. Important content's HTML code should be placed first, as search engines crawl HTML from top to bottom, and some search engines have length limits for crawling, ensuring important content will definitely be crawled.
4. Don't output important content using JS, as crawlers won't execute JS to get content.
5. Use iframes sparingly, as search engines won't crawl content within iframes.
6. Non-decorative images must have alt attributes.
7. Improve website speed: website speed is an important indicator in search engine ranking.

## How to Better Render Tens of Thousands of Records from Backend to Frontend?

Through virtual scrolling

**The principle of this technology is to only render content in the visible area, completely not rendering content in non-visible areas, and replacing rendered content in real-time when users scroll.**

Even if the list is very long, there are only ever a few DOM elements being rendered, and when we scroll the page, the DOM is updated in real-time.

["Frontend Advanced" High-performance Rendering of 100,000 Records (Virtual List)](https://juejin.im/post/6844903982742110216#heading-0)

## Differences between block, inline, and inline-block

display: block

1. Block elements occupy their own line, multiple block elements each start on a new line
2. By default, block elements automatically fill their parent element's width
3. Block elements can have width and height properties set, and even with width set, they still occupy their own line
4. Block elements can have margin and padding properties set

display: inline

1. Inline elements don't occupy their own line, multiple adjacent inline elements arrange in the same line until they can't fit, then wrap to a new line
2. Width changes with the element's content
3. Setting width and height properties on inline elements has no effect
4. For inline elements' margin and padding properties, horizontal margin-left and margin-right will create spacing effects; but vertical margin-top and margin-bottom won't create spacing effects. Padding properties will create spacing effects

display: inline-block

Simply put, it presents the object as an inline object, but the object's content is presented as a block object. Subsequent inline objects will be arranged in the same line. For example, we can give a link (a element) an inline-block property value, making it have both block's width and height characteristics and inline's same-line characteristics.

## img Tag's title and alt

title is the text displayed when the mouse moves over the element, applicable to multiple tags, while alt is the content displayed when the image fails to load, only applicable to img tags

## Waterfall Layout

**Characteristics of waterfall layout:**

- Content boxes have fixed width but variable height.
- Content boxes arrange from left to right, and when a row is full, remaining content boxes arrange in order after the shortest column.

[Implementing Waterfall Effect with Native JavaScript](https://zhuanlan.zhihu.com/p/55575862)

## JS Getting Current Domain, URL, Relative Path and Parameters

There are 2 methods to get the current domain with JS

```js
var domain = document.domain;
var domain = window.location.host;
```

4 methods to get the current URL

```js
var url = document.URL;
var url = document.location;
var url = window.location.href;
var url = self.location.href;
```

Method to get current URL parameters

```js
function GetUrlPara() {
  var url = document.location.toString();
  var arrUrl = url.split("?");
  var para = arrUrl[1];
  return para;
}
```
## Weak References

WeakMap only weakly references the keys, not the values. Values remain normal references.

```javascript
const wm = new WeakMap();
let key = {};
let obj = {foo: 1};

wm.set(key, obj);
obj = null;
wm.get(key)
// Object {foo: 1}
```

In the above code, the value `obj` is a normal reference. Therefore, even if we remove the reference to `obj` outside the WeakMap, the reference inside the WeakMap still exists.

## Reasons for Page Lag in Frontend Development

* **Delayed rendering causing frame drops**

  Due to the event loop mechanism, page rendering only occurs after each macro and micro task completes. Prolonged blocking of the JavaScript thread can lead to rendering delays and page lag.

* **Excessive page reflows and repaints**

* Resource loading blocking

* **Memory leaks causing high memory usage**

## JSBridge

JSBridge provides JavaScript interfaces to call native functions, enabling hybrid apps to access native features like geolocation, camera, and payments. It establishes bidirectional communication between native and web environments.

## Ensuring CDN Code is Up-to-Date

Add [contentHash] to output filenames - file name changes when content changes.

Differences between [hash] and [contentHash]:

1. hash (Same hash for all files, changes with any file modification - bad for caching)
2. chunkhash (Same hash for chunks, changes when any file in chunk changes)
3. contenthash (File-specific hash, ideal for caching CSS files)

## Handling Unsupported ES5 Features in TypeScript

Introduce corresponding polyfills for unsupported features.

## Cross-platform Ad Targeting (JD -> Douyin)

Although cookies can't be shared cross-domain, user identifiers like IMEI, MEID, ICCID, or phone numbers can be used with third-party platforms for data sharing.

## WebAssembly

WebAssembly serves as a compilation target for languages like C/C++/Rust, enabling high-performance web applications like Google Earth.

## File Upload Progress Tracking

Using XHR:
```javascript
xhr.onprogress = function(e) {
  if (e.lengthComputable) {
    console.log(`Progress: ${e.loaded}/${e.total} bytes`);
  }
};
```

Using Vue + Axios:
```javascript
function uploadConfig(e) {
  let formData = new FormData();
  formData.append('file', e.target.files[0]);
  axios.post(url, formData, {
    headers: {'Content-Type': 'multipart/form-data'}
  });
}
```

## iframe Usage

**Pros:**
1. Preserves embedded page structure
2. Easy centralized updates
3. Code reuse for headers/footers

**Cons:**
1. Management complexity
2. SEO unfriendly
3. Poor mobile compatibility

## escape vs encodeURI vs encodeURIComponent

1. `escape` (deprecated) encodes strings but not URLs
2. `encodeURI` preserves: `A-Za-z0-9;,/?:@&=+$-#_.!~*'()`
3. `encodeURIComponent` preserves: `A-Za-z0-9-_.!~*'()`

Usage examples:
```javascript
encodeURI("http://example.com/path with spaces");
// "http://example.com/path%20with%20spaces"

encodeURIComponent("param=value&");
// "param%3Dvalue%26"
```

## SSO Single Sign-On

1. User accesses app system which requires login
2. Redirect to CAS server (SSO login system)
3. After authentication, SSO writes login status to session and sets cookie in SSO domain
4. SSO generates Service Ticket (ST) and redirects back to app system with ST parameter
5. App system verifies ST with SSO backend
6. Upon validation, app system establishes session and sets cookie in its domain

Process for accessing app2 system:
1. User accesses app2 system which redirects to SSO
2. SSO recognizes existing login and generates new ST
3. Redirect to app2 with ST parameter
4. app2 validates ST with SSO backend
5. Establishes session and sets cookie

**Security Consideration:**
Directly returning user info via callback without ST validation is dangerous. Malicious users could forge user info by manually entering callback URLs with fabricated parameters.

## URL Encoding Best Practices

When encoding URL parameters:
```javascript
let param = "http://www.cnblogs.com/season-huang/";
let encodedParam = encodeURIComponent(param);
let url = `http://www.cnblogs.com?next=${encodedParam}`;
console.log(url); // "http://www.cnblogs.com?next=http%3A%2F%2Fwww.cnblogs.com%2Fseason-huang%2F"
```

Using encodeURIComponent ensures proper encoding of special characters like '/' and ':'.

## Security Implications

Never trust unverified callback parameters. Always validate service tickets through backend communication with SSO server to prevent forged authentication attempts.