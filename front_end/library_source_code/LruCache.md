# lru-cache 源码解析(Npm library)

@[TOC](文章目录)

<!-- TOC -->

- [lru-cache 源码解析(Npm library)](#lru-cache-源码解析npm-library)
- [正文](#正文)
  - [0. 基本信息](#0-基本信息)
  - [1. 源码解析](#1-源码解析)
    - [1.1 LRUCache 结构 & 构造函数](#11-lrucache-结构--构造函数)
    - [1.2 set(key, value, maxAge)](#12-setkey-value-maxage)
    - [1.3 get(key)、peek(key)](#13-getkeypeekkey)
    - [1.4 del(key)](#14-delkey)
    - [1.5 forEach(fn(value, key, cache), \[thissp\])](#15-foreachfnvalue-key-cache-thissp)
    - [1.6 dump、load](#16-dumpload)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 正文

## 0. 基本信息

- version：`v6.0.0`
- 功能：LRU 缓存算法实现

## 1. 源码解析

### 1.1 LRUCache 结构 & 构造函数

首先先看到 LRUCache 的数据结构与构造函数

- `index.js`

yallist 是作者自己写的一个双向链表库

```js
'use strict';

// A linked list to keep track of recently-used-ness
const Yallist = require('yallist');
```

接下来作者先定义了几个属性 key，分别代表不同含义

```js
const MAX = Symbol('max'); //                             Cache 最大容量
const LENGTH = Symbol('length'); //                       Cache 当前容量
const LENGTH_CALCULATOR = Symbol('lengthCalculator'); //  Cache 数据长度计算函数
const ALLOW_STALE = Symbol('allowStale'); //              标志：允许 stale 缓存返回
const MAX_AGE = Symbol('maxAge'); //                      默认过期时间
const DISPOSE = Symbol('dispose'); //                     卸载函数回调
const NO_DISPOSE_ON_SET = Symbol('noDisposeOnSet'); //    标识：重复 set 时是否调用 dispose
const LRU_LIST = Symbol('lruList'); //                    Cache 数据链表
const CACHE = Symbol('cache'); //                         Cache 缓存 Map
const UPDATE_AGE_ON_GET = Symbol('updateAgeOnGet'); //    标志：get 时是否更新 now

const naiveLength = () => 1;
```

核心实现属性为下列几个：
1. `MAX` Cache 容量限制
2. `LENGTH` 当前占用容量
3. `MAX_AGE` 过期时效
4. `LRU_LIST` 双向列表
5. `CACHE` $key-value$ Map 映射表

```js
// lruList is a yallist where the head is the youngest
// item, and the tail is the oldest.  the list contains the Hit
// objects as the entries.
// Each Hit object has a reference to its Yallist.Node.  This
// never changes.
//
// cache is a Map (or PseudoMap) that matches the keys to
// the Yallist.Node object.
class LRUCache {
  /**
   * 构造函数：初始化各项属性 & 标志
   * 调用 reset 重置 Cache
   * @param {*} options
   */
  // ? Read
  constructor(options) {
    if (typeof options === 'number') options = { max: options };

    if (!options) options = {};

    if (options.max && (typeof options.max !== 'number' || options.max < 0))
      throw new TypeError('max must be a non-negative number');
    // Kind of weird to have a default max of Infinity, but oh well.
    const max = (this[MAX] = options.max || Infinity);

    const lc = options.length || naiveLength;
    this[LENGTH_CALCULATOR] = typeof lc !== 'function' ? naiveLength : lc;
    this[ALLOW_STALE] = options.stale || false;
    if (options.maxAge && typeof options.maxAge !== 'number')
      throw new TypeError('maxAge must be a number');
    this[MAX_AGE] = options.maxAge || 0;
    this[DISPOSE] = options.dispose;
    this[NO_DISPOSE_ON_SET] = options.noDisposeOnSet || false;
    this[UPDATE_AGE_ON_GET] = options.updateAgeOnGet || false;
    this.reset();
  }
```

### 1.2 set(key, value, maxAge)

第一个要介绍的方法是 set 方法，也就是加入一个缓存数据

- `index.js`

set 方法我们分成两种情况，第一种是 Cache 里面已经存在相同 key

```js
  /**
   * 缓存数据
   * @param {*} key
   * @param {*} value
   * @param {*} maxAge
   * @returns
   */
  // ? Read
  set(key, value, maxAge) {
    maxAge = maxAge || this[MAX_AGE]; // 使用默认 maxAge

    if (maxAge && typeof maxAge !== 'number')
      throw new TypeError('maxAge must be a number');

    const now = maxAge ? Date.now() : 0;
    const len = this[LENGTH_CALCULATOR](value, key);

    // 1. 数据 key 已存在
    if (this[CACHE].has(key)) {
      // 数据项超过总体积
      if (len > this[MAX]) {
        del(this, this[CACHE].get(key));
        return false;
      }

      const node = this[CACHE].get(key);
      const item = node.value;

      // dispose of the old one before overwriting
      // split out into 2 ifs for better coverage tracking
      // 重复 set 时调用 dispose
      if (this[DISPOSE]) {
        if (!this[NO_DISPOSE_ON_SET]) this[DISPOSE](key, item.value);
      }

      // 更新 item
      item.now = now;
      item.maxAge = maxAge;
      item.value = value;
      this[LENGTH] += len - item.length;
      item.length = len;

      // 访问节点 & 裁剪过量数据
      this.get(key);
      trim(this);
      return true;
    }
```

第二种是针对新数据，则同时往 Map、链表推入一个 Entry 作为数据（注意这里链表保存的是 `Node<Entry>`，而 Map 映射表保存的则是 `key => Node<Entry>`，主要是删除的时候可以直接根据 Map 的 Node 查找并删除链表中的数据）

```js
    // 2. 新数据
    const hit = new Entry(key, value, len, now, maxAge);

    // oversized objects fall out of cache automatically.
    // 超过限制大小 => 直接调用 dispose 并返回
    if (hit.length > this[MAX]) {
      if (this[DISPOSE]) this[DISPOSE](key, value);

      return false;
    }

    // 推入数据
    this[LENGTH] += hit.length;
    this[LRU_LIST].unshift(hit);
    this[CACHE].set(key, this[LRU_LIST].head);
    trim(this);
    return true;
  }
```

而这个 Entry 则是具有下面数据结构

```js
/**
 * 缓存节点
 */
// ? Read
class Entry {
  constructor(key, value, length, now, maxAge) {
    this.key = key; //            Cache 哈希键
    this.value = value; //        Cache 值
    this.length = length; //      数据大小
    this.now = now; //            缓存最后访问时间
    this.maxAge = maxAge || 0; // 缓存过期期限
  }
}
```

也就是 Cache 缓存相关的标志

### 1.3 get(key)、peek(key)

第二个 get 方法，与其相似的 peek 方法

- `index.js`

```js
  /**
   * get 时更新 now
   * @param {*} key
   * @returns
   */
  // ? Read
  get(key) {
    return get(this, key, true);
  }

  /**
   * peek 与 get 相同，但不更新 now
   * @param {*} key
   * @returns
   */
  // ? Read
  peek(key) {
    return get(this, key, false);
  }
```

本质上都是一个获取数据的手段，差别在于第三个参数，也就是是否更新访问时间

```js
/**
 * 访问缓存节点
 * @param {*} self
 * @param {*} key
 * @param {*} doUse
 * @returns
 */
// ? Read
const get = (self, key, doUse) => {
  // 缓存节点
  const node = self[CACHE].get(key);
  if (node) {
    // entry
    const hit = node.value;
    if (isStale(self, hit)) {
      // 节点过期则删除
      del(self, node);
      if (!self[ALLOW_STALE]) return undefined;
    } else {
      // 更新节点
      if (doUse) {
        if (self[UPDATE_AGE_ON_GET]) node.value.now = Date.now();
        self[LRU_LIST].unshiftNode(node);
      }
    }
    return hit.value;
  }
};
```

从上面的步骤我们可以看到，首先检查数据是否过期(`isStale`)，然后对于 `doUse` 更新 now 并重新推入队列

### 1.4 del(key)

删除比较简单，分别从 Map 与双向链表删除就可以了

- `index.js`

```js
  /**
   * 删除指定 key 数据
   * @param {*} key
   */
  // ? Read
  del(key) {
    del(this, this[CACHE].get(key));
  }
```

```js
/**
 * 删除指定节点
 * @param {*} self
 * @param {*} node
 */
// ? Read
const del = (self, node) => {
  if (node) {
    const hit = node.value;
    // 调用卸载回调
    if (self[DISPOSE]) self[DISPOSE](hit.key, hit.value);

    self[LENGTH] -= hit.length;
    self[CACHE].delete(hit.key); // 从 Map 移除
    self[LRU_LIST].removeNode(node); // 从 List 移除
  }
};
```

### 1.5 forEach(fn(value, key, cache), \[thissp\])

遍历的时候，则是按照双向链表保存数据的顺序进行访问（也就是按照访问时间的顺序，新到旧）

- `index.js`

```js
  /**
   * 遍历链表
   * @param {*} fn
   * @param {*} thisp
   */
  // ? Read
  forEach(fn, thisp) {
    thisp = thisp || this;
    for (let walker = this[LRU_LIST].head; walker !== null; ) {
      const next = walker.next;
      forEachStep(this, fn, walker, thisp);
      walker = next;
    }
  }
```

```js
/**
 * 遍历单步
 * @param {*} self
 * @param {*} fn
 * @param {*} node
 * @param {*} thisp
 */
// ? Read
const forEachStep = (self, fn, node, thisp) => {
  let hit = node.value;
  if (isStale(self, hit)) {
    // 过期了移除
    del(self, node);
    // ALLOW_STALE = true 时才 hit
    if (!self[ALLOW_STALE]) hit = undefined;
  }
  // fn(value, key, cache)
  if (hit) fn.call(thisp, hit.value, hit.key, self);
};
```

### 1.6 dump、load

最后一部分是对于缓存数据的序列化与反序列化

- `index.js`

序列化的部分，删掉过期的，然后其他映射为 `{ k, v, e }` 的数据结构，分别表示 `key, value, expireTime`

```js
  /**
   * 序列化
   * @returns
   */
  // ? Read
  dump() {
    return this[LRU_LIST].map((hit) =>
      isStale(this, hit)
        ? false
        : {
            // k = key, v = value, e = expire
            k: hit.key,
            v: hit.value,
            e: hit.now + (hit.maxAge || 0),
          }
    )
      .toArray()
      .filter((h) => h);
  }
```

反序列化就是将序列化后的数据重新还原成 Cache 对象，也就是一个个 push 就行了

```js
  /**
   * 反序列化
   * @param {*} arr
   */
  // ? Read
  load(arr) {
    // reset the cache
    this.reset();

    const now = Date.now();
    // A previous serialized cache has the most recent items first
    for (let l = arr.length - 1; l >= 0; l--) {
      const hit = arr[l];
      const expiresAt = hit.e || 0;
      if (expiresAt === 0)
        // 1. 无 maxAge
        // the item was created without expiration in a non aged cache
        this.set(hit.k, hit.v);
      else {
        // 2. 有 maxAge
        const maxAge = expiresAt - now;
        // dont add already expired items
        if (maxAge > 0) {
          this.set(hit.k, hit.v, maxAge);
        }
      }
    }
  }
```

# 其他资源

## 参考连接

| Title                            | Link                                                                                                                                                                     |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| lru-cache - npm                  | [https://www.npmjs.com/package/lru-cache](https://www.npmjs.com/package/lru-cache)                                                                                       |
| isaacs/node-lru-cache - Github   | [https://github.com/isaacs/node-lru-cache](https://github.com/isaacs/node-lru-cache)                                                                                     |
| Least recently used (LRU) - wiki | [https://en.wikipedia.org/wiki/Cache_replacement_policies#Least_recently_used_(LRU)](https://en.wikipedia.org/wiki/Cache_replacement_policies#Least_recently_used_(LRU)) |
| yallist 源码解析(Npm library)    | [https://blog.csdn.net/weixin_44691608/article/details/121695318](https://blog.csdn.net/weixin_44691608/article/details/121695318)                                       |

## 阅读笔记参考

[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/lru-cache-6.0.0](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/lru-cache-6.0.0)
