## Differences Between Arrow Functions and Regular Functions

1. Arrow functions don't have a `prototype` (prototype object)

   ```js
   let fn = () => {
     console.log('---')
   }
   
   console.log(fn.prototype)
   // undefined
   ```

2. Arrow functions don't create their own `this`, so they don't have their own `this`. They only inherit `this` from the upper level of their scope chain (more strictly, from the first level up that has a this)

3. If there's no regular function outside an arrow function, in both strict and non-strict modes, its this will point to `window` (global object). In strict mode, directly calling a regular function will make its this point to undefined

   ```js
   let fn = () => {
     'use strict'
     console.log(this)
   }
   fn()  // Window
   
   let fn2 = function() {
     'use strict'
     console.log(this)
   }
   fn2() // undefined
   ```

4. `.call()/.apply()/.bind()` cannot change the this reference in arrow functions

5. If an arrow function's this points to `window` (global object), using `arguments` will cause an error. When an arrow function's this points to a regular function, its `arguments` inherits from that regular function

   ```js
   let fn = () => {
     console.log(arguments)
   }
   
   fn(1, 2, 3) // Error
   // Uncaught ReferenceError: arguments is not defined
   ```

   ```js
   let fn = function() {
     console.log(arguments)
     let arrowFn = () => {
       console.log(arguments)
     }
     arrowFn()
   }
   
   fn(1, 2, 3)
   // Prints the arguments object twice
   ```

6. Arrow functions cannot be used as constructors, which means you cannot use the `new` command with them, otherwise an error will be thrown

7. Cannot use the `yield` command, therefore arrow functions cannot be used as Generator functions



## Additional Note on Rest Parameters

An ES6 API used to get an array of a variable number of function parameters. This API is meant to replace `arguments`

```js
let fn = (a, ...b) => {
  console.log(a)
  console.log(b)
  console.log(Array.isArray(b))
}

fn(1, 2, 3)
/*
1
[2, 3]
true
*/
```

Advantages over `arguments`:

1. Can be used in both arrow functions and regular functions
2. More flexible, the number of parameters received is completely customizable
3. Better readability, parameters are defined within the function parentheses, not suddenly appearing as `arguments`
4. Rest is a real array and can use array APIs, while `arguments` is an array-like object

Two points to note about rest parameters:

1. Rest must be the last parameter of the function

2. The length property of a function does not include rest parameters

   ```js
   let fn = (a, ...b) => {
     console.log(a)
     console.log(b)
     console.log(Array.isArray(b))
   }
   
   console.log(fn.length) // 1
   ```