## Problems with Multiple `<script>` Tags

- Too Many Requests

When we need to depend on multiple modules, it will result in multiple requests, leading to excessive requests.

- Unclear Dependencies

We don't know their specific dependency relationships, which means it's easy to get the loading order wrong due to not understanding the dependencies between them.

- Hard to Maintain

The above two reasons make it difficult to maintain, and it's likely that changing one thing will affect the entire project, causing serious problems.

## Benefits of Modularization

- Avoid naming conflicts (reduce namespace pollution)
- Better separation, load on demand
- Higher reusability
- Better maintainability

## CommonJS

### Basic Rules

Allows modules to **synchronously load** other dependent modules through the require method, and then export the interfaces that need to be exposed through exports or module.exports.

```
module.exports default value is {}

exports is a reference to module.exports

exports by default points to module.exports' memory space

require() returns module.exports, not exports

If exports is reassigned, it breaks the reference to module.exports
```

**The basic functionality of the require command is to read and execute a JavaScript file, then return that module's exports object. If the specified module is not found, it will throw an error**.

Modules can be loaded multiple times, but they will only run once on the first load, and then the execution result is cached. For subsequent loads, it will directly read from the cache. To run the module again, you must clear the cache.

The order of module loading follows their order of appearance in the code.

Conventional way:

``` js
//Assume this is a.js file
var foo = 'a'
exports.foo = 'hello'
var age = 19
exports.age = age
//You can also export multiple items like this
module.exports = {
  foo: 'a',
  age: 19
}

//Assume this is b.js file
var res = require(./a)
console.log(res)
//{foo:'hello', age:19}
console.log(res.age)
//19

// You can also do this
var {foo, age} = require(./a)
```

Exporting a single item:

``` js
//a.js file
function add(x, y) {
  return x + y
}

module.exports = add

//b.js file
let foo = require('./a')
console.log(foo)
//foo is the add function
```

### Characteristics

Advantages:

1. Simple and easy to use
2. Server-side modules are easy to reuse

Disadvantages:

1. **Synchronous loading** is not suitable for browser environment, as synchronous means blocking loading, while browser resources are loaded asynchronously
2. Cannot load multiple modules in parallel non-blockingly

Why can't browsers use synchronous loading while servers can?

- Because modules are stored on the server side, synchronous module loading is possible for the server
- For the browser side, since modules are on the server side, loading time depends on factors like network speed, if it needs to wait for a long time, the entire application will be blocked
- Therefore, browser-side modules cannot use "synchronous loading" (CommonJS), they can only use "asynchronous loading" (AMD)

## ESModule

The design philosophy of ES6 modules is to be as static as possible, so that module dependencies and input/output variables can be determined at compile time. CommonJS and AMD modules can only determine these at runtime.

ES6 implements module functionality at the language standard level, and the implementation is quite simple. It can completely replace CommonJS and AMD specifications to become a universal module solution for browsers and servers.

### Basic Rules

Method 1:

``` js
// test.js
let num = 0;
const add = function (a, b) {
    return a + b;
};
export { num, add };

// Import
import { num, add } from './test';
```

Method 2:

``` js
// test.js
export let a = 1
export function test() {}

// Import
import {a, test} from './test.js'
```

Method 3:

``` js 
// test.js
// 1.
const obj = {}
export default obj
// 2.
export default function () {}
// 3.
export default function test() {}
// Error
export default const obj = {}

// Import
import diyname from './test.js'
```

### Comparison with CommonJS

1. `CommonJS` runs when loaded, `ESModule` outputs interfaces at compile time. Because `CommonJS` exports an object (i.e., module.exports property), this object is only generated after the script finishes running. ES6 modules are not objects, their external interface is just a static definition that is generated during code static analysis phase, so it will prioritize executing the code in the export file, whichever JS file is referenced first gets executed first
2. `CommonJS` outputs a **shallow copy** of values, `ESModule` outputs **dynamic read-only references**. For read-only, it means you cannot modify the value of imported variables, imported variables are read-only regardless of whether they are primitive data types or complex data types. When a module encounters the import command, it generates a read-only reference

3. When using the `require` command to load the same module, it won't execute that module again, but will get the value from the cache. In other words, no matter how many times a `CommonJS` module is loaded, it will only run once on the first load, and subsequent loads will return the result from the first run, unless the system cache is manually cleared. ESModule doesn't cache values, variables in the module are bound to their module. When the script actually executes, it goes to the loaded module to get values based on this read-only reference.

## AMD

The CommonJS specification is mainly used for server-side programming, where module loading is synchronous, which is not suitable for the browser environment because synchronous means blocking loading while browser resources are loaded asynchronously. Therefore, AMD CMD solutions were created.

require.js is an implementation of AMD

AMD also uses the require() statement to load modules, but unlike CommonJS, it requires two parameters:

``` js
require([module], callback);
```

The first parameter [module] is an array whose members are the modules to be loaded; the second parameter callback is the callback function after successful loading. If we rewrite the previous code in AMD form, it would look like this:

``` js
require(['math'], function (math) {
  math.add(2, 3);
});
```

math.add() and math module loading are not synchronous, the browser won't freeze. So obviously, AMD is more suitable for browser environments.

## CMD

## UMD

## Can ESModule Achieve Runtime Loading Since It's Compile-time Loading?

The standard usage of import for module importing is static, which means all imported modules are compiled at load time (unable to achieve on-demand compilation, reducing initial page load speed).

In some scenarios, you might want to import modules conditionally or on-demand. In these cases, you can use dynamic imports instead of static imports. Here are scenarios where dynamic imports might be needed:

- **When statically imported modules significantly slow down code loading and have a low probability of being used, or when they're not needed immediately.**

- **When statically imported modules consume a large amount of system memory and have a low probability of being used.**

- **When the imported module doesn't exist at load time and needs to be fetched asynchronously**

- **When the module specifier needs to be constructed dynamically. (Static imports can only use static specifiers)**

  ``` js
  let filename = 'module.js'; 
  
  import('./' + filename).then(module =>{
      console(module);
  }).catch(err => {
      console(err.message); 
  });
  ```

- When the imported module has side effects (side effects here can be understood as code that runs directly in the module), and these side effects are only needed when certain conditions are triggered. (In principle, modules shouldn't have side effects, but often you can't control the content of the modules you depend on)

**Don't abuse dynamic imports (only use when necessary). Static frameworks can better initialize dependencies and are more beneficial for static analysis tools and tree shaking to work effectively.**

Script code:

``` html
<script type="module">
  // Import doubleKill module
  import('./doubleKill.js').then((module) => {
    // Execute default method
    module.default();
    // Set color to red
    module.pColor('red');
  });
</script>
```

doubleKill.js code:

```js
// const and default functionality demo
export default () => {
  const p = document.querySelector('p');
  p.style.transform = 'scaleY(-1)';
};
export const pColor = (color) => {
  const p = document.querySelector('p');
  p.style.color = color;
}
```

This usage also supports the `await` keyword.

```js
let module = await import('/modules/my-module.js');
```

[Differences between ES6 modules and CommonJS modules](https://github.com/YvetteLau/Step-By-Step/issues/43)

https://segmentfault.com/a/1190000015991869

https://juejin.im/post/6844903744518389768#heading-50

https://juejin.im/post/6844903598480965646

https://www.jianshu.com/p/1cfc5673e61d

https://github.com/ljianshu/Blog/issues/48