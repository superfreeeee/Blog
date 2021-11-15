# delegate 源码解析

@[TOC](文章目录)

<!-- TOC -->

- [delegate 源码解析](#delegate-源码解析)
- [关联文章](#关联文章)
- [正文](#正文)
  - [0. 功能](#0-功能)
  - [1. 用法](#1-用法)
  - [2. 入口函数 delegate](#2-入口函数-delegate)
  - [3. 核心功能 _delegate](#3-核心功能-_delegate)
  - [4. 封装监听函数 listener](#4-封装监听函数-listener)
  - [5. 查找最近目标元素 closest](#5-查找最近目标元素-closest)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 关联文章

- [good-listener 源码解析](https://blog.csdn.net/weixin_44691608/article/details/121077328)

# 正文

## 0. 功能

之前写过的 [good-listener 源码解析](https://blog.csdn.net/weixin_44691608/article/details/121077328) 提过，good-listener 库本质上就是对 delegate 库的封装，本篇再向下深入探究 delegate 库的功能

## 1. 用法

基本用法可以参考 [README.md](https://www.npmjs.com/package/delegate)

- `/readme.md`（阅读笔记：`/src/delegate.js/delegate.d.ts`）

稍微整理一下可以看出 delegate 函数有两种重载形式

```ts
interface Listener {
  destroy(): void;
}

declare function delegate(): Listener;
declare function delegate(selector: string, type: string, callback: (e: Event) => void, useCapture: boolean): Listener;
declare function delegate(
  element: string | Element | Element[] | NodeList,
  selector: string,
  type: string,
  callback: (e: Event) => void,
  useCapture: boolean
): Listener;
```

## 2. 入口函数 delegate

- `/src/delegate.js`（阅读笔记：`/src/delegate.js/1_delegate.js`）

```js
function delegate(elements, selector, type, callback, useCapture) {
  // 监听元素对象
  if (typeof elements.addEventListener === 'function') {
      return _delegate.apply(null, arguments);
  }

  // 对应 4 参数形式
  if (typeof type === 'function') {
      return _delegate.bind(null, document).apply(null, arguments);
  }

  // 根据 CSS 选择器选择元素集合
  if (typeof elements === 'string') {
      elements = document.querySelectorAll(elements);
  }

  return Array.prototype.map.call(elements, function (element) {
      return _delegate(element, selector, type, callback, useCapture);
  });
}
```

入口函数 delegate 本身是对传入参数的校验和对齐，实际上的核心功能写在 _delegate 里面

## 3. 核心功能 _delegate

- `/src/delegate.js`（阅读笔记：`/src/delegate.js/2__delegate.js`）

```js
function _delegate(element, selector, type, callback, useCapture) {
  var listenerFn = listener.apply(this, arguments);

  element.addEventListener(type, listenerFn, useCapture);

  return {
    destroy: function () {
      element.removeEventListener(type, listenerFn, useCapture);
    },
  };
}
```

核心功能还是比较清晰，依赖于 `EventTarget.prototype.addEventListener` 方法，并返回一个用于销毁监听器的对象

## 4. 封装监听函数 listener

前面我们看到

```js
  var listenerFn = listener.apply(this, arguments);
```

使用 listener 函数对原始的 callback 回调进行封装

- `/src/delegate.js`（阅读笔记：`/src/delegate.js/3_listener.js`）

```js
function listener(element, selector, type, callback) {
  return function (e) {
    e.delegateTarget = closest(e.target, selector);

    if (e.delegateTarget) {
      callback.call(element, e);
    }
  };
}
```

listener 函数本质上就是做一件事：往 Event 对象(参数 `e`)添加一个 `delegateTarget` 属性

## 5. 查找最近目标元素 closest

而这个 `delegateTarget` 指向的元素，则是使用 `closest` 来寻找离事件触发目标最接近的目标选择器对象

也就是我们调用 `delegate` 时传入的 selector 选择器

- `/src/closest.js`（阅读笔记：`/src/closest.js`）

```js
var DOCUMENT_NODE_TYPE = 9;

/**
 * A polyfill for Element.matches()
 */
if (typeof Element !== 'undefined' && !Element.prototype.matches) {
  var proto = Element.prototype;

  proto.matches =
    proto.matchesSelector ||
    proto.mozMatchesSelector ||
    proto.msMatchesSelector ||
    proto.oMatchesSelector ||
    proto.webkitMatchesSelector;
}

function closest(element, selector) {
  while (element && element.nodeType !== DOCUMENT_NODE_TYPE) {
    if (typeof element.matches === 'function' && element.matches(selector)) {
      return element;
    }
    element = element.parentNode;
  }
}

module.exports = closest;
```

查找的过程就是从 element 触发，每次访问父节点 `element.parentNode`，使用 `matchesSelector` 来检验是否符合选择器条件

这里实际上有个遗留问题是，`matches` 本身已经作为 Element.prototype 的标准属性存在，所以兼容性上应该改为

```js
/**
 * A polyfill for Element.matches()
 */
if (typeof Element !== 'undefined' && !Element.prototype.matches) {
  var proto = Element.prototype;

  proto.matches =
    proto.matches ||   // 原始方法优先
    proto.matchesSelector ||
    proto.mozMatchesSelector ||
    proto.msMatchesSelector ||
    proto.oMatchesSelector ||
    proto.webkitMatchesSelector;
}
```

# 其他资源

## 参考连接

| Title                       | Link                                                                                                                                 |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| delegate - npm              | [https://www.npmjs.com/package/delegate](https://www.npmjs.com/package/delegate)                                                     |
| zenorocha/delegate - github | [https://github.com/zenorocha/delegate](https://github.com/zenorocha/delegate)                                                       |
| Element.matches() - MDN     | [https://developer.mozilla.org/zh-CN/docs/Web/API/Element/matches](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/matches) |

## 阅读笔记参考

[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/delegate-3.2.0](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/delegate-3.2.0)
