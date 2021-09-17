# Webpack 实践: 前端 SDK 开发（使用 webpack 打包 library）

@[TOC](文章目录)

<!-- TOC -->

- [Webpack 实践: 前端 SDK 开发（使用 webpack 打包 library）](#webpack-实践-前端-sdk-开发使用-webpack-打包-library)
- [前言](#前言)
- [正文](#正文)
  - [1. 第一版：源代码直接导出](#1-第一版源代码直接导出)
    - [1.1 目录结构 & 源代码](#11-目录结构--源代码)
    - [1.2 package.json 配置](#12-packagejson-配置)
    - [1.3 创建本地软连接](#13-创建本地软连接)
    - [1.4 引入结果测试](#14-引入结果测试)
  - [2. 第二版：使用 webpack 打包](#2-第二版使用-webpack-打包)
    - [2.1 安装依赖 & 目录结构](#21-安装依赖--目录结构)
    - [2.2 源代码扩展](#22-源代码扩展)
    - [2.3 webpack.config.js 配置](#23-webpackconfigjs-配置)
    - [2.4 打包成果](#24-打包成果)
    - [2.5 usage 项目中引入](#25-usage-项目中引入)
  - [3. 第三版：加上 babel 编译 & 导出 ts 声明文件](#3-第三版加上-babel-编译--导出-ts-声明文件)
    - [3.1 安装依赖](#31-安装依赖)
    - [3.2 修改 webpack.config.js 配置文件](#32-修改-webpackconfigjs-配置文件)
    - [3.3 babel.config.json、tsconfig.json 配置文件](#33-babelconfigjsontsconfigjson-配置文件)
    - [3.4 package.json 配置打包](#34-packagejson-配置打包)
  - [Bonus: 实时打包配置](#bonus-实时打包配置)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

平常我们写业务代码都是写一个项目，打包然后部署到机器上去就算了。然而每次写不同的项目的时候总是会存在很多重复性的业务代码，每次都用 ctrl c + ctrl v 大法也不是办法，同时对于各个项目又是在增加维护成本。这时候我们可以选择开发自己的 npm 包，然后在不同项目中分别引入相同的包，这样的包有的称为一个 library，也可以叫做一个 SDK（Software Development Kit）。

本篇的核心在于如何利用 webpack 对 library 进行打包并生成最后的可发布版本，并利用本地 link 的方式引用，并不涉及到真正的 npm 发包流程

# 正文

## 1. 第一版：源代码直接导出

首先我们先看到第一个最基础的版本，我们的源代码目录结构如下，非常之简单

### 1.1 目录结构 & 源代码

```tree
/library1
├── index.js
└── package.json
```

- `/library1/index.js`

```js
module.exports = greeting;

function greeting() {
  console.log('Hello World');
}
```

第一版的代码比较简单，就是先导出一个方法

### 1.2 package.json 配置

最重要的是在于 package.json 中属性的赋值，这是当一个 npm 包被引用的时候非常重要的部分

- `/library1/package.json`

```json
{
  "name": "webpack_library_1",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT"
}
```

- name：表示这个 npm 被引用的时候的包名，也是发布的时候在 npm 平台上会看到的包名
- version：带在包名后面，为引用的时候的唯一依据
- main：library 主入口，也就是引用这个 npm 项目的时候引用文件的主入口

### 1.3 创建本地软连接

接下来要使用这个包有几个方法，正常流程来说应该是要打包并发布到 npm 平台上，然后再通过 yarn add 下载下来。然而本篇并不打算涉及正式的发布流程，因此我们选用软连接的方式来引用本地包

首先我们先在 library1 项目目录下使用命令

```bash
# 在 library1 目录下
$ yarn link
yarn link v1.22.10
success Registered "webpack_library_1".
info You can now run `yarn link "webpack_library_1"` in the projects where you want to use this package and it will be used instead.
✨  Done in 0.06s.
```

我们可以看到名字为 webpack_library_1 的 npm 包已经在全局建立一个软连接，接下来我们可以再新建另一个项目，引用这个包，一样要用个命令来接上这个软连接

```bash
# 新建 usage 项目 ...
$ cd usage
$ yarn link webpack_library_1
yarn link v1.22.10
success Using linked package for "webpack_library_1".
✨  Done in 0.04s.
```

这样一来我们在使用 `import 'webpack_library_1'` npm/yarn 就会自动建立一个软连接到本地的全局仓库下，也就是直接引用我们本地的源文件

### 1.4 引入结果测试

- `/usage/src/libs/bindLibrary1.ts`

```ts
import greeting from 'webpack_library_1';

export default function run() {
  greeting();
}
```

我们可以看到就好像引用了一个完全是第三方的 npm 包一样

- 运行结果截图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_library_usage1.png)

## 2. 第二版：使用 webpack 打包

接下来我们再为 library 配置 webpack，让我们的 library 能经过正式的打包，进行依赖的导入、编译、压缩等手段，而这也是本片的主题，利用 webpack 来打包出一个 大家都可用的第三方 npm 包

### 2.1 安装依赖 & 目录结构

我们将在 library1 的基础上，安装 webpack 的相关依赖

```bash
# library2 目录下
$ yarn add webpack webpack-cli clean-webpack-plugin -D
```

而 library2 的目录结构如下

```tree
/library2
├── package.json
├── src
│   ├── index.js
│   └── other.js
├── webpack.config.js
└── yarn.lock
```

### 2.2 源代码扩展

接下来我们稍微扩展一下 library 的能力，创建一个新的模块 other.js

- `/library2/src/other.js`

```js
export const Log = {
  log: console.log,
};
```

导出一个 Log

- `/library2/src/index.js`

接下来从 index 模块将 other 模块的内容导出

```js
export { Log } from './other';
```

### 2.3 webpack.config.js 配置

最后我们补上 webpack 需要的配置文件，先来点简单的就行

- `/library2/webpack.config.js`

```js
const path = require('path');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = {
  mode: 'production',
  entry: './src/index',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'index.js',
    library: {
      name: '__webpack_library_2',
      type: 'umd',
    },
    globalObject: 'this',
  },
  resolve: {
    extensions: ['.js', '.ts', '.json'],
  },
  plugins: [new CleanWebpackPlugin()],
};
```

### 2.4 打包成果

最后在 package.json 文件中添加一条打包指令

- `/library2/package.json`

```json
{
    "scripts": {
        "build": "webpack"
    }
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_library_library2_build.png)

### 2.5 usage 项目中引入

最后跟第一版一样透过 yarn link 软连接的一套流程，我们就可以在 usage 项目中用上这个新的 npm 包

- `/usage/src/libs/bindLibrary2.ts`

```ts
import { Log } from 'webpack_library_2';

export default function run() {
  Log.log('Hello World by Log.log');
}
```

然后看一下结果哈

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_library_usage2.png)

## 3. 第三版：加上 babel 编译 & 导出 ts 声明文件

最后第三个版本，我们再为 library 加上 ts 编译，现在新的项目更多都是使用 typesciprt 作为编写语言了，前面我们已经加过 webpack 打包了，下面我们再额外加上 babel 编译和 typescript 的声明文件导出就行了

### 3.1 安装依赖

我们需要安装一些 babel 相关的依赖，以及 typescript 的编译器用于生成类型声明文件

```bash
# babel 相关
$ yarn add @babel/core @babel/preset-env @babel/preset-typescript babel-loader -D
# typescript 相关
$ yarn add typescript -D
```

### 3.2 修改 webpack.config.js 配置文件

- `/library3/webpack.config.js`

```js
module.exports = {
  // ...
  
  resolve: {
    extensions: ['.js', '.ts', '.json'],
  },
  module: {
    rules: [
      {
        test: /\.(js|ts)$/,
        loader: 'babel-loader',
      },
    ],
  },
  
  // ...
};
```

增加 resolve.extensions 使 webpack 能够解析 .ts 的文件；然后是为 .js、.ts 的文件加上 babel-loader 进行编译

### 3.3 babel.config.json、tsconfig.json 配置文件

另外还要加上 babel 和 ts 的相关配置文件

- `/library3/babel.config.json`

```json
{
  "presets": ["@babel/preset-env", "@babel/preset-typescript"]
}
```

- `/library3/tsconfig.json`

```ts
{
  "compilerOptions": {
    "emitDeclarationOnly": true,
    "declaration": true,
    "declarationDir": "types"
  }
}
```

这里的核心在于 `"emitDeclarationOnly": true` 的配置，由于我们已经使用 babel 的 preset-typescript 进行编译，实际上是不会用到 typescript编译器的，也不会生成我们想要的 .d.ts 类型声明文件；这时候我们就可以根据这个配置只生成声明文件，将源文件的编译和生成交给 babel+webpack 处理

### 3.4 package.json 配置打包

接下来我们需要稍稍修改打包的指令

- `/library3/package.json`

```json
{
    "main": "./dist/index.js",
    "types": "./types/index.d.ts",
    "scripts": {
        "build": "tsc && webpack",
        "build:watch": "webpack --watch"
    },
}
```

先用 tsc 加上刚才的配置，将类型声明文件生成到 types 目录下，之后引用这个包的人就会直接根据 types 属性指向的入口来获取声明文件中的类型定义(也就是 `"./types/index.d.ts"`)

最后我们就可以在引用包的时候同时看到类型定义

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_library_library3_emmet.png)

最后是运行结果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_library_usage3.png)

## Bonus: 实时打包配置

最后给出一个想法，具体的实现可以自己尝试，也可以到源代码中翻翻看。

要想实现多个包同时开发，并且实时打包编译，我们可以采用 `webpack --watch` 的观察模式，在源代码修改的时候实时打包，这样一来引用的主项目就会自动进行更新

# 结语

开发自己的 npm 包就好像象征着已经从初级的业务开发进阶到开始打造自己的开发工具一般。将一些实际业务项目中的产出总结，并抽象封装成更加好复用的模块对自己的提升是巨大的，供大家参考。

# 其他资源

## 参考连接

| Title                                                                        | Link                                                                                                                                                                                                                                         |
| ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 创建 library - webpack                                                       | [https://webpack.docschina.org/guides/author-libraries/](https://webpack.docschina.org/guides/author-libraries/)                                                                                                                             |
| 如何使用webpack打包一个库library                                             | [https://segmentfault.com/a/1190000021318631](https://segmentfault.com/a/1190000021318631)                                                                                                                                                   |
| Yarn local packages dependencies                                             | [https://stackoverflow.com/questions/48686053/yarn-local-packages-dependencies](https://stackoverflow.com/questions/48686053/yarn-local-packages-dependencies)                                                                               |
| Error: Cannot find module 'webpack'                                          | [https://stackoverflow.com/questions/29492240/error-cannot-find-module-webpack](https://stackoverflow.com/questions/29492240/error-cannot-find-module-webpack)                                                                               |
| TypeScript -- @babel/preset-typescript & ts-loader                           | [https://evanlouie.github.io/posts/typescript-babel-preset-typescript-ts-loader](https://evanlouie.github.io/posts/typescript-babel-preset-typescript-ts-loader)                                                                             |
| Webpack 转译 Typescript 现有方案                                             | [https://juejin.cn/post/6844904052094926855](https://juejin.cn/post/6844904052094926855)                                                                                                                                                     |
| webpack打包ts的两种方案对比                                                  | [https://juejin.cn/post/6844904160375078925#heading-8](https://juejin.cn/post/6844904160375078925#heading-8)                                                                                                                                 |
| Typescript + Webpack library generates "ReferenceError: self is not defined" | [https://stackoverflow.com/questions/64639839/typescript-webpack-library-generates-referenceerror-self-is-not-defined](https://stackoverflow.com/questions/64639839/typescript-webpack-library-generates-referenceerror-self-is-not-defined) |
| How to generate d.ts and d.ts.map files using webpack?                       | [https://stackoverflow.com/questions/55318663/how-to-generate-d-ts-and-d-ts-map-files-using-webpack](https://stackoverflow.com/questions/55318663/how-to-generate-d-ts-and-d-ts-map-files-using-webpack)                                     |
| javascript - 如何使用 webpack 生成 d.ts 和 d.ts.map 文件？                   | [https://www.coder.work/article/936583](https://www.coder.work/article/936583)                                                                                                                                                               |
| 【ts】TypeScript声明文件(.d.ts)                                              | [https://www.jianshu.com/p/7153d88fa232](https://www.jianshu.com/p/7153d88fa232)                                                                                                                                                             |
| Typescript无法解析npm包的定义文件                                            | [https://www.codenong.com/47984306/](https://www.codenong.com/47984306/)                                                                                                                                                                     |
| HtmlWebpackPlugin - webpack                                                  | [https://webpack.js.org/plugins/html-webpack-plugin/](https://webpack.js.org/plugins/html-webpack-plugin/)                                                                                                                                   |
| ReferenceError: HtmlWebpackPlugin is not defined #1453                       | [https://github.com/jantimon/html-webpack-plugin/issues/1453](https://github.com/jantimon/html-webpack-plugin/issues/1453)                                                                                                                   |
| JS模块规范：AMD、UMD、CMD、commonJS、ES6 module                              | [https://segmentfault.com/a/1190000012419990](https://segmentfault.com/a/1190000012419990)                                                                                                                                                   |
| 配置webpack的externals不打包第三方包                                         | [https://www.cnblogs.com/h2zZhou/p/12986126.html](https://www.cnblogs.com/h2zZhou/p/12986126.html)                                                                                                                                           |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/webpack/webpack_library](https://github.com/superfreeeee/Blog-code/tree/main/front_end/webpack/webpack_library)
