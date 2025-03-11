## Why Do We Need to Pass Props to super?

Sometimes, even when we don't pass props to super() in the constructor, we can still access this.props in the render function or other methods. How is this possible? In fact, React **assigns props to the instance right after** calling the constructor.

Does this mean we can use super() instead of super(props)?

It's best not to do this. Logically, we can't be certain there won't be issues, **because React only assigns this.props after the constructor has finished executing. This means this.props will be undefined during the period between the super call and the end of the constructor**.

```js
class Button extends React.Component {
  constructor(props) {
    super();  // props not passed
    console.log(props);      // {}
    console.log(this.props); // undefined
  }
}
```

If you call other internal methods in the constructor, debugging will become more difficult if errors occur. This is why developers are recommended to always execute super(props).

```js
class Button extends React.Component {
  constructor(props) {
    super(props) // passing props
    console.log(props)      // {}
    console.log(this.props) // {}
    console.log(props === this.props)    // true
  }
}
```

This ensures that this.props is assigned before the constructor finishes executing.

Reference:
https://www.jianshu.com/p/b55949fa20a2

## Immutability in React

Immutability means that data's internal structure and values cannot be changed. For example, some array operations in JavaScript are immutable (they return a new array instead of modifying the original array). String operations are always immutable (they create a new string containing the changes).

> "Arrays are reference types, and when modifying the current array using the push method, React's state management doesn't detect a change in the array's reference. Therefore, during React's dirty checking comparison of this array, it cannot determine that it has changed, and won't trigger React's state update to update the DOM."
>
> This statement seems problematic. It's not because arrays are reference types that React's state doesn't change when using push. You can try this.state = [], and even if the reference changes, it won't trigger a re-render.
>
> React's data updates use a push approach, meaning users need to call setState to tell React that the state has updated. Otherwise, React won't know, unlike Vue which uses dependency collection. setState calls a method to update the state.

Why is immutability important?

* Helps improve component or entire application performance
* Simpler Undo/Redo functionality
* Change Tracking
  For directly mutated objects, it's difficult to determine if they've been modified since all changes are made directly on the original object. This requires comparing the current object with a previous copy, traversing the entire object, and comparing each variable and value. This process can become increasingly complex. However, determining if an immutable object has been modified is quite simple. If the referenced object is different from before, then the object has changed. That's all there is to it.

## Lifting State Up

In React, sharing state between multiple components is achieved by moving it up to their closest common ancestor. This is called "lifting state up".

In React applications, any piece of mutable data should have a single "source of truth". Usually, state is first added to the component that needs to render the data. Then, if other components also need this state, you can lift it up to their closest common ancestor. You should rely on the [top-down data flow](https://reactjs.org/docs/state-and-lifecycle.html#the-data-flows-down) instead of trying to sync state between different components.

## Using Context for Ancestor-Descendant Component Communication

https://reactjs.org/docs/context.html

## React Lifecycle Methods

https://reactjs.org/docs/react-component.html

## React Refs

https://reactjs.org/docs/refs-and-the-dom.html

**You can't use the `ref` attribute on function components** because they don't have instances.

You can, however, **use the `ref` attribute inside a function component** as long as you refer to a DOM element or a class component.

## Hooks

### Custom Hooks

* **Do custom Hooks need to start with "`use`"?** Yes, this is mandatory. This convention is very important. Without it, React wouldn't be able to automatically check if your Hook violates the [Rules of Hooks](https://reactjs.org/docs/hooks-rules.html) because it couldn't tell if a function contains calls to Hooks inside it.

### useEffect Execution Timing

Unlike `componentDidMount` and `componentDidUpdate`, the function passed to `useEffect` is called after the browser has finished layout and painting, in a deferred event. This makes it suitable for many common side effects, like setting up subscriptions and event handlers, because most operations shouldn't block the browser from updating the screen.

However, not all effects can be deferred. For example, a DOM mutation that is visible to the user must be performed synchronously before the next paint so that the user doesn't perceive a visual inconsistency. For these types of effects, React provides useLayoutEffect(), which works similarly to useEffect() but fires before the browser repaints.

You can pass a second argument to `useEffect`, an array of values that the effect depends on. The effect will only execute when these values change. If you want to run an effect and clean it up only once (on mount and unmount), you can pass an empty array (`[]`) as a second argument. This tells React that your effect doesn't depend on any values from props or state, so it never needs to re-run.

The function passed to useEffect must return either a **cleanup function** or **undefined**. Any other return value will cause an error.

useLayoutEffect:

Execution timing is after the DOM tree is updated but before the browser repaints.

At this point, if you get style or DOM dimensions, you're getting the dimensions that the browser is about to render, not the dimensions currently displayed on the page.

It prevents render flickering by giving you one more chance to modify the DOM before rendering.