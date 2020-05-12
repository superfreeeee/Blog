# ES6: Symbol

@[TOC](文章目錄)

<!-- TOC -->

- [ES6: Symbol](#es6-symbol)
    - [簡介](#簡介)
    - [參考](#參考)
- [正文](#正文)
    - [Symbol 創建](#symbol-創建)
    - [Usage 使用](#usage-使用)
        - [作為屬性名](#作為屬性名)
        - [消除魔術字符串(Magic string)](#消除魔術字符串magic-string)
        - [非私有內部方法](#非私有內部方法)
    - [內置 Symbol 值](#內置-symbol-值)
        - [Overview](#overview)
        - [Symbol.hasInstance](#symbolhasinstance)
        - [Symbol.isConcatSpreadable](#symbolisconcatspreadable)
        - [Symbol.species](#symbolspecies)
        - [Symbol.match](#symbolmatch)
        - [Symbol.replace](#symbolreplace)
        - [Symbol.search](#symbolsearch)
        - [Symbol.split](#symbolsplit)
        - [Symbol.iterator](#symboliterator)
        - [Symbol.toPrimitive](#symboltoprimitive)
        - [Symbol.toStringTag](#symboltostringtag)
        - [Symbol.unscopables](#symbolunscopables)
- [結語](#結語)

<!-- /TOC -->

## 簡介

JS 的對象屬性名都是字符串，常常容易造成命名衝突，當我們寫好別人創造的對象時，可能會覆蓋掉其他人定義過的屬性。這時候 Symbol 登場了，作為 JS 第七種原生類型，每一個都擁有全場獨一無二的值，每次構造函數創建出來的值都是獨一無二了，同時能夠作為對象的屬性名，如此便能成功將對象的屬性區隔開來。

## 參考

<table>
    <tr>
        <td>ES6 symbol</td>
        <td><a href="http://caibaojian.com/es6/symbol.html">http://caibaojian.com/es6/symbol.html</a></td>
    <tr>
    </tr>
        <td>ES6 中的元编程</td>
        <td><a href="https://juejin.im/post/5a0e65c1f265da430702d6b9">https://juejin.im/post/5a0e65c1f265da430702d6b9</a></td>
    </>
</table>

# 正文

## Symbol 創建

Symbol 可以透過兩種方式創建：

1. `Symbol(string)`：構造函數創建，為全局獨一無二的值
2. `Symbol.for(string)`：創建 Symbol 的同時，向全局註冊一個字符串，字符串已經被註冊的時候就會引用同一個對象。使用 `Symbol.keyFor(symbol)` 查看構造 Symbol 對象時向全局註冊的值

```js
const s1 = Symbol()
s1 // Symbol()
typeof s1 // "symbol"
s1.toString() // "Symbol()"
s1 + '' // Cannot convert a Symbol value to a string

const s2 = Symbol()
s1 === s2 // false

const s3 = Symbol.for('1')
const s4 = Symbol.for('1')
s3 === s4 // true
Symbol.keyFor(s3) // "1"

const s5 = Symbol.for()
Symbol.keyFor(s5) // "undefined"
Symbol.keyFor(s1) // undefined
```

- 說明 1：s1、s2 為不同的對象，所以即使構造函數傳入的參數相同在 === 判斷下還是不同
- 說明 2：s3、s4 都使用了 `Symbol.for`，所以 s3 向全局註冊"1"之後，s4 創建時引用的是與 s3 相同的對象，所以比較相同
- 說明 3：s5 無參數的 `Symbol.for()` 向全局註冊的字符串為 "undefined"，而 s1 並未向全局註冊所以返回 undefined，兩者有區別。

## Usage 使用

### 作為屬性名

透過 ES6 擴展，對象屬性名可以使用方括號組合出屬性名，除了 string 之外 Symbol 也可以作為屬性名：

```js
const key = Symbol('key')
const func = Symbol('func')
const obj = {
  key: 'string key value',
  [key]: 'Symbol key value',
  [func]() {
    console.log('invoke function which name is a Symbol')
  }
}
obj // {key: "string key value", Symbol(key): "Symbol key value", Symbol(func): ƒ}
obj.key // string key value
obj[key] // Symbol key value
obj[func]() // invoke function which name is a Symbol
```

### 消除魔術字符串(Magic string)

我們模仿枚舉，檢查屬性名的時候可能會出現意想不到的重名，這時候就可以透過 Symbol 來避免這個問題：

```js
// bad
const shape = {
  triangle: 'triangle',
  square: 'square',
  rectangle: 'rectangle'
}
// good
const shape = {
  triangle: Symbol(),
  square: Symbol(),
  rectangle: Symbol()
}

function area(shape, options) {
  switch (shape) {
    case shape.triangle:
      return 0.5 * options.width * options.height
    case shape.square:
      return options.length * options.length
    case shape.rectangle:
      return options.width * options.height
  }
  return 0
}

area(shape.triangle, { width: 50, height: 100 })
// 2500
area(shape.square, { length: 50 })
// 2500
area(shape.rectangle, { width: 50, height: 100 })
// 5000
```

### 非私有內部方法

當然你也可以夠過 get/set 定義一個完全私有的方法，不過這邊的重點在於透過簡單的 Symbol 作為屬性可以對一些遍歷和獲取屬性名屏蔽：

```js
const top = Symbol()

class Stack {
  constructor() {
    this[top] = -1
  }
  pop() {
    const val = this[this[top]]
    delete this[this[top]--]
    return val
  }
  push(val) {
    this[++this[top]] = val
  }
}
stack.push(1) // Stack { '0': 1, [Symbol()]: 0 }
stack.push(2) // Stack { '0': 1, '1': 2, [Symbol()]: 1 }
stack.push(3) // Stack { '0': 1, '1': 2, '2': 3, [Symbol()]: 2 }

for (let key in stack) {
  console.log(`stack[${key}] = ${stack[key]}`)
}
// stack[0] = 1
// stack[1] = 2
// stack[2] = 3

stack.top = -1 // Stack { '0': 1, '1': 2, '2': 3, [Symbol()]: 2, 'top': -1 }
```

- 說明：`this[top]` 將作為棧頂指針，對 `for...in` 和 `Object.getOwnPropertyNames()` 都是透明的，模塊外只能透過 `Object.getOwnPropertySymbols()` 獲得 top 對象。並且再定義字符串作屬性名也沒有問題。

## 內置 Symbol 值

老實說前面的應用我個人認為在真正開發的時候其實都是可以避免的，或是最多就是多一層保險而已。然而接下來這部分我想才是 Symbol 最重要的應用：**內置屬性**。

我們都知道 JS 對象的接口方法、繼承方法等都是添加在原型上，然而如果原生添加了過多的屬性方法，都是透過字符串作為屬性名的話，似乎有點佔用空間並且擠壓到開發者的命名空間，Symbol 正是一個好的解決辦法！

### Overview

ES6 總共添加了 11 種內置 Symbol 值，有點像是 Symbol 類的靜態類型，取代字符串作為方法名，便能將這些特殊的方法與一般方法區隔開來，同時許多 JS 內置方法也將調用以 Symbol 值為名的方法。透過這些 Symbol 內置值，程序員開始能夠介入一些內置方法的操作而不用再透過多餘的方法調用。

| Values                    | About                      |
| ------------------------- | -------------------------- |
| Symbol.hasInstance        | instanceof 運算符          |
| Symbol.isConcatSpreadable | Array.prototype.concat()   |
| Symbol.species            |
| Symbol.match              | String.prototype.match()   |
| Symbol.replace            | String.prototype.replace() |
| Symbol.search             |
| Symbol.split              |
| Symbol.iterator           |
| Symbol.toPrimitive        |
| Symbol.toStringTag        |
| Symbol.unscopables        |

### Symbol.hasInstance

相當於置換 `instanceof` 運算符的行為，下列兩種調用等價：

```js
A instanceof B
// 等價於
B[Symbol.hasInstance](A)
```

因此我們可以這樣為我們的類定義：

```js
class B {
  [Symbol.hasInstance](other) {
    return typeof other === "object"
  }
}
'object' instanceof new B() // false
{} instanceof new B() // true
```

### Symbol.isConcatSpreadable

這個屬性對應的值是一個布林值，將決定對象被 `Array.prototype.concat()` 調用的時候是否被展開，默認為 `true`

```js
let arr1 = [3, 4]
;[1, 2].concat(arr1, 5)
// [ 1, 2, 3, 4, 5 ]

arr1[Symbol.isConcatSpreadable] = false
;[1, 2].concat(arr1, 5)
// [ 1, 2, [ 3, 4 ], 5 ]
```

### Symbol.species

`Symbol.species` 提供訂製構造函數的入口，當我們利用 `Array.prototype.map` 或其他內置方法從實例創建新對象的時候，內置方法就會調用 `this[Symbol.species]` 來構造新對象的類

應用場景通常是我們需要指定一個將會返回以新的自己的類的對象，例如 Promise 調用 then 方法之後會返回一個新的 Promise 等：

```js
class DelayPromise extends Promise {
  static get [Symbol.species]() {
    return Promise
  }
}
const p = new DelayPromise(() => {})
p instanceof DelayPromise // true

const p2 = p.then(() => {})
p2 instanceof DelayPromise // false
p2 instanceof Promise // true
```

### Symbol.match

當調用 `String.prototype.match()` 的時候，傳入參數存在 `Symbol.match` 的話就會以這個方法替代，即可以客製化 match 的行為

即以下兩個表達式等價：

```js
String.prototype.match(regexp)
// 等價於
regexp[Symbol.match](this)
```

```js
class Matcher {
  [Symbol.match](string) {
    return string.indexOf('e')
  }
}
'Hello world'.match(new Matcher()) // 1
```

### Symbol.replace

當調用 `String.prototype.replace()` 的時候，以 `Symbol.replace` 的方法來置換，即可以透過此方法客製化 replace 的行為，即下列表達式等價

```js
String.prototype.replace(searchVal, replaceVal)
// 等價於
searchVal[Symbol.replace](this, replaceVal)
```

```js
const replacement = {}
replacement[Symbol.replace] = (searchVal, replaceVal) => {
  let result = ''
  for (let c of searchVal) {
    if (c === 'e') {
      result += '-'
    } else {
      result += c
    }
  }
  return result
}
'Hello'.replace(replacement, '-') // H-llo
```

### Symbol.search

當調用 `String.prototype.search()` 的時候，以 `Symbol.search` 的方法來置換，即可以透過此方法客製化 search 的行為，即下列表達式等價

```js
String.prototype.search(regexp)
// 等價於
regexp[Symbol.search](this)
```

```js
class SearchEngine {
  constructor(target) {
    this.target = target
  }
  [Symbol.search](s) {
    return s.indexOf(this.target)
  }
}
'Hello World'.search(new SearchEngine('llo')) // 2
```

### Symbol.split

與 match, replace, search，就是用 `Symbol.split` 替代原本的 split（ES6 的團隊真的很喜歡字符串操作。。。

```js
String.prototype.split(separator, limit)
// 等價於
separator[Symbol.split](this, limit)
```

### Symbol.iterator

`Symbol.iterator` 為一個 Generator 函數，在使用 `for...of` 的時候依次調用這個函數，與 Generator 相關的細節請看另一篇：

```js
const obj = {}
obj[Symbol.iterator] = function* () {
  for (let i = 0; i < 11; i += 5) {
    yield i
  }
}
for (let val of obj) {
  console.log(val)
}
// output:
// 0
// 5
// 10
```

### Symbol.toPrimitive

當對象觸發隱式的類型轉換，如 `obj + 1` 或 `obj + '1'` 之類的動作，就會調用 `obj[Symbol.toPrimitive]` 方法，這個方法傳入一個狀態，狀態存在三種模式：

1. 'number'：被轉成數字的時候
2. 'string'：被轉成字符串的時候
3. 'default'：可以被轉成數字也可以被轉成字符串的時候

```js
const obj = {
  [Symbol.toPrimitive](mode) {
    switch (mode) {
      case 'number':
        return 100
      case 'string':
        return '***'
      case 'default':
        return '???'
      default:
        throw new Error('unexpected mode')
    }
  }
}
3 + obj // 3???
3 - obj // -97
3 * obj // 300
3 / obj // 0.03
'???' == obj // true
Number(obj) // 100
String(obj) // ***
```

### Symbol.toStringTag

一般來說如果直接調用 `Object.prototype.toStrign()` 的話應該會返回 `[object Object]` 或 `[object Array]`，而 `Symbol.toStringTag` 可以客製化這個方法（聽起來頗沒用）：

```js
const obj = {
  get [Symbol.toStringTag]() {
    return '???'
  }
}
obj.toString() // [object ???]
```

### Symbol.unscopables

`Symbol.unscopables` 指向一個對象，指定使用 `with` 關鍵字的時候會被忽略的屬性：

```js
class Obj {
  foo() {
    console.log('invoke foo in obj')
  }
  bar() {
    console.log('invoke bar in obj')
  }
  get [Symbol.unscopables]() {
    return {
      foo: true
    }
  }
}
function foo() {
  console.log('invoke foo out of obj')
}
function bar() {
  console.log('invoke bar out of obj')
}
with (Obj.prototype) {
  foo()
  bar()
}
```

with 的詳細說明詳見其他篇

# 結語

Symbol 看似不顯眼卻是相當前瞻的想法，考慮到未來將有更多的框架和庫都將提供一些定製化的對象，Symbol 能夠很好的避免使用者更動到原本提供的內部屬性。縱然外部的程序員依舊能夠透過 `Symbol.for` 或是 `Object.getOwnPropertySymbols`，但是只要使用者不作死，Symbol 已經能很大程度的屏蔽幾乎所有遍歷方法和捕捉對象屬性的方法。
