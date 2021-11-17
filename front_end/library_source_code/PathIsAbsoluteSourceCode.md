# path-is-absolute 源码解析(Deprecated)

@[TOC](文章目录)

<!-- TOC -->

- [path-is-absolute 源码解析(Deprecated)](#path-is-absolute-源码解析deprecated)
- [正文](#正文)
  - [0. 基本信息](#0-基本信息)
  - [1. 源码解析](#1-源码解析)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 正文

## 0. 基本信息

- version：`v2.0.0`
- 功能：NodeJS 的 path 模块 `isAbsolute` 方法的 polyfill

## 1. 源码解析

其实可以看到作者的源码基本也是参考 NodeJS 的源码，这应该是比较早期的库了，基本上我们直接用 Node 或是主动 install 官方的 `path` 模块就行了，看看就好

- `index.js`（阅读笔记：`index.js`）

```js
'use strict';

function posix(path) {
  return path.charAt(0) === '/';
}

function win32(path) {
  // https://github.com/nodejs/node/blob/b3fcc245fb25539909ef1d5eaa01dbf92e168633/lib/path.js#L56
  var splitDeviceRe = /^([a-zA-Z]:|[\\/]{2}[^\\/]+[\\/]+[^\\/]+)?([\\/])?([\s\S]*?)$/;
  var result = splitDeviceRe.exec(path);
  var device = result[1] || '';
  var isUnc = Boolean(device && device.charAt(1) !== ':');

  // UNC paths are always absolute
  return Boolean(result[2] || isUnc);
}

module.exports = process.platform === 'win32' ? win32 : posix;
module.exports.posix = posix;
module.exports.win32 = win32;
```

# 其他资源

## 参考连接

| Title                                  | Link                                                                                                                     |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| path-is-absolute - npm                 | [https://www.npmjs.com/package/path-is-absolute](https://www.npmjs.com/package/path-is-absolute)                         |
| sindresorhus/path-is-absolute - Github | [https://github.com/sindresorhus/path-is-absolute](https://github.com/sindresorhus/path-is-absolute)                     |
| path.isAbsolute - NodeJS               | [https://nodejs.org/api/path.html#path_path_isabsolute_path](https://nodejs.org/api/path.html#path_path_isabsolute_path) |

## 阅读笔记参考

[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/path-is-absolute-2.0.0](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/path-is-absolute-2.0.0)
