# Yarn 踩坑笔记: 安装异常 资源错误缓存处理(Error: SyntaxError: Invalid or unexpected token)

@[TOC](文章目录)

<!-- TOC -->

- [Yarn 踩坑笔记: 安装异常 资源错误缓存处理(Error: SyntaxError: Invalid or unexpected token)](#yarn-踩坑笔记-安装异常-资源错误缓存处理error-syntaxerror-invalid-or-unexpected-token)
- [正文](#正文)
  - [项目背景](#项目背景)
  - [问题描述](#问题描述)
  - [解决方案](#解决方案)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

## 项目背景

本篇描述的问题有可能出现在任何使用了包管理工具的前端项目中，并不只局限于 yarn。

## 问题描述

有时候我们发现项目安装完依赖之后(`npm install`、`yarn` 等)，突然发现跑不动了，报了个奇怪的错：

![](https://picures.oss-cn-beijing.aliyuncs.com/img/yarn_note_download_error_cache_1_error.png)

来我们放大一点报错信息

```
var _child_process = require("child_pr
                             ^^^^^^^^^

SyntaxError: Invalid or unexpected token
    at Object.compileFunction (node:vm:352:18)
    at wrapSafe (node:internal/modules/cjs/loader:1031:15)
    at Module._compile (node:internal/modules/cjs/loader:1065:27)
    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1153:10)
    at Module.load (node:internal/modules/cjs/loader:981:32)
    at Function.Module._load (node:internal/modules/cjs/loader:822:12)
    at Module.require (node:internal/modules/cjs/loader:1005:19)
    at require (node:internal/modules/cjs/helpers:102:18)
    at Object.<anonymous> (～/yarn_note_download_error_cache/node_modules/@babel/node/bin/babel-node.js:3:1)
    at Module._compile (node:internal/modules/cjs/loader:1101:14)
```

挑重点：

1. 异常类型：`SyntaxError: Invalid or unexpected token`
2. 抛错位置：`node_modules/@babel/node/bin/babel-node.js:3:1`

What?这么厉害的库有这种傻逼问题？进去看看出问题的文件到底怎么了

![](https://picures.oss-cn-beijing.aliyuncs.com/img/yarn_note_download_error_cache_2_file.png)

内容看起来出了点问题，尝试着删除 node_module 重新安装一遍

```bash
$ rm -rf node_modules && yarn && yarn start
```

结果还是一样

## 解决方案

实际上这是由于第一次安装的时候产生异常，导致文件内容下载不全，但是 yarn 并没有发现并将它 **缓存** 了，所以就算删掉重新 install 还是会产生一样的问题

因此我们只需要清理一下 yarn 的缓存重新安装就可以了

```bash
$ yarn cache clean
$ yarn reinstall  # write your shortcut in package.json like => "reinstall": "rm -rf node_modules && yarn"
```

# 其他资源

## 参考连接

| Title                   | Link                                                                       |
| ----------------------- | -------------------------------------------------------------------------- |
| yarn cache clean - yarn | [https://yarnpkg.com/cli/cache/clean](https://yarnpkg.com/cli/cache/clean) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/package/yarn_note_download_error_cache](https://github.com/superfreeeee/Blog-code/tree/main/front_end/package/yarn_note_download_error_cache)
