# JS 基礎：Hoisting 變量提升、TDZ 暫時性死區(Temporal Dead Zone)

@[TOC](文章目錄)

<!-- TOC -->

- [JS 基礎：Hoisting 變量提升、TDZ 暫時性死區(Temporal Dead Zone)](#js-基礎hoisting-變量提升tdz-暫時性死區temporal-dead-zone)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Scope 作用域](#scope-作用域)
    - [`var`](#var)
    - [`let`、`const`](#letconst)
  - [Hoisting 變量提升(對於 `var` 來說)](#hoisting-變量提升對於-var-來說)
  - [暫時性死區 TDZ (Temporal Dead Zone)](#暫時性死區-tdz-temporal-dead-zone)
- [結語](#結語)

<!-- /TOC -->

## 簡介

本篇將會分成三部分

1. 介紹 JS 語言裡的作用域
2. 介紹什麼叫做變量提升(hoisting)，以及身為解釋型語言的 JavaScript 為什麼會存在變量提升
3. `var` 與 `let`、`const` 之間的差異(`暫時性死區 TDZ = Temporal Dead Zone`)

## 參考

<table>
  <tr>
    <td>JavaScript到底是解释型语言还是编译型语言?</td>
    <td><a href="https://segmentfault.com/a/1190000013126460">https://segmentfault.com/a/1190000013126460</a></td>
  </tr>
  <tr>
    <td>ES6 let和const命令</td>
    <td><a href="http://caibaojian.com/es6/let.html">http://caibaojian.com/es6/let.html</a></td>
  </tr>
</table>

# 正文

## Scope 作用域

我們要了解`變量提升(hoisting)`、`暫時性死區(Temporal Dead Zone)`，或是到之後幾篇將會提到的`閉包(closure)`等概念之前，我們需要先明白 JS 的作用域。

JS 聲明變量的關鍵字有三個：`var`、`let`、`const`。我只能說，一切的災難都是從遠古時代的 `var` 的作用域引起的。

### `var`

`var` 關鍵字的作用域有兩種：`全局作用域(global scope)`和`函數作用域(function scope)`

也就是說除了函數，其他`塊(block)`訪問的變量範圍都是屬於全局作用域：

```js
var a = 0
console.log(a)

{
  var a = 1
}
console.log(a)

if (true) {
  var a = 2
}
console.log(a)

for (;;) {
  var a
  a = 3
  break
}
console.log(a)

while (true) {
  var a = 4
  break
}
console.log(a)

// output:
// 0
// 1
// 2
// 3
// 4
```

我們可以發現對於 `for`、`while`、`{}`、`if-else` 來說，變量的作用域都一樣都是全局變量，所以透過 var 聲明的變量我們只能透過函數來區隔：

```js
var a = 0

function f() {
  var a = 1
  console.log(`a inside = ${a}`)
}

f()
console.log(`a outside = ${a}`)

// output:
// a inside = 1
// a outside = 0
```

有關`閉包(closure)`的用法我們留到下一篇來解釋

### `let`、`const`

後來大家發現，只有全局作用域和函數作用域實在是太不方便了，而且與廣大開發者的既定習慣不同(尤其是熟悉 C-like 語言的開發者，更習慣`塊級作用域(block scope)`的變量聲明方式)。最著名的就是下面列出的經典面試題：

```js
for (var a = 0; a < 10; a++) {
  setTimeout(function () {
    console.log(a)
  }, 0)
}

// output:
// 10
// 10
// 10
// 10
// 10
// 10
// 10
// 10
// 10
// 10
```

由於 `var` 關鍵字只有全局和函數兩種，也就是說上面的代碼中在整個運行期間只有一個全局作用域之下的版本，所以調用十次`異步函數 setTimeout` 真正執行時訪問的 `a` 變量都是同一個，也就是在 for 循環結束之後自增到 10 的那一個。

因此特性引發了眾多災難，所以 ECMAScript 也在 ES6 的標準中納入了`塊級作用域(block scope)`，提供了兩個關鍵字用來聲明擁有塊級作用域的變量：`let`、`const` 分別用來定義變量和常量兩種。

接下來我們將上面的代碼改成用 `let` 關鍵字來聲明：

```js
for (let a = 0; a < 10; a++) {
  setTimeout(function () {
    console.log(a)
  }, 0)
}

// output:
// 1
// 2
// 3
// 4
// 5
// 6
// 7
// 8
// 9
// 10
```

這樣一來表現就正常了也是符合我們的期待，由於 `let` 聲明的變量 `a` 是綁定在塊(`{}`)上的，也就是說 for 循環期間十個循環體中的塊都是相互獨立開來的，而存在於其中的塊的 `a` 變量也在當下唯一的，所以即便使用 `setTimeout` 函數，回調函數中還是能夠正確訪問到正確的塊中的 `a` 變量，也就能正確的輸出 1 到 10。

## Hoisting 變量提升(對於 `var` 來說)

所謂的變量提升(hoisting)，是指在同樣的作用域下，在初始化前訪問變量是被允許的，只是值為 `undefined`，如下代碼示例：

```js
var a = 1

function f() {
  console.log(a)
  var a
}

f()
// output:
// undefined
```

有些文章會寫說 JS 引擎在運行時會修改源代碼的順序，使得所有變量都正確的聲明，並將初始化留到正確的位置，也就是說上面的代碼會被改成下面這樣：

```js
var a
console.log(a)
a = 1
console.log(a)
```

但其實這樣說一半對一半也不對，JS 引擎`並不會改變源代碼結構`，而是由於 JS 引擎執行代碼的機制所導致的：

JS 引擎(如大名鼎鼎的 V8 引擎)在運行 JS 代碼的時候，會經歷`詞法分析(Lexical analysis)`、`語法分析(Syntax analysis)`等一系列步驟最後形成一顆`抽象語法樹(AST)`（相關細節可以查詢`編譯原理`）。而 JS 引擎在解析出 AST 時會為變量聲明預留一個空間，也就是`在解析階段就會分配內存`，所以在聲明的語句前就能夠訪問變量(因為他已經分配好空間了！)，這就是變量提升的秘密，並不是真的修改了源代碼結構。

## 暫時性死區 TDZ (Temporal Dead Zone)

上面提到的變量提升(hoisting)是使用 `var` 關鍵字聲明變量的時候會產生的情形，而 ES6 新增的 `let`、`const` 也有類似的情況。然而不同的是，不可以在同一個塊(block scope)使用 `let`、`const` 聲明變量之前訪問這個變量，不然會出現如下報錯：

```js
console.log(a)
let a = 1

// output:
// ReferenceError: Cannot access 'a' before initialization
```

並且 `let`、`const` 是不能重複聲明的，所以也不能像下面這樣寫：

```js
let a = 1
console.log(a)
let a = 1

// output:
// SyntaxError: Identifier 'a' has already been declared
```

不過這個特性並不是要來搞事，反而是幫助我們檢查`變量在訪問之前有正確並且唯一的聲明`，所以 ES6 之後除了推薦使用 `let`、`const` 來操作正確的`塊級作用域(block scope)`之外，變量聲明都應該在訪問之前並且貼近訪問的部分，如下示例：

```js
let form = {}
validate(form)
submit(form)(
  (function mounted(ctx) {
    const name = 'John'
    ctx.name = name
  })({})
)
```

# 結語

所謂的`變量提升(hoisting)`以及`暫時性死區(Temporal Dead Zone)`其實很簡單，一個是 JS 引擎解析和分配內存的方式，一個是明確變量聲明和訪問的先後順序。其實只要搞清楚不同關鍵字的`作用域(scope)`，並且養成先聲明在訪問的好的編碼習慣，其實你幾乎不會遇到這些問題。
