# once 源码解析

@[TOC](文章目录)

<!-- TOC -->

- [once 源码解析](#once-源码解析)
- [正文](#正文)
  - [0. 基本信息](#0-基本信息)
  - [1. 源码解析](#1-源码解析)
    - [1.1 源码结构](#11-源码结构)
    - [1.2 once 基本款](#12-once-基本款)
    - [1.3 onceStrict 严格模式](#13-oncestrict-严格模式)
    - [1.4 原型方法](#14-原型方法)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 正文

## 0. 基本信息

- version：`v1.0.4`
- 功能：返回只执行一次的函数的高阶函数

吐槽：作者把 `wrappy` 库的 sample 包一包就也能发个包，也是很偷懒

## 1. 源码解析

### 1.1 源码结构

- `once.js`（阅读笔记：`/once.js/0_structure.js`）

```js
var wrappy = require('wrappy');
module.exports = wrappy(once);
module.exports.strict = wrappy(onceStrict);

once.proto = once(function () {});

function once(fn) {}

function onceStrict(fn) {}
```

整个库导出了两个方法，一个是基本款的 once；一个是严格版本onceStrict

### 1.2 once 基本款

- `once.js`（阅读笔记：`/once.js/1_once.js`）

```js
// called 标志是否调用过
// value 记录结果值
function once(fn) {
  var f = function () {
    if (f.called) return f.value;
    f.called = true;
    return (f.value = fn.apply(this, arguments));
  };
  f.called = false;
  return f;
}
```

基本款的基本就是用一个 `f.called` 属性来记录是否调用过了

注：这里其实可以加一句 `fn = null`，来释放对 fn 函数的指针，使 Node 编译器正确回收函数对象

### 1.3 onceStrict 严格模式

- `once.js`（阅读笔记：`/once.js/2_onceStrict.js`）

```js
// called 标志是否调用过
// onceError 为报错信息
function onceStrict(fn) {
  var f = function () {
    if (f.called) throw new Error(f.onceError);
    f.called = true;
    // 啊这里还要 value 干嘛啦，偷懒诶
    return (f.value = fn.apply(this, arguments));
  };
  var name = fn.name || 'Function wrapped with `once`';
  f.onceError = name + " shouldn't be called more than once";
  f.called = false;
  return f;
}
```

严格模式就是多次调用的时候多一个报错罢了

### 1.4 原型方法

- `once.js`（阅读笔记：`/once.js/3_proto.js`）

```js
once.proto = once(function () {
  Object.defineProperty(Function.prototype, 'once', {
    value: function () {
      return once(this);
    },
    configurable: true,
  });

  Object.defineProperty(Function.prototype, 'onceStrict', {
    value: function () {
      return onceStrict(this);
    },
    configurable: true,
  });
});
```

这个 proto 方法有点不知道作者想干嘛，实用价值说实在偏低，而且还会污染全局的 Function.prototype 方法

# 其他资源

## 参考连接

| Title       | Link                                                                     |
| ----------- | ------------------------------------------------------------------------ |
| once - npm  | [https://www.npmjs.com/package/once](https://www.npmjs.com/package/once) |
| isaacs/once | [https://github.com/isaacs/once](https://github.com/isaacs/once)         |

## 阅读笔记参考

[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/once-1.4.0](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/once-1.4.0)
