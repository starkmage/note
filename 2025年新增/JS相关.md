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
