# Babel 踩坑笔记: babel 命令失效(/bin/sh: babel: command not found)

@[TOC](文章目录)

<!-- TOC -->

- [Babel 踩坑笔记: babel 命令失效(/bin/sh: babel: command not found)](#babel-踩坑笔记-babel-命令失效binsh-babel-command-not-found)
- [前言](#前言)
- [正文](#正文)
  - [项目背景](#项目背景)
  - [问题描述](#问题描述)
  - [解决方案](#解决方案)
  - [开发注意点](#开发注意点)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

今天来分享一个使用 babel 的时候遇到的 bug，同时也是所有利用 node 创建可执行脚本命令的共同问题。

# 正文

## 项目背景

本问题适用于所有与 node 制作的命令行指令相关的问题

项目背景使用了 @babel/cli 包中自带的 babel 指令，首先先上配置文件

- `package.json`

```json
{
  "name": "bad_sample_xxx",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "scripts": {
    "build": "babel index.js -d lib"
  },
  "devDependencies": {
    "@babel/cli": "^7.14.5",
    "@babel/core": "^7.14.6",
    "@babel/preset-env": "^7.14.7"
  }
}
```

项目启动时的目标就是利用 @babel/cli 编译一个本地文件

## 问题描述

当我们使用 @babel/cli 的时候，就需要用到它附带的 `babel` 指令，如下使用方式

```bash
$ yarn babel index.js
```

这是最直接的使用命令的方式，结果是没问题的

![](https://picures.oss-cn-beijing.aliyuncs.com/img/babel_note_command_bad_sample1.png)

但是当我们换成使用 package.json 定义的命令的时候却报错了

![](https://picures.oss-cn-beijing.aliyuncs.com/img/babel_note_command_bad_sample2.png)

消息报错信息如下

```
yarn run v1.22.10
$ babel index.js -d lib
/bin/sh: babel: command not found
error Command failed with exit code 127.
info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
```

实际上问题在于 **项目路径**，这是由于上面使用的路径存在非法字符(前一个例子的目录名为 `bad_sample/xxx`)，由于 yarn 执行指令的原理在于将 `自身路径/.bin` 加入命令搜索列表，所以路径中包含 `/` 会造成命令搜索错误，最后变成 **command not found**

## 解决方案

解决方式就是将目录或是路径上存在不正常 `/` 符号的目录改名即可

如下我们把目录名从 `bad_sample/xxx` 改成 `good_sample`

再运行就正常了

![](https://picures.oss-cn-beijing.aliyuncs.com/img/babel_note_command_good_sample.png)

## 开发注意点

实际上在正常场景下 `yarn init` 或 `yarn install` 的时候就会检测路径是否存在异常字符，所以其实只要不是后来将项目擅自移动到异常路径下，实际上是比较难出现这个问题的。

# 结语

另外可能还有人是直接运行命令，但是实际上 npm 项目制作的命令是必须由 yarn 主动添加前缀，否则如果不是全局安装的话是找不到命令路径的。

# 其他资源

## 参考连接

| Title | Link |
| ----- | ---- |
|       | []() |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/babel/babel_note_command](https://github.com/superfreeeee/Blog-code/tree/main/front_end/babel/babel_note_command)

