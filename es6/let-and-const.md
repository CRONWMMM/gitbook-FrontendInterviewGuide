---
description: let 和 const 是 ES6 新增的用于声明变量和常量的关键字，他们的作用是什么？它们又有什么特性？它们与 var 定义的变量又有何区别？
---

# let 和 const

### let 和 const

{% hint style="info" %}
定义以后可改变的量就是变量，定义后不可改变的量就是常量。
{% endhint %}

#### 1. 变量和常量

在 `ES6` 中，使用 `let` 命令定义变量，使用 `const` 命令定义常量，也就是说 `let` 定义后变量是可修改的，`const` 定义后的常量不能被修改。这一特性目前被大多数浏览器原生支持，但是针对少部分不能支持的浏览器，我们可以使用 `babel` 将它编译成 `ES5` 语法，下面可以看下两者用 `babel` 编译后的代码有何区别：

```javascript
// ES6
let a = 1;
const b = 2;
```

```javascript
// babel 编译后
"use strict";

var a = 1;
var b = 2;
```

感觉没区别啊？？不要急，再加一段代码：

```javascript
// ES6
let a = 1;
const b = 2;
b = 3;
```

```javascript
// babel 编译后
"use strict";

function _readOnlyError(name) { throw new Error("\"" + name + "\" is read-only"); }

var a = 1;
var b = 2;
b = (_readOnlyError("b"), 3);
```

这样应该很容易看出区别了，当我们对一个常量进行变值操作，就会抛出一个错误告诉你：这个值是只读的。

{% hint style="info" %}
`b = (_readOnlyError("b"), 3)`这种操作我也没见过，不过猜测应该类似于 `b = _readOnlyError("b") && 3`
{% endhint %}

{% hint style="warning" %}
**注意：**由于`const` 定义的值是不可变的，这点在用 `const` 定义引用类型的时候要特别注意！如果是引用类型的 `const` 值，改变其中的属性是可行的，但是通常不建议这么做。
{% endhint %}

```javascript
const obj = {};
obj.a = 1;
obj.b = 2;
obj; // => {a: 1, b: 2}

obj = {}; // => 报错
```

#### 定义就要初始化

由于 `const` 定义的常量，定义后就不能修改的特性，决定了它定义的时候必须就初始化，否则就报错；而 `let` 则没有这种限制，它定义的变量完全可以在后面再初始化：

```javascript
let a;
a = 1;

const b; // Uncaught SyntaxError: Missing initializer in const declaration
b = 2;
```

{% hint style="warning" %}
上面这段代码由于直接违反了 `const` 的语法特性，因此在 `babel` 编译阶段就无法通过
{% endhint %}

### let 和 var

`let` 和 `var` 是两种声明变量的方式，二者主要有以下区别：

#### 1. 作用域不同

* `let`  所声明的变量会创建自己的块级作用域，创建的作用域是定义它的块级代码及其中包括的子块中，且无法自动往全局变量 `window` 上绑定属性。
* `var` 定义的变量，作用域为定义它的函数，或者全局，并且是能自动往全局对象 `window` 上绑定属性的。

能否创建自己的块级作用域这一差别，就会涉及到一道被问烂的面试题：

```javascript
var result = [];
(function () {
  for (var i = 0; i < 5; i++) {
    result.push(function () {
      console.log(i);
    });
  }
})();
result.forEach(function (item) {
  item()
});
// => 打印出五个 5
```

为什么打印出了五个5，而不是预期的 0,1,2,3,4 ？因为 `var` 不会创建自己的作用域，而 `js` 本身又是没有块级作用域这个概念的，`for` 循环中定义的变量就等于是直接定义在匿名函数中的变量，于是当这5个函数被掏出来执行的时候，循环早已完成，而函数读取的上面一层作用域中存储的变量 `i`，也早已经被累加成了5。

然而，这个问题只要将 `var` 关键字换成 `let` 就迎刃而解：

```javascript
var result = [];
(function () {
  for (let i = 0; i < 5; i++) {
    result.push(function () {
      console.log(i);
    });
  }
})();
result.forEach(function (item) {
  item()
}); // => 0,1,2,3,4
```

我们可以看看这段使用 `let` 的代码，最后被 `babel` 转译成什么样：

```javascript
"use strict";

var result = [];

(function () {
  var _loop = function _loop(i) {
    result.push(function () {
      console.log(i);
    });
  };

  for (var i = 0; i < 5; i++) {
    _loop(i);
  }
})();

result.forEach(function (item) {
  item();
});
```

从上面的代码我们就可以看出，`let` 创建作用域的方式，其实就是**创建了一个函数，在函数内定义一个同名变量并于外部将这个变量传入其中**，以此达到创建作用域的目的。

#### 2. 变量声明提升

* `let` 定义的变量不会进行变量声明提升操作，也就是说在访问该变量之前必须要先定义。
* `var` 定义的变量存在变量声明提升，因此在变量定义前就能访问到变量，值是 `undefined`。

```javascript
console.log(a); // undefined
var a = 1;

console.log(b); // Uncaught ReferenceError: Cannot access 'b' before initialization
let b = 2;
```

同样我们来看下上面这段代码被编译后的样子：

```javascript
"use strict";

console.log(a);
var a = 1;

console.log(b);
var b = 2;
```

看起来好像 `Babel` 无法编译这种阻止变量声明提升的语法，`let` 声明的变量无法提升的特性应该是浏览器内部的 JS 执行引擎支持和实现的。

#### 3. 暂时性死区

{% hint style="info" %}
只要块级作用域内存在`let / const`命令，它所声明的变量 / 常量就“绑定”（binding）这个区域，不再受外部的影响。ES6 明确规定，如果区块中存在`let`和`const`命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域，凡是在声明之前就使用这些变量，就会报错。这种特性也被成为暂时性死区。 
{% endhint %}

```javascript
var tmp = 123;

if (true) {
  tmp = 'abc'; // ReferenceError: tmp is not defined
  let tmp;
}
```

同样，这个特性也是被浏览器内部的 JS 执行引擎支持和实现的，`babel` 无法支持这种特性的编译，只能简单的将 `let` 编译成 `var`。但是有意思的是，由于 `let` 在 `if` 块中是可以构建自己的独立作用域的，`babel`  将 `tmp` 这个变量换了个名字来模拟实现块级作用域的创建： 

```javascript
"use strict";

var tmp = 123;

if (true) {
  _tmp = 'abc';

  var _tmp;
}
```

#### 可否重复定义

* `let` 定义的变量，一旦定义便不允许被重新定义。
* `var` 定义的变量，可以被重新定义。

```javascript
var a = 1;
var a = 2;

let b = 3;
let b = 4; // Uncaught SyntaxError: Identifier 'b' has already been declared
```

{% hint style="warning" %}
上面这段代码由于直接违反了 `let` 定义的变量无法重新定义的语法特性，因此同样在 `babel` 编译阶段就无法通过。
{% endhint %}

### 总结

这块内容并不复杂，我们可以做一个简单的总结：

**`let` 特性：**

* 创建块级作用域。
* 定义后不能重新定义。
* 不存在变量提升。
* 存在暂时性死区。
* 全局作用域下定义时不会被挂载到顶层对象上（`window对象 / global 对象`）

  ```javascript
  // 浏览器环境
  var a = 1;
  window.a; // => 1

  let b = 2;
  window.b; // => undefined
  ```

**`const` 特性：**

* 同 `let`
* 一旦初始化赋值，后面不能被修改
* 定义时就必须初始化

