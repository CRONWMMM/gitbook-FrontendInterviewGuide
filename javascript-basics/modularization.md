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

{% hint style="info" %}
[AMD](https://github.com/amdjs/amdjs-api/wiki/AMD) 是"Asynchronous Module Definition"的缩写，意思就是"异步模块定义" ，它采用异步方式加载模块，模块的加载不影响它后面语句的运行。所有依赖这个模块的语句，都定义在一个回调函数中，等到所有依赖项都加载完成之后，这个回调函数才会运行。
{% endhint %}

{% hint style="info" %}
 [CMD](https://github.com/seajs/seajs/issues/242) 是"Common Module Definition"的缩写，意思就是"公共模块定义"。CMD 可以使用 require 同步加载依赖，也可以使用 require.async 来异步加载依赖。
{% endhint %}

{% hint style="info" %}
AMD 和 CMD 都是非官方的两种 js 模块化规范，AMD 标准的代表框架是 RequireJS ，CMD 标准的代表框架是 SeaJS。
{% endhint %}

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
  
  // 也可以使用 require.async 来延迟加载
  require.async('./b', function(b) {
    b.doSomething();
  });
});
```

现在 `ESModule` 和 `CommonJS` 已经分别统一了浏览器端和 Node 端的模块加载， `AMD` 和 `CMD` 使用的比较少，不过作为很多老项目使用的模块化方案，还是值得了解一下的。

`AMD` 和 `CMD` 相比，很大的一个区别就是引入模块的时机，`AMD` 是前置依赖，也就是说，目标模块代码执行前，必须保证所有的依赖都被引入并且执行。`CMD` 是后置依赖，也就是说，只有在目标代码中手动执行 `require(..)` 的时候，相关依赖才会被加载并执行。

还有一个区别就是引入模块的方式，`AMD` 的定位是浏览器环境，所以是异步引入；而 `CMD` 的定位是浏览器环境和 Node 环境，它可以使用 `require` 进行同步引入，也可以使用 `require.async` 的方式进行异步引入。

#### UMD

从下面的代码中不难看出，其实 `UMD` 就是一种通用的模块化方式，它将 `AMD` 和 `CMD` 以及全局注册的方式做了整合而已，我们熟悉的 `jQuery` 和很多的工具库都是使用这种模块化的方式进行引入。

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

### CommonJS 实现模块化

{% hint style="info" %}
 `CommonJS` 是的 NodeJS 所使用的一种服务端的模块化规范，它将每一个文件定义为一个 `module` ，模块必须通过 `module.exports` 导出对外的变量或接口，通过 `require()` 来导入其他模块的输出到当前模块作用域中。
{% endhint %}

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

使用方式四：

```javascript
// a.js
const a = 1;
const func = function () {
    return a + 1;
}
// 利用 Node 提供的便捷写法 exports 来导出模块
exports.func = func;


// main.js
const { func } = require('./a.js');
console.log(func());
```

`CommonJS` 具有如下特点：

* 所有代码都运行于模块作用域，不会污染全局。
* 使用同步的方式加载，也就是说，只有加载完成才能执行后面的操作，这点和 `AMD` 不同，由于 `CommonJS` 的模块化是用在 Node 端也就是服务端，模块加载的时间损耗只是磁盘读取，这个加载速度是很快的，所以可以使用同步的方式。
* `CommonJS` 支持动态导入的方式,，比如：``require(`./${path}.js`)``
* 模块可以多次加载，但是只会在第一次加载时运行一次，然后加载结果会被缓存，后面再次加载会直接读取缓存结果，如果想让模块重新执行，就必须清除缓存。
* `CommonJS` 模块输出的是一个**值的拷贝，**这一点会在下面的 ESModule 和 CommonJS 对比中详细说明。
* 模块的加载顺序，按照其在代码中出现的顺序。

我们可以来模拟一个简化版 `CommonJS` 的实现：

1. 每一个模块内部都有一个 module 对象，代表当前模块，它需要具有以下属性：

   * `module.id` 模块的唯一标识符
   * `module.filename` 模块的文件名、
   * `module.loaded` 返回一个布尔值，代表模块是否加载完成
   * `module.parent` 返回一个对象，代表调用该模块的父模块
   * `module.children` 返回一个数组，内容为这个模块所依赖的其他模块
   * `module.exports` 最重要的一个，表示模块的对外输出内容

   ```javascript
   function Module (id, parent, children) {
       this.id = id;
       this.filename = 'filename.js';
       this.loaded = false;
       this.parent = parent;
       this.children = children;
       this.exports = {};
       // ...
   }

   const module = new Module('uuid', null, []);
   ```

2. `Node`会为每一个模块提供一个 export 变量，指向 `module.exports`：

   ```javascript
   function Module (id, parent, children) {
       this.id = id;
       this.filename = 'filename.js';
       this.loaded = false;
       this.parent = parent;
       this.children = children;
       this.exports = {};
       // ...
   }

   const module = new Module('uuid', null, []);
   let exports = module.exports;
   ```

3. 模块开发者向外部导入数据：

   ```javascript
   function Module (id, parent, children) {
       this.id = id;
       this.filename = 'filename.js';
       this.loaded = false;
       this.parent = parent;
       this.children = children;
       this.exports = {};
       // ...
   }

   const module = new Module('uuid', null, []);
   let exports = module.exports;

   module.exports = function () {
       console.log('module message');
   }
   ```

{% hint style="warning" %}
 **注意：**虽然 Node 原生提供了 `exports` 作为 `module.exports` 的简化写法，但是不能手动改变 `exports` 的赋值，比如这样：`exports = {}`，这样写就代表将 `module.exports` 的引用从 `exports` 上切断了。这就意味着：如果一个模块的对外接口是一个单一的值（例如：数字、函数、字符串），就不能使用 `exports` 只能使用 `module.exports` 输出 。
{% endhint %}

#### 关于 require

{% hint style="info" %}
`CommonJS` 中 `require` 的基本功能，是读入并执行一个 JavaScript 文件，然后返回该模块的 `exports` 对象，如果没有发现指定模块则报错。
{% endhint %}

* require 加载文件时，默认后缀为 `.js` 后缀。
* 如果 `require` 中的路径字符串参数以 `'/'` 开头，则会按照这个绝对路径查找文件。
* 如果 `require` 中的路径字符串参数以 `'./'` 开头，则会以当前执行脚本位置为起点，寻找对应的相对路径下的文件。
* 如果参数字符串不以 `'/'` 或者 `'./'` 开头，则会去寻找一个默认提供的核心模块（位于 Node 系统安装目录中），或者一个位于各级 `node_modules` 目录中的已安装模块（全局安装或者局部安装），举例来说，如果脚本 `'/home/user/projects/foo.js'` 执行了 `require('bar.js')` 命令，Node 会依次搜索以下文件：
  * `/usr/local/lib/node/bar.js`（Node 的核心模块）
  * `/home/user/projects/node_modules/bar.js`（当前执行脚本所在目录下的 node\_modules 文件）
  * `/home/user/node_modules/bar.js`（执行脚本所在目录下没有 node\_modules ，则继续查找上层文件夹的 node\_modules）
  * `/home/node_modules/bar.js`（继续查找上层的 node\_modules）
  * `/node_modules/bar.js`（最后查找全局的 node\_modules）
*  如果参数字符串不以“./“或”/“开头，而且是一个路径，比如`require('example-module/path/to/file')`，则将先找到`example-module`的位置，然后再以它为参数，找到后续路径。
* 如果指定的文件没有找到，Node 会为文件名添加 `.js / .json / .node` 后缀再次尝试匹配，`.json` 文件会以 JSON 格式的文本文件解析，`.node` 文件会以编译后的二进制文件解析。

### ESModule 实现的模块化

{% hint style="info" %}
`ESModule` 是 ES6 提供的官方 js 模块化方案。目前浏览器还不能全面支持 `ESModule` 的语法，需要用 babel 进行解析。
{% endhint %}

#### ESModule  常用语法

输出变量和函数接口：

```javascript
// lib.js
export var a = 1;
// 或者 export 函数
export function func () {};


// main.js
import { a, func } from './lib.js';
```

将内部变量函数等封装为一个对象输出：

```javascript
// lib.js
var a = 1;
function func () {}
export {a, func};


// main.js
import { a, func } from './lib.js';
```

将内部变量函数等改名后封装为一个对象输出：

```javascript
// lib.js
var a = 1;
function func () {}
// 改一个名字然后暴露
export {a as aa, func as foo};


// main.js
// 注意，这边引入的时候就必须要用更改后的名字
import { aa, foo} from './lib.js';
```

`export` 输出变量和函数接口，然后在另一个文件中使用 `import *` 的方式接收：

```javascript
// lib.js
export var a = 1;
// 或者 export 函数
export function func () {};


// main.js
import * from './lib.js';
console.log(a);
func();

// 或者这么接
import * as lib from './lib.js';
console.log(lib.a);
lib.func();

```

怕麻烦的话，你也可以直接输出一个 `default` 默认。 `export default` 命令用于指定模块的默认输出，一个模块只能有一个默认输出，因此`export default` 命令只能使用一次。所以，`import` 命令后面不用加大括号，因为只可能唯一对应`export default` 命令：

```javascript
// lib.js
export default function () {
    console.log('Hello ESModule');
}


// main.js
// 如果引入 default 默认值，就没有固定的名称了，叫什么都可以
import foo from './lib.js';
```

 **需要注意！**由于 `export` 导出的必须是接口，下面的写法会报错：

```javascript
// export 错误写法，因为导出的不是接口而是值
var a = 1;
function func () {}
// 报错
export a;
// 报错
export 1;
// 报错
export func;
```

但是 `export default` 例外，因为本质上，`export default` 就是输出一个叫做`default`的变量或方法，然后系统允许你为它取任意名字。所以，上面那种会报错的写法改写成 `default` 是有效的，不过注意，一个文件只能暴露一个 `default` ：

```javascript
var a = 1;
export default a;
```

最后，还有一种 import 和 export 结合的高级写法，传说中的模块继承，这种写法我们会在某些源码的 `main.js` 中见到：

```javascript
// app.vue
function vueComponent () {}
export { vueComponent };


// main.js
export { vueComponent } from './app.vue';
// 或者
export { vueComponent as newVueComponent } from './app.vue';
// 或者
export * from './app.vue';
// 或者
export * as components from './app.vue'; 
```

`default` 版本的模块继承：

```javascript
// app.vue
function vueComponent () {}
export default vueComponent;


// main.js
export { default } from './app.vue';
```

#### ESModule VS CommonJS

`ESModule` 对比 `CommonJS` 主要有以下不同：

* `CommonJS` 模块输出的是一个**值的拷贝**，`ESModule` 输出的是**值的引用**。

  `CommonJS` 输出的是**值的拷贝**，也就是说一旦输出，模块内部的变化就影响不到这个值

  ```javascript
  // lib.js
  let counter = 3;
  function incCounter() {
    counter++;
  }
  module.exports = {
    counter,
    incCounter,
  };


  // main.js
  let mod = require('./lib');

  console.log(mod.counter);  // 3
  mod.incCounter();
  console.log(mod.counter); // 4
  ```

   `mod.counter`是一个原始类型的值，会被缓存。除非写成一个函数，才能得到内部变动后的值。

  ```javascript
  // lib.js
  let counter = 3;
  function incCounter() {
      counter++;
  }
  module.exports = {
      get counter: {
          return counter;
      },
      incCounter
  };


  // main.js
  let mod = require('./lib');

  console.log(mod.counter);  // 3
  mod.incCounter();
  console.log(mod.counter); // 3
  ```

  当然，你也可以对外暴露一个对象，`CommonJS` 导出的是对象引用的值的复制，那么这种情况 ，也是能够得到内部变动的值的。

  ```javascript
  // lib.js
  let obj = {a: 1};
  function changeA() {
      obj.a = 2;
  }
  module.exports = {
      obj,
      changeA
  };


  //  main.js
  const mod = require('./lib.js');
  console.log(JSON.stringify(mod.obj)); // {"a":1}
  target.changeA();
  console.log(JSON.stringify(mod.obj)); // {"a":2}
  ```

  `ESModule` 输出的是**值的引用**，它不会缓存运行结果，而是动态地去被加载的模块取值，并且变量总是绑定其所在的模块：

  ```javascript
  // lib.js
  export let counter = 3;
  export function incCounter() {
    counter++;
  }

  // main.js
  import { counter, incCounter } from './lib';
  console.log(counter); // 3
  incCounter();
  console.log(counter); // 4
  ```

* `ESModule` 的模块化是**静态的**，和 `CommonJS` 不同，`ESModule` 模块不是对象，而是通过 `export` 命令显示输出的指定代码的片段，再通过 `import` 命令将代码命令输入。也就是说在**编译阶段**就需要确定模块之间的依赖关系，这一点不同于 `AMD / CMD / CommonJS` ，这三者都是在**运行时**确定模块间的依赖关系的。

####  ESModule 的其他细节特点

* ES6 的模块自动采用严格模式，不管你有没有在模块头部加上`"use strict";`
* ESModule 导出的模块是只读的，不能变更，否则报错：

  ```javascript
  // lib.js
  export let obj = {};

  // main.js
  import { obj } from './lib';

  obj.prop = 123; // OK
  obj = {}; // TypeError
  ```



### 相关参考

* [JavaScript 标准参考教程](https://javascript.ruanyifeng.com/nodejs/module.html)
* [前端面试之道](https://juejin.im/book/5bdc715fe51d454e755f75ef/section/5bdd0d83f265da615f76ba57#heading-6)
* [ES6 Module 的语法](https://es6.ruanyifeng.com/#docs/module)

