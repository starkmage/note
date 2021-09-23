## Node.js多进程

### 如何创建多进程

node中有提供`child_process`模块，这个模块中，提供了多个方法来创建子进程。

```js
const { spawn, exec, execFile, fork } = require('child_process');
```

这4个方法都可以创建子进程，不过使用方法还是稍微有点区别。

### 多进程之间的通信

node中进程的通信主要在主从（子）进程之间进行通信，子进程之间无法直接通信，若要相互通信，则要通过主进程进行信息的转发。

主进程和子进程之间是通过IPC（Inter Process Communication，进程间通信）进行通信的，IPC也是由底层的libuv根据不同的操作系统来实现的。

参考文章：

[node多进程的创建与守护](https://zhuanlan.zhihu.com/p/100550801)

[Node.js 多进程](https://www.runoob.com/nodejs/nodejs-process.html)

## Node 的优势

业务场景大多数情况下就分两种。一种是计算CPU密集型而另外一种呢是IO密集型。所以得分情况讨论。

在IO密集型的业务场景下，也就是我们大多数的web业务。Node.js利用自身异步IO非阻塞的特性再加上事件驱动模型。能够在消耗资源很少的情况下，支撑高并发。

很多人只知道Node.js是单线程，可是不知道支撑Node.js的IO操作是底层C++的libuv，而libuv是有线程池的。所以总是对Node.js有一些误解。

再说CPU密集型的业务场景，这个时候Node的单线程模型的劣势就体现出来了。因为CPU密集型的业务，Node的特性完全没有用物之地。所以这也是为什么几乎所有用Node的公司都是拿来写业务或者做BFF层的原因。

所以再回头来看问题，Node.js的高并发高性能的关键就就是自带异步IO非阻塞的特性和事件驱动模型。