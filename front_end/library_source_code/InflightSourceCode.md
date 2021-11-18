# inflight 源码解析

@[TOC](文章目录)

<!-- TOC -->

- [inflight 源码解析](#inflight-源码解析)
- [正文](#正文)
  - [0. 基本信息](#0-基本信息)
  - [1. 源码解析](#1-源码解析)
    - [1.0 源码结构](#10-源码结构)
    - [1.1 主入口 inflight](#11-主入口-inflight)
    - [1.2 启动函数创建着 makeres](#12-启动函数创建着-makeres)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 正文

## 0. 基本信息

- version：`v1.0.6`
- 功能：收集多个回调函数并返回只执行一次的启动函数，启动时一次调用多个回调

这个与 Promise 的想法有所不同，Promise 的概念是第一次调用就会尽可能的往后执行知道挂起，这个库实现的范式则是返回一个延迟启动的函数，有点像是 Rust 中的 Future 懒执行，只有执行 `.await` 才会开始动作一样

这种范式对于函数执行时机的把控是更好的

## 1. 源码解析

### 1.0 源码结构

- `inflight.js`（阅读笔记：`/inflight.js/0_structure.js`）

```js
var wrappy = require('wrappy');
var reqs = Object.create(null);
var once = require('once');

module.exports = wrappy(inflight);

function inflight(key, cb) {}

function makeres(key) {}

function slice(args) {}
```

项目的主入口是使用 `wrappy` 包装的 `inflight` 函数

### 1.1 主入口 inflight

- `inflight.js`（阅读笔记：`/inflight.js/1_inflight.js`）

```js
/* 加入队列 */
// if reqs[key] => 简单加入队列
// else         => 创建队列 & 返回启动函数
function inflight(key, cb) {
  if (reqs[key]) {
    reqs[key].push(cb);
    return null;
  } else {
    reqs[key] = [cb];
    return makeres(key);
  }
}
```

主入口是一个收集回调函数的方法，按 README 里面的演示，可以多次反复调用 `inflight`，但是只有第一次返回启动函数

### 1.2 启动函数创建着 makeres

- `inflight.js`（阅读笔记：`/inflight.js/1_inflight.js`）

```js
/* 创建启动函数 */
function makeres(key) {
  // 只执行一次
  return once(function RES() {
    // 记录回调队列
    var cbs = reqs[key];
    var len = cbs.length;
    var args = slice(arguments);

    // XXX It's somewhat ambiguous whether a new callback added in this
    // pass should be queued for later execution if something in the
    // list of callbacks throws, or if it should just be discarded.
    // However, it's such an edge case that it hardly matters, and either
    // choice is likely as surprising as the other.
    // As it happens, we do go ahead and schedule it for later execution.
    try {
      // 执行回调
      for (var i = 0; i < len; i++) {
        cbs[i].apply(null, args);
      }
    } finally {
      if (cbs.length > len) {
        // 避免奇怪的人在执行回调的过程中加入新的函数
        // added more in the interim.
        // de-zalgo, just in case, but don't call again.
        cbs.splice(0, len);
        // 下个 tick 再执行剩余回调
        process.nextTick(function () {
          RES.apply(null, args);
        });
      } else {
        // 清理资源
        delete reqs[key];
      }
    }
  });
}
```

`makeres` 返回一个懒执行的启动函数 `RES`，启动函数里面就是调用所有回调，然后进行资源清理

# 其他资源

## 参考连接

| Title                 | Link                                                                             |
| --------------------- | -------------------------------------------------------------------------------- |
| inflight - npm        | [https://www.npmjs.com/package/inflight](https://www.npmjs.com/package/inflight) |
| npm/inflight - Github | [https://github.com/npm/inflight](https://github.com/npm/inflight)               |

## 阅读笔记参考

[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/inflight-1.0.6](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/inflight-1.0.6)
