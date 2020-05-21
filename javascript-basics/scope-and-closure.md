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

1. **分词 / 词法分析：**编译器会先将一连串字符打断成（对于语言来说）有意义的片段，称为 token（记号），例如  `var a = 2;`。这段程序很可能会被打断成如下 token：`var`，`a`，`=`，`2`，和 `;`。
2. **解析 / 语法分析：**编译器将一个 `token` 的流（数组）转换为一个“抽象语法树”（`AST —— Abstract Syntax Tree`），它表示了程序的语法结构。
3. **代码生成：**编译器将上一步中生成的抽象语法树转换为机器指令，等待引擎执行。

#### 执行

编译器一顿操作猛如虎，生成了一堆机器指令，JS 引擎开心地拿到这堆指令，开始执行，这个时候我们要说的 `LHS` 和 `RHS` 就登场了。

`LHS (Left-hand Side)` 和 `RHS (Right-hand Side)` ，是在代码执行阶段 JS 引擎操作变量的两种方式，二者区别就是对变量的查询目的是 **变量赋值** 还是 **查询** 。

**`LHS`** 可以理解为变量在赋值操作符`(=)`的左侧，例如 `a = 1`，当前引擎对变量 `a` 查找的目的是**变量赋值**。这种情况下，引擎不关心变量 `a` 原始值是什么，只管将值 `1` 赋给 `a` 变量。

**`RHS`** 可以理解为变量在赋值操作符`(=)`的右侧，例如：`console.log(a)`，其中引擎对变量`a`的查找目的就是 **查询**，它需要找到变量 `a` 对应的实际值是什么，然后才能将它打印出来。

来看下面这段代码：

```javascript
var a = 2; // LHS 查询
```

这段代码运行时，引擎做了一个 `LHS` 查询，找到 `a` ，并把新值 `2` 赋给它。再看下面一段：

```javascript
function foo(a) { // LHS 查询
	console.log( a ); // RHS 查询
}

foo( 2 ); // RHS 查询
```

为了执行它，JS 引擎既做了 `LHS` 查询又做了 `RHS` 查询，只不过这里的 `LHS` 比较难发现。

总之，引擎想对变量进行获取 / 赋值，就离不开 `LHS` 和 `RHS` ，然而这两个操作只是手段，到哪里去获取变量才是关键。`LHS` 和 `RHS` 获取变量的位置就是 **作用域**。

### 什么是作用域

> 简单来说，**作用域** 指程序中定义变量的区域，它决定了当前执行代码对变量的访问权限。

`javascript` 中大部分情况下，只有两种作用域类型：

* **全局作用域：**全局作用域为程序的最外层作用域，一直存在。
* **函数作用域：**函数作用域只有函数被定义时才会创建，包含在父级函数作用域 / 全局作用域内。

{% hint style="info" %}
由于作用域的限制，每段独立的执行代码块只能访问自己作用域和外层作用域中的变量，无法访问到内层作用域的变量。
{% endhint %}

```javascript
/* 全局作用域开始 */
var a = 1;

function func () { /* func 函数作用域开始 */
    var a = 2;
    console.log(a);
}                  /* func 函数作用域结束 */

func(); // => 2

console.log(a); // => 1

/* 全局作用域结束 */
```

### 作用域链

上面代码示范中，可执行代码块是能够在自己的作用域中找到变量的，那么如果在自己的作用域中找不到目标变量，程序能否正常运行？来看下面的代码：

```javascript
function foo(a) {
	var b = a * 2;

	function bar(c) {
		console.log( a, b, c );
	}

	bar(b * 3);
}

foo(2); // 2 4 12
```

结合前面的知识我们知道，在 `bar` 函数内部，会做三次 `RHS` 查询从而分别获取到 `a` `b` `c` 三个变量的值。`bar` 内部作用域中只能获取到变量 `c` 的值，`a` 和 `b` 都是从外部 `foo` 函数的作用域中获取到的。

{% hint style="info" %}
当可执行代码内部访问变量时，会先查找本地作用域，如果找到目标变量即返回，否则会去父级作用域继续查找...一直找到全局作用域。我们把这种作用域的嵌套机制，称为 **作用域链**。
{% endhint %}

![&#x5D4C;&#x5957;&#x4F5C;&#x7528;&#x57DF;](../.gitbook/assets/scopeline.png)

用图片表示，上述代码一共有三层作用域嵌套，分别是：

1. 全局作用域
2. `foo` 作用域
3. `bar` 作用域

{% hint style="warning" %}
需要注意，函数参数也在函数作用域中。
{% endhint %}

### 词法作用域

明白了作用域和作用域链的概念，我们来看词法作用域。

 **词法作用域（`Lexical Scopes`）**是 `javascript` 中使用的作用域类型，**词法作用域** 也可以被叫做 **静态作用域**，与之相对的还有 **动态作用域**。那么 `javascript` 使用的 **词法作用域** 和 **动态作用域** 的区别是什么呢？看下面这段代码：

```javascript
var value = 1;

function foo() {
    console.log(value);
}

function bar() {
    var value = 2;
    foo();
}

bar();

// 结果是 ???
```

上面这段代码中，一共有三个作用域：

* 全局作用域
* `foo` 的函数作用域
* `bar` 的函数作用域

一直到这边都好理解，可是 `foo` 里访问了本地作用域中没有的变量 `value` 。根据前面说的，引擎为了拿到这个变量就要去 `foo` 的上层作用域查询，那么 `foo` 的上层作用域是什么呢？是它 **调用时** 所在的 bar 作用域？还是它 **定义时** 所在的全局作用域？

这个关键的问题就是 `javascript` 中的作用域类型——**词法作用域。**

{% hint style="info" %}
**词法作用域**，就意味着函数 **被定义的时候**，它的作用域就已经确定了，和拿到哪里执行没有关系，因此词法作用域也被称为 “**静态作用域**”。
{% endhint %}

如果是动态作用域类型，那么上面的代码运行结果应该是 `bar` 作用域中的 `2` 。也许你会好奇什么语言是动态作用域？`bash` 就是动态作用域，感兴趣的小伙伴可以了解一下。

### 块级作用域

什么是块级作用域呢？简单来说，花括号内 `{...}` 的区域就是块级作用域区域。

很多语言本身都是支持块级作用域的。上面我们说，`javascript` 中大部分情况下，只有两种作用域类型：**全局作用域** 和 **函数作用域**，那么 `javascript` 中有没有块级作用域呢？来看下面的代码：

```javascript
if (true) {
    var a = 1;
}

console.log(a); // 结果???
```

运行后会发现，结果还是 `1`，花括号内定义并赋值的 a 变量跑到全局了。这足以说明，`javascript` 不是原生支持块级作用域的，起码创作者创造这门语言的时候压根就没把块级作用域的事情考虑进去...（出来背锅！！）

但是 `ES6` 标准提出了使用 `let` 和 `const` 代替 `var` 关键字，来“创建块级作用域”。也就是说，上述代码改成如下方式，块级作用域是有效的：

```javascript
if (true) {
    let a = 1;
}

console.log(a); // ReferenceError
```

{% hint style="info" %}
关于 `let` 和 `const` 的更多细节，进入 [传送门](https://mitianyi.gitbook.io/frontend-interview-guide/es6/let-and-const)
{% endhint %}

### 创建作用域

在 `javascript` 中，我们有几种创建 / 改变作用域的手段：

1. 定义函数，创建函数作用（推荐）：

   ```javascript
   function foo () {
       // 创建了一个 foo 的函数作用域
   }
   ```

2. 使用 `let` 和 `const` 创建块级作用域（推荐）：

   ```javascript
   for (let i = 0; i < 5; i++) {
       console.log(i);
   }

   console.log(i); // ReferenceError
   ```

3. `try catch` 创建作用域（不推荐）,`err` 仅存在于 `catch` 子句中：

   ```javascript
   try {
   	undefined(); // 强制产生异常
   }
   catch (err) {
   	console.log( err ); // TypeError: undefined is not a function
   }

   console.log( err ); // ReferenceError: `err` not found
   ```

4. 使用 eval “欺骗” 词法作用域（不推荐）：

   ```javascript
   function foo(str, a) {
   	eval( str );
   	console.log( a, b );
   }

   var b = 2;

   foo( "var b = 3;", 1 ); // 1 3
   ```

5. 使用 `with` 欺骗词法作用域（不推荐）：

   ```javascript
   function foo(obj) {
   	with (obj) {
   		a = 2;
   	}
   }

   var o1 = {
   	a: 3
   };

   var o2 = {
   	b: 3
   };

   foo( o1 );
   console.log( o1.a ); // 2

   foo( o2 );
   console.log( o2.a ); // undefined
   console.log( a ); // 2 -- 全局作用域被泄漏了！
   ```

{% hint style="success" %}
总结下来，能够使用的创建作用域的方式就两种：**定义函数创建** 和 **`let const` 创建**。
{% endhint %}

### 作用域的运用

### 闭包

### 闭包的运用

### 内存泄露

### 内存泄露的解决方案

### 相关参考

* [“You Don’t Know JS:” My Understanding of Scopes and Closures](https://medium.com/better-programming/you-dont-know-js-my-understanding-of-scopes-closures-e0d2bfe4c328)
* [Closures - JavaScript \| MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)
*  [JavaScript深入之词法作用域和动态作用域](https://github.com/mqyqingfeng/Blog/issues/3)
* [你不懂 JS —— 作用域与闭包](https://www.kancloud.cn/kancloud/you-dont-know-js-scope-closures/516609)
* [从 LHS 与 RHS 角度浅谈Js变量声明与赋值](https://github.com/MrErHu/blog/issues/12)
* 《你不知道的JavaScript》

