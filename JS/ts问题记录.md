## TS的优势

### JavaScript 与 TypeScript 的区别

TypeScript 是 JavaScript 的超集，扩展了 JavaScript 的语法，因此现有的 JavaScript 代码可与 TypeScript 一起工作无需任何修改，TypeScript 通过类型注解提供编译时的静态类型检查。

TypeScript 可处理已有的 JavaScript 代码，并只对其中的 TypeScript 代码进行编译。

![img](https://www.runoob.com/wp-content/uploads/2019/01/ts-2020-11-26-1.png)

### 优势

* 编译时的静态类型检查

* 维护、重构方便
* 多人开发，代码质量更好控制
* 可以无需任何修改，同JS一起工

## 装饰器

TypeScript 中的 **装饰器（Decorator）** 是一种特殊语法，用于通过声明式的方式修改类、方法、属性或参数的行为。装饰器本质上是一个函数，它在代码编译阶段（而非运行时）被调用，通过 `@` 符号附加到目标上。

- **作用目标**：可以装饰类、类的方法、访问器（getter/setter）、属性、方法参数。
- **执行时机**：在代码编译时执行（由 TypeScript 编译器处理），而非运行时。
- **本质**：装饰器是一个函数，接收特定的元数据参数，返回修改后的目标或 void。

## 泛型

https://juejin.cn/post/6844904184894980104#heading-0

Partial，Record，这两个用到了

[详解TypeScript中的三个泛型工具类：Omit、Pick、Exclude](https://github.com/FE-wuhao/TrivialKnowledge/issues/13)

## TS 中 any 和 unkown 的区别

**unknown 和 any 的主要区别是 unknown 类型会更加严格：在对 unknown 类型的值执行大多数操作之前，我们必须进行某种形式的检查。而在对 any 类型的值执行操作之前，我们不必进行任何检查。**

举例说明：

```ts
let foo: any = 123;
console.log(foo.msg); // 符合TS的语法
let a_value1: unknown = foo;   // OK
let a_value2: any = foo;      // OK
let a_value3: string = foo;   // OK


let bar: unknown = 222; // OK 
console.log(bar.msg); // Error
let k_value1: unknown = bar;   // OK
let K_value2: any = bar;      // OK
let K_value3: string = bar;   // Error
```

因为bar是一个未知类型(任何类型的数据都可以赋给 unknown 类型)，所以不能确定是否有msg属性。不能通过TS语法检测；而 unkown 类型的值也不能将值赋给 any 和 unkown 之外的类型变量

> **总结：** any 和 unknown 都是顶级类型，但是 unknown 更加严格，不像 any 那样不做类型检查，反而 unknown 因为未知性质，不允许访问属性，不允许赋值给其他有明确类型的变量

## TS的编译原理

TS的编译部分主要由以下几个部分构成：

- **Scanner 扫描器**（`scanner.ts`）
- **Parser 解析器**（`parser.ts`）
- **Binder 绑定器**（`binder.ts`）
- **Checker 检查器**（`checker.ts`）
- **Emitter 发射器**（`emitter.ts`）

![](https://user-gold-cdn.xitu.io/2018/12/21/167d0e3f4f7cb536?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

大概工作流：

1. SourceCode（源码） + 扫描器 --> Token 流
2. Token 流 + 解析器 --> AST（抽象语法树）
3. AST + 绑定器 --> Symbols（符号）
4. AST + 符号 + 检查器 --> 类型验证
5. AST + 检查器 + 发射器 --> JavaScript 代码

参考文章：

https://juejin.cn/post/6844903745503903758

https://www.studyfe.cn/2019/08/05/typescript/compilationprinciple/