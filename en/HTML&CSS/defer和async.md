[Will the position of js affect the loading time of the first screen](https://blog.csdn.net/weixin_37722222/article/details/90021406)

[Mobile Performance Optimization: Page Visibility Time and Asynchronous Loading](https://fed.taobao.org/blog/taofed/do71ct/mobile-wpo-pageshow-async/)

[A Brief Discussion on async and defer in script Tags](https://www.cnblogs.com/jiasm/p/7683930.html)

[Rendering Pipeline: How Does CSS Affect White Screen Time During Initial Loading?](http://interview.poetries.top/browser/part5/lesson23.html)

[Differences Between DOMContentLoaded and load](https://www.cnblogs.com/caizhenbo/p/6679478.html)

[Impact and Reasons of Browser Resource Loading and Parsing](https://www.jianshu.com/p/2bc93efe0958)

[Download Order Issues Between Page JS Scripts and Resources like Images](https://www.cnblogs.com/wuguanglin/p/JSAndImgLoadOrder.html)

[In-Depth Browser Rendering Principles](https://github.com/ljianshu/Blog/issues/51)

### Regular script Tag

During document parsing, when encountering a script tag, page parsing stops for download (**However, Chrome optimizes this by quickly checking if there are other resources that need to be downloaded after encountering a script tag. If so, it will download those resources first before downloading the script's resource, saving some download time**).
Resource downloading occurs during parsing. Although script1 might load quickly, it remains suspended if script2 before it hasn't loaded & executed yet.
Page parsing continues only after both scripts have executed.

<img src="https://user-images.githubusercontent.com/9568094/31619989-a874ae42-b25b-11e7-9a80-e0f644f27849.png" style="zoom:33%;" />

<img src="https://user-images.githubusercontent.com/9568094/31621391-39849b1a-b25f-11e7-9301-641b1bc07155.png" style="zoom:33%;" />

### defer

HTML 4.0 specification, its purpose is to tell the browser to execute the specified script after DOM+CSSOM rendering is complete

```html
  <script defer src="xx.js"></script>
```

> - Browser starts parsing HTML webpage
> - During parsing, encounters script tag with defer attribute
> - Browser continues parsing HTML webpage, renders it after parsing completes, while downloading external script in parallel
> - Browser completes HTML webpage parsing, then executes downloaded script, after which triggers DOMContentLoaded

Downloaded script files execute before DOMContentLoaded event triggers (right after reading </html> tag), and execution order is guaranteed to match their appearance order on the page. So after adding defer attribute, domReady time isn't advanced, but it allows page to display faster.

Adding defer to scripts at the top of the page, in PC Chrome, has the same effect as putting the script at the bottom - the page displays first. However, for iOS Safari and iOS WebView, adding defer or putting script at bottom both result in long white screen time.

<img src="https://user-images.githubusercontent.com/9568094/31621324-046d4a44-b25f-11e7-9d15-fe4d6a5726ae.png" style="zoom:33%;" />

### async

HTML 5 specification, its purpose is to download script using another process, download doesn't block rendering, and execute immediately after download completes

```html
  <script async src="yy.js"></script>
```

> - Browser starts parsing HTML webpage
> - During parsing, encounters script tag with async attribute
> - Browser continues parsing HTML webpage while downloading external script in parallel. If parsing completes first, page displays and triggers DOMContentLoaded. In other words, DOMContentLoaded event triggering isn't affected by async script loading - it might trigger before script loading completes, but will definitely execute before load event
> - When script download completes, browser pauses HTML parsing to execute downloaded script
> - After script execution completes, browser resumes HTML parsing

async attribute ensures script downloads while browser continues rendering. However, async can't guarantee script execution order - whichever script downloads first executes first.

<img src="https://user-images.githubusercontent.com/9568094/31621170-b4cc0ef8-b25e-11e7-9980-99feeb9f5042.png" style="zoom:33%;" />

<img src="https://user-images.githubusercontent.com/9568094/31622216-6c37db9c-b261-11e7-8bd3-79e5d4ddd4d0.png" style="zoom:33%;" />

### How to Choose Between defer and async

- defer can guarantee execution order, async cannot
- async can trigger domReady (DOMContentLoaded) earlier, defer cannot
- defer still blocks rendering on iOS and some Android devices, causing long white screen time
- When script has both async and defer attributes, defer is ignored, browser behavior is determined by async attribute

### DOMContentLoaded vs load

DOMContentLoaded: Triggers after HTML document loads and all inline JS and synchronous code from external JS finishes executing, but some resources like images might not be visible yet.

load: Triggers after all resources in page DOM structure finish loading, including async JS scripts. Video, audio, flash don't affect load event triggering.

### About White Screen Time

- First screen time and DomContentLoad event don't necessarily have a fixed order, as first screen time counts when first screen images finish loading
- Early loading of all CSS is crucial for reducing first screen time
- Placing script tags at body bottom, with or without async/defer, won't affect first screen time but affects DomContentLoad and load timing, thus affecting when dependent code starts executing
- **White screen time: Time from entering URL and pressing enter until page starts showing content**
  - ```js
    (window.chrome.loadTimes().firstPaintTime - window.chrome.loadTimes().startLoadTime)*1000
    ```
- **First screen time: Time from entering URL and pressing enter until first screen content fully renders**
- **User operable time: domready trigger point, click events respond**
- **Total download time: window.onload trigger point**

### About Blocking

* CSS doesn't block DOM tree parsing (because CSSOM tree and DOM tree construction are parallel in GUI rendering thread)
* CSS blocks page rendering
* CSS blocks execution of subsequent JS statements because JS execution must wait for CSSOM tree construction as JS might operate on CSS, indirectly affecting DOMContentLoaded event triggering
* Why does JS block DOM parsing? Most direct accurate answer: **Because JS engine thread and GUI rendering thread are mutually exclusive, blocking is inevitable - rather than blocking, it's more accurate to say JS engine thread takes control from rendering process**
* **JS files not only block DOM construction but also cause CSSOM to block DOM construction**. Originally DOM and CSSOM construction don't affect each other, but once JavaScript is introduced, CSSOM starts blocking DOM construction because JavaScript can modify both DOM and styles (CSSOM). Since incomplete CSSOM is unusable, if JavaScript wants to access and modify CSSOM, it must have complete CSSOM when executing. This leads to a phenomenon where if browser hasn't completed CSSOM download and construction but we want to run script then, browser will delay script execution and DOM construction until CSSOM download and construction complete. In other words, **in this case, browser will first download and construct CSSOM, then execute JavaScript, finally continue DOM construction**.
* **DOMContentLoaded trigger doesn't require CSS to finish loading. However, with JS present, it must wait for JS synchronous code to finish executing before triggering DOMContentLoaded, and JS execution must wait for CSS parsing to complete, so CSS ends up affecting DOMContentLoaded triggering. CSS loading blocks execution of JS statements after it, JS before CSS doesn't need to wait for (care about) CSS.**

### Why Emphasize Putting CSS in Header and JS at Bottom?

Layout tree construction needs both DOM and CSSOM, so HTML and CSS both block rendering. Therefore, CSS should load early (e.g., in header) to shorten first render time.

Additionally, since CSS doesn't block document parsing but blocks document rendering, putting CSS in header helps generate CSSOM tree quickly. When subsequently rendering DOM, layout tree can be built in one go, requiring only one render; if CSS is placed later, DOM will be parsed once, then after CSS loads, previous DOM needs re-rendering, requiring two renders.

As for JS at bottom, it's because JS blocks parsing and rendering as mentioned before.

### First-Paint

People often mention putting JS at body bottom for optimization, because browser generates DOM tree by reading HTML code line by line, script tags at bottom won't affect previous page rendering. Question is, since page only renders after DOM tree fully generates, and browser must read all HTML to generate complete DOM tree, isn't it the same if script tags aren't at body bottom, since DOM tree generation needs entire document parsed?

![img](https://images2015.cnblogs.com/blog/746387/201704/746387-20170407181912191-1031407943.png)

Looking at Chrome's page rendering process, green line marks First Paint time. Why does First Paint occur when page paint should happen after render tree generates? Actually, modern browsers try to display content quickly for better user experience. They don't wait for all HTML parsing before building and laying out render tree, partial content gets parsed and displayed. **This means browsers can render incomplete DOM tree and CSSOM to quickly reduce white screen time**. If we put JS in header, JS will block DOM parsing, DOM content affects First Paint, causing First Paint delay. So we put JS later to reduce First Paint time, but this won't reduce DOMContentLoaded trigger time.

Before first script resource in body downloads, browser performs first render, merging DOM tree and CSSOM before this script tag into a Render tree, rendering to page. **This is the time point from white screen to first render, quite crucial**.

### Browser's Concurrent Download Thread Limit for Same Domain Resources, Chrome is 6

Browser won't exceed 6 concurrent downloads for **same domain**. Excess resources wait in queue, which is why we distribute resources across different domains to fully utilize this mechanism, maximizing concurrent resource downloads to complete page rendering quickly.

### A Recent Explanation

Browser triggers page rendering when encountering script tags
Many might not know this detail, which actually explains why JS execution waits for CSS download. First an example, HTML body structure:

```html
<body>
    <div></div>
    <script src="/js/sleep3000-logDiv.js"></script>
    <style>
        div {
            background: lightgrey;
        }
    </style>
    <script src="/js/sleep5000-logDiv.js"></script>
    <link rel="stylesheet" href="/css/common.css">
</body>
```

This is an extreme example but reveals important information. How will page behave?

Answer: light green first, then light grey, finally light blue. This shows browser renders page each time it encounters script tag. Same reason - browser doesn't know script content, so when encountering script, it must render page first to ensure script can get latest DOM element information, even if script might not need this information.

Summary
From above, we conclude:

CSS doesn't block DOM parsing but blocks DOM rendering.
JS blocks DOM parsing, but browser "peeks" at DOM, pre-downloading related resources.
Browser triggers page rendering when encountering script tag without defer/async attribute, so if previous CSS resources haven't loaded, browser waits for them to load before executing script.

https://ljf0113.github.io/2017/09/24/how-css-and-js-block-dom/

https://juejin.cn/post/6844903667733118983