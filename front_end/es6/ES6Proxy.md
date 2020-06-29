# ES6: Proxy 代理

@[TOC](文章目錄)

<!-- TOC -->

- [ES6: Proxy 代理](#es6-proxy-代理)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Conception 思想](#conception-思想)
  - [Create 創建實例](#create-創建實例)
    - [`Proxy.revocable(target, handler)`](#proxyrevocabletarget-handler)
  - [Handlers 處理方法](#handlers-處理方法)
    - [`get(target, propKey, receiver)`](#gettarget-propkey-receiver)
      - [參數說明](#參數說明)
      - [Sample](#sample)
    - [`set(target, propKey, value, receiver)`](#settarget-propkey-value-receiver)
      - [參數說明](#參數說明-1)
      - [Sample](#sample-1)
    - [`has(target, propKey)`](#hastarget-propkey)
      - [參數說明](#參數說明-2)
      - [Sample](#sample-2)
    - [`deleteProperty(target, propKey)`](#deletepropertytarget-propkey)
      - [參數說明](#參數說明-3)
      - [Sample](#sample-3)
    - [`defineProperty(target, propKey, propDesc)`](#definepropertytarget-propkey-propdesc)
      - [參數說明](#參數說明-4)
      - [Sample](#sample-4)
    - [`ownKeys(target)`](#ownkeystarget)
      - [參數說明](#參數說明-5)
      - [Sample](#sample-5)
    - [`getOwnPropertyDescriptor(target, propKey)`](#getownpropertydescriptortarget-propkey)
      - [參數說明](#參數說明-6)
      - [Sample](#sample-6)
    - [`preventExtensions(target)`](#preventextensionstarget)
      - [參數說明](#參數說明-7)
      - [Sample](#sample-7)
    - [`getPrototypeOf(target)`](#getprototypeoftarget)
      - [參數說明](#參數說明-8)
      - [Sample](#sample-8)
    - [`setPrototypeOf(target, proto)`](#setprototypeoftarget-proto)
      - [參數說明](#參數說明-9)
      - [Sample](#sample-9)
    - [`isExtensible(target)`](#isextensibletarget)
      - [參數說明](#參數說明-10)
      - [Sample](#sample-10)
    - [`apply(target, context, args)`](#applytarget-context-args)
      - [參數說明](#參數說明-11)
      - [Sample](#sample-11)
    - [`construct(target, args)`](#constructtarget-args)
      - [參數說明](#參數說明-12)
      - [Sample](#sample-12)
- [結語](#結語)

<!-- /TOC -->

## 簡介

本篇將要介紹一個 ES6 非常重要的特徵，也是 JS 原生支持的`元編程(Metaprogramming)`的集大成：`Proxy (代理)`對象，透過`攔截(intercept)`一些默認方法或操作來實現代理，提供程序員介入默認行為的接口，有點像是 C++ 的運算符重載。通常 Proxy 還會與 `Relect(反射)` 連用，不過本篇將先專注於 Proxy 的方法接口，下一篇詳細解說 Reflect 的用法。

## 參考

<table>
  <tr>
    <td>Proxy 和 Reflect</td>
    <td><a href="http://caibaojian.com/es6/proxy.html">http://caibaojian.com/es6/proxy.html</a></td>
  </tr>
</table>

# 正文

## Conception 思想

首先我們先釐清一下 Proxy 的代理行為是如何運作的，在 Proxy 構造函數傳入相對應的`處理器(Handler)`，proxy 對象就會"攔截"這個操作，也就是說默認行為將會被改變，像是傳入 `get(){ return 0 }` 的結果是訪問所有屬性都返回 0。

## Create 創建實例

創建一個 Proxy 實例很簡單

```js
const target = {}
const handler = {}
const proxy = new Proxy(target, handler)
```

傳入`需要代理的對象 target (Object 或是 Array)`，以及`攔截處理方法 handler`

### `Proxy.revocable(target, handler)`

除了使用 `Proxy` 構造函數之外，還可以使用 `Proxy.revocable` 創造一個`可撤銷(revoke)`的代理對象，並且如果調用 `revoke` 方法後將不得再訪問代理對象：

```js
const target = {}
const handler = {}
const { proxy, revoke } = Proxy.revocable(target, handler)

console.log(proxy)
revoke()
console.log(proxy)
// proxy has been revoked
```

## Handlers 處理方法

接下來我們先列出 Proxy 對象可以代理的所有操作接口：

```js
const handler = {
  // 代理(攔截)方法：屬性訪問 obj.xxx
  get(target, propKey, receiver) {},
  // 代理方法：屬性賦職 obj.xxx = yyy
  set(target, propKey, value, receiver) {},
  // 代理方法：檢查屬性存在 hasProperty(in 操作符)
  has(target, propKey) {},
  // 代理方法：刪除屬性 delete
  deleteProperty(target, propKey) {},
  // 代理方法：Object.defineProperty
  defineProperty(target, propKey, propDesc) {},
  // 代理方法：Object.keys
  ownKeys(target) {},
  // 代理方法：Object.getOwnPropertyDescriptor
  getOwnPropertyDescriptor(target, propKey) {},
  // 代理方法：Object.preventExtensions
  preventExtensions(target) {},
  // 代理方法：Object.getPrototypeOf
  getPrototypeOf(target) {},
  // 代理方法：Object.setPrototypeOf
  setPrototypeOf(target, proto) {},
  // 代理方法：Object.isExtensible
  isExtensible(target) {},
  // 代理方法：函數調用、apply、call
  apply(target, context, args) {},
  // 代理方法：創建對象的 new 操作符
  construct(target, args) {}
}
```

接下來我們一個個來看各個攔截方法的定義方法的應用實例

### `get(target, propKey, receiver)`

`get` 為攔截`訪問對象屬性`時的方法

#### 參數說明

- `target`：代理目標，即原對象
- `propKey`：訪問的屬性名
- `receiver`：即代理對象本身

#### Sample

- 模仿默認行為

```js
const obj = { a: 0, b: 1, c: 2 }
const proxy = new Proxy(obj, {
  // 默認行為
  get(target, propKey, receiver) {
    return target[propKey]
  }
}

console.log(proxy.a)  // 0
```

- 限定訪問已存在屬性

```js
const obj = { a: 0, b: 1, c: 2 }
const proxy = new Proxy(obj, {
  // 訪問控制
  get(target, propKey, receiver) {
    if(propKey in target) {
      console.log(target === obj)
      console.log(receiver === proxy)
      return target[propKey]
    } else {
      throw new ReferenceError(`property '${propKey}' not exist`)
    }
  }
}

console.log(proxy.b)
// true
// true
// 1
console.log(proxy.d)
// ReferenceError: property 'd' not exist
```

- 實現負數索引

```js
const array = [5, 4, 3, 2, 1]
const proxy = new Proxy(array, {
  // 負數
  get(target, propKey, receiver) {
    let idx = Number(propKey)
    if(idx < 0) {
      idx += target.length
    }
    return target[idx]
  }
}

console.log(proxy[2])  // 3
console.log(proxy[-1])  // 1
```

### `set(target, propKey, value, receiver)`

`set` 為攔截`對象屬性值賦值`的方法

#### 參數說明

- `target`：代理目標，即原對象
- `propKey`：訪問的屬性名
- `value`：設置屬性值
- `receiver`：即代理對象本身

#### Sample

- 模擬默認行為

```js
const obj = { a: 0, b: 1, c: 2 }
const proxy = new Proxy(obj, {
  // 默認行為
  set(target, propKey, receiver) {
    target[propKey] = value
  }
}

proxy.a = 123
```

- 限定賦值條件

```js
const obj = { a: 0, b: 1, c: 2 }
const proxy = new Proxy(obj, {
  // 限定整數
  set(target, propKey, value, receiver) {
    if(Number.isInteger(value)) {
      console.log(`set ${propKey} with ${value}`)
      target[propKey] = value
    } else {
      console.log('should be integer')
    }
  }
}

proxy.a = 123  // set a with 123
proxy.c = 'abc'  // should be integer
```

- 限定賦值條件 2

```js
const assertPrivate = (key) => {
  if(key[0] === '_') {
    throw new ReferenceError(`can't set private property`)
  }
}
const obj = { a: 0, b: 1, c: 2 }
const proxy = new Proxy(obj, {
  // 拒絕設置內部屬性
  set(target, propKey, value, receiver) {
    assertPrivate(propKey)
    target[propKey] = value
  }
}

proxy.a = 123  // set a with 123
proxy._a = 456  // ReferenceError: can't set private property
```

### `has(target, propKey)`

`set` 為攔截`hasProperty`的方法，常見場景是使用 `in` 操作符(注意不是 `for...in`)

#### 參數說明

- `target`：代理目標，即原對象
- `propKey`：訪問的屬性名

#### Sample

- 默認行為

```js
const obj = { a: 0, b: 1, c: 2 }
const proxy = new Proxy(obj, {
  // 默認行為
  has(target, propKey) {
    return propKey in target
  }
}

console.log('a' in proxy)  // true
console.log('d' in proxy)  // false
```

- 隱藏私有變量

```js
const obj = { a: 0, b: 1, c: 2, _a: 3, _b: 4, _c: 5 }
const proxy = new Proxy(obj, {
  // 隱藏私有變量
  has(target, propKey) {
    if(propKey[0] === '_') {
      return false
    }
    return propKey in target
  }
}

console.log('a' in proxy)  // true
console.log('_a' in proxy)  // false
```

### `deleteProperty(target, propKey)`

`deleteProperty` 用於攔截 `delete` 操作符，阻止刪除應該返回 false 或拋出異常

#### 參數說明

- `target`：代理目標，即原對象
- `propKey`：訪問的屬性名

#### Sample

- 默認行為

```js
const obj = { a: 0, b: 1, c: 2, _a: 3, _b: 4, _c: 5 }
const proxy = new Proxy(obj, {
  // 默認行為
  deleteProperty(target, propKey) {
    return delete target[propKey]
  },
}

console.log(proxy)  // { a: 0, b: 1, c: 2, _a: 3, _b: 4, _c: 5 }
console.log(delete proxy.a)  // true
console.log(proxy)  // { b: 1, c: 2, _a: 3, _b: 4, _c: 5 }
```

- 阻止刪除私有變量

```js
const obj = { a: 0, b: 1, c: 2, _a: 3, _b: 4, _c: 5 }
const proxy = new Proxy(obj, {
  // 阻止刪除私有變量
  deleteProperty(target, propKey) {
    if(propKey[0] === '_') {
      throw new ReferenceError("can't delete private property")
    }
    return delete target[propKey]
  }
}

console.log(proxy)
// { a: 0, b: 1, c: 2, _a: 3, _b: 4, _c: 5 }
console.log(delete proxy.a)
// true
console.log(proxy)
// { b: 1, c: 2, _a: 3, _b: 4, _c: 5 }
console.log(delete proxy._a)
// ReferenceError: can't delete private property
```

### `defineProperty(target, propKey, propDesc)`

看名字就知道啦，這個方法攔截 `Object.defineProperty`，同時也會攔截到`賦值(assign)`

#### 參數說明

- `target`：代理目標，即原對象
- `propKey`：訪問的屬性名
- `propDesc`：屬性修飾符對象

#### Sample

- 默認行為

```js
const obj = { a: 0, b: 1, c: 2, _a: 3, _b: 4, _c: 5 }
const proxy = new Proxy(obj, {
  defineProperty(target, propKey, propDesc) {
    return Object.defineProperty(target, propKey, propDesc)
  }
}

console.log(proxy)  // { a: 0, b: 1, c: 2, _a: 3, _b: 4, _c: 5 }
proxy._a = 123
proxy._d = 456
console.log(proxy)  // { a: 0, b: 1, c: 2, _a: 123, _b: 4, _c: 5, _d: 456 }
```

- 返回 `false`

```js
"use strict"
const obj = { a: 0, b: 1, c: 2, _a: 3, _b: 4, _c: 5 }
const proxy = new Proxy(obj, {
  defineProperty(target, propKey, propDesc) {
    // 在嚴格模式之下返回 false 將拋出異常
    return false
  }
}

proxy.a = 123
// TypeError: 'defineProperty' on proxy: trap returned falsish for property 'a'
```

### `ownKeys(target)`

攔截 `Object.keys` 方法

#### 參數說明

- `target`：代理目標，即原對象

#### Sample

- 默認行為

```js
const obj = { a: 0, b: 1, c: 2, _a: 3, _b: 4, _c: 5 }
const proxy = new Proxy(obj, {
  // 默認行為
  ownKeys(target) {
    return Object.keys(target)
  }
}

console.log(Object.keys(proxy))  // [ 'a', 'b', 'c', '_a', '_b', '_c' ]
```

- 隱藏私有屬性

```js
const obj = { a: 0, b: 1, c: 2, _a: 3, _b: 4, _c: 5 }
const proxy = new Proxy(obj, {
  // 隱藏私有屬性
  ownKeys(target) {
    return Reflect.ownKeys(target).filter(key => key[0] !== '_')
  }
}

console.log(Object.keys(proxy))  // [ 'a', 'b', 'c' ]
```

### `getOwnPropertyDescriptor(target, propKey)`

攔截 `Object.getOwnPropertyDescriptor`，返回`屬性描述對象(Property Descriptor)` 或 `undefined`

#### 參數說明

- `target`：代理目標，即原對象
- `propKey`：訪問的屬性名

#### Sample

- 默認行為

```js
const obj = { a: 0, b: 1, c: 2, _a: 3, _b: 4, _c: 5 }
const proxy = new Proxy(obj, {
  // 默認行為
  getOwnPropertyDescriptor(target, propKey) {
    return Object.getOwnPropertyDescriptor(target, propKey)
  }
}

console.log(Object.getOwnPropertyDescriptor(proxy, 'a'))
// { value: 0, writable: true, enumerable: true, configurable: true }
```

- 限制私有屬性的訪問

```js
const obj = { a: 0, b: 1, c: 2, _a: 3, _b: 4, _c: 5 }
const proxy = new Proxy(obj, {
  // 限制私有屬性訪問
  getOwnPropertyDescriptor(target, propKey) {
    if(propKey[0] === '_') {
      return undefined
    }
    return Object.getOwnPropertyDescriptor(target, propKey)
  }
}

console.log(Object.getOwnPropertyDescriptor(proxy, 'a'))
// { value: 0, writable: true, enumerable: true, configurable: true }
console.log(Object.getOwnPropertyDescriptor(proxy, '_a'))
// undefined

```

### `preventExtensions(target)`

攔截 `Object.preventExtensions`，必須返回一個 Boolean 值。只有在 `Object.isExtensible(target)` 返回 `false`（也就是不可擴展的情況下），`preventExtensions` 方法才能返回 `true`， 所以需要在內部主動調用一次 `Object.preventExtensions`

#### 參數說明

- `target`：代理目標，即原對象

#### Sample

- 默認行為

```js
const obj = { a: 0, b: 1, c: 2, _a: 3, _b: 4, _c: 5 }
const proxy = new Proxy(obj, {
  // 默認行為
  preventExtensions(target) {
    Object.preventExtensions(target)
    return true
  }
}

Object.preventExtensions(proxy)  // 無返回值
```

- 可擴展時返回 `true`

```js
const obj = { a: 0, b: 1, c: 2, _a: 3, _b: 4, _c: 5 }
const proxy = new Proxy(obj, {
  // 可擴展時返回 true
  preventExtensions(target) {
    console.log(`target is extensible: ${Object.isExtensible(target)}`)
    return true
  }
}

Object.preventExtensions(proxy)
// TypeError: 'preventExtensions' on proxy: trap returned truish but the proxy target is extensible
```

### `getPrototypeOf(target)`

攔截 `Object.getPrototypeOf`、`instanceof 運算符`等

#### 參數說明

- `target`：代理目標，即原對象

#### Sample

- 默認行為

```js
const obj = { a: 0, b: 1, c: 2, _a: 3, _b: 4, _c: 5 }
const proxy = new Proxy(obj, {
  // 默認行為
  getPrototypeOf(target) {
    return Object.getPrototypeOf(target)
  }
}

console.log(Object.getPrototypeOf(proxy))
// {constructor: ƒ, __defineGetter__: ƒ, __defineSetter__: ƒ, hasOwnProperty: ƒ, __lookupGetter__: ƒ, …}
console.log(Object.getPrototypeOf(proxy) === Object.prototype)
// true
console.log(proxy instanceof Object)
// true
```

- 原型訪問控制

```js
const obj = { a: 0, b: 1, c: 2, _a: 3, _b: 4, _c: 5 }
const proxy = new Proxy(obj, {
  // 模擬繼承 Promise
  getPrototypeOf(target) {
    return Promise.prototype
  }
}

console.log(Object.getPrototypeOf(proxy))
// Promise {Symbol(Symbol.toStringTag): "Promise", constructor: ƒ, then: ƒ, catch: ƒ, finally: ƒ}
console.log(proxy instanceof Promise)
// true
```

### `setPrototypeOf(target, proto)`

與 `getPrototypeOf` 是一對的，攔截 `Object.setPrototypeOf`，用於設置原型對象，與創建對象時使用的 `Object.create(proto)` 有同樣的效果

#### 參數說明

- `target`：代理目標，即原對象
- `proto`：新原型對象

#### Sample

- 默認行為

```js
const obj = { a: 0, b: 1, c: 2, _a: 3, _b: 4, _c: 5 }
const proxy = new Proxy(obj, {
  // 默認行為
  setPrototypeOf(target, proto) {
    Object.setPrototypeOf(target, proto)
    return true
  }
}

Object.setPrototypeOf(proxy, {})
```

- 禁止改變原型對象

```js
const obj = { a: 0, b: 1, c: 2, _a: 3, _b: 4, _c: 5 }
const proxy = new Proxy(obj, {
  // 禁止修改原型
  setPrototypeOf(target, proto) {
    throw new ReferenceError("Change prototype is forbidden")
  }
}

Object.setPrototypeOf(proxy, {})
// ReferenceError: Change prototype is forbidden
```

### `isExtensible(target)`

攔截 `Object.isExtensible` 方法，同時`代理對象(proxy)`的`可擴展性(extensible)`必須與`原對象(target)`相同，否則會拋出類型錯誤

#### 參數說明

- `target`：代理目標，即原對象

#### Sample

- 默認行為

```js
const obj = { a: 0, b: 1, c: 2, _a: 3, _b: 4, _c: 5 }
const proxy = new Proxy(obj, {
  // 默認行為
  isExtensible(target) {
    return Object.isExtensible(target)
  }
}

console.log(Object.isExtensible(proxy))  // true
```

- `可擴展性(extensible)` 與原對象不同步

```js
const obj = { a: 0, b: 1, c: 2, _a: 3, _b: 4, _c: 5 }
const proxy = new Proxy(obj, {
  // 可擴展性不同步
  isExtensible(target) {
    return !Object.isExtensible(target)
  }
}

console.log(Object.isExtensible(proxy))
// TypeError: 'isExtensible' on proxy: trap result does not reflect extensibility of proxy target (which is 'true')
```

### `apply(target, context, args)`

這是 Proxy 中比較特別的方法之一，前面都是代理一般的對象(object)，這邊 `apply` 方法是攔截當原對象被作為一個函數調用時的情況，三種調用方法：直接調用、`Function.prototype.apply`、`Function.prototype.call`，都會被攔截

#### 參數說明

- `target`：代理目標，即原對象
- `context`：為函數調用上下文
- `args`：為調用傳入參數列表

#### Sample

- 默認行為

```js
const f = function() {
  console.log('hello')
}
const proxy = new Proxy(f, {
  // 默認行為
  apply(target, object, args) {
    return target.apply(object, args)
  }
}

proxy()  // hello
```

- `apply`、`call` 參數形式

```js
const f = function() {
  console.log('hello')
}
const proxy = new Proxy(f, {
  // 展示不同調用方法攔截後的參數形式
  apply(target, object, args) {
    console.log(object)
    console.log(args)
    return target.apply(object, args)
  }
}

proxy()
// undefined
// []
// hello

proxy(1, 2, 3)
// undefined
// [ 1, 2, 3 ]
// hello

proxy.apply({ use: 'apply' }, [4, 5, 6])
// { use: 'apply' }
// [ 4, 5, 6 ]
// hello

proxy.call({ use: 'call' }, 7, 8, 9)
// { use: 'call' }
// [ 7, 8, 9 ]
// hello
```

- 說明：可以看到直接調用的參數列表，以及使用 `call` 方法傳入的不定參數將被收集成一個數組，透過 `apply` 調用

### `construct(target, args)`

前面的 `apply` 是攔截當`代理目標(target)`作為函數被調用的情況，而 `construct` 則是攔截當代理目標作為`構造函數(constructor)`調用時的情況，也就是使用 `new` 操作符的情況

由於是代理 `new` 創建對象的行為，所以必須返回一個對象，否則報錯

#### 參數說明

- `target`：代理目標，即原對象
- `args`：參數列表

#### Sample

- 默認行為

```js
const Person = function (id=-1, name) {
  this.id = id
  this.name = name
}
const proxy = new Proxy(Person, {
  // 默認行為
  construct(target, args) {
    // 注意：args 為一個數組，需要使用 ... 展開
    return new target(...args)
  }
}

console.log(new proxy(1, 'person 1'))
// Person { id: 1, name: 'person 1' }
```

- 返回非對象值

```js
const Person = function (id=-1, name) {
  this.id = id
  this.name = name
}
const proxy = new Proxy(Person, {
  // 返回非對象
  construct(target, args) {
    return 1
  }
}

new proxy()
// TypeError: 'construct' on proxy: trap returned non-object ('1')
```

# 結語

到此就全部完結啦！撒花～本篇只介紹了基本的 Proxy 攔截行為與 handler 的對應關係，通常 Proxy 會與 Reflect 連用，來發揮完整的`元編程(Metaprogramming)`能力。下一篇我們將要來介紹 `Reflect 反射` 與 Proxy 結合後體現出來的強大功能。
