## Browser Rendering Process

![](http://img.stark.pub/20201009155742.jpg)

### Building DOM Tree

Parsing algorithm consists of two phases: **Tokenization and Tree Construction**, corresponding to **Lexical Analysis and Syntax Analysis**

### Building CSSOM Tree

* **Formatting Stylesheets**—Browser cannot directly recognize CSS style text, so first thing rendering engine does after receiving CSS text is convert it to structured object, i.e., styleSheets.
* **Standardizing Style Properties**—like `em`->`px`, `red`->`#ff0000`
* **Computing Specific Styles for Each Node**—mainly two rules: **Inheritance** and **Cascade**

After style computation, all style values are attached to `window.computedStyle`, allowing convenient style access through JS.

### Generating Layout Tree

Now with `DOM Tree` and `DOM Styles`, next step is using browser's layout system to `determine element positions`, generating a `Layout Tree`.

Layout tree generation roughly works as follows:

1. Traverse generated DOM tree nodes, adding them to `Layout Tree`;
2. Calculate coordinate positions for layout tree nodes.

Notably, this layout tree only includes visible elements; `head` tags and elements with `display: none` won't be included.

Some articles mention generating `Render Tree` first, but that's pre-2016. Chrome team has done major restructuring, eliminating `Render Tree` generation process. Layout tree information is now comprehensive, fully providing `Render Tree` functionality.

### Building Layer Tree

Normally, node layers default to parent node's layer (these layers also called **compositing layers**). When does a layer become separate compositing layer?

Two situations need separate discussion: **explicit compositing** and **implicit compositing**.

#### Explicit Compositing

* Nodes with **stacking context**

  * HTML root element inherently has stacking context
  * Regular elements set to **position relative or absolute** with **z-index set** create stacking context
  * **position** set to **fixed** or **sticky**
  * Element's **opacity** value not 1
  * Element's **transform** value not none
  * Element's **filter** value not none (filters typically used for adjusting image, background, border rendering)
  * Element's **isolation** value is isolate (defines whether element must create new stacking context)
  * **will-change** specified property value is any above

* Places needing **clipping**.

  Like a div set to 100 * 100 pixels with lots of text inside, overflowing text needs clipping. If scrollbar appears, scrollbar gets promoted to separate layer.

#### Implicit Compositing

Simply put, when node with `lower stacking level` becomes separate layer, `all nodes with higher stacking levels` **will** become separate layers.

This implicit compositing actually hides huge risk. In large application, when element with low `z-index` becomes separate layer, elements stacked above it all become separate layers, potentially adding thousands of layers, greatly increasing memory pressure, even crashing page. This is **layer explosion** principle.

Implicit compositing fundamentally ensures correct layer overlap order, but in practice, easily leads to meaningless compositing layers

https://juejin.cn/post/6844903966573068301#heading-9

### Generating Paint List

Rendering engine breaks down layer painting into individual paint instructions, like paint background first, then draw border...... then combines these instructions sequentially into paint list, essentially planning for later **painting operations**.

### Generating Tiles

Now starting painting operation, actually done by dedicated thread in rendering process called **compositing thread**. When paint list ready, `main thread of rendering process` sends `commit` message to `compositing thread`, submitting paint list.

First thing compositing thread does is **tile** layers. Usually 256 * 256 or 512 * 512 specifications.

### Generating Bitmaps

Rendering process maintains dedicated **rasterization thread pool** for converting **tiles** to **bitmap data**. Compositing thread selects **tiles near viewport**, sends them to **rasterization thread pool** for bitmap generation.

Bitmap generation process actually uses GPU acceleration, generated bitmaps sent to `compositing thread`.

### Display Content

After rasterization, **compositing thread** generates draw command, "DrawQuad", sends to browser process. `viz component` in browser process receives command, draws page content to memory based on command, generating page, **then sends this memory to graphics card**.

Whether PC monitor or phone screen, all have fixed refresh rate, generally 60 HZ, meaning 60 frames, updating 60 images per second, each image staying about 16.7 ms. Each updated image comes from graphics card's **front buffer**. Graphics card receives page from browser process, composes corresponding image, saves to **back buffer**, then system automatically swaps `front buffer` and `back buffer` positions, cycling updates.

## Repaint

### Trigger Conditions

When DOM modifications cause style changes, like color changes, without affecting geometric properties, triggers `repaint`.

### Repaint Process

Since no DOM geometric property changes, element position information doesn't need updating, skipping layout process.

![](http://img.stark.pub/20201009155853.png)

Skips `generating layout tree` and `building layer tree` stages, directly generates paint list, continues with tiling, bitmap generation, etc.

Repaint doesn't necessarily cause reflow, but reflow always causes repaint.

## Reflow

### Trigger Conditions

1. DOM element's geometric properties or position changes, common geometric properties include `width`, `height`, `padding`, `margin`, `left`, `top`, `border` etc., easily understood;
2. DOM nodes experience `addition/reduction` or `movement`;
3. When reading/writing `offset` family, `scroll` family and `client` family properties, browser needs reflow to get these values;
4. Calling `window.getComputedStyle` method.

### Reflow Process

Following rendering pipeline, when reflow triggers, if DOM structure changes, re-renders DOM tree, then runs through all subsequent processes (including tasks outside main thread).

![](http://img.stark.pub/20201009160657.png)

Basically reruns parsing and compositing processes, very costly.

## Reducing Reflow and Repaint

### Minimizing Reflow and Repaint

1. **If need layout property values, assign to variable**, avoiding recalculating layout properties each time, e.g., var t = elem.scrollTop, using t multiple times only causes one reflow

2. **Avoid frequent style use, adopt class modification approach**

3. **Batch modify styles, or use CSS property shorthand**, same thinking as 2, **goal is merging multiple DOM and style modifications, handling once**

   Example:

   ```js
   const el = document.getElementById('test');
   el.style.padding = '5px';
   el.style.borderLeft = '1px';
   el.style.borderRight = '2px';
   ```

   Here, three style properties modified, each affecting element's geometric structure, causing reflow. **Most modern browsers optimize this, triggering only one reflow. But in older browsers or if other code accesses layout information** (layout information triggering reflow mentioned above) during above code execution, causes three reflows.

   Therefore, we can merge all changes and process sequentially, like:

   - Using cssText

     ```js
     const el = document.getElementById('test');
     el.style.cssText += 'border-left: 1px; border-right: 2px; padding: 5px;';
     ```

   - Modifying CSS class

     ```js
     const el = document.getElementById('test');
     el.className = 'active';
     ```

### Batch DOM Modification

Consider batch DOM node insertion scenario

1. **For complex DOM operations, can hide first (display: none), show after completion**

   ```js
   function appendDataToElement(parent, data) {
       let li;
       for (let i = 0; i < data.length; i++) {
           li = document.createElement('li');
           li.textContent = 'text';
           parent.appendChild(li);
       }
   }
   const ul = document.getElementById('list');
   ul.style.display = 'none';
   appendDataToElement(ul, data);
   ul.style.display = 'block';
   ```

2. **Use `createDocumentFragment` for batch DOM operations**

   ```js
   const ul = document.getElementById('list');
   const fragment = document.createDocumentFragment();
   appendDataToElement(fragment, data);
   ul.appendChild(fragment);
   ```

### GPU Acceleration

Rather than considering how to reduce reflow/repaint, we prefer no reflow/repaint!

Adding will-change: transform makes rendering engine create separate layer, when these transformations occur, only uses compositing thread to handle transformations, not involving main thread, greatly improving rendering efficiency. Of course this change isn't limited to `transform`, any CSS property achieving composition effect can use `will-change` declaration.

Above statement isn't strictly accurate

Compositing layer when needing repaint indeed only repaints itself, not affecting other layers, but before paint there's style, layout regeneration, meaning even if compositing layer only repaints itself, style and layout time consumption remains.

### Others

1. Avoid table layout, because table element triggering reflow causes all other elements in table to reflow
2. Apply debounce/throttle to resize, scroll etc.

## Compositing

Using CSS3's `transform`, `opacity`, `filter` properties can achieve compositing effect, commonly called **GPU acceleration**.

GPU acceleration reasons:

In compositing, skips layout and painting processes, directly enters `non-main thread` processing part, i.e., directly handled by `compositing thread`. Two major benefits:

1. Can fully utilize `GPU` advantages. During bitmap generation in compositing thread, calls thread pool, uses `GPU` acceleration, and GPU excels at handling bitmap data.
2. Doesn't occupy main thread resources, even if main thread blocks, effect still displays smoothly.

## Compositing Layer Advantages

Once renderLayer promotes to compositing layer gets own drawing context, enables hardware acceleration, beneficial for performance.

- Compositing layer bitmap handled by GPU, faster than CPU (after promotion to compositing layer, bitmap handled by GPU, but note, **only compositing processing (combining drawing context bitmap outputs) needs GPU, generating compositing layer bitmap (drawing context work) needs CPU**)
- When needing repaint, only repaints itself, not affecting other layers (when needing repaint can only repaint itself, not affecting other layers, but before paint there's style, layout, meaning even if compositing layer only repaints itself, style and layout time consumption remains.)

Note: can't abuse GPU acceleration, must analyze actual performance. **Creating rendering layer with GPU acceleration has cost, each new rendering layer means new memory allocation and more complex layer management**. Also mobile GPU and CPU bandwidth limited, too many rendering layers means compositing consumes more time, leading to more power consumption, more memory usage. **Overhead from excessive rendering layers might far exceed performance benefits**
https://juejin.cn/post/6844904040346681358

## How Many Reflows/Repaints Triggered

Suppose modifying div style with JS

```js
div.style.left = '10px';
div.style.top = '10px';
div.style.width = '10px';
div.style.height = '10px';
```

We modified element's left, top, width, height properties, meeting reflow conditions, **theoretically causes 4 reflows, but actually only 1** because modern browsers have rendering queue mechanism. When changing element style causing browser reflow/repaint, it enters **rendering queue**, browser continues checking, if more style modifications, they queue too, until no more modifications, browser batch executes queued changes optimizing reflow process, modifying styles together, optimizing 4 reflows to 1 (antique browsers do 4).

How about this?

```js
div.style.left = '10px';
console.log(div.offsetLeft);

div.style.top = '10px';
console.log(div.offsetTop);

div.style.width = '20px';
console.log(div.offsetWidth);

div.style.height = '20px';
console.log(div.offsetHeight);
```

4 times!

```text
offsetTop、offsetLeft、offsetWidth、offsetHeight
clientTop、clientLeft、clientWidth、clientHeight
scrollTop、scrollLeft、scrollWidth、scrollHeight
getComputedStyle()（currentStyle in IE）
```

These force queue flush requiring immediate style modification execution, **because browser uncertain if following code modifies same styles, must immediately execute rendering queue triggering reflow for correct immediate values**!!!

## Browser Rendering Update Timing

- Multiple modifications to same DOM in one event loop round, only last one painted.
- Rendering update occurs after event loop's tasks and microtasks complete, but not every event loop updates rendering, depends on whether DOM modified and browser thinks immediate new state presentation necessary. If multiple DOM modifications in one frame (time uncertain, browser frame rate fluctuates, 16.7ms just estimate), browser might accumulate changes, do one paint, which is reasonable.
- If want immediate changes every event loop round, use requestAnimationFrame, as its execution frequency determined by display.

## Reference Articles

[Sanyuan's Blog](http://47.98.159.95/my_blog/browser-render/002.html)

[Do You Really Understand Reflow and Repaint](https://segmentfault.com/a/1190000017329980)

[ByteDance Frontend Interview Question: How Many Reflows and Repaints Triggered](https://zhuanlan.zhihu.com/p/161468550)

[Exploring JavaScript Async and Browser Rendering Update Timing from Event Loop Specification](https://github.com/aooy/blog/issues/5)