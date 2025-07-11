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
