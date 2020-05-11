---
description: let 和 const 是 ES6 新增的用于声明变量的关键字，分别用于标识作用域中的变量和常量。
---

# let 和 const

### 变量和常量

> 定义以后可改变的量就是变量，定义后不可改变的量就是常量。

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



