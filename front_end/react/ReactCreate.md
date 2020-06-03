# React 項目啟動

@[TOC](文章目錄)

<!-- TOC -->

- [React 項目啟動](#react-項目啟動)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [創建項目](#創建項目)
- [結語](#結語)

<!-- /TOC -->

## 簡介

我們都知道當前三大主流框架為 Vue、React、Angular，幾乎瓜分了前端框架市，更有著許多以此為基礎延伸的框架（服務端渲染、靜態渲染、移動端）。本篇來介紹如何啟動一個 React 項目，使用的是 `cra(create-react-app)` 腳手架；React 項目也可以透過配置 webpack 手動搭建，不過那種方式就留到介紹 webpack 的時候來演示。

## 參考

<table>
  <tr>
    <td>React 官方</td>
    <td><a href="https://zh-hans.reactjs.org/">https://zh-hans.reactjs.org/</a></td>
  </tr>
  <tr>
    <td>Create React App</td>
    <td><a href=https://create-react-app.dev/docs/getting-started"">https://create-react-app.dev/docs/getting-started</a></td>
  </tr>
</table>

# 正文

不得不說，`create-react-app` 是一個封裝極好的腳手架工具，零配置、開箱即用的特性非常適合剛入門接觸 React 的使用者，可配置選項如下：

```bash
Options:
  -V, --version                            output the version number
  --verbose                                print additional logs
  --info                                   print environment debug info
  --scripts-version <alternative-package>  use a non-standard version of react-scripts
  --template <path-to-template>            specify a template for the created project
  --use-npm
  --use-pnp
  --typescript                             (this option will be removed in favour of templates in the next major
                                           release of create-react-app)
  -h, --help                               output usage information
```

主要分成：

- 自定義命令 `--scripts-version`
- 使用模版 `--template`
- 包管理器 `--use-npm`

## 創建項目

創建項目可以透過 `npx` 命令：

```bash
$ npx create-react-app <project-name> <options>
```

也可以使用 `npm init` 命令（npm 版本 6+）：

```bash
$ npm init react-app <project-name> <options>
```

也可以先下載 create-react-app 到全局然後運用本地版本創建項目（不推薦）：

```bash
$ npm i -g create-react-app
$ create-react-app <project-name> <options>
```

運行截圖如下：

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/cra_1.png)

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/cra_2.png)

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/cra_3.png)

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/cra_4.png)

接下來輸入

```bash
$ cd react-demo
$ npm start
```

就能啟動項目啦！就這麼簡單！

# 結語

本篇非常非常簡單的介紹了使用 `create-react-app` 腳手架創建 React 項目，希望能為想入坑 React 而不知道怎麼開始的小夥伴起一個頭。
