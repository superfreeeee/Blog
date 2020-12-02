# ES6 特性: Destructing Assignment 解構賦值

<!-- TOC -->

- [ES6 特性: Destructing Assignment 解構賦值](#es6-特性-destructing-assignment-解構賦值)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Array 數組的解構賦值](#array-數組的解構賦值)
    - [基本用法](#基本用法)
    - [rest 剩餘參數](#rest-剩餘參數)
    - [嵌套](#嵌套)
    - [默認值](#默認值)
    - [迭代器](#迭代器)
  - [Object 對象的解構賦值](#object-對象的解構賦值)
    - [前情提要：對象擴展](#前情提要對象擴展)
    - [基本用法](#基本用法-1)
    - [聲明後賦值](#聲明後賦值)
    - [嵌套](#嵌套-1)
    - [默認值](#默認值-1)
    - [從數組到對象](#從數組到對象)
  - [String 字符串的解構賦值](#string-字符串的解構賦值)
  - [Number & Boolean 數字和布林值的解構賦值](#number--boolean-數字和布林值的解構賦值)
    - [`null` & `undefined`](#null--undefined)
  - [Function Parameter 函數參數的解構賦值](#function-parameter-函數參數的解構賦值)
  - [應用](#應用)
    - [變量交換](#變量交換)
    - [函數返回多個值](#函數返回多個值)
    - [函數參數定義](#函數參數定義)
    - [JSON 提取數據](#json-提取數據)
    - [函數配置選項默認值](#函數配置選項默認值)
    - [遍歷 Map](#遍歷-map)
    - [導入模塊](#導入模塊)
- [結語](#結語)

<!-- /TOC -->

## 簡介

ES6 提供了許多語法糖，本篇介紹其中一個小但廣泛使用的特性：**解構賦值**。

## 參考

<table>
    <tr>
        <td>ES6 变量的解构赋值</td>
        <td><a href="http://caibaojian.com/es6/destructuring.html">http://caibaojian.com/es6/destructuring.html</a></td>
    </tr>
</table>

# 正文

解構賦值最重要的兩個部分是`Array數組`和`Object對象`的解構賦值。其他還有`String字符串`、`Number數字`、`Boolean布林`等解構賦值。

## Array 數組的解構賦值

### 基本用法

一言以敝之，就是變量排排站一個對一個，直接上代碼

```js
let array = [1, 2, 3, 4, 5]
let [a, b, c] = array
// a = 1, b = 2, c = 3
```

- 說明：a,b,c 對應右值的前三個元素，後面的元素忽略

### rest 剩餘參數

可使用剩餘參數捕捉不定長數組

```js
let [a, ...others] = [1, 2, 3, 4, 5]
// a = 1
// b = [ 2, 3, 4, 5 ]
```

### 嵌套

解構賦值還能夠嵌套使用，任意組合變量的層次結構（好吧我承認這有點多餘了反而增加了程序複雜度，不太推薦這樣搞）

```js
let [a, [b, c, ...d], , e, f] = [1, [2, 3, 4, 5, 6], 7, 8]
// a = 1
// b = 2
// c = 3
// d = [ 4, 5, 6 ]
// e = 8
```

- 說明：如果左邊變量省略，右值對應位置的變量值會被忽略；如果變量沒有對應值，默認賦值為 `undefined`

### 默認值

當你不確定右值的時候，為變量提供默認值是一個好辦法

```js
let [x = 0, y = 0] = [10]
// x = 10, y = 0
let [a = 1] = [undefined]
// a = 1
let [b = 1] = [null]
// b = null
function hi() {
  console.log('hello')
}
let [x = hi()] = [1]
let [y = hi()] = [undefined]
console.log(x)
console.log(y)
// hello
// 1
// undefined
let [x, y = x] = [1]
// x = 1, y = 1
```

- 說明 1：解構賦值使用 `===` 嚴格相等運算符比較，默認值只有當右值嚴格等於 `undefined` 的時候才會生效
- 說明 2：解構賦值是惰性求值的，也就是只有當右值嚴格等於 `undefined` 的時候才會開始計算默認值
- 說明 3：默認值可引用前面的變量

### 迭代器

除了一般的數組之外，其他任何具有迭代器的對象其實都可以使用數組解構賦值

```js
let [x, y, z] = new Set([1, 2, 2, 1, 6])
// x = 1, y = 2, z = 6
function* fibs() {
  let a = 0
  let b = 1
  while (true) {
    yield a
    ;[a, b] = [b, a + b]
  }
}
let [f0, f1, f2, f3] = fibs()
// f0 = 0, f1 = 1, f2 = 1, f3 = 2
```

## Object 對象的解構賦值

### 前情提要：對象擴展

ES6 對 Object 的寫法進行一定程度的簡化，上代碼

```js
let name = 'John'
let o = {
  name, // 等價於 name: name
  hi() {
    // 等價於 hi: function() {}
    console.log(this.name)
  }
}
o.hi()
```

### 基本用法

對象的解構賦值，便是找到相對應的 key，然後把值複製到值的位置

```js
let { foo, bar } = { foo: 'foo', bar: 'bar' }
// foo = 'foo', bar = 'bar'
let { foo: baz } = { foo: 'foo' }
// foo 未聲明
// baz = 'foo'
let { baz } = { foo: 'foo', bar: 'bar' }
// baz = undefined
```

### 聲明後賦值

有時候需要對變量重新賦值，但是對象解構依舊會重新聲明左值的變量，因此需要改變形式，上代碼

```js
let foo
let { foo } = { foo: 'foo' }
// SyntaxError: Identifier 'foo' has already been declared
let foo
{ foo } = { foo: 'foo' }
// SyntaxError: syntax error
let foo
;({ foo } = { foo: 'foo' })
// Success
```

- 注意：由於 js 語法解析，聲明後的解構賦值需要在外面加上一個圓括號他才會被解析成一個正常的表達式，否則會語法錯誤

### 嵌套

對象的解構賦值同樣也支持嵌套的形式（注意！一般的復值都是傳遞引用）

```js
let p = {
  id: 0,
  info: {
    name: 'john',
    age: 18,
    children: ['Lily', 'Bob', 'Andy']
  }
}
let {
  id,
  info: {
    name,
    age,
    children: [kid1, ...otherKids]
  }
} = p
// id = 0
// name = 'john'
// age = 18
// kid1 = 'Lily'
// otherKids = [ 'Bob', 'Andy' ]
let {
  foo: { baz }
} = { bar: 'bar' }
// TypeError: Cannot read property 'baz' of undefined
```

- 說明：`foo` 的鍵不存在，`baz` 無上下文報錯

到此，我們可以發現對象的解構賦值中，左值的對象的鍵代表的是一種模式，真正賦值的變量位於對象中值的位置

### 默認值

同樣的，對象解構賦值也支持默認值，不過形式相對於數組要稍微複雜一丟丟

```js
let { foo = 'uncatch', baz } = {}
// foo = 'uncatch', baz = undefined
let { foo: baz = 'default baz' } = {}
// foo 未聲明
// baz = 'default baz'
let { n = 1, u = 1, zero = 1 } = { n: null, u: undefined, zero: 0 }
// n = null
// u = 1
// zero = 0
```

- 說明：同樣的，對象的解構賦值也必須嚴格等於(===) `undefined` 默認值才會生效。

### 從數組到對象

對象的解構賦值還有一個很棒的特性，它會先將非對象類型轉換成類似對象類型，如數組就會轉化成類數組對象，以數字作為鍵、其他屬性如 `length`

```js
let arr = [1, 2, 3, 4, 5]
let { 0: first, [arr.length - 1]: last } = arr
// first = 1, last = 5
```

## String 字符串的解構賦值

右值為字符串時，將被解析成一個字符組成的數組

```js
let [a, b, c] = 'hello'
// a = 'h'
// b = 'e'
// c = 'l'
let { length: len, 0: firstChar } = 'hello'
// len = 5, firstChar = 'h'
```

## Number & Boolean 數字和布林值的解構賦值

數字和布林作為右值，會先被轉換成對象（基本上就是 Object.prototype 的原生屬性），所以基本上這個做什麼意義哈哈。

```js
let { toString: f1, valueOf: f2 } = 1
console.log(f1 === Object.prototype.toString)
// true
console.log(f2 === Object.prototype.valueOf)
// true
```

### `null` & `undefined`

`null` 和 `undefined` 不能被轉為對象，所以作為右值會報錯

```js
let { n } = null
// TypeError
let { u } = undefined
// TypeError
```

## Function Parameter 函數參數的解構賦值

有時候為了使用一個參數要從一個龐大的對象內挑出特定的屬性，其實是一個很麻煩的事情，我們想要直接傳入整個數據實體，讓方法自動找到他要的屬性，可以像下面這樣寫：

```js
function checkPassword({ password: p }) {
  // rule about p ...
  console.log(`password = ${p}`)
  return true
}
let person = {
  id: 0,
  name: 'John',
  password: '12345678',
  email: '181250003@smail.nju.edu.cn',
  addres: 'xxx'
  // ...
}
checkPassword(person)
// output: password = 12345678
;[
  [1, 2],
  [3, 4]
].map(([x, y]) => x + y)
// [ 3, 7 ]
function point({ x = 0, y = 0 } = {}) {
  return [x, y]
}
point() // [ 0, 0 ]
point({}) // [ 0, 0 ]
point({ x: 10 }) // [ 10, 0 ]
point({ x: 10, y: 20 }) // [ 10, 20 ]

function point2({ x, y } = { x: 0, y: 0 }) {
  return [x, y]
}
point2() // [ 0, 0 ]
point2({}) // [ undefined, undefined ]
point2({ x: 10 }) // [ 10, undefined ]
point2({ x: 10, y: 20 }) // [ 10, 20 ]
```

- 說明：倒數第二個示例中，調用 `point` 時未傳入參數，所以使用默認值 `{}`，再賦值給 `{ x: 0, y: 0 }`，所以 `x`、`y` 也具有默認值。可以與 `point2 函數做比較`

## 應用

### 變量交換

- 目標：交換 x, y 的值

```js
let x = 10
let y = 20
```

- 方法一：使用第三個變量 tmp

```js
let tmp = x
x = y
y = tmp
```

- 方法二：使用異或

```js
// X, Y 代表起始的 x, y 的值
x = x ^ y // x = X ^ Y, y = Y
y = x ^ y // x = X ^ Y, y = X
x = x ^ y // x = Y, y = X
```

- 方法三：ES6，兩個字，真香

```js
;[x, y] = [y, x]
```

### 函數返回多個值

函數返回多個值，python 最簡單直接返回，java 需要包裝成一個只讀的 Tuple 類以保證數據可靠性，ES6 的表現更接近 python，不過需要顯示包裝成一個數組

```js
function f1() {
  let a = 1,
    b = 2,
    c = 3
  return [a, b, c]
}
let [a, b, c] = f1()
// a = 1, b = 2, c = 3
function f2() {
  return {
    foo: 'foo',
    bar: 'bar',
    baz: 'baz'
  }
}
let { foo, bar } = f2()
// foo = 'foo', bar = 'bar'
```

### 函數參數定義

```js
function parse([cmd, ...params]) {
  console.log(`cmd = ${cmd}, params = ${params}`)
}
parse('add 1 2'.split(' '))
// cmd = add, params = 1,2
```

### JSON 提取數據

```js
let person = {
  id: 0,
  name: 'John',
  password: '12345678',
  email: '181250003@smail.nju.edu.cn',
  addres: 'xxx'
  // ...
}
let { id, name } = person
// id = 0
// name = 'John'
```

### 函數配置選項默認值

```js
function show({
  width = 50,
  height = 50,
  color = 'blue',
  fontSize = 30,
  margin
}) {
  console.log({
    width,
    height,
    color,
    fontSize,
    margin
  })
}
let square = {
  width: 100,
  height: 200
}
show(square)
// {
//   width: 100,
//   height: 200,
//   color: 'blue',
//   fontSize: 30,
//   margin: undefined
// }
```

### 遍歷 Map

可直接解開 Map 鍵值對

```js
let map = new Map()
map.set('key1', 'value1')
map.set('key2', 'value2')
for (let [key, value] of map) {
  console.log(`key = ${key}, value = ${value}`)
}
// key = key1, value = value1
// key = key2, value = value2
```

- 只取鍵或值

```js
for (let [key] of map) {
  console.log(`key = ${key}`)
}
// key = key1
// key = key2
for (let [, value] of map) {
  console.log(`value = ${value}`)
}
// value = value1
// value = value2
```

### 導入模塊

```js
import { mapGetters, mapMutations, mapActions } from 'vuex'
import { Table, Input } from 'ant-design'
```

# 結語

解構賦值基本上完全不影響原生 js 的執行邏輯，僅僅是簡化代碼的語法糖，可以說是無副作用又好用，熟練使用解構賦值能夠大大提升開發效率並且提高代碼可讀性。
