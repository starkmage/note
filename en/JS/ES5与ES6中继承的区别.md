## Inheritance in ES5

* Combination Inheritance

```js
// Combination Inheritance
function A() {}
function B() {
  A.call(this)
}
B.prototype = new A()
B.prototype.constructor = B
let b = new B()

console.log(B.__proto__ === A);	// false
console.log(B.__proto__ === Function.__proto__);  // true
console.log(B.prototype.constructor === B);	// true
console.log(B.prototype.__proto__ === A.prototype);	// true
console.log(A.__proto__ === Function.prototype);	// true
console.log(A.prototype.__proto__ === Object.prototype);	// true
```

* Parasitic Combination Inheritance

```js
// Parasitic Combination Inheritance
function A() {}
function B() {
  A.call(this)
}
B.prototype = Object.create(A.prototype, {
  constructor: {
    value: B,
    enumerable: true,
    writable: true,
    configurable: true
  }
})
let b = new B()

console.log(B.__proto__ === A);	// false
console.log(B.__proto__ === Function.__proto__);  // true
console.log(B.prototype.constructor === B);	// true
console.log(B.prototype.__proto__ === A.prototype);	// true
console.log(A.__proto__ === Function.prototype);	// true
console.log(A.prototype.__proto__ === Object.prototype);	// true
```

Both methods above have the same prototype chain, but the difference is that in combination inheritance, B calls the parent constructor when inheriting from A, which adds unnecessary parent properties to the child's prototype, resulting in memory waste. Parasitic combination inheritance optimizes this issue.

Prototype chain of instance b:

![](http://img.stark.pub/20200928190624.png)

Prototype chain of constructor B. A's prototype chain is the same. Note! B's `__proto__` is not A, but Function's `prototype`, which is different from class inheritance.

![](http://img.stark.pub/20200928190804.png)

Through A.call(this), instance b of constructor B inherits the instance properties of constructor A, but constructor A and constructor B do not have an inheritance relationship, meaning constructor B does not inherit properties on constructor A.

```js
function A() {}
A.zsj = 'static property'
function B() {
  A.call(this)
}
B.prototype = Object.create(A.prototype)
console.log(B.zsj);	// undefined
```

## Inheritance in ES6

In ES6, inheritance between two classes is implemented using the two keywords extends and super.

```js
class A {
  constructor() {}
}
class B extends A {
  constructor() {
    super()
  }
}
let b = new B()

console.log(B.__proto__ === A);	// true
console.log(B.prototype.constructor === B);	// true
console.log(B.prototype.__proto__ === A.prototype);	// true
console.log(A.__proto__ === Function.prototype);	// true
console.log(A.prototype.__proto__ === Object.prototype);	// true
```

Prototype chain of instance b:

![](http://img.stark.pub/20200928185321.png)

Prototype chain of class B as shown:

![](http://img.stark.pub/20200928185148.png)