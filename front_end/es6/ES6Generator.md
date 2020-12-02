# ES6 特性: Generator 生成器

@[TOC](文章目錄)

<!-- TOC -->

- [ES6 特性: Generator 生成器](#es6-特性-generator-生成器)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Basic 基礎定義和用法](#basic-基礎定義和用法)
    - [帶參數的 next](#帶參數的-next)
    - [遍歷自定義對象屬性](#遍歷自定義對象屬性)
    - [擴展運算符展開迭代器](#擴展運算符展開迭代器)
    - [作為對象屬性](#作為對象屬性)
    - [yield\* 語法](#yield-語法)
      - [協程(coroutine)](#協程coroutine)
      - [半協程(semi-coroutine)](#半協程semi-coroutine)
    - [this 綁定](#this-綁定)
  - [Generator 實例方法](#generator-實例方法)
    - [Generator.prototype.throw()](#generatorprototypethrow)
    - [Generator.prototype.return()](#generatorprototypereturn)
      - [finally](#finally)
  - [Generator 應用](#generator-應用)
    - [異步操作同步化](#異步操作同步化)
    - [控制流程管理](#控制流程管理)
    - [Iterator 接口](#iterator-接口)
- [結語](#結語)

<!-- /TOC -->

## 簡介

ES6 提供的 Generator 函數是一種異步編程的解決方案，和多線程不同的是，由於 JS 為單線程，所以異步編程的實現更像是程序之間控制權的轉移，也就是 Generator 函數可以在指定步驟打斷點暫停(yield)交出控制權，等待下次調用(next)後繼續執行。

Generator 函數將異步操作以更簡潔的形式提供給使用者，yield 可以放在幾乎任何位置，但是實際上使用時流程控制卻沒那麼方便，多數需要透過額外的自動化流程管理封裝 Generator 函數。

- 應用

作為迭代器(Iterator)的實現方案，Generator 函數可以作為一個生成序列的迭代器之外，還可以作為狀態機使用。

## 參考

<table>
  <tr>
    <td>Generator 函数</td>
    <td><a href="http://caibaojian.com/es6/generator.html">http://caibaojian.com/es6/generator.html</a></td>
  </tr>
</table>

# 正文

## Basic 基礎定義和用法

Generator 函數不可以是箭頭函數而必須是一般的函數(透過 function 聲明)，並在 function 後接一個 `*` 符號。函數內使用 `yield` 作為中斷點，Generator 的主程序都是惰性求值的，只有當調用 `next` 才會執行到下一個 `yield`，並返回一個對象：

```js
function* gen() {
  yield 'first next'
  yield 'second next'
  yield 'third next'
  return 'end'
}

const g = gen()
g.next() // { value: 'first next', done: false }
g.next() // { value: 'second next', done: false }
g.next() // { value: 'third next', done: false }
g.next() // { value: 'end', done: true }
g.next() // { value: undefined, done: true }
```

`next` 方法返回的對象只有兩種屬性：

1. `value` 為 yield 後面的表達式返回的值
2. `done` 表示 Generator 函數是否結束

因此我們可以像下列這樣使用，這也幾乎就是 `let...of` 內置方法的調用形式：

```js
const g = gen()
let res
while (!(res = g.next()).done) {
  console.log(res.value)
}
// first next
// second next
// third next

for (let value of gen()) {
  console.log(value)
}
// first next
// second next
// third next
```

### 帶參數的 next

如果調用 `next` 方法的時候傳入參數，將作為上次 yield 表達是的返回值：

```js
function* gen() {
  let val = 0
  while (true) {
    let next = yield val
    if (next) {
      val = next
    }
  }
}

const g = gen()
g.next() // { value: 0, done: false }
g.next(1) // { value: 1, done: false }
g.next() // { value: 1, done: false }
g.next(3) // { value: 3, done: false }
```

- 說明：第二次跟第四次調用的時候傳入參數，所以 next 返回的值將賦給 val 從而改變下次 while 循環返回的 val。

### 遍歷自定義對象屬性

```js
function* objectEntrys() {
  let keys = Object.keys(this)
  for (let key of keys) {
    yield [key, obj[key]]
  }
}
const obj = { id: 0, name: 'John', age: 18 }
obj[Symbol.iterator] = objectEntrys
for (let [key, value] of obj) {
  console.log(`${key}:${value}`)
}
```

### 擴展運算符展開迭代器

```js
function* range(...params) {
  if (params.length == 0) {
    return
  }
  if (params.length == 1) {
    params.unshift(0)
  }
  if (params.length == 2) {
    params.push(1)
  }
  let [start, end, step] = params
  while (start < end) {
    yield start
    start += step
  }
}
console.log([...range(10)])
// [0,1,2,3,4,5,6,7,8,9]
console.log([...range(2, 10)])
// [2,3,4,5,6,7,8,9]
console.log([...range(2, 10, 2)])
// [2,4,6,8]
```

### 作為對象屬性

Generator 函數作為對象的屬性並使用對象擴展的時候有特別的寫法，以下兩種寫法等價：

```js
let obj = {
  gen: function* () {}
}
// 等價於
let obj = {
  *gen() {}
}
```

也就是對象的默認迭代器需要這樣寫：

```js
let nums = {
  *[Symbol.iterator]() {
    yield 1
    yield 2
    yield 3
  }
}
for (let num of nums) {
  console.log(num)
}
// 1
// 2
// 3
```

### yield\* 語法

如果想要在 Generator 函數裡面調用另一個 Generator 函數， 默認情況下是無效的：

```js
function* foo() {
  yield console.log('foo 1')
  yield console.log('foo 2')
  yield console.log('foo 3')
}

function* bar() {
  yield console.log('bar 1')
  foo()
  yield console.log('bar 2')
  yield console.log('bar 3')
}

let b = bar()
b.next() // bar 1
b.next() // bar 2
b.next() // bar 3
```

需要加上 `yield*` 關鍵字：

```js
function* foo() {
  yield console.log('foo 1')
  yield console.log('foo 2')
  yield console.log('foo 3')
}

function* bar() {
  yield console.log('bar 1')
  yield* foo()
  yield console.log('bar 2')
  yield console.log('bar 3')
}

let b = bar()
b.next() // bar 1
b.next() // foo 1
b.next() // foo 2
```

相當於 `foo` 函數的內容被展開到 `bar` 函數內部，如下形式：

```js
function* bar() {
  yield console.log('bar 1')
  // replace `yield* foo()`
  {
    yield console.log('foo 1')
    yield console.log('foo 2')
    yield console.log('foo 3')
  }
  yield console.log('bar 2')
  yield console.log('bar 3')
}
```

#### 協程(coroutine)

這邊就不得不提到協程的概念，協程作為一種程序運行的方式，可以運行在單線程的情況下也能運行在多線程的情況下。與協程相近的概念有兩個：`子例程(subroutine)`和`線程(thread)`

- 子例程 vs 協程

在單線程的情況下，子例程可以看作是一種特殊的協程，他們都運用到程序段之間控制權的轉移；而不同的是子例程遵守堆棧式的"後進先出"的方式運行，只有當子例程執行完畢之後控制權才會回到調用者手上，而協程卻能夠由使用者自由定義控制權的轉換。並且相對於子例程和調用者共存在一個堆棧上，協程同時維護多個堆棧，以內存為代價實現多任務並行。

- 線程 vs 協程

在多線程的情況下，線程的表現與協程相似，都在執行過程中進行控制權的轉移實現多任務併發，同時每個任務都有自己上下文(context，也就是堆棧)；但不同的是普通的線程是搶先式的，如 Java 採用分時技術，不同線程調用的順序和時間是不能保證的，然而協程的調用順序和邏輯則能夠由程序員自己分配和實現。

#### 半協程(semi-coroutine)

Generator 函數便是 ES6 對協程的實現，然而通常意義上的協程能夠從任何地方控制協程的調用和暫停，然而 Generator 函數只有透過函數調用者才能將控制權交給創建好的協程：

```js
function* coroutine() {}
let c1 = coroutine()
let c2 = coroutine()
```

- 說明：如上代碼示例，後兩句表達式分別創建了兩個協程，卻只能透過 `ci.next()` 調用來將控制權轉交給對應的協程，若需要實現協程間的調用轉換則需要另外封裝成一個函數。

### this 綁定

ES6 規定 Generator 函數返回一個 Generator 的實例，並且 Generator 函數不可以做為構造函數，也就是說函數內部並不存在 `this` 對象：

```js
function* gen() {
  yield console.log('gen 1')
  yield console.log('gen 2')
  yield console.log('gen 3')
}
gen.prototype.hi = () => console.log('hello')

let g = gen()
g.next() // gen 1
g.next() // gen 2
g.hi() // hello
```

```js
function* Gen() {
  this.a = 10
}

let g = Gen()
g.a // undefined

let g2 = new Gen()
// TypeError: Gen is not a constructor
```

這時候如果想要在 Generator 對象裡面綁定上下文就需要借助 `Function.prototype.call` 方法：

```js
function* gen() {
  yield (this.id = 0)
  yield (this.name = 'John')
  yield (this.age = 18)
}
let obj = {}
let g = gen.call(obj)
g.next() // { value: 0, done: false }
g.next() // { value: 'John', done: false }
g.next() // { value: 18, done: false }
obj // { id: 0, name: 'John', age: 18 }
```

我們還可以對上下文做進一步的封裝，讓 Generator 函數本身擁有自己的上下文：

```js
function* gen() {
  yield (this.id = 0)
  yield (this.name = 'John')
  yield (this.age = 18)
}

function Gen() {
  return gen.call(gen.prototype)
}

let g = new Gen()
g.next() // { value: 0, done: false }
g.next() // { value: 'John', done: false }
g.next() // { value: 18, done: false }
g.id // 0
g.name // John
g.age // 18
```

## Generator 實例方法

### Generator.prototype.throw()

除了原本的 `next` 方法之外，還能夠向 Generator 實例內部拋出異常：

```js
function* gen() {
  try {
    yield 1
  } catch (e) {
    console.log('catch inside')
  }
  console.log('other operation')
}

let g = gen()
g.next()
try {
  console.log('invoke g.throw 1')
  g.throw()
  console.log('invoke g.throw 2')
  g.throw()
} catch (e) {
  console.log('catch outside')
}
// invoke g.throw 1
// catch inside
// other operation
// invoke g.throw 2
// catch outside

let g = gen()
try {
  console.log('invoke g.throw 1')
  g.throw()
  console.log('invoke g.throw 2')
  g.throw()
} catch (e) {
  console.log('catch outside')
}
// invoke g.throw 1
// catch outside
```

- 說明 1：第一次調用示例由於調用了一次 next 所以當前狀態停留在 `yield 1` 處於 catch 塊內，因此第一次 `g.throw()` 由內部的 catch 捕獲並繼續執行到下次 `yield`，而第二次的 `g.throw()` 由於未被內部捕獲所以在向外拋出被外部的 catch 捕獲。
- 說明 2：由於 g 尚未進入 try 塊因此直接結束調用並由外部捕獲。

```js
function* gen() {
  yield 1
  yield 2
  yield 3
}

let g = gen()
g.next()
try {
  g.throw()
} catch (e) {
  console.log('catch')
}
g.next()
// { value: 1, done: false }
// catch
// { value: undefined, done: true }
```

- 說明：這段代碼演示當 Generator 函數內部拋出未處理異常後，再次調用 `next` 返回的狀態為 `{ value: undefined, done: true }`

### Generator.prototype.return()

Generator 返回的實例還有一個 `return` 方法能直接終結迭代器：

```js
function* gen() {
  yield 1
  yield 2
  yield 3
  yield 4
}
let g = gen()
g.next()
g.next()
g.return(-1)
g.next()
// { value: 1, done: false }
// { value: 2, done: false }
// { value: -1, done: true }
// { value: undefined, done: true }
```

- 說明：`return` 方法傳入的值就作為返回值的 value 值

#### finally

當 Generator 方向內部進入 try 代碼塊的時候調用 `return` 方法就會直接進入 finally 再返回：

```js
function* gen() {
  try {
    yield 'try 1'
    yield 'try 2'
  } finally {
    yield 'finally 1'
    yield 'finally 2'
  }
  yield 'after try-finally'
}

let g = gen()
g.next() // { value: 'try 1', done: false }
g.return(-1) // { value: 'finally 1', done: false }
g.next() // { value: 'finally 2', done: false }
g.next() // { value: -1, done: true }
```

## Generator 應用

### 異步操作同步化

異步操作需要等到請求返回才能繼續執行下列流程，而通常主程序並不想要等待回覆可以先去執行其他代碼，遠古時代使用回調函數，ES6 也提供了另一種語法 Promise 來處理這種問題，這邊使用 Generator 函數：

```js
function* loadUI() {
  showLoading()
  yield loadUIData()
  hideUI()
}
let loader = loadUI()
// 加载UI
loader.next()
// 隐藏UI
loader.next()
```

```js
function* asyncRequest(url) {
  let res = yield request(url)
  if (res && res.status) {
    return {
      status: 'success',
      data: res.data
    }
  } else {
    return {
      status: 'error',
      data: res.data
    }
  }
}

let requestLoader = asyncRequest('http://localhost:8080/hello')
let res = requestLoader.next()
```

### 控制流程管理

如果有多個耗時的連續任務（不一定是異步函數），我們可以採用回調函數的形式：

```js
step1(function (value1) {
  step2(value1, function (value2) {
    step3(value2, function (value3) {
      step4(value3, function (value4) {
        // do something after step4
      })
    })
  })
})
```

我們也可以使用 Promise 的方法解決回調地獄：

```js
Promise.resolve(step1)
  .then(step2)
  .then(step3)
  .then(step4)
  .then((res) => {
    // other operation
  })
  .catch(errHandler)
```

最後我們可以使用一個 Generator 生成器生成一個任務隊列，透過 `next` 方法來控制步驟流程：

```js
function* loadingTasks(value1) {
  try {
    let value2 = yield step1(value1)
    let value3 = yield step2(value2)
    let value4 = yield step3(value3)
    let res = yield step4(value4)
    // other operation
  } catch (e) {
    // error handler
  }
}
```

然後我們再透過一個自動化任務控制器依序執行所有任務

```js
runTasks(loadingTasks(value1))

function runTasks(taskLoader) {
  let res = taskLoader.next(taskLoader.value)
  while (!res.done) {
    taskLoader.value = res.value
    runTasks(taskLoader)
  }
}
```

### Iterator 接口

我們可以為自定義對象部署客製化的 Iterator 函數，將由內置方法 `let...of` 調用：

```js
function* objectIterator() {
  let keys = Object.keys(this)
  for (let key of keys) {
    yield [key, obj[key]]
  }
}

let obj = { id: 0, name: 'John', age: 18 }
obj[Symbol.iterator] = objectIterator

for (let [key, value] of obj) {
  console.log(`{ ${key}: ${value} }`)
}
// { id: 0 }
// { name: John }
// { age: 18 }
```

# 結語

Generator 生成器在不同方面的應用不管是狀態機、異步函數同步化等還是比較複雜，基本上都需要一定程度的封裝和自動化流程控制管理等。ES6 提供的這個能力更像是一個踏板，幫助我們更好地瞭解和應用 Promise、ES7 的 async/await 等特性。
