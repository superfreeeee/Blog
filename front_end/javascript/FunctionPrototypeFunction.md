# JS 基礎：Function.prototype 三大方法

@[TOC](文章目錄)

<!-- TOC -->

- [JS 基礎：Function.prototype 三大方法](#js-基礎functionprototype-三大方法)
    - [簡介](#簡介)
    - [參考](#參考)
- [正文](#正文)
    - [目的](#目的)
    - [`Function.prototype.call` & `Function.prototype.apply`](#functionprototypecall--functionprototypeapply)
        - [call & apply 語法](#call--apply-語法)
        - [call & apply Sample](#call--apply-sample)
    - [`Function.prototype.bind`](#functionprototypebind)
        - [bind 語法](#bind-語法)
        - [bind Sample](#bind-sample)
    - [應用](#應用)
        - [self 替換](#self-替換)
        - [bind 函數](#bind-函數)
        - [arrow function 箭頭函數](#arrow-function-箭頭函數)
- [結語](#結語)

<!-- /TOC -->

## 簡介

Function.prototype 有三個經典的方法調用，分別是 `call`、`apply`、`bind`，一言以蔽之就是綁定調用上下文，接下來就快速進入三個方法介紹。

## 參考

<table>
    <tr>
        <td>函數原型最實用的 3 個方法 — call、apply、bind</td>
        <td><a href="https://medium.com/@realdennis/javascript-%E8%81%8A%E8%81%8Acall-apply-bind%E7%9A%84%E5%B7%AE%E7%95%B0%E8%88%87%E7%9B%B8%E4%BC%BC%E4%B9%8B%E8%99%95-2f82a4b4dd66">https://medium.com/@realdennis/javascript-%E8%81%8A%E8%81%8Acall-apply-bind%E7%9A%84%E5%B7%AE%E7%95%B0%E8%88%87%E7%9B%B8%E4%BC%BC%E4%B9%8B%E8%99%95-2f82a4b4dd66</a></td>
    </tr>
    <tr>
        <td>Function.prototype</td>
        <td><a href="https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Function/prototype">https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Function/prototype</a></td>
    </tr>
</table>

# 正文

## 目的

首先我們先明確這三個函數的目的，我們都知道 `function` 關鍵字定義的一般函數會以調用者或調用環境作為上下文，也就是運行時動態綁定 `this`，這時候容易出現回調函數沒有正確綁定到上下文的情況（用過 React 的人一定對這行代碼不陌生：`this.xxx = this.xxx.bind(this)`），以下我們就來講解 `call`、`apply`、`bind` 用法上的意義和區別

## `Function.prototype.call` & `Function.prototype.apply`

`call` 和 `apply` 幾乎完全一樣，除了傳入參數的形式略有差異

### call & apply 語法

```js
// call 語法
func.call(ctx[, arg1[, arg2[, ...]]])

// apply 語法
func.apply(ctx, [argArray])
```

- 說明：ctx 作為調用上下文為必選參數，而後面參數傳入的形式，call 逐個傳入參數；而 apply 則接受一個參數數組

### call & apply Sample

```js
function hi(a, b, c) {
  console.log(`Hello my name is ${this.name}`)
  console.log(`a = ${a}`)
  console.log(`b = ${b}`)
  console.log(`c = ${c}`)
}
let person = { name: 'John' }
let a = 1,
  b = 2,
  c = 3
hi.call(person, a, b, c)
// Hello my name is John
// a = 1
// b = 2
// c = 3

hi.apply(person, [a, b, c])
// Hello my name is John
// a = 1
// b = 2
// c = 3
```

- 說明：`hi` 方法需要接受多個參數，同時內部調用 `this.name` 屬性，這時候的 `this` 對象就是 call 或 apply 傳入的第一個參數也就是 person 對象。也就是說下列的調用形式等價：

```js
func.call(ctx, arg1, arg2, arg3)
// 等價於
func.apply(ctx, [arg1, arg2, arg3])
// 等價於
let ctx = {
  // some properties...
  func
}
ctx.func(arg1, arg2, arg3)
```

說實在使用上差異不大，最重要的差別可能要涉及到函數調用時的運行時機制，以及函數處理等。後續更進階可能還會用到 Thunk 函數、柯里化函數等手段，因此不需要在 call 和 apply 上過於糾結。

## `Function.prototype.bind`

顧名思義就是一個“綁定”方法，這個方法的作用就是返回一個綁定了上下文環境的函數

### bind 語法

```js
func.bind(ctx) // 返回綁定了上下文還的境函數
```

### bind Sample

延續剛剛用過的例子，這次我們先綁定好上下文環境之後就可以直接調用了：

```js
function hi(a, b, c) {
  console.log(`Hello my name is ${this.name}`)
  console.log(`a = ${a}`)
  console.log(`b = ${b}`)
  console.log(`c = ${c}`)
}
let person = { name: 'John' }
let a = 1,
  b = 2,
  c = 3

hi = hi.bind(person)
hi(a, b, c)
```

- 說明：由於 hi 方法已經被替換成綁定 person 的版本，因此調用的時候 this 就固定指向 person 對象而不是全局對象

## 應用

其實通常在寫業務邏輯的時候，比較少情況會出現 this 對象錯誤或非預期綁定的情況。更多的時候是出現在維護遺留項目(legacy project)的時候，想要複用過往的代碼，這時候我們可以有三種修改綁定的方法（詳見參考一鏈結）

### self 替換

在函數內部使用 self 保留當前環境的 this 指針

```js
let obj = {
  name: 'John',
  someFunc() {
    const self = this
    setTimeout(function () {
      console.log(`Hello my name is ${self.name}`)
    }, 1000)
  }
}
obj.someFunc()
```

- 說明：如果沒有用 self 保留 this 指針的話，由於事件循環機制(Event Loop)會使得真正調用函數時的 this 指向全局對象，所以我們需要用保留下來的 self 來取代原本要引用的 this

### bind 函數

我們也可以用本篇說到的 bind 函數在定義時就先綁定好上下文環境：

```js
let obj = {
  name: 'John',
  someFunc() {
    setTimeout(
      function () {
        console.log(`Hello my name is ${this.name}`)
      }.bind(this),
      1000
    )
  }
}
obj.someFunc()
```

### arrow function 箭頭函數

ES6 其實已經為這種情況提供語法糖，也就是箭頭函數的實現：

```js
let obj = {
  name: 'John',
  someFunc() {
    setTimeout(() => {
      console.log(`Hello my name is ${this.name}`)
    }, 1000)
  }
}
obj.someFunc()
```

- 說明：在之前的**ES6 特性：箭頭函數**篇就已經介紹過，箭頭函數的 this 在聲明時就會直接綁定到當前對象也就是 obj 對象

# 結語

本篇簡單介紹了 Function.prototype 的三大方法，本質上都是在處理上下文環境的綁定，可以在調用時指定上下文環境(call、apply)、也可以直接返回一個綁定好上下文環境的新函數(bind)，縱然這三個方法非常基礎，卻是使用時容易忽略的技術細節，也是其他高階用法的基礎。
