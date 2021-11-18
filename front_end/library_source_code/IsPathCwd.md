# is-path-cwd 源码解析(Npm library)

@[TOC](文章目录)

<!-- TOC -->

- [is-path-cwd 源码解析(Npm library)](#is-path-cwd-源码解析npm-library)
- [正文](#正文)
  - [0. 基本信息](#0-基本信息)
  - [1. 源码解析](#1-源码解析)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 正文

## 0. 基本信息

- version：`v3.0.0`
- 功能：检查字符串是否为当前工作目录（与 `process.cwd` 比较）

## 1. 源码解析

整个库唯一的功能就是与 `process.cwd()` 比较，然后对 win32 做忽略大小写的兼容

- `index.js`（阅读笔记：`index.js`）

```js
import process from 'node:process';
import path from 'node:path';

export default function isPathCwd(path_) {
  let cwd = process.cwd();

  path_ = path.resolve(path_);

  // win32 下忽略大小写
  if (process.platform === 'win32') {
    cwd = cwd.toLowerCase();
    path_ = path_.toLowerCase();
  }

  // 与 process.cwd 方法返回值做比较
  return path_ === cwd;
}
```

# 其他资源

## 参考连接

| Title                             | Link                                                                                                                                           |
| --------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| is-path-cwd - npm                 | [https://www.npmjs.com/package/is-path-cwd](https://www.npmjs.com/package/is-path-cwd)                                                         |
| sindresorhus/is-path-cwd - Github | [https://github.com/sindresorhus/is-path-cwd](https://github.com/sindresorhus/is-path-cwd)                                                     |
| process.cwd() - NodeJS            | [https://nodejs.org/dist/latest-v16.x/docs/api/process.html#processcwd](https://nodejs.org/dist/latest-v16.x/docs/api/process.html#processcwd) |

## 阅读笔记参考

[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/is-path-cwd-3.0.0](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/is-path-cwd-3.0.0)
