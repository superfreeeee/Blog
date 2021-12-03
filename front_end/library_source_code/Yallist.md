# yallist 源码解析(Npm library)

@[TOC](文章目录)

<!-- TOC -->

- [yallist 源码解析(Npm library)](#yallist-源码解析npm-library)
- [正文](#正文)
  - [0. 基本信息](#0-基本信息)
  - [1. 源码解析](#1-源码解析)
    - [1.0 类型 & 接口](#10-类型--接口)
    - [1.1 类型定义](#11-类型定义)
    - [1.2 增：push、pushNode、unshift、unshiftNode](#12-增pushpushnodeunshiftunshiftnode)
    - [1.3 删：pop、shift、splice](#13-删popshiftsplice)
    - [1.4 查：get、getReverse、toArray、toArrayReverse、slice、sliceReverse、reverse](#14-查getgetreversetoarraytoarrayreversesliceslicereversereverse)
    - [1.5 函数式：forEach、forEachReverse、map、mapReverse、reduce、reduceReverse](#15-函数式foreachforeachreversemapmapreversereducereducereverse)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 正文

## 0. 基本信息

- version：`v4.0.0`
- 功能：双向链表实现

yallist 这个库是一个 Npm 上非常多产的大佬 [isaacs](https://www.npmjs.com/~isaacs) 写的一个双向链表的实现。

就一个双向链表也有千万下载量我也是醉了hhh，可能是发布的比较早作为其他库的基础依赖吧，我也要发一个千万下载量的包hh

## 1. 源码解析

### 1.0 类型 & 接口

首先我们先看这个双向链表暴露的类型与接口

| 类型    | 作用       |
| ------- | ---------- |
| Node    | 链表节点类 |
| Yallist | 链表类     |

| Yallist 方法    | 作用                |
| --------------- | ------------------- |
| push            | 从尾部添加多个数据  |
| pushNode        | 从尾部添加一个节点  |
| unshift         | 从头部添加多个数据  |
| unshiftNode     | 从头部添加一个节点  |
| pop             | 从尾部删除一个数据  |
| shift           | 从头部删除一个数据  |
| forEach         | 遍历                |
| forEachReverse  | 反向遍历            |
| get             | 获取第 n 个数据     |
| getReverse      | 获取倒数第 n 个数据 |
| map             | 遍历 & 返回新链表   |
| mapReverse      | 反向 map            |
| reduce          | 聚合链表            |
| reduceReverse   | 反向 reduce         |
| toArray         | 转换成一般数组      |
| toArrayReverse  | toArray 后反转链表  |
| slice           | 切片                |
| sliceReverse    | slice 后反转链表    |
| splice          | 切片替换            |
| reverse         | 反转链表            |
| Symbol.iterator | 链表迭代器实现      |

下面我们将这些链表接口分成四类介绍
1. 增
2. 删
3. 查
4. 函数式

### 1.1 类型定义

首先我们先看到两个类型定义

- `yallist.js`

第一个是 Node 节点定义

```js
function Node (value, prev, next, list) {
  // 直接调用 Node 方法
  if (!(this instanceof Node)) {
    return new Node(value, prev, next, list)
  }

  this.list = list
  this.value = value

  // prev <-> cur
  if (prev) {
    prev.next = this
    this.prev = prev
  } else {
    this.prev = null
  }

  // cur <-> next
  if (next) {
    next.prev = this
    this.next = next
  } else {
    this.next = null
  }
}
```

他会直接将新节点与参数链接到一起，第二个类型是链表对象，也就是 Node 构造函数参数的 list

```js
function Yallist (list) {
  var self = this
  if (!(self instanceof Yallist)) {
    self = new Yallist()
  }

  self.tail = null
  self.head = null
  self.length = 0

  // 利用参数初始化 Yallist 对象
  if (list && typeof list.forEach === 'function') {
    list.forEach(function (item) {
      self.push(item)
    })
  } else if (arguments.length > 0) {
    for (var i = 0, l = arguments.length; i < l; i++) {
      self.push(arguments[i])
    }
  }

  return self
}
```

有了构造函数之后我们就可以来看一个个方法接口

### 1.2 增：push、pushNode、unshift、unshiftNode

第一类型的增加相关的接口有四个方法

- push

push 方法调用内部的 push 将所有参数一个个推入链表

```js
Yallist.prototype.push = function () {
  for (var i = 0, l = arguments.length; i < l; i++) {
    push(this, arguments[i])
  }
  return this.length
}
```

内部 push 方法就是创建一个 Node 直接接到 tail 节点之后

```js
function push (self, item) {
  self.tail = new Node(item, self.tail, null, self)
  if (!self.head) {
    self.head = self.tail
  }
  self.length++
}
```

- pushNode

pushNode 方法与 push 类似，不过就不需要创建 Node 的阶段

```js
Yallist.prototype.pushNode = function (node) {
  if (node === this.tail) {
    return
  }

  if (node.list) {
    // 从旧列表删除
    node.list.removeNode(node)
  }

  // 更新 node
  var tail = this.tail
  node.list = this
  node.prev = tail

  // 更新 tail
  if (tail) {
    tail.next = node
  }

  this.tail = node
  // 更新 head
  if (!this.head) {
    this.head = node
  }
  this.length++
}
```

- unshift

unshift 与 push 类似，差别在于 unshift 是插入到链表头部

```js
Yallist.prototype.unshift = function () {
  for (var i = 0, l = arguments.length; i < l; i++) {
    unshift(this, arguments[i])
  }
  return this.length
}
```

```js
function unshift (self, item) {
  self.head = new Node(item, null, self.head, self)
  if (!self.tail) {
    self.tail = self.head
  }
  self.length++
}
```

- unshiftNode

unshiftNode 与 pushNode 类似，改成从头部插入节点

```js
Yallist.prototype.unshiftNode = function (node) {
  if (node === this.head) {
    return
  }

  // 从原列表移除
  if (node.list) {
    node.list.removeNode(node)
  }

  // 更新 node
  var head = this.head
  node.list = this
  node.next = head
  // 更新 head
  if (head) {
    head.prev = node
  }

  this.head = node
  // 更新 tail
  if (!this.tail) {
    this.tail = node
  }
  this.length++
}
```

### 1.3 删：pop、shift、splice

第二个类型是删除方法

- pop

第一个 pop 就是从尾部删除一个节点

```js
Yallist.prototype.pop = function () {
  // 长度为 0
  if (!this.tail) {
    return undefined
  }

  var res = this.tail.value
  // 更新 tail
  this.tail = this.tail.prev
  if (this.tail) {
    // 还有剩余节点
    this.tail.next = null
  } else {
    // 没有剩余节点了
    this.head = null
  }
  this.length--
  return res
}
```

- shift

shift 则是从头部删除一个节点

```js
Yallist.prototype.shift = function () {
  // 没有节点
  if (!this.head) {
    return undefined
  }

  // 更新 head
  var res = this.head.value
  this.head = this.head.next
  if (this.head) {
    // 还有剩余节点
    this.head.prev = null
  } else {
    // 没有剩余节点
    this.tail = null
  }
  this.length--
  return res
}
```

- splice

splice 比较特别，是先实现类似 slice 的方法，然后将剩余参数插入到原位置

```js
Yallist.prototype.splice = function (start, deleteCount, ...nodes) {
  if (start > this.length) {
    start = this.length - 1
  }
  if (start < 0) {
    start = this.length + start;
  }

  for (var i = 0, walker = this.head; walker !== null && i < start; i++) {
    walker = walker.next
  }

  var ret = []
  // 每次提出一个数据并删除节点
  for (var i = 0; walker && i < deleteCount; i++) {
    ret.push(walker.value)
    walker = this.removeNode(walker)
  }
  // 删除节点数量超过链表长度
  if (walker === null) {
    walker = this.tail
  }

  // 删除节点数量小于链表长度
  if (walker !== this.head && walker !== this.tail) {
    walker = walker.prev
  }

  // 替入新节点
  for (var i = 0; i < nodes.length; i++) {
    walker = insert(this, walker, nodes[i])
  }
  return ret;
}
```

### 1.4 查：get、getReverse、toArray、toArrayReverse、slice、sliceReverse、reverse

查找方法比较重复，我们忽略 xxxReverse 方法，只看正向的

- get

get 方法很简单，找到第 n 个节点然后返回其数据

```js
Yallist.prototype.get = function (n) {
  for (var i = 0, walker = this.head; walker !== null && i < n; i++) {
    // abort out of the list early if we hit a cycle
    walker = walker.next
  }
  if (i === n && walker !== null) {
    return walker.value
  }
}
```

- toArray

toArray 就是遍历后将链表转换成普通的数组对象

```js
Yallist.prototype.toArray = function () {
  var arr = new Array(this.length)
  for (var i = 0, walker = this.head; walker !== null; i++) {
    arr[i] = walker.value
    walker = walker.next
  }
  return arr
}
```

- slice

slice 则是提取指定区间的数据切片

```js
Yallist.prototype.slice = function (from, to) {
  to = to || this.length
  if (to < 0) {
    to += this.length
  }
  from = from || 0
  if (from < 0) {
    from += this.length
  }

  // ========== 参数校验（允许负数索引） ==========
  var ret = new Yallist()

  // from、to 范围对齐
  if (to < from || to < 0) {
    return ret
  }
  if (from < 0) {
    from = 0
  }
  if (to > this.length) {
    to = this.length
  }

  // 将 from ~ to 的节点数据放入新的链表并返回
  for (var i = 0, walker = this.head; walker !== null && i < from; i++) {
    walker = walker.next
  }
  for (; walker !== null && i < to; i++, walker = walker.next) {
    ret.push(walker.value)
  }
  return ret
}
```

### 1.5 函数式：forEach、forEachReverse、map、mapReverse、reduce、reduceReverse

最后一部分是函数式方法，主要介绍三个 forEach、map、reduce

- forEach

forEach 是进行内部遍历，并对每个数据调用回调函数

```js
Yallist.prototype.forEach = function (fn, thisp) {
  thisp = thisp || this
  for (var walker = this.head, i = 0; walker !== null; i++) {
    // Array.prototype.forEach((value, index, arr) => void)
    fn.call(thisp, walker.value, i, this)
    walker = walker.next
  }
}
```

- map

map 与 forEach 几乎一样，差别是保存回调函数的执行结果并返回一个新链表

```js
Yallist.prototype.map = function (fn, thisp) {
  thisp = thisp || this
  var res = new Yallist()
  for (var walker = this.head; walker !== null;) {
    res.push(fn.call(thisp, walker.value, this))
    walker = walker.next
  }
  return res
}
```

- reduce

reduce 则是聚合，一次取两个数据，并允许传入第二个参数作为初始值

```js
Yallist.prototype.reduce = function (fn, initial) {
  var acc
  var walker = this.head
  if (arguments.length > 1) {
    // 有初始值
    acc = initial
  } else if (this.head) {
    // 没有初始值，至少有一个数据
    walker = this.head.next
    acc = this.head.value
  } else {
    // 一个数据都没有时至少需要提供一个初始值
    throw new TypeError('Reduce of empty list with no initial value')
  }

  for (var i = 0; walker !== null; i++) {
    // reduceFn(prev, next, index)
    acc = fn(acc, walker.value, i)
    walker = walker.next
  }

  return acc
}
```

# 其他资源

## 参考连接

| Title                   | Link                                                                           |
| ----------------------- | ------------------------------------------------------------------------------ |
| yallist - npm           | [https://www.npmjs.com/package/yallist](https://www.npmjs.com/package/yallist) |
| isaacs/yallist - Github | [https://github.com/isaacs/yallist](https://github.com/isaacs/yallist)         |

## 阅读笔记参考

[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/yallist-4.0.0](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/yallist-4.0.0)
