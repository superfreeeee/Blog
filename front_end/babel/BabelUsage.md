# Babel 实战: Node 环境下使用 Babel 开发的 4 种运行配置方案

@[TOC](文章目录)

<!-- TOC -->

- [Babel 实战: Node 环境下使用 Babel 开发的 4 种运行配置方案](#babel-实战-node-环境下使用-babel-开发的-4-种运行配置方案)
- [系列文章](#系列文章)
- [前言](#前言)
- [正文](#正文)
  - [0. Node 环境](#0-node-环境)
    - [0.1 运行脚本](#01-运行脚本)
  - [1. 使用 @babel/cli + node 运行](#1-使用-babelcli--node-运行)
    - [1.1 安装核心依赖](#11-安装核心依赖)
    - [1.2 配置文件 & 运行命令](#12-配置文件--运行命令)
  - [2. 使用 @babel/register 动态编译脚本](#2-使用-babelregister-动态编译脚本)
  - [3. 使用 @babel/node 依赖引入 Babel](#3-使用-babelnode-依赖引入-babel)
  - [4. 使用 webpack + babel-loader 编译 web 项目](#4-使用-webpack--babel-loader-编译-web-项目)
    - [4.1 安装依赖](#41-安装依赖)
    - [4.2 webpack.config.js 配置文件](#42-webpackconfigjs-配置文件)
  - [5. Bonus: 使用 @babel/node 编译 typescript 项目](#5-bonus-使用-babelnode-编译-typescript-项目)
    - [5.1 修改源代码](#51-修改源代码)
    - [5.2 引入 typescript 依赖 & 修改 babel.config.json 配置](#52-引入-typescript-依赖--修改-babelconfigjson-配置)
    - [5.3 运行命令 & 结果](#53-运行命令--结果)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 系列文章

- [Babel 入門：JavaScript 的下一代編譯器](https://blog.csdn.net/weixin_44691608/article/details/106579653)
- [Babel 应用: 利用 @babel/register 实现即时编译（在 Node 环境下使用 import/export ES6 语法）](https://blog.csdn.net/weixin_44691608/article/details/111932754)

# 前言

还不了解 Babel 的小伙伴可以先去看看前面两篇，本篇将从假设读者们已经对于 Babel 有初步的了解，为大家整理了 4 种在 Node 环境下配合 Babel 编译来运行项目的方法。

# 正文

## 0. Node 环境

第一种我们先演示不使用 Babel 的情况下，在普通的 Node 环境下运行 JS，我们可以使用 `nodex xxx` 来运行脚本。

### 0.1 运行脚本

然而 node 环境下默认只支持 CommonJS 的模块化方案，所以我们看下面的脚本(下列四种运行方案都是根据以此脚本为基础)

- `/src/other.js`

```js
export const msg = 'msg from other.js'
```

- `/src/index.js`

```js
import { msg } from './other'

console.log(`other msg: ${msg}`)
```

在这个脚本中我们使用了 ES6 的模块化方案，也就是说直接在 Node 环境下是不能运行的

- 运行结果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/babel_usage_console_esm_fail.png)

## 1. 使用 @babel/cli + node 运行

那么接下来我们就要加入 Babel 对代码进行编译，来让上面的脚本在 Node 环境下也能运行

### 1.1 安装核心依赖

首先又是安装依赖

```bash
$ yarn add @babel/core @babel/cli @babel/preset-env -D
```

- `@babel/core` Babel 核心
- `@babel/cli` Babel 命令行工具
- `@babel/preset-env` 支持最新 ES 语法

### 1.2 配置文件 & 运行命令

接下来我们就可以使用 `babel` 指令编译脚本，然后再使用 `node` 命令来运行编译后的脚本

首先先来个 babel 配置文件，来启用 `preset-env`

- `babel.config.json`

```json
{
  "presets": ["@babel/preset-env"]
}
```

接下来在 `package.json` 中写好运行脚本

- `package.json`

```json
{
  "scripts": {
    "start-cli": "yarn clean && yarn build-cli && node lib/index",
    "build-cli": "babel src/* -d lib/ > lib/log.txt",
    "clean": "rm -rf lib && mkdir lib"
  }
}
```

详细的 babel 命令含义这里就不解释了，看运行结果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/babel_usage_console_start_cli.png)

## 2. 使用 @babel/register 动态编译脚本

前面实际上是透过使用 `@babel/cli` 提供的 `babel` 命令静态编译脚本之后，再使用 `node` 命令执行脚本，但是这种方式实在是过于丑陋。

第二种方法我们使用 `@babel/register` 依赖，来动态的改变运行时的 `require` 钩子实现动态编译

- 安装依赖

```bash
$ yarn add @babel/register -D
```

- `/src/register.js`

```js
require('@babel/register')({
  presets: ['@babel/preset-env'],
})

module.exports = require('./index')
```

我们将使用替代的入口文件 `register.js`，然后在最后将真正的入口文件(`index.js`)导出；运行命令如下

- `package.json`

```json
{
  "scripts": {
    "start-node-register": "node src/register",
  }
}
```

最后结果运行成功

![](https://picures.oss-cn-beijing.aliyuncs.com/img/babel_usage_console_start_node_register.png)

## 3. 使用 @babel/node 依赖引入 Babel

然而使用 `@babel/register` 动态引入 Babel 实际上会修改 `require` 函数的钩子；我们还可以使用另一个依赖 `@babel/node` 提供的 `babel-node` 指令，让我们使用 Babel 就好像直接运行 Node 一样

- 安装依赖

```bash
$ yarn add @babel/node -D
```

- `package.json`

```json
{
  "scripts": {
    "start-babel-node": "babel-node src/index",
  }
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/babel_usage_console_start_babel_node.png)

沿用刚才写的 `babel.config.json` 配置文件，我们可以看到如此一来就像是只是从 `node xxx` 指令换成 `babel-node xxx` 一样丝滑。

## 4. 使用 webpack + babel-loader 编译 web 项目

上面提供的例子属于在 Node 环境下直接运行的 JS 脚本，然而作为一个前端人我们常常也需要写运行在浏览器的 web 项目，这时候我们就可以用上 webpack 作为打包工具，然后透过 webpack 的 `babel-loader` 来引入 Babel 对项目进行编译

### 4.1 安装依赖

```bash
$ yarn add webpack webpack-cli webpack-dev-server -D  # webpack 核心依赖
$ yarn add babel-loader -D  # babel 用的 loader
$ yarn add html-webpack-plugin clean-webpack-plugin webpackbar -D  # 其他插件(HTML插件、清理插件、进度条插件)
```

### 4.2 webpack.config.js 配置文件

```js
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
const WebpackBar = require('webpackbar')

module.exports = {
  // mode: 'development',
  mode: 'production',
  entry: path.join(__dirname, './src/index.js'),
  output: {
    path: path.join(__dirname, './dist'),
    filename: '[name].[chunkhash].bundle.js',
    publicPath: '/',
  },
  // devtool: 'inline-source-map',
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        exclude: /node-modules/,
        use: 'babel-loader',
      },
    ],
  },
  plugins: [
    new WebpackBar(),
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: './public/index.html',
      title: 'webpack + babel',
      hash: true,
    }),
  ],
  devServer: {
    contentBase: 'dist',
  },
}
```

我们只需要在 `module` 的部分，配置将所有以 `.js` 为副档名的文件都套用 `babel-loader`，这样它就会自动根据 `babel.config.json` 的配置预先将项目代码编译过后再投入打包结果当中；最后配置一下运行命令

- `package.json`

```json
{
  "scripts": {
    "start-webpack": "webpack serve",
    "build": "webpack",
  }
}
```

实际运行成果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/babel_usage_console_start_webpack.png)

打开 [http://localhost:8080](http://localhost:8080) 查看网页

![](https://picures.oss-cn-beijing.aliyuncs.com/img/babel_usage_page_start_webpack.png)

## 5. Bonus: 使用 @babel/node 编译 typescript 项目

最后给出一个额外的项目示例。随着前端项目规模的增长，前端工程化和稳定性的需求也越来越高，TS 或是其他静态类型检查的第三方库(如 `@flow`)势在必行，下面我们给出在使用 `@babel/node` 依赖的场景下如何编译运行 ts 项目

### 5.1 修改源代码

首先我们现将源代码改成 `.ts` 扩展名

- `/src/other2.ts`

```ts
export const msg: string = 'msg from other.ts'
```

- `/src/index2.ts`

```ts
import { msg } from './other2'

console.log(`(ts) other msg: ${msg}`)
```

### 5.2 引入 typescript 依赖 & 修改 babel.config.json 配置

由于我们直接使用 Babel 反而不需要安装原本的 `typescript` 依赖，而是安装 Babel 插件 `@babel/preset-typescript`

```bash
$ yarn add @babel/preset-typescript -D
```

接下来修改配置

- `babel.config.json`

```json
{
  "presets": ["@babel/preset-env", "@babel/preset-typescript"]
}
```

### 5.3 运行命令 & 结果

最后在 `babel-node` 运行命令后面添加 `-x .ts` 来支持 `.ts` 扩展名

```json
{
  "scripts": {
    "start-babel-node-ts": "babel-node src/index2 -x .ts",
  }
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/babel_usage_console_start_babel_node_ts.png)

我们可以看到，这样就可以使用 `babel-node` 的情况下支持 typescirpt 的编译了

# 结语

本篇介绍了四种使用 Babel 的方式，附带一个使用 `@babel/node` 的基础下编译 typescript 项目的示例。

# 其他资源

## 参考连接

| Title                    | Link                                                                                                     |
| ------------------------ | -------------------------------------------------------------------------------------------------------- |
| @babel/register          | [https://babeljs.io/docs/en/babel-register](https://babeljs.io/docs/en/babel-register)                   |
| @babel/node              | [https://babeljs.io/docs/en/babel-node.html](https://babeljs.io/docs/en/babel-node.html)                 |
| @babel/preset-typescript | [https://babeljs.io/docs/en/babel-preset-typescript](https://babeljs.io/docs/en/babel-preset-typescript) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/babel/babel_usage](https://github.com/superfreeeee/Blog-code/tree/main/front_end/babel/babel_usage)
