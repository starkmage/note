## CSS Selectors

CSS has 7 basic types of selectors:

- ID selector, such as #id{}
- Class selector, such as .class{}
- Attribute selector, such as a[href="segmentfault.com"]{}
- Pseudo-class selector, such as :hover{}
- Pseudo-element selector, such as ::before{}
- Tag selector, such as span{}
- Universal selector, such as *{}

**CSS3 Priority Rules:** Priority relationship: Inline styles > ID selectors > Class selectors = Attribute selectors = Pseudo-class selectors > Tag selectors = Pseudo-element selectors

**How is CSS priority calculated?**

Element selector: 1
Class selector: 10
ID selector: 100
Element tag: 1000

1. Styles declared with !important have the highest priority; if there's a conflict, further calculation is needed.
2. If priorities are the same, the last appearing style is selected.
3. **Inherited styles have the lowest priority.**

## Block Elements, Inline Elements, and Inline-Block Elements

### Block Elements

div, p, h, ul, ol, li, etc.

1. Each block element starts on a new line (occupies the entire line)
2. Height, width, line-height, and top/bottom margins can be set
3. Width is 100% of its parent container by default (same as parent element's width) unless specified otherwise

### Inline Elements

a, span, code, label, i, etc.

1. Appears on the same line as other elements
2. Height, width, and top/bottom margins **cannot** be set
3. Width is determined by the content (text or images) and cannot be changed

**absolute/fixed and float can only be applied to block elements; when applied to inline elements, the inline elements will automatically be converted to block**

### Inline-Block Elements

img, input, etc.

1. Appears on the same line as other elements
2. Height, width, line-height, and top/bottom margins can be set

## BFC (Block Formatting Context)

A Block Formatting Context is a container that manages block-level elements

### Triggering Conditions

1. Root element
2. position: absolute or fixed
3. Float elements (float not none)
4. overflow not visible (default value)
5. inline-block elements
6. Table elements
7. Flex elements (children of elements with display: flex)
8. Grid elements (direct children of elements with display: grid or inline-grid)

### Rendering Rules

1. Floating elements are included in BFC height calculations
2. BFC is an independent container; outside elements don't affect inside elements
3. BFC vertical margin collapse (vertical margins only collapse when elements belong to the same BFC)
4. BFC areas don't overlap with float element boxes

### Application Scenarios

1. Prevent float-caused parent element height collapse, clear floats
2. Avoid margin collapse

## Ways to Remove Elements from Document Flow

- float
- position: absolute
- position: fixed

## Default Value of z-index

Default value is auto, 0 is above auto, z-index:0 creates a new stacking context

## Comparison between Holy Grail Layout and Double Wing Layout

- Both layouts place the main column first in the document flow for priority loading.
- Both use float for all three columns and negative margins to create the three-column layout.
- The difference lies in how they handle the main column's position:
  **Holy Grail Layout uses left and right padding on the parent container plus relative positioning of the two side columns**;
  **Double Wing Layout nests the main column in a new parent block and uses left and right margins on the main column for layout adjustment**

## Negative Margins

- Non-floating

  - ```position: static```
    - margin-top/margin-left: Element moves up/left.
    - margin-bottom/margin-right: Element doesn't move, following elements move toward it and overlap it.

  - ```position: relative```
    - margin-top/margin-left: Element moves up/left.
    - margin-bottom/margin-right: Element doesn't move, following elements move toward it but are overlapped by it.

  - ```position: absolute```
    - margin-top/margin-left: Element moves up/left.
    - margin-bottom/margin-right: Element doesn't move, no effect on following elements (due to being removed from document flow).

- Floating

  - If margin direction matches float direction: Element moves in that direction.
  - If margin direction opposes float direction: Element stays put, preceding or following elements move toward it.

## Position Positioning Reference

### Absolute Elements

Absolute element containing block rules:

1. Nearest ancestor element with position value of fixed, absolute, relative, or sticky (i.e., position not static).
2. Positioned relative to the padding area (padding box) of the above ancestor element.

### Fixed Elements

Fixed element containing block rules:

- Positioned relative to the viewport.

### Common Rules

Absolute or fixed elements also share common containing block rules. **The following conditions make an ancestor element their containing block (relative to padding area):**

1. transform property value not none.
2. perspective property value not none.
3. will-change property value is transform or perspective; also works with filter value but only in Firefox.
4. filter property value not none.
5. contain property value is paint (contain: paint).

## position:sticky

### Usage

- position:sticky elements are called stickily positioned elements
- Simple explanation: Within the target area, it behaves like position:relative; when scrolling causes an element to reach the sticky positioning requirement relative to its parent (e.g., top: 100px), position:sticky behaves like fixed positioning
- Can be considered a combination of relative and fixed positioning
- Element's fixed offset is relative to its nearest ancestor with a scrolling box; if no ancestor can scroll, offset is calculated relative to viewport

### position:sticky Usage Conditions

1. Parent element cannot have overflow:hidden or overflow:auto
2. Must specify one of top, bottom, left, right; otherwise, only relative positioning applies
3. Parent element's height must not be less than sticky element's height
4. Sticky element only works within its parent element
5. Does not trigger BFC

### Issues

Poor browser compatibility

## How Do Browsers Parse CSS Selectors?

CSS **selectors** are parsed from right to left.

Parsing from left to right would require backtracking when rules don't match, causing performance loss.

Parsing from right to left first finds all rightmost nodes, then for each node, traverses up to find parent nodes until reaching the root element or matching rule, ending that branch's traversal.

The performance difference between these two matching rules is significant because right-to-left matching filters out many non-matching rightmost nodes (leaf nodes) in the first step, while left-to-right matching wastes performance on failed searches.

## How to Make Chrome Support Text Smaller Than 12px?

```css
p {
  font-size:10px;
  -webkit-transform:scale(0.8);
} //0.8 is the scale ratio
```
## Advantages and Disadvantages of Using Base64 Encoded Images

Base64 encoding is an image processing format that encodes images into a long string through specific algorithms. When displaying on a page, this string can replace the image's url attribute.

Advantages of using base64:

1. Reduces one HTTP request per image

Disadvantages of using base64:

1. According to base64 encoding principles, the encoded size will be 1/3 larger than the original file size. If large images are encoded into html/css, it will not only increase file size and affect loading speed, but also increase browser parsing and rendering time for html or css files.

2. Base64 cannot be directly cached. To cache, you can only cache files containing base64, such as HTML or CSS, which is much less effective than caching images directly.

3. Compatibility issues - browsers before IE8 don't support it.

Generally, small icons on websites can be introduced using base64 images.

## Difference Between ::before and :after

In CSS3, single colons are used to represent pseudo-classes, while double colons represent pseudo-elements. However, for compatibility with existing pseudo-element syntax, some browsers also allow single colons to represent pseudo-elements.

**Pseudo-classes generally match special states of elements, such as hover, link, etc., while pseudo-elements generally match special positions, such as after, before, etc.**

## Differences Between Pseudo-classes and Pseudo-elements

CSS introduced pseudo-classes and pseudo-elements to format information outside the document tree. In other words, pseudo-classes and pseudo-elements are used to style parts not in the document tree, such as the first letter of a sentence or the first element in a list.

Pseudo-classes are used to add corresponding styles when existing elements are in certain states, which change dynamically based on user behavior. For example, when users hover over a specified element, we can describe this element's state using :hover.

Pseudo-elements are used to create elements not in the document tree and add styles to them. They allow us to style certain parts of elements. For example, we can use ::before to add some text before an element and style this text. Although users can see this text, it's not actually in the document tree.

Sometimes you'll find pseudo-elements using two colons (::) instead of one (:). This is part of CSS3 and attempts to distinguish between pseudo-classes and pseudo-elements. Most browsers support both values. According to rules, (::) should be used instead of (:) to distinguish between pseudo-classes and pseudo-elements. However, since older versions of W3C specifications didn't make this special distinction, currently most browsers support both ways of representing pseudo-elements.

## Why Can img Set Width and Height Despite Being Inline

img is a replaceable element.

In CSS, the presentation effect of replaceable elements (replaced elements) is not controlled by CSS. These elements are external objects whose appearance rendering is independent of CSS.
Simply put, their content is not affected by the current document's styles. CSS can affect the position of replaceable elements but won't affect the content of the replaceable elements themselves.

## Browser Compatibility Issues Encountered

1. Different default margins and padding in browsers
   Solution: Add a global *{margin:0;padding:0;} to unify

2. Chrome's Chinese interface forces text smaller than 12px to display as 12px

   Solution: Mentioned earlier

3. After visiting a hyperlink, hover style doesn't appear, and visited hyperlinks no longer have hover and active styles

   Solution: Change the order of CSS properties to L-V-H-A

4. Issues in IE mode

## Explanation of CSS Sprites

CSS Sprites combines all images involved in a page into one large image, then uses combinations of CSS background-image, background-repeat, background-origin, and background-position for background positioning.
CSS Sprites can effectively reduce webpage HTTP requests, thus greatly improving page performance; CSS Sprites can reduce image bytes.

Advantages:

1. Reduces HTTP requests, greatly improving page loading speed
2. Increases image information redundancy, improves compression ratio, reduces image size
3. Easy to change styles, only need to modify colors or styles on one or a few images

Disadvantages:

1. Image merging is troublesome
2. Maintenance is troublesome, modifying one image might require re-layouting the entire image and styles

## How to Implement Single-line/Multi-line Text Overflow Ellipsis (...)

```css
/*Single-line text overflow*/
p {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}


/*Multi-line text overflow*/
p {
  position: relative;
  line-height: 1.5em;
  /*height is line-height times number of lines to display, here we show 2 lines, so it's 3*/
  height: 3em;
  overflow: hidden;
}

p:after {
  content: "...";
  position: absolute;
  bottom: 0;
  right: 0;
  background-color: #fff;
}
```

When first using Vue, scoped technology was advocated and widely used

```vue
<style scoped>
    .list-container:hover {
        background: orange;
    }
</style>
```

This optional scoped attribute will automatically add a unique attribute (like data-v-21e5b78) to scope CSS within components. During compilation, `.list-container:hover` will be compiled to something like `.list-container[data-v-21e5b78]:hover`

```html
<span data-v-0467f817 class="errShow">Username cannot be empty</span>
```

CSS style is compiled as follows

```css
.errShow[data-v-0467f817] {
    font-size: 12px;
    color: red;
}
```

However, it can't completely avoid conflicts. If a user also defines an errShow class name, it will affect the display of all components defined with the errShow class name

CSS modules does it more thoroughly, instead of adding attributes, it directly changes class names

```html
<span class="_3ylglHI_7ASkYw5BlOlYIv_0">Username cannot be empty</span>
```

Below is how to write CSS modules

Add the module attribute in the style tag to indicate opening CSS-loader's module mode

```vue
<style module>
.red {color: red;}
</style>
```

Use dynamic class binding :class in the template and add '$style.' before the class name

```vue
<template>
  <p :class="$style.red">
    This should be red
  </p >
</template>
```

If the class name contains hyphens, use bracket syntax

```html
<h4 :class="$style['header-tit']">Category Recommendations</h4>
```
## text-align and vertical-align

### text-align

text-align has the following options (similar to alignment options in word processors with left align, right align, center align)

* left: left alignment
* right: right alignment
* center: center alignment
* start: equals left if direction is left-to-right (ltr)
* end: same as above
* justify: justify alignment, ineffective for last line
* justify-all: forces justify alignment for last line
* match-parent: similar to inherit, but start and end values are determined by parent element's direction and replaced with appropriate left or right.

**Must be used under block-level element containers, doesn't control block element alignment itself, only controls alignment of internal inline elements, inline-block elements, and text. If used in tags like span, horizontal centering won't work.**

```html
div {
  height: 100px;
  width: 100px;
  background-color: yellowgreen;
  text-align: center;
}

span {
	background-color: pink;
}

<div>
  <span>hello</span>
</div>
```

![](http://img.stark.pub/20201104135400.png)

Doesn't affect internal block-level elements. Internal div inherits outer div's text-align, so its internal text is horizontally centered, but overall it's not horizontally centered relative to parent element

```html
#d {
  height: 100px;
  width: 100px;
  background-color: yellowgreen;
  text-align: center;
}

#dd {
  width: 80px;
  background-color: pink;
}

<div id="d">
  <div id="dd">hello</div>
</div>
```

![](http://img.stark.pub/20201104140158.png)

### line-height

"Line height" refers to the height of a line of text. Specifically, it's **the distance between the baselines of two lines of text**. Baseline is a concept used in English letters, like the second line from bottom in those four-lined English notebooks we used when first learning English.

Percentages or numbers without units are relative to inline content height

Analyzing a situation:

With an empty `div`, `<div></div>`, if no height value of at least 1 pixel is set, the div's height is 0. If a space or text is entered in this div, it will have a height. Why does the div have height after text is added?

This seems like a simple question but is crucial for understanding `line-height`. Some might think: text creates the space! Text occupies space, naturally expanding the div. But actually, after deeply understanding the `inline` model, you'll find it's not text that expanded the div's height, but `line-height`!

```html
#help {
  height: 100px;
  background-color: yellow;
}

#d1 {
  font-size: 30px;
  line-height: 0;
  background-color: green;
}

#d2 {
  font-size: 0;
  line-height: 30px;
  background-color: pink;
}

<div id="help"></div>
<div id="d1">test1</div>
<div id="d2">test2</div>
```

![](http://img.stark.pub/20201104144217.png)

Look at the effect: div1 has no height but text is displayed, div2 shows no text but still has height

Online sources often say setting line-height equal to height value achieves vertical centering for single-line text. This is correct but also problematic. The problem is with height. My expression would be: "Setting line-height to your desired box size achieves vertical centering for single-line text." The difference is I removed height - it's redundant, try it yourself. —Zhang Xinxu: [Deep Understanding and Application of CSS line-height](https://www.zhangxinxu.com/wordpress/2009/11/css%E8%A1%8C%E9%AB%98line-height%E7%9A%84%E4%B8%80%E4%BA%9B%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3%E5%8F%8A%E5%BA%94%E7%94%A8/)

### vertical-align

vertical-align percentage values are relative to the inherited line-height value

vertical-align property only works when an element is inline or inline-block.

This topic has extensive content, refer to Zhang Xinxu's article for details: [My Understanding of CSS vertical-align (Part 1)](https://www.zhangxinxu.com/wordpress/2010/05/%E6%88%91%E5%AF%B9css-vertical-align%E7%9A%84%E4%B8%80%E4%BA%9B%E7%90%86%E8%A7%A3%E4%B8%8E%E8%AE%A4%E8%AF%86%EF%BC%88%E4%B8%80%EF%BC%89/?shrink=1)

An example of achieving horizontal and vertical centering

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <style>
    html,
    body {
      margin: 0;
      padding: 0;
      width: 100%;
    }

    .text-container {
      height: 150px;
      text-align: center;
      background-color: pink;
      border: 3px solid greenyellow;
    }

    .text-container:after {
      content: "";
      display: inline-block;
      width: 0;
      height: 100%;
      vertical-align: middle;
    }

    span {
      vertical-align: middle;
      display: inline-block;
      max-width: 90%;
      max-height: 100px;
      overflow: hidden;
    }
  </style>
</head>

<body>
  <div class="text-container">
    <span>I am single-line text</span>
  </div>
  <div class="text-container">
    <span>I am multi-line text I am multi-line text I am multi-line text I am multi-line text I am multi-line text</span>
  </div>
</body>

</html>
```

vertical-align achieves approximate vertical centering

When setting element alignment to middle, it means: **The element's vertical center line aligns with the line halfway up from the parent element's baseline to x-height**. In other words, the yellow line aligns with the center of the red letter x. However, text has a sinking characteristic - originally x's center point should align with the white line, but text's sinking characteristic makes letters drop slightly. This prevents perfect alignment between yellow and white lines. This sinking amount relates to text size and font. When text is small enough, we can ignore this, and approximately, the white line aligns with the yellow line, achieving centering. But with large text, it doesn't work well.

<img src="https://user-gold-cdn.xitu.io/2018/2/9/16179fb93a9ff285?imageView2/0/w/1280/h/960/format/webp/ignore-error/1" style="zoom:50%;" />

## Common Layout Methods

* Flex layout
* Flow layout (percentage layout)
* Float layout (float)
* Position layout (position)
* rem layout
* vw, vh layout
* Responsive layout

## Analysis and Comparison of opacity: 0, visibility: hidden, display: none - Advantages, Disadvantages, and Use Cases

- `display: none;`

1. **DOM Structure**: Browser won't render elements with display property set to none, doesn't occupy space;
2. **Event Listening**: Cannot listen to DOM events;
3. **Performance**: Dynamically changing this property triggers reflow, poor performance;
4. **Inheritance**: Not inherited by child elements, as they won't be rendered either;
5. **Transition**: `transition` doesn't support `display`.

- `visibility: hidden;`

1. **DOM Structure**: Element is hidden but rendered, doesn't disappear, occupies space;
2. **Event Listening**: Cannot listen to DOM events;
3. **Performance**: Dynamically changing this property triggers repaint, better performance;
4. **Inheritance**: Inherited by child elements, children can cancel hiding by setting `visibility: visible;`;
5. **Transition**: `transition` doesn't support `visibility`.

- `opacity: 0;`

1. **DOM Structure**: 100% transparency, element hidden but occupies space;
2. **Event Listening**: Can listen to DOM events;
3. **Performance**: Promotes to composite layer, doesn't trigger repaint, better performance;
4. **Inheritance**: Inherited by child elements, and children **cannot** cancel hiding with `opacity: 1`;
5. **Transition**: `transition` supports `opacity`.

## What's the Real Difference Between line-height:150% and line-height:1.5?

150% calculates line height based on parent element's font size, and child elements continue using this calculated line height. While 1.5 calculates line height by multiplying child element's own font size by 1.5. Also, 1.5em follows the 150% calculation method.

With units, child elements inherit the calculated line spacing from parent elements; without units, they inherit the coefficient, and child elements calculate their own line spacing separately (recommended)

## Removing Spaces Between inline-block Elements

[N Ways to Remove Spaces Between inline-block Elements](https://www.zhangxinxu.com/wordpress/2012/04/inline-block-space-remove-%E5%8E%BB%E9%99%A4%E9%97%B4%E8%B7%9D/)