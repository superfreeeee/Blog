# single-line-log 源码解析(Npm library)

@[TOC](文章目录)

<!-- TOC -->

- [single-line-log 源码解析(Npm library)](#single-line-log-源码解析npm-library)
- [正文](#正文)
  - [0. 基本信息](#0-基本信息)
  - [1. 实现原理](#1-实现原理)
  - [2. 源码解析](#2-源码解析)
    - [2.1 逃脱序列定义](#21-逃脱序列定义)
    - [2.2 log 函数包装](#22-log-函数包装)
    - [2.3 代理标准流](#23-代理标准流)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 正文

## 0. 基本信息

- version：`v1.1.2`
- 功能：单行重复输出

在命令行我们常常有这么一个需求：需要在同一行不断输出相同的问题，或是修改部分文字，达到模拟动画的效果

今天要介绍的这个库就是用于实现单行重复输出的效果

## 1. 实现原理

在开始解析源码之前我们先来谈谈单行重复输出的实现原理。

动画本质上的意义实际上就是清空后重复输出，而对于命令行输出而言，清空前面的输出有几种方式：
1. 使用 `\b` 回退一个，然而这样有点蠢，塞一堆 `\b` 只为了清理一行，还不能指定游标位置
2. 使用 **ANSI escape sequences 逃脱序列**规范，简单来说就是基于 ASCII 规范的标准输入流定义了类似命令式的特殊字符写法，允许用户通过输出特殊字符来达到控制输出流游标的作用

下面是相关用法的介绍，可以自己去看看

传送门：[ANSI跳脫序列](https://zh.wikipedia.org/wiki/ANSI%E8%BD%AC%E4%B9%89%E5%BA%8F%E5%88%97)

## 2. 源码解析

下面我们就进入到源码解析的部分

### 2.1 逃脱序列定义

single-line-log 只用到了三种逃脱序列定义

- `index.js`

```js
//                          esc [ 1 0 0 0 D
var MOVE_LEFT = new Buffer('1b5b3130303044', 'hex').toString();
//                        esc [ 1 A
var MOVE_UP = new Buffer('1b5b3141', 'hex').toString();
//                           esc [ 0 K
var CLEAR_LINE = new Buffer('1b5b304b', 'hex').toString();
```

分别定义了：移动到当前行最左边、清理当前行、向上移动一行三种操作

### 2.2 log 函数包装

而核心的 log 函数就比较好理解了，记录输出字符的长度，并在下次输出前面加上清理用的逃脱序列就可以了

- `index.js`

```js
/**
 * 装饰 stream 输出流
 * @param {*} stream
 * @returns
 */
module.exports = function (stream) {
  // 保留原始 write 方法
  var write = stream.write;
  // 待写入字符串
  var str;

  stream.write = function (data) {
    // 保证数据源 str（所以说你的闭包呢hh）
    if (str && data !== str) str = null;
    return write.apply(this, arguments);
  };

  if (stream === process.stderr || stream === process.stdout) {
    process.on('exit', function () {
      // 退出 node 时打印空串做结
      if (str !== null) stream.write('');
    });
  }

  var prevLineCount = 0;
  var log = function () {
    str = '';
    var nextStr = Array.prototype.join.call(arguments, ' ');

    // Clear screen
    // 清理前一次的输出
    for (var i = 0; i < prevLineCount; i++) {
      str += MOVE_LEFT + CLEAR_LINE + (i < prevLineCount - 1 ? MOVE_UP : '');
    }

    // Actual log output
    // 真实输出
    str += nextStr;
    stream.write(str);

    // How many lines to remove on next clear screen
    // 计算下一次需要清理的距离
    var prevLines = nextStr.split('\n');
    prevLineCount = 0;
    for (var i = 0; i < prevLines.length; i++) {
      prevLineCount +=
        Math.ceil(stringWidth(prevLines[i]) / stream.columns) || 1;
    }
  };

  // 清理输出
  log.clear = function () {
    stream.write('');
  };

  return log;
};
```

参数接受一个输出流，并对输出流进行修饰（代理 write）方法，最后返回用于输出的 log 函数

log 方法实现核心步骤大致分为以下几步
1. 添加逃脱序列清理前一次输出字符
2. 打印当前输出
3. 计算当前输出长度，下次打印时需要清理的范围

### 2.3 代理标准流

最后还提供两个标准流的代理实现

- `index.js`

```js
// 代理 stdout、stderr
module.exports.stdout = module.exports(process.stdout);
module.exports.stderr = module.exports(process.stderr);
```

# 其他资源

## 参考连接

| Title                            | Link                                                                                                                                             |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| single-line-log - npm            | [https://www.npmjs.com/package/single-line-log](https://www.npmjs.com/package/single-line-log)                                                   |
| freeall/single-line-log - Github | [https://github.com/freeall/single-line-log](https://github.com/freeall/single-line-log)                                                         |
| ANSI跳脫序列 - wiki              | [https://zh.wikipedia.org/wiki/ANSI%E8%BD%AC%E4%B9%89%E5%BA%8F%E5%88%97](https://zh.wikipedia.org/wiki/ANSI%E8%BD%AC%E4%B9%89%E5%BA%8F%E5%88%97) |
| 跳脫字元 - wiki                  | [https://zh.wikipedia.org/wiki/%E8%BD%AC%E4%B9%89%E5%AD%97%E7%AC%A6](https://zh.wikipedia.org/wiki/%E8%BD%AC%E4%B9%89%E5%AD%97%E7%AC%A6)         |
| ASCII-Table                      | [https://en.m.wikipedia.org/wiki/File:ASCII-Table-wide.svg](https://en.m.wikipedia.org/wiki/File:ASCII-Table-wide.svg)                           |
| ES6 字符串的扩展                 | [http://caibaojian.com/es6/string.html](http://caibaojian.com/es6/string.html)                                                                   |

## 阅读笔记参考

[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/single-line-log-1.1.2](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/single-line-log-1.1.2)
