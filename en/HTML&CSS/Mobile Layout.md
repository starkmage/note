## Types of Pixels

### Device Pixels (Physical Pixels)

The physical pixels of the device screen, which are fixed in number for any device.

* Screen Resolution

  `iPhone XS Max` and `iPhone SE` have resolutions of `2688 x 1242` and `1136 x 640` respectively. This indicates the number of physical pixels vertically and horizontally on the phone

* Image Resolution

  An image with resolution `800 x 400` means it has `800` and `400` physical pixels vertically and horizontally

* PPI

  Pixels per inch. When describing images, higher PPI means better image quality; when describing screens, higher PPI means clearer screen.

### Device Independent Pixels

Emerged after Steve Jobs introduced Retina display, specifically after iPhone4. In iPhone4's Retina display, `2x2` device pixels are used as `1` device independent pixel, making the screen look more refined while keeping element sizes unchanged.

Opening Chrome's developer tools, we can simulate display conditions for various phone models. Each model shows a size, like `iPhone X` showing `375x812`. Actually, iPhone X's resolution is much higher than this - what's shown here is device independent pixels.

* **Device Pixel Ratio**

  DPR is the ratio of physical pixels to device independent pixels. Can also be said to equal device pixel count / ideal viewport pixel count

  In web, browsers provide `window.devicePixelRatio` to help us get `dpr`. In CSS, we can use media query `min-device-pixel-ratio`.

  We can see that a device's DPR is fixed.

### CSS Pixels

When writing `CSS`, we most commonly use `px` unit, i.e., `CSS pixels`. When page zoom is 1, one `CSS pixel` equals one device independent pixel.

But `CSS pixels` are easily changed. When users zoom in browser, `CSS pixels` get enlarged, making one `CSS pixel` span more physical pixels.

Page zoom factor = CSS pixels / device independent pixels.

**When you set width: 200px for an element, this element's width spans 200 CSS pixels**. But it doesn't necessarily span 200 device pixels - how many device pixels it spans depends on **phone screen characteristics** and **user zoom**.

## Viewport

### Layout Viewport

Layout viewport is the reference window for webpage layout. On `PC` browsers, layout viewport equals current browser window size (excluding `borders`, `margins`, scrollbars).

On mobile, layout viewport is given a default value, mostly `980px`, ensuring `PC` webpages can display on mobile browsers, though very small, and users can manually zoom webpage.

We can get layout viewport size by calling `document.documentElement.clientWidth / clientHeight`.

### Visual Viewport

The area users actually see through screen.

Visual viewport defaults to current browser window size (including scrollbar width).

When users zoom browser, layout viewport size doesn't change, so page layout remains unchanged, but zoom changes visual viewport size.

For example: When user zooms browser window to `200%`, CSS pixels in browser window enlarge with visual viewport, making one `CSS` pixel span more physical pixels, thus reducing visual viewport width/height by half.

We can get visual viewport size by calling `window.innerWidth / innerHeight`.

### Ideal Viewport

Ideal viewport can be understood as: **ideal layout viewport**. Ideal viewport width doesn't change, like iPhone 5's ideal viewport width is 320 px, unrelated to zoom.

Following code tells mobile browser to set layout viewport to ideal viewport:

```html
<meta name="viewport" content="width=device-width" />
```

Above code tells browser: **Set layout viewport width to ideal viewport.** So width here means layout viewport width, device-width actually is ideal viewport width.

Earlier when discussing `CSS pixels` mentioned `page zoom factor = CSS pixels / device independent pixels`, actually saying `page zoom factor = ideal viewport width / visual viewport width` is more accurate.

So, when page zoom is `100%`, `CSS pixels = device independent pixels`, `ideal viewport = visual viewport`.

We can get ideal viewport size by calling `screen.width / height`.

## meta viewport

We can use `viewport` in `<meta>` element to help set viewport, zoom etc., giving mobile better display effect.

```html
<meta name="viewport" content="width=device-width; initial-scale=1; maximum-scale=1; minimum-scale=1; user-scalable=no;">
```

Above is a `viewport` configuration, let's look at specific meanings:

|     `Value`     |           Possible Values            |                           Description                            |
| :-------------: | :-------------------------: | :-------------------------------------------------------: |
|     `width`     |   positive integer or `device-width`    |      Defines layout viewport width in `pixels`.      |
|    `height`     |   positive integer or `device-height`   |      Defines layout viewport height in `pixels`.      |
| `initial-scale` |        `0.0 - 10.0`         |                  Defines initial page zoom ratio.                   |
| `minimum-scale` |        `0.0 - 10.0`         |   Defines minimum zoom value; must be less than or equal to `maximum-scale`.   |
| `maximum-scale` |        `0.0 - 10.0`         |   Defines maximum zoom value; must be greater than or equal to `minimum-scale`.   |
| `user-scalable` | boolean value (`yes` or `no`) | If set to `no`, users can't zoom webpage. Default is yes. |

## Zoom

Above mentioned `width` can determine layout viewport width, actually it's not only determining factor, setting `initial-scale` might also affect layout viewport, because **layout viewport width takes maximum of `width` and visual viewport width.**

For example: If phone's ideal viewport width is `400px`, set `width=device-width`, `initial-scale=2`, then `visual viewport width = ideal viewport width / initial-scale` i.e. `200px`, layout viewport takes maximum value i.e. `device-width` `400px`.

If set `width=device-width`, `initial-scale=0.5`, then `visual viewport width = ideal viewport width / initial-scale` i.e. `800px`, layout viewport takes maximum value i.e. `800px`.

We can see ideal viewport doesn't change, while visual viewport changes with zoom.

## Two Major Mobile Layout Solutions

First about design draft, taking iPhone6's design size (750px) as standard for explanation, if design has 200px wide red square, we definitely can't directly write 200px in CSS style.

**Design draft size is based on device pixels**. Since our design is based on iPhone6, our design size equals iPhone6's device pixel size, 750px, while our CSS styles are calculated based on layout viewport size. Since we wrote following meta tag in html page:

```html
<meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=no" />
```

*width=device-width* makes layout viewport size equal ideal viewport. Because iPhone6's DPR (Device Pixel Ratio) is 2, device pixels 750, so iPhone6's ideal viewport size is 375px. So above code ultimately: makes our layout viewport width 375px.

Design total width is 750px, element width 200px, but when actually making page, layout viewport width is 375px, exactly half of design. So we can't directly use pixel sizes measured from design, proportionally, we should divide measured sizes by 2 to get sizes for CSS layout, accordingly, dividing 200px by 2 gets 100px. When coding set red square width to 100px.

That's one method, if don't want to divide by 2, another method is setting zoom ratio to 0.5, then 1 CSS pixel width = 1 device pixel width, making layout viewport width 750px, so we can directly use element widths from design.

Above only applies to iPhone 6 display, on iPhone 5, we'll find red square width still 100px, making proportion wrong. If we want to adapt all mobile devices, must set dynamically, this uses rem, rem is relative size unit, relative to html tag font size unit, example:
If html font-size = 18px;
Then 1rem = 18px, need remember, rem is based on html tag font size.

Below summarizing Taobao and NetEase's two solutions:

### Mobile Taobao

1. JS dynamically modifies meta tag, making layout viewport size equal design size, i.e. device pixel size,

```js
var scale = 1 / window.devicePixelRatio;
document.querySelector('meta[name="viewport"]').setAttribute('content','width=device-width,initial-scale=' + scale + ', maximum-scale=' + scale + ', minimum-scale=' + scale + ', user-scalable=no');
```

2. Dynamically set html font size:

```js
document.documentElement.style.fontSize = document.documentElement.clientWidth / 10 + 'px';
```

3. Convert design sizes to rem

Element rem size = element pixel size measured from design / dynamically set html tag font-size value.

Taking iPhone6 example, html tag font-size value equals 750 / 10 = 75px, making 1rem = 75px, so red square 200px converts to rem unit as 200 / 75 = 2.6666667rem.

What about iPhone5? Because iPhone5's device pixels are 640, iPhone's html tag font-size value is 640 / 10 = 64px, so 1rem = 64px, so element showing 200px in iPhone6 will show as 2.6666667 * 64 pixels in iPhone5, thus achieving element proportional scaling across different devices without affecting layout.

Opening browser, viewing page on iPhone6 and iPhone5 respectively, we'll find elements now scale proportionally according to phone size.

Above is mobile Taobao's method, has one drawback: when converting to rem units, need divide by font-size value, Taobao uses iPhone6 design, so converting sizes requires dividing by 75, not easy to calculate, needs calculator, affecting development efficiency. Also, when converting to rem units, encountering indivisible numbers we use long approximations like above 2.6666667rem, possibly causing page element size deviations.

**Why does Taobao solution using rem still need viewport scaling?**

Only one benefit: we can draw 1 physical pixel things with 1px, like 1 physical pixel thin border.

### NetEase

Don't modify meta tag, normally use 1:1 scale meta tag.

We know:
Device pixels = Design size = 750px
Layout viewport = 375px

Assume using iPhone6 design size as standard, under design size set font-size value as 100px.
Meaning: 750px wide page, we set 100px font-size value, then page width in rem equals 750 / 100 = 7.5rem.

Using total page width 7.5rem as standard, then in layout viewport, i.e. total page width 375px, what should font-size value be? Simple:

font-size = 375 / 7.5 = 50px

What about iPhone5? Because iPhone5's layout viewport width is 320px, if total page width uses 7.5 standard, then iPhone5's font-size value should be:

font-size = 320 / 7.5 = 42.666666667px

Meaning, regardless of device, we can set page total width as fixed rem value, like 7.5rem in this example, just need dynamically set font-size value based on layout viewport size:

```js
document.documentElement.style.fontSize = document.documentElement.clientWidth / 7.5 + 'px';
```

This way, on any device, our page total width is 7.5rem, so we can directly measure px unit sizes on design, then divide by 100 to convert to rem units for direct use, example, measuring element size as 200px in iPhone6 design, converting to rem units is 200 / 100 = 2rem, because we dynamically set html tag font-size value on different devices, same rem values correspond to different pixel values on different devices, achieving proportional scaling across devices.

Also, whether Taobao or NetEase approach, text font size shouldn't convert to rem units, but use media queries for dynamic setting.

## vw, vh Solution

`vh, vw` solution divides visual viewport width `window.innerWidth` and visual viewport height `window.innerHeight` into 100 parts.

- `vw(Viewport's width)`: `1vw` equals `1%` of visual viewport
- `vh(Viewport's height)`: `1vh` equals `1%` of visual viewport height
- `vmin`: smaller value between `vw` and `vh`
- `vmax`: larger value between `vw` and `vh`

If visual viewport is `375px`, then `1vw = 3.75px`, if `UI` gives element width as `75px` (device independent pixels), we just need set it as `75 / 3.75 = 20vw`.

We don't need calculate these proportions ourselves, we can use `PostCSS`'s `postcss-px-to-viewport` plugin to complete this process. When coding, we just need write `px` units. (Used in my mall project)

## 1px Problem

On 750px design, UI designer's expected 1px physical pixel corresponds to 0.5px device independent pixels on actual 375px design.

0.5px device independent pixels are supported by IOS-8, not by Android; so Android renders 0.5px device independent pixels as 1px device independent pixels, meaning when Android's device independent pixels are 1px on 375px design, it occupies 2px physical pixels, thicker.

## Reference Articles

[How Should I Do Mobile Development with a Design Draft?](https://juejin.im/post/6844903937368129550)

[What You Must Know About Mobile Adaptation](https://juejin.im/post/6844903845617729549#heading-28)

[An Article That Really Teaches You Mobile Page Development (1)](http://hcysun.me/2015/10/16/%E4%B8%80%E7%AF%87%E7%9C%9F%E6%AD%A3%E6%95%99%E4%BC%9A%E4%BD%A0%E5%BC%80%E5%8F%91%E7%A7%BB%E5%8A%A8%E7%AB%AF%E9%A1%B5%E9%9D%A2%E7%9A%84%E6%96%87%E7%AB%A0(%E4%B8%80)/)

[An Article That Really Teaches You Mobile Page Development (2)](http://hcysun.me/2015/10/19/%E4%B8%80%E7%AF%87%E7%9C%9F%E6%AD%A3%E6%95%99%E4%BC%9A%E4%BD%A0%E5%BC%80%E5%8F%91%E7%A7%BB%E5%8A%A8%E7%AB%AF%E9%A1%B5%E9%9D%A2%E7%9A%84%E6%96%87%E7%AB%A0-%E4%BA%8C/)