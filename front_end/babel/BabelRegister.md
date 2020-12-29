# Babel 应用: 利用 @babel/register 实现即时编译（在 Node 环境下使用 import/export ES6 语法）

@[TOC](文章目录)

<!-- TOC -->

- [Babel 应用: 利用 @babel/register 实现即时编译（在 Node 环境下使用 import/export ES6 语法）](#babel-应用-利用-babelregister-实现即时编译在-node-环境下使用-importexport-es6-语法)
  - [简介](#简介)
  - [参考](#参考)
  - [完整示例代码](#完整示例代码)
- [正文](#正文)
  - [使用 babel](#使用-babel)
  - [babel 依赖](#babel-依赖)
  - [项目实践](#项目实践)
    - [项目构建 & 安装依赖](#项目构建--安装依赖)
    - [编写工具函数(utils.js)和主要脚本(index.js)](#编写工具函数utilsjs和主要脚本indexjs)
    - [使用 @babel/register 运行时编译（重点）](#使用-babelregister-运行时编译重点)
    - [运行结果](#运行结果)
- [结语](#结语)

<!-- /TOC -->

## 简介

Node.js 的出现，大多数应用开始抛弃默认的浏览器环境，改成在 node 环境下开发，并透过 babel、webpack 等其他手段在部署上线的时候才编译打包成浏览器能解析的语法。

然而 Node.js 环境尽管已经相当完善也广泛使用，但是模块化机制这块还是一直采用的是 CommonJS 标准(使用 `require` 方法)，然而 javascript 已经在 ES6 提出公认的标准语法 import/export，下面我们将介绍如何透过使用 `@babel/register` 运行时即时编译来支持 ES6 的模块化语法。

## 参考

<table>
  <tr>
    <td>@babel/register-官方doc</td>
    <td><a href="https://www.babeljs.cn/docs/babel-register">https://www.babeljs.cn/docs/babel-register</a></td>
  </tr>
  <tr>
    <td>如何在 Node.js 中使用 import / export 的三种方法</td>
    <td><a href="https://blog.csdn.net/zwkkkk1/article/details/81564971">https://blog.csdn.net/zwkkkk1/article/details/81564971</a></td>
  </tr>
</table>

## 完整示例代码

<a href="https://github.com/superfreeeee/Blog-code/tree/main/front_end/babel:webpack:tools/babel_register">https://github.com/superfreeeee/Blog-code/tree/main/front_end/babel:webpack:tools/babel_register</a>

# 正文

## 使用 babel

很久以前曾写过一篇<a href="https://blog.csdn.net/weixin_44691608/article/details/106579653">Babel入門：JavaScript 的下一代編譯器</a>介绍 Babel 的基本概念，说白了就是在运行 js 前先行将代码编译成目标版本的语法。实现一次编写，根据使用环境需要配置编译就能够多次使用的概念。

本篇将借助 babel 的`运行时即时编译(runtime-compiler)`的使用方式，支持我们在开发时在 node 环境下使用标准的 ES6 模块化语法

## babel 依赖

要使用运行时即时编译 babel 我们需要引入三个依赖：

- `@babel/core`：babel 核心依赖
- `@babel/register`：用于将 babel 在运行时注册到 node 的 require 模块下（详细原理可以查阅参考一的官方说明）
- `@babel/preset-env`：使用预设环境，以支持 ES6 语法（可以设置编译结果需要的版本，通常直接引入就完事了，根据项目需要再另行配置）

## 项目实践

接下来我们就做一个超小型的 demo 来演示如何使用 babel 运行时即时编译的功能

### 项目构建 & 安装依赖

- 项目构建

```bash
$ mkdir babel_register
$ cd babel_register
$ yarn init -y
$ yarn add @babel/core @babel/register @babel/preset-env -D
$ touch start.js
$ touch src/index.js
$ touch src/utils.js
```

并到 `package.json` 中添加启动脚本 `"scripts: { "start": "node start" }"`，构建结果如下

- 项目结构

```
/babel_register
├── package.json
├── src
│   ├── index.js
│   └── utils.js
├── start.js
└── yarn.lock
```

- `package.json`

```json
{
  "name": "babel_register",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "scripts": {
    "start": "node start"
  },
  "devDependencies": {
    "@babel/core": "^7.12.10",
    "@babel/preset-env": "^7.12.11",
    "@babel/register": "^7.12.10"
  }
}
```

### 编写工具函数(utils.js)和主要脚本(index.js)

首先我们现用 ES6 的语法填一些内容，主要就是使用 import/export 的语法

- `utils.js`

```js
export function greeting () {
  console.log('hello world')
}
```

- `index.js`

```js
import { greeting } from './utils'

greeting()
```

### 使用 @babel/register 运行时编译（重点）

接下来就是本篇的重点核心，透过 `require('@babel/register')` 来将 babel 挂载到 require 模块上并实现运行时编译

- `start.js`

```js
require('@babel/register')({
  presets: ['@babel/preset-env']
})

module.exports = require('./src/index')
```

对就这么简单，引入 `@babel/register` 并调用，参数设置 `presets: ['@babel/preset-env']` 指定编译版本(支持 ES6 语法)，最后导出主要脚本(index.js)

### 运行结果

```bash
$ yarn start            
yarn run v1.22.10
$ node start
hello world
✨  Done in 1.14s.
```

很简单吧^^

# 结语

Babel 是一个非常强大的编译工具，透过自定义的插件和编译方式几乎能够支持任意语法，只要你能找到或自行开发插件。

本篇使用到的运行时编译配置在自己利用 Node 环境测试一些小项目的时候非常实用，可以补足 node 对于一些语法支持的不足(其实很少，`模块化机制`是一种)，同时使用 babel 也让你能够更早开始使用最新的如 ES9、ES10 的语法糖，写出最潮的 js 代码hh。
