---
description: >-
  俗话说：工欲善其事，必先利其器。学习 ES6 少不了的 步就是搭建 babel 的编译环境，毕竟现在主流浏览器对 ES6 的很多语法和 API
  支持不全面，还是需要 babel 转换成 ES5 的语法。
---

# 搭建 babel 编译环境

### 创建项目

* `mkdir ES6Learning && cd ES6Learning`

### 安装依赖

* `npm init` （初始化 `package.json` 文件）
* `npm install @babel/core @babel/cli @babel/preset-env` （安装 `babel` 编译所用到的相关模块）

### babel 配置文件

* `touch .babelrc` （生成 babel 的配置文件）
* 在 `.babelrc` 文件中写入如下信息：

  ```javascript
  {
      presets: [
          "@babel/preset-env"
      ]
  }
  ```

### 编写一个测试文件

`touch test.js` ，创建完成后用 `IDE` 工具打开，在里面写点 `ES6` 代码然后保存，比如：

```javascript
let str = 'Hello ES6!';
connsole.log(str);
```

### 运行 babel 编译

使用 `npx babel test.js –o demo.js` 命令来运行 babel 编译，将 `test.js` 文件内容编译到同目录下的 `demo.js` 文件中去，等待项目目录下生成 `demo.js` 文件后，打开 `demo.js` ，可以查看编译后的内容：

```javascript
"use strict";

var str = 'Hello ES6!';
console.log(str);
```

至此，babel 编译环境已经构建成功！

{% hint style="info" %}
`npx` 是 `npm 5.2` 版开始新增的一个命令，主要用途就是用项目内部安装的模块，比如说以 `mocha` 举例，之前是这样调用 `mocha` 的：

```text
# 项目的根目录下执行
$ node-modules/.bin/mocha --version
```

那么使用 npx 之后就可以这样调用：

```bash
$ npx mocha --version
```

更多使用细节可见：[npx 使用教程](http://www.ruanyifeng.com/blog/2019/02/npx.html)
{% endhint %}

### 实时编译

虽然上面的步骤已经将环境搭建完成，但我们在 test.js 中改一个字，就要重新编译一次，非常不痛快。我们可以开启 `babel` 的 `watch` 功能：

`npx babel test.js –o demo.js --watch`

 这样 `test.js` 文件发生了任何修改，`babel` 就能自动编译到 `demo.js` 中，是不是很方便？我们可以将这段代码写入 `package.json` 的 `script` 中，下次只需要运行 `npm start` 命令就能一键启动监听：

```javascript
{
    // ...
    "scripts": {
        "start": "npx babel test.js -o demo.js --watch"
    }
    // ...
}
```

好了，`babel` 编译环境已经搞定，下面就可以愉快的使用 `ES6` 了~



