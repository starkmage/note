Although the V8 engine has an automatic garbage collection mechanism, there is still a risk of memory leaks.

Memory lifecycle: Memory allocation -> Memory usage -> Memory release.

**Memory leak** refers to the failure of a program to release memory that is no longer in use due to negligence or errors. Memory leaks do not refer to the physical disappearance of memory, but rather to the application allocating a segment of memory and then, due to design errors, losing control of that segment of memory before releasing it, resulting in memory waste.

## Two Types of JS Memory Collection Algorithms

### Reference Counting Garbage Collection

> This is the most basic garbage collection algorithm. This algorithm simplifies the definition of "whether an object is no longer needed" to "whether there are other objects referencing it". If no references point to the object (zero references), the object will be reclaimed by the garbage collection mechanism.

Look at the example below, has the memory for "this object" been reclaimed?

```js
// "This object" is assigned to the obj variable
var obj = {
  a: 1,
  b: 2,
}
// b references "this object"
var b = obj; 
// Now, the original reference obj to "this object" has been replaced by b
obj = 1;
```

In the current execution environment, the memory for "this object" has not been reclaimed yet. You need to manually release the memory of "this object" (of course, this is assuming we haven't left the execution environment), for example:

```js
b = null;
// or b = 1, basically replace "this object" with something else
```

This way, the memory of the referenced "this object" is reclaimed.

ES6 distinguishes **references** as **strong references** and **weak references**, which currently only exist in Set and Map.

**Strong references** will have **reference counting** incremented, and only objects with a reference count of 0 will have their memory reclaimed, so generally manual memory reclamation is needed (provided that the **mark-and-sweep algorithm** hasn't executed yet and we're still in the current execution environment).

Whereas **weak references** don't trigger **reference counting** increments, and as soon as the reference count is 0, weak references automatically disappear without the need for manual memory reclamation.

### Mark-and-Sweep Algorithm

> When variables enter the execution environment, they are marked as "entering the environment", and when variables leave the execution environment, they are marked as "leaving the environment". Variables marked as "entering the environment" cannot be reclaimed because they are being used, while variables marked as "leaving the environment" can be reclaimed.

The environment can be understood as our scope, but global scope variables are only destroyed when the page is closed.

```js
// Assume these are global variables
// b is marked as entering the environment
var b = 2;
function test() {
  var a = 1;
  // When the function executes, a is marked as entering the environment
  return a + b;
}
// When the function execution ends, a is marked as leaving the environment and is reclaimed
// But b is not marked as leaving the environment
test();
```

## Some Scenarios of JavaScript Memory Leaks

### Accidental Global Variables

```js
// Defined in the global scope
function count(number) {
  // basicCount is equivalent to window.count = 2;
  count = 2;
  return count + number;
}
```

### Forgotten Timers

Forgetting to clear useless timers is one of the most common mistakes made by beginners.

Let's take a Vue component as an example.

```vue
<template>
  <div></div>
</template>

<script>
export default {
  methods: {
    refresh() {
      // Get some data
    },
  },
  mounted() {
    setInterval(function() {
      // Poll for data
      this.refresh()
    }, 2000)
  },
}
</script>
```

When the above component is destroyed, the `setInterval` is still running, and all the memory involved cannot be reclaimed (the browser will consider this as necessary memory, not garbage memory). You need to clear the timer when the component is destroyed, as follows:

```vue
<template>
  <div></div>
</template>

<script>
export default {
  methods: {
    refresh() {
      // Get some data
    },
  },
  mounted() {
    this.refreshInterval = setInterval(function() {
      // Poll for data
      this.refresh()
    }, 2000)
  },
  beforeDestroy() {
    clearInterval(this.refreshInterval)
  },
}
</script>
```

### Forgotten Event Listeners

Forgetting to clear useless event listeners is one of the most common mistakes made by beginners.

Let's continue using a Vue component as an example.

```js
<template>
  <div></div>
</template>

<script>
export default {
  mounted() {
    window.addEventListener('resize', () => {
      // Do some operations here
    })
  },
  // Add the following to reclaim memory
  beforeDestroy() {
    window.removeEventListener('resize', this.resizeEventCallback)
  },
}
</script>
```

When the above component is destroyed, the resize event is still being listened to, and all the memory involved cannot be reclaimed (the browser will consider this as necessary memory, not garbage memory). You need to remove the relevant events when the component is destroyed.

### Forgotten Publish-Subscribe Event Listeners

This follows the same principle as the **Forgotten Event Listeners** above.

Assume the publish-subscribe event has three methods: `$emit`, `$on`, and `$off`.

```vue
<template>
  <div @click="onClick"></div>
</template>

<script>
export default {
  methods: {
    onClick() {
      this.$bus.$emit('test', { type: 'click' })
    },
  },
  mounted() {
    this.$bus.$on('test', data => {
      // Some logic
      console.log(data)
    })
  },
  // Reclaim
  beforeDestroy() {
    this.$bus.$off('test')
  },
}
</script>
```

### Forgotten Set and Map Elements

For Set:

The following has a memory leak (when **members are reference types**, i.e., objects):

```js
let map = new Set();
let value = { test: 22 };
map.add(value);

value= null;
```

It needs to be changed to this to avoid memory leaks:

```js
let map = new Set();
let value = { test: 22};
map.add(value);

map.delete(value);
value = null;
```

There's a more convenient way, using WeakSet, where the members are **weak references**, and memory collection won't consider whether this reference exists.

```js
let map = new WeakSet();
let value = { test: 22};
map.add(value);

value = null;
```

For Map:

The following has a memory leak (when **keys are reference types**, i.e., objects):

```js
let map = new Map();
let key = new Array(5);
map.set(key, 1);
key = null;
```

It needs to be changed to this to avoid memory leaks:

```js
let map = new Map();
let key = new Array(5);
map.set(key, 1);

map.delete(key);
key = null;
```

There's a more convenient way, using WeakMap, where the keys are **weak references**, and memory collection won't consider whether this reference exists.

```js
let map = new WeakMap();
let key = new Array(5);
map.set(key, 1);

key = null;
```

### Forgotten Closures

First, look at this code:

```js
function closure() {
  const name = 'xianshannan'
  return () => {
    return name
      .split('')
      .reverse()
      .join('')
  }
}
const reverseName = closure()
// Here we call reverseName
reverseName();
```

Is there a memory leak above?

There is no memory leak above because the `name` variable is needed (not garbage). This also reflects a disadvantage of closures from the side: relatively high memory usage, which can impact performance if there are many.

But changing it to this would result in a memory leak:

```js
function closure() {
  const name = 'xianshannan'
  return () => {
    return name
      .split('')
      .reverse()
      .join('')
  }
}
const reverseName = closure()
```

Strictly speaking, in the case where the current execution environment has not ended, there is a memory leak. The `name` variable is called by the function returned by `closure`, but the returned function is not used. In this scenario, `name` is garbage memory. `name` is not necessary, but it still occupies memory and cannot be reclaimed.

### References to Detached DOM

Every DOM on a page occupies memory. Suppose there is an element A on a page, we get the DOM object of element A and assign it to a variable (the memory reference is the same), and then remove element A from the page. If this variable is not reclaimed for other reasons, then there is a memory leak.