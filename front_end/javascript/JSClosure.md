# JS 基礎：Closure 閉包

@[TOC](文章目錄)

<!-- TOC -->

- [JS 基礎：Closure 閉包](#js-基礎closure-閉包)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [IIFE 立即執行函數(Immediately Invoked Functions Expression)](#iife-立即執行函數immediately-invoked-functions-expression)
  - [Closure 閉包](#closure-閉包)
    - [for 循環：使用 IIFE 建立塊級作用域](#for-循環使用-iife-建立塊級作用域)
    - [PrimeCreator 質數(素數)製造機](#primecreator-質數素數製造機)
    - [Module 模塊化](#module-模塊化)
    - [Module Loader 模塊加載器](#module-loader-模塊加載器)
- [結語](#結語)

<!-- /TOC -->

## 簡介

在 ES6 新增 `let`、`const` 的塊級作用域變量聲明，以及 `import/export` 等模塊化語法之前，只能透過 `var` 以及，IIFE 來區隔命名空間以及作用域，寫法之複雜和繁瑣，並且大大降低代碼的可讀性。

本篇將會重點介紹  `立即執行函數(IIFE)`、`閉包(Closure)`的使用方法和具體應用。

## 參考

<table>
  <tr>
    <td>你懂 JavaScript 嗎？#15 閉包（Closure）</td>
    <td><a href="https://cythilya.github.io/2018/10/22/closure/">https://cythilya.github.io/2018/10/22/closure/</a></td>
  </tr>
  <tr>
    <td>[筆記] 談談JavaScript中的IIFEs(Immediately Invoked Functions Expressions)</td>
    <td><a href="https://pjchender.blogspot.com/2016/05/javascriptiifesimmediately-invoked.html">https://pjchender.blogspot.com/2016/05/javascriptiifesimmediately-invoked.html</a></td>
  </tr>
</table>

# 正文

## IIFE 立即執行函數(Immediately Invoked Functions Expression)

`立即執行函數(IIFE)`顧名思義，就是立即執行的函數（廢話！
），我們先來看一下他的使用形式：

```js
// 加載模塊時立即執行的函數
;(function loaded(w) {
  console.log('inovke immediately when js file was loaded')
})(window)

// 立即執行函數生成初始化值
const greeting = (function (name) {
  return `hello ${name}`
})('John')
```

你可能會想，阿就是一個立即執行的函數啊有什麼了不起的。先別急，IIFE 是實現`閉包(closure)`非常重要的一環，我們只要先知道 IIFE 有以下特性：

1. 解析到 IIFE 時將立即執行函數表達式，而不用聲明一個函數再調用（相當於生成一個暫時的函數，然後執行完後馬上消失）
2. 能夠區隔作用域，由於遠古時代只有 `var` 的函數作用域，所以可以使用 IIFE 的區隔命名空間，如下代碼：

```js
var b = (function (i) {
  var a = i
  return a
})(1)
console.log(b)
// 1
console.log(a)
// ReferenceError: a is not defined
```

接下來讓我們先繼續看下去

## Closure 閉包

在上面我們知道 IIFE 可以區隔作用域，因為 `var` 具有函數作用域的特性，所以我們可以將變量"鎖"在 IIFE 的函數作用域內，而避免裡面聲明的變量污染到全局環境。

所謂的`閉包(closure)`的作用在於，將變量聲明在一個`暫時存在的作用域(也就是一個函數)`當中，並且將訪問作用域內部的值的一些方法返回暴露出來。由於 JavaScript 與 Java 相似的是都有自己的垃圾回收處理機制，但是由於 IIFE 的返回值保留了暫時作用域的變量訪問，因此就會出現類似`內存泄露`的情況，在作用域結束之後分配給變量的內存依舊存在。

```js
var { greeting, setName } = (function () {
  var name = 'John'

  function greeting() {
    console.log(`Hello ${name}`)
  }

  function setName(val) {
    name = val
  }

  return {
    greeting,
    setName
  }
})()

greeting()
setName('Andy')
greeting()

// output:
// Hello John
// Hello Andy
```

我們可以發現 `name` 變量就這樣被"鎖"在 IIFE 裡面了，我們只能透過返回的 `greeting`、`setName` 函數來修改和訪問這個值（這邊使用了 ES6 的對象解構賦值），並且由於 `greeting` 存在對變量 `name` 的引用關係，所以 `name` 不會被垃圾回收器回收內存，而這也是`閉包(closure)`的命名由來。

接下來我們邊舉例子邊了解閉包的特性以及應用

### for 循環：使用 IIFE 建立塊級作用域

首先第一個例子我們先來改造一個經典的面試題：

- 原題

```js
for (var i = 0; i < 10; i++) {
  setTimeout(function () {
    console.log(i)
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

這個問題是因為 `var` 的作用域問題，在我<a href="https://blog.csdn.net/weixin_44691608/article/details/106766444">上一篇</a>有說到

- 解法一：使用 let

```js
for (let i = 0; i < 10; i++) {
  setTimeout(function () {
    console.log(i)
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

由於 `let` 聲明的變量屬於塊級作用域，所以 setTimeout 函數會調用正確的塊中的 i 值。但是在 ES6 之前大家是怎麼解決的呢？

- 解法二：使用 IIFE

```js
for (var i = 0; i < 10; i++) {
  ;(function (i) {
    setTimeout(function () {
      console.log(i)
    }, 0)
  })(i)
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

第二種解法的精髓在於，for 循環中每一圈我們都調用一個 IIFE，並將當圈的 i 值作為參數傳入，所以實際上每一圈都會創造出一個新的函數(也就是一個新的作用域)，然後每個作用域的參數 i 都將保存當圈的 i 值。

在 for 循環的例子裡面，我們透過 IIFE 來模擬塊級作用域的表現，也就是我們可以`使用 IIFE 來建立塊級作用域`

### PrimeCreator 質數(素數)製造機

```js
const PrimeCreator = (function () {
  let primes = [2, 3, 5]
  let index = -1

  function reset() {
    index = -1
  }

  function nextPrime() {
    index++
    if (index >= primes.length) {
      createNextPrime()
    }
    return primes[index]
  }

  function createNextPrime() {
    let nextPrime = primes[primes.length - 1] + 1
    let isPrime = false
    while (!isPrime) {
      isPrime = true
      for (let prime of primes) {
        if (nextPrime % prime === 0) {
          nextPrime++
          isPrime = false
          break
        }
      }
      if (isPrime) {
        break
      }
    }
    primes.push(nextPrime)
  }

  return {
    reset,
    nextPrime
  }
})()
```

這邊使用 IIFE 保存了一個質數列表，上次調用時建造過的質數會被保存在 `primes` 變量，
如過調用 `reset` 後，下次遍歷的時候就不需要重新計算，可以直接提取上次計算出來的結果。

這邊的重點不在於這個函數本身，而只是一個思想。當我們需要某個方法，而他可能需要保存自己的私有變量，或是索引表(Hash Table、Map Structure 等)，我們可以透過閉包的特性將一些狀態封裝到 IIFE 內部。這就有很多應用，如某某製造機(保存索引表，或是上次調用後的遺產)、狀態機(保存內部狀態)，以及後面將要提到的模塊加載器(保存各模塊又能夠區隔命名空間和作用域)

### Module 模塊化

接下來第三個例子我們來嘗試使用`閉包(closure)`來建立`模塊化(modules)`的概念。

由於在 ES6 引入 `import/export` 之前，JS 一直都沒辦法實現真正的模塊化，因此遠古時代的人們就想出了一個辦法，使用閉包來模擬模塊化的情形：

```js
const SquareModule = (function () {
  var _length = 100

  function setLength(length) {
    console.log(`set length = ${length}`)
    _length = length
  }

  function area() {
    return _length * _length
  }

  return {
    setLength,
    area
  }
})()

console.log(`area = ${SquareModule.area()}`)
SquareModule.setLength(20)
console.log(`area = ${SquareModule.area()}`)

// output:
// area = 10000
// set length = 20
// area = 400
```

我們透過 IIFE 建立一個內部作用域來模擬生成一個模塊，模塊內部的屬性(變量)對於外部是不可見的。

### Module Loader 模塊加載器

接下來是閉包最重要的應用，就是可以用來模擬 ES6 的模塊化系統：

```js
// 模塊管理器
const ModuleManager = (function Manager() {
  // 保存模塊，外部不可訪問
  const modules = {}

  // 加載模塊函數
  // name: 模塊名
  // deps: 依賴模塊名的數組
  // 模塊加載器: 動態解析模塊
  function load(name, deps, impl) {
    for (let i = 0; i < deps.length; i++) {
      deps[i] = modules[deps[i]]
    }
    modules[name] = impl.apply(null, deps)
  }

  // 提取模塊的接口
  function get(name) {
    return modules[name]
  }

  return {
    load,
    get
  }
})()

ModuleManager.load('Add', [], function () {
  function add(x, y) {
    return x + y
  }

  return {
    add
  }
})

ModuleManager.load('Mul', ['Add'], function (Add) {
  function mul(x, y) {
    let sum = 0
    while (y-- > 0) {
      sum = Add.add(sum, x)
    }
    return sum
  }

  return {
    mul
  }
})

const Add = ModuleManager.get('Add')
const Mul = ModuleManager.get('Mul')
console.log(Add.add(1, 2))
console.log(Mul.mul(3, 4))

// output:
// 3
// 12
```

這邊我們透過閉包隱藏真正保存模塊的對象(`modules` 變量)，並且加載器接收的第三個參數為其他模塊的生成函數，我們也可以在 `impl` 函數內部保存模塊自己的狀態。其實上面的寫法已經非常接近於 ES6 的模塊化系統了，非常值得參考（兩者具體比較如下）

- 閉包寫法

```js
ModuleManager.load('Mul', ['Add'], function (Add) {
  function mul(x, y) {
    let sum = 0
    while (y-- > 0) {
      sum = Add.add(sum, x)
    }
    return sum
  }

  return {
    mul
  }
})
```

- ES6 Module 寫法

```js
// Mul.js
import { add } from 'Add'

function mul(x, y) {
  let sum = 0
  while (y-- > 0) {
    sum = add(sum, x)
  }
  return sum
}

export { mul }
```

# 結語

本篇介紹了`閉包(closure)`的概念，以及數個應用的概念和示例。即便 ES6 以及其他模塊化技術已經非常普及，了解閉包的運行機制和理念也是非常重要。
