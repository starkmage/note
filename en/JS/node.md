## Node.js Multi-process

### How to Create Multiple Processes

Node provides a `child_process` module, which offers multiple methods to create child processes.

```js
const { spawn, exec, execFile, fork } = require('child_process');
```

All four methods can create child processes, though there are slight differences in how they are used.

### Communication Between Multiple Processes

In Node, process communication primarily occurs between the main (parent) and child processes. Child processes cannot communicate directly with each other; if they need to exchange information, it must be forwarded through the main process.

The main process and child processes communicate through IPC (Inter Process Communication), which is implemented by the underlying libuv library according to different operating systems.

Reference articles:

[Node Multi-process Creation and Daemon](https://zhuanlan.zhihu.com/p/100550801)

[Node.js Multi-process](https://www.runoob.com/nodejs/nodejs-process.html)

## Advantages of Node

Business scenarios generally fall into two categories: CPU-intensive and IO-intensive. So we need to discuss them separately.

In IO-intensive business scenarios, which include most of our web services, Node.js leverages its asynchronous non-blocking IO characteristics along with its event-driven model. This allows it to support high concurrency while consuming very few resources.

Many people only know that Node.js is single-threaded but are unaware that the IO operations supporting Node.js are handled by the C++ libuv library, which has a thread pool. This leads to some misconceptions about Node.js.

In CPU-intensive business scenarios, the disadvantages of Node's single-threaded model become apparent. For CPU-intensive tasks, Node's special features have no place to be utilized. This is why almost all companies that use Node use it for writing business logic or as a BFF (Backend for Frontend) layer.

So to revisit the question, the key to Node.js's high concurrency and high performance lies in its built-in asynchronous non-blocking IO characteristics and event-driven model.