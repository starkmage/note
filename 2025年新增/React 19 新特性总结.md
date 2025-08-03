# React 19 新特性总结

React 19 是一个重要的版本更新，带来了许多令人兴奋的新特性和改进[^1]。作为前端面试准备，以下是React 19的核心新特性详解。

## 一、Actions - 异步操作的革命性改进

### 概念介绍

Actions 是React 19中引入的一个新概念，用于简化异步数据处理和状态管理[^1][^2]。它支持在过渡中使用异步函数，自动处理待定状态、错误、表单和乐观更新[^1]。

### 核心特性

- **自动管理Pending状态**：Actions会自动管理提交数据的挂起状态，当发起请求时挂起状态自动启动，请求完成后自动重置[^3]
- **内置乐观更新支持**：支持在等待请求提交时向用户显示预期结果[^3]
- **简化错误处理**：提供内置的错误处理功能，集成错误边界[^2][^3]
- **表单支持**：`<form>`元素现在支持将函数传递给`action`和`formAction`属性[^3]


### 使用示例

```javascript
// 使用Actions处理表单提交
function UpdateName() {
  const [name, setName] = useState("");
  const [error, setError] = useState(null);
  const [isPending, startTransition] = useTransition();

  const handleSubmit = () => {
    startTransition(async () => {
      const error = await updateName(name);
      if (error) {
        setError(error);
        return;
      }
      redirect("/path");
    })
  };

  return (
    <form action={handleSubmit}>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <button type="submit" disabled={isPending}>
        {isPending ? 'Updating...' : 'Update'}
      </button>
    </form>
  );
}
```


## 二、新的Hook函数

### useOptimistic

`useOptimistic(state, updateFn)` 是 **引入的新 Hook**，主要用于实现前端的 **乐观 UI 更新** —— 即**在请求未完成之前，先在 UI 上“假装成功”地更新**，提高用户交互体验。

------

✅ 一句话解释

> `useOptimistic` 让你**同步更新 UI 状态**，**异步请求结果返回后自动合并**，适合表单提交、点赞等需要“立即响应”的场景。`useOptimistic` 只在你触发 server action（如 form 的 action 提交或 useFormState/useFormStatus）时才有效。

------

✨ Hook 签名

```ts
const [optimisticState, addOptimistic] = useOptimistic<TState, TUpdate>(
  state: TState,
  updateFn: (currentState: TState, update: TUpdate) => TState
);
```

- `state`: 原始状态值
- `updateFn`: 更新函数，接收当前状态和“局部更新对象”，返回新的状态
- `optimisticState`: 当前正在展示的“乐观状态值”
- `addOptimistic(update)`: 你可以调用这个函数来触发一次“乐观更新”，它会立即更新 UI

------

✅ 举个例子：乐观添加评论

```jsx
'use client';

import { useState, useOptimistic } from 'react';

export default function Comments() {
  const [comments, setComments] = useState([
    { id: 1, text: 'Hello world' },
  ]);

  // 第一步：定义如何乐观添加评论
  const [optimisticComments, addOptimisticComment] = useOptimistic(
    comments,
    (currentComments, newCommentText) => [
      ...currentComments,
      {
        id: Date.now(), // 临时 ID
        text: newCommentText,
        optimistic: true, // 可以用来标记是“假的”
      },
    ]
  );

  // 第二步：处理提交
  const handleSubmit = async (e) => {
    e.preventDefault();
    const form = e.target;
    const input = form.elements.comment;
    const newComment = input.value;

    // 乐观更新 UI
    addOptimisticComment(newComment);

    // 真正发送请求
    await fetch('/api/comments', {
      method: 'POST',
      body: JSON.stringify({ text: newComment }),
    });

    // 请求完成后，刷新真实数据
    const res = await fetch('/api/comments');
    const data = await res.json();
    setComments(data); // 替换为真实数据
    input.value = '';
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="comment" />
      <button type="submit">添加评论</button>

      <ul>
        {optimisticComments.map((comment) => (
          <li key={comment.id}>
            {comment.text}
            {comment.optimistic && '（发送中）'}
          </li>
        ))}
      </ul>
    </form>
  );
}
```

------

📌 使用场景总结

| 场景                      | 是否适合用 `useOptimistic` |
| ------------------------- | -------------------------- |
| 点赞 / 收藏按钮           | ✅ 非常合适                 |
| 添加/删除列表项（如评论） | ✅ 推荐                     |
| 表单提交 / 搜索           | ✅ 可用（展示 loading）     |
| 拖拽排序、投票交互等      | ✅ 合适                     |
| 数据一致性要求很高的操作  | ❌ 慎用（建议等后端响应）   |

------

🧠 面试怎么说？

> `useOptimistic` 是 React 18.3 新增的状态管理 Hook，用于实现乐观 UI 更新。它通过局部更新函数让 UI 能立即响应用户操作，等后端响应后再自动对状态做真实同步，提升用户体验。

### useActionState

`useActionState`是专门用于处理Actions中数据变化和状态更新的Hook[^8][^9][^10]。

**主要功能**：

- 管理待处理状态：自动跟踪并更新待处理状态[^8]
- 返回操作结果：提供操作的最终结果[^8]
- 简化表单处理：特别适用于表单提交和数据变更操作[^9]


### use API

React 19引入了全新的`use` API，可以在渲染期间读取资源[^11][^12]。

**特点**：

- 可以处理Promise和Context[^12]
- 简化异步数据获取[^12]
- 需要配合Suspense使用
- 可以在条件语句和循环中使用（打破了 Hooks 的规则限制）

```javascript
import { use, Suspense } from 'react';

const fetchUsers = async () => {
  const res = await fetch('https://api.example.com/users');
  return res.json();
};

const userPromise = fetchUsers();

const Users = () => {
  const users = use(userPromise);
  return (
    <ul>
      {users.map((user) => (
        <div key={user.id}>
          <h2>{user.name}</h2>
        </div>
      ))}
    </ul>
  );
};

function App() {
  return (
    <Suspense fallback={<h1>Loading...</h1>}>
      <Users />
    </Suspense>
  );
}
```


## 三、Server Components稳定支持

React 19将Server Components功能推向稳定[^2][^8]。Server Components提供了一种在服务器端预渲染组件的新方法[^8]。

### 主要优势

- **缩短初始页面加载时间**：通过在服务器端渲染组件，减少发送到客户端的JavaScript数量[^8]
- **增强代码可移植性**：允许开发者编写可同时在服务器和客户端运行的组件[^8]
- **更好的SEO**：服务器端渲染确保搜索引擎能够更好地抓取和索引内容[^8]


### 新指令

- **'use client'**：标记应在客户端运行的代码[^8]
- **'use server'**：标记在服务器端执行的函数[^8]


## 四、React Compiler

React 19引入了新的React Compiler（原名React Forget）[^13][^14][^15]。

### 核心功能

- **自动记忆化**：编译器自动对组件和Hook中的值进行记忆化处理[^13][^16]
- **性能优化**：将React代码转换为优化的JavaScript，提升性能[^14][^15]
- **无需手动优化**：开发者不再需要手动使用`useMemo`、`useCallback`和`React.memo`[^14][^16]


### 特点

- 仅在构建时使用的工具[^13][^16]
- 可以与纯JavaScript一起使用[^13][^16]
- 理解React规则，无需重写代码即可使用[^13][^16]


## 五、API和使用方式的改进

### ref作为属性

从React 19开始，可以在函数组件中将`ref`作为prop进行访问，不再需要`forwardRef`[^1][^17][^18]。

```javascript
// React 19
function MyInput({placeholder, ref}) {
  return <input placeholder={placeholder} ref={ref} />
}

// 使用
<MyInput ref={ref} />
```


### Context作为提供者

React 19中可以将`<Context>`直接渲染为提供者，无需使用`<Context.Provider>`[^1][^19]。

```javascript
// React 19
const ThemeContext = createContext('');

function App({children}) {
  return (
    <ThemeContext value="dark">
      {children}
    </ThemeContext>
  );
}
```


## 六、资源预加载和性能优化

### 资源预加载支持

React 19引入了多项新API，用于高效加载和预加载浏览器资源[^11][^20][^21]。

**新的API包括**：

- `preload`: 预加载资源[^20]
- `preinit`: 加载并执行脚本[^20]
- `prefetchDNS`: DNS预取[^20]
- `preconnect`: 预连接[^20]

```javascript
import { prefetchDNS, preconnect, preload, preinit } from 'react-dom'

function MyComponent() {
  preinit('https://.../script.js', {as: 'script' }); 
  preload('https://.../font.woff', { as: 'font' }); 
  preload('https://.../stylesheet.css', { as: 'style' }); 
  prefetchDNS('https://...'); 
  preconnect('https://...'); 
}
```


### 样式表优先级管理

React 19增强了样式表的加载管理，通过指定`precedence`属性，React可以动态调整样式表的插入顺序[^2]。

## 七、错误处理和开发体验改进

### 改进的错误报告

React 19改进了水合错误的错误报告，提供更清晰的不匹配差异信息[^1]。

### 第三方脚本兼容性

React 19修复了与第三方浏览器扩展（如Grammarly）的兼容性问题，改善了水合过程中的错误处理[^22][^23]。

## 八、升级指南

### 安装步骤

```bash
npm install --save-exact react@^19.0.0 react-dom@^19.0.0
```


### 代码迁移工具

React团队提供了自动化的codemod工具来帮助升级[^24]：

```bash
npx codemod@latest react/19/migration-recipe
```


### 重要变更

- **新的JSX转换是必需的**：React 19需要启用新的JSX转换[^24]
- **渲染错误处理变更**：错误不再重新抛出，改为报告给`window.reportError`或`console.error`[^24]


## 总结

React 19是一个重要的版本更新，主要特点包括：

1. **简化开发**：通过Actions、新的Hook和自动优化减少了样板代码
2. **性能提升**：React Compiler提供自动记忆化和优化
3. **更好的开发体验**：改进的错误处理、资源预加载和API简化
4. **服务器端能力**：稳定的Server Components支持
5. **向后兼容**：提供平滑的升级路径和迁移工具

# 你在Actions中的例子我没有看懂，useTransition不是React18引入的吗

你的理解是完全正确的！`useTransition` 确实是在 React 18 中引入的 Hook[^1][^2][^3]。我在之前的 Actions 例子中确实存在表述不够清晰的地方，让我来详细解释一下。

## React 18 vs React 19 中的 useTransition

### React 18 中的 useTransition

在 React 18 中，`useTransition` 主要用于标记非紧急的状态更新，让 React 能够区分高优先级和低优先级的更新[^4][^2]。但在 React 18 中有一个重要限制：

**React 18 的限制**：在异步操作（如 `await`）之后的状态更新需要重新包裹在 `startTransition` 中[^1]。这是一个已知的限制，官方文档明确提到："你必须将任意异步请求之后的状态更新用 `startTransition` 包裹，以将其标记为 Transition 更新"[^1]。

### React 19 中的改进

React 19 对 `useTransition` 进行了重要改进，新增了对异步函数的原生支持[^5][^6]：

**React 19 的新特性**：

- 支持在 `startTransition` 中直接使用异步函数
- 自动处理挂起状态、错误和乐观更新
- 无需在 `await` 后重新包裹状态更新


## 正确的代码示例对比

### React 18 的写法

```javascript
// React 18 - 需要手动管理状态
function UpdateName() {
  const [name, setName] = useState("");
  const [error, setError] = useState(null);
  const [isPending, setIsPending] = useState(false);

  const handleSubmit = async () => {
    setIsPending(true);
    const error = await updateName(name);
    setIsPending(false);
    if (error) {
      setError(error);
      return;
    }
    redirect("/path");
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <button type="submit" disabled={isPending}>
        {isPending ? 'Updating...' : 'Update'}
      </button>
      {error && <p>{error}</p>}
    </form>
  );
}
```


### React 19 的改进写法

```javascript
// React 19 - 使用 Actions 自动管理状态
function UpdateName() {
  const [name, setName] = useState("");
  const [error, setError] = useState(null);
  const [isPending, startTransition] = useTransition();

  const handleSubmit = () => {
    startTransition(async () => {  // React 19 支持异步函数
      const error = await updateName(name);
      if (error) {
        setError(error);
        return;
      }
      redirect("/path");
    });
  };

  return (
    <form action={handleSubmit}>  {/* React 19 支持 action 属性 */}
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <button type="submit" disabled={isPending}>
        {isPending ? 'Updating...' : 'Update'}
      </button>
      {error && <p>{error}</p>}
    </form>
  );
}
```


## Actions 的核心概念

在 React 19 中，**Actions** 指的是传递给 `startTransition` 的函数，特别是异步函数[^7][^5]。React 19 的官方定义是："按照约定，使用异步转换的函数被称为 'Actions'"[^5]。

**Actions 的特点**：

- 自动管理 pending 状态[^5]
- 内置错误处理机制[^5]
- 支持乐观更新[^5]
- 可以直接用于表单的 `action` 属性[^5]


## 总结

你的疑问很有道理。`useTransition` 确实是 React 18 的特性，但 React 19 对其进行了重要扩展，新增了对异步函数的支持，这就是 Actions 概念的核心[^5][^6]。我之前的例子应该更清楚地说明这是 React 19 中 `useTransition` 的**新增能力**，而不是完全的新特性。

React 19 的 Actions 本质上是对 React 18 中 `useTransition` 的增强，让异步状态管理变得更加简单和自动化[^8][^6]。
