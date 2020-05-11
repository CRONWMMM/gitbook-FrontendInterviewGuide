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

这个对比就涉及到 `let` 的作用了，估计大家早就清楚了，这边简要阐述一下：

#### 作用域不同

* `let`  所声明的变量，作用域是定义它的块级代码及其中包括的子块中，且无法自动往全局变量 window 上绑定属性。
* `var` 定义的变量，作用域为定义它的函数，或者全局，并且是能自动往全局对象 window 上绑定属性的。

### 变量声明提升及暂时性死区

* let 定义的变量不会进行变量声明提升操作，这一特性也被成为暂时性死区。
* var 定义的变量存在变量声明提升，因此在变量定义前就能访问到变量。

