# React 项目启动2：使用 webpack 手动创建 React 项目(附加 React Router + Redux)

@[TOC](文章目录)

<!-- TOC -->

- [React 项目启动2：使用 webpack 手动创建 React 项目(附加 React Router + Redux)](#react-项目启动2使用-webpack-手动创建-react-项目附加-react-router--redux)
- [前言](#前言)
- [正文](#正文)
  - [1 项目依赖](#1-项目依赖)
    - [1.1 选用技术栈](#11-选用技术栈)
    - [1.2 依赖包](#12-依赖包)
    - [1.3 依赖包版本号](#13-依赖包版本号)
    - [1.4 依赖包版本选用说明](#14-依赖包版本选用说明)
      - [1.4.1 `webpack-cli` & `webpack-dev-server`](#141-webpack-cli--webpack-dev-server)
      - [1.4.2 React Router](#142-react-router)
  - [2 项目初始化配置](#2-项目初始化配置)
    - [2.1 webpack 配置](#21-webpack-配置)
      - [2.1.1 入口 entry、出口 output、虚拟服务器 devServer](#211-入口-entry出口-output虚拟服务器-devserver)
      - [2.1.2 解析 resolve(路径别名、识别 .jsx 文件)](#212-解析-resolve路径别名识别-jsx-文件)
      - [2.1.3 模块加载 Loader](#213-模块加载-loader)
      - [2.1.4 插件 Plugins](#214-插件-plugins)
      - [2.1.5 index.html 默认页面生成模版](#215-indexhtml-默认页面生成模版)
      - [2.1.6 webpack 完整配置](#216-webpack-完整配置)
    - [2.2 babel 配置](#22-babel-配置)
    - [2.3 scripts 脚本(package.json)](#23-scripts-脚本packagejson)
  - [3 项目基础内容](#3-项目基础内容)
    - [3.1 应用核心页面 App](#31-应用核心页面-app)
    - [3.2 入口文件 index.js](#32-入口文件-indexjs)
    - [3.3 基础项目构建完成](#33-基础项目构建完成)
  - [4 使用 React Router](#4-使用-react-router)
    - [4.1 react-router-dom & react-router](#41-react-router-dom--react-router)
    - [4.2 路由定义](#42-路由定义)
    - [4.3 前端路由实现下页面刷新 404 Not Found 问题](#43-前端路由实现下页面刷新-404-not-found-问题)
    - [4.4 引入主页面 & 展示效果](#44-引入主页面--展示效果)
  - [5 使用 Redux](#5-使用-redux)
    - [5.1 核心概念](#51-核心概念)
    - [5.2 在全局状态之前](#52-在全局状态之前)
    - [5.3 Action](#53-action)
    - [5.4 Reducers](#54-reducers)
    - [5.5 Store 状态管理](#55-store-状态管理)
    - [5.6 React Redux](#56-react-redux)
      - [5.6.1 Provider](#561-provider)
      - [5.6.2 connect(mapStateToProps, mapDispatchToProps)](#562-connectmapstatetoprops-mapdispatchtoprops)
      - [5.6.3 使用 Redux 的计数器](#563-使用-redux-的计数器)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码实现](#完整代码实现)

<!-- /TOC -->

# 前言

之前写过一篇 <a href="https://blog.csdn.net/weixin_44691608/article/details/106516736">React 項目啟動：create-react-app</a>，用的是 `create-react-app` 脚手架来创建 react 项目。虽然用起来是挺方便，而且环境，依赖包都初始化好了，但是还是存在几个问题

- 初始包体积过大：透过模版生成的项目可能包含了我们不需要的第三方库导致体积虚增
- 配置困难：由于默认情况下，cra 的 webpack 配置是隐藏的，我们需要使用 `npm run eject` 来暴露默认配置；同时由于存在默认配置和一些第三方库的使用，对于不常使用 cra 的小伙伴来说还必须花费额外的代价来理解和维护默认的配置，并在此基础上进行扩展，那还不如自己重新创一个，干净又省力

因此本篇就带大家来一遍手动创建 React 项目，配合服用 webpack + babel 的编译 & 打包工具，不仅是提升对于 React 项目的掌控性，同时练习在大型项目中自定义配置和项目构建的必备技能，下面我们马上开始。

# 正文

## 1 项目依赖

首先在编写任何一个项目之前，最重要没有之一的一件事就是先确定要使用的核心依赖和核心依赖的版本，由于依赖的包可能还在持续更新、发布，对于其功能的增删改都会在整个项目的开发过程中造成巨大的影响。

所以在开始项目之前我们先来确定一下下面要用的各个依赖有哪些，分别又是哪些版本

### 1.1 选用技术栈

首先明确一下，本次尝试创建的项目使用以下技术栈

- MVVM 框架：React
- 路由方案：React Router
- 状态管理：Redux(React Redux)
- 编译工具：babel
- 打包工具：webpack

### 1.2 依赖包

- react 核心依赖

```bash
$ yarn add react react-dom  # 核心中的核心的两个包
```

- React Router & Redux

```bash
$ yarn add react-router-dom  # react-router 也行
$ yarn add redux react-redux  # redux 核心和 react 版本的增强 react-redux
```

- webpack

```bash
$ yarn add webpack webpack-cli webpack-dev-server -D  # webpack 核心 & 命令 & 开发用虚拟服务器
$ yarn add style-loader css-loader file-loader csv-loader xml-loader -D  # 各个 loader
$ yarn add webpack-manifest-plugin html-webpack-plugin clean-webpack-plugin -D  # 各个 plugin
```

- babel

```bash
$ yarn add @babel/core @babel/preset-react babel-loader -D  # babel 核心、react 预设、loader for webpack
```

### 1.3 依赖包版本号

- `package.json`

```json
{
  // ...
  "dependencies": {
    "react": "^17.0.2",
    "react-dom": "^17.0.2",
    "react-router-dom": "^5.2.0",
    "redux": "^4.1.0",
    "react-redux": "^7.2.4"
  },
  "devDependencies": {
    "@babel/core": "^7.14.0",
    "@babel/preset-react": "^7.13.13",
    "babel-loader": "^8.2.2",
    "css-loader": "^5.2.4",
    "csv-loader": "^3.0.3",
    "file-loader": "^6.2.0",
    "style-loader": "^2.0.0",
    "xml-loader": "^1.2.1",
    "webpack": "^5.36.1",
    "webpack-cli": "^3.3.12",
    "webpack-dev-server": "^3.11.2",
    "webpack-manifest-plugin": "^3.1.1",
    "html-webpack-plugin": "^5.3.1",
    "clean-webpack-plugin": "^4.0.0-alpha.0"
  }
}
```

### 1.4 依赖包版本选用说明

基本上所有依赖就是选用当前最新的版本(本篇发布时最新的版本)，需要特别注意的有

#### 1.4.1 `webpack-cli` & `webpack-dev-server`

奇怪的是都是 webpack 自家人却搭不起来，如果使用了 `webpack-dev-server@3.x.x` 时，`webpack-cli@4.x.x` 是不能用的，会一直重复报一些包不能存在或其他错误，这时候需要改回使用前一个大版本 `webpack-cli@3.3.12`

#### 1.4.2 React Router

React Router 在使用的时候，具体能够引用的包有两种：`react-router-dom`、`react-router`。其实核心都是 `react-router-dom`，而 `react-router` 则是对其进行了一些封装，本篇选择直接使用 `react-router-dom`

如果你比较偏好使用 `react-router` 的话，需要注意像是 `BrowserRouter` 等组件需要在 `react-router@3.x.x` 以前，`react-router@4.x.x` 以后的版本中 `Router` 的用法又不太一样了，这是需要注意的坑

## 2 项目初始化配置

弄好依赖的引入之后，我们可以开始项目的初始配置

### 2.1 webpack 配置

首先在根目录下面建立 `webpack.config.js` 作为 webpack 的配置文件，下面的配置都在该文件中进行。webpack 的详细配置意义请参阅官方文档说明，本篇仅仅只会对配置项进行说明

#### 2.1.1 入口 entry、出口 output、虚拟服务器 devServer

首先是入口模块和打包后文件的配置：

模块入口为 `src/index.js`，打包后的 js 文件以名字和生成结果的哈希值(`[name]-[chunkhash]`)进行命名

- `webpack.config.js`

```js
module.exports = {
  mode: 'development',
  entry: path.join(__dirname, 'src/index.js'),
  output: {
    path: path.join(__dirname, 'dist'),  // 打包目标路径
    filename: '[name].[chunkhash].js',
  },
  devServer: {
    contentBase: './dist',  // 虚拟服务器依赖路径
  },
  // ...
```

#### 2.1.2 解析 resolve(路径别名、识别 .jsx 文件)

接下来是设置路径别名(例如 vue 脚手架默认配置 `@` 映射为 `/src` 等)，本篇就是为 `@` 符号设置了别名(使用 `resolve.alias` 配置)；

再来是默认情况下 webpack 会将未添加扩展名的引入都视为 `.js` 副档名，然而 React 应用中我们常用 `.jsx` 的文件来使用 JSX 的特性，因此需要额外添加默认的搜索扩展名(使用 `resolve.extensions` 配置)

- `webpack.config.js`

```js
module.exports = {
  // ...
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src'),
    },
    extensions: ['.js', '.jsx', '.json'],
  },
  // ...
```

#### 2.1.3 模块加载 Loader

接下来配置 webpack 的各个 loader。在 webpack 的体系下，一切皆模块，但是又只认识 js，所以我们可以透过各式各样的 loader 来解析其他格式的文件，并转化为等值有效的 js 模块完成打包目标

- `webpack.config.js`

```js
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        use: 'babel-loader',
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
      {
        test: /\.(png|svg|jpg|gif)$/,
        use: ['file-loader'],
      },
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/,
        use: ['file-loader'],
      },
      {
        test: /\.(csv|tsv)$/,
        use: ['csv-loader'],
      },
      {
        test: /\.xml$/,
        use: ['xml-loader'],
      },
    ],
  },
```

`test` 为扩展名的匹配，`use` 指定匹配副档名配对使用的 loader 名称

#### 2.1.4 插件 Plugins

第四个是配置一些插件，本篇使用了三个插件：

> `html-webpack-plugin`：为项目自动插入 html 文件作为 js 模块载体
> `webpack-manifest-plugin`：显示 webpack 打包的详细信息
> `clean-webpack-plugin`：重新打包后清理不必要的旧资源

- `webpack.config.js`

```js
module.exports = {
  // ...
  plugins: [
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: '/public/index.html',
      title: '手动搭建 React 项目'
    }),
    new WebpackManifestPlugin(),
    new CleanWebpackPlugin()
  ],
  // ...
```

#### 2.1.5 index.html 默认页面生成模版

最后我们可以给出一个默认的 html 文件作为 `html-webpack-plugin` 生成文件的模版(可以引入一些 webpack 参数来扩展配置)

- `/public/index.html`

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
    <div id="app"></div>
  </body>
</html>
```

#### 2.1.6 webpack 完整配置

- `webpack.config.js`

```js
const path = require('path')

const HtmlWebpackPlugin = require('html-webpack-plugin')
const { WebpackManifestPlugin } = require('webpack-manifest-plugin')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

module.exports = {
  mode: 'development',
  entry: path.join(__dirname, 'src/index.js'),
  output: {
    path: path.join(__dirname, 'dist'),
    filename: '[name].[chunkhash].js',
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src'),
    },
    extensions: ['.js', '.jsx', '.json'],
  },
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        use: 'babel-loader',
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
      {
        test: /\.(png|svg|jpg|gif)$/,
        use: ['file-loader'],
      },
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/,
        use: ['file-loader'],
      },
      {
        test: /\.(csv|tsv)$/,
        use: ['csv-loader'],
      },
      {
        test: /\.xml$/,
        use: ['xml-loader'],
      },
    ],
  },
  plugins: [
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: '/public/index.html',
      title: '手动搭建 React 项目'
    }),
    new WebpackManifestPlugin(),
    new CleanWebpackPlugin()
  ],
  devServer: {
    contentBase: './dist',
  },
}
```

值得注意的是 `webpack-manifest-plugin` 和 `clean-webpack-plugin` 的引入方式，一个小坑

```js
const { WebpackManifestPlugin } = require('webpack-manifest-plugin')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
```

### 2.2 babel 配置

babel 的配置比较简单，而且前面已经在 webpack 的 loader 部分指定了为 `.js`、`.jsx` 文件进行解析，所以主要就是再在根目录下创建 `babel.config.json` 指定编译预设(prset)就行了

- `babel.config.json`

```json
{
  "presets": ["@babel/react"]
}
```

### 2.3 scripts 脚本(package.json)

最后一项配置就是 `package.json` 中关于开发、打包的脚本设置：

1. 开发环境下我们使用 `webpack-dev-server` 命令启动虚拟服务器进行开发
  - `--hot` 表示尝试模块热更新
  - `--inline` 刷新模式
  - `--history-api-fallback` 避免刷新后 404 的问题
2. 生产环境下直接使用 `webpack` 命令

- `package.json`

```json
{
  // ...
  "scripts": {
    "start": "webpack-dev-server --hot --inline --history-api-fallback",
    "build": "webpack --config webpack.config.js"
  },
  // ...
}
```

## 3 项目基础内容

到目前为止我们仅仅只是对 React 进行了基础的运行配置，所以我们目前就先实现一个再简单不过的页面作为示例，能跑就好

### 3.1 应用核心页面 App

- `App.jsx`

```js
import React from 'react'

function App() {
  return (
    <div>
      <h1>Hello World</h1>
      <h3>Include React Router & Redux usage</h3>
      <h3>Build by webpack + Babel</h3>
    </div>
  )
}

export default App
```

一开始先给出一个简单基本的 Hello World 页面

### 3.2 入口文件 index.js

接下来是在入口文件将 App 组件挂载到 dom 上

- `index.js`

```js
import React from 'react'
import ReactDOM from 'react-dom'

import App from './App'

ReactDOM.render(
  <App></App>,
  document.getElementById('app')
)
```

### 3.3 基础项目构建完成

最后使用 `yarn start` 启动前端就完成啦！一个纯粹的 React 项目大功告成啦！

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_create_manual_basic.png)

## 4 使用 React Router

基础建设(React 核心、babel + webpack 配置)都已经搭建完成了，接下来就是继续扩展，使用前端开发的两个好朋友：路由 & 状态管理，我们从路由开始

### 4.1 react-router-dom & react-router

在准备引入 React Router 的时候发现有两个库都能用：`react-router-dom` 和 `react-router`

仔细查阅相关资料之后知道了其实 react-router 是对于 react-router-dom 的再封装，然而说实在没太大感觉，原本的 react-router-dom 其实也不算太难用，所以本篇决定直接使用 react-router-dom 包

### 4.2 路由定义

首先是在源代码目录下创建 `/router` 子目录，用于放置各种路由，这边只定义了一个作为主界面的路由

- `/router/Main.jsx`

```js
import React from 'react'
import { BrowserRouter, Link, Route, Switch } from 'react-router-dom'
import About from '../pages/about'
import Home from '../pages/home'

function Main() {
  const defaultLink = <Link to="/">Default</Link>
  const homeLink = <Link to="/home">Home</Link>
  const aboutLink = <Link to="/about">About</Link>
  const otherLink = <Link to="/other">Other</Link>

  return (
    <BrowserRouter>
      {defaultLink} | {homeLink} | {aboutLink} | {otherLink}
      <Switch>
        <Route exact path="/">
          <h2>Default Content</h2>
        </Route>
        <Route exact path="/home" component={Home}></Route>
        <Route exact path="/about" component={About}></Route>
        <Route>
          <h2>Unknown, Redirect to 404</h2>
        </Route>
      </Switch>
    </BrowserRouter>
  )
}

export default Main
```

使用要点：

- `BrowserRouter` 使用 HTML5 的 History API 来作为路由管理，其实也就是 Vue 里面的 history mode，关于前端路由可以看我之前写过的一篇：<a href="https://blog.csdn.net/weixin_44691608/article/details/115462057">JS 前端路由：单页面应用的路由原理和实现</a>
- `Switch`、`Route` 组件合作定义了且唯一匹配的路由视图，使用 `to` 指定路由、`component` 指定组件
  - 在 `Switch` 的最后放置一个默认路由相当于是对于其他任意错误路径的匹配定向到 404 Not Found

### 4.3 前端路由实现下页面刷新 404 Not Found 问题

先不论 hash 模式的前端路由，在 history 模式下前端路由的访问看起来就好像真的 URL 访问一样，所以如果直接将路径输入浏览器或透过其他手法重定向到 SPA 内的其他非默认页面，会产生 404 Not Found 的问题。然而这个并不是前端路由内部能够解决的问题，而是与前端项目部署和启动的静态资源服务器的配置相关，可以参考 <a href="https://blog.csdn.net/hsany330/article/details/90670186">React-Router browserHistory浏览器刷新出现页面404解决方案</a> 给出的解决方案

本篇由于只是启动在测试环境之下，所以直接选择使用 webpack-dev-server 的 `--history-api-fallback` 启动参数来解决

### 4.4 引入主页面 & 展示效果

最后就是将路由组件插入到主页面当中

- `App.jsx`

```js
// ...

function App() {
  return (
    <div>
      {/* others */}
      <hr />
      <h1>React Router</h1>
      <Main></Main>
    </div>
  )
}

export default App
```

- 效果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_create_manual_router1.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_create_manual_router2.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_create_manual_router3.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_create_manual_router4.png)

我们可以看到当我们点击不同路由的时候，就会根据 `Route` 的匹配规则渲染对应的组件

由于本篇不是要详细解释 React Router 的使用方法，所以点到为止。下面引入 Redux 作为状态管理。

## 5 使用 Redux

Redux 相对来说是一个较为复杂的概念，而且原本的 Redux 是独立于 React 使用的状态管理库，我们甚至可以在 Vue 中使用 Redux 作为状态管理。下面我们仅仅只是简单介绍一下在 React 中使用 Redux 的基础用法，更进阶的应用方式会在之后特别提出来写一篇

### 5.1 核心概念

Redux 的状态管理与其他状态管理库相似，就是在任意时刻都维护一个稳定的状态对象，并且所有的状态更新都需要透过统一的 **动作(Action)** 来完成，而不同动作的前后顺序和更新时机不应该互相重叠，也就是由 Redux 来给我们一个承诺：

> 在任意时间点只会存在一个静态且唯一的 **全局状态快照(state)**，并且一个快照应该对应唯一一个 **视图(view)**，这也是我们使用全局状态管理的终极目标。

下面是 Redux 的运行时动态组织结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_create_manual_redux_structure.png)

- `Store`：维护一个全局状态
- `React Component`：根据状态绘制视图(view)，即页面逻辑
- `Action`：表示一个状态更新的行为(请求)，由视图(React Component)发起，可以附带动作类型(action.type)和任意参数(payload)，所以通常我们会为每个 action 定义一个 `Action Creator` 来为不同的行为自动的创建 Action 对象
- `Reducers`：Action 仅仅只是更新动作的一个声明表示，Reducers 才是真正根据指定动作计算更新并返回新的状态真正对象。通常在状态管理中我们要求需要返回一个全新的对象，这个就跟双向绑定原理相关和观察者更新相关的部分不做解释，返回不同的对象就是了！

### 5.2 在全局状态之前

在使用 Redux 之前，我们先来看看不使用全局状态管理的时候的计数器是怎么实现的：

- `/pages/counter/index.jsx`

```js
function Counter() {
  const [count, setCount] = useState(0)
  const increment = () => setCount(count + 1)
  const reset = () => setCount(0)
  return (
    <div>
      <h2>Counter Page</h2>
      <h4>count: {count}</h4>
      <button onClick={increment}>Increment</button>
      <button onClick={reset}>Reset</button>
    </div>
  )
}
```

这里使用 React Hooks 的方式来声明一个函数组件，定义了 `count` 状态和 `increment`、`reset` 两个操作函数，看起来一切是那么的理所当然。

但是设想一个情况：有一天我们需要在该组件外部对组件内部的状态进行管理，可能是引用、改变、甚至介入对于 `count` 状态的状态管理，这时候我们就要透过不断的将 state 提升到高层共同组件，再一步步透过 props 单向数据流流到目标组件，真是有够麻烦，而这也是我们使用全局状态管理的初衷。

### 5.3 Action

接下来的 Redux 使用配置，我们从定义 **状态(state)** 跟 **动作(Action)** 开始

- `/store/CounterAction.js`

```js
export const CounterActionType = {
  INC: Symbol('increment'),
  RESET: Symbol('reset'),
  TOGGLE: Symbol('toggle'),
}

export const incrementAction = () => ({ type: CounterActionType.INC })

export const resetAction = () => ({ type: CounterActionType.RESET })

export const toggleVisibleAction = () => ({ type: CounterActionType.TOGGLE })
```

首先我们定义了三种操作行为：

- `incrementAction`：自增(count + 1)
- `resetAction`：重置(count = 0)
- `toggleVisibleAction`：展示/隐藏计数器

### 5.4 Reducers

接下来我们定义 Reducers 以及具体动作对应的实际状态更新方法

- `/store/CounterReducer.js`

```js
import { CounterActionType } from './CounterAction'

const defaultState = {
  count: 0,
  visible: true,
}

const updateState = (target, source) => Object.assign({}, target, source)

const counterIncrement = (state) => ({ count: state.count + 1 })

const counterReset = () => ({ count: 0 })

const toggleVisible = (state) => ({ visible: !state.visible })

const counterReducer = (state = defaultState, action) => {
  let newState
  switch (action.type) {
    case CounterActionType.INC:
      newState = counterIncrement(state)
      break
    case CounterActionType.RESET:
      newState = counterReset()
      break
    case CounterActionType.TOGGLE:
      newState = toggleVisible(state)
      break
    default:
      newState = state
  }
  return updateState(state, newState)
}

export default counterReducer
```

`defaultState` 为默认情况下的状态，而 `counterReducer` 为我们的 Reducers 主体，根据指定的动作类型(action.type)将请求转发给具体的 更新函数逻辑并返回新的 **状态对象(newState)**。

注意：我们透过 `updateState` 包装 `Object.assign` 的调用，来保证所有状态函数返回新的状态对象

### 5.5 Store 状态管理

用过 Vue 的就知道，store 就是我们最核心的状态管理总机啦(虽然应该是 React 先出来而参考 Redux 才实现的 Vuex，不过作者倒是先学了 Vue 才回头来看 React 的)

- `/store/index.js`

```js
import { createStore } from 'redux'
import CounterReducer from './CounterReducer'

const store = createStore(CounterReducer)

export default store
```

我们使用 Redux 提供的 `createStore` 方法来根据 Reducer 创建好全局状态管理对象，组件或是其他部分的人只需要透过调用 store 的方法就能得到全局状态的快照/修改全局状态啦！store 有以下几个方法：

- `store.getState()`：获得全局状态快照，也就是获得某个时间点的 state
- `store.dispatch()`：发起修改行为，也就是进行状态的更新，传入 Action，并交由 Reducer 来执行实际的状态更新逻辑，并返回新的状态
- `store.subscribe()`：由于 React 全权代理了所有 dom 的操作，所以我们需要使用 `subscribe` 方法直接让组件订阅状态的变化以保证视图和状态的同步更新

### 5.6 React Redux

然而直接使用最原始的 Redux 还是有点难用，所以我们通常会与 `react-redux` 连用，使得 Redux 无缝插入 React 的逻辑体系之中

#### 5.6.1 Provider

首先第一步是将我们刚刚创建好的 store 对象绑定到全局，也就是根组件上，使用了 `<Provider>` 组件作为外包对象

- `index.js`

```js
// ...

ReactDOM.render(
  <Provider store={store}>
    <App></App>
  </Provider>,
  document.getElementById('app')
)
```

#### 5.6.2 connect(mapStateToProps, mapDispatchToProps)

接下来我们可以使用 `connect(a, b)(Component)` 函数对组件进行扩展，使函数组件可以直接透过 `props` 拿到全局状态和更新操作方法

- `/pages/counter/index.jsx`

```js
const mapStateToProps = (state) => {
  const { count, visible } = state
  return { count, visible }
}

const mapDispatchToProps = (dispatch) => {
  return {
    incrementAction: () => dispatch(incrementAction()),
    resetAction: () => dispatch(resetAction()),
    toggleVisibleAction: () => dispatch(toggleVisibleAction()),
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(Counter)
```

我们可以看到 `connect` 方法的第一次调用需要传入两个方法

- `mapStateToProps`：全局状态 state 到 props 的映射，也就是将全局状态映射到 props 属性的方法
- `mapDispatchToProps`：状态更新 action 到 props 的映射，允许组件直接透过 props 发起全局状态的更新

#### 5.6.3 使用 Redux 的计数器

接下来我们就可以直接透过 props 来间接引用全局状态(`mapStateToProps` 相当于直接为组件定义客制化的 getters)和修改全局状态

- `/pages/counter/index.jsx`

```js
import React, { useState } from 'react'
import { connect } from 'react-redux'
import { incrementAction, resetAction } from '@/store/CounterAction'
import { toggleVisibleAction } from '../../store/CounterAction'

function Counter({
  count,
  visible,
  incrementAction,
  resetAction,
  toggleVisibleAction,
}) {
  return (
    <div>
      <h2>Counter Page</h2>
      <h4>count: {visible ? count : 'invisible'}</h4>
      <button onClick={incrementAction}>Increment</button>&nbsp;
      <button onClick={resetAction}>Reset</button>&nbsp;
      <button onClick={toggleVisibleAction}>{visible ? 'hide' : 'show'}</button>
    </div>
  )
}

const mapStateToProps = (state) => {/* ... */}

const mapDispatchToProps = (dispatch) => {/* ... */}

export default connect(mapStateToProps, mapDispatchToProps)(Counter)
```

最后一样将组件插入到主页面就能够看到效果啦

- `App.jsx`

```js
function App() {
  return (
    <div>
      {/* ... */}
      <hr />
      <h1>React Redux</h1>
      <Counter></Counter>
    </div>
  )
}

export default App
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_create_manual_redux1.png)

初始页面，多次点击 `increment` 来递增计数器(发起 `dispatch(incrementAction())`)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_create_manual_redux2.png)

再点击 `reset` 按钮(发起 `dispatch(resetAction())`)之后来重置计数器

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_create_manual_redux1.png)

最后点击 `visible/hide` 按钮(发起 `dispatch(toggleVisibleAction())`)可以切换计数器的展示/隐藏

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_create_manual_redux3.png)

# 结语

本篇作为 React 的入门款，其实只用到了 React 的皮毛，不过作为项目和工程基础，了解一个 React 项目搭建的全貌和每个依赖发挥的作用和开箱方法也是非常重要的！后面还有更多 React 的高阶使用技巧需要作者去学习再来跟大家分享～

# 其他资源

## 参考连接

<table>
  <tr>
    <td>扩展create-react-app的webpack配置</td>
    <td><a href="https://blog.csdn.net/qq_37860930/article/details/85162024">https://blog.csdn.net/qq_37860930/article/details/85162024</a></td>
  </tr>
  <tr>
    <td>从零开始手动搭建react项目小记，告别脚手架</td>
    <td><a href="https://blog.csdn.net/aMeiHui/article/details/109118135">https://blog.csdn.net/aMeiHui/article/details/109118135</a></td>
  </tr>
  <tr>
    <td>教你如何一步一步手动创建react项目</td>
    <td><a href="https://blog.csdn.net/qq_41672008/article/details/105899383">https://blog.csdn.net/qq_41672008/article/details/105899383</a></td>
  </tr>
  <tr>
    <td>解决webpack5下使用webpack-dev-server报错</td>
    <td><a href="https://blog.csdn.net/peter_hzq/article/details/109683913">https://blog.csdn.net/peter_hzq/article/details/109683913</a></td>
  </tr>
  <tr>
    <td>【译】关于Webpack中一些让人困惑的地方的解答</td>
    <td><a href="https://blog.csdn.net/weixin_33921089/article/details/89353692">https://blog.csdn.net/weixin_33921089/article/details/89353692</a></td>
  </tr>
  <tr>
    <td>webpack的react的jsx后缀文件获取</td>
    <td><a href="https://blog.csdn.net/qq_43505774/article/details/106134070">https://blog.csdn.net/qq_43505774/article/details/106134070</a></td>
  </tr>
  <tr>
    <td>Webpack：clean-webpack-plugin 清理資源</td>
    <td><a href="https://blog.csdn.net/weixin_44691608/article/details/106753972">https://blog.csdn.net/weixin_44691608/article/details/106753972</a></td>
  </tr>
  <tr>
    <td>3. webpack 自定义模板 html-webpack-plugin插件</td>
    <td><a href="https://blog.csdn.net/hbiao68/article/details/104054932/">https://blog.csdn.net/hbiao68/article/details/104054932/</a></td>
  </tr>
  <tr>
    <td>webpack-dev-server的配置和使用——热更新、热替换、proxy代理</td>
    <td><a href="https://blog.csdn.net/ganle/article/details/106455612">https://blog.csdn.net/ganle/article/details/106455612</a></td>
  </tr>
  <tr>
    <td>connect-history-api-fallback库的理解</td>
    <td><a href="https://blog.csdn.net/astonishqft/article/details/82762354">https://blog.csdn.net/astonishqft/article/details/82762354</a></td>
  </tr>
  <tr>
    <td>React Router 官方文档</td>
    <td><a href="https://reactrouter.com/web/guides/quick-start">https://reactrouter.com/web/guides/quick-start</a></td>
  </tr>
  <tr>
    <td>reactjs/react-router-tutorial-github</td>
    <td><a href="https://github.com/reactjs/react-router-tutorial/tree/master/lessons">https://github.com/reactjs/react-router-tutorial/tree/master/lessons</a></td>
  </tr>
  <tr>
    <td>react进阶（二）——react-router基本使用和匹配规则</td>
    <td><a href="https://blog.csdn.net/qq_40566547/article/details/92384457">https://blog.csdn.net/qq_40566547/article/details/92384457</a></td>
  </tr>
  <tr>
    <td>react-router 中 exact 精确匹配使用</td>
    <td><a href="https://www.cnblogs.com/PasserByOne/p/13378788.html">https://www.cnblogs.com/PasserByOne/p/13378788.html</a></td>
  </tr>
  <tr>
    <td>React-Router browserHistory浏览器刷新出现页面404解决方案</td>
    <td><a href="https://blog.csdn.net/hsany330/article/details/90670186">https://blog.csdn.net/hsany330/article/details/90670186</a></td>
  </tr>
  <tr>
    <td>Redux 入门教程（一）：基本用法-阮一峰</td>
    <td><a href="https://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_one_basic_usages.html">https://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_one_basic_usages.html</a></td>
  </tr>
  <tr>
    <td>Redux 入门教程（三）：React-Redux 的用法-阮一峰</td>
    <td><a href="https://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_three_react-redux.html">https://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_three_react-redux.html</a></td>
  </tr>
  <tr>
    <td>react-redux实践</td>
    <td><a href="https://www.jianshu.com/p/b872ec0f3f5c">https://www.jianshu.com/p/b872ec0f3f5c</a></td>
  </tr>
</table>

## 完整代码实现

<a href="https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_create_manual">https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_create_manual</a>


