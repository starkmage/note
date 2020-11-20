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