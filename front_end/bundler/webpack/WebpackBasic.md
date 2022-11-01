# Webpack：入門

@[TOC](文章目錄)

<!-- TOC -->

- [Webpack：入門](#webpack入門)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Overview 概述](#overview-概述)
  - [Configuration 配置](#configuration-配置)
  - [起步](#起步)
    - [Install 安裝](#install-安裝)
    - [Entry 入口](#entry-入口)
    - [Output 輸出文件](#output-輸出文件)
    - [Loader 模塊加載器](#loader-模塊加載器)
    - [Plugin 插件](#plugin-插件)
    - [Mode 模式](#mode-模式)
  - [完整項目](#完整項目)
    - [Directory 項目目錄結構](#directory-項目目錄結構)
    - [各文件內容（需要手動創建的部分）](#各文件內容需要手動創建的部分)
- [結語](#結語)

<!-- /TOC -->

## 簡介

現代的前端項目已經不再是簡單的 HTML+CSS+JavaScript 了，可能會使用一些響應式框架(Vue、React)，使用了 CSS 預處理(SCSS、LESS)，使用許多第三方庫。如果使用舊版的技術可能需要引入大量的 `<scirpt>` 標標籤，甚至將會存在許多根本沒用到的代碼。到了 node、npm 出現的時代，透過 `node_modules` 管理第三方庫的依賴，也能夠使用 `import/export` 等技術管理模塊化，但是還存在許多外部資源(字體、圖片)也無法直接引入 JS 解析，就會形成大量引用散落在 html 文件中。並且如果在運行時因為原生的靜態引入方法造成客戶端網絡的瓶頸而產生加載延遲。

這時候我們就需要透過`打包(bundling)`的手段，將一些靜態關聯的模塊一次打包成一個完整的 JS 文件，這樣客戶端就能夠一次引入必要的部份，剩下的再透過動態加載的方式引入

在 Webpack 之前也出現過一些類似的打包工具(Grunt、Gulp、Brunch)那時稱作任務管理器(Task Runner)，但是在打包的過程可能會出現"牽一髮而動全身"的問題，僅僅修改了一行代碼就必須導致整個項目重新打包。

Webpack 解決了這些問題，透過建立`依賴圖(Dependency Graph)`，不僅能夠實現按需加載（避免不必要的代碼引用），還能夠根據依賴圖實現部分重新打包，大幅優化打包的速度。並且開放 `Loader`、`Plugin` 提供了擴展的可能性，只要編寫相應的加載器(Loader)或是插件(Plugin)，能夠實現任何資源的加載。

## 參考

<table>
  <tr>
    <td>webpack官方</td>
    <td><a href="https://webpack.docschina.org/">https://webpack.docschina.org/</a></td>
  </tr>
  <tr>
    <td>Getting-Started</td>
    <td><a href="https://webpack.docschina.org/guides/getting-started/">https://webpack.docschina.org/guides/getting-started/</a></td>
  </tr>
</table>

# 正文

本篇將會簡單介紹 Webpack 的配置結構，並且透過參考官網的起步自製的一個流程，邊做邊說明。

## Overview 概述

Webpack 的核心思想就是將一個項目視作許多`模塊(Module)`之間的連結，透過指定`入口(Entry)`和`出口(output)`，並借助`模塊加載器(Loader)`、`插件(Plugin)`來對不同的模塊類型做不同的預處理，最終建立`依賴圖(Dependency Graph)`。並且也能夠透過依據目的來區分不同的打包`模式(mode)`。

這邊幾乎點出了 Webpack 配置文件裡面的所有配置項：

1. Entry 入口：依賴圖的起點，通常作為源文件代碼中的項目入口模塊
2. Output 輸出：打包後輸出的文件
3. Loader 加載器：負責對其他模塊類型進行預處理，將其他模塊透過一些`步驟`處理為 JS 的語法
4. Plugin 插件：可以說是更強大且更完整的 Loader，詳細差別下面將會提到
5. Mode 模式：透過指定不同的打包模式，可控制打包的行為（進行打包優化、壓縮代碼等）

## Configuration 配置

Webpack 配置最重要的就是要有一個`配置文件`(Loader 有三種配置，不過依舊推薦都寫在配置文件內)，配置文件的結構如下：

```js
// webpack.config.js
module.exports = {
  // 配置入口
  entry: Object | String,
  // 配置輸出文件
  output: {
    filename: String,
    path: String
  },
  // 配置模塊加載器 Loader
  module: {
    rules: [
      {
        test: RegExp, // 模塊名匹配規則
        use: Array<String> | String // 指定加載器
      }
    ]
  },
  // 配置插件(JS 對象)
  plugins: Array<Plugin>,
  // 配置打包模式
  mode: String
}
```

接下來我們就來邊做邊介紹，這邊的簡易項目參考官方的 <a href="https://webpack.docschina.org/guides/getting-started/">Getting Start</a> 流程

## 起步

首先我們先建立一個 npm 項目：

```bash
$ mkdir webpack-demo
$ cd webpack-demo
$ npm init -y
```

就會產生一個 `package.json` 文件：

- package.json

```json
{
  "name": "webpack-demo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

### Install 安裝

首先先安裝 Webpack 的核心模塊(`webpack`)以及 CLI 命令行工具(`webpack-cli`)

```bash
$ npm i -D webpack webpack-cli
```

這時候的 `package.json` 將會多出開發項配置，並且我們將 `main` 改成 `private`，詳情查看 npm 官方文檔：

- package.json

```json
{
  "name": "webpack-demo",
  "version": "1.0.0",
  "description": "",
- "main": "index.js",
+ "private": true,
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
+ "devDependencies": {
+   "webpack": "^4.43.0",
+   "webpack-cli": "^3.3.11"
+ }
}
```

並且建立最基礎的項目架構：

```
  webpack-demo
  |- package.json
+ |- /dist
+   |- index.html
+ |- /src
+   |- index.js
```

並再簡單佈置一下 `index.html` 跟 `index.js`：

- index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Webpack-demo</title>
  </head>
  <body>
    <div id="root"></div>

    <script src="main.js"></script>
  </body>
</html>
```

- index.js

```js
function component() {
  const div = document.createElement('div')

  div.innerHTML = ['Hello', 'Webpack'].join(' ')

  return div
}

const root = document.getElementById('root')
root.appendChild(component())
```

這時候已經可以使用默認配置進行打包了，先用 npm 配置 webpack 指令：

- package.json

```json
"scripts": {
  "build": "webpack"
}
```

接著執行 `npm run build` 就能夠看到默認配置的打包結果了

### Entry 入口

接下來我們將透過配置文件來配置，首先來配置入口。

先建立 `webpack.config.js` 文件：

```
  webpack-demo
  |- package.json
  |- node_modules
+ |- webpack.config.js
  |- /dist
    |- index.html
    |- main.js
  |- /src
    |- index.js
```

然後加上入口(Entry)配置：

- webpack.config.js

```js
const path = require('path')

module.exports = {
  entry: './src/index.js'
}
```

只有一個字符串的時候表示單入口，通常單頁面應用(SPA:Single Page Application)使用單入口就足以應付；而多頁應用就需要配置多個入口，如下配置：

```js
module.exports = {
  entry: {
    // 入口名：入口文件路徑
    app: './src/index.js',
    print: './src/print.js'
  }
}
```

### Output 輸出文件

有了入口還不夠，我們還需要配置輸出文件，告訴 Webpack 我們要輸出到哪，並指定輸出文件的名稱：

- webpack.config.js

```js
const path = require('path')

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js', // 默認配置為 main.js
    path: path.resolve(__dirname, 'dist')
  }
}
```

與此同時我們需要修改 `index.html` 的 `<script>` 標籤，以及 `npm script` 裡面的指令：

- index.html

```html
-
<script src="main.js"></script>
+
<script src="bundle.js"></script>
<!-- 這邊引入的 js 文件對應 Output(輸出) 的 filename -->
```

- package.json

```json
{
  // ...
  "scripts": {
    "build": "webpack --config webpack.config.js"
  }
}
```

這時候再執行 `npm run build` 就能夠看到配置入口出口文件之後的打包情形了（可以刪掉上次打包遺留的 `main.js`）：

```
  webpack-demo
  |- package.json
  |- node_modules
  |- webpack.config.js
  |- /dist
    |- index.html
+   |- bundle.js
  |- /src
    |- index.js
```

如此一來 webpack 就以 `index.js` 作為入口將相關靜態引入的依賴打包成 `bundle.js` 放置到 `dist` 目錄之下。現在在瀏覽器開啟 `/dist/index.html` 文件就能夠看到當前的網頁樣式了。

### Loader 模塊加載器

由於 webpack 默認只能打包 `.js`、`.json` 類型的文件(模塊)，如果想要加載其他類型的模塊就需要透過對應的`加載器(Loader)`。加載器相當於對其他類型的文件進行預處理，最後轉變成 JS 文件可讀的形式(url、import 啥的)

這邊我們以加載 `.css` 做範例，對應的加載器為 `style-loader` 和 `css-loader`

Loader 存在三種使用方式：配置方式（寫在配置文件內）、內聯方式（寫在 JS 文件內）、CLI（作為命令行參數），推薦使用第一種，後面兩種就不解釋了，自己查官網

首先我們先安裝相關的加載器：

```bash
$ npm i -D style-loader css-loader
```

再加入要引入的 `.css` 文件：

```
  webpack-demo
  |- package.json
  |- node_modules
  |- webpack.config.js
  |- /dist
    |- index.html
    |- bundle.js
  |- /src
    |- index.js
+   |- index.css
```

並且填充一下內容

- index.css

```css
* {
  color: red;
}
```

然後在 `index.js` 中引入依賴：

- index.js

```js
+ import './index.css'

  function component() {
    const div = document.createElement('div')

    div.innerHTML = ['Hello', 'Webpack'].join(' ')

    return div
  }

  const root = document.getElementById('root')
  root.appendChild(component())
```

最後為配置文件添加 Loader 相關的配置：

- webpack.config.js

```js
module.exports = {
  // ...
  module: {
    rules: [
      {
        // test 表示文件名的匹配模式
        test: /\.css$/,
        // use 表示使用的 loader
        use: ['style-loader', 'css-loader']
      }
    ]
  }
}
```

注意！這邊的 loader 支持戀是調用

這時候再運行 `npm run build` 就能看到打包後的情形了：

```
Hash: 2d96f286fe38387953e0
Version: webpack 4.43.0
Time: 596ms
Built at: 2020-06-10 16:12:56
    Asset      Size  Chunks             Chunk Names
bundle.js  4.74 KiB       0  [emitted]  main
Entrypoint main = bundle.js
[0] ./src/index.js 230 bytes {0} [built]
[1] ./src/index.css 519 bytes {0} [built]
[3] ./node_modules/css-loader/dist/cjs.js!./src/index.css 253 bytes {0} [built]
    + 2 hidden modules
```

### Plugin 插件

除了 Loader 之外，還有另一個更強大的功能是引入`插件(plugins)`。插件本身將作為一個具有 `apply` 方法的對象，在整個編譯週期都能訪問，傳入一個類似的配置對象，實際上將作為 `compile` 參數調用插件的 `apply` 方法，我們現在向我們的項目加入 `html-webpack-plugin` 插件：

- 安裝

```bash
$ npm i -D html-webpack-plugin
```

然後修改目錄結構，創建 `public` 目錄並創建 `index.html` 作為網頁入口，並且先暫時將 `dist` 目錄下的文件清空：

```
  webpack-demo
  |- package.json
  |- node_modules
  |- webpack.config.js
  |- /dist
-   |- index.html
-   |- bundle.js
+ |- /public
+   |- index.html
  |- /src
    |- index.js
    |- index.css
```

- index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>webpack-demo</title>
  </head>
  <body></body>
</html>
```

並且在配置文件中添加 `html-webpack-plugin` 的配置，需要往 `plugins` 插入一個插件對象，引用方式如下：

- webpack.config.js

```js
const HtmlWebpackPlugin = require('html-webpack-plugin')
const path = require('path')

module.exports = {
  // ...
  plugins: [new HtmlWebpackPlugin({ template: './public/index.html' })]
}
```

最後執行 `npm run build` 打包項目，就能夠看到 `dist` 目錄下出現的 `index.html` 啦！內容如下：

- index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width,initial-scale=1" />
    <title>webpack-demo</title>
  </head>
  <body>
    <script src="bundle.js"></script>
  </body>
</html>
```

咦？你有發現嗎，除了原先 `/public/index.html` 的內容之外，webpack 還自動將打包後的輸出文件 `bundle.js` 引入到 html 中，這也是 vue-cli 或 react-react-app 創建的項目中，打包項目的時候的 html 配置來來源喔（這邊的 html 位置也是模仿腳手腳手架生成的目錄結構，其實直接放在 `src` 目錄之下也並無不可）

### Mode 模式

最後的最後稍微介紹一下，webpack 配置文件中還有極為重要的一項，就是配置`打包模式(Mode)`，可選值如下：

| 模式        | 描述                         |
| ----------- | ---------------------------- |
| development | 開發模式，開發時打包用       |
| production  | 生產模式，需要部署上線時使用 |
| none        | 不配置模式                   |

配置模式最主要的核心就是控制 `process.env.NODE_ENV` 值的轉變，在 node 環境之下會影響到許多配置項和功能表現，配置方法如下：

- webpack.config.js

```js
const path = require('path')

module.exports = {
  mode: 'development'
  // ...
}
```

基於 webpack 的打包模式，因此就有人提出將不同模式的不同配置分成多個配置文件的形式，並且放如統一的 `config` 目錄下，形成如下結構：

```
  webpack-demo
  |- config
    |- webpack.base.config.js
    |- webpack.dev.config.js
    |- webpack.prod.config.js
```

- `webpack.base.config.js`：表示基礎配置，將由 `dev` 和 `prod` 繼承
- `webpack.dev.config.js`：繼承 `base`，作為`開發模式`下的配置使用
- `webpack.prod.config.js`：繼承 `base`，作為`生產模式`下的配置使用

三者的簡要結構如下，這邊需要用到一個 `webpack-merge` 的插件，本篇不使用也將不做介紹，未來會在 webpack 專欄的其他文章中單獨介紹：

- webpack.base.config.js

```js
module.exports = {
  // ...
}
```

- webpack.dev.config.js

```js
const merge = reuqire('webpack-merge')
const config = require('./webpack.base.config')

module.exports = merge(config, {
  mode: 'development'
  //
})
```

- webpack.prod.config.js

```js
const merge = reuqire('webpack-merge')
const config = require('./webpack.base.config')

module.exports = merge(config, {
  mode: 'production'
  //
})
```

這樣就可以完成配置文件的分離和基礎配置的復用

## 完整項目

以上流程創建了一個使用 `webpack.config.js` 做配置文件，引用了 `style-loader`、`css-loader` 加載 CSS 文件，使用 `html-webpack-plugin` 配置 html 入口文件，作為一個小型測試 webpack 功能的項目配置和使用方法呈現給大家。以下是項目目錄結構以及各文件內容：

### Directory 項目目錄結構

```
webpack-demo
|- package.json
|- node_modules
|- webpack.config.js
|- /dist
  |- index.html
  |- bundle.js
|- /public
  |- index.html
|- /src
  |- index.js
  |- index.css
```

- `package.json`：由 `npm init -y` 命令自動生成
- `node_modules`：作為 npm 包安裝的目錄
- `dist`：作為編譯打包之後的目標目錄
- `webpack.config.js`、`/src/index.js`、`/src/index.css`、`/public/index.html`：需要手動創建並完成內容

### 各文件內容（需要手動創建的部分）

- package.json

```json
{
  "name": "webpack-demo",
  "version": "1.0.0",
  "description": "",
  "private": true,
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack --config webpack.config.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "css-loader": "^3.5.3",
    "html-webpack-plugin": "^4.3.0",
    "style-loader": "^1.2.1",
    "ts-loader": "^7.0.5",
    "webpack": "^4.43.0",
    "webpack-cli": "^3.3.11"
  }
}

```

- /public/index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>webpack-demo</title>
  </head>
  <body></body>
</html>
```

- /src/index.js

```js
import './index.css'

function component() {
  const div = document.createElement('div')

  div.innerHTML = ['Hello', 'Webpack'].join(' ')

  return div
}

const root = document.getElementById('root')
root.appendChild(component())
```

- /src/index.css

```css
* {
  color: red;
}
```

- webpack.config.js

```js
const HtmlWebpackPlugin = require('html-webpack-plugin')
const path = require('path')

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      }
    ]
  },
  plugins: [new HtmlWebpackPlugin({ template: './public/index.html' })]
}
```

鏘鏘！這樣就完成最基礎的 webpack 配置與使用啦

# 結語

本篇簡單介紹了 webpack 的主要配置屬性和使用的範例，作者為了縮小篇幅真的簡化了很多東西，希望能夠幫助剛開始接觸 webpack 的小夥伴們一點一點的熟悉各種 Loader 以及 Plugin 的用途和用法。之後會介紹加上 `babel-loader` 來引入 babel 成為編譯的一環，以及透過 `webpack-dev-server` 來創建開發用的簡單網頁服務器等功能。

並且近年來前端工程化的議題也正在迅速發酵，webpack 作為三大框架之中 Vue、React 所使用的打包工具，或許在不遠的將來，webpack 的原理已經優化或許也將成為前端面試的重要課題之一。
