# Does let Really Not Have Variable Hoisting?

Regarding let, many articles describe it as not having variable hoisting, which has been a confusing issue for me. After comprehensively reviewing many resources, I'll first state what I believe is the reasonable conclusion: let does have variable hoisting! Below, I'll explain my understanding of this in detail.

## The Process of Variable Creation

In JS, a variable goes through the following lifecycle stages, which form the basis for our discussion (we'll temporarily set aside the issue of destruction):

* Declaration
* Initialization
* Assignment

Additional note:

As for const, there is no assignment phase, only initialization.

## Variable Environment and Lexical Environment

We know that JS code must be compiled by the JS engine before it can run. After compilation, a piece of JS code generates two parts:

* Execution Context
* Executable Code

The execution context can be further divided into variable environment and lexical environment (we won't discuss the specific details of execution context here; you can look them up if needed). Variables and functions declared with var are placed in the variable environment, while variables declared with let and const are placed in the lexical environment. The lexical environment is like a stack, with each scope being a separate area within the lexical environment.

## Variable Hoisting

During the compilation phase, when the JS engine encounters a var declaration, such as var a = 1, it places a in the variable environment and initializes it as `undefined`. This completes what we commonly refer to as variable hoisting. When executing the code and reaching the statement var a = 1, it then assigns the value 1 to a.

For let b = 2, during the compilation phase, b is placed in the lexical environment, but **it is not initialized**. This creates what we commonly refer to as the temporal dead zone issue.

Only when executing the code and reaching the statement let b = 2 does let b get initialized to 2.

Let's look at a code example:

```js
console.log(b)
let b = 2
//This is a piece of code in a script tag
//Uncaught ReferenceError: Cannot access 'b' before initialization

let c
console.log(c)
//c is declared as undefined
//undefined
```

Isn't this easier to understand now? The error message says b is not initialized, indicating that b has already been declared.

To add a note, when operating directly in the browser console, the result is as follows, which can be more confusing:

* When operating directly in the console, the scope is global, whereas in a script tag of an HTML file, the scope is script.
* Variables declared with let do not belong to the global scope, so printing b in the console shows "not defined" because b is indeed not declared in the global scope.
* For more details on this, you can refer to [here](https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/30).

```js
console.log(b)
let b = 2
//This is directly in the console
//Uncaught ReferenceError: b is not defined
```

## Why Can't Variables Be Redeclared?

We know that having two var a = 1 statements is not a problem; the second assignment will overwrite the first. But for let, this would cause an error. Without going into specific details, we mentioned earlier that each scope is a separate area in the lexical environment, and redeclaration is not allowed within this separate area.

## let x = x

This statement, let x, is not a problem. Before execution, x's declaration has been hoisted. Now it's trying to initialize x, but the right-side x is a variable that has been declared but not initialized, so it will throw an error. Moreover, x will forever remain in a state of being declared but not initialized, creating a permanent temporal dead zone. This indicates that let has only one chance for initialization.

## Summary

* var's **declaration** and **initialization** are hoisted
* function's declaration, initialization, and assignment are all hoisted
* let only has its declaration hoisted