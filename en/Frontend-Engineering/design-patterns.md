## Singleton Pattern

Ensures a class has only one instance and provides a global access point to it. In other words, when creating a new object using the same class for the second time, you should get an object that is exactly the same as the one created the first time.

### Singleton Applications

* Global variables `window` and `document` in browsers are both singletons. These objects are the same whenever you access them. `window` represents the window containing the DOM document, and `document` is the DOM document loaded in the window, each providing their related methods.

- In ES6's new syntax Module feature, variables exported from modules through `import/export` are singletons, meaning if you change the value of an internal variable in the module somewhere, when others reference this value again, it will be the changed value.
- The global state maintained by Vuex for global state management in Vue projects, and the router instance maintained by `vue-router`, are singleton applications in single pages of a single-page application (but not applications of the singleton pattern).

### Implementation

- Always returns the same instance when accessed;
- Self-instantiating, whether created at initial loading or on first access;
- Generally provides a getInstance method to obtain its instance;

### Advantages

- Singleton pattern only exists as one instance in memory after creation, saving memory costs and instantiation performance costs;
- Singleton pattern can solve multiple resource occupancy, for example, when writing files, having only one instance can avoid simultaneous operations on a file;
- Using only one instance can also reduce pressure on the garbage collection mechanism, which in browsers means less system lag, smoother operations, and less CPU resource usage;

### Disadvantages

- Singleton pattern is not friendly to extension, generally not easy to extend because singleton pattern usually self-instantiates without interfaces;

### Application Scenarios

- When instantiating a class consumes too many resources, singleton pattern can be used to avoid performance waste;
- When a project needs a public state, singleton pattern needs to be used to ensure access consistency;

``` js
class SingleTon {
  constructor(name) {
    this.name = name
  }
  
  static instance = null // Initialize a variable

  static getInstance(name) {
    // When creating new object, check if the object exists globally, if yes, return that object, if no, create a new object and return it.
    if (!this.instance) {
      this.instance = new SingleTon(name)
    }
    return this.instance
  }
}

var oA = SingleTon.getInstance('Lee')
var oB = SingleTon.getInstance('Fan')
console.log(oA === oB) // true
```

## Factory Pattern

**Returns instances of different classes based on different inputs, generally used to create objects of the same type. The main idea of the factory method is to separate object creation from object implementation.**

Object constructor

``` js
function createPerson(name, age) {
  var obj = new Object()
  obj.name = name
  obj.age = age
  obj.sayName = function() {
    console.log(this.name)
  }
  return obj
}

var person1 = createPerson('zsj', 19)
```

## Publish-Subscribe Pattern

Event-bus

node.js's EventEmitter

## Observer Pattern

Vue's reactive system

## Reference Articles

[Common Design Patterns in JavaScript](https://juejin.im/post/6844903607452581896#heading-0)