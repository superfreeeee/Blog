# mime-type 源码解析(Npm library)

@[TOC](文章目录)

<!-- TOC -->

- [mime-type 源码解析(Npm library)](#mime-type-源码解析npm-library)
- [正文](#正文)
  - [0. 基本信息](#0-基本信息)
  - [1. 源码解析](#1-源码解析)
    - [1.0 API](#10-api)
    - [1.1 MIME 类型源 & 初始化](#11-mime-类型源--初始化)
    - [1.2 lookup](#12-lookup)
    - [1.3 contentType](#13-contenttype)
    - [1.4 extension](#14-extension)
    - [1.5 charset](#15-charset)
  - [2. 小结](#2-小结)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 正文

## 0. 基本信息

- version：`v2.1.34`
- 功能：MIME 类型相关工具函数导出

本质上与 mime 库类似，不过 mime 库基于包大小的考量与用户使用习惯问题，简化了工具函数并优化打包体积

## 1. 源码解析

### 1.0 API

mime-type 相比于 mime 算是稍微比较旧一丢丢，不过保留的工具函数也是略多，使用的时候还是各取所需吧

下面是 mime-types 提供的几个 API

| Method            | Usage                                                                         |
| ----------------- | ----------------------------------------------------------------------------- |
| lookup(filename)  | 查找目标文件对应的 MIME 类型，实际上就是进行 $extension \to MIME type$ 的转换 |
| contentType(type) | 获取给定 MIME type 的完整 Type 说明                                           |
| extension(type)   | lookup 的相反，实现 $MIME type \to extension$ 的转换                          |
| charset(type)     | 查找 MIME 类型的编码类型                                                      |

### 1.1 MIME 类型源 & 初始化

mime-type 的类型依据也是依赖于 mime-db 这个库，然后在此之上进行一些简单的方法封装

代码入口的一开始就引入 mime-db 并进行导出变量的初始化

- `index.js`

```js
var db = require('mime-db');
var extname = require('path').extname;

/**
 * Module exports.
 * @public
 */

exports.charset = charset;
exports.charsets = { lookup: charset };
exports.contentType = contentType;
exports.extension = extension;
exports.extensions = Object.create(null); // type => ext[]
exports.lookup = lookup;
exports.types = Object.create(null); // ext => type

// 初始化 extensions/types
// Populate the extensions/types maps
populateMaps(exports.extensions, exports.types);
```

而这个初始化方法也比较简单，就是把 mime-db 导出的对象映射展开成 `exports.extensions` 与 `exports.types` 两个对象

```js
function populateMaps(extensions, types) {
  // source preference (least -> most)
  var preference = ['nginx', 'apache', undefined, 'iana'];

  Object.keys(db).forEach(function forEachMimeType(type) {
    var mime = db[type];
    var exts = mime.extensions;

    // 1. exts 为空
    if (!exts || !exts.length) {
      return;
    }

    // mime -> extensions
    extensions[type] = exts;

    // extension -> mime
    for (var i = 0; i < exts.length; i++) {
      var extension = exts[i];

      if (types[extension]) {
        var from = preference.indexOf(db[types[extension]].source);
        var to = preference.indexOf(mime.source);

        if (
          types[extension] !== 'application/octet-stream' &&
          (from > to ||
            (from === to && types[extension].substr(0, 12) === 'application/'))
        ) {
          // skip the remapping
          continue;
        }
      }

      // set the extension -> mime
      types[extension] = type;
    }
  });
}
```

### 1.2 lookup

下面我们按照 README 文档的 API 顺序依次讲解

- `index.js`

```js
var extname = require('path').extname;

function lookup(path) {
  if (!path || typeof path !== 'string') {
    return false;
  }

  // ========== 以上为类型检查 ==========

  // get the extension ("ext" or ".ext" or full path)
  // 获取 ext 的部分
  var extension = extname('x.' + path)
    .toLowerCase()
    .substr(1);

  if (!extension) {
    return false;
  }

  return exports.types[extension] || false;
}
```

lookup 方法进行简单的类型检查，然后使用 `path.extname` 抽取扩展名，最后从 `exports.types` 里面查找最终 MIME 类型

### 1.3 contentType

- `index.js`

```js
function contentType(str) {
  // TODO: should this even be in this module?
  if (!str || typeof str !== 'string') {
    return false;
  }

  // ========== 以上为类型检查 ==========

  // 有 /   => MIME 类型
  // 没有 / => 一般文件路径
  var mime = str.indexOf('/') === -1 ? exports.lookup(str) : str;

  if (!mime) {
    return false;
  }

  // TODO: use content-type or other module
  if (mime.indexOf('charset') === -1) {
    // 补上 charset
    var charset = exports.charset(mime);
    if (charset) mime += '; charset=' + charset.toLowerCase();
  }

  return mime;
}
```

contentType 方法则是在 lookup 的基础之上，多加一个 charset 的输出

### 1.4 extension

- `index.js`

```js
function extension(type) {
  if (!type || typeof type !== 'string') {
    return false;
  }

  // ========== 以上为类型检查 ==========

  // TODO: use media-typer
  // 从输入 type 抽取 MIME 类型
  var match = EXTRACT_TYPE_REGEXP.exec(type);

  // get extensions
  // 从 extensions 获取 exts
  var exts = match && exports.extensions[match[1].toLowerCase()];

  if (!exts || !exts.length) {
    return false;
  }

  // 返回第一种扩展名
  return exts[0];
}
```

extension 则与 lookup 一个样子，差别在于它是从 `exports.extensions` 查询结果

### 1.5 charset

- `index.js`

```js
function charset(type) {
  if (!type || typeof type !== 'string') {
    return false;
  }

  // ========== 以上为类型检查 ==========

  // 抽取 mime 类型
  // TODO: use media-typer
  var match = EXTRACT_TYPE_REGEXP.exec(type);
  var mime = match && db[match[1].toLowerCase()];

  if (mime && mime.charset) {
    // db.json 导出属性优先
    return mime.charset;
  }

  // default text/* to utf-8
  if (match && TEXT_TYPE_REGEXP.test(match[1])) {
    // text/* 的默认 UTF-8
    return 'UTF-8';
  }

  // 其他默认没有 charset
  return false;
}
```

charset 需要基于 mime-db 返回的对象进行查找，然后第二优先级则是赋予所有 `text/*` 类型默认 UTF-8 的编码集

## 2. 小结

MIME 类型相关的库有以下

| 库         | 作用                             |
| ---------- | -------------------------------- |
| mime-db    | MIME 类型的统一标准导出          |
| mime-type  | MIME 类型工具函数集合            |
| mime       | 优化体积的 MIME 类型工具函数集合 |
| mime-score | MIME 类型定义的标准程度评分      |

前面三个库作者都完成源码解析了，最后剩下 mime-score 库了

# 其他资源

## 参考连接

| Title                      | Link                                                                                 |
| -------------------------- | ------------------------------------------------------------------------------------ |
| mime-types - npm           | [https://www.npmjs.com/package/mime-types](https://www.npmjs.com/package/mime-types) |
| jshttp/mime-types - Github | [https://github.com/jshttp/mime-types](https://github.com/jshttp/mime-types)         |

## 阅读笔记参考

[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/mime-types-2.1.34](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/mime-types-2.1.34)
