# React Fiber

https://yuanbao.tencent.com/chat/naQivTmsDa/e8d99773-3712-4504-9710-e55e1f7e6fc1?projectId=2f13020843e3426ab5f1fc84dfe2aa87

## 关于React Fiber任务可暂停的粒度

在 React Fiber 的渲染过程中，**不同 Fiber 节点之间可以暂停**（通过时间切片和优先级调度），但**同一个 Fiber 节点内部的多个 Hook 调用是不可暂停的**。

------

**为什么 Fiber 节点之间可以暂停？**

Fiber 的增量渲染以**单个 Fiber 节点**为任务分片单位：

- **协调阶段（Reconciliation  Phase）**：

  React 按 `child → sibling → return`的顺序遍历 Fiber 树，每处理完一个 Fiber 节点后，会检查剩余时间片（默认 5ms）。若时间不足，则暂停并记录当前节点，下次恢复时继续处理下一个节点。

  **剩余时间是什么？**

  “剩余时间是指浏览器主线程在当前帧（通常为 16.6ms，60FPS）的空闲时段（通过 `requestIdleCallback`或 React 调度器模拟获取）

**关键点**：

- 暂停的边界是**完整的 Fiber 节点**，而非节点内部的子操作。

------

**🧠 为什么 Fiber 节点内部 Hook 不可暂停？**

1. **React 的调和过程（Reconciliation）**

- 调和过程中，每个组件对应一个 Fiber 节点。
- 当调和这个节点时，React 会：
  - **执行函数组件**（重新调用函数体）
  - **同步执行所有 Hooks**（`useState`, `useEffect`, etc）

2. **Hooks 是有调用顺序依赖的**

- 每次组件 render 时，Hooks 是依赖于调用顺序来读取/更新状态的（靠 `ReactCurrentDispatcher` 和 Hook 链表）。
- 如果中途暂停，会打断 Hook 链表的构建 → 状态错乱。

## React Fiber暂停任务后，是怎么知道上次的任务执行到哪里了的

“React 通过 **全局指针 `workInProgress`** 和 **Fiber 节点的链表结构** 记录进度：

1. **遍历机制**：`performUnitOfWork`按 `child → sibling → return(parent)`的顺序处理 Fiber 节点，每次更新 `workInProgress`指针。
2. **中断时**：`workInProgress`停留在最后一个未完成的节点。
3. **恢复时**：直接从 `workInProgress`继续遍历，无需重新计算。
4. **双缓冲保护**：通过 `current`和 `workInProgress`两棵树确保中断不会导致 UI 不一致。”

## **如果高优先级任务打断了低优先级渲染，React 如何恢复？**

高优先级任务会强制丢弃未完成的 `workInProgress`树，并重新从根节点开始渲染。但通过 `alternate`属性复用之前的 Fiber 节点，避免重复创建对象，保证性能

## 通过 alternate属性复用之前的 Fiber 节点，避免重复计算的原理

每个 Fiber 节点都有一个 `alternate`属性，指向 **另一个 Fiber 节点**，形成 **“镜像”关系**：

- **`current`树**：当前已渲染到 DOM 的 Fiber 树（即屏幕上显示的 UI 对应的 Fiber 结构）。
- **`workInProgress`树**：正在构建的新 Fiber 树（可能未完成，可中断）。

**关键点**：

- `current.alternate === workInProgress`
- `workInProgress.alternate === current`
- 两棵树通过 `alternate`互相引用，构成双缓冲。

“`alternate`是 Fiber 双缓冲机制的核心属性，它让 React 能在 `current`（当前 UI）和 `workInProgress`（新构建的树）之间复用 Fiber 节点：

1. **复用逻辑**：更新时优先从 `current.alternate`获取节点，仅更新变化的 `props`或 `state`，避免重新创建对象。
2. **性能优势**：减少内存分配、保留 Hook 状态、优化 Diff 算法。
3. **中断恢复**：即使渲染被打断，也能通过 `alternate`找回之前的工作进度。”

**如果组件 `key`变化，`alternate`还生效吗？**

> “不生效。React 通过 `key`和 `type`判断是否复用节点。如果 `key`变化，会销毁旧节点并创建新节点，`alternate`关系也会断开。”

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

**可中断渲染的具体解释：**

在 React Fiber 中，**高优先级更新可以打断低优先级渲染**，但这里的“打断”并不是指在单个 Fiber 节点内部暂停执行，而是指在 Fiber 树的协调过程中，React 可以**丢弃未完成的低优先级渲染任务**，并立即开始处理高优先级更新。

------

## 2. **通过并发特性局部启用并发行为**

React 18 提供了以下 API 来**手动标记优先级**，实现并发效果：

✅ **`startTransition`**

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

✅ **`useDeferredValue`**

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

✅ **`Suspense`**

配合懒加载或数据获取，实现更流畅的异步渲染：

```
<Suspense fallback={<Spinner />}>
  <LazyComponent />
</Suspense>
```

**不开启并发模式（legacy 模式）时，React 仍然有“优先级调度系统”，但它只用于“调度任务”何时执行，而不是“可中断渲染”**。也就是说，**优先级仍然存在，但调度能力是有限的**。

## 3. 📌 更深入理解（关键点）

**✅ 有调度系统（`scheduler`）**

- React 18 **无论是否开启并发模式，都会用内部的 `scheduler` 模块**。
- 这个模块有五个优先级（immediate, user-blocking, normal, low, idle）。
  - `Immediate`：用户输入（最高优先级）。
  - `UserBlocking`：动画、交互。
  - `Normal`：普通更新（如数据请求）。
  - `Low`：非紧急任务（如日志记录）。
  - `Idle`：空闲时执行。
- 例如你使用 `startTransition`，React 会把它调度为较低优先级任务。

**❌ 没有真正的“并发能力”**

如果你没有使用 `createRoot()`，而是用了 `ReactDOM.render()`（即 legacy 模式）：

- React 会**按优先级“排队执行任务”**，但一旦任务开始执行，**不能通过优先级中断、暂停或打断它**。
- 所以即使低优先级任务排在后面，它一旦开始执行，仍会一次性完成（“同步渲染”），用户依旧会感觉卡顿。

------

**📊 类比：非并发 vs 并发模式的优先级调度能力**

| 能力                    | 非并发模式                               | 并发模式 (Concurrent Mode) |
| ----------------------- | ---------------------------------------- | -------------------------- |
| 使用 scheduler 模块     | ✅ 有                                     | ✅ 有                       |
| setState 支持优先级分类 | ✅ 有                                     | ✅ 有                       |
| startTransition 起作用  | ✅ 起作用但不能中断                       | ✅ 起作用且可以中断         |
| 渲染可被中断（yield）   | ❌ 不支持，仅有被动暂停（防止浏览器卡死） | ✅ 支持                     |
| Suspense 异步边界       | 🚫 降级为同步                             | ✅ 异步、流式渲染           |

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

### 2. 为什么是链表结构

**React 使用链表结构管理 Hook，是为了在多次渲染中维护 Hook 的调用顺序和状态一致性，同时支持不同类型 Hook（useState、useReducer、useEffect 等）的插入与组合。**

**🧠 为什么使用链表而不是数组？**

1. **Hook 的调用顺序必须一致**

- React 函数组件在每次 render 时会重新调用函数体，React 必须能根据“调用顺序”正确地匹配每个 Hook 的状态。
- React 不能靠名字来区分 `a`、`b`、`c`，只能靠**调用顺序**。

2. **链表可以动态扩展不同 Hook 类型的数据结构**

- useState 和 useEffect 维护的数据结构不一样（一个是 state，一个是 effect queue）
- 使用链表，可以在节点中存储不同类型 Hook 的数据，同时保持执行顺序。

3. **链表可以避免稀疏数组/索引错乱问题**

- 如果用了数组，那 Hook 的增删改可能会导致索引错乱。
- 链表结构更适合在运行时构建和遍历，能逐一连接每个 Hook 实例，动态生成和更新。

------

### 3. **不同 Hook 的具体存储**

✅ `useState` / `useReducer`

- **`memoizedState`**：存储当前的状态值（如 `count`）。
- **`queue`**：存储待处理的更新（`dispatch` 触发的动作）。

✅ `useMemo` / `useCallback`

- **`memoizedState`**：存储缓存的值或函数（如 `[value, deps]`）。

✅ `useEffect` / `useLayoutEffect`

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

### 4. **Fiber 节点的关键属性**

```
type Fiber = {
  memoizedState: Hook | null,  // 函数组件的 Hook 链表
  updateQueue: UpdateQueue | null, // 用于存储 effect 链表（useEffect）
  // ...其他属性（如 type, child, sibling 等）
};
```

### 5. **不管组件定义了多少 `useEffect`，`memoizedState` 只会有一个 Effect 对象吗**

❌ 不正确

- 每个 `useEffect` 都会在 `fiber.memoizedState` 链表中创建一个 **独立的 Hook 节点**，每个节点的 `memoizedState` 存储自己的 Effect 对象。
- 但 React 会将这些 Effect 对象 **额外链接到 `fiber.updateQueue`** 中，形成环形链表供调度使用。

### **6. 为什么这样设计？**

1. **Hook 独立性**：每个 Hook 的状态（如 `useState` 的 state、`useEffect` 的 deps）需要隔离，避免互相污染。
2. **批量调度效率**：通过 `updateQueue` 链表，React 可以在提交阶段（commit phase）高效遍历并执行所有 Effect。

------

### 7. 总结

- **Hooks 状态**：统一通过 `fiber.memoizedState` 链表管理。
- **Effect 处理**：额外通过 `fiber.updateQueue` 调度，确保生命周期正确执行。

可以通过 React 源码（如 `ReactFiberHooks.js`）进一步验证这一机制。

# React Context的原理

https://yuanbao.tencent.com/chat/naQivTmsDa/8bd0e448-3ed9-45b2-b26b-0e6fee84e761?projectId=2f13020843e3426ab5f1fc84dfe2aa87

# Redux的原理

手写题

Redux 的状态更新是高度优化的。它采用订阅-发布模式，**只有当组件所依赖的 state 发生变化时，组件才会重新渲染**。这比 `Context` 的无条件重新渲染效率更高。

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

# React HOC 面试介绍

**1. 什么是 HOC？**

（Higher-Order Component，高阶组件）

- **定义**：HOC 是一个函数，接受一个组件作为参数，返回一个新的增强后的组件。
- 它是 React 组件复用逻辑的一种高级技巧，属于“组件组合”的模式。
- 公式表达：`const EnhancedComponent = higherOrderComponent(WrappedComponent)`

**2. HOC 的作用**

- 代码复用、抽象状态和逻辑，比如权限控制、数据获取、UI 状态管理、性能优化等。
- 避免重复代码，提高组件的复用性和维护性。

**3. HOC 的核心原理**

- 接收一个原始组件，返回一个包装组件。
- 包装组件在渲染时，会渲染原始组件并传入增强后的 props 或状态。
- 通过闭包和组合，实现增强功能。

**4. HOC 简单示例**

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

**5. HOC 使用场景**

- 权限校验：包装组件根据权限控制渲染内容。
- 数据获取：包装组件在挂载时请求数据，传递给原始组件。
- 事件处理：统一处理某些事件逻辑。
- 性能优化：例如缓存渲染结果。

**6. 面试常见问题**

- **HOC 与 Render Props 区别？**
  - HOC 是通过函数接受组件并返回新组件，Render Props 是通过一个函数作为子组件提供渲染内容。
- **HOC 如何避免命名冲突？**
  - 使用 `hoist-non-react-statics` 库拷贝静态属性，避免丢失原组件静态方法。
- **HOC 如何传递 refs？**
  - 需要用 `React.forwardRef` 来转发 ref，否则 ref 指向的是包装组件而非被包裹组件。

**7. HOC 的缺点**

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

| 阶段                             | 特点                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| 协调阶段（Reconciliation Phase） | 构建新 Fiber 树，进行 diff，标记变化（effectTag），是**可中断**的 |
| 提交阶段（Commit Phase）         | 应用所有变更到真实 DOM，调用生命周期等，是**不可中断**的     |

------

✅ 判断是否进入提交阶段的关键逻辑

React 内部在构建完 Work-in-Progress Fiber 树后，会检查是否存在副作用（即变化）。如果有，就进入提交阶段。

👇 具体表现：

1. **在协调阶段：**
   - React 会遍历整个 Fiber 树，执行组件的 render 函数，构建新的子树结构。
   - 在构建过程中，每个节点如果发生了变化（比如 DOM 更新、ref 变化、副作用 hook 等），都会被打上一个 effectTag 标记（如 `Placement`, `Update`, `Deletion` 等）。
   - `effectTag`是Fiber 节点的一个属性。
2. **协调结束后：**
   - 如果根节点 `root.finishedWork` 上有副作用（`root.finishedWork.flags !== NoFlags || root.finishedWork.subtreeFlags !== NoFlags`），则进入提交阶段。
   - 如果没有任何副作用，React 会跳过提交阶段（即不触发 DOM 更新，不调用副作用 hook）。

**在React 18+的并发模式下，协调可能被中断多次，但只有在整个Fiber树的副作用收集完成后才会真正进入提交阶段。提交阶段本身是同步不可中断的，这是React保证UI一致性的关键设计。**

整个 Fiber 树" 指的是 **从当前应用的根节点（Root）开始，包含所有需要更新的组件及其子组件构成的完整树形结构**。

# effectList是什么

 React 的 **Fiber 架构** 中，`effectList`（副作用链表）是一个 **单向链表**，用于在协调阶段（Reconciliation / Render Phase）高效收集需要执行副作用的 Fiber 节点，并在提交阶段（Commit Phase）批量处理这些副作用（如 DOM 操作、生命周期调用等）。

------

**1. `effectList`的作用**

- **减少遍历成本**：直接访问需要更新的节点，无需遍历整棵树。
- **批量更新**：避免频繁操作 DOM，合并变更以提高性能。
- **支持并发渲染**：可中断的协调阶段 + 原子化的提交阶段，保证 UI 一致性。

------

**2. `effectList`的结构**

`effectList`是一个 **单向链表**，通过 Fiber 节点的 `nextEffect`指针连接：

```
// Fiber 节点的简化结构
const fiber = {
  tag: 'HostComponent', // 节点类型（组件、DOM 等）
  effectTag: Placement, // 副作用标记（插入、更新、删除等）
  nextEffect: null,     // 指向下一个需要处理副作用的 Fiber
  // ...其他属性（child, sibling, return 等）
};
```

 **3.`effectList`的构建过程**

在协调阶段，React 会：

1. **标记副作用**：通过 Diff 算法确定 Fiber 节点的变更，并设置 `effectTag`（如 `Placement`、`Update`）。
2. **链接副作用节点**：
   - 如果当前 Fiber 有副作用（`effectTag !== NoEffect`），将其添加到父 Fiber 的 `effectList`。
   - 通过 `nextEffect`指针形成链表。
3. **回溯到根节点**：最终，根节点（`HostRoot`）的 `effectList`会包含所有需要处理的副作用节点。

# react协调阶段是递归还是循环

在 React 的 **协调阶段（Reconciliation）**，Fiber 架构使用 **循环（Loop）** 而非传统的递归（Recursion）来处理组件树的更新。这是 React 16+ 引入 Fiber 架构的核心优化之一。

------

**1. 为什么用循环替代递归？**

在 React 15 及之前，协调阶段使用 **递归遍历虚拟 DOM 树**，存在以下问题：

- **不可中断**：递归一旦开始就必须执行完，可能导致主线程长时间阻塞（影响交互响应）。
- **性能瓶颈**：深层次组件树会导致调用栈过深，可能触发堆栈溢出。
- **无法增量渲染**：无法拆分任务为小片段以利用浏览器的空闲时间（如 `requestIdleCallback`）。

React 16 引入 **Fiber 架构** 后，改用 **循环 + 链表结构** 的迭代方式，实现了：

✅ **可中断和恢复**：React 可以在浏览器空闲时处理一部分任务，避免阻塞。

✅ **增量渲染**：将任务拆分为多个小单元（Fiber 节点），按优先级调度。

✅ **更好的错误处理**：每个 Fiber 节点可以独立捕获错误（如 `Error Boundaries`）。

**2. Fiber 节点的遍历方式**

- **深度优先遍历（DFS）**，但使用 **循环 + 指针（`child`, `sibling`, `return`）** 替代递归：

  - `child`：指向第一个子 Fiber 节点。
  - `sibling`：指向下一个兄弟 Fiber 节点。
  - `return`：指向父节点（用于回溯）。

- **示例遍历顺序**：

  ```
  A
  / \
  B   C
    / \
   D   E
  ```

  遍历顺序：`A → B → C → D → E`（DFS），但通过循环实现。

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

**一道题目：**

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

### **🎯 小结图（逻辑流程）**

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

### **✅ 小结**

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

**同时有capture和bubble时，capture先：**

```html
<body>
  <button id="btn">Click</button>

  <script>
    const element = document.querySelector('#btn')
    element.addEventListener('click', () => {
      console.log('bubble');
    })
    element.addEventListener('click', () => {
      console.log('capture');
    }, true)
      
    // capture
    // bubble
  </script>
</body>
```

# `ReactDOM.createPortal` 

 React 提供的一种 **将子组件渲染到父组件 DOM 层级之外** 的方式。它用于解决一些 UI 组件需要“跳出当前组件层级”的问题，比如：

- 模态框（Modal）
- 弹窗（Dialog）
- Tooltip
- Toast 等浮层组件

------

✅ 语法

```js
ReactDOM.createPortal(children, container)
```

- `children`：要渲染的 React 元素
- `container`：目标 DOM 节点（通常是 `document.body` 下的某个元素）

------

✅ 举个例子

```jsx
import ReactDOM from 'react-dom';

function Modal({ children }) {
  return ReactDOM.createPortal(
    <div className="modal">{children}</div>,
    document.getElementById('modal-root') // 脱离父组件 DOM
  );
}
<!-- index.html -->
<body>
  <div id="root"></div>
  <div id="modal-root"></div> <!-- modal 渲染在这里 -->
</body>
```

------

✅ 为什么需要 Portal？

正常情况下，React 组件会被渲染在组件树下的某个位置。但像 modal、tooltip 这些浮层需要：

- 脱离当前 DOM 层级（避免被 `overflow:hidden`、`z-index` 限制）
- 放到 `body` 下面全屏展示
- 但仍然**保留 React 的组件行为（状态、事件、context）**

这时就用 `createPortal`。

------

✅ 特点总结

| 特点                | 描述                                                     |
| ------------------- | -------------------------------------------------------- |
| 保持 React 生命周期 | Portal 中的组件和父组件一样有相同的生命周期和 state 管理 |
| 渲染位置不同        | 虽然逻辑上是子组件，但渲染到了别的 DOM 节点中            |
| 常用于              | Modal、Tooltip、Dropdown、Toast 等浮层组件               |

------

✅ 注意事项

- `createPortal` **不改变组件的 React 层级**，只是改变了 DOM 位置
- 仍然可以访问父组件传递的 props、context
- 不会打破 React 的事件冒泡系统

# React流式渲染介绍

**什么是流式渲染(Streaming Rendering)**

流式渲染是React 18引入的一项重要特性，它允许服务器在生成HTML时逐步"流式"发送内容到浏览器，而不是等待整个页面完全渲染后再一次性发送。

**核心优势**

1. **更快的首屏渲染**：浏览器可以更早开始接收和显示内容
2. **更好的用户体验**：用户不会长时间面对空白页面
3. **更高效的资源利用**：服务器可以并行处理多个请求

**主要实现方式**

1. `renderToPipeableStream` (Node.js环境)

```
import { renderToPipeableStream } from 'react-dom/server';

const { pipe } = renderToPipeableStream(<App />, {
  bootstrapScripts: ['/main.js'],
  onShellReady() {
    response.setHeader('Content-type', 'text/html');
    pipe(response);
  }
});
```

2. `renderToReadableStream` (边缘计算/Web Streams环境)

```
import { renderToReadableStream } from 'react-dom/server';

async function handler(request) {
  const stream = await renderToReadableStream(<App />, {
    bootstrapScripts: ['/main.js'],
  });
  
  return new Response(stream, {
    headers: { 'Content-Type': 'text/html' },
  });
}
```

**配合Suspense使用**

流式渲染与React的Suspense组件完美配合：

```
function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Comments />
    </Suspense>
  );
}

function Comments() {
  const comments = use(fetchComments()); // 异步数据获取
  return <div>{comments.map(/* ... */)}</div>;
}
```

**实际应用场景**

1. 内容密集型页面（如新闻、电商产品页）
2. 需要大量数据加载的仪表盘
3. 社交媒体动态页面
4. 任何需要优化首屏性能的场景

**注意事项**

1. 需要现代浏览器支持
2. 与传统SSR相比，客户端JavaScript包可能需要更复杂的处理
3. 错误处理需要特别关注
4. SEO优化需要额外考虑

流式渲染代表了React服务端渲染的未来方向，能够显著提升大型应用的性能表现。

# useCallback(fn, deps)和useMemo(() => fn, deps)等价吗

**实现角度**

**在 React 源码中，`useCallback` 实际上是基于 `useMemo` 实现的：**

```
function useCallback(callback, deps) {
  return useMemo(() => callback, deps);
}
```

**性能考虑**

两者在性能上没有显著差异

**示例**

```js
// 这两个是等价的
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);

const memoizedCallback = useMemo(() => () => {
  doSomething(a, b);
}, [a, b]);
```

总结：在记忆化函数这个特定场景下，两者功能等价。

# react 虚拟dom diff的原理

**✅ 一句话总结：**

> React 使用**Heuristic（启发式）算法**，以 **O(n)** 的复杂度快速比较新旧虚拟 DOM，找出最小差异，并更新真实 DOM。

------

**✅ 背景：为什么要用 Virtual DOM Diff？**

直接操作真实 DOM 成本高（慢），React 通过构建 Virtual DOM 来避免频繁真实 DOM 更新。每次更新时，React 会：

1. 构建一棵新的 Virtual DOM 树（新状态下）
2. 对比新旧两棵 Virtual DOM 树（diff）
3. 找出差异并映射为真实 DOM 的更新操作（patch）

------

**✅ 核心 Diff 策略（重点考点）**

React 做了 **三条重要的优化假设**，以提升 diff 性能：

1. **同层比较（Tree Diff）**

> React 只比较**同一层级**的节点，不会跨层对比。

```jsx
<div>
  <p>A</p>
</div>

// 👇更新为👇

<div>
  <span>A</span>
</div>
```

不会比较 `<p>` 和 `<span>` 的子节点，只比较当前层级的**标签类型**，判断是否需要替换整棵子树。

 **React 的递归比较规则**

**默认行为：按需递归**

- 如果 **父组件的类型或 `props` 没有变化**，React 会跳过所有子组件的递归比较（性能优化）。
- 如果 **父组件需要更新**，React 会递归检查直接子节点，但不会无限制深入所有嵌套层级。

```jsx
function Parent() {
  return (
    <div>
      <Child />  {/* 直接子层级（Level 1） */}
    </div>
  );
}

function Child() {
  return (
    <div>
      <Grandchild /> {/* 嵌套子层级（Level 2），默认不比较 */}
    </div>
  );
}
```

- 当 `<Parent>` 更新时：
  1. React 先检查 `<Parent>` 的直接子节点 `<div>` 和 `<Child>`。
  2. 如果 `<Child>` 的 `props` 或类型没有变化，**停止递归**，不会检查 `<Grandchild>`。
  3. 如果 `<Child>` 需要更新，React 才会继续检查 `<Grandchild>`（按需递归）。

------

2. **不同类型的元素，直接销毁重建（Type Diff）**

```jsx
<div>hello</div> → <span>hello</span>
```

`div` 和 `span` 类型不同，React 直接卸载旧组件，创建新组件，不做更深的比较。

------

3. **列表 Diff（Keyed Diff）**

> 如果节点在列表中，React 会通过 **key** 属性优化判断是否是“相同的节点”。

```jsx
[
  <li key="a">A</li>,
  <li key="b">B</li>
]
👇更新为👇
[
  <li key="b">B</li>,
  <li key="a">A</li>
]
```

- 有 `key`：React 通过 `key` 判断复用哪一项，避免重新创建
- 没有 `key`：React 默认用 **位置（index）** 做对比，容易误判更新而不是复用，性能变差、动画错乱

------

**✅ Fiber 架构的 Diff（React 16+）**

React 16 之后，引入了 **Fiber 架构**，使得 Diff 更具弹性：

| 特点             | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| **分片执行**     | React 把 Diff 拆成小任务，避免长任务阻塞主线程（通过时间切片） |
| **优先级调度**   | 不同更新有不同优先级，比如输入变化 > 数据请求结果            |
| **中断恢复能力** | 渲染中断后可恢复，不必从头再来                               |
| **双缓存机制**   | 当前 Fiber 树 和 workInProgress Fiber 树做比较，提高效率     |

------

**✅ 总结**

> React 的 Virtual DOM diff 使用启发式算法做同层比较，结合 `key` 做高效列表复用。在 Fiber 架构下支持异步中断、优先级调度，使 diff 更加高效和灵活。

# react虚拟dom 在什么场景会有很严重性能问题

React 的虚拟 DOM（Virtual DOM）通常会带来性能提升，但在某些特定场景下，它反而可能造成**严重的性能问题**，主要原因包括：

------

❗️1. **频繁更新导致 Reconciliation 过于频繁**

场景：

- 页面中存在大量组件，每秒触发多次更新（如动画、输入联想、滚动事件中频繁更新状态等）。
- 比如：每 16ms（60fps）都在 `setState`，会导致 React 每帧都要走一次 diff 流程。

问题：

- 虚拟 DOM diff 和更新频繁进行，CPU 开销大，造成卡顿甚至掉帧。

示例：

```jsx
const [count, setCount] = useState(0);
useEffect(() => {
  const id = setInterval(() => setCount(c => c + 1), 10);
  return () => clearInterval(id);
}, []);
```

------

❗️2. **长列表未优化渲染**

场景：

- 渲染上千个 DOM 节点（如聊天记录、表格数据等）时，任何状态更新都会让整棵虚拟 DOM 重建 & diff。

问题：

- 虚拟 DOM diff 时间线性增长，浏览器真实 DOM patch 的成本也很高。
- 即使 diff 有优化（比如 key），也挡不住大规模节点的重建和 patch。

解决方案：

- 使用 windowing 技术，如 `react-window`、`react-virtualized`。

------

❗️3. **组件结构复杂 + 多层嵌套**

场景：

- 有大量嵌套组件，比如每一个 `<TableRow>` 内再包含多个 `<Cell>`，而 `props` 每次都略有变动。

问题：

- React 需要递归比较多层虚拟 DOM，并逐层执行 `render()` 或 `function component()`，即便实际变化不大。

解决方案：

- 使用 `React.memo`、`useMemo`、`useCallback` 来减少不必要的重新渲染。

------

❗️4. **没有合理使用 key 或 key 使用不当**

场景：

- 在数组渲染中，使用了 index 作为 key，或 key 不唯一。

问题：

- React 的 diff 算法基于 key 做节点复用和比较，key 错误会导致节点整体销毁重建，严重影响性能和动画表现。

------

❗️5. **React 中频繁触发 Context 更新**

场景：

- 使用 `React Context` 管理全局状态（如 theme、user、language 等），且更新频繁。

问题：

- 所有消费了该 context 的组件都会重新渲染，不管是否使用到了更新的字段。

解决方案：

- 将 context 拆分为更细粒度的多个 context，或结合 Zustand/Recoil/MobX 等更高性能的状态管理库。

------

**🔚 总结**

| 场景             | 原因            | 优化方向                          |
| ---------------- | --------------- | --------------------------------- |
| 高频状态更新     | diff 频繁       | 防抖节流、`requestAnimationFrame` |
| 长列表渲染       | 虚拟 DOM 规模大 | 虚拟滚动                          |
| 嵌套组件复杂     | 多次递归 diff   | React.memo、拆分组件              |
| key 不合理       | 节点无法复用    | 使用稳定唯一 key                  |
| Context 频繁更新 | 广播式重新渲染  | 拆分 context、状态隔离            |

# 部分场景下js原生table的性能会好吗

JS 原生 DOM 操作的 `<table>` 元素在某些**特定场景下确实可能比 React 渲染更高效**，尤其是在数据量大、变动频繁但 DOM 结构相对稳定时，性能优势更明显。

------

**✅ JS 原生 `<table>` 性能更好的原因**

1. **绕过 Virtual DOM 的中间过程**

React 每次更新：

- 会重新执行 `render()` 或函数组件
- 构建新的虚拟 DOM 树
- 与旧树做 diff
- 最后 patch 到真实 DOM

而原生 JS 操作：

- 直接增删改 DOM，不经过中间抽象层，**省略 diff 成本**

------

2. **对表格结构更友好**

HTML 表格（`<table><tr><td>...</td></tr></table>`）的 DOM 特性：

- 有天然的行列组织结构
- 浏览器对 `<table>` 有优化机制（如重排合并）

React 渲染 `<table>` 时因为多层嵌套组件（比如 `<Table><Row><Cell>...</Cell></Row></Table>`）会引入冗余结构，反而不如直接使用原生 DOM 高效。

------

3. **局部更新更容易精准控制**

用原生 JS 可以做到：

- 只修改某个 `<td>` 的 `.textContent` 或 `.className`
- 避免无关行列的更新

React 虽然也能做到（需复杂的 `memo`、`shouldComponentUpdate` 优化），但默认是重新渲染整行或整个表格。

------

**❗️原生表格操作也有劣势**

| 项目     | 原生 DOM                   | React             |
| -------- | -------------------------- | ----------------- |
| 性能     | 更快（特别是大数据）       | 中等，diff 有开销 |
| 开发效率 | 手动更新 DOM、管理状态复杂 | 声明式、易维护    |
| 可读性   | 易出错、容易混乱           | 结构清晰、组件化  |
| 状态同步 | 需要手动处理               | 状态驱动 UI       |

------

**✅ 适合原生表格的场景**

- 渲染大量静态表格数据（比如几千条数据）
- 表格更新频率极高（如交易行情）
- 对渲染性能要求极高的报表或仪表盘
- 想使用虚拟滚动、手动更新单元格内容（跳过 diff）

------

**✅ React 和原生结合的折中方案**

- 使用 React 管理整体结构，但关键区域用 `ref` + 原生 DOM 操作更新
- 或用 React 生成字符串模板，然后手动插入 `table.innerHTML = str`

------

**✅ 结论**

> **在追求极致性能、尤其是大表格更新频繁的场景下，JS 原生表格操作会比 React 更高效。**

但需要牺牲可维护性和开发效率，所以：

- ✅ 原生：高性能数据密集型表格
- ✅ React：一般表格，交互复杂、需要组件化开发

# React批处理更新

在**同一帧内**多次调用 `setState`，React **默认只会触发一次重渲染**，因为它具有**“批处理更新（Batching）”机制**。

------

**✅ 简答结论**

> **在事件处理函数或生命周期中，同一帧内多次调用 `setState`，React 会批量处理，只重渲染一次。**

------

**✅ 示例代码**

```jsx
function App() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    setCount(c => c + 1);
    setCount(c => c + 1);
    setCount(c => c + 1);
  };

  console.log('render', count);

  return <button onClick={handleClick}>Click me</button>;
}
```

点击一次按钮，控制台打印：

```
render 3
```

并且只 **渲染一次**。

------

**❓为什么不会渲染 3 次？**

React 使用了 **自动批处理机制（Automatic Batching）**：

- 会把**同步的多个 `setState` 调用合并成一次更新**
- 在**同一事件循环或生命周期中**合并（如 `onClick`、`useEffect`）

------

**⚠️ 注意：不同写法效果不同**

```js
// 不推荐（不会正确递增）
setCount(count + 1);
setCount(count + 1);
setCount(count + 1); // 最终 count 只 +1

// 推荐（正确累加）
setCount(c => c + 1);
setCount(c => c + 1);
setCount(c => c + 1); // 最终 count +3
```

------

**🧠 延伸：在哪些场景不批处理？**

在 React 18 之前，**setTimeout、Promise** 等**异步操作中不会自动批处理**：

```jsx
setTimeout(() => {
  setCount(c => c + 1);
  setCount(c => c + 1); // React 17 及以下：会渲染两次
});
```

**React 18 起：**

支持**自动批处理异步更新**，所以即使在 `setTimeout` 内也只渲染一次。

如果你使用 React 18，你会看到这些都被合并了。

------

**✅ 总结**

| 场景                                               | 多次 `setState` 是否合并 | 会渲染几次 |
| -------------------------------------------------- | ------------------------ | ---------- |
| 同一事件处理函数中                                 | ✅ 是                     | 1 次       |
| 同一个 `useEffect` 中                              | ✅ 是                     | 1 次       |
| 异步回调（setTimeout、Promise）中（React 18 之前） | ❌ 否                     | 多次       |
| 异步回调（React 18 起）                            | ✅ 是                     | 1 次       |

# React的"Lifting State Up"（状态提升）

简单来说，当多个子组件需要共享或同步某个状态时，你应该将这个状态**从子组件中移除，提升到它们最近的共同父组件中**。

- **状态的唯一性**：状态只应该存在于一个“单一数据源”中。如果多个组件需要依赖同一份状态，那么这个状态就应该放在它们共同的祖先组件中。
- **自上而下的数据流**：状态提升后，父组件负责管理这个状态。它将状态通过 `props` 传递给需要使用它的子组件。如果子组件需要修改这个状态，它不能直接修改，而是调用父组件通过 `props` 传递下来的**函数**，让父组件来执行状态更新。
