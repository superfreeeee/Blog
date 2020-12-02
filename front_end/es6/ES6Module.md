# ES6 特性: Module 模塊化

@[TOC](文章目錄)

<!-- TOC -->

- [ES6 特性: Module 模塊化](#es6-特性-module-模塊化)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Usage 應用](#usage-應用)
  - [Overview 概述](#overview-概述)
    - [`export` 導出](#export-導出)
      - [`export var` 個別導出](#export-var-個別導出)
      - [`export {}` 一組導出](#export--一組導出)
      - [`export { as }` 導出別名](#export--as--導出別名)
      - [`export default` 默認導出](#export-default-默認導出)
      - [導出引用](#導出引用)
    - [`import` 導入](#import-導入)
      - [`import {}` 個別導入](#import--個別導入)
      - [`import { as }` 導入別名](#import--as--導入別名)
      - [`import * as` 整體導入](#import--as-整體導入)
      - [`import` 默認導入](#import-默認導入)
    - [`export + default` 複合寫法](#export--default-複合寫法)
      - [提案：簡化先導入再導出](#提案簡化先導入再導出)
    - [`import()` 動態導入](#import-動態導入)
      - [語法](#語法)
- [結語](#結語)

<!-- /TOC -->

## 簡介

在 ES6 之前，JS 語言並不具備模塊(Module)的特性，隨著系統規模和代碼量急劇的增加，模塊化勢在必行。一開始出現了如 CommonJS、AMD 等標準，一直到 ES6 才出現統一標準化的模塊化語法，有了 Module 就能夠實現代碼分離，並且為動態加載代碼鋪路，接下來就讓我們來瞧瞧 ES6 標準裡的模塊化語法。

## 參考

<table>
  <tr>
    <td>ES6 module</td>
    <td><a href="http://caibaojian.com/es6/module.html">http://caibaojian.com/es6/module.html</a></td>
  </tr>
</table>

# 正文

## Usage 應用

由於部分瀏覽器版本可能不支持模塊化系統(較新版的應該都沒問題)，依需求可以使用 babel 進行轉換，可在我的 <a href="https://blog.csdn.net/weixin_44691608/article/details/106579653">Babel 入門</a>找到簡單的使用介紹，並使用 node 環境執行代碼，這邊主要介紹語法。

## Overview 概述

模塊化的語法最主要就是兩個關鍵字：`export` 和 `import`。一個導出，一個導入，簡單明瞭。

### `export` 導出

ES6 的模塊系統，將一個 JS 文件視為一個模塊，開發者就可以使用 `export` 向模塊外部暴露(導出)變量或是方法，最基本的用法如下：

#### `export var` 個別導出

```js
// a.js
export const a = 1
export const o = { value: 2 }
export const f = function () {
  console.log('invoke f')
}
export class C {}

// b.js
import { a, o, f, C } from './a'
```

這邊使用 `{ a, o, f, C }` 表示從 a 模塊中導入 `a`、`o`、`f`、`C` 三個變量，後續會介紹 `import` 相關語法。

#### `export {}` 一組導出

除了像上面一樣一個個導出變量或方法之外，還可以使用 `export {}` 來一次導出一組變量，如下：

```js
// a.js
const a = 1
const o = { value: 2 }
const f = function () {
  console.log('invoke f')
}
class C {}

export { a, o, f, C }

// b.js
import { a, o, f, C } from './a'
```

**注意！**這裏與 CommonJS 不同，`export` 後面的 `{}` 並不是一個對象，所以並不是 `{ a:a }` 的縮寫，也不能這樣使用，別名需要使用後面的 `as` 語法。

將多個變量個別導出，與一次導出一組變量是等價的，如下：

```js
export const a = 1
export const b = 2
export const c = 3
// 等價於
const a = 1
const b = 2
const c = 3
export { a, b, c }
```

#### `export { as }` 導出別名

能夠導出變量之後，我們可能希望裡外的命名空間能夠區別開來，這時候就可以使用關鍵字 `as` 來改變對外暴露的接口：

```js
// a.js
const hello = function () {
  console.log('hello')
}
export { hello as hi }

// b.js
import { hi } from './a'
```

- 說明：由於導出時為 hello 取了別名 hi，因此其他模塊引用的時候要使用 hi，hello 將被視作未定義

#### `export default` 默認導出

有時候使用者並不知道模塊內的變量或方法的確切名字，相反的我們希望使用者只使用模塊導出的唯一值，那們我們就可以使用 `export default` 語法而不需要為其指定一個名字：

```js
// a.js
export default {
  el: '#app',
  name: 'default object'
}

// b.js
import A from './a'
```

- 說明 1：注意這邊的 `export default {}`，由於 `export default` 本身也是導出某個值，所以這裡的 `{}` 確實是指一個對象，與 `export {}` 表示導出一組值不同

- 說明 2：這時候導入就不需要使用 `{}`，後面的 `import` 會介紹。

其實默認(`default`)導出是一種別名，如下代碼：

```js
export default const a = 1
// 等價於
export { a as default }
```

#### 導出引用

這邊需要注意的一點是，模塊導出的是變量的引用，而不是靜態的副本，也就是說導入的變量值實際指向存在於外部模塊的變量，是會動態修改值得，看代碼：

```js
// a.js
export let i = 1

export const inc = function () {
  i++
}

// b.js
import { i, inc } from './a'

console.log(`i = ${i}`)
inc()
console.log(`i = ${i}`)

// output:
// i = 1
// i = 2
```

- 說明：由於導入進來的 `i` 變量指向 `a.js` 中的變量，因此調用 `inc` 函數後修改了 `i` 的值，所以第二次打印的時候 `i` 的值為 2。

---

到此，我們已經具備多種手段導出模塊內(單個 JS 文件)的各種變量或方法了，接下來我們來介紹如何導入模塊

### `import` 導入

"不要重複造輪子"，很多時候我們都需要引用別人寫好的模塊，甚至自己的項目也需要模塊化，這時候就需要向外部模塊`導入(import)`各樣的變量或方法

#### `import {}` 個別導入

一個模塊可能對外暴露多個接口（可能使用 `export` 一個個導出，或是使用 `export {}` 一次導出多個），從外部模塊的角度來說  就是一組接口，因此在 `{}` 填上需要使用到的變量名（與 ES6 的對象解構賦值相似但是**不同**）

```js
// a.js
export const a = 1
export const b = 2

// b.js
import { a, b } from './a'

console.log(`a = ${a}`)
console.log(`b = ${b}`)

// output:
// a = 1
// b = 2
```

#### `import { as }` 導入別名

當導入的變量與當前模塊的命名空間產生衝突時，可以使用別名來改變導入變量的名稱，以避免衝突：

```js
// a.js
export let i = 1
export const inc = () => i++

// b.js
import { i as a_i, inc } from './a'
const i = 'i in b.js'
console.log(`i = ${a_i}`)
inc()
console.log(`i = ${a_i}`)
```

- 說明：這邊 b 模塊內部已經佔用名字 i 了，因此從 a 模塊導入的變量必須使用別名 a_i 以避免命名空間衝突

#### `import * as` 整體導入

有時候我們並不想一一列出導入的變量名，我們可以使用 `*` 來表示導入整個模塊，並透過 `as` 為整個模塊取一個別名，這樣就能夠實現一次性導入整個模塊：

```js
// a.js
export let i = 1
export const inc = () => i++

// b.js
import * as A from './a'
console.log(`i = ${A.i}`)
A.inc()
console.log(`i = ${A.i}`)
```

#### `import` 默認導入

這邊對應 `export default` 的默認導出，當使用默認導出的時候，就可以將模塊本身也代表著某個變量（默認導出的變量），這時候就不需要指定任何名稱也不需要使用 `{}` 語法了：

```js
// a.js
export default {
  el: '#app',
  name: 'componentA'
}

// b.js
import ComponentA from './a'
console.log(ComponentA)

// output:
// { el: '#app', name: 'componentA' }
```

### `export + default` 複合寫法

有時候有些模塊作為統一向外暴露的接口，並不需要對導入的模塊的變量做任何處理直接導出，你可能會想到這麼寫：

```js
import A from './a'
export { A }
```

ES6 提供了一種複合語法，你也可以說是語法糖吧，就是導入同時導出，如下：

```js
// a.js
export const a = 'default a'

// b.js
export * from './a'

// c.js
import { a } from './b'
```

如果你是要導出一組值則可以這樣寫：

```js
// a.js
export const a = 1
export const b = 2
export const c = 3

// b.js
export { a, b, c } from './a'

// c.js
import { a, b, c } from './a'
```

這邊如果你想要導出默認路由的話寫 `export default` 會變成僅僅只是默認導出而不是導入默認模塊，要改變成如下形式：

```js
// a.js
export default 123

// b.js
export { default } from './a'

// c.js
import B from './b'
```

或是改變默認導出

```js
// a.js
export const a = 123

// b.js
export { a as default } from './a'

// c.js
import B from './b'
```

#### 提案：簡化先導入再導出

現行的複合語法是先導入再導出，因此會出現下面這種有點憋腳的寫法：

```js
export { a } from './a'
```

因此 ES7 有一種提案是拿掉輸出後的大括號：

```js
export a from './a'
```

- 說明：而默認導入則能夠使用 `default` 作為替代，寫法上更加簡潔可讀。

### `import()` 動態導入

上述提到的 `import` 屬於靜態導入，也就是說會先於所有表達式執行，如下代碼：

```js
// a.js
console.log('load a.js')
export default 1

// b.js
console.log('load b.js')
export default 2

// c.js
console.log('load c.js')
import A from './a'
console.log(A)
import B from './b'
console.log(B)

// output:
// load a.js
// load b.js
// load c.js
// 1
// 2
```

同時如果 `import` 語句放在非底層代碼塊的時候將會報錯：

```js
if (true) {
  import A from './a'
}
```

因此就有一個提案說到，應該引入一個動態加載，不僅可以通過條件控制來決定動態引入哪些模塊，同時還能實現`懶加載(lazy-import)`，將真正的依賴引入延遲到運行時，能夠大幅減少引入大量大體積依賴的開銷，提案中的語法如下，`import()` 函數將會返回一個 Promise 對象：

#### 語法

```js
import('./a').then((A) => {
  // ...
})
```

這邊的動態加載與 node 的 `require` 也有不同，提案中的 `import()` 屬於異步加載，而 node 的 `require` 則屬於同步加載。

# 結語

本篇簡單介紹了 ES6 標準下的模塊化規範，如果你看到舊版代碼與 ES6 的語法不同，可能是 ES5 以前的模塊化規範（CommonJS、AMD 等）。一個有良好設計架構並且採用模塊化的技術，可以說是構建一個大型系統的基本要素，即便不是構建大型系統，模塊化依舊能為代碼風格以至體系結構帶來巨大的好處，也是身為一個開發者必須要具備的素養。
