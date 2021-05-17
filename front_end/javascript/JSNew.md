# JS 基础: 从 5 种创建对象的方式看 new 操作符的作用与实现

@[TOC](文章目录)

<!-- TOC -->

- [JS 基础: 从 5 种创建对象的方式看 new 操作符的作用与实现](#js-基础-从-5-种创建对象的方式看-new-操作符的作用与实现)
- [前言](#前言)
- [正文](#正文)
  - [创建 Object 对象的五种方式](#创建-object-对象的五种方式)
    - [使用 Object.create(proto) 创建对象](#使用-objectcreateproto-创建对象)
  - [自定义类型](#自定义类型)
    - [函数属性 & 原型](#函数属性--原型)
    - [创建自定义类型对象](#创建自定义类型对象)
    - [自定义类型小结](#自定义类型小结)
  - [再谈对象的创建](#再谈对象的创建)
    - [1. 构造函数的返回值](#1-构造函数的返回值)
    - [2. 自定义对象修改原型后再创建](#2-自定义对象修改原型后再创建)
  - [new 关键字实现](#new-关键字实现)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码参考](#完整代码参考)

<!-- /TOC -->

# 前言

本篇要来谈谈 JS 语言中的创建普通 Object 对象的五种方法，并引出类型相关的属性与 **new** 关键字在创建对象的时候所表现的行为，最后给出一个 **new** 关键字的实现。

# 正文

## 创建 Object 对象的五种方式

首先第一段我们先来熟悉一下，在 JS 语言中我们能够使用的 5 种创建对象的方式

- `createObject.js`

```js
const obj1 = {}
const obj2 = Object()
const obj3 = new Object()
const obj4 = Object.create({})
const obj5 = Object.create(null)

console.log('obj1 = {}', obj1)
console.log('obj2 = Object()', obj2)
console.log('obj3 = new Object()', obj3)
console.log('obj4 = Object.create()', obj4)
console.log('obj5 = Object.create(null)', obj5)

console.log(
  `obj1.__proto__ === Object.prototype = `,
  obj1.__proto__ === Object.prototype
)
console.log(
  `obj2.__proto__ === Object.prototype = `,
  obj2.__proto__ === Object.prototype
)
console.log(
  `obj3.__proto__ === Object.prototype = `,
  obj3.__proto__ === Object.prototype
)
console.log(
  `obj4.__proto__.__proto__ === Object.prototype = `,
  obj4.__proto__.__proto__ === Object.prototype
)
console.log(`obj5.__proto__ = `, obj5.__proto__)
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_new_createObject.png)

五种创建方式分别对应 `obj1 ~ obj5`，我们可以看到五种方式都创建出一个新的对象，区别在于原型对象(部分浏览器透过 `__proto__` 属性来对外暴露)，前三种方式都指向了 `Object.prototype` 属性，也就是之前 <a href="https://blog.csdn.net/weixin_44691608/article/details/106722970">JS 基礎：Prototype Chain 原型鏈</a> 提过的：

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_new_object_prototype.png)

构造函数、原型对象与实例对象之间的关系。

### 使用 Object.create(proto) 创建对象

特别注意的是后面两种创建对象的方式：

- 对于 `obj4`：我们发现 `obj4.__proto__` 并不直接指向 `Object.prototype` 而是需要再访问一层 `__proto__` 才会是 `Object.prototype`，其实就是下面这样一个结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_new_object_prototype4.png)

> Object.create({}) 将以参数作为原型创建一个新的对象

- 对于 `obj5`：我们传入 `null` 作为原型，所以当然就不存在所谓的原型对象，`__proto__` 就变成了 `undefined`

## 自定义类型

上面我们创建的是内置的 Object 类型的对象，下面我们定义一个自定义类型 MyClass：

- `createMyClass.js`

```js
let uid = 0

function MyClass() {
  this.id = uid++
}
```

下面先上代码再来一一解说

```js
console.group(`Reflect.ownKeys(MyClass)`, Reflect.ownKeys(MyClass))
for (const key of Reflect.ownKeys(MyClass)) {
  console.log(`MyClass.${key} = `, MyClass[key])
}
console.groupEnd()
```

```js
console.group('MyClass test')
console.log(
  `MyClass.prototype.constructor === MyClass: ${
    MyClass.prototype.constructor === MyClass
  }`
)
console.log(
  `MyClass.prototype.__proto__ === Object.prototype: ${
    MyClass.prototype.__proto__ === Object.prototype
  }`
)
console.groupEnd()
```

```js
const c1 = new MyClass()

const c2 = {}
MyClass.call(c2)

const c3 = Object.create(MyClass.prototype)

const c4 = Object.create(null)
MyClass.call(c4)

console.group('MyClass create')
console.log(`c1 = new MyClass()`, c1)
console.log(`c2 = MyClass.call({})`, c2)
console.log(`c3 = Object.create(MyClass.prototype)`, c3)
console.log(`c4 = MyClass.call(Object.create(null))`, c4)

console.log(
  `c1.__proto__ === MyClass.prototype: ${c1.__proto__ === MyClass.prototype}`
)
console.log(
  `c2.__proto__ === Object.prototype: ${c2.__proto__ === Object.prototype}`
)
console.log(
  `c3.__proto__ === MyClass.prototype: ${c3.__proto__ === MyClass.prototype}`
)
console.log(`c4.__proto__ === undefined: ${c4.__proto__ === undefined}`)
console.groupEnd()
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_new_createMyClass.png)

### 函数属性 & 原型

首先第一步我们先来看看作为一个函数会存在哪些属性(第一块输出)

内置函数类型有以下属性：

- length: 形式参数个数
- name: 函数名称
- arguments: 参数列表
- caller: 调用主体
- prototype: 原型对象

这边我们就是简单定义了一个函数后面要作为构造函数使用，所以 `prototype` 指向的原型对象也很单纯，就跟前面的 Object 关系雷同

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_new_myClass_prototype.png)

### 创建自定义类型对象

接下来是 `obj1 ~ obj4` 的对象创建

- `obj1`：第一种就是直接使用 `new MyClass()` 来创建对象，也是平常我们最常使用的方式
- `obj2`：第二种稍微改变以下，我们先创建一个普通对象 `{}`，然后在调用构造函数对其进行初始化。其实这就跟以前描述过的 **new** 关键字的行为很像(`new MyClass` 会在 new 关键字创建对象、使用 MyClass 构造函数初始化)，然而下面我们可以看到，`obj2.__proto__` 的原型直接指向 `Object.prototpye` 了，也就是说类型信息(MyClass 与其原型)并没有如我们所想的出现在 `obj2` 的原型链上，也就是说对 `obj2` 调用 `MyClass` 的原型方法是无效的
- `obj3`：既然第二种没有原型，那我们就采用上面提过的 `Object.create` 的方式来创建对象吧；但是我们从输出中看到类型是对了(`__proto__` 指向 `MyClass.prototype`)，但是却不存在 id 也就是仅仅添加了类型信息而没有执行构造函数，这边我们已经隐隐约约看出来 **new** 关键字要怎么实现了，后面再提
- `obj4`：第四种与第二种类似，区别在于我们的原始对象使用 `Object.create(null)`，也就是一个不包括原型(类型信息)的对象来作为载体

### 自定义类型小结

到此我们已经大致明白一个自定义类型与创建对象的时候类型信息(原型对象)要如何指定和进行初始化了

- `Object.create(proto)` 能根据我们指定的原型来创建对象(返回对象的 `__proto__` 原型指向参数的原型)，但不会对对象进行初始化
- `Ctor.call(obj)` 透过以某个对象作为上下文调用构造函数，相当于是对该对象进行初始化

## 再谈对象的创建

在进入最终 **new** 关键字的实现之前，我们再来探索以下 `prototype` 原型对象之间的关系

### 1. 构造函数的返回值

首先我们先来看看当我们定义一个函数作为构造函数的时候，返回值会对 **new** 关键字造成什么影响

- `createMyClassWithReturn.js`

```js
let uid = 0

function MyClass(index = 0) {
  this.id = uid++
  // return primitive
  return [0, false, 'string', {}, [], null, undefined, function () {}][index]
}

const c0 = new MyClass(0)
const c1 = new MyClass(1)
const c2 = new MyClass(2)
const c3 = new MyClass(3)
const c4 = new MyClass(4)
const c5 = new MyClass(5)
const c6 = new MyClass(6)
const c7 = new MyClass(7)

console.log(`return 0: `, c0)
console.log(`return false: `, c1)
console.log(`return 'string': `, c2)
console.log(`return {}: `, c3)
console.log(`return []: `, c4)
console.log(`return null: `, c5)
console.log(`return undefined: `, c6)
console.log(`return function() {}: `, c7)
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_new_createMyClassWithReturn.png)

我们可以看到，参数为**基本类型(Number, Boolean, String, null, undefined)**的时候不影响 new 的对象创建，然而返回**引用类型(Object, Array, Function)**的时候，虽然会执行构造函数(根据 id 确实递增了)，但是会返回 return 之后的引用类型对象(这里其实就可以添加一个 `Proxy` 的妙用，本篇就不多做解释)

所以我们就可以总结出：

> 构造函数的返回值为**引用类型**的时候会改变返回对象

### 2. 自定义对象修改原型后再创建

第二个是当我们修改 `prototype.constructor` 的时候，对于不同的对象创建会有什么影响：

- `createMyClassWithProto.js`

```js
let uid = 0

function MyClass() {
  this.id = uid++
}

function OtherClass() {
  return {}
}

MyClass.prototype.constructor = OtherClass
```

```js
console.group(`Reflect.ownKeys(MyClass)`, Reflect.ownKeys(MyClass))
for (const key of Reflect.ownKeys(MyClass)) {
  console.log(`MyClass.${key} = `, MyClass[key])
}
console.groupEnd()

console.group(`Reflect.ownKeys(OtherClass)`, Reflect.ownKeys(OtherClass))
for (const key of Reflect.ownKeys(OtherClass)) {
  console.log(`MyClass.${key} = `, OtherClass[key])
}
console.groupEnd()

console.group('compare MyClass & OtherClass')
console.log(
  `MyClass.prototype === OtherClass.prototype: ${
    MyClass.prototype === OtherClass.prototype
  }`
)
console.groupEnd()
```

```js

const c1 = new MyClass()

const c2 = {}
MyClass.call(c2)

const c3 = Object.create(MyClass.prototype)

const c4 = Object.create(null)
MyClass.call(c4)

console.group('MyClass create')
console.log(`c1 = new MyClass()`, c1)
console.log(`c2 = MyClass.call({})`, c2)
console.log(`c3 = Object.create(MyClass.prototype)`, c3)
console.log(`c4 = MyClass.call(Object.create(null))`, c4)

console.log(
  `c1.__proto__ === MyClass.prototype: ${c1.__proto__ === MyClass.prototype}`
)
console.log(
  `c2.__proto__ === Object.prototype: ${c2.__proto__ === Object.prototype}`
)
console.log(
  `c3.__proto__ === MyClass.prototype: ${c3.__proto__ === MyClass.prototype}`
)
console.log(`c3 instanceof MyClass: ${c3 instanceof MyClass}`)
console.log(`c3 instanceof OtherClass: ${c3 instanceof OtherClass}`)
console.log(`c4.__proto__ === undefined: ${c4.__proto__ === undefined}`)
console.groupEnd()
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_new_createMyClassWithProto.png)

我们就不对代码一一讲解了，直接来看四个对象的差异

- `obj1`: 使用 new 关键字创建的对象，有 `id` 属性，表示确实是经过 MyClass 方法进行初始化的，然而其原型的构造函数(`MyClass.prototype.constructor`)却指向了 OtherClass，同时上面的输出我们可以看到，`MyClass.prototype` 与 `OtherClass.prototype` 是不相同的，仅仅只是指向了相同的构造函数(这里的意思就是如果我们再利用 `obj1` 的原型来创建新的对象的时候，反而会使用 OtherClass 函数进行对象初始化)。两个类和原型的关系如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_new_myClass_otherClass_prototype.png)

- `obj2`: 第二种跟上面一样，仅仅将构造函数作为初始化用，所以 `obj2.__proto__` 还是指向了 `Objec.prototype` 没有附带类型信息
- `obj3`: 第三种使用 `Object.create(MyClass.prototype)` 来构造对象，但是没有进行初始化(注意：这边可以选择使用 MyClass 函数或是 OtherClass 函数进行初始化都可以，不影响对象附带的类型信息)
- `obj4`: 第四种则是 `Object.create(null)` 不附带类型信息只进行初始化

## new 关键字实现

其实上面几个例子已经向我们演示了，就跟 [自定义类型小结](#自定义类型小结) 提到的一样，我们使用 `Object.create(proto)` 来制定类型信息；`Ctor.call(obj)` 来执行对象初始化(构造函数调用)

下面我们就给出最终版本的 **new** 关键字实现与原生关键字的比较

- `newObject.js`

```js
function newObject(Ctor, ...args) {
  const obj = Object.create(Ctor.prototype)
  const ret = Ctor.apply(obj, args)
  return ret instanceof Object ? ret : obj
}

let uid = 0

function MyClass() {
  this.id = uid++
}

const c1 = new MyClass()
const c2 = newObject(MyClass)
console.log(`c1 = new MyClass()`, c1)
console.log(`c2 = newObject(MyClass)`, c2)

console.log(
  `c1.__proto__ === MyClass.prototype: ${c1.__proto__ === MyClass.prototype}`
)
console.log(
  `c2.__proto__ === MyClass.prototype: ${c2.__proto__ === MyClass.prototype}`
)
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_new_newObject.png)

我们实现的 `newObject` 方法(模拟 new 关键字)只做了两件事：

1. 使用构造函数原型创建对象：`const obj = Object.create(Ctor.prototype)`
2. 使用构造函数初始化对象：`const ret = Ctor.apply(obj, args)`
3. 最后返回的时候检查构造函数的返回对象：`return ret instanceof Object ? ret : obj`
   1. 引用类型则返回
   2. 非引用类型则忽略

# 结语

本篇从原型链上和原型附带的类型信息角度来研究 **new** 关键字的作用和实现，其实还是比较简单易懂的，主要是为下一篇的 JS 继承做铺垫。

# 其他资源

## 参考连接

<table>
  <tr>
    <td>js中new操作符做了什么并实现自己的new操作符</td>
    <td><a href="https://blog.csdn.net/weixin_43911758/article/details/116841042">https://blog.csdn.net/weixin_43911758/article/details/116841042</a></td>
  </tr>
  <tr>
    <td>JS 实现new 关键字</td>
    <td><a href="https://www.cnblogs.com/xingguozhiming/p/11000118.html">https://www.cnblogs.com/xingguozhiming/p/11000118.html</a></td>
  </tr>
  <tr>
    <td>Object.create()</td>
    <td><a href="https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create">https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create</a></td>
  </tr>
</table>

## 完整代码参考

<a href="https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_new">https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_new</a>
