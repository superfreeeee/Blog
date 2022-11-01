# Webpack 踩坑笔记: webpack-dev-server 启动命令失败(Error: Cannot find module 'webpack-cli/bin/config-yargs')

@[TOC](文章目录)

<!-- TOC -->

- [Webpack 踩坑笔记: webpack-dev-server 启动命令失败(Error: Cannot find module 'webpack-cli/bin/config-yargs')](#webpack-踩坑笔记-webpack-dev-server-启动命令失败error-cannot-find-module-webpack-clibinconfig-yargs)
- [前言](#前言)
- [正文](#正文)
  - [项目背景](#项目背景)
  - [问题描述](#问题描述)
  - [解决方案](#解决方案)
    - [方案 1：使用新命令 webpack serve(推荐)](#方案-1使用新命令-webpack-serve推荐)
    - [方案 2：降低 webpack-cli 版本](#方案-2降低-webpack-cli-版本)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

今天给大家分享一个日常使用 webpack 的时候遇到的问题。

# 正文

## 项目背景

webpack 是前端打包工具的一方霸主，相信写过前端的人多多少少都知道 webpack 的存在也用过，使用 vue-cli 脚手架 `vue init webpack <repo-name>`，或是使用 create-react-app 脚手架 `npx create-react-app <repo-name>` 实际上都会创建一个使用了 webpack 的前端项目。

本篇单纯使用了 webpack、webpack-cli、webpack-dev-server 这三个包来复现问题场景。

## 问题描述

本篇使用了 `webpack、webpack-cli、webpack-dev-server` 三个使用 webpack 开发时都会引入的基础包

- `webpack` 为 webpack 的核心依赖
- `webpack-cli` 为 webpack 命令行指令的依赖
- `webpack-dev-server` 为开发时使用的开发服务器依赖(基于 NodeJS 创建一个轻量级 Http 服务器用于提供前端静态资源)

通常情况下我们都直接引入依赖最新的版本号如下

- `package.json`

```json
{
  // ...
  "devDependencies": {
    "webpack": "^5.39.0",
    "webpack-cli": "^4.7.2",
    "webpack-dev-server": "^3.11.2"
  }
}
```

然后我们使用旧版的 `webpack-dev-server` 启动命令时会报错

- `package.json`

```json
{
  // ...
  "scripts": {
    "old-serve": "webpack-dev-server",
  },
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_note_dev_server_old_console.png)

详细报错信息如下

```
superfree@chaoyouiandeMBP webpack_note_dev_server % yarn old-serve
yarn run v1.22.10
$ webpack-dev-server
internal/modules/cjs/loader.js:968
  throw err;
  ^

Error: Cannot find module 'webpack-cli/bin/config-yargs'
Require stack:
- /.../front_end/webpack_note_dev_server/node_modules/webpack-dev-server/bin/webpack-dev-server.js
    at Function.Module._resolveFilename (internal/modules/cjs/loader.js:965:15)
    at Function.Module._load (internal/modules/cjs/loader.js:841:27)
    at Module.require (internal/modules/cjs/loader.js:1025:19)
    at require (internal/modules/cjs/helpers.js:72:18)
    at Object.<anonymous> (/.../front_end/webpack_note_dev_server/node_modules/webpack-dev-server/bin/webpack-dev-server.js:65:1)
    at Module._compile (internal/modules/cjs/loader.js:1137:30)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:1157:10)
    at Module.load (internal/modules/cjs/loader.js:985:32)
    at Function.Module._load (internal/modules/cjs/loader.js:878:14)
    at Function.executeUserEntryPoint [as runMain] (internal/modules/run_main.js:71:12) {
  code: 'MODULE_NOT_FOUND',
  requireStack: [
    '/.../front_end/webpack_note_dev_server/node_modules/webpack-dev-server/bin/webpack-dev-server.js'
  ]
}
error Command failed with exit code 1.
info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
```

## 解决方案

实际上这是因为 webpack-cli@4+ 与 webpack-dev-server@3 的版本冲突问题

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_note_dev_server_old_version_conflict.png)

这时候我们有两种解决方案

### 方案 1：使用新命令 webpack serve(推荐)

第一种比较推荐，不需要修改依赖版本可以照常使用最新的依赖

我们可以放弃 `webpack-dev-server` 指令，而是该用 `webpack serve` 来替换

- `package.json`

```json
{
  // ...
  "scripts": {
+   "new-serve": "webpack serve --mode=production"
  },
}
```

运行结果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_note_dev_server_new_console.png)

我们可以看到使用新命令就能正常启动了

### 方案 2：降低 webpack-cli 版本

第二种解决方案，我们可以选择降低 webpack-cli 的版本来与 webpack-dev-server@3 兼容

我们将 webpack-cli 降低到前一个大版本号的最近版 `3.3.12`

```bash
$ yarn upgrade webpack-cli@^3.3.12
```

更新好后可以看到 webpack-cli 的版本已经更新了

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_note_dev_server_old_version_upgrade.png)

接下来我们就能够继续使用原来的 `webpack-dev-server` (前面定义过的 `yarn old-serve` 指令) 来启动我们的开发服务器

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_note_dev_server_old_console_upgrade.png)

# 结语

本篇介绍了两种针对 webpack-cli 与 webpack-dev-server 版本冲突的结局方案，希望遇到相同问题的人也能顺利解决，欢迎留言区讨论。

# 其他资源

## 参考连接

| Title                                                              | Link                                                                                                                               |
| ------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------- |
| Error: Cannot find module ‘webpack-cli/bin/config-yargs‘的解决方法 | [https://blog.csdn.net/cc18868876837/article/details/113901344](https://blog.csdn.net/cc18868876837/article/details/113901344)     |
| webpack-cli4 启动webpack-dev-server报错                            | [https://blog.csdn.net/hetingtingitsme/article/details/116426569](https://blog.csdn.net/hetingtingitsme/article/details/116426569) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/webpack/webpack_note_dev_server](https://github.com/superfreeeee/Blog-code/tree/main/front_end/webpack/webpack_note_dev_server)
