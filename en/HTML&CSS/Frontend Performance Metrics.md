## Document Loading Related

### Time to First Byte (TTFB)

The time from when the browser **starts requesting the page until it receives the first byte**, this time period includes DNS lookup, TCP connection, and SSL connection.

### DomContentLoaded (DCL)

The time when the DomContentLoaded event is triggered. When the **HTML** document has been completely loaded and parsed, the DOMContentLoaded event is triggered, **without waiting for CSS stylesheets, images, async scripts, and sub-frames to finish loading.**

### Load (L)

The time when the onLoad event is triggered. The onLoad event is only triggered after all page resources (such as images, CSS) have finished loading.

## Content Rendering Related

### First Paint (FP)

The time from start of loading until the browser **first paints pixels** to the screen

This change might be a simple background color update or inconspicuous content, it doesn't indicate page content completeness, and might report a time when no visible content was painted.

### First Contentful Paint (FCP)

The time when the browser **first renders content from the DOM**, content must be text, images (including background images), non-white canvas or SVG, also includes text with loading web fonts.

The so-called white screen time can be described by either FP or FCP, there's no strict standard

### First Meaningful Paint (FMP)

The time when the **main content of the page is painted** to the screen

### Largest Contentful Paint (LCP)

The time when the **largest content element in the viewport is rendered** to the screen

### First Screen Paint (FSP)

The time from when the page starts loading until all first-screen content is completely painted, when users can see all content in the first screen.

**This is also known as first screen time**

## Interaction Response Related

### First CPU Idle (FCI)

The time when the page first becomes able to respond to user input.

### Time to Interactive (TTI)

Represents the time point when the webpage first **becomes fully interactive**, the browser can now continuously respond to user input with smooth interaction

Both FCI and TTI are times when the page can respond to user input. FCI occurs when **users can start** interacting with the page; TTI occurs when **users can fully (sustainably)** interact with the page.

Reference Articles:

[Frontend Performance Optimization Guide [7]--Web Performance Metrics](https://juejin.cn/post/6844904153869713416#heading-0)

## Monitoring Methods

### Synthetic Monitoring

Synthetic monitoring? It's submitting a page that needs performance auditing in a simulated scenario, running your page through a series of tools and rules to extract performance metrics and generate an audit report.

### Real User Monitoring

Real User Monitoring is a passive monitoring technology, it's an application service where monitored web applications integrate with the service through methods like SDKs, collecting and reporting performance metrics data from real user visits and interactions, forming performance analysis reports after data cleaning and processing.

https://www.infoq.cn/article/dxa8am44oz*lukk5ufhy