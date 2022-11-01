# Webpack 插件: webpackbar & progress-bar-webpack-plugin 进度条插件分享

@[TOC](文章目录)

<!-- TOC -->

- [Webpack 插件: webpackbar & progress-bar-webpack-plugin 进度条插件分享](#webpack-插件-webpackbar--progress-bar-webpack-plugin-进度条插件分享)
- [系列文章](#系列文章)
- [前言](#前言)
- [正文](#正文)
  - [0. 为什么需要进度条？](#0-为什么需要进度条)
  - [1. 安装依赖](#1-安装依赖)
  - [2. 使用示例](#2-使用示例)
    - [2.1 progress-bar-webpack-plugin 效果](#21-progress-bar-webpack-plugin-效果)
    - [2.2 webpackbar 效果](#22-webpackbar-效果)
  - [3. 踩坑笔记: 进度条跑位](#3-踩坑笔记-进度条跑位)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 系列文章

- [Webpack：入門](https://blog.csdn.net/weixin_44691608/article/details/106675550)
- [Webpack：clean-webpack-plugin 清理資源](https://blog.csdn.net/weixin_44691608/article/details/106753972)

# 前言

今天来跟大家分享两个使用 webpack 作为打包工具的时候使用的两种进度条插件。

# 正文

## 0. 为什么需要进度条？

为什么需要加进度条呢？我们先看看下图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_bars_none_sample.gif)

普通基本款的 webpack 打包的时候大概就是长这样，如果项目很大很大，又没报错我都以为他卡住还是坏掉了，加个进度条看起来也比较舒服，等起来也比较有个底，总之提升一下开发幸福度！

## 1. 安装依赖

首先安装一下依赖，本篇要介绍的进度条插件有两种

1. [progress-bar-webpack-plugin](https://www.npmjs.com/package/progress-bar-webpack-plugin)

```bash
$ yarn add progress-bar-webpack-plugin -D
```

2. [webpackbar](https://www.npmjs.com/package/webpackbar)

```bash
$ yarn add webpackbar -D
```

## 2. 使用示例

使用上几乎差不多，就是在配置文件里面添加插件

- `webpack.config.js`

```js
const WebpackBar = require('webpackbar')
const ProgressBarWebpackPlugin = require('progress-bar-webpack-plugin')

module.exports = {
    // ...
    plugins: [
      new WebpackBar(),
      new ProgressBarWebpackPlugin(),
    ],
    // ...
```

下面直接看看效果

备注：由于基础项目体积太小看不到进度条出现，所以另外引入一个 `lodash` 库来增加打包体积，延长时间方便看出效果hh

### 2.1 progress-bar-webpack-plugin 效果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_bars_progress_bar_webpack_plugin_sample.gif)

### 2.2 webpackbar 效果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_bars_webpackbar_sample.gif)

个人认为 webpackbar 展现出来的效果还是比较好看的

## 3. 踩坑笔记: 进度条跑位

使用进度条的时候有一个重点，如果在打包的过程中输出了 webpack 以外的 **额外警告信息**，则会影响到进度条输出的效果(注意不是进度条插件本身的问题，de了很久的bug hh)

以下是我自己测试的时候遇到了的问题：babel 的设置为默认的 `compact: "auto"` 时，有可能会差生

```
[BABEL] Note: The code generator has deoptimised the styling of "unknown" as it exceeds the max of "500KB".
```

的提示信息，造成输出样式的跑位，如下图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_bars_progress_bar_webpack_plugin_sample2.gif)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_bars_webpackbar_sample2.gif)

这时候就需要在 babel 的配置项里面(本篇位于 `babel.config.json` 文件里面)手动将 compact 设置为固定值(`true、false` 都行，只要指定一个 babel 就不会警告)，就可以避免警告

```json
{
    "presets": ["@babel/preset-env"],
    "compact": true
}
```

最后放一下打包后的页面结果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_bars_page_sample.png)

完整项目示例可以参照文章末尾链接

# 结语

进度条插件简单来说就是让打包看起来没那么无聊，提升一下开发体验hh。同时如果输出跑位还可以顺便看看是报什么错或是警告，还可以顺便查一查学一学，了解一下平常没有注意到的打包运行细节

# 其他资源

## 参考连接

| Title                                                                                                  | Link                                                                                                                           |
| ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------ |
| npm packages - webpackbar                                                                              | [https://www.npmjs.com/package/webpackbar](https://www.npmjs.com/package/webpackbar)                                           |
| npm packages - progress-bar-webpack-plugin                                                             | [https://www.npmjs.com/package/progress-bar-webpack-plugin](https://www.npmjs.com/package/progress-bar-webpack-plugin)         |
| \[BABEL\] Note: The code generator has deoptimised the styling of "unknown" as it exceeds the max of " | [https://blog.csdn.net/xss392795158/article/details/103718334/](https://blog.csdn.net/xss392795158/article/details/103718334/) |
| webpack4 常用插件列表及使用说明                                                                        | [https://segmentfault.com/a/1190000015355816](https://segmentfault.com/a/1190000015355816)                                     |
| BABEL配置项中的COMPACT问题                                                                             | [https://www.freesion.com/article/9598855218/](https://www.freesion.com/article/9598855218/)                                   |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/webpack/webpack_bars](https://github.com/superfreeeee/Blog-code/tree/main/front_end/webpack/webpack_bars)
