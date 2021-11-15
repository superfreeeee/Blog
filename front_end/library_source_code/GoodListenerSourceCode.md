# good-listener 源码解析

@[TOC](文章目录)

<!-- TOC -->

- [good-listener 源码解析](#good-listener-源码解析)
- [正文](#正文)
  - [0. 使用](#0-使用)
  - [1. 主流程](#1-主流程)
    - [1.1 辅助工具函数(类型判断)](#11-辅助工具函数类型判断)
    - [1.2 核心方法 listen](#12-核心方法-listen)
    - [1.3 监听元素 listenNode](#13-监听元素-listennode)
    - [1.4 监听元素集合 listenNodeList](#14-监听元素集合-listennodelist)
    - [1.5 根据选择器监听 listenSelector](#15-根据选择器监听-listenselector)
    - [1.6 监听器返回对象](#16-监听器返回对象)
  - [2. 总结](#2-总结)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 正文

## 0. 使用

good-listener 的使用方法主要就是一个 `listen` 核心函数，函数签名如下

```js
/**
 * Validates all params and calls the right
 * listener function based on its target type.
 *
 * @param {String|HTMLElement|HTMLCollection|NodeList} target
 * @param {String} type
 * @param {Function} callback
 * @return {Object}
 */
function listen(target, type, callback) {
```

细节可以去看 [README.md](https://www.npmjs.com/package/good-listener)

## 1. 主流程

本篇源码解析我们跟着核心函数的主流程往下走

### 1.1 辅助工具函数(类型判断)

开始之前先看看库内部自己定义的一些类型判断函数

- `/src/is.js`（阅读笔记：`/src/is.js`）

检查是否为元素节点

```js
exports.node = function(value) {
    return value !== undefined
        && value instanceof HTMLElement
        && value.nodeType === 1;
};
```

检查是否为元素节点集合

```js
exports.nodeList = function(value) {
    var type = Object.prototype.toString.call(value);

    return value !== undefined
        && (type === '[object NodeList]' || type === '[object HTMLCollection]')
        && ('length' in value)
        && (value.length === 0 || exports.node(value[0]));
};
```

字符串

```js
exports.string = function(value) {
    return typeof value === 'string'
        || value instanceof String;
};
```

函数

```js
exports.fn = function(value) {
    var type = Object.prototype.toString.call(value);

    return type === '[object Function]';
};
```

### 1.2 核心方法 listen

- `/src/listen.js`（阅读笔记：`/src/listen.js`）

```js
function listen(target, type, callback) {
    if (!target && !type && !callback) {
        throw new Error('Missing required arguments');
    }

    if (!is.string(type)) {
        throw new TypeError('Second argument must be a String');
    }

    if (!is.fn(callback)) {
        throw new TypeError('Third argument must be a Function');
    }

    if (is.node(target)) {
        return listenNode(target, type, callback);
    }
    else if (is.nodeList(target)) {
        return listenNodeList(target, type, callback);
    }
    else if (is.string(target)) {
        return listenSelector(target, type, callback);
    }
    else {
        throw new TypeError('First argument must be a String, HTMLElement, HTMLCollection, or NodeList');
    }
}
```

核心方法前半部属于参数校验，而后半部则根据 target 的类型委托给 `listenNode`、`listenNodeList`、`listenSelector` 实际的功能函数

### 1.3 监听元素 listenNode

- `/src/listen.js`（阅读笔记：`/src/listen.js`）

```js
function listenNode(node, type, callback) {
    node.addEventListener(type, callback);

    return {
        destroy: function() {
            node.removeEventListener(type, callback);
        }
    }
}
```

监听元素直接使用 `addEventListener` API

### 1.4 监听元素集合 listenNodeList

- `/src/listen.js`（阅读笔记：`/src/listen.js`）

```js
function listenNodeList(nodeList, type, callback) {
    Array.prototype.forEach.call(nodeList, function(node) {
        node.addEventListener(type, callback);
    });

    return {
        destroy: function() {
            Array.prototype.forEach.call(nodeList, function(node) {
                node.removeEventListener(type, callback);
            });
        }
    }
}
```

对于元素集合就是遍历并逐个监听

### 1.5 根据选择器监听 listenSelector

- `/src/listen.js`（阅读笔记：`/src/listen.js`）

```js
function listenSelector(selector, type, callback) {
    return delegate(document.body, selector, type, callback);
}
```

比较特别的是当 target 属于字符串，也就是 CSS 选择器的字符串的时候，则是利用 delegate 库来实现

### 1.6 监听器返回对象

总结前面 1.4、1.5、1.6 我们可以看出 listen 维护了一个函数签名，规定返回的对象必包含如

```js
{
    destroy() {}
}
```

的对象结构

## 2. 总结

本质上 good-listener 这个库是对于 delegate 库的一种封装，并额外支持了直接监听元素节点或元素节点集合，并额外添加了参数校验

# 其他资源

## 参考连接

| Title                            | Link                                                                                                                       |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| good-listener - MDN              | [https://www.npmjs.com/package/good-listener](https://www.npmjs.com/package/good-listener)                                 |
| zenorocha/good-listener - github | [https://github.com/zenorocha/good-listener](https://github.com/zenorocha/good-listener)                                   |
| SVGElement - SVG                 | [https://developer.mozilla.org/en-US/docs/Web/API/SVGElement](https://developer.mozilla.org/en-US/docs/Web/API/SVGElement) |

## 阅读笔记参考

[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/good-listener-1.2.2](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/good-listener-1.2.2)
