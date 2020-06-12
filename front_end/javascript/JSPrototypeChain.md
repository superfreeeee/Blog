# JS 基礎：Prototype Chain 原型鏈

@[TOC](文章目錄)

<!-- TOC -->

- [JS 基礎：Prototype Chain 原型鏈](#js-基礎prototype-chain-原型鏈)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Object 對象創建](#object-對象創建)
    - [直接量 `{}`](#直接量-)
    - [內置構造函數 `new Object()`](#內置構造函數-new-object)
    - [使用 `Object.create(proto, propertiesObject)`](#使用-objectcreateproto-propertiesobject)
  - [Class 類型](#class-類型)
  - [Prototype 原型](#prototype-原型)
    - [Function without prototype](#function-without-prototype)
    - [Function with prototype](#function-with-prototype)
    - [Concept 概念](#concept-概念)
      - [Sample](#sample)
    - [Prototypte is an Object 原型也是對象](#prototypte-is-an-object-原型也是對象)
  - [Prototype Chain 原型鏈（也就是所謂的`繼承Inherit`啦！）](#prototype-chain-原型鏈也就是所謂的繼承inherit啦)
    - [Sample](#sample-1)
  - [Built-in Classes 內置類(內置的構造函數)](#built-in-classes-內置類內置的構造函數)
  - [補充：ES6 Class](#補充es6-class)
- [結語](#結語)

<!-- /TOC -->

## 簡介

說到面對對象編程(OOP)，當然不能沒有繼承(Inherit)。作為代碼複用最基礎的方式便是透過繼承，將子類透過繼承父類的方法或屬性來擴展對象的能力。對象(Object)類型肯定是 JS 代碼中最常用的類型，那麼 JS 是如何實現繼承的呢？

然而與 Java、C++ 的`基於類型(class-based)`的繼承機制不同，JavaScript 並沒有實作`類型(class)`定義(即便是 ES6 的 `class` 也僅僅是語法糖)。JS 是`基於原型(prototype-based)`的繼承機制，而 JS 中的類型定義其本質就是一個函數。接下來就讓我娓娓道來 JavaSciprt 基於原型的繼承機制吧。

## 參考

<table>
  <tr>
    <td>JavaScript深入之从原型到原型链</td>
    <td><a href="https://github.com/mqyqingfeng/blog/issues/2">https://github.com/mqyqingfeng/blog/issues/2</a></td>
  </tr>
  <tr>
    <td>繼承與原型鏈</td>
    <td><a href="https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Inheritance_and_the_prototype_chain">https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Inheritance_and_the_prototype_chain</a></td>
  </tr>
</table>

# 正文

## Object 對象創建

首先我們先來看看最基本的`對象(Object)`是如何聲明和創建的：

### 直接量 `{}`

```js
const o1 = { name: 'John', age: 18 }
```

### 內置構造函數 `new Object()`

```js
const o1 = new Object()
const o2 = new Object(null)
const o3 = new Object({})
const o4 = new Object(o1) // 這邊傳遞的是引用，o1 和 o4 將指向同一個對象
```

### 使用 `Object.create(proto, propertiesObject)`

```js
// 使用 Object.create
const o6 = Object.create(Object.prototype)
const o7 = Object.create(Object.prototype, {
  name: {
    value: 'John'
  },
  age: {
    value: 18,
    writable: false
  }
})
```

**注意！** `Object.create` 方法非常重要，將作為下面繼承方式的基礎

## Class 類型

有了對象之後，我們創建出來的依舊是一個個獨立的對象，沒有統一的規範和樣式。在面對對象的編程思維(OOP)中，需要定義`類型(Class)`，而所有`對象實例(Instance)`都應該基於類型來創建，創建出來的物件簡稱為對象或是實例(反正就是一個 `Object` 或是稱為 `Instance`)。

但是 JavaScript 裡面根本沒有 `class` 關鍵字(這邊不考慮 ES6 提供的 `class` 語法糖)，沒有辦法像 Java、C++ 那樣聲明定義類型然後創建對象。當然 JS 的對象機制也不是基於類型(class-based)的，而是`基於原型(prototype-based)`的

在 JS 中，`類型(class)`本質上就是一個`函數(Function)`，有趣的是同樣使用 `new` 關鍵字來創建對象實例：

```js
// 聲明 Person 類型，使用 this 來指向對象實例的屬性
function Person() {
  this.name = 'John'
  this.age = 18
}

const person = new Person()
```

可以從上面的形式看出來，作為"類型"的函數就好像這個類型的構造函數一樣，下面貼上 babel 編譯後的代碼讓大家也能了解到 ES6 的 `class` 語法糖是如何轉換回 ES5 以前的代碼的（有興趣的可以自己到<a href="https://babeljs.io/repl">Babel 官方</a>嘗試）：

- ES6 class

```js
class Person {
  constructor() {
    this.name = 'John'
    this.age = 18
  }
}
```

- babel 編譯後

```js
'use strict'

var Person = function Person() {
  this.name = 'John'
  this.age = 18
}
```

我們能夠看到 `Person` 類就被編譯為一個函數，而`構造函數(constructor)`便是這個方法的主函數體

## Prototype 原型

現在我們能夠基於類型創建對象了，接下來我們來了解什麼叫`原型(prototype)`。

 首先我們先來看沒有使用原型的時候，為對象添加方法會發生什麼事：

### Function without prototype

```js
function Person(name) {
  this.name = name
  this.hi = function () {
    console.log(`Hi! I am ${this.name}`)
  }
}
const p1 = new Person('John')
const p2 = new Person('Andy')
p2.hi = function () {
  console.log('new hi function')
}
p1.hi()
p2.hi()

// output:
// Hi! I am John
// new hi function
```

這邊我們發現 p1、p2 的 `hi` 方法是互相獨立的，也就導致我們修改了 p2 的方法之後，p1 的方法還是原來的形式。

然而對於面對對象編程(OOP)來說，同樣類型的方法相同，並且應該只能存在一個函數體，而不是為每個對象創建一個新的方法。這時候我們就必須使用到原型。

### Function with prototype

我們先看範例，再來說明原型的概念：

```js
function Person(name) {
  this.name = name
}
Person.prototype.hi = function () {
  console.log(`Hi! I am ${this.name}`)
}
const p1 = new Person('John')
const p2 = new Person('Andy')
p1.hi()
p2.hi()

// output:
// Hi! I am John
// Hi! I am Andy
```

太神奇了傑克！如此一來，我們透過將 hi 方法變成 `Person` 類型的 `prototype` 對象上，就使得所有基於 `Person` 類型所創建的對象都能調用同樣的方法，也只需要定義一次。那麼話說回來，這個 `prototype` 到底是個什麼玩意兒？

### Concept 概念

在 JS 中，所謂`類型(class)`的本質就是一個`函數(Function)`（上面也提過了），它在`創建對象(new)`的時候將被作為`構造函數(constructor)`來調用。而一個`對象(object)`的創建必定離不開三大要素：

1. `構造函數(constructor)`：即定義類型的原函數
2. `原型對象(prototype)`：每個構造函數都會存在一個原型對象(**注意**！原型本身也是一個對象)
3. `對象實例(object)`：真正運行時存活在堆(Heap)中的對象實例

三個要素之間的關係如下圖：

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/three_element.png)

- 橢圓形表示一個存活在`堆(Heap)`中的`對象(object)`
- 六邊表示`構造方法(constructor)`，也就是最原始的類型函數聲明

並且三者之間有如下訪問規則：

1. `__proto__`：對象訪問其原型
2. `prototype`：構造函數訪問其原型
3. `constructor`：對象訪問其構造函數時

通常一個對象最基本只會擁有 `__proto__` 屬性，而其 `constructor` 屬性則是綁定在其`原型對象(prototype)`之上。

接下來我們舉個例子：

#### Sample

- 類型聲明 & 創建對象

```js
function Person(name) {
  this.name = name
}
const person = new Person()
```

- 對應類圖

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/three_element_sample.png)

### Prototypte is an Object 原型也是對象

**注意**！這邊必須記得，所謂的`原型(prototype)`本身也是某一個對象，這對接下來了解`原型鏈`以及`繼承機制`非常重要

## Prototype Chain 原型鏈（也就是所謂的`繼承Inherit`啦！）

接下來我們就可以透過將某個`對象實例(object)`變成新的類型的`原型對象(prototype)`，如此便能夠創造出 JS 語言中的繼承形式，也就是所謂的`原型鏈(prototype chain)`：

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/prototype_chain.png)

如上圖所示，B 繼承 A，C 又繼承 B。

而 JS 在運行時會先查找對象自身的屬性，若訪問的屬性不存在則會在原型鏈上不斷向上查找直到最頂部。接下來我們看一個範例：

### Sample

首先我們先借助 babel 編譯後的一個函數：

```js
function _inheritsLoose(subClass, superClass) {
  subClass.prototype = Object.create(superClass.prototype)
  subClass.prototype.constructor = subClass
  subClass.__proto__ = superClass
}
```

- 說明：為了要實現類型間的繼承，我們需要對子類進行三個步驟：

1. `subClass.prototype = Object.create(superClass.prototype)`：創建一個父類的對象實例並作為子類的原型對象
2. `subClass.prototype.constructor = subClass`：將原型對象的構造函數重新指向子類
3. `subClass.__proto__ = superClass`：將子類的構造函數原型指向父類的構造函數

接下來寫出我們的`類(class 本質上是 funciton)`定義（參考 babel 編譯後代碼）：

```js
const SuperClass = (function () {
  function SuperClass() {
    this.name = 'super class'
  }

  const _proto = SuperClass.prototype

  _proto.hi = function hi() {
    console.log(this.name)
  }

  return SuperClass
})()

const SubClass = (function (_SuperClass) {
  _inheritsLoose(SubClass, _SuperClass)

  function SubClass() {
    let _this

    _this = _SuperClass.call(this) || this
    _this.name = 'sub class'
    return _this
  }

  return SubClass
})(SuperClass)

const sub = new SubClass()
sub.hi()

// output:
// sub class
```

如果上面的寫法太難的話這邊給出一個白話文版本（不過上面的代碼還是建議把它看懂，也是提升自己的代碼素養）：

```js
function SuperClass() {
  this.name = 'super class'
}

SuperClass.prototype.hi = function () {
  console.log(this.name)
}

function SubClass() {
  this.name = 'sub class'
}

SubClass.prototype = Object.create(SuperClass.prototype)
SubClass.prototype.constructor = SubClass
SubClass.__proto__ = SuperClass

const sub = new SubClass()
sub.hi()

// output:
// sub class
```

下面用圖來表示繼承的步驟：

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/prototype_chain_sample.png)

示例代碼中調用的 `hi` 方法步驟：

1. `sub` 對象中查找，沒找到對應函數
2. 再到 `SubClass.prototype`，也就是 `sub` 的原型對象中查找，還是沒有
3. 最後到 `SuperClass.prototype` 中找到對應函數並調用，同時 `SuperClass.prototype` 就是 `SubClass.prototype` 的原型

## Built-in Classes 內置類(內置的構造函數)

最後來介紹一下 JS 的內置函數。平常寫代碼的時候可能會用過如 `Boolean`、`Number`、`Object`、`Array`、`RegExp` 等內置函數，而這些內置函數的關係又是如何呢？話不多說先上圖：

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/built_in_relationship.png)

我們發現`內置類`與我們自己定義的類型是擁有同樣層級的原型鏈的，謎底揭曉（好吧其實也不是什麼大秘密）

這也是為什麼所有對象必定都有一個 `toString` 方法，因為他定義在 `Object.prototype` 上，而 `Object.prototype` 就是所有對象的原型鏈的根源，他本身是沒有原型的(`Object.prototype.__proto__ === null`)，並且所有構造函數的原型都是 `Function.prototype`(`Function.__proto__ === Function.prototype` 的定義能夠保證一致性)

## 補充：ES6 Class

最後貼上一段 ES6 的 Class 寫法，以及 babel 編譯過後的結果供大家參考：

- ES6 class

```js
class A {
  constructor() {
    this.a = 'a'
  }

  showDetail() {
    console.log(`this.a = ${this.a}`)
  }
}

class B extends A {
  constructor() {
    super()
    this.b = 'b'
  }

  showDetail() {
    super.showDetail()
    console.log(`this.b = ${this.b}`)
  }
}

class C extends B {
  constructor() {
    super()
    this.a = 'ca'
    this.b = 'cb'
  }

  showDetail() {
    super.showDetail()
  }
}

const a = new A()
const b = new B()
const c = new C()
console.log('invoke a.show')
a.showDetail()
console.log('invoke b.show')
b.showDetail()
console.log('invoke c.show')
c.showDetail()

// output:
// invoke a.show
// this.a = a
// invoke b.show
// this.a = a
// this.b = b
// invoke c.show
// this.a = ca
// this.b = cb
```

- babel 編譯後

```js
'use strict'

function _inheritsLoose(subClass, superClass) {
  subClass.prototype = Object.create(superClass.prototype)
  subClass.prototype.constructor = subClass
  subClass.__proto__ = superClass
}

var A = /*#__PURE__*/ (function () {
  function A() {
    this.a = 'a'
  }

  var _proto = A.prototype

  _proto.showDetail = function showDetail() {
    console.log('this.a = ' + this.a)
  }

  return A
})()

var B = /*#__PURE__*/ (function (_A) {
  _inheritsLoose(B, _A)

  function B() {
    var _this

    _this = _A.call(this) || this
    _this.b = 'b'
    return _this
  }

  var _proto2 = B.prototype

  _proto2.showDetail = function showDetail() {
    _A.prototype.showDetail.call(this)

    console.log('this.b = ' + this.b)
  }

  return B
})(A)

var C = /*#__PURE__*/ (function (_B) {
  _inheritsLoose(C, _B)

  function C() {
    var _this2

    _this2 = _B.call(this) || this
    _this2.a = 'ca'
    _this2.b = 'cb'
    return _this2
  }

  var _proto3 = C.prototype

  _proto3.showDetail = function showDetail() {
    _B.prototype.showDetail.call(this)
  }

  return C
})(B)

var a = new A()
var b = new B()
var c = new C()
console.log('invoke a.show')
a.showDetail()
console.log('invoke b.show')
b.showDetail()
console.log('invoke c.show')
c.showDetail()
```

# 結語

本篇介紹了原型鏈，也是 JS 面對對象編程的基礎。雖然與 Java、C++ 等基於類型的繼承機制不同，但是 JavaScript 的原型鏈也有其自己的好處。完全搞清楚原型鏈是身為一個 JavaScript 工程師必備的基礎技能之一，唯有暸解語言最基礎的能力，才能寫出更穩定而健全的代碼。
