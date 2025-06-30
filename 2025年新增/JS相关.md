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
