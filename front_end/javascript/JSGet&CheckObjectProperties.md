# JS 基础: Get & Check Object Properties 获取和检查对象属性

@[TOC](文章目录)

<!-- TOC -->

- [JS 基础: Get & Check Object Properties 获取和检查对象属性](#js-基础-get--check-object-properties-获取和检查对象属性)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [获取属性](#获取属性)
    - [`Object.keys(obj)`](#objectkeysobj)
    - [`Object.getOwnPropertyNames(obj)`](#objectgetownpropertynamesobj)
    - [`Object.getOwnPropertySymbols(obj)`](#objectgetownpropertysymbolsobj)
    - [`getOwnProperties` & `getOwnEnumerableProperties`](#getownproperties--getownenumerableproperties)
  - [检查属性存在](#检查属性存在)
    - [`Object.prototype.hasOwnProperty(prop)`](#objectprototypehasownpropertyprop)
    - [`in` 关键字](#in-关键字)
    - [`hasInheritProperty`](#hasinheritproperty)
- [结语](#结语)

<!-- /TOC -->

## 简介

JS 中的对象可说是千变万化，对象的属性可能随时都在改变，那么我们要如何动态的获取对象的所有属性呢？

本篇就将从 Object 的各种静态方法来切入，透过使用 `Object.keys`、`Object.getOwnPropertyNames`、`Object.getOwnPropertySymbols`、`Object.prototype.hasOwnProperty`、`in` 等方法和关键字来获取和检查对象的属性。

## 参考

<table>
  <tr>
    <td>Object.keys()-MDN</td>
    <td><a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/keys">https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/keys</a></td>
  </tr>
  <tr>
    <td>Object.getOwnPropertyNames()-MDN</td>
    <td><a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyNames">https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyNames</a></td>
  </tr>
  <tr>
    <td>Object.getOwnPropertySymbols()-MDN</td>
    <td><a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertySymbols">https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertySymbols</a></td>
  </tr>
  <tr>
    <td>Object.prototype.hasOwnProperty()-MDN</td>
    <td><a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty">https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty</a></td>
  </tr>
  <tr>
    <td>in-MDN</td>
    <td><a href="https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/in">https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/in</a></td>
  </tr>
</table>

# 正文

正文部分分成两个模块，一个是获取对象的属性，一个是检查对象是否拥有属性（自己的属性或是继承的属性）

## 获取属性

接下来我们要透过下列几个方法来获取对象的属性：

- `Object.keys(obj)`
- `Object.getOwnPropertyNames(obj)`
- `Object.getOwnPropertySymbols(obj)`

### `Object.keys(obj)`

第一个方法是 `Object.keys(obj)`，它可以获取所有自有且 `enumerable: true` 的属性，示例如下：

```js
const object = { a: 1, b: 2, c: 3 }
console.log(Object.keys(object))
Object.defineProperty(object, 'c', { enumerable: false })
console.log(Object.keys(object))
```

```
[ 'a', 'b', 'c' ]
[ 'a', 'b' ]
```

我们可以看到，c 被设置成 `enumerable: false` 之后就不会再被 `Object.keys` 所发现

### `Object.getOwnPropertyNames(obj)`

第二个方法 `Object.getOwnPropertyNames(obj)` 跟 `Object.keys(obj)` 几乎一样，唯一不同的是 `enumerable: false` 的属性也会被找到，接着上面的例子：

```js
const object = { a: 1, b: 2, c: 3 }
console.log(Object.getOwnPropertyNames(object))
Object.defineProperty(object, 'c', { enumerable: false })
console.log(Object.getOwnPropertyNames(object))
```

```
[ 'a', 'b', 'c' ]
[ 'a', 'b', 'c' ]
```

看到了即使 c 被设为 `enumerable: false`，还是被 `Object.getOwnPropertyNames` 找出来了

### `Object.getOwnPropertySymbols(obj)`

现在还存在一个问题是，ES6 新增了一个基础类型 `Symbol`，同时它是除了字符串(String)外能够作为键的唯一选择，使用如下：

```js
const symbolA = Symbol('a')
const object = { a: 1, b: 2, c: 3, [symbolA]: 4 }
console.log(object)
console.log(Object.getOwnPropertyNames(object))
```

```
{ a: 1, b: 2, c: 3, [Symbol(a)]: 4 }
[ 'a', 'b', 'c' ]
```

好像有点问题，`Object.getOwnPropertyNames` 找不到以 Symbol 类型的键，这时候即必须使用 `Object.getOwnPropertySymbols`；并且与 `Object.getOwnPropertyNames` 一样，即使属性 `enumerable: true` 也会被找出来

```js
const symbolA = Symbol('a')
const symbolB = Symbol('b')
const object = { a: 1, b: 2, c: 3, [symbolA]: 4, [symbolB]: 5 }
Object.defineProperty(object, 'c', { enumerable: false })
Object.defineProperty(object, symbolB, { enumerable: false })
console.log(object)
console.log(Object.keys(object))
console.log(Object.getOwnPropertyNames(object))
console.log(Object.getOwnPropertySymbols(object))
```

```
{ a: 1, b: 2, [Symbol(a)]: 4 }
[ 'a', 'b' ]
[ 'a', 'b', 'c' ]
[ Symbol(a), Symbol(b) ]
```

### `getOwnProperties` & `getOwnEnumerableProperties`

根据上述的方法，我们可以抽象出两个个获得所有属性名的方法，一个`获取所有自有属性(getOwnProperties)`；一个`获取所有可枚举(enumerable: false)的自有属性(getOwnEnumerableProperties)`：

```js
// 获取所有属性
function getOwnProperties (obj) {
  const keys = Object.getOwnPropertyNames(obj)
  const symbols = Object.getOwnPropertySymbols(obj)
  return [...keys, ...symbols]
}

// 获取所有可枚举自有属性
function getOwnEnumerableProperties (obj) {
  const props = getOwnProperties(obj)
  const enumerableProps = props.filter(prop => Object.getOwnPropertyDescriptor(obj, prop).enumerable)
  return enumerableProps
}

const symbolA = Symbol('a')
const symbolB = Symbol('b')
const object = { a: 1, b: 2, c: 3, [symbolA]: 4, [symbolB]: 5 }
Object.defineProperty(object, 'c', { enumerable: false })
Object.defineProperty(object, symbolB, { enumerable: false })
console.log(getOwnProperties(object))
console.log(getOwnEnumerableProperties(object))
```

```
[ 'a', 'b', 'c', Symbol(a), Symbol(b) ]
[ 'a', 'b', Symbol(a) ]
```

## 检查属性存在

检查属性的方法有两种：

- `Object.prototype.hasOwnProperty`：检查对象自有属性是否包含指定属性
- `in` 关键字：检查原型链上是否包含指定属性

### `Object.prototype.hasOwnProperty(prop)`

```js
const nums = [ -1, -2, -3 ]
console.log('----- check nums -----')
console.log(getOwnProperties(nums))
console.log(`nums.hasOwnProperty(0): ${nums.hasOwnProperty(0)}`)
console.log(`nums.hasOwnProperty(1): ${nums.hasOwnProperty(1)}`)
console.log(`nums.hasOwnProperty(2): ${nums.hasOwnProperty(2)}`)
console.log(`nums.hasOwnProperty(3): ${nums.hasOwnProperty(3)}`)
console.log(`nums.hasOwnProperty(-1): ${nums.hasOwnProperty(-1)}`)
console.log(`nums.hasOwnProperty('length'): ${nums.hasOwnProperty('length')}`)
console.log(`nums.hasOwnProperty('toString'): ${nums.hasOwnProperty('toString')}`)

const sa = Symbol('a')
const sb = Symbol('b')
const object = { a: 1, b: 2, [sa]: 3, [sb]: 4 }
Object.defineProperty(object, 'b', { enumerable: false })
Object.defineProperty(object, sb, { enumerable: false })
console.log('----- check object -----')
console.log(getOwnProperties(object))
console.log(getOwnEnumerableProperties(object))
console.log(`object.hasOwnProperty('a'): ${object.hasOwnProperty('a')}`)
console.log(`object.hasOwnProperty('b'): ${object.hasOwnProperty('b')}`)
console.log(`object.hasOwnProperty(sa): ${object.hasOwnProperty(sa)}`)
console.log(`object.hasOwnProperty(sb): ${object.hasOwnProperty(sb)}`)
console.log(`object.hasOwnProperty('toString'): ${object.hasOwnProperty('toString')}`)
```

```
----- check nums -----
[ '0', '1', '2', 'length' ]
nums.hasOwnProperty(0): true
nums.hasOwnProperty(1): true
nums.hasOwnProperty(2): true
nums.hasOwnProperty(3): false
nums.hasOwnProperty(-1): false
nums.hasOwnProperty('length'): true
nums.hasOwnProperty('toString'): false
----- check object -----
[ 'a', 'b', Symbol(a), Symbol(b) ]
[ 'a', Symbol(a) ]
object.hasOwnProperty('a'): true
object.hasOwnProperty('b'): true
object.hasOwnProperty(sa): true
object.hasOwnProperty(sb): true
object.hasOwnProperty('toString'): false
```

我们可以看到不管 `enumerable` 的描述符是 true 还是 false，只要是自有属性 `hasOwnProperty` 都会返回 true；

对于数组来说的`属性名是下标`而不是属性值；

`toString` 等从原型继承过来的属性则为 false。

### `in` 关键字

```js
const nums = [ -1, -2, -3 ]
console.log('----- check nums -----')
console.log(getOwnProperties(nums))
console.log(`0 in nums: ${0 in nums}`)
console.log(`1 in nums: ${1 in nums}`)
console.log(`2 in nums: ${2 in nums}`)
console.log(`3 in nums: ${3 in nums}`)
console.log(`-1 in nums: ${-1 in nums}`)
console.log(`'length' in nums: ${'length' in nums}`)
console.log(`'toString' in nums: ${'toString' in nums}`)

const sa = Symbol('a')
const sb = Symbol('b')
const object = { a: 1, b: 2, [sa]: 3, [sb]: 4 }
Object.defineProperty(object, 'b', { enumerable: false })
Object.defineProperty(object, sb, { enumerable: false })
console.log('----- check object -----')
console.log(getOwnProperties(object))
console.log(getOwnEnumerableProperties(object))
console.log(`'a' in object: ${'a' in object}`)
console.log(`'b' in object: ${'b' in object}`)
console.log(`sa in object: ${sa in object}`)
console.log(`sb in object: ${sb in object}`)
console.log(`'toString' in object: ${'toString' in object}`)
```

```
----- check nums -----
[ '0', '1', '2', 'length' ]
0 in nums: true
1 in nums: true
2 in nums: true
3 in nums: false
-1 in nums: false
'length' in nums: true
'toString' in nums: true
----- check object -----
[ 'a', 'b', Symbol(a), Symbol(b) ]
[ 'a', Symbol(a) ]
'a' in object: true
'b' in object: true
sa in object: true
sb in object: true
'toString' in object: true
```

与 `Object.prototype.hasOwnProperty` 不同的是从原型链上继承下来的属性（如 `toString`），在 `in` 操作符检查之下结果也是 true

### `hasInheritProperty`

基于 `Object.prototype.hasOwnProperty`、`in` 两种判断方式，我们就可以写出一个`检查是否为继承属性(hasInheritProperty)`的方法

```js
function hasInheritProperty (obj, prop) {
  return prop in obj && !obj.hasOwnProperty(prop)
}

const sa = Symbol('a')
const object = { a: 1, [sa]: 2 }
console.log(hasInheritProperty(object, 'a'))
console.log(hasInheritProperty(object, sa))
console.log(hasInheritProperty(object, 'toString'))
```

```js
false
false
true
```

# 结语

本篇介绍了如何在 JS 中获取对象属性，以及检查对象是否包含属性（自有或继承的），最后自己实现了三个新的方法 `getOwnProperties 获取所有自有属性`、`getOwnEnumerableProperties 获取所有可枚举自有属性`、`hasInheritProperty 检查是否为继承属性`，供大家参考。
