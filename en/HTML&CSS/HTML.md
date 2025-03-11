## Standard Mode and Compatibility Mode

The term "DOCTYPE" is an abbreviation for "document type," and it is not an HTML tag with an opening or closing tag. It serves as a document type declaration in markup languages, specifically indicating the version in which the HTML document is written. The DOCTYPE declaration must be the first line of an HTML document, preceding the opening `<html>` tag, and it is case-insensitive.

Standard mode, also known as strict mode, and compatibility mode, alternatively referred to as quirks mode or mixed mode, are both modes defined in HTML4.01. The declaration references the Document Type Definition (DTD) since HTML4.01 is based on SGML. The DTD specifies the rules of the markup language, allowing browsers to accurately render content.

In contrast, HTML5 is not based on SGML, eliminating the need for DTD references. **HTML5 does not have distinct standard and compatibility modes; instead, it adheres to its own HTML standard. It is backward-compatible, meaning that specifying `<!DOCTYPE html>` as the first line of the document declares the use of the HTML5 standard.**

```html
html5
<!DOCTYPE html>

html4.01 triggers standard mode
<!-- HTML 4.01 Strict -->
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"  "http://www.w3.org/TR/html4/strict.dtd"> 

Triggers compatibility mode
<!-- HTML 4.01 Transitional -->
<!DOCTYPE HTML PUBLIC  "-//W3C//DTD HTML 4.01 Transitional//EN"  "http://www.w3.org/TR/html4/loose.dtd"> 
 
<!-- HTML 4.01 Frameset --> 
<!DOCTYPE HTML PUBLIC  "-//W3C//DTD HTML 4.01 Frameset//EN"  "http://www.w3.org/TR/html4/frameset.dtd"> 
```

**If the document lacks a DOCTYPE declaration, the browser will be unable to determine the type of the HTML or XHTML document, leading it to enter quirks mode. Additionally, in versions prior to IE6, the browser will always default to quirks mode.**

**In standard mode, the document's layout and JavaScript operations adhere to the highest standards supported by the browser. In compatibility mode, the page is displayed in a lenient, backward-compatible manner, emulating behaviors of older browsers to prevent site malfunctions. If the browser enters compatibility mode, it interprets and renders the page in its own way. Consequently, across different browsers, the displayed styles and effects may vary.**

References:

[The Role and Types of Doctype in HTML](https://www.liudaima.com/a/45.html)

[What's the Purpose of !Doctype at the Beginning of HTML Files?](https://www.jianshu.com/p/ce48b13a4e1e)

[Strict Mode vs Quirks Mode - How to Trigger These Modes and Their Significance](https://blog.csdn.net/binglingnew/article/details/17301433)

## New features in HTML 5

[Reference Link](https://www.cnblogs.com/ainyi/p/9777841.html)

1. Semantic Markup: header, footer, section, nav, aside, article
2. Enhanced Forms: input starts to support some types(color, date, datetime, email, month, number, range, search, tel, time, url, week)
3. New Form Types: datalist, keygen, output
4. New Form Attributes: placeholder, required, min and max, step, height and width, autofocus, multiple
5. audio, video
6. canvas
7. location
8. drag
9. storage: localStorage, sessionStorage
10. new events: onresize, ondrag, onscroll, onmousewheel, onerror, onplay, onpause
11. WebSocket

## Differences between HTML5 and HTML4.01

### Declaration

1. HTML5 uses the declaration as below

```html
<!DOCTYPE html>
```

### Standard

2. HTML5's document parsing is no longer based on the SGML (Standard Generalized Markup Language) standard; instead, it has its own set of standards.

As a result, there is no need for a DOCTYPE to reference a Document Type Definition (DTD) in HTML5. Consequently, HTML5 eliminates the distinction between strict mode and quirks mode. HTML5 adopts a relatively lenient syntax, and in its implementation, efforts have been made to achieve maximum backward compatibility.

### Tag

3. New Semantic Markup

```html
<header>, <footer>, <section>, <article>, <nav>, <hgroup>, <aside>, <figure>
```

Semantic interpretation: [IFE-NOTE: Page Structure Semantics](https://rainylog.com/post/ife-note-1/)

4. Deprecated embellishment tags to achieve a more thorough separation of style and structure.

```html
<big>, <u>, <font>, <basefont>, <center>, <s>, <tt>
```

5. Added `<audio>, <video>`
6. Added `<canvas>`

### Attribute

6. New Form Features

```html
<!-- Requires correctly formatted email address -->
<input type=email >
<!-- Requires correctly formatted URL address -->
<input type=url >
<!-- Requires number input, default has up/down buttons -->
<input type=number >
<!-- Time series, currently only supported by Opera and Chrome -->
<input type=date >
<input type=time >
<input type=datetime >
<input type=datetime-local >
<input type=month >
<input type=week >
<!-- Default placeholder text -->
<input type=text placeholder="your message" >
<!-- Default focus attribute -->
<input type=text autofocus="true" >
```

7. New attributes in other tags

```html
<!-- meta tag adds charset attribute -->
<meta charset="utf-8">
<!-- script tag adds async attribute -->
<script async></script>
```

8. Enable certain attribute names to default to boolean properties.

```html
<!-- Just writing attribute name defaults to true -->
<input type="checkbox"  checked/>
<!-- attribute="attribute" is also true -->
<input type="checkbox"  checked="checked"/>
```

### Storage

9. Introduced WebStorage, including localStorage and sessionStorage.

10. Introduced IndexedDB and Web SQL, allowing the creation of database tables and storage of data on the client side.
11. Introduced the Application Cache, enabling web caching for offline use. This is achieved by creating a cache manifest file to establish application caching, providing foundational technical support for Progressive Web Apps (PWAs).

## meta tag attributes

Commonly used to define page descriptions, keywords, last modification date, and other metadata. This metadata serves browsers (for page layout or reloading), search engines, and other web services.

### charset

Define the character set for a web document.

```html
<!-- Define document character set -->
<meta charset="utf-8" />
```

### name + content

```html
<!-- Web page author -->
<meta name="author" content="Open Source Tech Team"/>
<!-- Website URL -->
<meta name="website" content="https://sanyuan0704.github.io/frontend_daily_question/"/>
<!-- Copyright information -->
<meta name="copyright" content="2018-2019 demo.com"/>
<!-- Keywords for SEO -->
<meta name="keywords" content="meta,html"/>
<!-- Page description -->
<meta name="description" content="Page description"/>
<!-- Search engine indexing method, generally 'all', no need to delve deep -->
<meta name="robots" content="all" />
```

Especially for setting the viewport on mobile devices, the most commonly used.

```html
<!-- Common mobile viewport settings -->
<meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0, user-scalable=no"/>
<!-- 
  Viewport parameter details:
  width: width (number / device-width) (default 980 pixels)
  height: height (number / device-height)
  initial-scale: initial zoom ratio (range >0 to 10)
  minimum-scale: minimum zoom ratio allowed
  maximum-scale: maximum zoom ratio allowed
  user-scalable: whether user can manually scale (no,yes)
 -->
```

### http-equiv

```html
<!-- expires specifies when the web page expires. Once expired, must be downloaded from server. -->
<meta http-equiv="expires" content="Fri, 12 Jan 2020 18:18:18 GMT"/>
<!-- Wait certain time to refresh or redirect to other URL. 1 below means 1 second -->
<meta http-equiv="refresh" content="1; url=https://www.baidu.com"/>
<!-- Prevent browser from reading page from local cache, meaning page can't be accessed without network once left -->
<meta http-equiv="pragma" content="no-cache"/>
<!-- Another way to set cookies, can specify expiration time -->
<meta http-equiv="set-cookie" content="name=value expires=Fri, 12 Jan 2001 18:18:18 GMT,path=/"/>
<!-- Browser version to use -->
<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
<!-- For WebApp fullscreen mode, hide status bar/set status bar color, content values: default | black | black-translucent -->
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />
```

## Differences between link and @import

1. `<link>` is an HTML tag that not only loads CSS files but also has various use cases, like icon and font, `@import` is provided by CSS and serves the only purpose of importing style sheets.

2. Different importation code:

   * link

   ```html
   <link rel="stylesheet" href="style.css" type="text/css" />
   ```

   * @import

   ```html
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

## Differences between Canvas and SVG

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

```js
var w;
// Start timing
function startWorker()
{
  // Check if browser supports Web Worker
  if(typeof(Worker) !== "undefined") {
    if(typeof(w) == "undefined") {
      w = new Worker("demo_workers.js");
    }
    // Listen for messages
    w.onmessage = function (event) {
      document.getElementById("result").innerHTML=event.data;
    };
  }
  else {
   document.getElementById("result").innerHTML = "Your browser doesn