# JS 基礎：void 冷知識

@[TOC](文章目錄)

<!-- TOC -->

- [JS 基礎：void 冷知識](#js-基礎void-冷知識)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [語法](#語法)
  - [Usage 作用](#usage-作用)
  - [Application](#application)
    - [`<a href="javascript:void(0)"></a>`](#a-hrefjavascriptvoid0a)
    - [IIFE(Immediately Invoked Function Expression)](#iifeimmediately-invoked-function-expression)
    - [箭頭函數](#箭頭函數)
- [結語](#結語)

<!-- /TOC -->

## 簡介

今天來介紹一下 JS 規範中超級沒有存在感的運算符：`void`。啥？聽都沒聽過？我也沒聽過，要說本篇的來源是某次在 <a href="https://www.babeljs.cn/">babel 首頁</a>編譯後的代碼裡面無意間撇到一個 `void`

```js
// 有興趣的可以到 babel 首頁嘗試
// 編譯前
() => { this }

// 編譯後
var _this = void 0;
(funciont() {
  _this;
}());

// 有趣的是英文版首頁的編譯又是另一種結果
() => { void 0; };
```

當下心理道：這都是些啥東東的呢。。。

## 參考

<table>
  <tr>
    <td>JS 冷知識: 你所不知道的 void</td>
    <td><a href="https://kuro.tw/posts/2019/08/04/JS-%E5%86%B7%E7%9F%A5%E8%AD%98-%E4%BD%A0%E6%89%80%E4%B8%8D%E7%9F%A5%E9%81%93%E7%9A%84-void/">https://kuro.tw/posts/2019/08/04/JS-%E5%86%B7%E7%9F%A5%E8%AD%98-%E4%BD%A0%E6%89%80%E4%B8%8D%E7%9F%A5%E9%81%93%E7%9A%84-void/</a></td>
  </tr>
  <tr>
    <td>void 运算符</td>
    <td><a href="https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/void">https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/void</a></td>
  </tr>
</table>

# 正文

我們直接進入正題，void 到底哪來的？void 的作用是啥？void 都用在哪？

## 語法

在介紹其來源之前，我們先來介紹一下他的語法：

```js
void expression
void expression
```

void 關鍵字作為一元運算符後面能夠接一個表達式，並且不管表達式為何都會返回 `undefined`（這很重要，先記下來）

也就是說如下代碼：

```js
let i = void 0
// i = undefined
let j = void (i = 2)
// i = 2
// j = undefined
```

## Usage 作用

好了我們現在知道他會返回 `undefined` 了，然後勒？能幹嘛？

`void` 作為一個運算符關鍵字其實早早就被加入到 ES 的標準裡面。而我猜加入他的初衷可能是因為：`undefined` 並不是保留字。是的沒錯你眼睛沒花，在早早的遠古時代大家就發現 ES 規範裡面並沒有把 `undefined` 劃入保留字裡面，也就是說你是可以創建一個名叫 `undefined` 的變量的！（雖然這樣完全沒有意義）

```js
var undefined = 0
// 親側這樣寫是合法的，但是現在的 chrome 瀏覽器已經失效了
```

當然現在也不會有人這麼無聊，所以我也不想在這個問題上糾結太多。但是也不知道 ES 小組到底是不是因為注意到了這件事，又不想破壞原本的規範（可能是考慮到或許有人的程序聲明了叫 undefined 的變量，所以不想破壞兼容行，我猜啦），因此才推出一個新的運算符 void。他能夠成為替代 undefined，確保表達是一定會返回 undefined。

## Application

### `<a href="javascript:void(0)"></a>`

當初早期的時候一定可以常常看見 html 的鏈結標籤(`<a>`)中出現過這種東西：

```html
<a href="javascript:void(0)"></a>
```

這是由於 `a` 標籤的默認行為是會把 href 解析 url 後的資源用當前網頁替代，因此如果沒有返回 void 就會把整個網頁換成一個 0。現在許多回調方法，或是綁定的監聽函數都會要求調用 `e.preventDefault()` 也是為了同樣的效果。

**注意！**當然，現在繼續使用 `javascript:` 這種偽協議已經不再推薦了

### IIFE(Immediately Invoked Function Expression)

依舊是遠古時代，當開發者需要區分作用域或加載時立刻執行的代碼塊，曾經引入過一種立即函數(IIFE)方法，他本身也是一個表達式，有可能會造成返回洩漏，所以也可以透過 void 包裝起來：

```js
void (function (val) {
  console.log(123 + val)
})(123)
```

如此一來就能避免 IIFE 的返回值暴露到調用函數之外了

### 箭頭函數

ES6 出現的箭頭函數有時候也會有和 IIFE 類似的疑慮，當沒有函數體也不需要返回值的時候就也需要使用 void 做包裝：

```js
let user = {
  name: 'John',
  password: '123456789'
}
const setPassword = (newPassword) => (void user.password = newPassword)
```

如此一來就可以阻止箭頭函數的洩漏

# 結語

本篇介紹了一個較為冷門的運算符 void。即時在實際開發的時候很少人會直接使用到，但是在 babel 編譯的背後依舊能看見他的身影，說明這個運算符依舊在不那麼顯眼的地方發揮它的作用，並且許多 babel 實現的兼容舊版本的能力有需多便是要依靠更接近 JS 本身語言特性的 API，多學多會多看多聽，技多不壓身，說不定哪天就遇到必須使用 void 來解決的問題也說不定。
