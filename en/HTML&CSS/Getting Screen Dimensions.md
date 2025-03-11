#### document.documentElement.clientWidth and document.documentElement.clientHeight

Obtains the width and height of the screen's visible area, excluding scrollbars and toolbars. (In mobile, this is the layout viewport)

```js
document.documentElement.clientWidth = width + padding
document.documentElement.clientHeight = height + padding
```

#### window.innerWidth and window.innerHeight

Obtains the width and height of the visible area, but window.innerWidth includes the width of the vertical scrollbar, and window.innerHeight includes the height of the horizontal scrollbar. (In mobile, this is the visual viewport)

```js
window.innerWidth = width + padding + border + vertical scrollbar width
window.innerHeight = height + padding + border + horizontal scrollbar height
```

#### window.outerWidth and window.outerHeight

Obtains the width and height of the window including toolbars and scrollbars.

```js
window.outerWidth = width + padding + border + vertical scrollbar width
window.outerHeight = height + padding + border + horizontal scrollbar height + toolbar height
```

#### document.body.clientWidth and document.body.clientHeight

document.body.clientWidth also obtains the width of the visible area, but document.body.clientHeight obtains the height of the body content. If the content is only 200px, then this height is also 200px.

#### offsetWidth & offsetHeight

Returns the element's own width and height + padding + border + scrollbar.

#### offsetLeft & offsetTop

All HTML elements have offsetLeft and offsetTop properties that return the X and Y coordinates of the element

> 1. For descendant elements of positioned elements and some other elements (table cells), these properties return coordinates relative to the ancestor element
>
> 2. For general elements, they are relative to the document, returning document coordinates
>
> The offsetParent property specifies the parent element these properties are relative to; if offsetParent is null, then these properties are all document coordinates

#### scrollWidth & scrollHeight

These two properties are the element's content area plus padding, plus the size of any overflow content, meaning they include the height/width of the scroll area.

Therefore, when there is no overflow, these properties are equal to clientWidth and clientHeight.

#### scrollLeft & scrollTop

Specifies the position of the element's scrollbars

scrollLeft and scrollTop are writable properties; you can set them to make the content within elements scroll.

![](http://img.stark.pub/20210320162035.png)

#### Reference Articles

[Differences between Getting Screen Width: width(), outerWidth, innerWidth, clientWidth](https://segmentfault.com/a/1190000010746091)