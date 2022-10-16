# React SSR: 基于 express 自构建 SSR 服务端渲染

@[TOC](文章目录)

<!-- TOC -->

- [React SSR: 基于 express 自构建 SSR 服务端渲染](#react-ssr-基于-express-自构建-ssr-服务端渲染)
- [完整代码示例](#完整代码示例)
- [前情提要](#前情提要)
- [构建 CSR 项目](#构建-csr-项目)
  - [项目初始化 & 安装依赖](#项目初始化--安装依赖)
  - [编写配置](#编写配置)
  - [内容填充](#内容填充)
- [构建 express 服务端](#构建-express-服务端)
  - [安装依赖](#安装依赖)
  - [基础服务端构建](#基础服务端构建)
- [CSR 与 SSR 并行](#csr-与-ssr-并行)
  - [CSR 产物](#csr-产物)
  - [SSR v1：全量替换](#ssr-v1全量替换)
  - [SSR v2：hydrate 注入](#ssr-v2hydrate-注入)
- [遗留问题](#遗留问题)
- [参考连接](#参考连接)

<!-- /TOC -->

# 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_ssr_simple](https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_ssr_simple)

# 前情提要

在本篇最开头，我们先明确本项目的目标，既不是创建一个 React SSR 的框架，也不是企图完成一个从 CSR 迁移到 SSR 的项目构建手册或是模版。

本篇项目的宗旨在于使用 React 提供的基础 API 走通一个最基础版本的 SSR 渲染流程，并借此引出 SSR 渲染的实际操作流程，以及后续关于 SSR 渲染需要注意的问题与思考。

Let's go~

# 构建 CSR 项目

首先相信会点进来看 SSR 的对于 React 基础应该都不陌生了。本篇第一节需要先构建一个基础版的 React 项目，最原始版本的就是基于 CSR 的简单 SPA 项目。

## 项目初始化 & 安装依赖

作者习惯使用 pnpm 进行项目构建，你也可以选择你习惯的管理器

```bash
pnpm init
pnpm i react react-dom # react 相关
pnpm i -D webpack webpack-cli webpack-dev-server # webpack 相关
pnpm i -D @babel/core @babel/preset-env @babel/preset-react @babel/preset-typescript babel-loader # babel 相关
pnpm i -D style-loader css-loader sass-loader sass # 样式相关 loader
pnpm i -D webpackbar html-webpack-plugin clean-webpack-plugin mini-css-extract-plugin # webpack 插件
```

每个 loader 或是 plugin 的作用就不展开说明

## 编写配置

安装好依赖之后先写点配置

- `.babelrc`

```json
{
  "presets": [
    "@babel/preset-env",
    "@babel/preset-react",
    "@babel/preset-typescript"
  ],
  "plugins": []
}
```

- `webpack.config.js`

```js
const path = require('path');

const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const WebpackBar = require('webpackbar');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

const IS_PROD = process.env.NODE_ENV;

const config = {
  // Start mode / environment
  mode: IS_PROD ? 'production' : 'development',

  // Entry files
  entry: path.resolve(__dirname, 'src/index'),

  // Output files and chunks
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name]-[chunkhash:8].js',
  },

  // Resolve files configuration
  resolve: {
    extensions: ['.js', '.jsx', '.ts', '.tsx', '.json', '.scss'],
  },

  // Module/Loaders configuration
  module: {
    rules: [
      {
        test: /\.(js|jsx|ts|tsx)$/,
        exclude: /node_modules/,
        use: 'babel-loader',
      },
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader'],
      },
      {
        test: /\.module.(sass|scss)$/,
        use: [
          MiniCssExtractPlugin.loader,
          {
            loader: 'css-loader',
            options: {
              modules: {
                localIdentName: '[path][name]__[local]--[hash:base64:5]',
              },
              sourceMap: true,
            },
          },
          'sass-loader',
        ],
      },
    ],
  },

  // Plugins
  plugins: [
    new WebpackBar(),
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: path.resolve(__dirname, 'public/index.html'),
      minify: IS_PROD,
    }),
    new MiniCssExtractPlugin({
      filename: 'styles-[chunkhash:8].css',
    }),
  ],

  // Webpack chunks optimization
  optimization: {
    splitChunks: {
      cacheGroups: {
        default: false,
        venders: false,

        vendor: {
          chunks: 'all',
          name: 'vender',
          test: /node_modules/,
        },

        styles: {
          name: 'styles',
          type: 'css/mini-extract',
          chunks: 'all',
          enforce: true,
        },
      },
    },
  },

  // DevServer for development
  devServer: {
    port: 3000,
    historyApiFallback: true,
  },

  // Generate source map
  devtool: 'source-map',
};

module.exports = config;
```

- `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "es5",
    "allowJs": true,
    "jsx": "react",
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true,
    "experimentalDecorators": true,
    "strictNullChecks": true,
    "noImplicitAny": true,
    "lib": ["DOM", "ESNext"]
  },
  "include": ["src"],
  "exclude": ["node_modules"]
}
```

- `package.json`

```json
{
  "name": "react_ssr_simple",
  // ...
  "scripts": {
    "start": "NODE_ENV=development webpack serve",
    "start:ssr": "nodemon server",
    "build": "NODE_ENV=production webpack"
  },
  "dependencies": {
    // ...
  },
  "devDependencies": {
    // ...
  }
}
```

上面也都是一些常见的配置，就不展开讨论了

## 内容填充

配置写好之后我们就简单填充一下网站内容即可

- `/src/App.tsx`

```ts
import React, { FC } from 'react';
import styles from './App.module.scss';
import Counter from './components/Counter';

const App: FC = () => {
  return (
    <div className={`${styles.container} my-app`}>
      <h1>App</h1>
      <Counter />
    </div>
  );
};

export default App;
```

- `/src/index.tsx`

```ts
import React from 'react';
import ReactDOM from 'react-dom';
import { createRoot } from 'react-dom/client';

import App from './App';

import './index.module.scss';

// ReactDOM.render(<App />, document.querySelector('#app'));
const root = createRoot(document.querySelector('#app') as HTMLElement);
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

- `/public/index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>React App</title>
    <meta name="description" content="React App create by @youxian/cli" />
  </head>
  <body>
    <div id="app"></div>
  </body>
</html>
```

值得一提的是这里我们引入的是 React 18，所以用了新的 `createRoot` API，可能有些人习惯用的是 `ReactDOM.render` 也没关系

以上构建确保能够使用 `pnpm start` 并能够在浏览器访问，同时 `pnpm build` 能正确构建产物即可

# 构建 express 服务端

完成基础 React 项目创建之后，我们需要来构建一个 Node.js 服务器，本篇使用 express(你喜欢的话使用 Node.js 原生 http 模块或是其他如 Kao 都没关系)

## 安装依赖

```bash
pnpm i -D nodemon express
pnpm i -D @babel/register ignore-styles
```

由于后续需要在服务端解析并使用 React 组件相关代码，我们使用 `@babel/register` 来进行代码转译

## 基础服务端构建

- `/server/index.js`

```js
const path = require('path');

// ignore `.scss` imports
require('ignore-styles');

// transpile imports on the fly
require('@babel/register')({
  configFile: path.resolve(__dirname, '../.babelrc'),
  extensions: ['.js', '.jsx', '.ts', '.tsx'],
});

// import express server
require('./express.js');
```

入口的部分如上述代码，我们先注册 `ignore-styles` 与 `@babel/register` 使我们的服务端能正确转译使用 TS 代码；核心逻辑则放在 `express.js` 文件下

- `/server/express.js`

```js
const fs = require('fs');
const path = require('path');

const express = require('express');
const React = require('react');
const ReactDOMServer = require('react-dom/server');

// create express application
const app = express();

// serve static assets
app.get(
  /\.(js|css|map|ico)$/,
  express.static(path.resolve(__dirname, '../dist'))
);

// for any other requests, send `index.html` as a response
app.use('^/$', (req, res) => {
  // read `index.html` file
  const indexHTML = fs.readFileSync(
    path.resolve(__dirname, '../dist/index.html'),
    { encoding: 'utf8' }
  );

  // set header and status
  res.contentType('text/html');
  res.status(200);

  return res.send(indexHTML);
});

// run express server on port 9000
app.listen(9000, () => {
  console.log('Express server started at http://localhost:9000');
});
```

最后先运行客户端打包，然后启动服务端，访问网页即可

```bash
pnpm build
pnpm start:ssr
open http://localhost:9000
```

在初始版本中，实际上我们可以看到仅仅是将打包后的产物作为静态页面返回而已，接下来才会真正进入 SSR 的环节。

# CSR 与 SSR 并行

## CSR 产物

在开始进行 SSR 改造之前，我们先来看一下 CSR 版本的产物

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_ssr_simple_1_csr.png)

我们可以看到服务端返回的 html 当中实际上是一个空的 `<div id="app"></div>`，真正的内容是在 `script` 加载完毕之后使用 `ReactDOM.render` 替换进来的。

CSR 的问题便是在于如果这个页面过于复杂，或是因为网络等问题出现加载缓慢的情况，则在 JS 执行完毕之前用户就只能看到白屏

## SSR v1：全量替换

接下来第一个版本的 SSR 的思想就是：在服务端预先渲染页面的 html，然后客户端 script 执行完毕之后再进行替换

- `/server/express.js`

下面我们对服务端代码稍微进行改造

```js
const ReactDOMServer = require('react-dom/server');

// ...

// for any other requests, send `index.html` as a response
app.use('^/$', (req, res) => {
  const { default: App } = require('../src/App.tsx');

  // read `index.html` file
  let indexHTML = fs.readFileSync(
    path.resolve(__dirname, '../dist/index.html'),
    { encoding: 'utf8' }
  );

  const html = ReactDOMServer.renderToString(<App />);

  indexHTML = indexHTML.replace(
    '<div id="app"></div>',
    `<div id="app">${html}</div>`
  );

  // set header and status
  res.contentType('text/html');
  res.status(200);

  return res.send(indexHTML);
});
```

我们利用 `ReactDOMServer.renderToString` 方法，传入我们的根组件 `<App />` 并获得最终 html 内容，替换到 `index.html` 之后回传客户端，就完成了所谓的 SSR 服务端渲染

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_ssr_simple_2_ssr_fullpage.png)

我们可以看到，这一次请求服务端的 html 里面已经有内容了。如此一来浏览器就能够在接收到 html 的同时开始渲染页面，并同时并行的请求 `script` 数据，然后在 script 数据加载完成之后进行替换，流程如下：

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_ssr_simple_3_ssr_fullpage_workflow.png)

上述的流程就是一个基础版本的 SSR，然而这样我们还不满意，因为最后一步的 `ReactDOM.render` 方法，实际上会在 script 脚本执行的过程中重新进行所有 dom 节点的删除、创建等所有计算，相当于整个页面重新渲染，就浪费了前面在服务端渲染好的部分

## SSR v2：hydrate 注入

React 的作者们肯定也是想到了这一块，所以提供了一个 API 叫做 `ReactDOM.hydrate`，来支持关于 SSR 渲染的部分

具体用法我们只需要添加以下代码

- `/src/index.prod.tsx`

```ts
import React from 'react';
import ReactDOM from 'react-dom';

import App from './App';

import './index.module.scss';

ReactDOM.hydrate(<App />, document.querySelector('#app'));
```

如上，我们只需要将最后的 `ReactDOM.render` 改成 `ReactDOM.hydrate` 就可以了

React.hydrate 方法会自动识别 `<div id="app">` 内部写入的在服务端完成创建的节点并进行复用，相当于是在不重新删除、生成新节点的状况下，只注入 React 页面运行时所需要的动态内容。

如果我们没有在服务端进行初始内容的 html 注入，那么 ReactDOM.hydrate 的效果就与 ReactDOM.render 是一样的，不受影响。

# 遗留问题

本篇使用非常初级的方式来构建 React SSR 服务端渲染的页面，跟最一开始提的一样，一开始就不是以构建完整项目的方向进行。

上述流程创建的 SSR 渲染还遗留了几个问题尚未解决：

1. 路由部分

    服务端渲染的路由与客户端渲染是完全不一样的。CSR 下的路由实际上完全由客户端拦截浏览器的历史记录操作，并使用 JS 进行组件的抽换来模拟路由的改变；SSR 下的路由则是在切换的同时需要重复进行上述先给 html、后注入 runtime 的模式，因此同样需要在服务端提供页面的部分做些手脚。

    有机会作者会再后续出一篇相关的博客进行说明。

2. CSS Module

    另一个问题在于 CSS Module 的使用。自己配过 CSS Module 的应该明白所谓的 module 实际上是打包的过程中为每一个 stylesheet 文件生成一组新的样式名映射，然后在组件内透过 JS 对象获得真实生成的名称。

    但是在服务端渲染的情况下，我们就需要再对服务端读取组件文件的过程进行处理，保证打包时与运行时 SSR 渲染出的 stylesheet 保持一致，否则后来使用 `renderToString` 方法生成的节点 className 就无法与客户端渲染的样式表对上，进而产生样式不套用的问题。

    目前作者参考过一些定制的 `sass-loader` 搭配 babel 的写法，但是还没有试出来，欢迎读者自行查阅相关资料后尝试~

# 参考连接

| Title                                                   | Link                                                                                                                                                                                         |
| ------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| A beginner’s guide to React Server-Side Rendering (SSR) | [https://medium.com/jspoint/a-beginners-guide-to-react-server-side-rendering-ssr-bf3853841d55](https://medium.com/jspoint/a-beginners-guide-to-react-server-side-rendering-ssr-bf3853841d55) |
| course-one/react-ssr - Github                           | [https://github.com/course-one/react-ssr](https://github.com/course-one/react-ssr)                                                                                                           |
| webpack-contrib/mini-css-extract-plugin - Github        | [https://github.com/webpack-contrib/mini-css-extract-plugin](https://github.com/webpack-contrib/mini-css-extract-plugin)                                                                     |
| Server Side Rendering with CSS Modules                  | [https://medium.com/@mattvagni/server-side-rendering-with-css-modules-6b02f1238eb1](https://medium.com/@mattvagni/server-side-rendering-with-css-modules-6b02f1238eb1)                       |
| babel-plugin-css-modules-transform - npm                | [https://www.npmjs.com/package/babel-plugin-css-modules-transform](https://www.npmjs.com/package/babel-plugin-css-modules-transform)                                                         |
