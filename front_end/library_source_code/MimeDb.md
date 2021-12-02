# mime-db 源码解析(Npm library)

@[TOC](文章目录)

<!-- TOC -->

- [mime-db 源码解析(Npm library)](#mime-db-源码解析npm-library)
- [正文](#正文)
  - [0. 基本信息](#0-基本信息)
  - [1. 源码解析](#1-源码解析)
    - [1.1 导出模块入口](#11-导出模块入口)
    - [1.2 MIME 数据来源](#12-mime-数据来源)
    - [1.3 db.json 生成](#13-dbjson-生成)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 正文

## 0. 基本信息

- version：`v1.51.0`
- 功能：导出标准 MIME 类型映射表

## 1. 源码解析

### 1.1 导出模块入口

mime-db 与 mime 类型，同时作为 mime 库构建的强依赖，而实际上 mime-db 的源码没啥内容，实际上就是导出一个 `db.json` 文件而已

- `index.js`

```js
module.exports = require('./db.json')
```

而这个 `db.json` 的内容结构如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/mime_db_1_structure.png)

第一层的 key 是 MIME 类型，映射到的对象有来源(`source`)、编码类型(`charset`)、是否可压缩(`compressible`)

### 1.2 MIME 数据来源

而这个 `db.json` 的数据又是哪来的呢？实际上作者写了一个类似爬虫的方式，从不同标准的网页抓取类型定义并生成统一格式

![](https://picures.oss-cn-beijing.aliyuncs.com/img/mime_db_2_src.png)

而抓取下来的文件就如上图被保存在 `src` 目录下，我们观察 `package.json` 文件就能看到类似的 fetch 指令

- `package.json`

```json
{
  "scripts": {
    "build": "node scripts/build",
    "fetch": "node scripts/fetch-apache && gnode scripts/fetch-iana && node scripts/fetch-nginx",
    "lint": "eslint .",
    "test": "mocha --reporter spec --bail --check-leaks test/",
    "test-ci": "nyc --reporter=lcov --reporter=text npm test",
    "test-cov": "nyc --reporter=html --reporter=text npm test",
    "update": "npm run fetch && npm run build",
    "version": "node scripts/version-history.js && git add HISTORY.md"
  }
}
```

而解析各网页爬下来的数据这里就不展开，其实就是对数据对象的适配和转换

### 1.3 db.json 生成

而 mime-db 做的事实际上就是再根据这些 `-types.json` 文件进行解析、合并然后生成最终的 `db.json` 导出

- `/scripts/build.js`

首先是导入数据，也就是前面提到的几个 `-types.json` 文件

```js
var db = {};

// initialize with all the IANA types
/**
 * 初始化所有 -types.json 文件
 */
// ? Read
addData(db, require('../src/iana-types.json'), 'iana');

// add the mime extensions from Apache
addData(db, require('../src/apache-types.json'), 'apache');

// add the mime extensions from nginx
addData(db, require('../src/nginx-types.json'), 'nginx');

// now add all our custom data
addData(db, require('../src/custom-types.json'));
```

`addData` 就是简单抽取目标对象的几个指定的需要属性罢了

```js
function addData(db, mime, source) {
  Object.keys(mime).forEach(function (key) {
    var data = mime[key];
    var type = key.toLowerCase();
    var obj = (db[type] = db[type] || createTypeEntry(source));

    // add missing data
    // 添加 obj.charset
    setValue(obj, 'charset', data.charset);
    // 添加 obj.compressible
    setValue(obj, 'compressible', data.compressible);

    // append new extensions
    // 添加 obj.extensions
    appendExtensions(obj, data.extensions);
  });
}
```

接下来是利用 `custom-suffix.json` 对 db 对象进行一些属性值的不全

```js
// finally, all custom suffix defaults
var mime = require('../src/custom-suffix.json');
// ? Read
// suffix => s
Object.keys(mime).forEach(function (suffix) {
  var s = mime[suffix];

  // type => d
  Object.keys(db).forEach(function (type) {
    // 相同后缀类型
    if (type.substr(0 - suffix.length) !== suffix) {
      return;
    }

    var d = db[type];
    if (d.compressible === undefined) d.compressible = s.compressible;
  });
});
```

最后调用 `write-db.js` 模块生成最终的 `db.json` 文件

- `/scripts/lib/write-db.js`

```js
module.exports = function writeDatabaseSync(fileName, obj) {
  var fd = fs.openSync(fileName, 'w');
  // keys 排序
  var keys = Object.keys(obj).sort();

  /**
   整体结构如下
   {
     "<type>": {
       "<key>": "<val>",
       // ...
       "<key>": "<val>"
     },
     // ...
   }
   */
  // {
  fs.writeSync(fd, '{\n');

  keys.forEach(function (key, i, arr) {
    // "<type>": {
    fs.writeSync(fd, '  ' + JSON.stringify(key) + ': {');

    var end = endLine.apply(this, arguments);
    var data = obj[key];
    var keys = Object.keys(data).sort(sortDataKeys);

    // 无内容直接闭合 }
    if (keys.length === 0) {
      fs.writeSync(fd, '}' + end);
      return;
    }

    // 有内容先换行
    fs.writeSync(fd, '\n');
    keys.forEach(function (key, i, arr) {
      var end = endLine.apply(this, arguments);
      var val = data[key];

      if (val !== undefined) {
        var str =
          Array.isArray(val) &&
          val.some(function (v) {
            return String(v).length > 15;
          })
            ? JSON.stringify(val, null, 2).split('\n').join('\n    ')
            : JSON.stringify(val);
        // "<key>": "<val>"<end>
        fs.writeSync(fd, '    ' + JSON.stringify(key) + ': ' + str + end);
      }
    });
    // }
    fs.writeSync(fd, '  }' + end);
  });

  // }
  fs.writeSync(fd, '}\n');

  fs.closeSync(fd);
};
```

实际上不得不说这个 write 有点无聊，或许可以使用像是 prettier 直接进行格式化就能生成好看的输出格式了

# 其他资源

## 参考连接

| Title                                  | Link                                                                                                                                                                                                           |
| -------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| mime-db - npm                          | [https://www.npmjs.com/package/mime-db](https://www.npmjs.com/package/mime-db)                                                                                                                                 |
| jshttp/mime-db - Github                | [https://github.com/jshttp/mime-db](https://github.com/jshttp/mime-db)                                                                                                                                         |
| String.prototype.localeCompare() - MDN | [https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/localeCompare](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/localeCompare) |

## 阅读笔记参考

[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/mime-db-1.51.0](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/mime-db-1.51.0)
