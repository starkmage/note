# React Fiber

https://yuanbao.tencent.com/chat/naQivTmsDa/e8d99773-3712-4504-9710-e55e1f7e6fc1?projectId=2f13020843e3426ab5f1fc84dfe2aa87

# React 18的并发模式

https://yuanbao.tencent.com/chat/naQivTmsDa/cdec852a-906f-4def-8c0b-1f504cc221e7?projectId=2f13020843e3426ab5f1fc84dfe2aa87

## 1. **使用 `createRoot` 替代 `ReactDOM.render`**

这是启用并发能力的**第一步**（React 18 的默认行为）：

```
import { createRoot } from 'react-dom/client';

// 替换旧的 ReactDOM.render
const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

- 作用：
  - 启用**可中断渲染**（高优先级更新可打断低优先级渲染）。
  - 但**不会自动启用时间切片或全局优先级调度**（需配合并发特性使用）。

------

## 2. **通过并发特性局部启用并发行为**

React 18 提供了以下 API 来**手动标记优先级**，实现并发效果：

#### ✅ **`startTransition`**

标记非紧急更新（如搜索筛选），允许被高优先级操作（如用户输入）打断：

```
import { startTransition } from 'react';

function handleSearch(query) {
  // 高优先级更新：立即执行
  setInputValue(query);

  // 低优先级更新：允许被打断
  startTransition(() => {
    setSearchResults(filterResults(query));
  });
}
```

#### ✅ **`useDeferredValue`**

延迟派生值的更新（如输入框的自动补全）：

```
import { useDeferredValue } from 'react';

function SearchResults() {
  const query = useDeferredValue(inputValue);
  // query 会在输入稳定后更新，减少渲染压力
}
```

`useDeferredValue`的更新时机和 "稳定" 判断机制是 React 调度系统的一部分，它主要依赖于 React 的 Concurrent Mode（并发模式）和优先级调度机制。以下是它的工作原理：

1. **更新时机的判断机制**

- **优先级调度**：React 会将更新分为不同的优先级。用户输入（如键盘输入）是高优先级更新，而 `useDeferredValue`的更新是低优先级更新。
- **渲染间隙检测**：React 会在浏览器空闲时（通过 `requestIdleCallback`类似的机制）检查是否有待处理的低优先级更新。
- **帧时间预算**：React 会计算当前帧剩余的时间，如果时间充足就处理低优先级更新。

2. **如何确定 inputValue "稳定"了**

- **时间阈值**：React 内部有一个默认的时间阈值（通常是几百毫秒），如果在这个时间内没有新的高优先级更新进来，就认为值已经 "稳定"。
- **批量更新**：React 会批量处理快速连续的更新，当更新停止一段时间后触发 deferred value 的更新。
- **中断机制**：如果在处理低优先级更新时又有新的高优先级更新进来，React 会中断当前渲染，优先处理高优先级更新。

3. **实际工作流程示例**

1. 用户快速输入 "hello"：
   - 输入 "h" → 高优先级更新 `query="h"`
   - `deferredQuery`仍为 ""（旧值）
2. 短暂停顿（比如 300ms 内没有新输入）：
   - React 认为输入 "稳定"
   - 调度低优先级更新，`deferredQuery`更新为 "h"
3. 用户继续快速输入 "ello"：
   - 每次按键都触发高优先级更新
   - `deferredQuery`可能保持在 "h" 直到输入停止
4. 最终停止输入：
   - 经过阈值时间后
   - `deferredQuery`更新为 "hello"

|   特性   |    useDeferredValue    |    防抖(debounce)    |
| :------: | :--------------------: | :------------------: |
| 触发机制 | React 调度系统自动管理 | 需要手动设置延迟时间 |
|  响应性  |      保持界面响应      |     完全阻止更新     |
| 适用场景 |        渲染优化        |     事件频率控制     |
|  优先级  |     内置优先级系统     |     无优先级概念     |

`useDeferredValue`的核心优势在于它能让界面保持响应，同时又不完全阻止更新，而是在系统资源允许时自动处理更新。

#### ✅ **`Suspense`**

配合懒加载或数据获取，实现更流畅的异步渲染：

```
<Suspense fallback={<Spinner />}>
  <LazyComponent />
</Suspense>
```

**不开启并发模式（legacy 模式）时，React 仍然有“优先级调度系统”，但它只用于“调度任务”何时执行，而不是“可中断渲染”**。也就是说，**优先级仍然存在，但调度能力是有限的**。

## 📌 更深入理解（关键点）

### ✅ 有调度系统（`scheduler`）

- React 18 **无论是否开启并发模式，都会用内部的 `scheduler` 模块**。
- 这个模块有五个优先级（immediate, user-blocking, normal, low, idle）。
- 例如你使用 `startTransition`，React 会把它调度为较低优先级任务。

### ❌ 没有真正的“并发能力”

如果你没有使用 `createRoot()`，而是用了 `ReactDOM.render()`（即 legacy 模式）：

- React 会**按优先级“排队执行任务”**，但一旦任务开始执行，**不能中断、暂停或打断它**。
- 所以即使低优先级任务排在后面，它一旦开始执行，仍会一次性完成（“同步渲染”），用户依旧会感觉卡顿。

------

## 📊 类比：非并发 vs 并发模式的优先级调度能力

| 能力                    | 非并发模式         | 并发模式 (Concurrent Mode) |
| ----------------------- | ------------------ | -------------------------- |
| 使用 scheduler 模块     | ✅ 有               | ✅ 有                       |
| setState 支持优先级分类 | ✅ 有               | ✅ 有                       |
| startTransition 起作用  | ✅ 起作用但不能中断 | ✅ 起作用且可以中断         |
| 渲染可被中断（yield）   | ❌ 不支持           | ✅ 支持                     |
| Suspense 异步边界       | 🚫 降级为同步       | ✅ 异步、流式渲染           |

# React Hook的原理

https://yuanbao.tencent.com/chat/naQivTmsDa/e093df06-72e2-4f39-8050-6ccadbbb0c81?projectId=2f13020843e3426ab5f1fc84dfe2aa87

# React Hooks 在 Fiber 架构中的存储机制

------

### 1. **Hooks 的存储位置**

所有 Hooks（包括 `useState`、`useMemo`、`useEffect` 等）的**状态**都存储在 Fiber 节点的 `memoizedState` 属性上，但具体结构有所不同：

- 函数组件 Fiber：`fiber.memoizedState`是一个单向链表，每个节点对应一个 Hook（按调用顺序存储）。

  - 每个 Hook 节点的结构如下：

    ```ts
    type Hook = {
      memoizedState: any,      // 当前状态（如 state、effect 对象等）
      baseState: any,         // 基础状态（用于更新计算）
      queue: UpdateQueue<any>, // 更新队列（用于 useState/useReducer）
      next: Hook | null,      // 指向下一个 Hook
    };
    ```

------

### 2. **不同 Hook 的具体存储**

#### ✅ `useState` / `useReducer`

- **`memoizedState`**：存储当前的状态值（如 `count`）。
- **`queue`**：存储待处理的更新（`dispatch` 触发的动作）。

#### ✅ `useMemo` / `useCallback`

- **`memoizedState`**：存储缓存的值或函数（如 `[value, deps]`）。

#### ✅ `useEffect` / `useLayoutEffect`

- `memoizedState`：存储一个 effect 对象，结构如下：

  ```ts
  type Effect = {
    tag: number,           // 标识 effect 类型（如 PassiveEffect、LayoutEffect）
    create: () => (() => void) | void, // 副作用函数
    destroy: (() => void) | void,      // 清理函数
    deps: Array<any> | null,           // 依赖数组
    next: Effect | null,   // 指向下一个 effect
  };
  ```

- **Effect 链表**：所有组件的 effect 会额外保存在 Fiber 的 `updateQueue` 属性中（一个环形链表），用于 React 的调度和提交阶段处理。

------

### 3. **Fiber 节点的关键属性**

```
type Fiber = {
  memoizedState: Hook | null,  // 函数组件的 Hook 链表
  updateQueue: UpdateQueue | null, // 用于存储 effect 链表（useEffect）
  // ...其他属性（如 type, child, sibling 等）
};
```

------

### 4. **你的说法是否正确？**

- 部分正确：
  - ✅ `useState`、`useMemo` 的状态确实存储在 `fiber.memoizedState` 的 Hook 链表中。
  - ✅ `useEffect` 的 effect 对象也**间接**通过 `memoizedState` 关联（但实际执行时会被收集到 `updateQueue`）。
- 需要修正：
  - `useEffect` 的 effect 对象不仅存在于 Hook 节点的 `memoizedState` 中，还会被添加到 Fiber 的 `updateQueue` 链表，以便 React 统一调度。

------

### 5. **示例流程**

1. 组件首次渲染：
   - 每个 Hook 调用会创建一个 Hook 节点，链接到 `fiber.memoizedState`。
   - `useEffect` 的 effect 对象会被添加到 `fiber.updateQueue`。
2. 更新阶段：
   - React 遍历 `fiber.memoizedState` 链表，复用或更新 Hook。
   - 比较 `deps` 决定是否重新执行 effect。

### 6.**不管组件定义了多少 `useEffect`，`memoizedState` 只会有一个 Effect 对象吗**

❌ 不正确

- 每个 `useEffect` 都会在 `fiber.memoizedState` 链表中创建一个 **独立的 Hook 节点**，每个节点的 `memoizedState` 存储自己的 Effect 对象。
- 但 React 会将这些 Effect 对象 **额外链接到 `fiber.updateQueue`** 中，形成环形链表供调度使用。

### **7. 为什么这样设计？**

1. **Hook 独立性**：每个 Hook 的状态（如 `useState` 的 state、`useEffect` 的 deps）需要隔离，避免互相污染。
2. **批量调度效率**：通过 `updateQueue` 链表，React 可以在提交阶段（commit phase）高效遍历并执行所有 Effect。

------

### 总结

- **Hooks 状态**：统一通过 `fiber.memoizedState` 链表管理。
- **Effect 处理**：额外通过 `fiber.updateQueue` 调度，确保生命周期正确执行。

可以通过 React 源码（如 `ReactFiberHooks.js`）进一步验证这一机制。

# React Context的原理

https://yuanbao.tencent.com/chat/naQivTmsDa/8bd0e448-3ed9-45b2-b26b-0e6fee84e761?projectId=2f13020843e3426ab5f1fc84dfe2aa87

# Redux的原理

手写题

# 在React中使用Suspense动态加载组件

Suspense是React的一个特性，它允许你在组件加载时显示一个备用内容（fallback），直到组件准备好渲染。这在动态加载组件（代码分割）时特别有用。

### 基本用法

```jsx
import React, { Suspense } from 'react';

// 使用React.lazy动态导入组件
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}
```

### 关键点说明

1. **React.lazy()**：用于动态导入组件，它返回一个Promise
2. **Suspense组件**：包裹懒加载的组件
3. **fallback属性**：在组件加载期间显示的备用内容

### 多个懒加载组件

```jsx
<Suspense fallback={<div>Loading...</div>}>
  <OtherComponent />
  <AnotherComponent />
</Suspense>
```

### 嵌套Suspense

```jsx
<Suspense fallback={<div>Loading outer...</div>}>
  <Component1 />
  <Suspense fallback={<div>Loading inner...</div>}>
    <Component2 />
  </Suspense>
</Suspense>
```

1. Suspense目前只支持React.lazy和React 18的并发特性
2. 确保fallback内容不会导致布局跳动（CLS）
3. 对于复杂应用，考虑将Suspense放在路由级别

通过合理使用Suspense，你可以显著改善大型React应用的加载体验。

## React HOC 面试介绍

### 1. 什么是 HOC？

（Higher-Order Component，高阶组件）

- **定义**：HOC 是一个函数，接受一个组件作为参数，返回一个新的增强后的组件。
- 它是 React 组件复用逻辑的一种高级技巧，属于“组件组合”的模式。
- 公式表达：`const EnhancedComponent = higherOrderComponent(WrappedComponent)`

### 2. HOC 的作用

- 代码复用、抽象状态和逻辑，比如权限控制、数据获取、UI 状态管理、性能优化等。
- 避免重复代码，提高组件的复用性和维护性。

### 3. HOC 的核心原理

- 接收一个原始组件，返回一个包装组件。
- 包装组件在渲染时，会渲染原始组件并传入增强后的 props 或状态。
- 通过闭包和组合，实现增强功能。

### 4. HOC 简单示例

```jsx
function withLoading(WrappedComponent) {
  return function WithLoadingComponent({ isLoading, ...props }) {
    if (isLoading) {
      return <div>Loading...</div>;
    }
    return <WrappedComponent {...props} />;
  };
}
```

### 5. HOC 使用场景

- 权限校验：包装组件根据权限控制渲染内容。
- 数据获取：包装组件在挂载时请求数据，传递给原始组件。
- 事件处理：统一处理某些事件逻辑。
- 性能优化：例如缓存渲染结果。

### 6. 面试常见问题

- **HOC 与 Render Props 区别？**
  - HOC 是通过函数接受组件并返回新组件，Render Props 是通过一个函数作为子组件提供渲染内容。
- **HOC 如何避免命名冲突？**
  - 使用 `hoist-non-react-statics` 库拷贝静态属性，避免丢失原组件静态方法。
- **HOC 如何传递 refs？**
  - 需要用 `React.forwardRef` 来转发 ref，否则 ref 指向的是包装组件而非被包裹组件。

### 7. HOC 的缺点

- 组件树嵌套层级增加（“Wrapper Hell”）。
- 调试时组件层级复杂。
- 不支持使用 Hook（HOC 本身是函数，Hook 只能在函数组件里使用）。

# React为什么分成react和react-dom两个库

🎯 简答：

> React 被拆成 `react` 和 `react-dom` 两个库，是为了**实现平台无关性**和**架构解耦** —— 也就是：
>  **“核心逻辑与平台渲染解耦”**。

------

🧠 React 与 ReactDOM 的职责划分

| 库              | 作用                     | 包含内容                                                     |
| --------------- | ------------------------ | ------------------------------------------------------------ |
| **`react`**     | **核心库**               | 提供组件系统、Hooks、虚拟 DOM、调度、context、生命周期等     |
| **`react-dom`** | **平台绑定（DOM 绑定）** | 提供将虚拟 DOM 渲染到真实 DOM 的功能（如 `ReactDOM.render`、事件系统、DOM diff） |

也就是说：

- `react`：只关心“怎么描述 UI 组件、怎么构建组件树”
- `react-dom`：关心“怎么把组件树挂到浏览器 DOM 上”

------

**🧩 更深理解：为什么这么做？**

✅ 1. **实现跨平台（平台无关性）**

React 并不仅仅运行在浏览器 DOM 中，它还可以运行在其他环境，比如：

| 平台                   | 对应库                   |
| ---------------------- | ------------------------ |
| 浏览器                 | `react-dom`              |
| 移动端（React Native） | `react-native`           |
| 服务端渲染             | `react-dom/server`       |
| 命令行终端（Ink）      | `ink`                    |
| 画布（canvas）         | `react-canvas`（第三方） |

> 所有这些平台都依赖同一个 `react` 核心库，复用虚拟 DOM 构建逻辑，但使用不同的 “渲染器” 来最终把 UI 渲染出来。

------

✅ 2. **职责分离，更易维护与拓展**

- 核心库负责调度、组件逻辑、Hooks、上下文
- DOM 库负责：
  - 渲染虚拟 DOM 到 HTML 元素
  - 处理事件系统（如合成事件）
  - 管理 DOM 节点生命周期

这样当你只想测试组件逻辑时，可以只引入 `react`，而不用管 DOM。

------

✅ 3. **为自定义渲染器开放（React Reconciler）**

React 团队甚至将协调器（Reconciler）也抽成独立模块，使你可以：

- 自定义渲染器（写一个 `react-vue`, `react-skia`）
- 用 `react-reconciler` 构建一个自定义平台（比如你想让 React 渲染成 Excel 表格、3D 场景等）

------

**🛠 结构图总结：**

```
     +-----------------+               +----------------+
     |     react       |     <------   | 你的组件逻辑   |
     | (核心逻辑层)     |               +----------------+
     |  - JSX 构建       |
     |  - Fiber协调      |
     |  - Hooks机制      |
     +-----------------+
              ↓
     +---------------------+
     |  渲染器 (Renderer)   | --> 例如：react-dom / react-native / ink
     +---------------------+
              ↓
     +---------------------+
     |     具体平台         |
     |  (DOM / Native / ...)|
     +---------------------+
```

------

✅ 总结一句话：

> React 拆成 `react` 和 `react-dom`，是为了**让 UI 构建逻辑与平台渲染逻辑解耦**，实现跨平台能力、便于维护、支持自定义渲染器 —— 这是 React 架构优雅的体现。

# React一次渲染时是怎么判断要进入到提交阶段了

React 在一次更新（render）中，**判断是否从“协调阶段”（Reconciliation）进入“提交阶段”（Commit）**的核心，是看：

> 🔍 是否生成了一个“有变更的 Fiber 树” —— 即 Work-in-Progress Fiber 树中是否包含 effect（副作用）标记。

------

🧠 背景知识

React 的更新过程可分为两个主要阶段：

| 阶段                     | 特点                                                         |
| ------------------------ | ------------------------------------------------------------ |
| 协调阶段（Render Phase） | 构建新 Fiber 树，进行 diff，标记变化（effectTag），是**可中断**的 |
| 提交阶段（Commit Phase） | 应用所有变更到真实 DOM，调用生命周期等，是**不可中断**的     |

------

✅ 判断是否进入提交阶段的关键逻辑

React 内部在构建完 Work-in-Progress Fiber 树后，会检查是否存在副作用（即变化）。如果有，就进入提交阶段。

👇 具体表现：

1. **在协调阶段：**
   - React 会遍历整个 Fiber 树，执行组件的 render 函数，构建新的子树结构。
   - 在构建过程中，每个节点如果发生了变化（比如 DOM 更新、ref 变化、副作用 hook 等），都会被打上一个 effect 标记（如 `Placement`, `Update`, `Deletion` 等）。
2. **协调结束后：**
   - 如果根节点 `root.finishedWork` 上有副作用（`root.finishedWork.flags !== NoFlags || root.finishedWork.subtreeFlags !== NoFlags`），则进入提交阶段。
   - 如果没有任何副作用，React 会跳过提交阶段（即不触发 DOM 更新，不调用副作用 hook）。

**在React 18+的并发模式下，协调可能被中断多次，但只有在整个Fiber树的副作用收集完成后才会真正进入提交阶段。提交阶段本身是同步不可中断的，这是React保证UI一致性的关键设计。**

# 如果一个回调函数任务很多，有几千行逻辑要执行

如果一个回调函数（比如在 React 中的某个事件处理函数、Effect 回调或更新调度函数）**本身逻辑很长、有几千行代码需要执行**，React 本身是**不会自动拆分这段任务的**，那么它会产生以下几个影响和应对策略：

------

🧨 问题：长任务的影响

1. **阻塞主线程**

- JS 是单线程的，这几千行代码执行期间：
  - 浏览器不能响应用户交互（如点击、滚动）。
  - 页面会“卡住”或掉帧。
  - UI 更新延迟，React 的调度优先级机制也被阻断。

2. **打断调度机制**

- 在 React 18 的并发模式下，协调阶段可以被打断（利用时间分片机制 `Scheduler` + `MessageChannel`）。
- 但如果你在某个任务中执行了巨量逻辑，**React 的调度也无法插入**，造成帧丢失或“长任务”（> 50ms）。

**✅ 最佳实践**

1. **优先拆分长任务**（`setTimeout`、`queueMicrotask`）。
2. **纯计算任务 → Web Worker**。
3. **React 18+ → `useTransition`或 `useDeferredValue`**。
4. **避免同步长任务**，否则 UI 会卡顿。

------

**结论**

如果回调函数有**几千行同步逻辑**，React **不会自动优化**，必须手动拆分任务或使用 Web Workers，否则会阻塞渲染。React 18 的并发模式（`useTransition`）能部分缓解问题，但最佳方案仍是**减少主线程计算量**。

# React的事件绑定机制

## 一道题目

```react
const App = () => {
  const handleClick = () => {
    console.log('react写的')
  }

  return <div>
    <div id='a' onClick={handleClick}>Button</div>
  </div>
}
```

这个Button的dom，在浏览器控制台获取通过选择器获取后，通过addEventListener给他加上一个新的click回调，比如console.log('浏览器改的')，点击Button会发生什么.

```js
const element = document.querySelector('#a'); 
element.addEventListener('click', () => {  console.log('浏览器改的'); });
```

**答案：**

```
浏览器改的
react写的
```

**原因：**

React 里 `<div onClick={handleClick}>` 实际上会**通过事件委托机制**，将 click 事件绑定在根容器（`#root`）上，而不是直接绑定在这个 `div` 上。

🔁 点击时发生了什么？

当你点击 `<div>Button</div>` 时：

1. **浏览器原生事件捕获/冒泡流程开始**
2. 点击事件冒泡到这个元素时：
   - 会先触发这个 `div` 上用 `addEventListener` 加的监听器（`console.log('浏览器改的')`）
3. 然后事件继续冒泡，传到 React 挂载的容器元素（如 `#root`）上：
   - React 的合成事件系统捕捉到事件，在对应组件上查找是否注册了 `onClick` 事件，有就触发 `handleClick`，打印 `'react写的'`

------

🧠 简单说：

- 原生事件监听器是直接绑定在 DOM 上的，会优先触发
- React 的事件绑定是“委托”在 `root` 上，是**后触发的**

## React 是如何实现事件委托和合成事件系统

React 的事件系统是它最核心的机制之一，它使用**事件委托（Event Delegation）+ 合成事件（SyntheticEvent）\**实现了一套\**跨浏览器一致、可中断、优先级可控**的事件系统。

下面我会从整体到细节为你展开讲解：

------

🧭 一句话总结：

> React 并不是把 `onClick` 直接绑定到每个 DOM 元素上，而是统一绑定到容器（比如 `#root`）上，通过事件冒泡，捕获事件，再派发给对应组件的回调函数。这就是**事件委托**。
>  同时 React 会用一套自己的 `SyntheticEvent` 包装原生事件，这就是**合成事件系统**。

------

### ✅ React 事件系统架构全景

🌳 事件委托模型：

| 特性     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| 委托对象 | React 会在 `root` 容器上只绑定一次事件监听器（如 `click`, `input`, `keydown` 等） |
| 原理     | 依赖浏览器的事件冒泡模型                                     |
| 优势     | 减少事件绑定数量，提高性能；支持事件优先级、调度中断等高级控制 |

🎭 合成事件系统（SyntheticEvent）：

| 特性         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| 包装原生事件 | React 创建一个 `SyntheticEvent` 实例，它包装了浏览器原生事件对象 |
| 跨浏览器兼容 | 提供统一的事件属性和方法（如 `event.target`, `stopPropagation`, `preventDefault`） |
| 可回收       | React 会复用事件对象，提高性能（通过“事件池”机制）           |

------

🔄 生命周期：一次事件是如何在 React 中被处理的？

以下是点击事件的处理流程：

```jsx
<div onClick={handleClick}>Click Me</div>
```

步骤如下：

------

1️⃣ React 初始化时绑定全局事件监听器

```js
// ReactDOM 会在 root container 上注册监听器
container.addEventListener('click', dispatchEvent, false)
```

> 这个监听器是一次性的，所有 `onClick` 都通过这一个入口进来。

------

2️⃣ 用户点击 DOM 元素 → 浏览器触发原生事件 → 冒泡到容器上

- 浏览器触发原生 `click` 事件
- 事件冒泡到 `#root` 容器
- 被 React 的 `dispatchEvent()` 捕获

------

3️⃣ React 执行 dispatchEvent → 开始事件派发流程

React 内部主要函数是：

```ts
dispatchEventForPluginEventSystem(
  domEventName,      // 如 'click'
  eventSystemFlags,  // 标记 phase 类型（冒泡 / 捕获）
  nativeEvent,       // 原生事件对象
  target             // 事件目标 DOM
)
```

------

4️⃣ React 从事件目标 Fiber 节点向上“冒泡”寻找事件回调

> React 事件系统会构建一条 Fiber 节点链，从目标 Fiber 一直向上走到 root，查找是否有 `onClick`。

```ts
accumulateTwoPhaseListeners(targetFiber, 'onClick') // 收集冒泡和捕获监听器
```

------

5️⃣ 创建 SyntheticEvent 并派发回调

```ts
const syntheticEvent = createSyntheticEvent(nativeEvent)
listener(syntheticEvent)
```

> `syntheticEvent` 是合成事件对象，具有 `persist()`, `stopPropagation()`, `preventDefault()` 等方法。

------

6️⃣ 调用你的回调函数

```ts
const handleClick = (e) => {
  console.log('react写的')
  e.stopPropagation()
  e.preventDefault()
}
```

> React 会用你写的函数去调用收集到的监听器队列。

------

### 💡 React 合成事件的核心实现原理

✅ 创建 SyntheticEvent 实例：

```ts
function createSyntheticEvent(nativeEvent) {
  const syntheticEvent = {
    nativeEvent,
    currentTarget: null,
    stopPropagation: () => nativeEvent.stopPropagation(),
    preventDefault: () => nativeEvent.preventDefault(),
    // ...
  };
  return syntheticEvent;
}
```

------

✅ 为什么要用合成事件？

| 原因                | 好处                                              |
| ------------------- | ------------------------------------------------- |
| 跨浏览器兼容        | React 抹平 IE、Chrome、Firefox 的事件行为差异     |
| 支持事件池回收      | 性能优化，避免频繁创建新对象（React 17 以前启用） |
| 支持优先级调度      | 可集成到调度系统中，如高优任务中断、延迟处理      |
| 可中断 & 批处理更新 | 支持异步调度和 `flushSync` 控制更新粒度           |

------

### 🧠 React 17+ 变了什么？

- **事件绑定不再统一绑到 root 上**，而是绑定在事件目标容器上（例如你调用 `ReactDOM.render` 的那个 DOM）
- 合成事件仍保留，只是绑定更局部

### 🧠 延伸理解：React 18 中的 `createRoot`

如果你用的是 React 18：

```js
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);
```

> 那绑定事件监听器的仍然是你传入的那个 DOM —— `document.getElementById('root')`。

------

### 🎯 小结图（逻辑流程）

```
原生事件触发
       ↓
冒泡到 React 容器 DOM
       ↓
React 捕捉并执行 dispatchEvent
       ↓
构建 Fiber 冒泡路径，收集监听器
       ↓
创建 SyntheticEvent
       ↓
依次执行你的事件回调
```

------

如果你想进一步了解调度优先级、事件批处理等我也可以展开，比如：

- React 是如何处理多个事件之间的 flush
- 如何实现事件优先级（比如 Input 比 MouseMove 更重要）
- React 事件系统与 Scheduler 的集成

## React里stopPropagation和preventDefault作用和原理

这是一个非常核心的 React 事件机制问题，我们来系统解释：

------

🎯 简答版：

| 方法                | 作用                                           |
| ------------------- | ---------------------------------------------- |
| `stopPropagation()` | 阻止事件冒泡到更高层级（如父组件）             |
| `preventDefault()`  | 阻止浏览器的默认行为（比如表单提交、链接跳转） |

React 中的这两个方法，**作用与原生 DOM 是一样的**，但实现上是通过 **合成事件（SyntheticEvent）封装** 的。

------

🧠 合成事件回顾

在 React 里事件是被封装成了 `SyntheticEvent`（合成事件）的：

```js
const handleClick = (e) => {
  e.stopPropagation();
  e.preventDefault();
}
```

这个 `e` **不是原生 DOM 事件对象**，而是 React 构造的合成事件实例（`SyntheticEvent`），它会在内部调用原生事件的方法来达到相同目的。

------

🔎 详细解释和原理

### ✅ `preventDefault()`：阻止默认行为

例如：

```js
<a href="https://example.com" onClick={(e) => e.preventDefault()}>
  Click me
</a>
```

👆 阻止了 `<a>` 标签跳转页面的默认行为。

🌟 React 实现：

```js
e.preventDefault()
```

内部实际上是这样的：

```js
preventDefault() {
  this.defaultPrevented = true;
  const event = this.nativeEvent;
  if (event.preventDefault) {
    event.preventDefault();
  } else {
    event.returnValue = false; // IE fallback
  }
}
```

> 也就是说，**它调用了原生事件的 `preventDefault` 方法**，完全等效于 DOM 的行为。

------

### ✅ `stopPropagation()`：阻止冒泡传播

比如：

```js
<div onClick={() => console.log('parent')}>
  <button onClick={(e) => {
    e.stopPropagation();
    console.log('child');
  }}>Click</button>
</div>
```

点击按钮时只输出 `child`，不会输出 `parent`，因为冒泡被阻断了。

🌟 React 实现：

```js
stopPropagation() {
  const event = this.nativeEvent;
  if (event.stopPropagation) {
    event.stopPropagation();
  }
  this.isPropagationStopped = true;
}
```

> React 会记录一个标志 `isPropagationStopped`，在事件系统内部检查这个标志来决定是否继续执行父节点的回调。

------

### ⚠️ 注意：React 的 `stopPropagation` 不一定影响原生事件监听器

React 的事件系统是在冒泡阶段中**模拟自己的事件冒泡流程**，所以：

```js
// 原生监听器
document.getElementById('root').addEventListener('click', () => {
  console.log('原生事件');
});

// React 监听器
<div onClick={(e) => e.stopPropagation()}></div>
```

👆 `stopPropagation()` 只能阻止 React 的事件传播，不会阻止**原生绑定在 DOM 上的事件监听器**。

你要让原生事件也阻止，得这样做：

```js
e.nativeEvent.stopPropagation();
```

------

### ✅ 小结

| 方法                | React 封装方式                                   | 是否影响原生事件            |
| ------------------- | ------------------------------------------------ | --------------------------- |
| `preventDefault()`  | 包装并调用原生 `preventDefault()`                | ✅ 有效                      |
| `stopPropagation()` | 包装并调用原生 `stopPropagation()`，并设置标志位 | 🚫 默认不会阻止原生 DOM 事件 |
| `e.nativeEvent`     | 可以访问原生事件对象                             | ✅ 可调用原生 API            |

------

🎁 延伸：事件穿透与停止的顺序逻辑

React 合成事件会：

- 在你调用 `e.stopPropagation()` 时，停止在 React 的 Fiber 冒泡链上传播
- 但**不影响原生 `addEventListener` 的事件继续触发**（除非你调用的是 `e.nativeEvent.stopPropagation()`）

------

如果你想我画个图解释 React 如何在 Fiber 节点中模拟事件冒泡，我也可以继续展开。

## 如果是原生HTML，上面的代码输出什么

```html
<body>
  <button id="btn">Click</button>

  <script>
    const btn = document.getElementById('btn');

    btn.onclick = () => {
      console.log('DOM0 onclick');
    };

    btn.addEventListener('click', () => {
      console.log('DOM2 addEventListener');
    });


  </script>
</body>
```

代码所示的两个绑定方式，谁先被注册，谁先触发

- **内联HTML的`onclick`属性**：

  ```
  <button onclick="console.log('Inline')">Click</button>
  ```

  这类事件的触发时机等同于直接赋值 `onClick`，且会覆盖已有的 `element.onclick`，但不会影响 `addEventListener`的监听器。**内联 `onclick`总是先于 `addEventListener`触发**（无论注册顺序）

**避免混用两者**：混用可能导致执行顺序难以维护，尤其是团队协作时
