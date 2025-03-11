## Low-Code Concept

#### Why Low-Code?

Based on current visualization and model-driven concepts, low-code solutions can significantly reduce business delivery costs and provide a new development paradigm for business. **Visual building allows product and operations personnel to participate, which can greatly alleviate the human resource bottleneck of software developers and further promote deep cooperation between technology and business.**

#### Write Once, Run Anywhere

One-time coding, multi-platform adaptation for PC, mobile, mini-programs, and clients

#### Visual Development

Visual drag-and-drop components

## Micro-Frontend

Micro-frontend architecture has the following core values:

- **Technology Stack Independent**
  The main framework does not restrict the technology stack of accessing applications, micro-applications have complete autonomy

- Independent Development and Deployment
  Micro-application repositories are independent, front-end and back-end can be developed independently, and the main framework automatically completes synchronization updates after deployment

- Incremental Upgrade
  When facing various complex scenarios, it's usually difficult to perform a full technology stack upgrade or refactoring of an existing system. Micro-frontend is a very good means and strategy for implementing progressive refactoring

- Independent Runtime
  State isolation between each micro-application, runtime state not shared

**Micro-frontend architecture aims to solve the problem of application unmaintainability that occurs when a normal application evolves into a monolithic application over a relatively long time span due to the increase and changes in participating personnel and teams.**

What's the difference between widget-level micro-frontend applications and business components? — **The difference lies in whether your implementation is technology stack independent or not.**

**Applications should not have any direct or indirect coupling in terms of technology stack, dependencies, and implementation.**

https://qiankun.umijs.org/zh/guide

https://zhuanlan.zhihu.com/p/95085796

## Combining Visual Development with Micro-Frontend

How to view visual building platforms?

Most businesses are reluctant to try this approach to develop their core products, why? Simple, it's uncontrollable. The upper limit of my product is determined by the platform rather than my own coding ability, which is critical.

Especially for some core modules, future customized modifications might not be supported.

But what if there's a micro-frontend mechanism? The building platform only needs to implement related protocols, and platform-generated pages can be easily integrated into our own applications. **During development, we can choose to write pages that need strong control ourselves, and use visual generation for edge pages, without any psychological burden.**

## Cross-Platform Development Technology

### Background

Before choosing technology, first be clear about which platforms need to be crossed?

Web, mobile clients, PC clients, mini-programs

* For Web only, or Web and desktop cross-platform, Electron still has advantages, using Vue, React for UI, with a complete ecosystem, many developers, and acceptable performance
* If the platforms include iOS and Android, Flutter is probably the better cross-platform solution at the current stage (2021) and for the next few years

### Why Not Use H5 Hybrid Solution Based on Web

Although many platforms now support h5 development, like Electron framework for desktop client development,

However, performance issues are insurmountable. JS execution and layout/rendering are mutually exclusive and cannot run in parallel, causing too long JS execution tasks to affect normal rendering leading to stuttering. Additionally, W3C has too much historical baggage.

### Flutter

Flutter 2.0 released stable web development, supporting both HTML and Canvas rendering modes. It's currently one of the most comprehensive cross-platform development frameworks, using its own C++ engine for rendering interfaces at the bottom layer, not using webview, and unlike RN which uses system components. Simply put, each platform just provides Flutter with a canvas.

* High development efficiency
* Google's own product

Still has many issues

* Dart language, limited applicability, nesting hell, high maintenance cost
* Average performance experience
* Low community heat, poor ecosystem
* Once you go deep, must learn client development knowledge, relatively high requirements for developers

### React Native

How does RN achieve cross-platform? This actually relies entirely on React's vdom. Using JS to generate vdom, then mapping vdom to Native's layout structure, and finally letting Native render the view to achieve cross-platform development.

Simply put, it combines Web ecosystem with Native components, letting JS execute code and using Native components for rendering.

Putting JS execution, layout, and rendering (Native components) in three separate processes avoids interface stuttering when JS executes complex tasks. By abandoning many standards in CSS and only supporting partial flex layout capabilities, it reduces layout and rendering complexity.

Disadvantages:

- Inconsistent component and layout mechanisms between iOS/Android make cross-platform consistency difficult to guarantee
- Dependence on Native mechanisms also makes some CSS properties difficult to implement, such as the notorious z-index problem
- Borrows Web ecosystem but isn't completely Web ecosystem, many inconsistencies, most common complaint is inability to use familiar CSS layout methods
- Once you go deep, knowing only JS won't solve problems, must learn Java and OC, relatively high requirements for developers

### Mini-Programs

* Each platform launches its own framework
* Based on Webview rendering

### Reference Articles

https://www.cnblogs.com/skychx/p/cross-platform-tech.html

https://zhuanlan.zhihu.com/p/300680122

## About JSBridge

https://juejin.cn/post/6844903585268891662

## WebView

### What is it

Simply put, it's a container concept within APP. For better understanding, here are some online explanations

Explanation 1

On Android platform, there's a control in SDK called [WebView](https://developer.android.com/reference/android/webkit/WebView), on iOS/MacOS platform, there's a control in SDK called [WebView](https://developer.apple.com/documentation/webkit/webview), used for mobile APP to embed Web technology, load Web content, based on Webkit engine.

Explanation 2

It's a component encapsulated in the system SDK. It's a trimmed-down version of the browser with built-in Webkit kernel in phones, but without address bar and navigation bar, just purely displaying a web page interface.

### WebKit

WebKit is an open-source browser engine proposed by Apple, Safari and Chrome are both based on webkit engine (although Google later established Blink/Chromium based on WebKit, which is now Chrome browser's kernel)

Traditionally, WebKit included a web engine WebCore and a script engine JavaScriptCore, however, as JavaScript engines become increasingly independent, WebKit and WebCore are now basically used interchangeably (for example, Google Chrome uses V8 engine but still claims to be WebKit kernel).

Now WebKit kernel basically only includes the rendering engine, but traditionally should include JS engine

### Difference from Mobile Browser

Android, iOS platform browsers are also **based on Webkit engine**, are more complete browsers than WebView, after all WebView is just a control in system SDK.

### Why Slow

When we open a WebView page, the page often loads slowly for several seconds before showing the needed page, why?

When App first opens, **browser kernel is not initialized by default; only when creating WebView instance, will WebView's basic framework be created.**

So unlike browsers, **the first step of opening WebView in App is not establishing connection, but starting browser kernel**.

### Unique Advantage

Native APP needs new version to modify frontend content, while pages through webview method only need to modify html code or js files (if obtained from server, just need new files deployed), users just need to refresh to use updated version.

### Reference Articles:

https://tech.meituan.com/2017/06/09/webviewperf.html

https://juejin.cn/post/6932083257286590477

https://www.zhihu.com/question/40871670