---
description: let 和 const 是 ES6 新增的用于声明变量的关键字，分别用于标识作用域中的变量和常量。
---

# let 和 const

### 变量和常量

{% hint style="info" %}
定义以后可改变的量就是变量，定义后不可改变的量就是常量。
{% endhint %}

也就是说 `let` 定义的变量，定义后是可修改的，`const` 定义的常量定义后不能被修改。我们可以看下两者用 `babel` 编译后的代码有何区别：

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

### let 和 var

`let` 和 `var` 是两种声明变量的方式，二者主要有以下区别：

#### 作用域不同

* `let`  所声明的变量会创建自己的作用域，创建的作用域是定义它的块级代码及其中包括的子块中，且无法自动往全局变量 `window` 上绑定属性。
* `var` 定义的变量，作用域为定义它的函数，或者全局，并且是能自动往全局对象 `window` 上绑定属性的。

这就涉及到了一道被问烂的面试题：

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

为什么打印出了五个5，而不是预期的 0,1,2,3,4 ？因为 `var` 不会创建自己的作用域，而 `js` 本身是没有块级作用域这个概念的，`for` 循环中定义的变量就等于是直接定义在匿名函数中的变量，于是当这5个函数被掏出来执行的时候，循环早已完成，而他们读取的上面一层作用域中存储的变量 `i`，也早已经被累加成了5。

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

#### 变量声明提升

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

#### 暂时性死区

{% hint style="info" %}
只要块级作用域内存在`let`命令，它所声明的变量就“绑定”（binding）这个区域，不再受外部的影响，这种特性被成为暂时性死区。
{% endhint %}

```javascript
var tmp = 123;

if (true) {
  tmp = 'abc'; // ReferenceError: tmp is not defined
  let tmp;
}
```

同样，这个特性也是被浏览器内部的 JS 执行引擎支持和实现的，`babel` 无法支持这种特性的编译，只能简单的将 `let` 编译成 `var`。但是有意思的是，由于 `let` 在 `if` 块中是可以构建自己的独立作用域的，`babel`  将 `tmp` 这个变量换了个名字来实现块级作用域的创建： 

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

这段代码就不用看编译后的效果了，因为如果对 `let` 定义的变量再次重新定义，是在 `babel` 编译阶段就会报错了，不会生成任何内容。

