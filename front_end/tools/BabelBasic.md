# Babel 入門：JavaScript 的下一代編譯器

@[TOC](文章目錄)

<!-- TOC -->

- [Babel 入門：JavaScript 的下一代編譯器](#babel-入門javascript-的下一代編譯器)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Concept 概念](#concept-概念)
  - [Core Module 核心模塊](#core-module-核心模塊)
    - [1. @babel/core](#1-babelcore)
    - [2. @babel/cli](#2-babelcli)
    - [3. @babel/preset-env](#3-babelpreset-env)
    - [4. @babel/polyfill（不推薦）](#4-babelpolyfill不推薦)
    - [5. @babel/preset-stage-x](#5-babelpreset-stage-x)
    - [6. babel-loader](#6-babel-loader)
    - [7. @babel/preset-react](#7-babelpreset-react)
    - [8. @babel/preset-flow](#8-babelpreset-flow)
    - [9. @babel/preset-typescript](#9-babelpreset-typescript)
    - [presets、plugins](#presetsplugins)
  - [CLI 命令行工具](#cli-命令行工具)
    - [Install 安裝](#install-安裝)
    - [Usage 使用方式](#usage-使用方式)
      - [Shell](#shell)
      - [npm script(推薦)](#npm-script推薦)
    - [Sample](#sample)
  - [Config 配置選項](#config-配置選項)
    - [Basic 基本語法](#basic-基本語法)
    - [Sample：@babel/preset-env](#samplebabelpreset-env)
      - [Install 安裝](#install-安裝-1)
      - [Options 配置項](#options-配置項)
      - [Sample](#sample-1)
    - [Sample：@babel/plugin-transform-for-of](#samplebabelplugin-transform-for-of)
      - [Install 安裝](#install-安裝-2)
      - [Options 配置項](#options-配置項-1)
      - [Sample](#sample-2)
- [結語](#結語)

<!-- /TOC -->

## 簡介

當今前端生態越來越複雜，大量的框架、寫法層出不窮，甚至作為 JS 語言指導手冊的 ECMAScript 標準也年年再更新。越來越多的特性、語法糖被加入到 JS 的大家族當中，大量的 ES 版本、瀏覽器版本支援限制，**兼容性**一直以來都是困擾著開發者許久的問題。

- 目標：兼容舊版本 & 編譯非標準語法糖

Babel 作為 JavaScript 的預編譯處理器，除了能夠將新語法、新的 API 向下兼容，自動編譯成需要的兼容性格式。甚至可以透過 babel 添加一些處於實驗階段甚至僅僅只是提案階段的語法，babel 就好像開啟了一扇語法糖的大門，可以容納包羅萬象幾乎任何語法，只需要找到對應的 babel 插件編譯，作為開發者的你也可以編寫自己的 babel 插件，創造自己的語法糖來簡化開發過程。

## 參考

<table>
  <tr>
    <td>babel 官方</td>
    <td><a href="https://www.babeljs.cn/">https://www.babeljs.cn/</a></td>
  </tr>
  <tr>
    <td>Babel 設定</td>
    <td><a href="https://ithelp.ithome.com.tw/articles/10200299">https://ithelp.ithome.com.tw/articles/10200299</a></td>
  </tr>
</table>

# 正文

## Concept 概念

Babel 本質上是以**插件(plugin)**的形式呈現，核心代碼(core)會先解析整個 js 文件，再根據**配置項(.babelrc)**編譯最終的代碼形式。同時可以引入一些組合好的包，官方稱為**預設(preset)**，就不需要將插件一個一個導入了

**插件(plugin)** 和 **預設(preset)** 就是配置項裡面最主要的兩個配置選項，下面將會介紹詳細用法

## Core Module 核心模塊

這邊介紹配置 babel 的時候會使用到的核心模塊，由於 babel 為 JS 代碼的預處理，所以在引入時是引入到開發環境中(`--save-dev`)

### 1. @babel/core

用了這麼久的 npm，一看就知道了吧，這是 babel 的核心模塊，負責整個 babel 編譯過程和插件使用的基礎。不過這邊要注意一點，真正編譯的模塊是透過插件的形式引入的，所以如果只安裝 `core` 而沒有配置插件的話，babel 只會原封不動的複製一遍代碼而已。

### 2. @babel/cli

babel 的命令行工具，通常現在的前端項目大多會與 webpack 集成使用，這也是推薦的使用方式，因此 cli 模塊顯得可有可無，基本上只有在測試語法的時候單獨使用 babel 才會派得上用場

### 3. @babel/preset-env

這是官方推薦的主要 preset，原本舊版本的 babel 使用的時候需要引入特定的 `babel-preset-20xx`，版本管理上頗為麻煩。因此官方推出了 `preset-env`，裡面所包含的插件由官方負責管理，隨時代加入最新版本的 ECMAScript 插件，省去了很多配置管理上的困擾。

### 4. @babel/polyfill（不推薦）

有時候會遇到一些新版 API，是舊版本代碼無法轉換的，這時候需要引入 `polyfill` 來加入那些新 API 的舊版本替代方法。

**注意！**：現在 `polyfill` 已經**不推薦**使用了，新版 API 的替代方法注入已經可以透過 `@babel/preset-env` 的 `useBuiltIns` 配置項以及直接安裝 `core-js` 來取代

### 5. @babel/preset-stage-x

除了 ES 標準特性之外，ES 還有許多處於不同階段的語法特性，這邊可以透過引入 `stage-x` 來使用這些特性，並交給 babel 負責編譯。以下是各階段的說明：

- `stage-0`：Strawman (展示階段)
- `stage-1`：Proposal (徵求意見階段)
- `stage-2`：Draft (草案階段)
- `stage-3`：Candidate (候選階段)
- `stage-4`：Finished (定案階段)

階段前面的所包含的語法集合越廣，如 `stage-0` 將會支持 `stage-2`、`stage-3` 的所有語法，所以並不需要重複引入

### 6. babel-loader

現在主流框架如 Vue、React 等都會使用 webpack 作為打包工具，而 `babel-loader` 的作用就是將 babel 編譯過程加入到 webpack 打包過程當中。

### 7. @babel/preset-react

編譯 React 項目的時候需要引入的預設，裡面所添加的插件和配置選項可到官方查看

### 8. @babel/preset-flow

babel 對 flow 靜態檢查的編譯支持

### 9. @babel/preset-typescript

babel 對 typescript 的編譯支持

### presets、plugins

官方提供或是較為熱門的預設和插件可以在官方文檔的 <a href="https://babeljs.io/docs/en/presets">General \> presets</a>、<a href="https://babeljs.io/docs/en/plugins">General \> plugins</a>找到相關目錄

## CLI 命令行工具

這邊僅僅簡單介紹一下 babel-cli 的一些常用選項，正常項目主要都會透過 webpack 打包，所以這部分主要是自己測試或學習的時候才有必要。

### Install 安裝

以下是使用 CLI 工具時的安裝命令，同時安裝 babel 的核心模塊，上面提到的其他模塊也是一樣的安裝方法，依樣畫葫蘆就是了

```bash
$ npm i -D @babel/cli @babel/core
```

此時的 package.json 會做如下更改：

```json
{
  "devDependencies": {
+   "@babel/cli": "^7.0.0",
+   "@babel/core": "^7.0.0"
  }
}
```

### Usage 使用方式

#### Shell

安裝完 CLI 工具之後，可以直接透過命令行執行

```bash
$ npx src -d lib
# 等價於
$ ./node_modules/.bin/babel src -d lib
```

- 說明：由於 babel 命令被安裝到 `./node_modules/.bin/` 目錄之下，npm 版本 5.2 以上的可以使用 `npx <cmd>` 來調用 bin 裡面的指令，更加方便和優雅

#### npm script(推薦)

除了直接在命令行調用 babel 指令之外，還可以寫在 package.json 的 script 裡面，再透過 `npm run xxx` 來使用，如下：

```json
// package.json
{
  "script": {
    "build": "babel src -d lib"
  }
}
```

```bash
$ npm run build  # 執行 npx babel src -d lib
```

### Sample

這邊只介紹簡單的 babel-cli 用法，畢竟與 webpack 結合之後這些指令其實不太重要

```bash
# 編譯後輸出到標準輸出流(stdout)
$ npx babel index.js

# 編譯成目標文件，使用 --out-file（縮寫 -o）選項
$ npx babel src/index.js --out-file lib/index.js
$ npx babel src/index.js -o lib/index.js

# 實現同步編譯（熱編譯），使用 --watch（縮寫 -w）選項
$ npx babel src/index.js --watch -o lib/index.js
$ npx babel src/index.js -w -o lib/index.js

# 保存資源依賴圖（Source map），使用 --source-maps 選項
$ npx babel src/index.js -o lib/index.js --source-maps
# 或是作為註解放到目標文件內（inline 選項）
$ npx babel src/index.js -o lib/index.js --source-maps inline

# 編譯整個目錄，使用 --out-dir（縮寫 -d）選項
$ npx babel src --out-dir lib
$ npx babel src -d lib

# 忽略特定文件，使用 --ignore 選項
$ npx babel src -d lib --ignore "src/**/*.spec.js","src/**/*.test.js"

# blablabla
```

其他還有如`複製文件(--copy-files、--no-copy-ignoree)`、`管道流(piping <)`、`指定插件(--plugins)`、`指定預設(--presets)`、`自定義配置文件路徑(--config-file)`等就不一一示範了，接下來的配置項內容才是重點

## Config 配置選項

上面簡單介紹了一些 CLI 的命令行選項，但是對於完整的項目來說應該要寫成一個配置文件比較恰當，這邊則介紹配置項裡面的各個參數設置。由於不同預設(preset)和插件(plugin)的配置選項不同，我們只拿幾個來作為示例

### Basic 基本語法

babel 配置文件寫在 `.babelrc` 並放在根目錄裡面，是一個 json 類型的文件（也可以使用 `--config-file` 來指定配置文件位置），語法如下：

```json
// .babelrc
{
  "presets": [
    <preset-name> | [<preset-name>] | [<preset-name>, <preset-option>],
    ...
  ],
  "plugins": [
    <plugin-name> | [<plugin-name>] | [<plugin-name>, <plugin-option>],
    ...
  ]
}
```

- `<preset-name>`、`<plugin-name>`：預設或插件的名字(需要安裝在 node_module 裡面)，或是插件位置的路徑
- `<preset-option>`、`<plugin-option>`：預設或插件的配置選項

如果只有簡單的配置也可以直接寫在 `package.json` 裡面，以 `babel` 為鍵名：

```json
// package.json
{
  "name": "babel-test",
  "version": "1.0.0",
  "devDependencies": {
    // ...
  },
  "babel": {
    "presets": [],
    "plugins": []
  }
}
```

### Sample：@babel/preset-env

用於支持 ES 各版本的編譯，<a href="https://babeljs.io/docs/en/babel-preset-env">官方配置說明</a>

#### Install 安裝

```bash
$ npm i -D @babel/preset-env
```

#### Options 配置項

這邊只介紹幾個比較常見的，剩餘參數細節可自行查詢官方說明

| 配置項      | 含義                                                                                                                                              |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| targets     | 編譯的目標瀏覽器版本<br/>語法：`String | Array<String> | { [String]: String }`<br/>默認值：`{}`                                                   |
| debug       | 使用 console.log 輸出編譯信息，默認 `false`                                                                                                       |
| modules     | 指定模塊化方案<br/>可選值：`"amd" | "umd" | "systemjs" | "commonjs" | "cjs" | "auto" | false`<br/>默認值：`auto`<br/>`false` 表示不修改模塊化方式 |
| useBuiltIns | API 補丁（原來 `polyfill` 的作用）<br/>可選值：`"usage" | "entry" | false`<br/>默認值：`false`                                                    |

#### Sample

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "targets": {
          "chrome": "58",
          "ie": "11"
        },
        "debug": true,
        "modules": "cjs",
        "useBuiltIns": "usage"
      }
    ]
  ],
  "plugins": []
}
```

### Sample：@babel/plugin-transform-for-of

用於支持 ES6 的 `for-of` 語法，<a href="https://babeljs.io/docs/en/babel-plugin-transform-for-of">官方配置說明</a>

#### Install 安裝

```bash
$ npm i -D @babel/plugin-transform-for-of
```

#### Options 配置項

| 配置項         | 含義                                                                                   |
| -------------- | -------------------------------------------------------------------------------------- |
| loose          | 快速模式，將循環編譯成較為非規範的形式，可提高運行效率（具風險）<br/>默認值：`false`   |
| allowArrayLike | 允許類數組對象<br/>默認值：`false`                                                     |
| assumeArray    | 將循環對象視為數組（類數組對象、生成器、迭代器將被轉換成一般數組）<br/>默認值：`false` |

#### Sample

```json
{
  "presets": [],
  "plugins": [
    [
      "@babel/plugin-transform-for-of",
      {
        "loose": true,
        "allowArrayLike": true,
        "assumeArray": true
      }
    ]
  ]
}
```

---

可以看得出來，其實配置項官方都寫得清清楚楚嘛，本篇只是挑出一個預設跟一個插件舉例，其實對著官方說明使用就行啦！

# 結語

babel 在使用上基本就只有配置問題，要用哪些預設(presets)和插件(plugins)啊啥的。但是更重要的是這個工具本身所代表的意義，除了消除代碼與舊版瀏覽器的兼容性問題之外，他還提供了一種未來的可能，透過編寫自己的編譯插件，基本上是有可能支持無限多種的語法糖，是一套擴展性極強的預編譯處理器。
