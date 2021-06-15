# Vue 踩坑笔记: 引入 ElementUI 时打包失败修复记录(ERROR in ./node_modules/element-ui/lib/theme-chalk/index.css Module build failed: ModuleBuildError: Module build failed: TypeError: this.getOptions is not a function)

@[TOC](文章目录)

<!-- TOC -->

- [Vue 踩坑笔记: 引入 ElementUI 时打包失败修复记录(ERROR in ./node_modules/element-ui/lib/theme-chalk/index.css Module build failed: ModuleBuildError: Module build failed: TypeError: this.getOptions is not a function)](#vue-踩坑笔记-引入-elementui-时打包失败修复记录error-in-node_moduleselement-uilibtheme-chalkindexcss-module-build-failed-modulebuilderror-module-build-failed-typeerror-thisgetoptions-is-not-a-function)
- [前言](#前言)
- [正文](#正文)
  - [项目背景](#项目背景)
  - [问题描述](#问题描述)
  - [解决方案：降低 postcss-loader 版本](#解决方案降低-postcss-loader-版本)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

本篇重点在于分享使用 Vue 开发时遇到的版本踩坑笔记，希望能帮助到遇到相同问题的人

# 正文

## 项目背景

本次踩坑笔记的项目背景为：

使用 Vue 框架开发知识图谱可视化页面，引入 d3 作为图谱库，使用 ECharts 作辅助完成图标配置，全站以 ElementUI 作为基础 UI 风格框架

## 问题描述

遇到问题的是进行项目打包的时候，开发时使用 webpack-dev-server 还好好的，一打包(`yarn build`)就错误，错误信息如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/vue_note_elementui_postcss_old_error.png)

还有文字版的

```log
ERROR in ./node_modules/element-ui/lib/theme-chalk/index.css
Module build failed: ModuleBuildError: Module build failed: TypeError: this.getOptions is not a function
    at Object.loader (/.../Frontend-cosin/node_modules/postcss-loader/dist/index.js:40:24)
    at /.../Frontend-cosin/node_modules/webpack/lib/NormalModule.js:195:19
    at /.../Frontend-cosin/node_modules/loader-runner/lib/LoaderRunner.js:367:11
    at /.../Frontend-cosin/node_modules/loader-runner/lib/LoaderRunner.js:233:18
 @ ./src/main.js 13:0-46

ERROR in ./node_modules/element-ui/lib/theme-chalk/index.css
Module build failed: ModuleBuildError: Module build failed: TypeError: this.getOptions is not a function
    at Object.loader (/.../Frontend-cosin/node_modules/postcss-loader/dist/index.js:40:24)
    at /.../Frontend-cosin/node_modules/webpack/lib/NormalModule.js:195:19
    at /.../Frontend-cosin/node_modules/loader-runner/lib/LoaderRunner.js:367:11
    at /.../Frontend-cosin/node_modules/loader-runner/lib/LoaderRunner.js:233:18
    at runMicrotasks (<anonymous>)
    at processTicksAndRejections (internal/process/task_queues.js:97:5)
 @ ./node_modules/element-ui/lib/theme-chalk/index.css
 @ ./src/main.js

  Build failed with errors.

error Command failed with exit code 1.
info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
```

## 解决方案：降低 postcss-loader 版本

经过多方问题排查之后终于定位到问题，是 `postcss-loader` 的问题，其实我们也可以从报错信息中看到

```
at Object.loader (/.../Frontend-cosin/node_modules/postcss-loader/dist/index.js:40:24)
```

原来的 `package.json` 标记的版本号如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/vue_note_elementui_postcss_old_package_json.png)

主要问题在于：**`postcss-loader` 版本过高**

所以我们可以采用下面的命令来降低 `postcss-loader` 的版本到 **`4.0.4`**

```bash
$ yarn remove postcss-loader
$ yarn add postcss-loader@4.0.4 -D
```

更新依赖之后的 `package.json` 如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/vue_note_elementui_postcss_new_package_json.png)

确定版本降低之后，运行 `yarn build` 就打包成功了

![](https://picures.oss-cn-beijing.aliyuncs.com/img/vue_note_elementui_postcss_new_success.png)

大功告成

# 结语

本篇记录一次 Vue + ElementUI 开发时的踩坑经验，希望遇到相同问题的也能顺利解决，欢迎在留言区讨论。

# 其他资源

## 参考连接

| Title                                                    | Link                                                                             |
| -------------------------------------------------------- | -------------------------------------------------------------------------------- |
| 解决postcss、postcss-loader 和less-loader 导致的报错问题 | [https://www.jianshu.com/p/18f86a0bae33](https://www.jianshu.com/p/18f86a0bae33) |
| 【报错问题】Vue element-ui 提示 ‘element-ui/lib/theme-chalk/index.css’ 找不到                                                         | [https://blog.csdn.net/weixin_42512937/article/details/103475319](https://blog.csdn.net/weixin_42512937/article/details/103475319)                                                                             |

## 完整代码示例

[]()

