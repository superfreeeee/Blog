# ES6 特性: Arrow Function箭頭函數

<!-- TOC -->

- [ES6 特性: Arrow Function箭頭函數](#es6-特性-arrow-function箭頭函數)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Declaration 宣告](#declaration-宣告)
    - [規則](#規則)
  - [Restrict 限制](#restrict-限制)
  - [`this`](#this)
    - [全局使用](#全局使用)
    - [類對象中使用](#類對象中使用)
    - [中間休息](#中間休息)
    - [保留 `this`](#保留-this)
  - [易犯錯誤](#易犯錯誤)
    - [一般對象方法](#一般對象方法)
    - [Event 事件](#event-事件)
- [結語](#結語)

<!-- /TOC -->

## 簡介

ECMAScript 2015(ES6)為 JavaScript 帶來全新的變化，豐富的語法糖和更精練的編程，讓代碼能更明顯的表現出業務邏輯忽略繁雜的語法規則，其中影響前端開發者最多沒有之一的就是所謂的 Arrow Function(箭頭函數)。好吧其實他也沒那麼偉大，和傳統的 function 宣告的函數比起來其實多了一些特性，接下來就讓我來細細展開。

## 參考

<table>
    <tr>
        <td>Arrow Function</td>
        <td><a href="https://pjchender.blogspot.com/2017/01/es6-arrow-function.html">https://pjchender.blogspot.com/2017/01/es6-arrow-function.html</a></td>
    </tr>
    <tr>
        <td>阮一峰 ES6 入门</td>
        <td><a href="https://es6.ruanyifeng.com/#docs/function">https://es6.ruanyifeng.com/#docs/function</a></td>
    </tr>
</table>

# 正文

## Declaration 宣告

語法和原本的 function 雷同，但是更加簡便

```js
// origin function
const sayHi = function () {
  console.log('hello')
}

// arrow function
const sayHi = () => {
  console.log('hello')
}
const add = (x, y) => x + y
;[1, 2, 3].map((x) => x * x)
```

### 規則

- 一個參數可以不加圓括號，多個參數必須加圓括號
- 不寫花括號代表直接返回值，花括號內需顯式 return 返回

---

## Restrict 限制

箭頭函數在使用上相較於原來的 function 函數還是有不小的限制

1. 箭頭函數不可作為 class 的構造函數聲明
2. 箭頭函數內部不存在 `arguments` 對象，使用 rest 參數(`...`運算符)代替
3. 箭頭函數內部不可用 `yield`，即箭頭函數不可聲明為 `Generator` 函數

## `this`

接下來就是重頭戲啦，箭頭函數和原來的函數最大的差異在於 `this` 的綁定上面。一句話概述的話，那就是**箭頭函數的 this 在聲明環境綁定，而原本的函數定義則在調用環境時綁定**

### 全局使用

首先來看看全局定義下的兩種函數的差異

```js
const f = function () {
  console.log(this)
}

const f_ = () => {
  console.log(this)
}

f() // [object Window]
f_() // [object Window]
```

好吧，看起來沒什麼差異，因為聲明環境和調用環境都屬於全局(global)，而全局對象在不同環境有不同的表現，如瀏覽器是 `window`、Node 中是 `global`等。

### 類對象中使用

再來我們看看在類對象內部使用兩種函數的後果（參考自 阮一峰 ES6 入門）

```js
function Timer() {
  this.s1 = 0
  this.s2 = 0
  // 計數器傳入箭頭函數
  setInterval(() => {
    this.s1++
  }, 500)
  // 計數器傳入一般函數
  setInterval(function () {
    this.s2++
  }, 500)
}

const timer = new Timer()

setTimeout(() => {
  console.log(timer.s1)
  console.log(timer.s2)
}, 2000)
// 4
// 0
```

可以看出來箭頭函數中 this 指向 timer，使 s1 增加到 4；但是一般函數設置在 setInterval 時執行的位置位於全局，所以真正的 this.s2 指向了全局對象的 s2 變量。

### 中間休息

好吧到此為止，我們可以這樣了解：

箭頭函數的 `this` 直接繼承了定義時刻當下的 this 變量(當然 `arguments`, `super`, `new.target` 也有類似的表現)，而一般函數在調用的時候，`this` 被執行函數的上下文對象所替代了。

### 保留 `this`

現在我們可以透過保留定義時刻的上下文綁定到一般函數內部，如下

```js
function Timer() {
  this.s1 = 0
  const _this = this
  setInterval(() => {
    _this.s1++
  }, 500)
}
const timer = new Timer()
setTimeout(() => {
  console.log(timer.s1)
}, 2100)
// 4
```

以此證明，箭頭函數提供了一種更便捷自動綁定 `this` 的語法糖，程序猿可以依據情況自由選擇是否綁定當前上下文的 `this`。箭頭函數相當於提供了 `Function.prototype.bind` 的作用的語法，寫過 React 會發現如果不使用箭頭函數需要為每個方法綁定一次 `this` 這有夠煩的。

---

## 易犯錯誤

### 一般對象方法

在一般對象中將箭頭函數作為一個屬性聲明，但是聲明對象並沒有 `this` 變量，所以真正的 `this` 還要繼承於再外面一層的 `this`

```js
function Timer() {
  this.s = 'outer s'
  const obj = {
    s: 'inner s',
    getS: () => {
      console.log(this.s)
    }
  }
  obj.getS()
}
const timer = new Timer()
// outer s
```

### Event 事件

綁定響應事件的時候，需要動態綁定 `this` 到觸發的元素身上，這時候不應該使用箭頭函數

```js
const btn = document.querySelector('.btn')
btn.addEventListener('click', (e) => {
  this.classList.add('visible')
})
```

由於使用箭頭函數提前在定義時綁定了 `this`，所以真正觸發事件的時候，響應函數捕捉到的 `this` 是定義時的 `this` 也就是全局對象，因此 btn 元素的 classList 並不會被改變

# 結語

本篇簡單介紹了 ES6 的 Arrow Function(箭頭函數)和一般函數的差異，除了享受 ES6 帶來的語法糖之外，作為一個開發者最重要的是要釐清兩個函數綁定 `this` 的機制和差異，在精簡化語句之前更重要的是要把代碼寫對。
