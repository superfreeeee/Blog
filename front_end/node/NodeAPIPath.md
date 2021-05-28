# Node API: Path 模块总整理

@[TOC](文章目录)

<!-- TOC -->

- [Node API: Path 模块总整理](#node-api-path-模块总整理)
- [前言](#前言)
- [正文](#正文)
  - [0. API 概述](#0-api-概述)
  - [1. path.basename(path, ext) 返回目标文件名](#1-pathbasenamepath-ext-返回目标文件名)
  - [2. path.dirname(path) 返回文件所在目录名](#2-pathdirnamepath-返回文件所在目录名)
  - [3. path.extname(path) 返回文件扩展名](#3-pathextnamepath-返回文件扩展名)
  - [4. path.sep、path.delimiter 返回当前系统路径分界符/分隔符](#4-pathseppathdelimiter-返回当前系统路径分界符分隔符)
  - [5. path.normalize(path) 标准化路径](#5-pathnormalizepath-标准化路径)
  - [6. path.isAbsolute(path) 检查是否为绝对路径](#6-pathisabsolutepath-检查是否为绝对路径)
  - [7. path.join(...paths) 返回合成路径结果](#7-pathjoinpaths-返回合成路径结果)
  - [8. path.resolve(...paths) 返回合成路径的绝对路径](#8-pathresolvepaths-返回合成路径的绝对路径)
  - [9. path.format(obj) 返回结果路径](#9-pathformatobj-返回结果路径)
  - [10. path.parse(path) 返回路径解析对象](#10-pathparsepath-返回路径解析对象)
  - [11. path.relative(from, to) 返回相对路径](#11-pathrelativefrom-to-返回相对路径)
  - [12. path.posix、path.win32 返回 posix / win32 相关方法](#12-pathposixpathwin32-返回-posix--win32-相关方法)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

本篇要来介绍 Node.js 下的 `Path` 模块的所有 API 各个方法的作用和演示，主要作为学习记录，详细 API 可以参考官网文档，本篇可能在某些部分不会过分深入讨论(~~能用就好了~~)，下面我们进入正题。

# 正文

## 0. API 概述

看模块名就知道，path 模块是与路径相关的一些操作方法。首先先上一张 API 表

| Method                     | Parameter                        | Description            |
| -------------------------- | -------------------------------- | ---------------------- |
| `path.basename(path, ext)` | path 目标路径<br/>ext 除去扩展名 | 返回目标文件名         |
| `path.dirname(path)`       | path 目标路径                    | 返回文件所在目录名     |
| `path.extname(path)`       | path 目标路径                    | 返回文件扩展名         |
| `path.sep`                 |                                  | 返回当前系统路径分界符 |
| `path.delimiter`           |                                  | 返回当前系统路径分隔符 |
| `path.normalize(path)`     | path 目标路径                    | 标准化路径             |
| `path.isAbsolute(path)`    | path 目标路径                    | 检查是否为绝对路径     |
| `path.join(...paths)`      | paths 路径片段                   | 返回合成路径结果       |
| `path.resolve(...paths)`   | paths 路径片段                   | 返回合成路径的绝对路径 |
| `path.format(obj)`         | obj 路径描述对象                 | 返回结果路径           |
| `path.parse(path)`         | path 目标路径                    | 返回路径解析对象       |
| `path.relative(from, to)`  | from 原始路径<br/>to 预比较路径  | 返回相对路径           |
| `path.posix`               |                                  | 返回 posix 相关方法    |
| `path.win32`               |                                  | 返回 win32 相关方法    |

- 工具方法：下面示例代码用到的几个工具方法，避免误会

- `src/utils.js`

```js
const log = console.log

const group = (tag, cb) => {
  console.group(tag)
  cb()
  console.groupEnd()
}

module.exports = {
  log,
  group,
}
```

下面我们给出每个 API 的测试代码(详细作用、各个 API 的特别注意点都放在代码注释里面了，测试代码的运行平台为 macOS 属于 posix 类型(相对于 win32))

## 1. path.basename(path, ext) 返回目标文件名

- `src/path/basename.js`

```js
const { log, group } = require('../utils')

const path = require('path')

/* 当前文件名(可去除扩展名) */
group('path.basename(path, ext)', () => {
  log(`path.basename('/a/index.html'):          ${path.basename('/a/index.html')}`)
  log(`path.basename('./a/index.html'):         ${path.basename('./a/index.html')}`)
  log(`path.basename('a/index.html'):           ${path.basename('a/index.html')}`)
  log(`path.basename('a/index.html/'):          ${path.basename('a/index.html/')}`)  // 忽略末尾分隔符
  log(`path.basename('C:\\a\\index.html'):        ${path.basename('C:\\a\\index.html')}`)  // 不处理非本平台分隔符
  log(`path.basename('/a/index.html', '.html'): ${path.basename('/a/index.html', '.html')}`)  // 去除扩展名
  log(`path.basename('/a/index.html', '.c'):    ${path.basename('/a/index.html', '.c')}`)
})
```

- 输出

```
path.basename(path, ext)
  path.basename('/a/index.html'):          index.html
  path.basename('./a/index.html'):         index.html
  path.basename('a/index.html'):           index.html
  path.basename('a/index.html/'):          index.html
  path.basename('C:\a\index.html'):        C:\a\index.html
  path.basename('/a/index.html', '.html'): index
  path.basename('/a/index.html', '.c'):    index.html
```

## 2. path.dirname(path) 返回文件所在目录名

- `src/path/dirname.js`

```js
const { log, group } = require('../utils')

const path = require('path')

/* 返回当前文件目录 */
group('path.dirname(path)', () => {
  log(`path.dirname('/a/index.html'): ${path.dirname('/a/index.html')}`)
  log(`path.dirname('/a/index'):      ${path.dirname('/a/index')}`)
  log(`path.dirname('/a/index/'):     ${path.dirname('/a/index/')}`) // 忽略末尾分隔符
})
```

- 输出

```
path.dirname(path)
  path.dirname('/a/index.html'): /a
  path.dirname('/a/index'):      /a
  path.dirname('/a/index/'):     /a
```

## 3. path.extname(path) 返回文件扩展名

- `src/path/extname.js`

```js
const { log, group } = require('../utils')

const path = require('path')

/* 返回文件扩展名 */
group('path.extname(path)', () => {
  log(`path.extname('/a/index.html'):          ${path.extname('/a/index.html')}`) // 取 . 后作为扩展名
  log(`path.extname('/a/index.test.js'):       ${path.extname('/a/index.test.js')}`)
  log(`path.extname('/a/index.test.js/'):      ${path.extname('/a/index.test.js/')}`) // 忽略末尾分隔符
  log(`path.extname('/a/index.test.js/index'): ${path.extname('/a/index.test.js/index')}`) // 只取最后一个文件的最后一个 . 后作为扩展名
})
```

- 输出

```
path.extname(path)
  path.extname('/a/index.html'):          .html
  path.extname('/a/index.test.js'):       .js
  path.extname('/a/index.test.js/'):      .js
  path.extname('/a/index.test.js/index'): (输出为 '')
```

## 4. path.sep、path.delimiter 返回当前系统路径分界符/分隔符

- `src/path/sep_delimiter.js`

```js
const { log, group } = require('../utils')

const path = require('path')

/* 系统路径分界符 */
group('path.delimiter', () => {
  log(`path.sep: '${path.sep}'`) // 返回当前系统目录分隔符
  log(`path.delimiter: '${path.delimiter}'`) // 返回当前系统路径分隔符
  log(`process.env.PATH: ${process.env.PATH}`)
  log('process.env.PATH.split(path.delimiter):', process.env.PATH.split(path.delimiter))
})
```

- 输出

```
path.delimiter
  path.sep: '/'
  path.delimiter: ':'
  process.env.PATH: /usr/local/mongodb-macos-x86_64-4.4.4/bin:/usr/local/redis-6.2.1/src:/Users/superfree/.cargo/bin
  process.env.PATH.split(path.delimiter): [
    '/usr/local/mongodb-macos-x86_64-4.4.4/bin',
    '/usr/local/redis-6.2.1/src',
    '/Users/superfree/.cargo/bin'
  ]
```

(去除了大部分的 env.PATH 不给你看，能看出作用就行了)

## 5. path.normalize(path) 标准化路径

- `src/path/normalize.js`

```js
const { log, group } = require('../utils')

const path = require('path')

/* 标准化路径 */
group('path.normalize(path)', () => {
  log(`path.normalize(''):                    ${path.normalize('')}`) // 默认 . (参数不可为空)
  log(`path.normalize('/a/b/../c/////./../'): ${path.normalize('/a/b/../c/////./../')}`) // 简化 .、..、多重分隔符
})
```

- 输出

```
path.normalize(path)
  path.normalize(''):                    .
  path.normalize('/a/b/../c/////./../'): /a/
```

## 6. path.isAbsolute(path) 检查是否为绝对路径

- `src/path/isAbsolute.js`

```js
const { log, group } = require('../utils')

const path = require('path')

/* 检查是否为绝对路径 */
group('path.isAbsolute(path)', () => {
  log(`path.isAbsolute('/a/index.html'):  ${path.isAbsolute('/a/index.html')}`)
  log(`path.isAbsolute('a/index.html'):   ${path.isAbsolute('a/index.html')}`)
  log(`path.isAbsolute('./a/index.html'): ${path.isAbsolute('./a/index.html')}`)
  log(`path.isAbsolute('./a/index.html'): ${path.isAbsolute('./a/index.html')}`)
})
```

- 输出

```
path.isAbsolute(path)
  path.isAbsolute('/a/index.html'):  true
  path.isAbsolute('a/index.html'):   false
  path.isAbsolute('./a/index.html'): false
  path.isAbsolute('./a/index.html'): false
```

## 7. path.join(...paths) 返回合成路径结果

- `src/path/join.js`

```js
const { log, group } = require('../utils')

const path = require('path')

/* 合成路径 */
group('path.join(...paths)', () => {
  log(`path.join('a', 'b', 'c/d', '..', 'e'):  ${path.join('a', 'b', 'c/d', '..', 'e')}`) // 使用相对路径
  log(`path.join('/a', 'b', 'c/d', '..', 'e'): ${path.join('/a', 'b', 'c/d', '..', 'e')}`) // 使用绝对路径
  log(`path.join():                            ${path.join()}`) // 默认为当前目录 .
})
```

- 输出

```
path.join(...paths)
  path.join('a', 'b', 'c/d', '..', 'e'):  a/b/c/e
  path.join('/a', 'b', 'c/d', '..', 'e'): /a/b/c/e
  path.join():                            .
```

## 8. path.resolve(...paths) 返回合成路径的绝对路径

- `src/path/resolve.js`

```js
const { log, group } = require('../utils')

const path = require('path')

/* 合并路径后生成绝对路径 */
group('path.resolve(...paths)', () => {
  log(`path.resolve('a', 'b', 'c'):    ${path.resolve('a', 'b', 'c')}`) // 相对路径
  log(`path.resolve('/a', 'b', 'c'):   ${path.resolve('/a', 'b', 'c')}`) // 绝对路径
  log(`path.resolve('/a', 'b/', 'c/'): ${path.resolve('/a', 'b/', 'c/')}`) // 尾部分隔符被删除
})
```

- 输出

```
path.resolve(...paths)
  path.resolve('a', 'b', 'c'):    /Users/username/Desktop/Blog/code/test/front_end/node_api/a/b/c
  path.resolve('/a', 'b', 'c'):   /a/b/c
  path.resolve('/a', 'b/', 'c/'): /a/b/c
```

## 9. path.format(obj) 返回结果路径

- `src/path/format.js`

```js
const { log, group } = require('../utils')

const path = require('path')

/* 按对象构建文件名 */
group('path.format(pathObj)', () => {
  log(
    `path.format({ root: 'ignored', dir: '/dir', base: 'base.txt' }):             ${path.format({
      root: 'ignored',
      dir: '/dir',
      base: 'base.txt',
    })}`
  ) // 有 dir 时忽略 root
  log(
    `path.format({ root: '/root' + path.sep, base: 'base.txt', ext: 'ignored' }): ${path.format({
      root: '/root' + path.sep,
      base: 'base.txt',
      ext: 'ignored',
    })}`
  ) // 用 root 时不添加分隔符(自己加)，并忽略 ext
  log(
    `path.format({ dir: '/dir', name: 'name', ext: '.txt' }):                     ${path.format({
      dir: '/dir',
      name: 'name',
      ext: '.txt',
    })}`
  ) // 没有 base 时使用 name + ext
})
```

- 输出

```
path.format(pathObj)
  path.format({ root: 'ignored', dir: '/dir', base: 'base.txt' }):             /dir/base.txt
  path.format({ root: '/root' + path.sep, base: 'base.txt', ext: 'ignored' }): /root/base.txt
  path.format({ dir: '/dir', name: 'name', ext: '.txt' }):                     /dir/name.txt
```

## 10. path.parse(path) 返回路径解析对象

- `src/path/parse.js`

```js
const { log, group } = require('../utils')

const path = require('path')

/* 解析路径 */
group('path.parse(path)', () => {
  log(`path.parse('/a/index.html'):`, path.parse('/a/index.html')) // root 对应根目录
  log(`path.parse('./a/index.html/'):`, path.parse('./a/index.html/')) // 忽略尾部分隔符
  log(`path.parse('../a/../index.test.js'):`, path.parse('../a/../index.test.js')) // 不会对路径进行标准化
})
```

- 输出

```
path.parse(path)
  path.parse('/a/index.html'): {
    root: '/',
    dir: '/a',
    base: 'index.html',
    ext: '.html',
    name: 'index'
  }
  path.parse('./a/index.html/'): {
    root: '',
    dir: './a',
    base: 'index.html',
    ext: '.html',
    name: 'index'
  }
  path.parse('../a/../index.test.js'): {
    root: '',
    dir: '../a/..',
    base: 'index.test.js',
    ext: '.js',
    name: 'index.test'
  }
```

## 11. path.relative(from, to) 返回相对路径

- `src/path/relative.js`

```js
const { log, group } = require('../utils')

const path = require('path')

/* 返回相对路径 */
group('path.relative(from, to)', () => {
  log(`path.relative('/a/b/c', '/c/d/e'):    ${path.relative('/a/b/c', '/c/d/e')}`)
  log(`path.relative('/a/b/c', '/a/b/c/d'):  ${path.relative('/a/b/c', '/a/b/c/d')}`)
  log(`path.relative('./a/b/c', '/a/b/c/d'): ${path.relative('./a/b/c', '/a/b/c/d')}`)
})
```

- 输出

```
path.relative(from, to)
  path.relative('/a/b/c', '/c/d/e'):    ../../../c/d/e
  path.relative('/a/b/c', '/a/b/c/d'):  d
  path.relative('./a/b/c', '/a/b/c/d'): ../../../../../../../../../../../a/b/c/d
```

## 12. path.posix、path.win32 返回 posix / win32 相关方法

- `src/path/extname.js`

```js
const { log, group } = require('../utils')

const path = require('path')

/* 平台特定实现 */
group('path.posix / path.win32', () => {
  log(`path.posix.delimiter: '${path.posix.delimiter}'`)
  log(`path.posix.sep:       '${path.posix.sep}'`)
  log(`path.win32.delimiter: '${path.win32.delimiter}'`)
  log(`path.win32.sep:       '${path.win32.sep}'`)
})
```

- 输出

```
path.posix / path.win32
  path.posix.delimiter: ':'
  path.posix.sep:       '/'
  path.win32.delimiter: ';'
  path.win32.sep:       '\'
```

# 结语

本篇主要是简单记录 Node API 的 path 模块使用，后续会在继续整理其他模块的使用。有机会再写写别的有关基于 Node.js 的应用实现。

# 其他资源

## 参考连接

| Title                             | Link                                                             |
| --------------------------------- | ---------------------------------------------------------------- |
| Node.js v14.16.1 文档 - path 模块 | [http://nodejs.cn/api/path.html](http://nodejs.cn/api/path.html) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/node:express/node_api](https://github.com/superfreeeee/Blog-code/tree/main/front_end/node:express/node_api)
