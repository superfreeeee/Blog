# Webpack 实战: 多入口项目打包 & 代码拆分实战分享

@[TOC](文章目录)

<!-- TOC -->

- [Webpack 实战: 多入口项目打包 & 代码拆分实战分享](#webpack-实战-多入口项目打包--代码拆分实战分享)
- [前言](#前言)
  - [多入口：多页面、跨端](#多入口多页面跨端)
- [正文](#正文)
  - [1. 单入口配置](#1-单入口配置)
    - [1.1 安装依赖](#11-安装依赖)
    - [1.2 单入口配置文件](#12-单入口配置文件)
    - [1.3 基础项目代码](#13-基础项目代码)
    - [1.4 初次打包 & 查看运行结果](#14-初次打包--查看运行结果)
  - [2. 多入口配置](#2-多入口配置)
    - [2.1 项目扩展](#21-项目扩展)
      - [2.1.1 核心模块扩展](#211-核心模块扩展)
      - [2.1.2 添加其他入口模块](#212-添加其他入口模块)
      - [2.1.3 添加其他入口模版](#213-添加其他入口模版)
    - [2.2 修改配置](#22-修改配置)
    - [2.3 打包并运行多模块项目](#23-打包并运行多模块项目)
  - [3. 代码拆分](#3-代码拆分)
    - [3.1 引入 lodash 作为第三方库的大文件示例](#31-引入-lodash-作为第三方库的大文件示例)
    - [3.2 再打包查看 lodash 打包情形](#32-再打包查看-lodash-打包情形)
    - [3.3 使用 splitChunks 代码分割实现打包优化](#33-使用-splitchunks-代码分割实现打包优化)
    - [3.4 代码分割后的打包情形](#34-代码分割后的打包情形)
    - [3.5 代码分割下的模块依赖](#35-代码分割下的模块依赖)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

今天来与大家分享 webpack 的多入口配置。在[前一篇：Webpack：入門](https://blog.csdn.net/weixin_44691608/article/details/106675550)中我们提到，webpack 能够从入口模块出发，自动解析不同模块之间(同时支持多种模块化规范)的静态和动态依赖关系，并将项目的最终版本代码打包成单一的 JS 文件，减少前端项目实际上线运行时需要引入的文件数量；同时，将静态依赖的文件模块打包成单一的 JS 文件，也有助于进行代码优化，以及与动态引入的脚本进行分离。

## 多入口：多页面、跨端

然而在一般情况下，我们通常只会配置一个单入口的文件，而这也是 SPA 项目的魅力所在。然而在真正的大型项目之中，我们总不能真的将整个网站的所有模块全部打包在一起，有时我们会需要根据不同的大入口来进行页面的区分；同时对于支持多端的项目，我们也需要为不同部署目标建立独立的入口脚本和页面。

上面提到的不管是 **多页面项目**，或是 **跨端项目** (许多移动端的页面运行时只支持多页面的形式)，事实上都是一种 **多入口项目** 的体现。由于网站本身的主要模块大部分其实是能够共用的，只不过是需要针对不同的入口(跨端、多页面)进行不同的配置、依赖关系的绑定。

这时候我们就需要利用 webpack 提供我们的多入口配置方式，webpack 就会为我们从不同的入口开始解析依赖和打包。下面我们就来看看实际操作的时候要如何配置 webpack。

# 正文

## 1. 单入口配置

一开始我们先配置一个与原本 SPA 相同的单入口项目

### 1.1 安装依赖

一样第一步先初始化 npm 项目并安装需要的依赖

- 初始化项目

```bash
$ mkdir webpack_multiple_entry
$ cd webpack_multiple_entry
$ yarn init -y
```

- 安装依赖

```bash
$ yarn add webpack webpack-cli -D  # webpack 核心依赖
$ yarn add html-webpack-plugin clean-webpack-plugin -D  # 必要插件
$ yarn add progress-bar-webpack-plugin webpack-manifest-plugin -D  # 其他可选插件
```

### 1.2 单入口配置文件

接下来是编写我们的配置文件

- `webpack.single.config.js`

```js
const path = require('path')

const { CleanWebpackPlugin } = require('clean-webpack-plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { WebpackManifestPlugin } = require('webpack-manifest-plugin')
const ProgressBarWebpackPlugin = require('progress-bar-webpack-plugin')

module.exports = {
  mode: 'production',
  entry: path.join(__dirname, 'src/entryA.js'),
  output: {
    path: path.join(__dirname, 'dist'),
    filename: '[name]-[chunkhash].js',
  },
  plugins: [
    new CleanWebpackPlugin(),
    new WebpackManifestPlugin(),
    new ProgressBarWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: `public/entryA.html`,
      filename: `entryA.html`,
      title: `Webpack 单入口配置 - entryA`,
    }),
  ],
}
```

我们以 `src/entryA.js` 为入口，将编译结果输出到 `dist` 目录下，并在 `HtmlWebpackPlugin` 中传入配置对象

### 1.3 基础项目代码

下面给出配置文件中使用的 html 模版以及基础代码文件，首先是 html 模版文件

- `/public/entryA.html`

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
    <h1>Webpack Multiple Entry - Entry A</h1>
  </body>
</html>
```

接下来是脚本内容

- `/src/entryA.js`

```js
import { group } from './modules/utils'
import { a } from './modules/a'
import { b } from './modules/b'

group('entryA', () => { a(); b() })
```

- `/src/modules/a.js`

```js
import { log } from './utils'

export function a() { log('invoke function a from src/modules/a.js') }
```

- `/src/modules/b.js`

```js
import { log } from './utils'

export function b() { log('invoke function b from src/modules/b.js') }
```

都是很简单的代码，过一下就好

### 1.4 初次打包 & 查看运行结果

下面我们就配置好 yarn 命令之后运行打包

```json
{
    // ...
    "scripts": {
        "build-single": "webpack --config webpack.single.config.js",
    }
    // ...
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_multiple_entry_single1.png)

并打开打包结果的 html 文件查看运行结果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_multiple_entry_single2.png)

这时候我们看到项目中的输出结果有这些文件：

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_multiple_entry_single3.png)

## 2. 多入口配置

前面我们建立好单入口(`entryA` 模块)之后，下面我们进入本篇的重点：多入口的文件配置

### 2.1 项目扩展

首先我们先假设项目进行了一定程度的扩展

#### 2.1.1 核心模块扩展

首先添加了 `c.js` 新模块

- `/src/modules/c.js`

```js
import { log } from './utils'

export function c() { log('invoke function c from src/modules/c.js') }
```

#### 2.1.2 添加其他入口模块

接下来我们假设项目需要两个新的入口模块用来应对不同的项目入口(多页面、跨端)，分别是 `entryB、entryC`

- `/src/entryB.js`

```js
import { group } from './modules/utils'
import { b } from './modules/b'
import { c } from './modules/c'

group('entryB', () => { b(); c() })
```

- `/src/entryC.js`

```js
import { group } from './modules/utils'
import { a } from './modules/a'
import { c } from './modules/c'

group('entryC', () => { a(); c() })
```

也就是说这时候整个项目的依赖关系如下图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_multiple_entry_multiple_dependency.png)

而对于不同入口，需要打包的依赖模块也是不同的

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_multiple_entry_multiple_dependency2.png)

#### 2.1.3 添加其他入口模版

当然有了 JS 模块作为入口之外，每个入口还需要一个独立的 html 模版

- `/public/entryB.html`

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
    <h1>Webpack Multiple Entry - Entry B</h1>
  </body>
</html>
```

- `/public/entryC.html`

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
    <h1>Webpack Multiple Entry - Entry C</h1>
  </body>
</html>
```

### 2.2 修改配置

接下来我们就透过修改 webpack 配置项里面的 `entry` 属性来将项目改造成多入口

- `webpack.multiple.config.js`

```js
const path = require('path')

const { CleanWebpackPlugin } = require('clean-webpack-plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { WebpackManifestPlugin } = require('webpack-manifest-plugin')
const ProgressBarWebpackPlugin = require('progress-bar-webpack-plugin')

module.exports = {
  mode: 'production',
  entry: {
    entryA: path.join(__dirname, 'src/entryA.js'),
    entryB: path.join(__dirname, 'src/entryB.js'),
    entryC: path.join(__dirname, 'src/entryC.js'),
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name]-[chunkhash].js',
  },
  plugins: [
    new CleanWebpackPlugin(),
    new WebpackManifestPlugin(),
    new ProgressBarWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: 'public/entryA.html',
      filename: 'entryA.html',
      title: 'Webpack 多入口配置 - entryA',
      chunks: ['entryA'],
    }),
    new HtmlWebpackPlugin({
      template: 'public/entryB.html',
      filename: 'entryB.html',
      title: 'Webpack 多入口配置 - entryB',
      chunks: ['entryB'],
    }),
    new HtmlWebpackPlugin({
      template: 'public/entryC.html',
      filename: 'entryC.html',
      title: 'Webpack 多入口配置 - entryC',
      chunks: ['entryC'],
    }),
  ],
}
```

简单来说我们就是将 `entry` 配置项改为一个对象，每一个键值对代表了一个入口的名称和入口模块路径(`chunkNmae: chunkEntry`)；然后配置多个 `HtmlWebpackPlugin` 实例，来为不同入口创建不同的 html 模版(透过 `chunks` 指定依赖模块)

然而上面的重复代码是不能被接受的，稍微简化一下写法

```js
const chunks = ['entryA', 'entryB', 'entryC']

module.exports = {
  mode: 'production',
  entry: {
    entryA: path.join(__dirname, 'src/entryA.js'),
    entryB: path.join(__dirname, 'src/entryB.js'),
    entryC: path.join(__dirname, 'src/entryC.js'),
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name]-[chunkhash].js',
  },
  plugins: [
    // others
    ...chunks.map(
      (name) =>
        new HtmlWebpackPlugin({
          template: `public/${name}.html`,
          filename: `${name}.html`,
          title: `Webpack 多入口配置 - ${name}`,
          chunks: [name],
        })
    ),
  ],
}
```

### 2.3 打包并运行多模块项目

接下来就可以进行打包了(启动命令定义我就省略了)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_multiple_entry_multiple1.png)

然后分别打开每个页面查看运行结果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_multiple_entry_multiple2.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_multiple_entry_multiple3.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_multiple_entry_multiple4.png)

而这时候的打包输出目录如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_multiple_entry_multiple5.png)

我们可以看到每个入口都产生了自己的 html 模版，以及各自对应的入口模块

## 3. 代码拆分

前面我们一共创建了三个入口，实际上我们看到打包后的各个模块入口如下

- `entryA-xxx.js`

```js
;(() => {
  'use strict'
  const o = console.log
  var n
  ;(n = () => {
    o('invoke function a from src/modules/a.js'),
      o('invoke function b from src/modules/b.js')
  }),
    console.group('entryA'),
    n(),
    console.groupEnd()
})()
```

- `entryB-xxx.js`

```js
;(() => {
  'use strict'
  const o = console.log
  var n
  ;(n = () => {
    o('invoke function b from src/modules/b.js'),
      o('invoke function c from src/modules/c.js')
  }),
    console.group('entryB'),
    n(),
    console.groupEnd()
})()
```

- `entryC-xxx.js`

```js
;(() => {
  'use strict'
  const o = console.log
  var n
  ;(n = () => {
    o('invoke function a from src/modules/a.js'),
      o('invoke function c from src/modules/c.js')
  }),
    console.group('entryC'),
    n(),
    console.groupEnd()
})()
```

我们可以看到，实际上 `a.js、b.js、c.js` 三个模块被重复打包进三个入口对应的入口模块中了。在代码量小的时候还能够接受，然而当我们引用一些代码量比较大的第三方模块的时候，重复打包会显得有些浪费。

### 3.1 引入 lodash 作为第三方库的大文件示例

因此下面我们引入 lodash 库作为较大的第三方库并加入打包作为示例

首先安装依赖

```bash
$ yarn add lodash
```

接下来我们修改一下 `a.js` 这个模块，在其中引入 lodash 库

- `/src/modules/a.js`

```js
import { log } from './utils'
import _ from 'lodash'

log(`load module a with lodash ${_.VERSION}`)

export function a() { log('invoke function a from src/modules/a.js') }
```

### 3.2 再打包查看 lodash 打包情形

接下来我们直接使用前面配置好的命令进行打包

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_multiple_entry_split1.png)

从打包日志我们可以看到 `entryA、entryC` 的模块大小暴涨到接近 70KB，这是因为事实上 lodash 同时被编译到了两个模块当中了，下面我们看看打包结果就知道了

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_multiple_entry_split2.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_multiple_entry_split3.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_multiple_entry_split4.png)

我们可以看到除了 entryB 还与原本相似之外，entryA、entryC 都多了好多代码，而这正是 lodash 被打包进来之后的样子

### 3.3 使用 splitChunks 代码分割实现打包优化

然而前面这样子是不可取的，lodash 的代码实际上并不需要复制一次，只要两个相关的页面有正确的引入就行。这时候我们就可以借助 webpack 的 `splitChunks` 配置选项(本质上是一个内置的插件)启用代码分割来进行打包优化

事实上在 webpack4+ 的环境下，默认就会对打包代码进行分割，然而需要满足以下条件

- 共享代码块 or `node_modules` 中的代码块，同时体积必须超过 30KB
- 按需加载并行请求不超过 5 个
- 并行初始页面加载不超过 3 个

后两个条件目前体会比较小，主要核心条件在于第一条：**代码体积需要超过 30KB**，而这也是本次测试为何需要额外引入 lodash 的原因

接下来我们修改一下 webpack 的配置，加上代码分隔的优化方案

- `webpack.config.js`

```js
const path = require('path')

const { CleanWebpackPlugin } = require('clean-webpack-plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { WebpackManifestPlugin } = require('webpack-manifest-plugin')
const ProgressBarWebpackPlugin = require('progress-bar-webpack-plugin')

const chunks = ['entryA', 'entryB', 'entryC']

module.exports = {
  mode: 'production',
  entry: {
    entryA: path.join(__dirname, 'src/entryA.js'),
    entryB: path.join(__dirname, 'src/entryB.js'),
    entryC: path.join(__dirname, 'src/entryC.js'),
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name]-[chunkhash].js',
  },
  plugins: [
    new CleanWebpackPlugin(),
    new WebpackManifestPlugin(),
    new ProgressBarWebpackPlugin(),
    ...chunks.map(
      (name) =>
        new HtmlWebpackPlugin({
          template: `public/${name}.html`,
          filename: `${name}.html`,
          title: `Webpack 多入口配置 - ${name}`,
          chunks: [name],
        })
    ),
  ],
  optimization: {
    splitChunks: {
      chunks: 'all',
    },
  },
}
```

我们先来解说一下最后一块的配置项

```js
module.exports = {
  // ...
  optimization: {
    splitChunks: {
      chunks: 'all',
    },
  },
}
```

`splitChunks` 是内置插件的配置项键名，它里面有好多关于代码分隔的配置选项，目前我们看到的就是一个
`chunks` 有几个可选值

- `async(默认)`：只对异步加载的模块进行拆分
- `initial`：只从入口模块进行拆分
- `all`：不论如何满足条件(前面提过的三个默认条件)的都要拆分

也就是说 3.2 节的例子 lodash 复制了一份的原因是因为他是同步引入的模块

### 3.4 代码分割后的打包情形

下面我们就来看看执行代码分割之后的打包情形

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_multiple_entry_split_after1.png)

我们可以看到 entryA、entryC 的体积缩小了好多，然后出现了一个新的模块 486xxx，可以猜到这就是我们的 lodash 模块被打包后的 chunk。下面是打包后的目录结果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_multiple_entry_split_after2.png)

同时我们可以从打包后的 html 文件看到不同入口依赖的模块

- `/dist/entryA.html`

```js
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width,initial-scale=1" />
    <title>Webpack 多入口配置 - entryA</title>
    <script defer="defer" src="486-25ae42129fb8db535b06.js"></script>
    <script defer="defer" src="entryA-6411ec0ba12b719a045d.js"></script>
  </head>
  <body>
    <h1>Webpack Multiple Entry - Entry A</h1>
  </body>
</html>
```

- `/dist/entryB.html`

```js
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width,initial-scale=1" />
    <title>Webpack 多入口配置 - entryB</title>
    <script defer="defer" src="entryB-43578622041fe38a4610.js"></script>
  </head>
  <body>
    <h1>Webpack Multiple Entry - Entry B</h1>
  </body>
</html>
```

- `/dist/entryC.html`

```js
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width,initial-scale=1" />
    <title>Webpack 多入口配置 - entryC</title>
    <script defer="defer" src="486-25ae42129fb8db535b06.js"></script>
    <script defer="defer" src="entryC-a72ea5d71395792a8026.js"></script>
  </head>
  <body>
    <h1>Webpack Multiple Entry - Entry C</h1>
  </body>
</html>
```

我们看到 entryA、entryC 引入了这个 `486xxx` 的 chunk 看来是符合我们对于他是 lodash 打包结果的猜测，我们再看看页面结果输出

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_multiple_entry_split_after3.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_multiple_entry_split_after4.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_multiple_entry_split_after5.png)

我们可以看到 entryA、entryC 确实额外打印了 lodash 的版本

### 3.5 代码分割下的模块依赖

最后我们给出一张在代码分割的打包优化下的模块依赖关系

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_multiple_entry_split_dependency.png)

webapck 会根据我们的依赖关系进行代码分割，先将原始的模块(module)进行拆分/合并成一个个 chunks，最后页面只需要引入必须的 chunks 就好了。

# 结语

以上就是 webpack 多入口项目打包实战分享。关于 webpack 的应用还是有非常多的发展空间，本篇介绍的多入口配置适合于中大型的项目，多入口也能够在项目中期代码量达到一定的需求以及功能的跟进之后，再进行多入口分离也无不可。供大家参考～

# 其他资源

## 参考连接

| Title                                                  | Link                                                                                                                                                                                                 |
| ------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 手摸手 Webpack 多入口配置实践                          | [https://segmentfault.com/a/1190000020351701](https://segmentfault.com/a/1190000020351701)                                                                                                           |
| webpack如何配置多入口多出口打包                        | [https://blog.csdn.net/weixin_39162041/article/details/104429517?utm_source=app&app_version=4.7.1](https://blog.csdn.net/weixin_39162041/article/details/104429517?utm_source=app&app_version=4.7.1) |
| [webpack 学习系列] bundle splitting，code splitting    | [https://www.jianshu.com/p/3c998c6d9f1e](https://www.jianshu.com/p/3c998c6d9f1e)                                                                                                                     |
| 详解webpack4之splitchunksPlugin代码包分拆              | [https://www.jb51.net/article/151976.htm](https://www.jb51.net/article/151976.htm)                                                                                                                   |
| webpack4系列教程（六）：使用SplitChunksPlugin分割代码  | [https://www.jianshu.com/p/2cc8457f1a10](https://www.jianshu.com/p/2cc8457f1a10)                                                                                                                     |
| 理解webpack4.splitChunks之chunks                       | [https://www.cnblogs.com/kwzm/p/10314827.html](https://www.cnblogs.com/kwzm/p/10314827.html)                                                                                                         |
| webpack4 optimization.splitChunks的注意点              | [https://blog.csdn.net/sunq1982/article/details/81511848](https://blog.csdn.net/sunq1982/article/details/81511848)                                                                                   |
| 在webpack中，module、chunk和bundle到底是什么样的存在？ | [https://zhuanlan.zhihu.com/p/98677441](https://zhuanlan.zhihu.com/p/98677441)                                                                                                                       |
| webpack 中，module，chunk 和 bundle 的区别是什么？     | [https://www.cnblogs.com/skychx/p/webpack-module-chunk-bundle.html](https://www.cnblogs.com/skychx/p/webpack-module-chunk-bundle.html)                                                               |
| webpack 插件总结归类                                   | [https://segmentfault.com/a/1190000016816813?utm_source=tag-newest#articleHeader3](https://segmentfault.com/a/1190000016816813?utm_source=tag-newest#articleHeader3)                                 |
| webpack4 常用插件列表及使用说明                        | [https://segmentfault.com/a/1190000015355816](https://segmentfault.com/a/1190000015355816)                                                                                                           |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/webpack/webpack_multiple_entry](https://github.com/superfreeeee/Blog-code/tree/main/front_end/webpack/webpack_multiple_entry)
