# JS 基础: JavaScript 中 4 种继承方式 & instanceof 实现

@[TOC](文章目录)

<!-- TOC -->

- [JS 基础: JavaScript 中 4 种继承方式 & instanceof 实现](#js-基础-javascript-中-4-种继承方式--instanceof-实现)
- [前言](#前言)
- [正文](#正文)
  - [1. 借用构造函数的继承](#1-借用构造函数的继承)
  - [2. 借用原型对象的继承](#2-借用原型对象的继承)
  - [3. 创建实例对象作为原型的继承](#3-创建实例对象作为原型的继承)
  - [4. 创建未初始化实例作为原型的继承](#4-创建未初始化实例作为原型的继承)
  - [Babel 编译后的继承](#babel-编译后的继承)
  - [instanceof 关键字实现](#instanceof-关键字实现)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码参考](#完整代码参考)

<!-- /TOC -->

# 前言

如果对于 JS 的原型链(prototype chain)与类型判断还不太了解的，推荐你先把下面两篇过了再回来看，会比较了解本篇的原理和目的

- <a href="https://blog.csdn.net/weixin_44691608/article/details/106722970">JS 基礎：Prototype Chain 原型鏈</a>
- <a href="https://blog.csdn.net/weixin_44691608/article/details/110741112">JS 基础: typeof & instanceof 类型检查</a>

同时，前一篇 <a href="https://blog.csdn.net/weixin_44691608/article/details/116931811">JS 基础: 从 5 种创建对象的方式看 new 操作符的作用与实现</a> 我们对与对象的创建进行深度的探讨，也稍微了解几种对象创建的方式。接下来本篇将以实现 JS 继承为目标展开讨论，最后额外给出一个 `instanceof` 关键字的实现

# 正文

下面我们循序渐进给出四个版本的继承实现，最后给出 `instanceof` 的实现用于检测继承类型并加深对原型链的认识

## 1. 借用构造函数的继承

第一种方法我们透过在子类构造函数内调用父类的构造函数来二次初始化对象：

- `inherit1_borrow_constructor.js`

```js
function SuperClass() {
  this.name = 'I am super class'
}

SuperClass.prototype.greeting = function () {
  console.log('super class prototype greeting')
}

function SubClass() {
  SuperClass.call(this)
}

const sub = new SubClass()

console.log('sub', sub)
console.log(`sub.constructor === SubClass: ${sub.constructor === SubClass}`)
console.log(
  `sub.__proto__ === SubClass.prototype: ${
    sub.__proto__ === SubClass.prototype
  }`
)
console.log(
  `sub.__proto__.__proto__ === Object.prototype: ${
    sub.__proto__.__proto__ === Object.prototype
  }`
)
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_inherit_instanceof_implement_1_borrow_constructor_console.png)

从输出我们可以看到，创建出来的对象虽然是 SubClass 类型没错，对象也正确的进行初始化了；但是我们发现对象中完全没有包含父类的类型信息(`sub.__proto__.__proto__` 直接指向 `Object.prototype`)，也就是说其实实际上的原型(类型)关系如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_inherit_instanceof_implement_1_borrow_constructor.png)

子类对象仅仅只是借用了一下父类的构造函数，而实例本身的原型链上则完全不存在父类的任何信息，这时候父类的原型方法(通常透过 `SuperClass.prototype.Xxx = function() {/* ... */}` 定义)也是完全不能调用的，显然这不是我们想要的继承(当然如果你要实现的就是单纯的对象初始化，同时实例方法在父类中使用 `this.Xxx = function () {}` 引入也未尝不可)

## 2. 借用原型对象的继承

既然我们希望构建完后能附带父类信息(原型链)，那就给他父类的原型，代码如下：

- `inherit2_borrow_prototype.js`

```js
function SuperClass() {
  this.name = 'I am super class'
}

SuperClass.prototype.greeting = function () {
  console.log('super class prototype greeting')
}

function SubClass() {}

SubClass.prototype = SuperClass.prototype

const sub = new SubClass()

console.log('sub', sub)
console.log(`sub.constructor === SuperClass: ${sub.constructor === SuperClass}`)
console.log(
  `sub.__proto__ === SubClass.prototype: ${
    sub.__proto__ === SubClass.prototype
  }`
)
console.log(
  `sub.__proto__ === SuperClass.prototype: ${
    sub.__proto__ === SuperClass.prototype
  }`
)
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_inherit_instanceof_implement_2_borrow_prototype_console.png)

我们从输出看到，创建实例的原型(`__proto__` 属性)直接指向了 `SuperClass.prototype`(确实我们也是这么写的 `SubClass.prototype = SuperClass.prototype`)。看似也继承了 SuperClass 的原型方法 `greeting`，但是直接借用了父类的原型对象其实是产生了下面的这种结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_inherit_instanceof_implement_2_borrow_prototype.png)

如果后续我们想要为 SubClass 添加原型方法其实是会直接修改 `SuperClass.prototype` 对象，同时进而影响到所有其他基于父类 SuperClass 类型构建的对象。所以从这边我们发现：我们既需要原型对象能附带父类的原型信息(接到父类的原型链上)，同时我们又希望子类能有自己的一个独立于父类原型的原型对象，下面我们来看看第三种实现。

## 3. 创建实例对象作为原型的继承

既然我们要保留父类的原型信息，又要创建一个子类专属的原型对象，那我们可以选择使用父类直接构造一个对象实例来作为子类的原型对象：

- `inherit3_instance_prototype.js`

```js
function SuperClass() {
  this.name = 'I am super class'
}

SuperClass.prototype.greeting = function () {
  console.log('super class prototype greeting')
}

function SubClass() {}

SubClass.prototype = new SuperClass()

const sub = new SubClass()

console.log('sub', sub)
console.log(`sub.constructor === SuperClass: ${sub.constructor === SuperClass}`)
console.log(
  `sub.__proto__ === SubClass.prototype: ${
    sub.__proto__ === SubClass.prototype
  }`
)
console.log(
  `sub.__proto__.__proto__ === SuperClass.prototype: ${
    sub.__proto__.__proto__ === SuperClass.prototype
  }`
)
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_inherit_instanceof_implement_3_instance_prototype_console.png)

这种实现的对象关系结构如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_inherit_instanceof_implement_3_instance_prototype.png)

看起来好像一切都如我们所想的，原型链上保留了父类的原型(继承了父类的原型方法)，同时有子类专属的原型对象保留原型方法。但是这种方案其实还是有两个小漏洞：

1. 子类原型的构造函数还是指向父类构造函数(`SubClass.prototype.constructor === SuperClass`)：这样就不对了，既然是子类的原型那构造函数当然要指向自己才对嘛
2. 作为原型的父类实例已经初始化过：很多时候我们的构造函数会对一些实例变量或是实例方法进行初始化，然而在一般的继承场景之中，除非我们显式的声明要调用父类的构造方法进行初始化(如在 java 中的 `super` 关键字)，否则我们通常希望使用一个纯粹而未进行实例变量初始化的对象来作为我们的原型对象

要达成这种效果也很简单，就是用 [上一篇](https://blog.csdn.net/weixin_44691608/article/details/116931811) 提过的 `Object.create` 方法来进行原型对象的创建

## 4. 创建未初始化实例作为原型的继承

最终的继承实现我们就选择使用 `Object.create` 方法来创建我们的子类原型对象

- `inherit4_final.js`

```js
function SuperClass() {
  this.name = 'I am super class'
}

SuperClass.prototype.greeting = function () {
  console.log('super class prototype greeting')
}

function SubClass() {}

SubClass.prototype = Object.create(SuperClass.prototype)
SubClass.prototype.constructor = SubClass

const sub = new SubClass()

console.log('sub', sub)
console.log(`sub.constructor === SubClass: ${sub.constructor === SubClass}`)
console.log(
  `sub.__proto__ === SubClass.prototype: ${
    sub.__proto__ === SubClass.prototype
  }`
)
console.log(
  `sub.__proto__.__proto__ === SuperClass.prototype: ${
    sub.__proto__.__proto__ === SuperClass.prototype
  }`
)
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_inherit_instanceof_implement_4_final_console.png)

对象结构如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_inherit_instanceof_implement_4_final.png)

最重要的是以下几个改变：

- 透过 `Object.create(SuperClass.prototype)` 创建的原型对象会以 `SuperClass.prototype` 作为原型创建对象，同时并不会对对象调用构造函数来初始化
- 更改原型对象后同时会改变 `constructor` 构造函数的指向，使用 `SubClass.prototype.constructor = SubClass` 重新将构造函数指回当前类型即可

## Babel 编译后的继承

最后我们来看看 Babel 编译 ES6 的 Class 定义的类型是如何转为 ES5 的实现的：

- `/src/index.js`

```js
class A {
  constructor() {
    this.name = 'A'
  }
}

class B extends A {
  constructor() {
    super()
    this.name = 'B'
  }
}
```

- `/lib/index.js`

```js
'use strict'

function _inheritsLoose(subClass, superClass) {
  subClass.prototype = Object.create(superClass.prototype)
  subClass.prototype.constructor = subClass
  _setPrototypeOf(subClass, superClass)
}

function _setPrototypeOf(o, p) {
  _setPrototypeOf =
    Object.setPrototypeOf ||
    function _setPrototypeOf(o, p) {
      o.__proto__ = p
      return o
    }
  return _setPrototypeOf(o, p)
}

var A = function A() {
  this.name = 'A'
}

var B = /*#__PURE__*/ (function (_A) {
  _inheritsLoose(B, _A)

  function B() {
    var _this

    _this = _A.call(this) || this
    _this.name = 'B'
    return _this
  }

  return B
})(A)
```

我们定义了一个 $A$ 作为父类，一个 $B$ 类型作为子类，看到最主要的核心其实就是 `_inheritsLoose` 方法(使用 loose mode 看起来比较像是人写的能看懂hh)的实现，其实就跟我们实现的[最终版本](#4-创建未初始化实例作为原型的继承)几乎一样的，而 `_setPrototypeOf` 方法则是额外对构造函数的 `__proto__` 原型进行修正(其实一般函数/构造函数作为普通对象的时候，其 `__proto__` 原型其实就是指向 `Function.prototype` 所以这边差异不大)

第二个需要注意的是，在 ES6 中规定子类必须使用 `super()` 关键字调用父类的构造方法进行对象初始化，其实这就对应了编译后的 `_this = _A.call(this) || this`；如果没有调用 `super`，`_this` 就会变成 `undefined` 使对象的创建出现异常

## instanceof 关键字实现

最后的最后，我们知道了在 JS 中类型的继承是透过原型链来实现，那么关于类型检查的方法肯定绕不过 `instanceof` 关键字的实现：

> instanceof 关键字就是寻找对象的原型链上是否存在目标类型的原型对象

简而言之就是不断向上查找目标对象的 `__proto__` 指向的原型对象，是否与目标类型(构造函数指向的原型对象 `Ctor.prototype`)相同，代码如下：

- `instanceof.js`

```js
function _instanceof(obj, target) {
  const targetProto = target.prototype
  let proto = obj.__proto__
  while (proto) {
    if (proto === targetProto) return true
    proto = proto.__proto__
  }
  return false
}

class A {}

class B extends A {}

class C extends B {}

class D extends A {}

group('test a', () => {
  const a = new A()
  console.log(`a instanceof A: ${a instanceof A},\t_instanceof(a, A): ${_instanceof(a, A)}`)
  console.log(`a instanceof B: ${a instanceof B},\t_instanceof(a, B): ${_instanceof(a, B)}`)
  console.log(`a instanceof C: ${a instanceof C},\t_instanceof(a, C): ${_instanceof(a, C)}`)
  console.log(`a instanceof D: ${a instanceof D},\t_instanceof(a, D): ${_instanceof(a, D)}`)
})

group('test b', () => {
  const b = new B()
  console.log(`b instanceof A: ${b instanceof A},\t_instanceof(b, A): ${_instanceof(b, A)}`)
  console.log(`b instanceof B: ${b instanceof B},\t_instanceof(b, B): ${_instanceof(b, B)}`)
  console.log(`b instanceof C: ${b instanceof C},\t_instanceof(b, C): ${_instanceof(b, C)}`)
  console.log(`b instanceof D: ${b instanceof D},\t_instanceof(b, D): ${_instanceof(b, D)}`)
})

group('test c', () => {
  const c = new C()
  console.log(`c instanceof A: ${c instanceof A},\t_instanceof(c, A): ${_instanceof(c, A)}`)
  console.log(`c instanceof B: ${c instanceof B},\t_instanceof(c, B): ${_instanceof(c, B)}`)
  console.log(`c instanceof C: ${c instanceof C},\t_instanceof(c, C): ${_instanceof(c, C)}`)
  console.log(`c instanceof D: ${c instanceof D},\t_instanceof(c, D): ${_instanceof(c, D)}`)
})

group('test d', () => {
  const d = new D()
  console.log(`d instanceof A: ${d instanceof A},\t_instanceof(d, A): ${_instanceof(d, A)}`)
  console.log(`d instanceof B: ${d instanceof B},\t_instanceof(d, B): ${_instanceof(d, B)}`)
  console.log(`d instanceof C: ${d instanceof C},\t_instanceof(d, C): ${_instanceof(d, C)}`)
  console.log(`d instanceof D: ${d instanceof D},\t_instanceof(d, D): ${_instanceof(d, D)}`)
})
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_inherit_instanceof_implement_console.png)

我们可以看到我们实现的 `_instanceof` 方法与原生的 `instanceof` 关键字表现相同，搞定！

# 结语

写过好多 JS 的前端项目，对 JS 了解比较深刻之后再回来从原型链的角度重新诠释并实现 JS 语言的继承机制，确实是比当初懵懵懂懂的时候硬看要轻松许多，希望本篇能帮助读者对于 JS 的原型链、原型对象关系和类型/继承机制有更深刻的认识。

# 其他资源

## 参考连接

<table>
  <tr>
    <td>js继承的6种方式</td>
    <td><a href="https://www.cnblogs.com/ranyonsue/p/11201730.html">https://www.cnblogs.com/ranyonsue/p/11201730.html</a></td>
  </tr>
  <tr>
    <td>JS继承的5种方式（面试）</td>
    <td><a href="https://zhuanlan.zhihu.com/p/81266626">https://zhuanlan.zhihu.com/p/81266626</a></td>
  </tr>
  <tr>
    <td>Object.setPrototypeOf()-MDN</td>
    <td><a href="https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf">https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf</a></td>
  </tr>
  <tr>
    <td>关于Babel 6的 loose mode</td>
    <td><a href="https://blog.csdn.net/weixin_34194359/article/details/88039623">https://blog.csdn.net/weixin_34194359/article/details/88039623</a></td>
  </tr>
</table>

## 完整代码参考

<a href="https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_inherit_instanceof_implement">https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_inherit_instanceof_implement</a>
