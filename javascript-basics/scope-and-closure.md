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

### 作用域的应用场景

作用域的一个常见运用场景之一，就是 **模块化**。

{% hint style="info" %}
由于 `javascript` 并未原生支持模块化导致了很多令人口吐芬芳的问题，比如 **全局作用域污染** 和 **变量名冲突**，代码结构臃肿且复用性不高。在正式的模块化方案出台之前，开发者为了解决这类问题，想到了 **使用函数作用域来创建模块** 的方案。
{% endhint %}

```javascript
function module1 () {
    var a = 1;
    console.log(a);
}

function module2 () {
    var a = 2;
    console.log(a);
}

module1(); // => 1
module2(); // => 2
```

上面的代码中，构建了 `module1` 和 `module2` 两个代表模块的函数，两个函数内分别定义了一个同名变量 `a` ，由于函数作用域的隔离性质，这两个变量被保存在不同的作用域中（不嵌套），JS 引擎在执行这两个函数时会去不同的作用域中读取，并且外部作用域无法访问到函数内部的 `a` 变量。这样一来就巧妙地解决了 **全局作用域污染** 和 **变量名冲突** 的问题；并且，由于函数的包裹写法，这种方式看起来封装性好多了。

然而上面的函数声明式写法，看起来还是有些冗余，更重要的是，`module1` 和 `module2` 的函数名本身就已经对全局作用域造成了污染。我们来继续改写：

```javascript
// module1.js
(function () {
    var a = 1;
    console.log(a);
})();

// module2.js
(function () {
    var a = 2;
    console.log(a);
})();
```

将函数声明改写成 **立即调用函数表达式（`Immediately Invoked Function Expression` 简写 `IIFE`**），封装性更好，代码也更简洁，解决了模块名污染全局作用域的问题。

{% hint style="warning" %}
函数声明和函数表达式，最简单的区分方法，就是看是不是 `function` 关键字开头：是 `function` 开头的就是函数声明，否则就是函数表达式。
{% endhint %}

上面的代码采用了 `IIFE` 的写法，已经进化很多了，我们可以再把它强化一下，强化成后浪版，赋予它判断外部环境的权利——**选择的权力**。

```javascript
(function (global) {
    if (global...) {
        // is browser
    } else if (global...) {
        // is nodejs
    }
})(window);
```

让后浪继续奔涌，我们的想象力不足以想象  `UMD` 模块化的代码：

```javascript
// UMD 模块化
(function (root, factory) {
    if (typeof define === 'function' && define.amd) {
        // AMD
        define(['jquery'], factory);
    } else if (typeof exports === 'object') {
        // Node, CommonJS-like
        module.exports = factory(require('jquery'));
    } else {
        // Browser globals (root is window)
        root.returnExports = factory(root.jQuery);
    }
}(this, function ($) {
    //    methods
    function myFunc(){};

    //    exposed public method
    return myFunc;
}));
```

我看着作用域的模块化应用场景，真的是满怀羡慕。如果你也和我一样羡慕并且，想了解更多关于模块化的东西，请进入 [传送门](https://mitianyi.gitbook.io/frontend-interview-guide/javascript-basics/modularization)。

### 闭包

说完了作用域，我们来说说 **闭包**。

> 能够访问其他函数内部变量的函数，被称为 **闭包**。

上面这个定义比较难理解，简单来说，**闭包就是函数内部定义的函数，被返回了出去并在外部调用**。我们可以用代码来表述一下：

```javascript
function foo() {
	var a = 2;

	function bar() {
		console.log( a );
	}

	return bar;
}

var baz = foo();

baz(); // 这就形成了一个闭包
```

我们可以简单剖析一下上面代码的运行流程：

1. 编译阶段，变量和函数被声明，作用域即被确定。
2. 运行函数 `foo()`，此时会创建一个 `foo` 函数的执行上下文，执行上下文内部存储了 `foo` 中声明的所有变量函数信息。
3. 函数 `foo` 运行完毕，将内部函数 `bar` 的引用赋值给外部的变量 `baz` ，此时 `baz` 指针指向的还是 `bar` ，因此哪怕它位于 `foo` 作用域之外，它还是能够获取到 `foo` 的内部变量。
4. `baz` 在外部被执行，`baz` 的内部可执行代码 `console.log` 向作用域请求获取 `a` 变量，本地作用域没有找到，继续请求父级作用域，找到了 `foo` 中的 `a` 变量，返回给 `console.log`，打印出 `2`。

闭包的执行看起来像是开发者使用的一个小小的 “作弊手段” ——**绕过了作用域的监管机制，从外部也能获取到内部作用域的信息**。闭包的这一特性极大地丰富了开发人员的编码方式，也提供了很多有效的运用场景。

### 闭包的应用场景

{% hint style="info" %}
闭包的应用，大多数是在需要维护内部变量的场景下。
{% endhint %}

#### 单例模式

单例模式是一种常见的涉及模式，它保证了一个类只有一个实例。实现方法一般是先判断实例是否存在，如果存在就直接返回，否则就创建了再返回。单例模式的好处就是避免了重复实例化带来的内存开销：

```javascript
// 单例模式
function Singleton(){
    this.data = 'singleton';
}

Singleton.getInstance = (function () {
    var instance;
    
    return function(){
        if (instance) {
            return instance;
        } else {
            instance = new Singleton();
            return instance;
        }
    }
})();

var sa = Singleton.getInstance();
var sb = Singleton.getInstance();
console.log(sa === sb); // true
console.log(sa.data); // 'singleton'
```

#### 模拟私有属性

`javascript` 没有 `java` 中那种 `public` `private` 的访问权限控制，对象中的所用方法和属性均可以访问，这就造成了安全隐患，内部的属性任何开发者都可以随意修改。虽然语言层面不支持私有属性的创建，但是我们可以用闭包的手段来模拟出私有属性：

```javascript
// 模拟私有属性
function getGeneratorFunc () {
    var _name = 'John';
    var _age = 22;
    
    return function () {
        return {
            getName: function () {return _name;},
            getAge: function() {return _age;}
        };
    };
}

var obj = getGeneratorFunc()();
obj.getName(); // John
obj.getAge(); // 22
obj._age; // undefined
```

#### 柯里化

> 柯里化（`currying`），是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数而且返回结果的新函数的技术。

这个概念有点抽象，实际上柯里化是高阶函数的一个用法，`javascript` 中常见的 `bind` 方法就可以用柯里化的方法来实现：

```javascript
Function.prototype.myBind = function (context = window) {
    if (typeof this !== 'function') throw new Error('Error');
    let selfFunc = this;
    let args = [...arguments].slice(1);
    
    return function F () {
        // 因为返回了一个函数，可以 new F()，所以需要判断
        if (this instanceof F) {
            return new selfFunc(...args, arguments);
        } else  {
            // bind 可以实现类似这样的代码 f.bind(obj, 1)(2)，所以需要将两边的参数拼接起来
            return selfFunc.apply(context, args.concat(arguments));
        }
    }
}
```

柯里化的优势之一就是 **参数的复用**，它可以在传入参数的基础上生成另一个全新的函数，来看下面这个类型判断函数：

```javascript
function typeOf (value) {
    return function (obj) {
        const toString = Object.prototype.toString;
        const map = {
            '[object Boolean]'	 : 'boolean',
            '[object Number]' 	 : 'number',
            '[object String]' 	 : 'string',
            '[object Function]'  : 'function',
            '[object Array]'     : 'array',
            '[object Date]'      : 'date',
            '[object RegExp]'    : 'regExp',
            '[object Undefined]' : 'undefined',
            '[object Null]'      : 'null',
            '[object Object]' 	 : 'object'
        };
        return map[toString.call(obj)] === value;
    }
}

var isNumber = typeOf('number');
var isFunction = typeOf('function');
var isRegExp = typeOf('regExp');

isNumber(0); // => true
isFunction(function () {}); // true
isRegExp({}); // => false
```

通过向 `typeOf` 里传入不同的类型字符串参数，就可以生成对应的类型判断函数，作为语法糖在业务代码里重复使用。

### 闭包的问题

闭包的使用场景如此广泛，那我们是不是可以大量使用呢？不可以，闭包过度使用会导致性能问题，还是看上面的代码：

```javascript
function foo() {
	var a = 2;

	function bar() {
		console.log( a );
	}

	return bar;
}

var baz = foo();

baz(); // 这就形成了一个闭包
```

乍一看，好像没什么问题，然而，它却有可能导致 **内存泄露**。

我们知道，`javascript` 内部的垃圾回收机制用的是引用计数收集：即当内存中的一个变量被引用一次，计数就加一。垃圾回收机制会以固定的时间轮询这些变量，将计数为 `0` 的变量标记为失效变量并将之清除从而释放内存。

上述代码中，理论上来说 `foo` 函数作用域隔绝了外部环境，所有变量引用都在函数内部完成，`foo` 运行完成以后，内部的变量就应该被销毁，内存被回收。然而闭包导致了全局作用域始终存在一个 `baz` 的变量在引用着 `foo` 内部的 `bar` 函数，这就意味着 `foo` 内部定义的 `bar` 函数引用数始终为 `1`，垃圾运行机制就无法把它销毁。更糟糕的是，`bar` 有可能还要使用到父作用域 `foo` 中的变量信息，那它们自然也不能被销毁... JS 引擎无法判断你什么时候还会调用闭包函数，只能一直让这些数据占用着内存。

{% hint style="danger" %}
这种由于闭包使用过度而导致的内存占用无法释放的情况，我们称之为：**内存泄露**。
{% endhint %}

### 内存泄露

{% hint style="danger" %}
**内存泄露** 是指当一块内存不再被应用程序使用的时候，由于某种原因，这块内存没有返还给操作系统或者内存池的现象。内存泄漏可能会导致应用程序卡顿或者崩溃。
{% endhint %}

### 内存泄露的排查手段

### 相关参考

* [“You Don’t Know JS:” My Understanding of Scopes and Closures](https://medium.com/better-programming/you-dont-know-js-my-understanding-of-scopes-closures-e0d2bfe4c328)
* [Closures - JavaScript \| MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)
*  [JavaScript深入之词法作用域和动态作用域](https://github.com/mqyqingfeng/Blog/issues/3)
* [你不懂 JS —— 作用域与闭包](https://www.kancloud.cn/kancloud/you-dont-know-js-scope-closures/516609)
* [从 LHS 与 RHS 角度浅谈Js变量声明与赋值](https://github.com/MrErHu/blog/issues/12)
* 《你不知道的JavaScript》

