# JS 模块化: AMD 模块化方案的理解与应用

@[TOC](文章目录)

<!-- TOC -->

- [JS 模块化: AMD 模块化方案的理解与应用](#js-模块化-amd-模块化方案的理解与应用)
- [前言](#前言)
  - [JS 模块化标准：CommonJS、AMD、ES6 Module(ESM = ECMAScript Module)](#js-模块化标准commonjsamdes6-moduleesm--ecmascript-module)
  - [动机](#动机)
- [正文](#正文)
  - [1. 模块化之前](#1-模块化之前)
    - [1.1 同步脚本](#11-同步脚本)
    - [1.2 在 \<head\> 中引入 脚本](#12-在-head-中引入-脚本)
    - [1.3 使用 window.onload](#13-使用-windowonload)
  - [2. 使用 RequireJS 实现 AMD 规范](#2-使用-requirejs-实现-amd-规范)
    - [2.1 引入 RequireJS](#21-引入-requirejs)
    - [2.2 AMD 模块规范](#22-amd-模块规范)
    - [2.3 Sample 1: 基本模块定义](#23-sample-1-基本模块定义)
    - [2.4 Sample 2:](#24-sample-2)
    - [2.5 Sample 3: require 内置模块](#25-sample-3-require-内置模块)
    - [2.6 Sample 4: 默认参数 & CommonJS 风格模块](#26-sample-4-默认参数--commonjs-风格模块)
  - [3. 使用 Webpack 打包 AMD 模块](#3-使用-webpack-打包-amd-模块)
    - [3.1 r.js 打包(弃用)](#31-rjs-打包弃用)
    - [3.2 gulp 踩坑](#32-gulp-踩坑)
    - [3.3 webpack.config.js 配置文件](#33-webpackconfigjs-配置文件)
    - [3.4 html 模版 & 模块定义](#34-html-模版--模块定义)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

今天要给大家带来的是 JS 模块化规范中的 **AMD 模块化方案**。

## JS 模块化标准：CommonJS、AMD、ES6 Module(ESM = ECMAScript Module)

在 JavaScript 演化的过程中，一开始作为一个脚本语言是没有"模块(module)"的概念的，一切都在全局作用域之下运行。

然而随着代码规模的增长，对 JS 脚本的模块化是必然的选择。问题在于由于一开始并没有共同的标准(直到后来 ES6 推出的 ESM = ECMAScrip Module 解决方案)，在此之前存在 **CommonJS、AMD** 两种模块化方案。

由于 CommonJS 是 Node 默认采用的方案，还算是比较常见；而本篇要介绍的则是另一种模块化方案：**AMD(Asynchronous Module Definition) 规范**

## 动机

会想写这篇主要是想要探索 JQuery 的源码的时候发现使用的语法比较特别，经过调查发现 JQuery 在 1.7.0 版本之后使用的是 AMD 的模块化方案，因此决定先来了解一下 AMD 模块化方案的使用形式。

# 正文

## 1. 模块化之前

在进入具体的模块化规范之前，我们先来看看几个网页引用 JS 脚本的问题

### 1.1 同步脚本

第一种是同步引入的脚本，稍微写过页面的人应该都知道引用外部的 JS 脚本必须放在 body 的最后面，因为默认的脚本引入是同步的，必须等待 dom 解析完毕之后引入脚本才能看到正确的 dom 树，也才能正确的访问节点

- `sample/sample1/index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>JS 模块化: AMD 模块化方案的理解和应用(使用 Webpack 打包)</title>
  </head>
  <body>
    <h1>引用同步脚本</h1>

    <script src="index.js"></script>
  </body>
</html>
```
- `sample/sample1/index.js`

```js
console.log('hello world sample1')
```

- 运行截图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_module_amd_sample1.png)

### 1.2 在 \<head\> 中引入 脚本

然而很多时候我们并不希望脚本的引入顺序干扰到原本 html 页面的标签，所以我们可能想要将脚本放在 \<head\> 标签当中

- `sample/sample2/index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>JS 模块化: AMD 模块化方案的理解和应用(使用 Webpack 打包)</title>
    <script src="before.js"></script>
  </head>
  <body>
    <h1>在 head 中引用同步脚本</h1>

    <script src="after.js"></script>
  </body>
</html>
```

- `sample/sample2/before.js`

```js
console.log('sample2: import in <head>')

console.log(document.querySelector('h1'))
```

- `sample/sample2/after.js`

```js
console.log('sample2: import at the end of <body>')

console.log(document.querySelector('h1'))
```

这里我们分别引入了两个脚本，我们发现对于 before.js 来说，dom 还没被解析完毕所以找到的元素为 `null` 

- 运行截图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_module_amd_sample2.png)

### 1.3 使用 window.onload

为了确保脚本在 dom 解析完毕后才执行，我们可以利用 `window.onload` 事件来执行我们的脚本

- `sample/sample3/index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>JS 模块化: AMD 模块化方案的理解和应用(使用 Webpack 打包)</title>
    <script src="before.js"></script>
  </head>
  <body>
    <h1>使用 onload 确保脚本加载后执行</h1>

    <script src="after.js"></script>
  </body>
</html>
```

- `sample/sample3/before.js`

```js
window.onload = function () {
  console.log('sample3: import in <head>')

  console.log(document.querySelector('h1'))
}
```

- `sample/sample3/after.js`

```js
var cb = window.onload || function() {}
window.onload = function () {
  cb()
  console.log('sample3: import at the end of <body>')

  console.log(document.querySelector('h1'))
}
```

然而这样又会因为 `window.onload` 只能赋值一次而产生回调方法覆盖的问题，所以才需要向上面一样使用 `cb` 来保留原有的回调

- 运行截图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_module_amd_sample3.png)

## 2. 使用 RequireJS 实现 AMD 规范

然而上面提到的最后一种情况使用 `window.onload` 实际上是对代码有破坏性的，同时也还是与脚本的调用顺序有关(保留前一次挂载的 `window.onload` 回调)，这时候就可以用上 AMD 的模块化方案了。

使用 AMD 的概念便是：

> 每一个 js 文件属于一个独立的模块，每个模块异步加载(在 dom 完全解析之后才执行脚本内容)

### 2.1 引入 RequireJS

而我们实现 AMD 模块化的第一个方式就是引入 RequireJS 库

- `sample/sample4/index.html`

```js
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>JS 模块化: AMD 模块化方案的理解和应用(使用 Webpack 打包)</title>
    <script src="js/lib/require.js" data-main="js/index.js"></script>
    <!-- <script src="js/lib/require.js" data-main="js/index2.js"></script> -->
    <!-- <script src="js/lib/require.js" data-main="js/index3.js"></script> -->
    <!-- <script src="js/lib/require.js" data-main="js/index4.js"></script> -->
  </head>
  <body>
    <h1>RequireJS 初探</h1>
  </body>
</html>
```

首先我们先到官网[下载脚本](https://requirejs.org/docs/download.html)，接着就可以在 \<head\> 标签中引入

```html
<script src="js/lib/require.js" data-main="js/index.js"></script>
```

`src` 为 RequireJS 脚本的路径，而 `data-main` 则是 AMD 模块的入口脚本

### 2.2 AMD 模块规范

而下面我们就要在 `index.js` 中定义 AMD 模块。AMD 模块的定义方式就是使用 `define` 方法，函数标签如下

```js
define(module)
define(dependencies, module)
define(moduleName, dependencies, module)
```

一个 AMD 模块的定义可以有三种实参调用形式：

- 一个参数的时候：参数(module)表示的就是要定义的模块，可以是一个对象；也可以写成一个函数，函数返回模块对象
- 两个参数的时候：前面的参数表示该模块依赖的其他模块列表
- 三个参数的时候：在前面再加上一个模块名称，使用少于三个参数的调用形式的时候，默认以文件名作为模块名称

### 2.3 Sample 1: 基本模块定义

首先我们先看第一种，最基本的模块使用方式

- `sample/sample4/js/index.js`

```js
/**
 * * simple define
 */
define(function () {
  'use strict'
  console.log('sample 4: using RequireJS')
  console.log(document.querySelector('h1'))
})
```

模块也可以不返回任何东西，这时候引用模块仅仅作为一次性的脚本，而不导出任何对象

- 运行截图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_module_amd_sample4.png)

### 2.4 Sample 2: 

第二个示例我们演示当一个模块依赖于其他模块的用法(也就是前面提过的两个参数的用法)

- `sample/sample4/js/index2.js`

```js
/**
 * * dependency
 */
define(['js/other.js'], function (other) {
  'use strict'
  console.log('other', other)
})
```

- `sample/sample4/js/other.js`

```js
define(function () {
  const data = {
    name: 'data.name in other.js',
  }
  return data
})
```

我们看到 `other.js` 模块导出了一个 `data`，而这个 data 就作为 `index.js` 模块定义时的参数，这个参数的顺序是根据依赖列表的顺序决定

- 运行截图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_module_amd_sample4_2.png)

### 2.5 Sample 3: require 内置模块

除了自定义模块之外，RequireJS 还提供了内置的模块，这边演示一个名为 `require` 的内置模块，使用方式如下

- `sample/sample4/js/index3.js`

```js
/**
 * * require module
 */
define(['require', 'js/other.js'], function (require, other) {
  'use strict'
  console.log('other', other)

  const other2 = require('js/other.js')
  console.log('other2', other2)
  console.log('other === other2:', other === other2)
})
```

而这个 `require` 方法就跟我们熟悉的 CommonJS 规范的 `require` 函数是类似的，用于异步加载其他模块

- 运行截图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_module_amd_sample4_3.png)

同时我们还能够看到，不管是透过依赖列表引入的模块，或是使用 `require` 引入的，引用同一个模块则会返回一样的指针，不会产生值拷贝或其他问题

### 2.6 Sample 4: 默认参数 & CommonJS 风格模块

在不使用前两个参数的情况下，其实模块定义的函数默认接受了三个参数

- `sample/sample4/js/index4.js`

```js
/**
 * * default modules
 */
define(function (require, exports, module) {
  console.group('index.js')
  // console.log('require', require)
  // console.log('exports', exports)
  // console.log('module', module)
  // console.log('module.exports === exports', module.exports === exports)

  const other2 = require('js/other2.js')
  console.log('other2', other2)
  console.groupEnd()
})
```

- require 为异步加载函数
- exports 为当前模块的导出值(这个就跟 CommonJS 的 exports 变量相似)
- module 则是指向当前模块的指针

我们看到上面的 `index` 模块引入了一个 `other2` 模块，而这个 `other2` 则是在 AMD 模块的规范之下采用 CommonJS 风格的模块定义形式(使用 `require` 导入、使用 `exports` 导出变量)

- `sample/sample4/js/other2.js`

```js
define(function (_, exports, module) {
  exports.a = 123
  exports.b = 456

  console.group('other2.js')
  console.log('module', module)
  console.log('module.exports', module.exports)
  console.groupEnd()
})
```

- 运行截图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_module_amd_sample4_4.png)

## 3. 使用 Webpack 打包 AMD 模块

然而在真正的前端工程当中，其实我们可能会使用到好多 JS 模块，也就是一个个 JS 文件，每次打开网页都要发出好多请求效率很差，我们就需要进行所谓的"优化(optimize)"，也就是吧静态关联的文件打包在一起。

### 3.1 r.js 打包(弃用)

RequireJS 提供了一个官方的库 r.js 来做这件事，然而打包的功能还是比较初级，所以这边就不使用

### 3.2 gulp 踩坑

一开始查到一些使用 gulp 进行 AMD 模块打包的方法，但是坑还是比较多的，gulp 的不同版本还会产生与插件不兼容的情况，还是比较烦的，所以最终选择使用 webpack 来打包我们的 JS 工程

备注：使用 gulp 打包的重点在于 `amd-optimize` 插件，而且只能与 gulp3 兼容，不能使用 gulp4

### 3.3 webpack.config.js 配置文件

webpack 的基础配置就不再重新提了，不知道 webpack 的可以先去看[前一篇：Webpack 入門](https://blog.csdn.net/weixin_44691608/article/details/106675550)

webpack 官方其实就说了，原生支持 AMD 模块方式！(害我前面搞 gulp 搞了半天啧啧)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_module_amd_webpack_module.png)

所以我们其实就是配置一个 webpack 项目就行了！只是文件使用 AMD 的规范来写

第一步：初始化项目 & 安装依赖

```bash
$ yarn init -y
$ yarn add webpack webpack-cli webpack-dev-server -D
$ yarn add html-webpack-plugin clean-webpack-plugin -D
```

第二步给出配置文件

- `webpack.config.js`

```js
const path = require('path')

const HtmlWebpackPlugin = require('html-webpack-plugin')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

module.exports = {
  mode: 'production',
  entry: path.join(__dirname, 'src/index'),
  output: {
    path: path.join(__dirname, 'dist'),
    filename: '[name]-[chunkhash].js',
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: './public/index.html',
      title: 'JS 模块化: AMD 模块化方案的理解和应用(使用 Webpack 打包)',
    }),
  ],
  devServer: {
    contentBase: 'dist',
  },
}
```

### 3.4 html 模版 & 模块定义

下面给出项目的模版和具体的脚本内容

- `public/index.html`

```js
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title><%= htmlWebpackPlugin.options.title %></title>
  </head>
  <body>
    <div id="app">
      <h1>使用 Webpack 解析并打包 AMD 模块</h1>
    </div>
  </body>
</html>
```

- `src/index.js`

```js
define(['./utils/amdFn.js', './utils/cjsFn.js'], function (fn1, fn2) {
  console.group('index.js')
  console.log('fn1', fn1)
  console.log('fn2', fn2)
  fn1.f()
  fn1.g()
  fn2.f()
  fn2.g()
  console.groupEnd()
})
```

- `src/utils/amdFn.js`

```js
define(function () {
  function f() {
    console.log('invoke function f from amdFn.js')
  }

  function g() {
    console.log('invoke function g from amdFn.js')
  }

  return {
    f,
    g,
  }
})
```

- `src/utils/cjsFn.js`

```js
define(function (require, exports) {
  exports.f = function f() {
    console.log('invoke function f from cjs.js')
  }

  exports.g = function g() {
    console.log('invoke function g from cjs.js')
  }
})
```

我们使用 AMD 规范定义两个风格的 JS 模块，最后在 `index` 模块中调用

最后 `yarn build` 打包后，打开打包后的页面

- 运行截图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_module_amd_sample4_4.png)

# 结语

# 其他资源

## 参考连接

| Title                     | Link                                                                                                                                                                                               |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| js-模块化(三大模块化规范) | [https://www.cnblogs.com/john-hwd/p/10580620.html](https://www.cnblogs.com/john-hwd/p/10580620.html)                                                                                               |
| webpack - module          | [https://webpack.docschina.org/concepts/modules/](https://webpack.docschina.org/concepts/modules/)                                                                                                 |
| require.js                | [https://requirejs.org/](https://requirejs.org/)                                                                                                                                                   |
| gulp打包amd工程           | [https://blog.csdn.net/weixin_33916256/article/details/89149723?utm_source=app&app_version=4.8.1](https://blog.csdn.net/weixin_33916256/article/details/89149723?utm_source=app&app_version=4.8.1) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_module_amd](https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_module_amd)
