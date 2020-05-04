---
description: >-
  一门语言发展壮大的毕竟之路就是模块化的出现，很多语言天生设计中就带有这一特性，可惜的是 JS 从一开始就不是天生支持模块化的，变量污染和命名冲突问题让众多
  js 开发者不得不寻求一种实现 js 模块化的方法。从最初的自执行函数，到 AMD / CMD ，再到后来的 Commonjs 规范，以及 ES6 中出现的
  ESModule，语言的发展让我们可以选择不同的模块化技术来应对的不同场景。
---

# 模块化

### 为什么使用模块化？

* 解决变量间相互污染的问题，以及变量命名冲突的问题
* 提高代码的可维护性、可拓展性和复用性

### 自执行函数实现的模块化

自执行函数实现模块化的方法非常简单：

```javascript
// 自执行函数实现模块化
(function () {
    var a = 1;
    console.log(a); // 1
})();

(function () {
    var a = 2;
    console.log(a); // 2
})();
```

自执行函数本质上是通过函数作用域解决了命名冲突、污染全局作用域的问题。

### AMD 、CMD 和 UMD

#### AMD 和 CMD

> AMD 和 CMD 是非官方的两种 js 模块化规范，AMD 标准的代表框架是 RequireJS ，CMD 标准的代表框架是 SeaJS。

```javascript
// AMD
define(['./a', './b'], function(a, b) {
  // 加载模块完毕可以使用
  a.do();
  b.do();
});

// CMD
define(function(require, exports, module) {
  // 加载模块
  // 可以把 require 写在函数体的任意地方实现延迟加载
  var a = require('./a');
  a.doSomething();
});
```

现在 ESModule 和 CommonJS 已经分别统一了浏览器端和 Node 端的模块加载， AMD 和 CMD 使用的比较少，不过作为很多老项目使用的模块化方案，还是值得了解一下的。

AMD 和 CMD 相比，很大的一个区别就是引入模块的时机：

* AMD 是前置依赖，也就是说，目标模块代码执行前，必须保证所有的依赖都被引入并且执行。
* CMD 是后置依赖，也就是说，只有在目标代码中手动执行 `require(..)` 的时候，相关依赖才会被加载并执行。

#### UMD

```javascript
// UMD
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

从代码中不难看出，其实 `UMD` 就是一种通用的模块化方式，将 `AMD` 和 `CMD` 以及全局注册的方式做了整合而已，我们熟悉的 `jQuery` 和很多的工具库都是使用这种模块化的方式进行引入。

### CommonJS 实现模块化

> `CommonJS` 是的 NodeJS 所使用的一种服务端的模块化规范，它将每一个文件定义为一个 `module` ，模块必须通过 `module.exports` 导出对外的变量或接口，通过 `require()` 来导入其他模块的输出到当前模块作用域中

使用方式一：

```javascript
// a.js
const a = 1;
const func = function () {
    return a + 1;
}
// 将 func 作为一个模块导出
module.exports = func;


// main.js
const func = require('./a.js');
console.log(func());
```

使用方式二：

```javascript
// a.js
const a = 1;
const func = function () {
    return a + 1;
}
// 将 func 作为模块的一个属性导出
module.exports.func = func;


// main.js
const { func } = require('./a.js');
console.log(func());
```

使用方式三：

```javascript
// a.js
const a = 1;
const func = function () {
    return a + 1;
}
// 将 func 作为模块的一个属性导出，等同于上面一中写法
module.exports = {
    func
};


// main.js
const { func } = require('./a.js');
console.log(func());
```

### ESModule 实现的模块化



### 相关参考

* [JavaScript 标准参考教程](https://javascript.ruanyifeng.com/nodejs/module.html)
* [前端面试之道](https://juejin.im/book/5bdc715fe51d454e755f75ef/section/5bdd0d83f265da615f76ba57#heading-6)

