---
description: >-
  近年来 ECMAScript
  发展十分迅速，越来越多的新的语法及提案出台，但是浏览器的更新速度却远远跟不上语言的迭代速度，这种情况下，就需要有一个工具能将新的语法、提案转换为浏览器能够识别的脚本语言，于是
  babel 应运而生。
---

# babel

### 什么是 babel

{% hint style="info" %}
`babel` 负责解析新的 `ESMAScript` 的标准，`babel` 运行编译时，会从项目根目录下读取 `.babelrc` 的配置文件，通过分析预设（presets）和插件（plugins）等配置项目，来对最新语法进行编译。
{% endhint %}

### babel、babel-polyfill 和 babel-runtime

* `babel` 将 `Javascript` 分为 `synax`（语法）和 `api` 两部分，`babel` 只负责编译 `ECMAScript` 的最新语法，比如 `let`、 `const`、 箭头函数等，而对于新的 `api` 比如 `Promise`、`include`、`Object.assign` 这种则不进行编译，而是交给 `babel-polyfill` 进行编译。
* `babel-polyfill` 负责将新的 API 封装成浏览器可识别的 API 来覆盖原生，并将所有生成的覆盖代码插入项目源码。
* `babel-polyfill` 虽然提供了新的 API 支持，但是缺陷也比较明显：
  * 全局注册覆盖的 API，污染全局变量
  * 会将新的 API 全部封装导出到源码，不管项目中有没有使用过相关内容，导致导出项目包增大。
* `babel-runtime` 针对 `babel-polyfill` 的问题做了优化：
  * 通过导出包 / 引入包的方式提供新封装的 API ，解决了污染全局变量的问题
  * 结合 `babel-transform-runtime-plugin` 实现自动检测项目所用到的 API ，有选择的编译和引入，解决了导出包增大的问题。
  * 但是 `babel-runtime` 有个问题，就是不能支持实例上的新增 API 方法，例如`[1,2,3].includes(1)` 这种，具体见 [stackOverFlow](https://stackoverflow.com/questions/31781756/is-there-any-practical-difference-between-using-babel-runtime-and-the-babel-poly)。
* `babel-polyfill` 和 `babel-runtime` 对比：
  * `babel-polyfill` 适合在比较大的项目中引入。
  * `babel-runtime` 适合在组件、类库中引用。



