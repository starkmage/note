## React PureComponent

React 15.3 introduced a new `PureComponent` class. As the name suggests, `pure` means pure, and `PureComponent` is a pure component that replaces its predecessor `PureRenderMixin`.

`PureComponent` is one of the most important ways to optimize React applications. It's easy to implement - just change the inheritance class from `Component` to `PureComponent`. This can reduce unnecessary `render` operations, thereby improving performance, and you can write less `shouldComponentUpdate` code.

When a component updates, if both the component's `props` and `state` haven't changed, the `render` method won't be triggered, avoiding the generation and comparison of the `Virtual DOM`, thus achieving performance improvement.

Reference: https://juejin.cn/post/6844903480369512455

## Vue2 and React Diff Comparison

Similarities:

* Both compare at the same level

Differences:

* Vue's list comparison uses a two-ends-to-middle approach

* React uses a left-to-right sequential comparison approach. React first iterates through the new collection, using unique keys to determine if the same node exists in the old collection. If not, it creates one; if yes, it performs a move operation. When rendering arrays in React, if child components don't provide keys, React will default to using the loop index as the key for the first render

* When moving the last node of a collection to the first position, React will move each preceding node sequentially, while Vue will only move the last node to the first position

* Vue2's core Diff algorithm uses a double-ended comparison algorithm, comparing from both ends of the new and old children simultaneously, using key values to find reusable nodes before performing related operations. Compared to React's Diff algorithm, this can reduce the number of node movements under the same circumstances, reducing unnecessary performance overhead

## What is the Main Goal of React Fiber?

*React Fiber's* goal is to improve its applicability in animation, layout, and gesture areas. Its main feature is **incremental rendering**: the ability to split rendering tasks into smaller chunks and distribute tasks across multiple frames.

## How Can setState in React.Component Update ReactDOM?

https://overreacted.io/how-does-setstate-know-what-to-do/

Renderers like `react-dom` and `react-native` **set a special field on the created class.** This field is called `updater`.

`setState` reads `this.updater` set by React DOM and lets React DOM schedule and handle the update.

**Hooks use a "dispatcher" object instead of the `updater` field.** When you call `React.useState()`, `React.useEffect()`, or other built-in Hooks, these calls are forwarded to the current dispatcher. Each renderer sets the dispatcher before rendering your components.

## Parent-Child Component Rendering in React

When a parent component renders, by default it will trigger the render process of its child components, and the child components' render process will trigger their child components' render process, all the way down to React elements (like `<div>` elements in JSX). When the render process reaches leaf nodes (React elements), the diff process begins, and the diff algorithm decides whether to actually update the DOM elements.

https://zhuanlan.zhihu.com/p/61547606

## React Hooks: useMemo, useCallback, and memo

Class components have pureComponent (shallow comparison) and shouldComponentUpdate (custom deep comparison)

https://juejin.cn/post/6844903954539626510

memo and pureComponent serve the same purpose

`useCallback` and `useMemo` both execute when the component first renders and then re-execute when their dependencies change. Both hooks return cached values: `useMemo` returns a cached **variable**, while `useCallback` returns a cached **function**

Using only useMemo and useCallback for performance optimization might not achieve the expected results because while the child component won't re-render if the props reference doesn't change, it will still re-execute. That's why memo should also be used.

memo uses shallow comparison by default. If you want to control the comparison process, you can pass a custom comparison function as the second parameter. This second parameter is equivalent to shouldComponentUpdate.

```react
function MyComponent(props) {
  /* render using props */
}
function areEqual(prevProps, nextProps) {
  /*
  return true if passing nextProps to render would return
  the same result as passing prevProps to render,
  otherwise return false
  */
}
export default React.memo(MyComponent, areEqual);
```

Difference between useMemo and memo: memo wraps the entire component, while useMemo targets specific props properties. Of course, it can also wrap the entire component. Here's a quote:

```text
props itself is a new object on every render. memo() does shallow comparison by default. 
You can think of memo() as a useMemo() with a list of all your props like [props.foo, props.bar, props.foobar] except you don't need to type them all manually. It's a shortcut for the most common case.
```

Reference articles:

https://zhuanlan.zhihu.com/p/268802571

https://juejin.cn/post/6844904032113278990

## Understanding React Side Effects

A side effect (Side Effect) refers to when a function does something unrelated to its return value, such as: modifying global variables, modifying input parameters, or even console.log(). Therefore, ajax requests and DOM modifications are considered side effects.

## What Are the Current Render Reasons?

1. Props changed

   Self-explanatory, the props passed to the component changed

2. The parent component rendered

Child component rendering caused by parent component rendering. Performance optimization usually targets components with this type of re-rendering. However, note that if a component uses useContext, re-renders caused by Provider value changes are also marked as "The parent component rendered"

3. Hooks changed

Re-renders caused by Hook state changes, typically referring to when the update function returned by useState is called

4. Class component state changes
5. Context wrapped

## Controlled vs Uncontrolled Components

**Controlled Components:**

Components that either have no internal state or whose internal state is completely determined by props

**Uncontrolled Components:**

Components that have internal state not controlled by props

Controlled and uncontrolled components are common in components related to user interaction. A typical example is the native input component, which can be either controlled or uncontrolled depending on how it's written

```react
/* Controlled input */
<input value={text} onChange={handleChange} />

/* Uncontrolled input */
<input defaultValue={initialText} onChange={handleChange} />
```

The React community has reached a consensus that if a component can run in both controlled and uncontrolled modes, generally passing defaultValue/defaultChecked indicates uncontrolled mode, while passing value/checked and onChange indicates controlled mode.

**Advantages and Disadvantages of Controlled Components**

**Advantages**

Controlled components are completely controlled by parent component props, meaning when using multiple controlled components, you can naturally access and modify all component states in the parent component. When multiple controlled components need to communicate or link their states, the parent component can easily update child component states as needed.

**Disadvantages**

Component state isn't self-contained, and performance is poor. All controlled component states are stored in the parent component, so when a controlled component needs to update its UI, it must trigger the parent component's state update to update itself, and the parent component's update will trigger all child components to update.

Performance issues commonly appear in CRUD lists and complex form business logic. Even if a controlled component is completely independent of its sibling components, updates will trigger sibling components to re-render.

More complex to use due to many props. Not conducive to separating concerns in the parent component.

**Advantages and Disadvantages of Uncontrolled Components**

**Advantages**

The advantages and disadvantages of uncontrolled components are opposite to those of controlled components. The advantage is better performance, as updates don't depend on the parent component, thus avoiding triggering sibling component updates. Due to high logic cohesion and less dependency on parent component props, they're also simpler to use.

**Disadvantages**

Resetting and related updates of uncontrolled components are more difficult and complex, requiring unmounting the component and reinitializing it, typically solved using the key prop

## Differences Between React Class Components and Function Components

* Syntax
* State management: setState vs useState
* Lifecycle hooks: function components don't have lifecycle methods, can only simulate them using useEffect and useLayoutEffect

* Calling method

  If `SayHi` is a function, React needs to call it:

  ```react
  // Your code
  function SayHi() { 
      return <p>Hello, React</p> 
  } 
  // React internal
  const result = SayHi(props) // » <p>Hello, React</p>
  ```

  If `SayHi` is a class, React needs to instantiate it with the `new` operator first, then call the render method of the generated instance:

  ```react
  // Your code
  class SayHi extends React.Component { 
      render() { 
          return <p>Hello, React</p> 
      } 
  } 
  // React internal
  const instance = new SayHi(props) // » SayHi {} 
  const result = instance.render() // » <p>Hello, React</p>
  ```

  **As you can imagine, when function components re-render, they'll re-call the component method to return a new React element, while class components will create a new component instance and call its render method to return a React element when re-rendering. This also explains why this is mutable in class components**

* **Getting render-time values, the biggest difference**

  Reference article: https://segmentfault.com/a/1190000020861150

## React Hooks Principles

https://github.com/brickspert/blog/issues/26

* Q: Why does an empty array as the second parameter equate to `componentDidMount`?

  A: Because the dependencies never change, the callback won't execute a second time.

* Q: Why can Hooks only be called at the top level of functions? Why shouldn't they be called in loops, conditions, or nested functions?

  A: The memoizedState array stores data in the order of Hook definitions. If the Hook order changes, memoizedState won't be aware of it.

* Q: How do custom Hooks affect the function components that use them?

  A: They share the same memoizedState and the same order.

* Q: How is the "Capture Value" feature produced?

  A: Every time there's a ReRender, the function component is re-executed. For previously executed function components, no operations are performed.

* React uses a linked list-like structure instead of an array. All hooks are chained in order through next.

* Where are memoizedState and cursor stored? How do they correspond one-to-one with each function component?

  We know that React generates a component tree (or Fiber linked list), where each node corresponds to a component. Hooks data is stored as component information on these nodes, born and dying with the components.

## Why Isn't React MVVM?

MVVM's most distinctive feature: two-way binding.

React doesn't have this.

React is a unidirectional data flow library, where state drives view.

```text
State --> View --> New State --> New View
```

## When is setState Synchronous and When is it Asynchronous in React?

In React, **if it's an event handler triggered by React (like event handling triggered by onClick), calling setState won't synchronously update this.state. Other setState calls will synchronously execute this.state**. "Other" refers to event handler functions directly added through addEventListener bypassing React, and asynchronous calls through setTimeout/setInterval.

**Reason:** In React's setState function implementation, it uses a variable isBatchingUpdates to determine whether to directly update this.state or put it in a queue for later. By default, isBatchingUpdates is false, meaning setState will synchronously update this.state. However, **there's a function batchedUpdates that sets isBatchingUpdates to true, and React calls this batchedUpdates before calling event handler functions, resulting in setState not synchronously updating this.state during React-controlled event handling processes**.

**Note:** The "asynchronous" nature of setState doesn't mean it's implemented with asynchronous code. The execution process and code itself are synchronous, but because synthetic events and hook function calls occur before updates, you can't immediately get the updated value in synthetic events and hook functions, creating a so-called "asynchronous" effect. Of course, you can get the updated result through the callback in the second parameter setState(partialState, callback).

Here, asynchronous means multiple states are combined for batch updating.

https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/17

https://blog.csdn.net/qq_43182723/article/details/106802413

## Is React's event Object Native or Synthetic?

In React, all events are synthetic, not native DOM events, but you can get the DOM event through the e.nativeEvent property.

Vue uses native events.

## How to Handle State Preservation Without keep-alive in React?

Someone once raised this feature in an official [issue](https://github.com/facebook/react/issues/12039), but the official team believes this feature could easily cause memory leaks and has indicated they don't currently plan to support it

**Manual state preservation** is a common solution. You can save data through state management layers like `redux` using the `componentWillUnmount` lifecycle in combination with React components, and restore data through the `componentDidMount` lifecycle

When there's little state to preserve, this method can quickly implement the functionality we need, but when there's a large amount of data or many variable situations, manual state preservation can become troublesome

https://juejin.cn/post/6844903942522929160#heading-1