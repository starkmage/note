# The Process of V8 Compiling a Piece of Code

First, it's important to understand that machines cannot read JS code. Machines can only understand specific **machine code**. So if we want to run JS logic on a machine, we must translate JS code into machine code for the machine to recognize. **JS is an interpreted language**. For interpreted languages, the **interpreter** performs the following analysis on the source code:

- Generate an **AST** (Abstract Syntax Tree) through **lexical analysis** and **syntax analysis**
- Generate **bytecode**

Then the interpreter executes the program based on the bytecode. However, the entire execution process of JS is actually more complex than this. Let's break it down step by step.

## Generating AST

Generating an AST consists of two steps—lexical analysis and syntax analysis.

### Lexical Analysis

Lexical analysis, or tokenization, works by breaking down lines of code into individual tokens. For example, the following line of code:

```js
let name = 'sanyuan'
```

![](http://47.98.159.95/my_blog/week07/7.jpg)

One line is broken down into these 4 tokens, which is the function of lexical analysis.

### Syntax Analysis

The generated token data is transformed into an AST according to certain syntax rules. For example:

```js
let name = 'sanyuan'
console.log(name)
```

The final generated AST looks like this:

<img src="http://47.98.159.95/my_blog/week07/8.jpg" style="zoom: 80%;" />

After generating the AST, the compiler/interpreter's subsequent work relies on the AST rather than the source code. After generating the AST, the next step is to create an execution context. Regarding execution context, these articles explain it well:

http://interview.poetries.top/browser/part2/lesson07.html#_1-%E7%BC%96%E8%AF%91%E9%98%B6%E6%AE%B5

## Generating Bytecode

After generating the AST, **V8's interpreter** (also called Ignition) directly generates **bytecode**. However, **bytecode** cannot be run directly by the machine. You might ask, if it can't be executed, why convert it to bytecode? Why not convert the AST directly to machine code for the machine to execute? Indeed, V8 did this in its early days, but later, due to the large size of machine code, it caused serious memory usage problems.

It's easy to conclude that bytecode is much lighter than machine code. So why does V8 use bytecode, and what exactly is bytecode?

> Bytecode is a type of code between AST and machine code, but it is not related to a specific type of machine code. Bytecode needs to be converted to machine code by an interpreter and then executed.

**Bytecode still needs to be converted to machine code, but unlike before, now we don't need to convert all bytecode to machine code at once. Instead, the interpreter executes the bytecode line by line, eliminating the need to generate binary files, which greatly reduces memory pressure.**

## Executing Code

During the execution of bytecode, if V8 finds that a certain part of code appears repeatedly, it marks it as `hot code` (HotSpot), and then compiles this code into **machine code** and saves it. The tool used for this compilation is V8's **compiler** (also called `TurboFan`). Under this mechanism, the longer the code executes, the higher the execution efficiency becomes, because more and more bytecode is marked as hot code, and when encountered, the corresponding machine code is executed directly without having to convert it to machine code again.

**When you hear someone say that JS is just an interpreted language, strictly speaking, this statement is problematic. This is because bytecode not only works with the interpreter but also interacts with the compiler, so JS is not a completely interpreted language. The fundamental difference between a compiler and an interpreter is that the former generates binary files while the latter does not.**

**Moreover, this technology that combines bytecode with compilers and interpreters is called Just-In-Time compilation, which is what we often hear as `JIT`.**

## Summary

To summarize the entire process of executing a piece of JS code in V8:

1. First, generate an `AST` through lexical analysis and syntax analysis
2. Convert the AST to bytecode
3. The interpreter executes the bytecode line by line, and when it encounters hot code, it activates the compiler to compile it, generating corresponding machine code to optimize execution efficiency

## Reference Articles

Here are some additional notes on the differences between compilation and interpretation, which can be found in these articles:

Compiled languages need to go through the compilation process of the **compiler** before program execution, and after compilation, they directly retain binary files that the machine can understand. This way, each time the program runs, the binary file can be run directly without needing to be recompiled. Languages like C/C++, GO, etc., are compiled languages.

Programs written in interpreted languages need to be dynamically interpreted and executed by the **interpreter** each time they run. Languages like Python, JavaScript, etc., are interpreted languages.

http://47.98.159.95/my_blog/js-v8/003.html#_1-%E7%94%9F%E6%88%90-ast

https://segmentfault.com/a/1190000013126460

http://interview.poetries.top/browser/part3/lesson14.html#%E7%BC%96%E8%AF%91%E5%99%A8%E5%92%8C%E8%A7%A3%E9%87%8A%E5%99%A8

https://zhuanlan.zhihu.com/p/96502646

## How to Use AST to Transform JS Code

Tree structure

The uses of `AST` are very broad, to list a few simply:

- Error prompts, code formatting, code highlighting, code auto-completion, etc. in `IDEs`
- Code error or style checking by `JSLint`, `JSHint`, etc.
- Code packaging by `webpack`, `rollup`, etc.
- Transformation of `TypeScript`, `JSX`, etc. into native `Javascript`

https://www.cnblogs.com/goloving/p/14078228.html