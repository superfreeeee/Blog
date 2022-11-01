# Webpack：clean-webpack-plugin 清理資源

@[TOC](文章目錄)

<!-- TOC -->

- [Webpack：clean-webpack-plugin 清理資源](#webpackclean-webpack-plugin-清理資源)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Install 安裝](#install-安裝)
  - [None 未使用插件](#none-未使用插件)
  - [Default 默認無配置](#default-默認無配置)
  - [Options 配置選項](#options-配置選項)
- [結語](#結語)

<!-- /TOC -->

## 簡介

之前寫過一篇<a href="https://blog.csdn.net/weixin_44691608/article/details/106675550">Webpack：入門</a>介紹過了基本的 Webpack 配置和使用。今天來介紹 `clean-webpack-plugin` 這個插件的作用。

`clean-webpack-plugin` 的作用從名字就能看出這個插件的精髓就是`清理`的。webpack 作為一個自動化打包構建工具，由於每次構建都會產生新的文件，有的會覆蓋舊的同名文件，有的則不會，甚至如果使用 `hash` 生成文件名就會每次都產生新的文件，而舊的並不會自動消失。所以就有人開發出了簡單的 `clean-webpack-plugin` 來自動清除舊版本的編譯後文件啦！

## 參考

<table>
  <tr>
    <td>clean-webpack-plugin--github</td>
    <td><a href="https://github.com/johnagan/clean-webpack-plugin">https://github.com/johnagan/clean-webpack-plugin</a></td>
  </tr>
  <tr>
    <td>webpack4.0 clean-webpack-plugin 插件跳坑指南</td>
    <td><a href="https://juejin.im/post/5d107a56e51d45773d46863c">https://juejin.im/post/5d107a56e51d45773d46863c</a></td>
  </tr>
</table>

# 正文

簡單來說，`clean-webpack-plugin` 就是每次打包前先清理一次資源，就這樣，沒錯你沒看錯就這樣。此外它還提供一些參數來指定打包時的行為，接下來我們將透過展示幾個範例來說明：

## Install 安裝

老掉牙了，用之前當然要些安裝一下包嘛（這邊默認你已經配置好基本款的 webpack 囉，不會的話可以看這裡<a href="https://blog.csdn.net/weixin_44691608/article/details/106675550">Webpack：入門</a>）

```bash
$ npm i -D clean-webpack-plugin
```

## None 未使用插件

我們先來看看沒有使用 `clean-webpack-plugin` 插件的時候進行多次打包的情形，以下是我們目前的配置，使用 `[hash]` 插入導出文件名，可以避免打包誤讀緩存：

- webpack.config.js

```js
const path = require('path')

module.exports = {
  mode: 'development',
  entry: './src/index.js',
  output: {
    filename: 'bundle.[hash].js',
    path: path.resolve(__dirname, 'dist')
  },
  plugins: []
}
```

- 打包前

```
webpack-demo
|- dist
  |- index.html
|- src
  |- index.js
  |- a.js
  |- b.js
  |- c.js
|- node_modules
|- webpack.config.js
```

- 一次打包

```
webpack-demo
|- dist
  |- 1.bundle.2da25527cf81f4a077d4.js
  |- 0.bundle.2da25527cf81f4a077d4.js
  |- bundle.2da25527cf81f4a077d4.js
  |- index.html
|- src
  |- index.js
  |- a.js
  |- b.js
  |- c.js
|- node_modules
|- webpack.config.js
```

然後稍微改動個一、兩行代碼

- 二次打包

```
webpack-demo
|- dist
  |- 1.bundle.2da25527cf81f4a077d4.js
  |- 1.bundle.7a681edeaf5d57498b89.js
  |- 0.bundle.2da25527cf81f4a077d4.js
  |- 0.bundle.7a681edeaf5d57498b89.js
  |- bundle.2da25527cf81f4a077d4.js
  |- bundle.7a681edeaf5d57498b89.js
  |- index.html
|- src
  |- index.js
  |- a.js
  |- b.js
  |- c.js
|- node_modules
|- webpack.config.js
```

Boom！文件開始爆炸性的增長，每次編譯都要手動刪除也太麻煩，接下來看看使用默認配置的 `clean-webpack-plugin` 插件

## Default 默認無配置

如果你沒什麼東西想要配置的，這個插件也具備開箱即用的功能，默認可以不接受任何配置（3.0.0 以後默認會清理 `output` 目錄下的所有文件；2.0.0 需要傳入指定目錄名）

- webpack.config.js

```js
// 3.0.0 使用 export { CleanWebpackPlugin } 導出
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
// 2.0.2 使用 export default CleanWebpackPlugin 導出
// const CleanWebpackPlugin = require('clean-webpack-plugin')

const { CleanWebpackPlugin } = require('clean-webpack-plugin')
const path = require('path')

module.exports = {
  // ...
  output: {
    filename: 'bundle.[hash].js',
    path: path.resolve(__dirname, 'dist')
  },
  plugins: [
    // ...
    new CleanWebpackPlugin()
    // 2.0.2 需要傳入目標目錄
    // new CleanWebpackPlugin(['dist'])
  ]
}
```

在默認無配置的狀態下，將會清除目標目錄(`dist`)下的所有文件。每次打包之後，目標文件夾 `dist` 下的文件都會是最新一次打包的結果。

## Options 配置選項

通常使用默認的配置就足夠了，這邊列出可配置的選項（寫出來的值為默認值）：

```js
module.exports = {
  // ...
  plugins: [
    // ...
    new CleanWebpackPlugin({
      // 模擬刪除，實際上沒刪，可用在使用 unsafe 配置的時候設為 true
      dry: false,

      // 將日誌打印到控制台上，就是告訴你刪了哪些東西哈
      verbose: false,

      // 自動刪除為使用的資源
      cleanStaleWebpackAssets: true,

      // 保護當前資源
      protectWebpackAssets: true,

      /* WARNING 以下為 unsafe 配置，可看作實驗中特性 */
      // 下面屬性就不介紹了，除非需要詳細配置不然真的是很少用到
      // 有興趣可以到參考一的 github 連結上查看
      cleanOnceBeforeBuildPatterns: ['**/*', '!static-files*'],
      cleanOnceBeforeBuildPatterns: [],
      cleanAfterEveryBuildPatterns: ['static*.*', '!static1.js'],
      dangerouslyAllowCleanPatternsOutsideProject: true
    })
  ]
}
```

# 結語

由於這個插件本身就是清理遺留資源這個功能最重要，所以其他配置選項就不深究。切記！不要為了技術而技術，技術永遠學不完，真的需要詳細配置時再查也不遲，所以本篇就介紹到這裡。後續如果寫到 webpack 打包優化的時候說不定有機會再提到 `clean-webpack-plugin` 的配置。
