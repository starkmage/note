## Differences Between Set, Map, WeakSet, and WeakMap

Set

1. Members cannot be duplicated
2. Only has values, no keys, somewhat similar to an array
3. Can be iterated, with methods including add, delete, has

WeakSet

1. Members are all objects
2. Members are all weak references and can disappear at any time. Can be used to store DOM nodes, not easily causing memory leaks
3. **Cannot be iterated**, methods include add, delete, has

Map

1. Essentially a collection of key-value pairs, similar to a collection
2. Can be iterated, has many methods, can convert between various data formats

WeakMap

1. Can **only accept objects as keys** (except null), does not accept other types of values as keys
2. Objects pointed to by **keys** are not counted in the garbage collection mechanism
3. **Cannot be iterated**, methods include get, set, has, delete

## The Development History and Pros & Cons of JS Asynchronous Solutions

1. Callback Functions

   Advantages:

   * **Solved the synchronous problem** (if one task takes a long time, all subsequent tasks must wait in line, which delays the execution of the entire program)

   Disadvantages:

   * Callback hell, **cannot use try catch to capture errors, cannot return**
   * Lack of sequentiality: Callback hell leads to debugging difficulties, inconsistent with the brain's thinking pattern
   * Nested functions have coupling issues; once changed, it affects everything else, i.e., (**inversion of control**)
   * Too many nested functions make error handling difficult

2. Promise

   Implements chained calls, meaning that each 'then' returns a brand new Promise. If we return something in a then, the result will be wrapped by Promise.resolve()

   Advantages:

   * Solves the callback hell problem

   Disadvantages:

   * **Cannot cancel a Promise, errors need to be caught through callback functions**

3. Async/await

   Through **coroutines and Promise**, writing asynchronous code in a synchronous form, the ultimate solution for asynchronous operations

   Advantages:

   * **Code is cleaner, no need to write a bunch of then chains like with Promise, addresses the callback hell problem**

   Disadvantages:

   * **await converts asynchronous code to synchronous code; if multiple asynchronous operations have no dependencies but use await, it will lead to performance degradation.**

     ```js
     async function test() {
       // If the following code has no dependencies, Promise.all can be used
       // If there are dependencies, it's actually an example of solving callback hell
       await fetch('XXX1')
       await fetch('XXX2')
       await fetch('XXX3')
     }
     ```

## Summary of Methods for Iterating Over Objects in JS

1. Object.keys(obj)

   Returns an array including all enumerable properties of the object itself (not including inherited ones) (not including Symbol properties)

2. for...in

   Iterates over the object's own and inherited enumerable properties (not including Symbol properties)

3. Object.getOwnPropertyNames(obj)

   Returns an array containing all properties of the object itself (not including Symbol properties, but including non-enumerable properties)

4. Reflect.ownKeys(obj)

   Returns an array containing all properties of the object itself, regardless of whether the property name is a Symbol or string, and regardless of whether it is enumerable

| Method | Inherited | Non-enumerable | Symbol |
| ------------------------------- | ------ | ---------- | ------ |
| Object.keys(obj) | ❌ | ❌ | ❌ |
| for...in | ✅ | ❌ | ❌ |
| Object.getOwnPropertyNames(obj) | ❌ | ✅ | ❌ |
| Reflect.ownKeys(obj) | ❌ | ✅ | ✅ |