## Why JavaScript is Single-threaded

This is mainly related to the purpose of JavaScript. JS was created as a browser scripting language, primarily to implement user interaction with the browser and manipulate the DOM. This determined that it could only be single-threaded, otherwise it would bring very complex synchronization issues.

For example: if JS had been designed with multiple threads, and one thread wanted to modify a DOM element while another thread wanted to delete that DOM element, what should the browser do? Therefore, to avoid complexity, JavaScript has been single-threaded since its inception, which has become a core feature of the language.

## Two Major Features of Node.js

### Node.js Non-blocking I/O, Asynchronous I/O

#### What is I/O

I/O means Input/Output. In the browser, there is only one type of I/O, which is using Ajax to send network requests and then read the returned content, which belongs to `network I/O`.

In Node.js, I/O scenarios are much more extensive, mainly divided into two types:

- `File I/O`, such as using the `fs` module to read and write files
- `Network I/O`, such as using the `http` module to initiate network requests

#### Blocking, Non-blocking I/O

`Blocking` and `non-blocking I/O` actually refer to the operating system kernel, not Node.js itself. The characteristic of `blocking I/O` is that it **must wait until the operating system completes all operations before the call is considered complete**, and the process will be blocked. `Non-blocking I/O` returns immediately after the call, without waiting for the operating system kernel to complete the operation.

For the former, during the I/O operation process in the operating system, our application is actually in a waiting state and cannot do anything. If we switch to `non-blocking I/O`, after the call returns, our Node.js application can complete other tasks while the operating system is also performing I/O. This makes full use of the waiting time, improving execution efficiency, but it also creates a problem: how does the Node.js application know that the operating system has completed the I/O operation?

To let Node.js know that the operating system has completed the I/O operation, it needs to repeatedly check with the operating system whether it has completed, and this repeated checking method is called `polling`. For polling, the CPU either repeatedly checks I/O, repeatedly checks file descriptors, or sleeps, none of which utilizes the CPU well. The ideal situation we want is:

After the Node.js application initiates an I/O call, it can directly go to execute other logic, and after the operating system silently completes the I/O, it sends a completion signal to Node.js, and Node.js executes the callback operation.

This is the effect of asynchronous I/O.

#### Asynchronous I/O

Asynchronous I/O in Node.js uses a multi-threaded approach, with four key elements: `EventLoop`, `I/O observers`, `request objects`, and `thread pool` working together to achieve this.

So what's the difference between `synchronous I/O` and `asynchronous I/O`? Is it possible to achieve `asynchronous I/O` just by implementing `non-blocking I/O`?

No, it's not.

- `Synchronous I/O` blocks the process when doing `I/O operations`, so `blocking I/O`, `non-blocking I/O`, and `I/O multiplexing` are all `synchronous I/O`.
- `Asynchronous I/O` will not cause any blocking when doing `I/O operations`.

Why isn't `non-blocking I/O` considered `asynchronous I/O` if it doesn't block? Actually, when `non-blocking I/O` has data ready, it still needs to block the process to get data from the kernel. So it doesn't qualify as `asynchronous I/O`.

https://www.zhihu.com/question/19732473

### Node.js Event-Driven

Event-driven is the second major feature of Node.js. What is event-driven? **Simply put, it's about performing corresponding operations by monitoring changes in event states. For example, when reading a file, when the file is read completely or an error occurs, the corresponding state is triggered, and the corresponding callback function is called to handle it.**

`nodejs` runs in a **single thread**, and through an **event-loop**, it cycles through messages in the **event-queue** for processing. The processing is basically calling the callback function corresponding to that **message**. When an event state changes, a message is pushed into the **message queue**.

There are several points to note about Node.js's event-driven model:

- Because it's **single-threaded**, the **event loop** is paused when executing code in a `js` file sequentially.
- After the `js` file execution is complete, the **event loop** starts running and takes messages from the **message queue** to start executing callback functions.
- Because it's **single-threaded**, the **event loop** is paused when callback functions are being executed.
- When it comes to I/O operations, `nodejs` opens an independent thread to perform `asynchronous I/O` operations, and after the operation is complete, it pushes a message into the **message queue**.

[nodejs asynchronous I/O and event-driven](https://segmentfault.com/a/1190000005173218)

Node.js's event-driven model is based on EventEmitter.

## Three Major Characteristics of Object-Oriented JavaScript

* Encapsulation

  Encapsulation is about packaging abstracted data and operations on that data together. The data is protected internally, and other parts of the program can only manipulate the data through authorized operations (member methods).

   JS encapsulation only has two states: public and private.

  ```js
  function Person(name, age){
    // Public
    this.name = name
    // Private
    var age = age
  }
  var p1 = new Person('zs', 20)
  ```

* Inheritance

  In ES5, the most commonly used inheritance methods are combination inheritance and parasitic combination inheritance, while in ES6 it's the extends keyword.

* Polymorphism

  Polymorphism is about separating what is being done from who is doing it. The same operation applied to different objects can have different interpretations and produce different execution results.

  Because JS is a dynamic language, it inherently has polymorphism.

  The example below illustrates that whether an animal can make a sound depends only on makeSound, not on a specific type of object.

  ```js
   var makeSound=function (animal) {
     animal.sound();
   }
   var Duck=function () {
   }
   var Dog=function () {
  ```