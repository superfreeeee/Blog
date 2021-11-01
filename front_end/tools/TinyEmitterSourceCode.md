# tiny-emitter 源码解析

@[TOC](文章目录)

<!-- TOC -->

- [tiny-emitter 源码解析](#tiny-emitter-源码解析)
- [正文](#正文)
  - [0. 关于 Emitter](#0-关于-emitter)
  - [1. 源码结构](#1-源码结构)
  - [2. 监听事件 on](#2-监听事件-on)
  - [3. 取消监听 off](#3-取消监听-off)
  - [4. 触发事件 emit](#4-触发事件-emit)
  - [5. 单次触发事件 once](#5-单次触发事件-once)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 正文

## 0. 关于 Emitter

`tiny-emitter` 是 Node 提供的 EventEmitter 的简单版本，也就是面试的时候常见的 Emitter 实现

## 1. 源码结构

tiny-emitter 的源码就只有一个 `index.js` 文件

- `/index.js`（阅读笔记：`/index.js/1_structure.js`）

```js
function E () {
  // Keep this empty so it's easier to inherit from
  // (via https://github.com/lipsmack from https://github.com/scottcorgan/tiny-emitter/issues/3)
}

E.prototype = {
  on: function (name, callback, ctx) {},
  once: function (name, callback, ctx) {},
  emit: function (name) {},
  off: function (name, callback) {}
};

module.exports = E;
module.exports.TinyEmitter = E;
```

一共就实现了四个方法，同时为了保证方便继承，构造函数为空

## 2. 监听事件 on

- `/index.js`（阅读笔记：`/index.js/2_on.js`）

```js
E.prototype = {
  // ...

  on: function (name, callback, ctx) {
    var e = this.e || (this.e = {});

    (e[name] || (e[name] = [])).push({
      fn: callback,
      ctx: ctx
    });

    return this;
  },
  
  // ...
};
```

`e` 为监听函数对象，保存了 `Map<name, listener[]>` 的结构，on 方法使用 `(e[name] || (e[name] = []))` 来保证监听函数列表的存在

## 3. 取消监听 off

- `/index.js`（阅读笔记：`/index.js/3_off.js`）

off 稍微复杂一些，一开始先挑出原始的监听队列，并将非目标的监听函数复制到新的队列

```js
E.prototype = {
  // ...

  off: function (name, callback) {
    var e = this.e || (this.e = {});
    var evts = e[name];
    var liveEvents = [];
  
    if (evts && callback) {
      for (var i = 0, len = evts.length; i < len; i++) {
        if (evts[i].fn !== callback && evts[i].fn._ !== callback)
          liveEvents.push(evts[i]);
      }
    }
```

最后检查如果没有监听函数了就释放数组

```js
    (liveEvents.length)
      ? e[name] = liveEvents
      : delete e[name];
  
    return this;
  }
  
  // ...
};
```

## 4. 触发事件 emit

- `/index.js`（阅读笔记：`/index.js/4_emit.js`）

emit 方法先挑出函数列表 `evtArr`，并使用 arguments 类数组对象获取参数，最后依次调用

```js
E.prototype = {
  // ...

  emit: function (name) {
    var data = [].slice.call(arguments, 1);
    var evtArr = ((this.e || (this.e = {}))[name] || []).slice();
    var i = 0;
    var len = evtArr.length;

    for (i; i < len; i++) {
      evtArr[i].fn.apply(evtArr[i].ctx, data);
    }

    return this;
  },
  
  // ...
};
```

## 5. 单次触发事件 once

- `/index.js`（阅读笔记：`/index.js/5_once.js`）

单次触发的妙用在于，对原函数进行加工成调用后会 off 自己的函数，并将原函数放在 `_` 属性下，我们可以看到上面的 off 方法就有检查这个 `_` 属性

```js
E.prototype = {
  // ...

  once: function (name, callback, ctx) {
    var self = this;
    function listener () {
      self.off(name, listener);
      callback.apply(ctx, arguments);
    };

    listener._ = callback
    return this.on(name, listener, ctx);
  },
  
  // ...
};
```

# 其他资源

## 参考连接

| Title                             | Link                                                                                       |
| --------------------------------- | ------------------------------------------------------------------------------------------ |
| tiny-emitter - npm                | [https://www.npmjs.com/package/tiny-emitter](https://www.npmjs.com/package/tiny-emitter)   |
| scottcorgan/tiny-emitter - github | [https://github.com/scottcorgan/tiny-emitter](https://github.com/scottcorgan/tiny-emitter) |

## 阅读笔记参考

[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/tiny-emitter-2.1.0](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/tiny-emitter-2.1.0)
