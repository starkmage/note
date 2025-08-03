# Grid布局

https://yuanbao.tencent.com/chat/naQivTmsDa/269418ae-a475-40e6-a140-58acc482bdbe?projectId=2f13020843e3426ab5f1fc84dfe2aa87

# Grid Box和Flex Box区别

| 特性       | Flexbox                      | Grid                       |
| ---------- | ---------------------------- | -------------------------- |
| 📏 布局维度 | 一维（行或列）               | 二维（行和列）             |
| 📐 控制方式 | 内容驱动布局（内容决定排布） | 布局驱动内容（先设好格子） |
| 📦 主用途   | 元素按顺序自动排列           | 复杂网格或卡片区域排布     |
| 🧠 适合场景 | 导航栏、按钮组、横向滚动区域 | 页面整体布局、复杂卡片列表 |
| 🧲 对齐能力 | 强，方向灵活                 | 更强，可精确控制网格区域   |
| 🚦 布局流   | 主轴 + 交叉轴                | 明确的网格线和区域         |

# CSS in JS

https://yuanbao.tencent.com/chat/naQivTmsDa/4add0a54-18d7-4669-b379-072ac128b176?projectId=2f13020843e3426ab5f1fc84dfe2aa87

# DOM大小改变会引起重排还是重绘

------

✅ 简明回答：

> **DOM 大小（如 width、height、padding、border）改变，会引起 重排（reflow）和 重绘（repaint）\*。**

------

📘 定义回顾：

| 名称                | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| **重排（Reflow）**  | 也叫布局（layout），浏览器需要重新计算元素的位置和大小       |
| **重绘（Repaint）** | 在位置和大小确定后，浏览器需要重新绘制像素（颜色、边框、阴影等） |

------

✅ DOM 大小变化为什么会触发重排？

因为大小变化（比如：

- `width`
- `height`
- `padding`
- `margin`
- `border`

）都会影响到该元素 **自身** 以及可能影响 **其周围兄弟元素和父元素的布局关系**。

👉 所以，浏览器必须：

1. 重新计算布局（重排）
2. 然后更新屏幕上的像素（重绘）

------

🧠 举例说明：

```js
element.style.width = '300px';
```

这会：

- ✅ 触发重排（元素尺寸变化，布局系统需重新计算）
- ✅ 触发重绘（像素变化）

------

📌 对比：只触发重绘的属性有哪些？

以下操作 **只触发重绘，不会引发重排**：

- `color`
- `background-color`
- `visibility`
- `box-shadow`
- `outline`
- `border-color`

```js
element.style.backgroundColor = 'red'; // ✅ 只触发重绘，不影响布局
```

------

📦 总结表格

| 操作 / 属性                             | 是否触发重排     | 是否触发重绘   |
| --------------------------------------- | ---------------- | -------------- |
| 改变宽高、padding、margin、border       | ✅ 是             | ✅ 是           |
| 改变颜色、阴影                          | ❌ 否             | ✅ 是           |
| 改变 display、position 等影响布局的属性 | ✅ 是             | ✅ 是           |
| 改变 visibility                         | ❌ 否             | ✅ 是           |
| 改变 transform / opacity                | ❌ 否（GPU 加速） | ✅ 是（合成层） |

------

⚠️ 性能建议

- **频繁操作 DOM 尺寸（引起重排）会很慢**，比如设置 1000 个元素的高度。
- 可以用：
  - `requestAnimationFrame` 合并操作
  - `classList` 批量切换样式
  - `will-change` 或 transform、opacity 等复合层优化

