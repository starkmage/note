# async、await与generator

https://yuanbao.tencent.com/chat/naQivTmsDa/b63b2d60-59bd-47d9-b62e-11fe6dd4b971?projectId=2f13020843e3426ab5f1fc84dfe2aa87

(calculateFPS)是怎么给函数传参的

#  `performance.now()` 是什么？

`performance.now()` 是 Web Performance API 提供的高精度时间戳方法，具有以下特点：

- **高精度**：返回以毫秒为单位的时间戳，精度可达微秒级（至少0.1ms）
- **相对时间**：返回从页面加载开始的时间（不是系统时间）
- **单调递增**：不受系统时间调整影响，保证始终递增
- **用途**：非常适合用于性能测量、动画计时等场景

对比 `Date.now()`：

```js
console.log(Date.now());       // 返回自1970年1月1日至今的毫秒数（系统时间）
console.log(performance.now()); // 返回自页面加载开始的毫秒数（高精度）
```

# `requestAnimationFrame` 如何传递时间戳参数

`requestAnimationFrame` 会自动将高精度时间戳作为参数传递给回调函数：

```js
requestAnimationFrame((timestamp) => {
  // 这个timestamp参数由浏览器自动传入
  console.log(timestamp); // 类似 performance.now() 的值
});
```

# Proxy对象

## 🔍 什么是 `Proxy`？

> `Proxy` 是 ES6 引入的原生对象，用于**创建一个对象的代理**，从而可以拦截和自定义对象的基本操作（如读取属性、赋值、函数调用等）。

------

## 🧠 核心语法

```js
const proxy = new Proxy(target, handler);
```

- `target`：要代理的对象（可以是对象、数组、函数等）
- `handler`：一个对象，定义拦截逻辑（称为“捕捉器”trap）

------

## 🔧 常用的拦截操作（trap）

| trap 名称        | 拦截行为               | 示例                     |
| ---------------- | ---------------------- | ------------------------ |
| `get`            | 读取属性               | `proxy.name`             |
| `set`            | 设置属性               | `proxy.name = 'Tom'`     |
| `has`            | 判断属性是否存在       | `'name' in proxy`        |
| `deleteProperty` | 删除属性               | `delete proxy.name`      |
| `apply`          | 调用函数（函数代理时） | `proxyFn()`              |
| `construct`      | 使用 `new` 创建实例时  | `new proxyConstructor()` |

------

## ✅ 示例 1：拦截对象属性读取和写入

```js
const user = { name: "Tom", age: 25 };

const proxyUser = new Proxy(user, {
  get(target, prop) {
    console.log("读取属性:", prop);
    return target[prop];
  },
  set(target, prop, value) {
    console.log("设置属性:", prop, "=", value);
    target[prop] = value;
    return true;
  }
});

proxyUser.name;          // 控制台：读取属性: name
proxyUser.age = 30;      // 控制台：设置属性: age = 30
```

------

## ✅ 示例 2：数组保护

```js
const list = [1, 2, 3];

const safeList = new Proxy(list, {
  get(target, prop) {
    if (prop < 0 || prop >= target.length) {
      return undefined;
    }
    return target[prop];
  }
});

console.log(safeList[1]);  // 2
console.log(safeList[99]); // undefined，而不是抛错
```

------

## ✅ 示例 3：函数防抖（函数代理 + apply）

```js
function search() {
  console.log("请求接口:", Date.now());
}

const debouncedSearch = new Proxy(search, {
  apply(target, thisArg, args) {
    clearTimeout(target.timer);
    target.timer = setTimeout(() => target.apply(thisArg, args), 300);
  }
});

window.addEventListener("input", debouncedSearch);
```

------

## ✅ 示例 4：构造函数代理（拦截 new）

```js
class Person {
  constructor(name) {
    this.name = name;
  }
}

const PersonProxy = new Proxy(Person, {
  construct(target, args) {
    console.log("正在创建实例:", args);
    return new target(...args);
  }
});

const p = new PersonProxy("Alice"); // 控制台：正在创建实例: ["Alice"]
```

------

## 🎯 Proxy 常见用途总结

| 用途            | 示例                                    |
| --------------- | --------------------------------------- |
| 数据保护        | 拦截非法访问或修改属性                  |
| 日志/监控       | 读取属性或调用函数时记录日志            |
| 接口防抖/节流   | 包装函数执行逻辑                        |
| 统一格式转换    | 拦截数据访问，实现小驼峰/下划线自动转换 |
| 虚拟属性/懒加载 | 只有访问到某属性时才真正计算/加载       |
| mock 数据代理   | 用于测试或前端 mock                     |

------

## 🧠 面试建议回答方式

> Proxy 是我比较喜欢的一个特性，在项目中我用它做过接口缓存、访问控制、防抖包装，还模拟过前端 mock 接口返回。它相比 Object.defineProperty 更灵活，支持函数、数组、构造器等的拦截。

------

## ⚠️ 注意事项

- `Proxy` 是**不可 polyfill 的**，只能在支持 ES6 的环境使用。
- Proxy 性能开销稍高，不建议对大量数据长链式访问场景使用。

# Reflect

> `Reflect` 是 ES6 引入的一个内置对象，提供了**操作对象的低级方法集合**，和 `Object` 类似，但更偏向底层行为控制。

它的目标是：

- **统一语言内部方法的调用方式**
- **作为 Proxy trap 的默认处理函数**
- **避免破坏原始行为或重复实现底层逻辑**

✅ 常用 API 及说明

| Reflect 方法                       | 等价于                                 | 用途说明                   |
| ---------------------------------- | -------------------------------------- | :------------------------- |
| `Reflect.get(obj, key)`            | `obj[key]`                             | 安全地读取属性             |
| `Reflect.set(obj, key, val)`       | `obj[key] = val`                       | 安全地设置属性，返回布尔值 |
| `Reflect.has(obj, key)`            | `key in obj`                           | 判断属性是否存在           |
| `Reflect.deleteProperty(obj, key)` | `delete obj[key]`                      | 删除属性                   |
| `Reflect.ownKeys(obj)`             | `Object.getOwnPropertyNames + Symbols` | 获取所有键                 |
| `Reflect.defineProperty`           | 类似 `Object.defineProperty`           | 定义属性                   |
| `Reflect.construct()`              | 构造函数执行 `new` 行为                | 用于 Proxy 构造器拦截      |
| `Reflect.apply()`                  | 执行函数（类似 `fn.apply`）            | 用于函数调用代理           |

# Object.is和===什么区别

`Object.is` 和 `===`（严格相等）在大多数情况下表现一致，但它们在**少数边界情况**有**显著不同**。

| 情况           | `===` 结果 | `Object.is` 结果 | 说明                           |
| -------------- | ---------- | ---------------- | ------------------------------ |
| `+0` 和 `-0`   | `true`     | `false`          | `Object.is` 区分 +0 和 -0      |
| `NaN` 和 `NaN` | `false`    | `true`           | `Object.is` 能判断 NaN === NaN |

# 浏览器的进程与线程

浏览器是一个多进程、多线程的复杂软件系统，为了提升**性能、稳定性、安全性**，它把不同模块分散在多个进程和线程中。我们可以从以下几个方面来详细介绍：

------

## 一、浏览器的主要**进程**

现代浏览器（如 Chrome）采用多进程架构，以下是主要的进程类型：

| 进程名                              | 描述                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| **浏览器主进程（Browser Process）** | 唯一一个，负责协调、界面、资源管理、网络请求、页面调度等。也称UI进程。 |
| **渲染进程（Renderer Process）**    | 每个 Tab 页面一个或多个，负责 HTML、CSS、JS 的解析和渲染。   |
| **GPU 进程**                        | 负责图形相关任务，处理合成图层、加速渲染。                   |
| **网络进程（Network Process）**     | 独立出来负责网络请求、缓存、Cookie、HTTP 等。                |
| **插件进程（已过时）**              | 老版本用来运行 Flash 等插件，现在已基本废弃。                |
| **扩展进程**                        | 运行浏览器扩展程序，如 Chrome 插件。                         |

------

## 二、渲染进程内部的主要线程

每个**渲染进程**本身是多线程的，包含以下关键线程：

| 线程                                         | 职责                                                         |
| -------------------------------------------- | ------------------------------------------------------------ |
| **主线程（UI/Main Thread）**                 | 执行 JS、解析 HTML/CSS、构建 DOM、处理事件循环、调度任务等。是 Web 开发中最核心的线程。 |
| **渲染线程（合成线程 / Compositor Thread）** | 负责图层合成、绘制前准备，用于页面滚动、合成动画等，独立于主线程。 |
| **JS 引擎线程（V8线程）**                    | V8 JavaScript 引擎在主线程中运行，执行 JS 脚本。             |
| **样式计算线程 / Layout Thread**             | 和主线程配合，计算 CSS、布局位置等，通常是主线程处理。       |
| **工作线程（Web Worker）**                   | JS 可创建的线程，执行耗时操作，避免阻塞主线程（但不能访问 DOM）。 |
| **定时器线程（Timer Thread）**               | 管理 `setTimeout`、`setInterval` 的回调队列与触发。          |
| **事件触发线程**                             | 来自网络、输入、用户操作等事件从这里进入主线程的事件循环。   |
| **线程池（例如 Fetch、Promise、Web APIs）**  | 浏览器内部线程池处理异步任务，如网络请求、异步 I/O。         |

------

## 三、渲染流程：线程参与关系

页面加载和渲染过程中，各个线程协作如下：

```plaintext
1. HTML/CSS 解析（主线程）
2. 构建 DOM/CSSOM（主线程）
3. JS 执行、修改 DOM（主线程 / V8）
4. 样式计算、布局（主线程）
5. 绘制图层（主线程）
6. 图层合成（合成线程）
7. GPU 加速绘制（GPU 进程）
```

------

## 四、异步事件和线程关系

浏览器的异步机制（Event Loop）涉及多个线程：

### 1. 主线程中的 Event Loop

主线程通过事件循环机制管理执行任务队列：

- **宏任务（Macro Task）**：如 `setTimeout`、`setInterval`、`requestAnimationFrame`、UI 渲染、事件回调。
- **微任务（Micro Task）**：如 `Promise.then`、`MutationObserver`、queueMicrotask。

### 2. 其他线程配合

| 异步机制                   | 线程支持                                      | 说明                       |
| -------------------------- | --------------------------------------------- | -------------------------- |
| `setTimeout`/`setInterval` | **定时器线程** 触发事件回调 -> 传入主线程执行 |                            |
| `fetch`、XHR               | **网络线程** 发起请求，完成后触发事件回调     |                            |
| `Web Worker`               | 独立线程                                      | 不能访问 DOM，适合密集计算 |
| `requestIdleCallback`      | 主线程空闲时调度执行                          |                            |
| `requestAnimationFrame`    | 合成线程 + 主线程                             | 下次重绘前执行，用于动画   |

------

## 五、总结图示：进程与线程关系（简化）

```
浏览器进程（UI）
├── 负责地址栏、书签栏、前进/后退、Tab 管理
│
├── 启动渲染进程（每个页面一个）
│   ├── 主线程（HTML/CSS/JS 执行）
│   ├── JS 引擎线程（V8）
│   ├── 事件线程（监听输入/鼠标）
│   ├── 合成线程（绘制图层）
│   ├── Timer 线程（定时器）
│   ├── Worker线程（如 Web Worker）
│
├── 网络进程（独立）
│   └── 管理 HTTP 请求，缓存，Cookie
│
└── GPU 进程（独立）
    └── 图形绘制、硬件加速
```

------

## 六、面试总结记忆建议

- “一个主进程 + 多个渲染进程 + 网络/GPU等辅助进程”。
- 主线程负责执行 JS、构建 DOM。
- 合成线程、GPU 分担 UI 绘制压力。
- Web Worker 和主线程通信，用于并行计算。
- 定时器、网络等事件由后台线程触发，**通过事件循环传入主线程执行回调**。

## 七、主线程重点

```css
🧠 渲染进程（Renderer Process）
├── 👨‍💻 主线程（Main Thread）
│   ├── 💡 包含 V8 JavaScript 引擎（JS 执行）
│   ├── 🧱 负责 HTML/CSS 解析，构建 DOM/CSSOM
│   ├── 🧮 计算样式、布局
│   ├── 🔄 事件循环、处理回调
│
├── 🎨 合成线程（Compositor Thread）
│   └── 合成图层，准备 GPU 绘制
│
├── 📦 Web Worker 线程（可选）
├── 🕒 定时器线程（可选）
```

**主线程 = 包含了 V8 JS 引擎的 HTML 渲染调度线程**

V8 JavaScript 引擎线程（JS 引擎线程）:

- 实际上并不是单独的线程，而是**运行在主线程中的 JavaScript 引擎**。
- Chrome 使用 V8 引擎，在主线程内执行 JS 代码。
- 所以你看到 JS 运行慢、阻塞页面，其实就是**主线程被 JS 占用**。

# requestIdleCallback是什么

`requestIdleCallback` 是浏览器提供的一个 API，用于 **在浏览器主线程空闲时执行非关键任务**，不会阻塞页面渲染。它适合处理那些**可延迟执行、非紧急的任务**，比如预加载数据、日志上报、懒初始化等。

------

### ✅ 一句话理解

> `requestIdleCallback` 让你“等浏览器空了再说”，不会影响用户交互和渲染性能。

------

### 📦 基本用法

```js
requestIdleCallback((deadline) => {
  while (deadline.timeRemaining() > 0 && tasks.length > 0) {
    doSomeTask(tasks.shift());
  }
});
```

------

### 参数解释

回调函数接收一个 `IdleDeadline` 对象，包含：

| 属性/方法         | 说明                                          |
| ----------------- | --------------------------------------------- |
| `timeRemaining()` | 返回当前空闲时间的剩余毫秒数（一般最多 50ms） |
| `didTimeout`      | 如果设置了 `timeout` 并超时了，则为 `true`    |

------

### 🧪 示例：空闲时加载图片

```js
requestIdleCallback((deadline) => {
  while (deadline.timeRemaining() > 0 && imageQueue.length > 0) {
    const img = new Image();
    img.src = imageQueue.shift();
    document.body.appendChild(img);
  }
});
```

------

### ⏱ 可选参数：`timeout`

```js
requestIdleCallback(callback, { timeout: 2000 });
```

> 如果在 2 秒内始终没有空闲时间，它也会强制执行。

------

### 🔁 对比 `setTimeout`

| 特点             | `requestIdleCallback`  | `setTimeout`         |
| ---------------- | ---------------------- | -------------------- |
| 触发时机         | 浏览器空闲时           | 到时间立即执行       |
| 是否影响渲染     | ❌ 尽量不影响页面流畅性 | ✅ 可能阻塞渲染或交互 |
| 是否保证按时执行 | ❌ 不保证               | ✅ 定时后立即执行     |

------

### ⚠️ 浏览器兼容性

- 支持度：仅现代浏览器（Chrome、Edge 支持，**Safari 不支持**，部分 Firefox 支持）
- 可以用 polyfill 替代：

```js
window.requestIdleCallback = window.requestIdleCallback || function (cb) {
  return setTimeout(() => {
    const start = Date.now();
    cb({
      didTimeout: false,
      timeRemaining: () => Math.max(0, 50 - (Date.now() - start))
    });
  }, 1);
};
```

------

### 🔧 实际应用场景

- ✅ 懒加载非首屏图片
- ✅ 首屏渲染后上报日志或打点
- ✅ 低优先级的缓存预热
- ✅ 富文本编辑器初始化大量按钮或模块
