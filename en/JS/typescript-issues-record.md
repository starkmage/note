## Advantages of TS

### Differences Between JavaScript and TypeScript

TypeScript is a superset of JavaScript that extends JavaScript's syntax, allowing existing JavaScript code to work with TypeScript without any modifications. TypeScript provides compile-time static type checking through type annotations.

TypeScript can handle existing JavaScript code and only compiles the TypeScript portions of it.

![img](https://www.runoob.com/wp-content/uploads/2019/01/ts-2020-11-26-1.png)

### Advantages

* Compile-time static type checking
* Easier maintenance and refactoring
* Better code quality control in multi-developer projects
* Can work alongside JavaScript without any modifications

## Decorators

* https://www.cnblogs.com/winfred/p/8216650.html
* https://saul-mirone.github.io/zh-hans/a-complete-guide-to-typescript-decorator/
* Types
  * Class decorators
  * Method decorators
  * Method parameter decorators
  * Property decorators
* The essence of decorators is functions executed at compile time (not TypeScript compilation, but during the compilation phase in the JavaScript execution engine)

## Generics

https://juejin.cn/post/6844904184894980104#heading-0

Partial and Record are two commonly used generics

[Detailed explanation of three generic utility types in TypeScript: Omit, Pick, Exclude](https://github.com/FE-wuhao/TrivialKnowledge/issues/13)

## Difference Between any and unknown in TS

**The main difference between unknown and any is that unknown is more strict: before performing most operations on values of unknown type, we must perform some kind of check. In contrast, we don't need to perform any checks before operating on values of any type.**

Example:

```ts
let foo: any = 123;
console.log(foo.msg); // Valid TS syntax
let a_value1: unknown = foo;   // OK
let a_value2: any = foo;      // OK
let a_value3: string = foo;   // OK


let bar: unknown = 222; // OK 
console.log(bar.msg); // Error
let k_value1: unknown = bar;   // OK
let K_value2: any = bar;      // OK
let K_value3: string = bar;   // Error
```

Since bar is of unknown type (any type of data can be assigned to the unknown type), we cannot be certain if it has a msg property. This cannot pass TS syntax checking. Also, values of unknown type cannot be assigned to variables of types other than any and unknown.

> **Summary:** Both any and unknown are top-level types, but unknown is more strict. Unlike any, which doesn't perform type checking, unknown, due to its unknown nature, doesn't allow property access and doesn't allow assignment to variables with other explicit types.

## TypeScript Compilation Principles

The compilation part of TS mainly consists of the following components:

- **Scanner** (`scanner.ts`)
- **Parser** (`parser.ts`)
- **Binder** (`binder.ts`)
- **Checker** (`checker.ts`)
- **Emitter** (`emitter.ts`)

![](https://user-gold-cdn.xitu.io/2018/12/21/167d0e3f4f7cb536?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

General workflow:

1. SourceCode + Scanner --> Token stream
2. Token stream + Parser --> AST (Abstract Syntax Tree)
3. AST + Binder --> Symbols
4. AST + Symbols + Checker --> Type validation
5. AST + Checker + Emitter --> JavaScript code

Reference articles:

https://juejin.cn/post/6844903745503903758

https://www.studyfe.cn/2019/08/05/typescript/compilationprinciple/