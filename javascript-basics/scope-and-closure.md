---
description: >-
  几乎所有语言的最基础模型之一就是在变量中存储值，并且在稍后取出或修改这些值。在变量中存储值和取出值的能力，给程序赋予了状态。这就引伸出两个问题：这些变量被存储在哪里？程序如何在需要的时候找到它们？回答这些问题需要一组明确定义的规则，它定义了如何存储变量，以及如何找到这些变量。我们称这组规则为：作用域。
---

# 作用域和闭包

### LHS 和 RHS 查询

{% hint style="info" %}
在说 `javascript` 中的作用域之前，我想应该先了解一下 `LHS` 和 `RHS` 查询，这对于作用域的理解有所帮助。
{% endhint %}

虽然 `javascript` 被认为是一门解释型语言/动态语言，但是它其实是一种编译型的语言。一般来说，需要运行一段 `javascript` 代码，有两个必不可少的东西：**JS 引擎** 和  **编译器。**前者类似于总管的角色，负责整个程序运行时所需的各种资源的调度；后者只是前者的一部分，负责将 `javascript` 源码编译成机器能识别的机器指令，然后交给引擎运行。

#### 编译

在 `javascript` 中，一段源码在被执行之前大概会经历以下三个步骤，这也被称之为 **编译**：

1. **分词/词法分析：**编译器会先将一连串字符打断成（对于语言来说）有意义的片段，称为 token（记号），例如  `var a = 2;`。这段程序很可能会被打断成如下 token：`var`，`a`，`=`，`2`，和 `;`。
2. **解析/语法分析：**编译器将一个 `token` 的流（数组）转换为一个“抽象语法树”（`AST —— Abstract Syntax Tree`），它表示了程序的语法结构。
3. **代码生成：**编译器将上一步中生成的抽象语法树转换为机器指令，等待引擎执行。

#### 执行

编译器一顿操作猛如虎，生成了一堆机器指令，JS 引擎开心地拿到这堆指令，开始执行，这个时候我们要说的 `LHS` 和 `RHS` 就登场了。

`LHS (Left-hand Side)` 和 `RHS (Right-hand Side)` ，是在代码执行阶段 JS 引擎操作变量的两种方式，二者区别就是对变量的查询目的是 **变量赋值** 还是 **查询** 。

**`LHS`** 可以理解为变量在赋值操作符`(=)`的左侧，例如 `a = 1`，当前引擎对变量 `a` 查找的目的是**变量赋值**。这种情况下，引擎不关心变量 `a` 原始值是什么，只要知道有没有这个变量就可以了。

**`RHS`** 可以理解为变量在赋值操作符`(=)`的右侧，例如：`console.log(a)`，其中引擎对变量`a`的查找目的就是 **查询**，它需要找到变量 `a` 对应的实际值是什么。

来看下面这段代码：

```javascript
var a = 2; // LHS 查询
```

这段代码运行时，引擎做了一个 `LHS` 查询，找到 `a` ，并把新值赋给它。再看下面一段：

```javascript
function foo(a) { // LHS 查询
	console.log( a ); // RHS 查询
}

foo( 2 ); // RHS 查询
```

为了执行它，JS 引擎既做了 `LHS` 查询又做了 `RHS` 查询，只不过这里的 `LHS` 比较难发现。

### 什么是作用域

> 简单来说，**作用域** 指程序中定义变量的区域，它决定了当前执行代码对变量的访问权限。

### 词法作用域

词法作用域（`Lexical Scopes`）是 JavaScript 中使用的作用域类型，词法作用域也可以被叫做静态作用域，与之相对的还有动态作用域。



### 相关参考

* [“You Don’t Know JS:” My Understanding of Scopes and Closures](https://medium.com/better-programming/you-dont-know-js-my-understanding-of-scopes-closures-e0d2bfe4c328)
* [Closures - JavaScript \| MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)
*  [JavaScript深入之词法作用域和动态作用域](https://github.com/mqyqingfeng/Blog/issues/3)
* [你不懂 JS —— 作用域与闭包](https://www.kancloud.cn/kancloud/you-dont-know-js-scope-closures/516609)
* [从 LHS 与 RHS 角度浅谈Js变量声明与赋值](https://github.com/MrErHu/blog/issues/12)
* 《你不知道的JavaScript》

