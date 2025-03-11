## 3 Methods to Determine if an Object is an Array in JS

### 1. Object.prototype.toString.call()

Every object that inherits from Object has a `toString` method. If the `toString` method hasn't been overridden, it returns `[object type]`, where type is the object's type. However, when types other than Object directly use the `toString` method, they return a string of their content. Therefore, we need to use the call or apply method to change the execution context of the toString method.

```js
const an = ['Hello','An'];
an.toString(); // "Hello,An"
Object.prototype.toString.call(an); // "[object Array]"
```

This method can determine all basic data types, even null and undefined.

```js
Object.prototype.toString.call('An') // "[object String]"
Object.prototype.toString.call(1) // "[object Number]"
Object.prototype.toString.call(Symbol(1)) // "[object Symbol]"
Object.prototype.toString.call(null) // "[object Null]"
Object.prototype.toString.call(undefined) // "[object Undefined]"
Object.prototype.toString.call(function(){}) // "[object Function]"
Object.prototype.toString.call({name: 'An'}) // "[object Object]"
```

### 2. instanceof

The internal mechanism of `instanceof` is to determine whether the object's prototype chain contains the type's `prototype`.

Using `instanceof` to determine if an object is an array, `instanceof` will check if the object's prototype chain contains the corresponding `Array` prototype. If found, it returns `true`; otherwise, it returns `false`.

```js
[]  instanceof Array; // true
```

### 3. Array.isArray()

A method added in ES5. When `Array.isArray()` doesn't exist, you can implement it using `Object.prototype.toString.call()`. The principle is simple:

```js
Array.prototype.myIsArray = function(arr) {
  return Object.prototype.toString.call(arr) === '[object Array]'
}
```

Reference article:

[Question 21: What are the differences and pros/cons between these 3 methods for determining if something is an array: Object.prototype.toString.call(), instanceof, and Array.isArray()?](https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/23)

## Declaration Functions vs. Assignment Functions

```js
// Declaration function
function A() {}

// Assignment function
var B = function() {}
```

Declaration functions undergo declaration hoisting, initialization hoisting, and assignment hoisting, while assignment functions only undergo declaration hoisting and initialization hoisting.

```js
A() // ---
function A() {
    console.log('---')
}

B()	// undefined is not a function error!
var B = function() {
    console.log('---')
}
```

This needs to be understood from the mechanism of variable hoisting that occurs during pre-compilation.

```js
// demo1
fn()
function fn() {
  console.log('---');
}
var fn = function() {
  console.log('+++')
}
fn()

/*
---
+++
*/

// demo2—Different order, same result
fn()
var fn = function() {
  console.log('+++')
}
function fn() {
  console.log('---');
}
fn()
/*
---
+++
*/
```

## Which Has Better Performance: call or apply?

call has better performance than apply. The format of parameters passed to call is exactly the format needed internally, so it avoids the extra step of destructuring the second parameter of apply.

## Why is the Performance of a Regular for Loop Far Better Than forEach? Please Explain the Reason

- A for loop doesn't have any additional function call stack and context.
- The function signature of forEach is actually:

```js
array.forEach(function(currentValue, index, arr), thisValue)
```

It's not just syntactic sugar for a regular for loop; it has many parameters and contexts to consider during execution, which may slow down performance.

## In an Array with 100,000 Elements, How Much Time Difference is There Between Accessing the First Element and the 100,000th Element?

Arrays can directly access elements by index, so the time complexity of accessing any element is O(1). Therefore, the conclusion is: **The time consumption is almost identical, and the difference can be ignored.**

From a principle perspective:

JavaScript doesn't have true arrays. All arrays are actually objects, and their "indices" look like numbers but are converted to strings to be used as property names (object keys). So whether you're accessing the 1st or the 100,000th element, it's a process of precisely looking up a key in a hash table, and the time consumption is roughly the same.

## The Principle of How JS Stores Numbers

JS uses the IEEE754 standard's 64-bit floating-point format to represent numbers.

![img](https://fengmumu1.github.io/2018/06/30/js-number/64w.jpg)

*sign*: Sign bit, 0 for positive, 1 for negative.

*exponent*: Exponent bits.

*fraction*: Significant digits. **IEEE754 stipulates that when storing significant digits in a computer, the first digit is always assumed to be 1, so it's omitted and only the rest is stored.** For example, when storing 1.01, only 01 is stored, and the first digit 1 is added back when reading. **So 52 bits of significant digits can actually store 53 bits.** (If the first digit is 0, it can be adjusted by moving the exponent.)

Why is the range of safe integers **-2^53 to 2^53 (excluding boundaries)**?

Only within this range is there a one-to-one correspondence between integers and 64-bit floating-point numbers.

## What Problems Arise When Using await in forEach? How to Solve This Problem?

Problem: **For asynchronous code, forEach cannot guarantee execution in order.**

```js
async function test() {
	let arr = [4, 2, 1]
	arr.forEach(async item => {
		const res = await handle(item)
		console.log(res)
	})
	console.log('End')
}

function handle(x) {
	return new Promise((resolve, reject) => {
		setTimeout(() => {
			resolve(x)
		}, 1000 * x)
	})
}

test()
/* 
Expected result:
4, 2, 1
Actual result:
1, 2, 4
*/
```

How to solve this problem?

It's actually quite simple; you can easily solve it using `for...of`.

```js
async function test() {
  let arr = [4, 2, 1]
  for(const item of arr) {
		const res = await handle(item)
		console.log(res)
  }
	console.log('End')
}
```

for...of doesn't use the simple, brute-force approach of forEach to iterate, but instead uses a special means—an `iterator`.

First, for arrays, they are `iterable data types`. So what are `iterable data types`?

> Data types that natively have the [Symbol.iterator] property are iterable data types. Examples include arrays, array-like objects (such as arguments, NodeList), Sets, and Maps.

Code to iterate through the array [4,2,1] using an iterator:

```js
let arr = [4, 2, 1];
```