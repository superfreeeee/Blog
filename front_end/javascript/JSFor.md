# JS 基础：一次搞懂 for、for...in、for...of 循环

@[TOC](文章目录)

<!-- TOC -->

- [JS 基础：一次搞懂 for、for...in、for...of 循环](#js-基础一次搞懂-forforinforof-循环)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [样例](#样例)
  - [一般 for 循环](#一般-for-循环)
    - [Scope 作用域](#scope-作用域)
  - [for...in 循环](#forin-循环)
    - [`enumerable: false`](#enumerable-false)
    - [模仿 for...in](#模仿-forin)
    - [for...in 小结](#forin-小结)
  - [for...of 循环](#forof-循环)
    - [Symbol.iterator](#symboliterator)
    - [for...of 小结](#forof-小结)
- [结语](#结语)

<!-- /TOC -->

## 简介

在 JavaScript 里面，for 循环有三种使用方式：`for`、`for...in`、`for...of`(ES6 新增)，我们们还可以透过数组的 `forEach` 等方法进行另一种遍历。本篇暂时不加入 Array.prototype 的方法，单纯讨论 `for` 关键字的不同使用形式。

## 参考

<table>
  <tr>
    <td>JavaScript for 循环-菜鸟教程</td>
    <td><a href="https://www.runoob.com/js/js-loop-for.html">https://www.runoob.com/js/js-loop-for.html</a></td>
  </tr>
  <tr>
    <td>for...in-MDN</td>
    <td><a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in">https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in</a></td>
  </tr>
  <tr>
    <td>js中in关键字的使用方法</td>
    <td><a href="https://www.cnblogs.com/memphis-f/p/12073013.html">https://www.cnblogs.com/memphis-f/p/12073013.html</a></td>
  </tr>
</table>

# 正文

## 样例

在介绍三种循环的意义之前，先给出三种形式的使用样例：

```js
// 待遍历变量
const nums = [ 1, 2, 3, 4, 5 ]
const object = { a: 1, b: 2, c: 3 }
// 为 object 添加 Symbol.iterator 供 let...of 使用
object[Symbol.iterator] = function* () {
  for (let prop in this) {
    yield { [prop]: this[prop] }
  }
}

/* 一般 for 循环 */
// 按下标访问数组
console.log('----- for: array -----')
for (let i = 0; i < nums.length; i++) {
  console.log(nums[i])
}

// 按自有属性访问对象键值对
console.log('----- for: object -----')
const objectProps = Object.getOwnPropertyNames(object)
for (let i = 0; i < objectProps.length; i++) {
  console.log(object[objectProps[i]])
}

/* for...in */
// for...in 遍历数组
console.log('----- for...in: array -----')
for (let index in nums) {
  console.log(`nums[${index}] = ${nums[index]}`)
}

// for...in 遍历对象
console.log('----- for...in: object -----')
for (let prop in object) {
  console.log(`object.${prop} = ${object[prop]}`)
}

/* for...of */
// for...of 遍历数组
console.log('----- for...of: array -----')
for (let num of nums) {
  console.log(num)
}

// for...of 遍历对象（透过 Symbol.iterator 方法）
console.log('----- for...of: object(through object[Symbol.iterator]) -----')
for (let item of object) {
  console.log(item)
}
```

- 输出

```
----- for: array -----
1
2
3
4
5
----- for: object -----
1
2
3
----- for...in: array -----
nums[0] = 1
nums[1] = 2
nums[2] = 3
nums[3] = 4
nums[4] = 5
----- for...in: object -----
object.a = 1
object.b = 2
object.c = 3
----- for...of: array -----
1
2
3
4
5
----- for...of: object(through object[Symbol.iterator]) -----
{ a: 1 }
{ b: 2 }
{ c: 3 }
```

接下来就来看看三个用法究竟是怎么运作的吧

## 一般 for 循环

一般 for 循环就跟 C/C++、Java 等的数组循环方式一模一样，语法如下：

```js
for ( initialize ; end-condition ; step-action ) {
  // statements
}
```

第一块 initialize 指定初始化条件；第二块 end-condition 为终止条件（若为 false 则退出循环）；第三块 step-action 为每次循环后的固定操作（可能是递增递减或"下一步"等自定义操作）。使用范例如下：

```js
// in JavaScript
const nums = [ 1, 2, 3, 4, 5 ]
console.log('----- normal for -----')
for (let i = 0; i < nums.length; i++) {
  console.log(`nums[${i}]: ${nums[i]}`)
}
```

```c
// in C
#define N 5
int nums[N] = { 1, 2, 3, 4, 5 };
for (let i = 0; i < N; i++) {
  printf("nums[%d]: %d\n", i, nums[i]);
}
```

可以发现与 C 语言的 for 循环几乎一样

### Scope 作用域

值得一提的是如果你还在使用 `var` 关键字，或是在一些比较老旧的代码里面，由于 `var` 关键字的作用域和`事件循环模型(Event Loop)`的联合作用可能会产生意想不到的结果，以下为经典面试题：

```js
for (var i = 0; i < 5; i++) {
  setTimeout(function () {
    console.log(i)
  }, 0)
}
```

- 输出

```
5
5
5
5
5
```

由于使用 `var` 声明的 i 属于全局变量，使用 setTimeout 会将方法放入异步队列，直到同步的 `for` 循环操作结束才执行，而执行时 i 已经递增到 5 了，所以打印结果为 5 个 5

以下有两种解决方案：

> 1. 使用 IIFE 立即执行函数表达式

```js
for (var i = 0; i < 5; i++) {
  (function (i) {
    setTimeout(function () {
      console.log(i)
    }, 0)
  }(i))
}
```

> 2. 使用 ES6 的 let 关键字（块级作用域）

```js
for (let i = 0; i < 5; i++) {
  setTimeout(function () {
    console.log(i)
  }, 0)
}
```

所谓的`块级作用域(block scope)`指的是每个`块(block)`有自己的局部变量，也就是 for 循环的五次循环存在各自的 i，互不相干。

有关变量作用域、事件循环机制、块级作用域可以参考我的前几篇文章：

- <a href="https://blog.csdn.net/weixin_44691608/article/details/106766444">JS基礎：Hoisting 變量提升、TDZ 暫時性死區(Temporal Dead Zone)</a>
- <a href="https://blog.csdn.net/weixin_44691608/article/details/106485760">JS基礎：Event Loop事件循環機制</a>

## for...in 循环

第二种循环是 `for...in` 循环，对于数组和对象有不同的表现：

```js
const nums = [ 1, 2, 3, 4, 5 ]
console.log('----- for i in nums -----')
for (let i in nums) {
  console.log(i)
}

const object = { a: 1, b: 2, c: 3 }
console.log('----- for i in object -----')
for (let i in object) {
  console.log(i)
}
```

输出

```
----- for i in nums -----
0
1
2
3
4
----- for i in object -----
a
b
c
```

我们可以看到对于数组遍历的是下标，而对于对象则是属性。其实两者本质上是一样的，记得以前提过的`类数组对象`，由于数组才在内存中的保存模型是以下标为键的特别对象，所以本质上 `for...in` 就是遍历"对象"的属性

### `enumerable: false`

然而遍历属性还是存在某些条件的，我们总不能每次循环连 `toString`、`length` 啥的边缘属性都打印出来，JS 原本也不是这么做的。

这边就要牵涉到对象的`属性描述符(property description)`了，其中的 `enumerable` 属性决定该属性会不会被 `for...in` 循环访问到，我们接着使用上面示例做一些修改：

```js
const object = { a: 1, b: 2, c: 3 }
console.log('----- for i in object -----')
for (let i in object) {
  console.log(i)
}

Object.defineProperty(object, 'a', {
  enumerable: false
})
console.log('----- for i in object -----')
for (let i in object) {
  console.log(i)
}
```

输出

```
----- for i in object -----
a
b
c
----- for i in object -----
b
c
```

我们可以看到当我们将 a 属性设为 `enumerable: false` 时，`for...in` 循环便会跳过 a 属性

### 模仿 for...in

知道了 for...in 的规则，我们可以透过一些方法来以一般 for 循环来模拟 for...in 的作用：

```js
// 用于查看对象所有属性的描述符
function showDetail (o, name) {
  console.log(`----- show detail: ${name} -----`)
  const propertyNames = Object.getOwnPropertyNames(o)
  console.log(`Object.getOwnPropertyNames(${name})`)
  console.log(propertyNames)
  for (let i = 0; i < propertyNames.length; i++) {
    console.log(`property: ${propertyNames[i]}`)
    console.log(Object.getOwnPropertyDescriptor(o, propertyNames[i]))
  }
}

showDetail(nums, 'nums')
showDetail(object, 'object')

console.log('----- for i in nums -----')
for (let i in nums) {
  console.log(i)
}

// 模拟 for...in
console.log('----- fake for i in nums -----')
const numsProps = Object.getOwnPropertyNames(nums)
for (let i = 0; i < numsProps.length; i++) {
  if (Object.getOwnPropertyDescriptor(nums, numsProps[i]).enumerable) {
    console.log(numsProps[i])
  }
}

console.log('----- for i in object -----')
for (let i in object) {
  console.log(i)
}

// 模拟 for...in
console.log('----- fake for i in object -----')
const objectProps = Object.getOwnPropertyNames(object)
for (let i = 0; i < objectProps.length; i++) {
  if (Object.getOwnPropertyDescriptor(object, objectProps[i]).enumerable) {
    console.log(objectProps[i])
  }
}
```

输出

```
----- show detail: nums -----
Object.getOwnPropertyNames(nums)
[ '0', '1', '2', '3', '4', 'length' ]
property: 0
{ value: 1, writable: true, enumerable: true, configurable: true }
property: 1
{ value: 2, writable: true, enumerable: true, configurable: true }
property: 2
{ value: 3, writable: true, enumerable: true, configurable: true }
property: 3
{ value: 4, writable: true, enumerable: true, configurable: true }
property: 4
{ value: 5, writable: true, enumerable: true, configurable: true }
property: length
{ value: 5, writable: true, enumerable: false, configurable: false }
----- show detail: object -----
Object.getOwnPropertyNames(object)
[ 'a', 'b', 'c' ]
property: a
{ value: 1, writable: true, enumerable: false, configurable: true }
property: b
{ value: 2, writable: true, enumerable: true, configurable: true }
property: c
{ value: 3, writable: true, enumerable: true, configurable: true }
----- for i in nums -----
0
1
2
3
4
----- fake for i in nums -----
0
1
2
3
4
----- for i in object -----
b
c
----- fake for i in object -----
b
c
```

透过 `Object.getOwnPropertyNames` 和 `Object.getOwnPropertyDescriptor` 遍历所有属性并过滤描述符 `enumerable: false` 便实现了 `for...in` 的作用

有关对象的`属性描述符(property description)`可以看我之前的文章：

- <a href="https://blog.csdn.net/weixin_44691608/article/details/105872149">JS基礎：Object.DefineProperty</a>


### for...in 小结

关于 `for...in` 用法的小结：

- 对于数组：遍历下标
- 对于对象：遍历 `enumerable: true` 的所有属性

```js
console.log('---- real meaning for array -----')
for (let index in nums) {
  console.log(`nums[${index}]: ${nums[index]}`)
}

console.log('---- real meaning for object -----')
for (let attr in object) {
  console.log(`object.${attr}: ${object[attr]}`)
}
```

输出

```
---- real meaning for array -----
nums[0]: 1
nums[1]: 2
nums[2]: 3
nums[3]: 4
nums[4]: 5
---- real meaning for object -----
object.b: 2
object.c: 3
```

## for...of 循环

第三种 `for...of` 循环是 ES6 的新特性，先看看用法：

```js
const nums = [ 1, 2, 3, 4, 5 ]
console.log('----- for num of nums -----')
for (let num of nums) {
  console.log(num)
}

const string = 'abcde'
console.log('----- for c of string -----')
for (let c of string) {
  console.log(c)
}

const object = { a: 1, b: 2, c: 3 }
console.log('----- for item of object -----')
try {
  for (let item of object) {
    console.log(item)
  }
} catch (err) {
  console.log(err)
}

```

```
----- for num of nums -----
1
2
3
4
5
----- for c of string -----
a
b
c
d
e
----- for item of object -----
TypeError: object is not iterable
```

看起来好像是遍历了所有成员，但是对对象却报错了，到底怎么回事？

### Symbol.iterator

其实 `for...of` 循环并不是真正的遍历对象，而是调用了该对象的 `Symbol.iterator`，不信我演给你看：

```js
const fakeNums = {}
fakeNums[Symbol.iterator] = function* () {
  yield 1
  yield 2
  yield 3
  yield 4
  return
}
console.log('----- for num of fakeNums -----')
for (let num of fakeNums) {
  console.log(num)
}
```

```
----- for num of fakeNums -----
1
2
3
4
```

`Symbol.iterator` 是一个`生成器函数(Generator)`，每次迭代返回一个值(透过 `yield` 关键字)，所以我们如果想要使得对象能够被遍历，只要给他一个 Symbol.iterator 方法就得了：

```js
const object = { a: 1, b: 2, c: 3 }
object[Symbol.iterator] = function* () {
  for (let prop in this) {
    yield { [prop]: this[prop] }
  }
}

console.log('----- for item of object -----')
for (let item of object) {
  console.log(item)
}
```

```
----- for item of object -----
{ a: 1 }
{ b: 2 }
{ c: 3 }
```

有关 `Symbol.iterator` 可以看：

- <a href="https://blog.csdn.net/weixin_44691608/article/details/106087643">ES6特性：Symbol獨一無二的類型</a> 中关于 Symbol.iterator 的描述


### for...of 小结

`for...of` 循环调用的是对象的 `Symbol.iterator` 生成器函数，相当于使用内置的迭代器，也可以自己构造一个迭代器

# 结语

到此结束啦，三种 for 循环的使用方式和相关知识整理，供大家参考。
