## Browser

This part has been summarized in previous articles, so I won't elaborate here. This time I'll summarize a few new examples I've encountered.

### Standard promise + setTimeout

The constructor function of a promise is executed immediately upon creation. The resolve/reject functions are used to change the state of the promise, and only when this state is passed to then, the functions in then will be pushed into the microtask queue.

```js
const first = () => (new Promise((resolve, reject) => {
  console.log(3)
  let p = new Promise((resolve, reject) => {
    console.log(7)
    setTimeout(() => {
      console.log(5)
      resolve(6)
    }, 0)
    resolve(1)
  })
  resolve(2)
  p.then((arg) => {
    console.log(arg)
  })

}))

first().then((arg) => {
  console.log(arg)
})
console.log(4)
// 3 7 4 1 2 5
```

```js
setTimeout(() => {
  console.log("0")
}, 0)
new Promise((resolve, reject) => {
  console.log("1")
  resolve()
}).then(() => {
  console.log("2")
  new Promise((resolve, reject) => {
    console.log("3")
    resolve()
  }).then(() => {
    console.log("4")
  }).then(() => {
    console.log("5")
  })
}).then(() => {
  console.log("6")
})

new Promise((resolve, reject) => {
  console.log("7")
  resolve()
}).then(() => {
  console.log("8")
})
//1, 7, 2, 3, 8, 4, 6, 5, 0
```

If the parameter of then is not a function, it won't be added to the microtask queue and will still be executed synchronously.

```js
setTimeout(() => {
  console.log(1);
}, 0);

new Promise(function (resolve) {
  resolve();
  console.log(2);
}).then(console.log(3))

console.log(4);
// 2 3 4 1
```

### async + await

`async/await` uses `coroutines` and `Promise` to achieve the effect of writing asynchronous code in a synchronous way, where `Generator` is an implementation of `coroutines`.

Coroutines are lighter than threads. Coroutines exist within a thread environment, and a thread can have multiple coroutines. You can think of coroutines as individual tasks within a thread. Unlike processes and threads, coroutines are not managed by the operating system but are controlled by specific application code.

A thread can only execute one coroutine at a time. For example, if coroutine A is currently executing and there's another coroutine B, to execute B's task, coroutine A must transfer control of the JS thread to coroutine B. Then B executes, and A is essentially in a paused state.

The above principle leads to some differences in execution order:

First, let's look at the Promise case:

```js
console.log('a');

async function test() {
  console.log(1)
  new Promise((resolve, reject) => {
    resolve(2)
  }).then(value => {console.log(value)})
  console.log(3)
}

test()
console.log('b')

Promise.resolve('4').then(
  value => {
    console.log(value);
  }
)
// a 1 3 b 2 4
```

Now with await:

```js
console.log('a');

async function test() {
  console.log(1)
  let res = await 2
  console.log(res)
  console.log(3)
}

test()
console.log('b')

Promise.resolve('4').then(
  value => {
    console.log(value);
  }
)
// a 1 b 2 3 4
```

When executing await, the JS engine pauses the current coroutine and gives thread control to the parent coroutine. For a detailed explanation, see [this article](http://47.98.159.95/my_blog/js-async/011.html#async).

#### Super Hard Example 1

```js
console.log('script start');

setTimeout(() => {
  console.log('啊啊啊');
}, 1 * 2000);

Promise.resolve()
.then(function() {
  console.log('promise1');
}).then(function() {
  console.log('promise2');
});


async function foo() {
  await bar()
  console.log('async1 end')
}
foo()

async function errorFunc () {
  try {
    await Promise.reject('error!!!')
  } catch(e) {
    console.log(e)
  }
  console.log('async1');
  return Promise.resolve('async1 success')
}
errorFunc().then(res => console.log(res))

function bar() {
  console.log('async2 end') 
}

console.log('script end');
/*
First round: script start, async2 end, script end
Second round: promise1, async1 end, error!!!, async1
Third round: promise2, async1 success
Fourth round: 啊啊啊
*/
```

#### Super Hard Example 2

```js
new Promise((resolve, reject) => {
  console.log(1)
  resolve()
})
  .then(() => {
    console.log(2)
    new Promise((resolve, reject) => {
      console.log(3)
      setTimeout(() => {
        reject();
      }, 3 * 1000);
      resolve()
    })
      .then(() => {
        console.log(4)
        new Promise((resolve, reject) => {
          console.log(5)
          resolve()
        })
          .then(() => {
            console.log(7)
          })
          .then(() => {
            console.log(9)
          })
      })
      .then(() => {
        console.log(8)
      })
  })
  .then(() => {
    console.log(6)
  })
/*
First round: 1
Second round: 2, 3
Third round: 4, 5, 6
Fourth round: 7, 8
Fifth round: 9
*/
```