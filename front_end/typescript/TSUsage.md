# TS 基础: 运用 TypeScript 进行开发的 5 种方式

@[TOC](文章目录)

<!-- TOC -->

- [TS 基础: 运用 TypeScript 进行开发的 5 种方式](#ts-基础-运用-typescript-进行开发的-5-种方式)
- [前言](#前言)
- [正文](#正文)
  - [1. 使用官方 tsc 命令编译](#1-使用官方-tsc-命令编译)
    - [1.1 安装编译器依赖 typescript](#11-安装编译器依赖-typescript)
    - [1.2 指定编译文件](#12-指定编译文件)
    - [1.3 使用 tsconfig.json 配置文件](#13-使用-tsconfigjson-配置文件)
    - [1.4 监听变化实时编译](#14-监听变化实时编译)
    - [1.5 编译后执行代码](#15-编译后执行代码)
  - [2. 使用 ts-node 编译后立即执行](#2-使用-ts-node-编译后立即执行)
  - [3. 使用 nodemon 监听变化自动运行](#3-使用-nodemon-监听变化自动运行)
  - [4. 使用 tsc 编译 + html 直接引入](#4-使用-tsc-编译--html-直接引入)
  - [5. 使用 webpack 集成 typescript](#5-使用-webpack-集成-typescript)
    - [5.1 生产环境打包](#51-生产环境打包)
    - [5.2 开发环境](#52-开发环境)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 前言

JavaScript 以其动态类型和灵活的变量使用著称，然而在大型工程项目，或是严谨的代码规范之下，却造成我们在工程项目中为了确保兼容性和代码可用性不得不进行额外的类型检查，多余的测试等，不仅影响开发/测试效率，同时也会一定程度的增加运行时的开销。

在这样的环境之下，**TypeScript** 应运而生，与 Flow 相似，都是作为 **静态类型检查工具** 而存在。所谓 **静态类型检查工具** 的意思是在 TS 被编译过后会还原成最基本不带任何类型的原生 JS 代码。而且比 Flow 更强大的是，TypeScript 除了添加了静态类型之外，同时还提供了更进阶/实验中的 JS 新特性，也就是因此 TS 同时被称为 JS 的超集。

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_usage_superset_abstract.png)

本篇将来带大家熟悉到底如何来运用 TypeScript 来进行开发

# 正文

## 1. 使用官方 tsc 命令编译

第一种方法我们需要用到官方的编译工具，先安装依赖

### 1.1 安装编译器依赖 typescript

我们可以选择全局或是本地(项目内)安装编译用的依赖

```bash
$ yarn add -g typescript # 全局安装
$ yarn add typescript -D # 个别项目
```

### 1.2 指定编译文件

接下来可以直接写一个 `.ts` 的文件

- `src/index.ts`

```ts
const msg1: string = 'Hello'
const msg2: string = 'World'
const msg = `${msg1} ${msg2}!`
console.log(msg)
```

并直接使用 `tsc` 命令进行编译

- `package.json`

```json
{
    "script": {
        "basic": "tsc src/index.ts"
    }
}
```

```bash
$ yarn basic
```

然后我们就可以看到编译后的结果

- `src/index.js`

```js
var msg1 = 'Hello';
var msg2 = 'World';
var msg = msg1 + " " + msg2 + "!";
console.log(msg);
```

### 1.3 使用 tsconfig.json 配置文件

然而每次都要在编译命令直接指定文件非常麻烦，使用编译参数也显得非常复杂，不如写成一个配置文件：

- `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES6",
    "outDir": "lib"
  },
  "include": ["src/*"]
}
```

我们可以透过 `include` 指定编译文件，`outDir` 指定输出目录，`target` 指定输出代码版本。更多配置可以参考 ts 文档

接下来我们就能够不指定文件直接使用配置文件进行编译(未指定文件的情况下会查找当前目录下的 `tsconfig.json` 作为默认配置文件)

- `package.json`

```json
{
    "script": {
        "use-config": "tsc"
    }
}
```

```bash
$ yarn use-config
```

看看输出

- `lib/index.js`

```js
const msg1 = 'Hello';
const msg2 = 'World';
const msg = `${msg1} ${msg2}!`;
console.log(msg);
```

我们可以看到输出的代码根据配置文件被编译到 `lib` 目录下了，同时编译的结果也转为使用 ES6 版本的语法

### 1.4 监听变化实时编译

有时候我们在开发的过程或是尝试一些简单的特性的时候，每次修完都要输入 `tsc` 编译一下太麻烦，所以我们可以使用 `--watch`(`-w`) 参数来开启观察模式，他会持续观察文件的变化并在每次保存的时候自动重新编译。

- `package.json`

```json
{
    "script": {
        "basic-watch": "tsc src/index.ts -w"
    }
}
```

### 1.5 编译后执行代码

最后一件事情就是执行代码。我们不管是用 ts 写还是用 js 写就是要拿来跑的。所以第一种选择我们可以直接使用命令运行编译后的 js 文件

```bash
$ node lib/index.js
Hello World!
```

但是手动输入也是挺麻烦的，我们可以写到 `package.json` 里面合并为一个命令

- `package.json`

```json
{
    "script": {
        "use-config-run": "tsc && node lib",
    }
}
```

```bash
$ yarn use-config-run
yarn run v1.22.10
$ tsc && node lib
Hello World!
✨  Done in 3.80s.
```

## 2. 使用 ts-node 编译后立即执行

自己写成一个 yarn 命令其实也不是不行，但是已经有一个包想到了我们总是要在 node 环境下执行 ts 文件。既然我们都会经过 `tsc` 编译、`node` 运行命令，不如就使用一个 `ts-node` 来运行 ts 文件

首先安装依赖

```bash
$ yarn add ts-node -D
```

接下来我们可以直接复用之前的 `src/index.ts` 文件，配上运行命令

- `package.json`

```json
{
    "script": {
        "use-ts-node": "ts-node src/index.ts",
    }
}
```

```bash
$ yarn use-ts-node
yarn run v1.22.10
$ ts-node src/index.ts
Hello World!
✨  Done in 2.16s.
```

这里实际上 `ts-node` 就是 `tsc + node` 的组合效果，不同的是这里不需要指定输出目录也不会产生实际的输出文件，而是直接在内存中编译并执行

## 3. 使用 nodemon 监听变化自动运行

有了 `ts-node` 帮我们直接在 node 环境中运行 ts 文件了，接下来再搭上 `nodemon` 来实时监听文件变化并执行

一样先来安装依赖

```bash
$ yarn add nodemon -D
```

再来是运行命令

- `package.json`

```json
{
    "script": {
        "use-nodemon": "nodemon src/index.ts",
    }
}
```

```bash
$ yarn use-nodemon
yarn run v1.22.10
$ nodemon src/index.ts
[nodemon] 2.0.7
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: ts,json
[nodemon] starting `ts-node src/index.ts`
Hello World!
[nodemon] clean exit - waiting for changes before restart
[nodemon] restarting due to changes...
[nodemon] starting `ts-node src/index.ts`
Hello World!
[nodemon] clean exit - waiting for changes before restart
```

下面我们可以看到运行后会执行第一次的编译并执行(其内部用的也是 `ts-node` 命令)，而后只要文件改动就会立马重新编译并运行(`restarting due to changes...` 那一句)

## 4. 使用 tsc 编译 + html 直接引入

前面我们已经可以在 node 环境中运行 ts 文件了，但是在前端开发中我们更常遇到的是写到网页中在浏览器运行 js。这时候我们就必须要将代码(js 或是 ts 编译后的结果)引入到 html 再打开网页由浏览器来执行

第一种我们使用一个比较智障的方式：使用 `tsc` 编译之后，直接在 html 文件中引入编译后的 js 文件如下(这边一样复用前面的 `index.ts`)

- `public/ref.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>TS Usage</title>
  </head>
  <body>
    <h1>运用 TS 的五种方式 - html 引用</h1>

    <script src="../src/index.js"></script>
  </body>
</html>
```

同时我们再配合使用 `-w` 进行实时编译的方式，其实也不是不能用hh

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_usage_4_html_script.png)

## 5. 使用 webpack 集成 typescript

最后一种就比较接近我们实际开发时候的场景，下面我们将 typescript 集成到 webpack 打包工具内，就可以在项目中使用 ts 来进行开发，打包工具(babel 或 tsc 会帮我们编译)会自动的编译后合并到原来的代码当中

一样第一步先引入依赖

```bash
$ yarn add webpack webpack-cli@3.3.12 webpack-dev-server -D  # webapck 核心
$ yarn add ts-loader typescript -D  # TypeScript 依赖
```

然后编写配置文件

- `webpack.config.js`

```js
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

module.exports = {
  mode: 'production',
  entry: path.join(__dirname, 'src/index.ts'),
  output: {
    path: path.join(__dirname, 'dist'),
    filename: '[name]-[chunkhash].js',
  },
  module: {
    rules: [
      {
        test: /\.ts/,
        use: 'ts-loader',
      },
    ],
  },
  plugins: [
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'public/index.html',
      title: 'TS Usage'
    }),
    new CleanWebpackPlugin(),
  ],
  devServer: {
    contentBase: 'dist',
    port: 8080,
  },
}
```

这边我们区分成生产环境的打包和开发时的服务器(webpack-dev-server)两种

### 5.1 生产环境打包

首先配置打包入口和出口

```js
module.exports = {
  // ...
  entry: path.join(__dirname, 'src/index.ts'),
    output: {
    path: path.join(__dirname, 'dist'),
    filename: '[name]-[chunkhash].js',
  },
  // ....
}
```

再来是配置 `.ts` 文件的编译 loader

```js
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.ts/,
        use: 'ts-loader',
      },
    ],
  },
  // ....
}
```

最后我们再加上 clean-webpack-plugin 来清理编译输出、htmlWebpackPlugin 来生成 html 页面

```js
module.exports = {
  // ...
  plugins: [
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'public/index.html',
      title: 'TS Usage'
    }),
    new CleanWebpackPlugin(),
  ],
  // ....
}
```

最后附上 html 模版

- `public/index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title><%= htmlWebpackPlugin.options.title %></title>
  </head>
  <body>
    <h1>运用 TS 的五种方式 - HtmlWebpackPlugin</h1>
  </body>
</html>
```

运行命令打包

- `package.json`

```json
{
    "scripts": {
        "build": "webpack",
    }
}
```

```bash
$ yarn build
yarn run v1.22.10
$ webpack
asset index.html 352 bytes [emitted]
asset main-c8965112341f1c28f2bf.js 28 bytes [emitted] [immutable] [minimized] (name: main)
./src/index.ts 94 bytes [built] [code generated]
webpack 5.37.1 compiled successfully in 2254 ms
✨  Done in 3.62s.
```

### 5.2 开发环境

另外我们再使用 webpack-dev-server 配置一个开发环境，首先在配置文件加上

- `webpack.config.js`

```js
module.exports = {
  // ...
  devServer: {
    contentBase: 'dist',
    port: 8080,
  },
  // ....
}
```

然后是运行命令

- `package.json`

```json
{
    "scripts": {
        "serve": "webpack-dev-server"
    }
}
```

```bash
$ yarn serve
yarn run v1.22.10
$ webpack-dev-server
ℹ ｢wds｣: Project is running at http://localhost:8080/
ℹ ｢wds｣: webpack output is served from /
ℹ ｢wds｣: Content not from webpack is served from dist
```

打开页面就能看到了

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_usage_5_webpack_dev_server.png)

# 结语

TypeScript 使得自由却混乱的 JS 得到类型的约束，同时也在一定程度上规范程序员的表达，使得代码的可读性、可维护性、可扩展性慢慢地提高，对于项目的稳定性和可用性也有极大的提升，同时透过纯粹的编译在运行时仅保留最原始的 JS 内容，可以说是未来前端开发的必备技能没有之一。

# 其他资源

## 参考连接

| Title                           | Link                                                                                                                               |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| Typescript 中文网-tsconfig.json | [https://www.tslang.cn/docs/handbook/tsconfig-json.html](https://www.tslang.cn/docs/handbook/tsconfig-json.html)                   |
| 让你的项目使用Ts吧              | [https://blog.csdn.net/weixin_30897079/article/details/101335674](https://blog.csdn.net/weixin_30897079/article/details/101335674) |
| node工具之nodemon               | [https://www.jianshu.com/p/f60e14db0b4e](https://www.jianshu.com/p/f60e14db0b4e)                                                   |
| ts-node - github                | [https://github.com/TypeStrong/ts-node](https://github.com/TypeStrong/ts-node)                                                     |
| nodemon 首页                    | [https://nodemon.io/](https://nodemon.io/)                                                                                         |
| nodemon - github                | [https://github.com/remy/nodemon](https://github.com/remy/nodemon)                                                                 |

## 阅读笔记参考

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/typescript/ts_usage](https://github.com/superfreeeee/Blog-code/tree/main/front_end/typescript/ts_usage)
