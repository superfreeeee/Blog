# wrappy 源码解析

@[TOC](文章目录)

<!-- TOC -->

- [wrappy 源码解析](#wrappy-源码解析)
- [正文](#正文)
  - [0. 基本信息](#0-基本信息)
  - [1. 源码解析](#1-源码解析)
  - [2. 作者实现](#2-作者实现)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 正文

## 0. 基本信息

- version：`v1.0.2`
- 功能：实现函数科里化包装

## 1. 源码解析

整个项目很小，就一个方法

- `wrappy.js`（阅读笔记：`wrappy.js`）

整个函数的目标就是实现函数的包装，一共接受两个参数

```js
module.exports = wrappy;
function wrappy(fn, cb) {
  // 一次传入两个参数
  if (fn && cb) return wrappy(fn)(cb);
```

fn 为我们执行真正的函数之前需要执行的包装方法，cb 则是我们要包装的目标函数，然后返回一个包装后的函数，也就是可以利用 `fn` 实现一个高阶函数生成器

```js
  if (typeof fn !== 'function') {
    throw new TypeError('need wrapper function');
  }

  // 复制所有属性到包装函数里面
  Object.keys(fn).forEach(function (k) {
    wrapper[k] = fn[k];
  });

  return wrapper;

  function wrapper() {
    // 调用 fn，传入 cb 当参数（在 arguments 数组当中）
    var ret = fn.apply(this, arguments);
    var cb = arguments[arguments.length - 1];
    if (typeof ret === 'function' && ret !== cb) {
      Object.keys(cb).forEach(function (k) {
        ret[k] = cb[k];
      });
    }
    return ret;
  }
}
```

后半部分就是返回一个 wrapper 函数，延迟绑定需要的回调函数(`cb`，由 arguments 传入 fn 的调用)

对就这么简单hh

## 2. 作者实现

其实作者本身并不太喜欢看到像是什么 arguments 的调用hh，当然我更喜欢用比较新的语法如 `...args` 来传递参数，所以以下是作者仿造原库，自己实现的版本

- `wrappy_mine.js`

```js
module.exports = function wrappy(fn, cb) {
  if (fn && cb) return wrappy(fn)(cb);

  Object.assign(wrapper, fn);

  return wrapper;

  function wrapper(cb, ...args) {
    const ret = fn.apply(this, [cb, ...args]);
    if (typeof cb === 'function' && ret !== cb) {
      Object.assign(ret, cb);
    }
    return ret;
  }
};
```

其他属性值的传递使用 `Object.assign` 方法

# 其他资源

## 参考连接

| Title           | Link                                                                         |
| --------------- | ---------------------------------------------------------------------------- |
| wrappy - npm    | [https://www.npmjs.com/package/wrappy](https://www.npmjs.com/package/wrappy) |
| npm/wrappy - Github | [https://github.com/npm/wrappy](https://github.com/npm/wrappy)               |

## 阅读笔记参考

[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/wrappy-1.0.2](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/wrappy-1.0.2)
