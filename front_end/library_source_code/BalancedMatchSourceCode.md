# balanced-match 源码解析

@[TOC](文章目录)

<!-- TOC -->

- [balanced-match 源码解析](#balanced-match-源码解析)
- [正文](#正文)
  - [0. 基本信息](#0-基本信息)
    - [0.1 Usage](#01-usage)
    - [0.2 Version: v2.0.0](#02-version-v200)
    - [0.2 Doc](#02-doc)
  - [1. 源码解析](#1-源码解析)
    - [1.0 源码项目结构](#10-源码项目结构)
    - [1.1 主入口](#11-主入口)
    - [1.2 balanced](#12-balanced)
    - [1.3 maybeMatch](#13-maybematch)
    - [1.4 range](#14-range)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 正文

## 0. 基本信息

### 0.1 Usage

`balanced-match` 这个库的目标很简单：匹配第一个符合条件的一对字符串，同时将其拆解成前、中、后等三部分

### 0.2 Version: v2.0.0

这个库相对比较稳定了，也没啥好改的

本篇研究的是目前最新版本：`v2.0.0`

### 0.2 Doc

相关文档都写在 READEME 里面，还是比较简明的

传送门：[balanced-match - npm](https://www.npmjs.com/package/balanced-match)

## 1. 源码解析

### 1.0 源码项目结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/balanced_match_source_code_0_structure.png)

整个项目非常小，就是一个 `index.js`

### 1.1 主入口

- `index.js`（阅读笔记：`/index.js/0_structure.js`）

```js
'use strict';

function balanced(a, b, str) {}

function maybeMatch(reg, str) {}

balanced.range = range;

function range(a, b, str) {}

module.exports = balanced;
```

整个项目就三个方法，并向外导出两个方法：`balanced` 和 `balanced.range`

### 1.2 balanced

接下来来看主入口的细节

- `index.js`（阅读笔记：`/index.js/1_balanced.js`）

```js
/**
 * @param {string | RegExp} a
 * @param {string | RegExp} b
 * @param {string} str
 */
function balanced(a, b, str) {
  // 适配 regexp 输入
  if (a instanceof RegExp) a = maybeMatch(a, str);
  if (b instanceof RegExp) b = maybeMatch(b, str);

  // 寻找范围
  const r = range(a, b, str);

  // 计算前后结果
  return (
    r && {
      start: r[0],
      end: r[1],
      pre: str.slice(0, r[0]),
      body: str.slice(r[0] + a.length, r[1]),
      post: str.slice(r[1] + b.length),
    }
  );
}
```

函数同时允许传入字符串与正则表达式，所以先使用 `maybeMatch` 方法进行适配，然后调用 `range` 方法取得范围，最后重新从原字符串抽取返回结果

### 1.3 maybeMatch

- `index.js`（阅读笔记：`/index.js/2_maybeMatch.js`）

```js
/**
 * @param {RegExp} reg
 * @param {string} str
 */
function maybeMatch(reg, str) {
  // match[0] 为第一个匹配的字符串
  const m = str.match(reg);
  return m ? m[0] : null;
}
```

作者没有对正则表达式使用过多的奇淫技巧，实际上就是直接去原字符串搜有没有匹配的，然后直接转化回字符串

### 1.4 range

range 函数可以说是这个库的核心，在于根据 a、b 查找第一个非嵌套对，下面拆分成几个步骤

- `index.js`（阅读笔记：`/index.js/3_range.js`）

```js
/**
 * @param {string} a
 * @param {string} b
 * @param {string} str
 */
function range(a, b, str) {
  let begs, beg, left, right, result;
  let ai = str.indexOf(a);
  let bi = str.indexOf(b, ai + 1);
  let i = ai;

  // 至少存在一对结果
  if (ai >= 0 && bi > 0) {
    // 是否相等
    if (a === b) {
      return [ai, bi];
    }
    begs = [];
    left = str.length;
```

一开始先确保至少存在一对，同时如果匹配到同一个下标就直接返回(前后之间不可能存在任何其他嵌套对了)

```js
    while (i >= 0 && !result) {
      // 搜集所有匹配 a 的下标
      if (i === ai) {
        begs.push(i);
        ai = str.indexOf(a, i + 1);
      } else if (begs.length === 1) {
        // 只剩下一个 begs 时输出结果
        result = [begs.pop(), bi];
      } else {
        beg = begs.pop();
        if (beg < left) {
          // 每弹出一个 a，记录最后一对结果下标
          left = beg;
          right = bi;
        }

        bi = str.indexOf(b, i + 1);
      }

      i = ai < bi && ai >= 0 ? ai : bi;
    }
```

接下来是循环过程，首先收集所有符合 a 的字符串；然后一步步弹出 a 来与下一个 b 匹配，最后返回第一个符合条件的 a、b 对

```js
    // 对于 begs 没用完的情况（a 出现次数 > b）
    if (begs.length) {
      result = [left, right];
    }
  }

  return result;
}
```

对于 a 出现次数比 b 多的时候，则根据前面记录的 left、right 来找到最后一个符合条件的 a、b 对（也就是最外层的第一对字符串）

# 其他资源

## 参考连接

| Title                            | Link                                                                                                                                                                                               |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| balanced-match - npm             | [https://www.npmjs.com/package/balanced-match](https://www.npmjs.com/package/balanced-match)                                                                                                       |
| balanced-match - Github          | [https://github.com/juliangruber/balanced-match](https://github.com/juliangruber/balanced-match)                                                                                                   |
| String.prototype.indexOf() - MDN | [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/indexOf](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/indexOf) |

## 阅读笔记参考

[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/balanced-match-2.0.0](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/balanced-match-2.0.0)
