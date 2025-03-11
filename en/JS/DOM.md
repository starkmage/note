## HTMLCollection and NodeList

### Similarities

1. Both are array-like objects with a `length` property

2. Both have a common method: `item`, which allows accessing elements in the result via `item(index)` or `item(id)`


### Differences

1. `HTMLCollection` only contains element nodes (elementNode), which are HTML tags; `NodeList` can contain any node type, with line breaks also treated as text nodes
2. `HTMLCollection` has one additional method compared to `NodeList`: `namedItem`, which allows retrieving node information by passing an id or name attribute
3. The getElementsBy series returns a Live Node List, while querySelectorAll returns a Static Node List.

### How to Obtain

`getElementsByClassName` and similar methods return an `HTMLCollection`, while `queryselectorAll` returns a `NodeList`

``` html
<body>
  <div class="demo">
    Content1
    <p>Content2</p>
    <span>Content3</span>
  </div>
  
  <script>
    let d1 = document.getElementsByClassName('demo')
    let d2 = document.querySelectorAll('.demo')
    console.log(d1);
    console.log(d2);
  </script>
</body>
```

![](http://img.stark.pub/20201029195736.png)

`elm.children` returns an `HTMLCollection`, while `elm.childrenNodes` returns a `NodeList`

``` js
let d = document.querySelector('.demo')

console.log(d.children);
console.log(d.childNodes);
```

![](http://img.stark.pub/20201029200144.png)

## Three Differences Between append and appendChild in DOM API

append and appendChild are two commonly used methods for adding elements to the Document Object Model (DOM). They can often be used interchangeably without much trouble, but if they were the same, why would there be two APIs?... They are similar, but not the same.

### .append()

This method is used to add elements in the form of Node objects or DOMString (basically text).

#### Inserting a Node Object

```js
const parent = document.createElement('div');
const child = document.createElement('p');
parent.append(child);
// This appends the child element to the div element
// Then the div looks like this: <div> <p> </ p> </ div>
```

This appends the child element to the `div` element, and then the `div` looks like this

```html
<div> <p> </ p> </ div>
```

#### Inserting a DOMString

```js
const parent = document.createElement('div');
parent.append('Appended text');
```

Then the `div` looks like this

```html
<div>Appended text</ div>
```

### .appendChild()

Similar to the `.append` method, this method is used for elements in the DOM, but in this case, it only accepts a Node object.

#### Inserting a Node Object

```js
const parent = document.createElement('div');
const child = document.createElement('p');
parent.appendChild(child);
```

This appends the child element to the `div` element, and then the `div` looks like this

```html
<div> <p> </ p> </ div>
```

#### Inserting a DOMString

```js
const parent = document.createElement('div');
parent.appendChild('Appending Text');
// Uncaught TypeError: Failed to execute 'appendChild' on 'Node': parameter 1 is not of type 'Node'
```

### Differences

`.append` accepts both Node objects and DOMString, while `.appendChild` only accepts Node objects.

```js
const parent = document.createElement('div');
const child = document.createElement('p');
// Appending node objects
parent.append(child) // Works fine
parent.appendChild(child) // Works fine
// Appending DOMStrings
parent.append('Hello world') // Works fine
parent.appendChild('Hello world') // Throws an error
```

`.append` has no return value, while `.appendChild` returns the appended Node object.

```js
const parent = document.createElement('div');
const child = document.createElement('p');
const appendValue = parent.append(child);
console.log(appendValue) // undefined
const appendChildValue = parent.appendChild(child);
console.log(appendChildValue) // <p></p>
```

`.append` allows you to add multiple items, while `.appendChild` only allows a single item.

```js
const parent = document.createElement('div');
const child = document.createElement('p');
const childTwo = document.createElement('p');
parent.append(child, childTwo, 'Hello world'); // Works fine
parent.appendChild(child, childTwo, 'Hello world');
// Works, but only adds the first element and ignores the rest
```

### Summary

In cases where `.appendChild` can be used, `.append` can also be used, but not vice versa.

## nodeValue, value, innerText, innerHTML, and textContent

1. innerHTML parses content as HTML, which takes longer
2. nodeValue uses direct text, doesn't parse HTML, and is faster; can be used with any Node
3. textContent uses direct text, doesn't parse HTML, and is faster (recommended); can be used with any Node
4. innerText takes styling into account; for example, it doesn't retrieve hidden text; can only be called on HTML elements
5. value is generally the value entered in an input

Difference between nodeValue and textContent:

``` html
<p id="demo">
  Content
</p>

<script>
  let p = document.getELementById('demo')
  console.log(p.nodeValue)	// null
  console.log(p.textContent) // Content
  let t = p.childNodes[0]	// t is the #text node
  console.log(t.nodeValue) // Content
  console.log(p.textContent) // Content
</script>
```

``` html
<div class="demo">
  Content1
  <p>Content2</p>
  <span>Content3</span>
</div>

<script>
  let d = document.querySelector('.demo')
  console.log(d.innerHTML);
  console.log(d.innerText);
  console.log(d.textContent);
</script>
```

![](http://img.stark.pub/20201104224139.png)

## Understanding the Implementation of JavaScript Drag Functionality, What is the Specific Implementation Method?

Approach:

1. When the mouse is pressed down, record the distance from the mouse position to the edge of the element being moved.

2. As the mouse moves, update the element's position in real-time
   * For specific calculation, taking the vertical direction as an example: the mouse's vertical distance from the browser, minus the previously recorded vertical distance from the mouse to the element, equals the value of the element's top positioning property
3. When the mouse is released, complete the element's displacement.

``` html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <title>Title</title>
  <style>
    div {
      width: 300px;
      height: 200px;
      background-color: pink;
      position: fixed;
      top: 0;
      left: 0;
    }
  </style>
</head>

<body>
  <div></div>
  <script>
    var divEle = document.getElementsByTagName('div')[0]
    var mouse = {
      mouseLock: false,
      mouseEleClient: {
        t: 0,
        l: 0
      }
    }
    divEle.onmousedown = function (e) { //Triggered when mouse is pressed down on divEle
      e = e || window.event
      var l = e.clientX
      var t = e.clientY
      mouse.mouseEleClient.t = t - this.offsetTop
      mouse.mouseEleClient.l = l - this.offsetLeft
      mouse.mouseLock = true
    }
    divEle.onmouseup = function (e) {//Triggered when mouse is released
      mouse.mouseLock = false
    }
    window.onmousemove = function (e) { //Continuously triggered when mouse moves
      if (mouse.mouseLock) {
        e = e || window.event
        var l = e.clientX
        var t = e.clientY
        var divT = t - mouse.mouseEleClient.t
        var divL = l - mouse.mouseEleClient.l
        divEle.style.top = divT + 'px'
        divEle.style.left = divL + 'px'
      }
    }
  </script>
</body>

</html>
```

## What Type of Task is a DOM-Triggered Callback?

https://www.zhihu.com/question/362096226