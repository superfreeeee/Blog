# mime 源码解析(Npm library)

@[TOC](文章目录)

<!-- TOC -->

- [mime 源码解析(Npm library)](#mime-源码解析npm-library)
- [正文](#正文)
  - [0. 基本信息](#0-基本信息)
  - [1. 源码解析](#1-源码解析)
    - [1.0 MIME 类型定义](#10-mime-类型定义)
    - [1.1 标准版 vs 轻量版(lite)](#11-标准版-vs-轻量版lite)
    - [1.2 Mime 类](#12-mime-类)
    - [1.3 define](#13-define)
    - [1.4 getType & getExtension](#14-gettype--getextension)
  - [2. 构建依赖](#2-构建依赖)
    - [2.0 项目打包 & 发布](#20-项目打包--发布)
    - [2.1 mime-db 引入](#21-mime-db-引入)
    - [2.2 划分 standard/other](#22-划分-standardother)
  - [3. 小结](#3-小结)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 正文

## 0. 基本信息

- version：`v3.0.0`
- 功能：提供标准 MIME 类型与扩展名映射查询

## 1. 源码解析

### 1.0 MIME 类型定义

首先这个库本身最重要的就是这个 MIME 类型的定义，才有$类型 \leftrightarrow 扩展名$的映射

![](https://picures.oss-cn-beijing.aliyuncs.com/img/mime_1_types.png)

mime 库将类型定义保存在 `types` 目录下，又分成 `standard.js` 与 `other.js` 两个文件

导出对象我们也可以看到就是一个 `type => Array<ext>` 的数据结构

而实际上这里的类型定义是从 `mime-db` 这个库来的，我们在 2. 部分会说到

### 1.1 标准版 vs 轻量版(lite)

将类型定义分成两个文件的好处就是打包的时候可以选取需要的部分进行打包

mime 将发布版本分成标准版的(`require('mime')`)与轻量版(`require('mime/lite')`)

- 完整版 `index.js`

```js
'use strict';

let Mime = require('./Mime');

// ? Read
// 完整版引入 standard 与 other
module.exports = new Mime(
  require('./types/standard'),
  require('./types/other')
);
```

- 轻量版 `lite.js`

```js
'use strict';

let Mime = require('./Mime');

// ? Read
// 轻量版只引入 standard
module.exports = new Mime(require('./types/standard'));
```

### 1.2 Mime 类

看到上面两个入口文件，我们就可以很清楚地看出导出对象就是核心的 MIME 类型

- `Mime.js`

我们看到类型定义，也就是构造函数的部分

```js
'use strict';

/**
 * @param typeMap [Object] Map of MIME type -> Array[extensions]
 * @param ...
 */
// ? Read
function Mime() {
  // _types: ext => type
  this._types = Object.create(null);
  // _extensions: type => ext
  this._extensions = Object.create(null);

  for (let i = 0; i < arguments.length; i++) {
    this.define(arguments[i]);
  }

  this.define = this.define.bind(this);
  this.getType = this.getType.bind(this);
  this.getExtension = this.getExtension.bind(this);
}
```

MIME 类型有两个重要属性：
1. `_types` 保存了$扩展名 \to MIME 类型$的映射
2. `_extensions` 保存了$MIME 类型 \to 扩展名$的映射

### 1.3 define

构造函数里面使用了 `define` 方法来导入类型映射对象，默认导入的是 `standard.js` 与 `other.js` 对象，实际上我们也可以导入自定义的类型映射对象

下面我们就来看看 `define` 方法如何将映射对象转换为 `_types` 与 `_extensions` 保存

- `Mime.js`

```js
Mime.prototype.define = function (typeMap, force) {
  // typeMap: type => [ext]
  for (let type in typeMap) {
    let extensions = typeMap[type].map(function (t) {
      return t.toLowerCase();
    });
    type = type.toLowerCase();

    // this._types: ext => type
    for (let i = 0; i < extensions.length; i++) {
      const ext = extensions[i];

      // '*' prefix = not the preferred type for this extension.  So fixup the
      // extension, and skip it.
      // define 阶段忽略 * 开头
      if (ext[0] === '*') {
        continue;
      }

      // 不可重复定义
      if (!force && ext in this._types) {
        throw new Error(
          'Attempt to change mapping for "' + ext +
          '" extension from "' + this._types[ext] + '" to "' + type +
          '". Pass `force=true` to allow this, otherwise remove "' + ext +
          '" from the list of extensions for "' + type + '".'
        );
      }

      this._types[ext] = type;
    }

    // Use first extension as default
    // 默认使用第一个 extension 作 _extensions 的 key
    // this._extensions: type => ext
    if (force || !this._extensions[type]) {
      const ext = extensions[0];
      this._extensions[type] = ext[0] !== '*' ? ext : ext.substr(1);
    }
  }
};
```

实际上也很简单，就是将 type 与 extension 都转为小写，然后一个个映射对写进去

### 1.4 getType & getExtension

构造函数使用 `define` 方法导入之后，`getType`、`getExtension` 两个方法就很简单了

- `Mime.js`

```js
/**
 * Lookup a mime type based on extension
 * 按 ext 查 type
 */
// ? Read
Mime.prototype.getType = function (path) {
  path = String(path);
  // 忽略所有 .*\/ .
  let last = path.replace(/^.*[/\\]/, '').toLowerCase();
  let ext = last.replace(/^.*\./, '').toLowerCase();

  let hasPath = last.length < path.length; // 是否存在前缀
  let hasDot = ext.length < last.length - 1; // 是否有扩展名

  // 有扩展名 or 没有路径 => 否则返回 null
  return ((hasDot || !hasPath) && this._types[ext]) || null;
};

/**
 * Return file extension associated with a mime type
 * 按 type 查 ext
 */
// ? Read
Mime.prototype.getExtension = function (type) {
  type = /^\s*([^;\s]*)/.test(type) && RegExp.$1;
  return (type && this._extensions[type.toLowerCase()]) || null;
};

module.exports = Mime;
```

直接从 `this._types` 和 `this._extensions` 取，否则返回 null 就行了

## 2. 构建依赖

mime 这个库本身其实就是处理 MIME 类型映射，可以说是非常简单。

另一部分比较有趣的是我们可以看看 mime 这个库是如何根据 mime-db 这个库来生成我们前面提到的 `standard.js` 与 `other.js` 两个文件

### 2.0 项目打包 & 发布

首先我们可以看到 `package.json` 文件里面的 scripts，观察作者是如何进行打包

- `package.json`

```json
{
  "scripts": {
    "prepare": "node src/build.js && runmd --output README.md src/README_js.md",
    "release": "standard-version",
    "benchmark": "node src/benchmark.js",
    "md": "runmd --watch --output README.md src/README_js.md",
    "test": "mocha src/test.js"
  },
}
```

很明显的能看出打包指令文件大概就是这个 `build.js`

### 2.1 mime-db 引入

- `/src/build.js`

打包的一开始是先引入 mime-db，并提取 mime-db 导出的 `db.json`

```js
#!/usr/bin/env node

'use strict';

let fs = require('fs');
let path = require('path');
let mimeScore = require('mime-score');

let db = require('mime-db');
let chalk = require('chalk');
```

然后第一步则是先清理 mime-db 导出对象里定义重复定义的 extension

```js
let STANDARD_FACET_SCORE = 900;

let byExtension = {};

// Clear out any conflict extensions in mime-db
/**
 * 清理 mime-db 导出的重复 key 的 entry
 */
// ? Read
for (let type in db) {
  const entry = db[type];
  entry.type = type;
  if (!entry.extensions) continue;

  entry.extensions.forEach(function(ext) {
    let drop;
    let keep = entry;
    if (ext in byExtension) {
      let e0 = entry;
      let e1 = byExtension[ext];

      e0.pri = mimeScore(e0.type, e0.source);
      e1.pri = mimeScore(e1.type, e1.source);

      // drop score 比较低的
      drop = e0.pri < e1.pri ? e0 : e1;
      keep = e0.pri >= e1.pri ? e0 : e1;

      // Prefix lower-priority extensions with '*'
      drop.extensions = drop.extensions.map(function(e) {
        return e === ext ? '*' + e : e;
      });

      console.log(
        ext + ': Preferring ' + chalk.green(keep.type) + ' (' + keep.pri +
        ') over ' + chalk.red(drop.type) + ' (' + drop.pri + ')' + ' for ' + ext
      );
    }

    // Cache the hightest ranking type for this extension
    // byExtension: ext => entry
    if (keep === entry) byExtension[ext] = entry;
  });
}
```

### 2.2 划分 standard/other

第二步则是使用 mime-score 检查 MIME 类型登记，然后再分别写入两个文件

- `/src/build.js`

```js
// ? Read
// typesObj = {}
// => 'module.exports = typesObj;'
function writeTypesFile(types, path) {
  fs.writeFileSync(path, 'module.exports = ' + JSON.stringify(types) + ';');
}

// Segregate into standard and non-standard types based on facet per
// https://tools.ietf.org/html/rfc6838#section-3.1
let standard = {};
let other = {};

// ? Read
Object.keys(db).sort().forEach(function(k) {
  let entry = db[k];

  // 将 mime-db 导出的 db.json 分成 standard.js 与 other
  if (entry.extensions) {
    if (mimeScore(entry.type, entry.source) >= STANDARD_FACET_SCORE) {
      standard[entry.type] = entry.extensions;
    } else {
      other[entry.type] = entry.extensions;
    }
  }
});

// ? 写入 ../types
writeTypesFile(standard, path.join(__dirname, '../types', 'standard.js'));
writeTypesFile(other, path.join(__dirname, '../types', 'other.js'));
```

## 3. 小结

作者会想看这个库是作为研究 http-server 这个库的前置知识，同时除了能了解实际上 MIME 类型的几个核心来源之外，也能够观察这些库的打包发布手段

# 其他资源

## 参考连接

| Title                                                           | Link                                                                                                                                                                                                                                                           |
| --------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| mime - npm                                                      | [https://www.npmjs.com/package/mime](https://www.npmjs.com/package/mime)                                                                                                                                                                                       |
| MIME 类型 - MDN                                                 | [https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types#%E9%87%8D%E8%A6%81%E7%9A%84mime%E7%B1%BB%E5%9E%8B](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types#%E9%87%8D%E8%A6%81%E7%9A%84mime%E7%B1%BB%E5%9E%8B) |
| broofa/mime - Github                                            | [https://github.com/broofa/mime](https://github.com/broofa/mime)                                                                                                                                                                                               |
| mime-db - npm                                                   | [https://www.npmjs.com/package/mime-db](https://www.npmjs.com/package/mime-db)                                                                                                                                                                                 |
| mime-score - npm                                                | [https://www.npmjs.com/package/mime-score](https://www.npmjs.com/package/mime-score)                                                                                                                                                                           |
| rfc6838 - Media Type Specifications and Registration Procedures | [https://datatracker.ietf.org/doc/html/rfc6838](https://datatracker.ietf.org/doc/html/rfc6838)                                                                                                                                                                 |
| MIME Sniffing                                                   | [https://mimesniff.spec.whatwg.org/](https://mimesniff.spec.whatwg.org/)                                                                                                                                                                                       |

## 阅读笔记参考

[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/mime-3.0.0](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/mime-3.0.0)
