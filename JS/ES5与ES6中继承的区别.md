## ES5中的继承

* 组合式继承

``` js
// 组合式继承
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

* 寄生组合式继承

``` js
// 寄生组合式继承
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

上面两种方式的原型链都是一致的，不同的是组合式继承 B 在继承 A 的时候调用了父类构造函数，导致子类的原型上多了不需要的父类属性，存在内存上的浪费，而寄生组合继承进行了优化。

实例 b 原型链

![](http://img.stark.pub/20200928190624.png)

构造函数 B 的原型链，A的原型链也是一样的，注意！B 的 `__proto__` 并不是 A，而是 Function 的 `prototype`，这也是与 class 继承不同的地方

![](http://img.stark.pub/20200928190804.png)

通过 A.call(this)，构造函数 B 的实例 b 继承了构造函数 A 的实例属性，但是构造函数 A 与 构造函数 B 并没有继承关系，即构造函数 B 没有继承构造函数 A 上面的属性。

``` js
function A() {}
A.zsj = '静态属性'
function B() {
  A.call(this)
}
B.prototype = Object.create(A.prototype)
console.log(B.zsj);	// undefined
```

## ES6中的继承

在 ES6 中，两个类之间的继承就是通过 extends 和 super 两个关键字实现的。

``` js
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

实例 b 的原型链

![](http://img.stark.pub/20200928185321.png)

类 B 的原型链如图所示

![](http://img.stark.pub/20200928185148.png)

简单理解可以是这样

![](https://cdn.jsdelivr.net/gh/starkmage/ImgHosting/starkmage-picgo/20200828122513.png)

``` js
class A {
  constructor() {}
}
A.zsj = '静态属性'
class B extends A {
  constructor() {
    super()
  }
}
console.log(B.zsj);	// 静态属性
```



## 总结

1. ES5 中子类构造函数不会继承父类构造函数的属性，而 ES6 中子类会继承父类的属性。ES5 中子类构造函数的 `__proto__` 指向的是 Function.prototype ，而 ES6 中是指向父类构造函数的。
2. ES5，构造函数 B 的实例继承构造函数 A 的实例属性是通过 A.call(this) 来实现的，继承实质上是先创建子类的实例对象`this`，然后再将父类的属性添加到`this`上（`Parent.apply(this)`）。
3. ES6的继承机制完全不同，类 B 的实例继承类 A 的实例属性，是通过 super() 实现的。实质是先将父类实例对象的属性和方法，加到`this`上面（所以必须先调用`super`方法），然后再用子类的构造函数修改`this`。
4. ES5的继承时通过原型或构造函数机制来实现。ES6通过`class`关键字定义类，里面有构造方法，类之间通过`extends`关键字实现继承。子类必须在`constructor`方法中调用`super`方法，否则新建实例报错。因为子类没有自己的`this`对象，而是继承了父类的`this`对象，然后对其进行加工。如果不调用`super`方法，子类得不到`this`对象。

在不是继承原生构造函数（比如继承 Array）的情况下，A.call(this) 与 super() 在功能上是没有区别的，用 [babel 在线转换](https://babeljs.io/repl/#?babili=false&evaluate=true&lineWrap=false&presets=es2015,react,stage-2&targets=&browsers=&builtIns=false&debug=false&code=) 将类的继承转换成 ES5 语法，babel 也是通过 A.call(this) 来模拟实现 super() 的。

但是在继承原生构造函数的情况下，A.call(this) 与 super() 在功能上是有区别的，ES5 中 A.call(this) 中的 this 是构造函数 B 的实例，也就是在实现实例属性继承上，ES5 是先创造构造函数 B 的实例，然后在让这个实例通过 A.call(this) 实现实例属性继承，在 ES6 中，是先新建父类的实例对象`this`，然后再用子类的构造函数修饰 this，使得父类的所有行为都可以继承。

参考文章：

[ES6 与 ES5 继承的区别](https://juejin.im/post/6844903924015120397)