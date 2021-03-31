# JS 函数式编程: 高阶函数之柯里化(currying)和反柯里化(uncurrying)

@[TOC](文章目录)

<!-- TOC -->

- [JS 函数式编程: 高阶函数之柯里化(currying)和反柯里化(uncurrying)](#js-函数式编程-高阶函数之柯里化currying和反柯里化uncurrying)
  - [简介](#简介)
  - [参考](#参考)
  - [完整示例代码](#完整示例代码)
- [正文](#正文)
  - [柯里化 Currying](#柯里化-currying)
    - [实现目标](#实现目标)
    - [基础实现](#基础实现)
    - [特殊终止条件](#特殊终止条件)
    - [函数内部柯里化](#函数内部柯里化)
    - [柯里化的应用](#柯里化的应用)
      - [环境兼容性](#环境兼容性)
      - [`Function.prototype.bind`](#functionprototypebind)
  - [反柯里化 Uncurrying](#反柯里化-uncurrying)
    - [`Function.prototype.call` 实现](#functionprototypecall-实现)
    - [`Function.prototype.apply` 实现](#functionprototypeapply-实现)
    - [`Reflect.apply` 实现](#reflectapply-实现)
- [结语](#结语)

<!-- /TOC -->

## 简介

**柯里化(currying)** 和 **反柯里化(uncurrying)** 是函数式编程中一个非常重要的技巧。**部分求值**、**惰性求值**、**提前绑定** 等特性，能够在原有函数的基础之上创造出更多更灵活的用法。本篇就来介绍到底什么是函数柯里化。

## 参考

<table>
  <tr>
    <td>JavaScript高阶函数之currying和uncurrying</td>
    <td><a href="https://www.imooc.com/article/details/id/4381">https://www.imooc.com/article/details/id/4381</a></td>
  </tr>
  <tr>
    <td>JavaScript高阶函数之uncurrying和currying</td>
    <td><a href="https://blog.csdn.net/xuyankuanrong/article/details/80775504">https://blog.csdn.net/xuyankuanrong/article/details/80775504</a></td>
  </tr>
  <tr>
    <td>Favoring Curry</td>
    <td><a href="https://fr.umio.us/favoring-curry/">https://fr.umio.us/favoring-curry/</a></td>
  </tr>
  <tr>
    <td>JavaScript之高阶函数</td>
    <td><a href="https://www.jianshu.com/p/f019f980a50d">https://www.jianshu.com/p/f019f980a50d</a></td>
  </tr>
  <tr>
    <td>function中的callee和caller</td>
    <td><a href="https://blog.csdn.net/qq_35087256/article/details/80023131">https://blog.csdn.net/qq_35087256/article/details/80023131</a></td>
  </tr>
  <tr>
    <td>JavaScript中的函数式编程</td>
    <td><a href="https://www.jianshu.com/p/a5131f3dfb0f">https://www.jianshu.com/p/a5131f3dfb0f</a></td>
  </tr>
  <tr>
    <td>前端柯里化的三种作用</td>
    <td><a href="https://blog.csdn.net/qq_39674542/article/details/82657109">https://blog.csdn.net/qq_39674542/article/details/82657109</a></td>
  </tr>
</table>

## 完整示例代码

<a href="https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_currying">https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_currying</a>

# 正文

## 柯里化 Currying

- 从 **表现形式** 的角度，我们可以这样描述柯里化函数：

    柯里化后的函数，每次可以只接受原函数需要的部分参数，并返回一个能继续接受剩余参数的函数，直到 **参数全部传入** 或 **满足终止条件时** 才返回结果。

- 从 **应用场景** 的角度我们可以说：

    1. 函数定制/提前绑定：根据柯里化函数的特性，我们可以提前传入/判断环境参数进行绑定，返回一个定制好的函数，不仅能够很好的避免表达式的重复，也能更清晰的表示函数逻辑
    2. 延迟执行：从原来向函数传入所有需要的参数，变成依次传入部分参数的形式，能够将各个参数计算的时机分开，同时也能够延迟最终结果的执行。

### 实现目标

首先我们先给出一个最浅白的例子来表达我们想要完成的柯里化的目标：

```js
// simple.js
const f1 = function (a, b, c) {
  return [a, b, c]
}

const f2 = (a) => (b) => (c) => [a, b, c]

f1(1, 2, 3) // [1, 2, 3]
f2(1)(2)(3) // [1, 2, 3]
```

上面的例子说明了柯里化最基本的样貌，我们希望透过某个柯里化函数(currying)，来完成 `f2 = currying(f1)` 的转化。

### 基础实现

第一种给出一个最经典也是通用版本的柯里化实现方案：

```js
// currying.js
function currying(fn) {
  const len = fn.length
  const params = []
  const inner = (...args) => {
    args.forEach((arg) => params.push(arg))
    if (params.length >= len) {
      const res = fn(...params.slice(0, len))
      params.length = 0
      return res
    } else {
      return inner
    }
  }
  return inner
}
```

代码解释：我们先记录原函数需要的参数数量，然后建立一个内部递归函数 `inner`，该函数会不断收集新的参数，等参数足够后才真正调用原方法 `fn(...)`；同时，为了使该柯里化函数返回的函数能重复使用，每次调用原方法之后需要清除已经收集的参数列表 `params.length = 0`，下面给出测试用例(使用 `jest` 测试框架)

```js
// currying.test.js
test('test currying', () => {
  function abc(a, b, c) {
    return [a, b, c]
  }
  const curried = currying(abc)
  expect(curried(1)(2)(3)).toEqual([1, 2, 3])
  expect(curried(1, 2)(3)).toEqual([1, 2, 3])
  expect(curried(1, 2, 3)).toEqual([1, 2, 3])
})
```

我们可以看到 `abc` 函数从原来需要三个参数，变成可以接受多次调用直到累计满足三个参数才返回结果。

### 特殊终止条件

前一种经典的实现的终止条件(调用时机)是根据原函数参数数量来决定，有的时候被柯里化的函数可能会有不同种的终止条件，如下面这个不定参数的加总函数：

```js
function adder(...nums) {
  let res = 0
  nums.forEach((num) => (res += num))
  return res
}
```

要想对这种函数进行柯里化，使用前面给出的那种方案是不行的，所以接下来我们给出一个可以 **接受无限参数** 的柯里化方案，其终止条件为无参数传入的调用：

```js
// currying.js
function curryingInfinite(fn) {
  const params = []
  const inner = (...args) => {
    if (args.length === 0) {
      const res = fn(...params)
      params.length = 0
      return res
    } else {
      args.forEach((arg) => params.push(arg))
      return inner
    }
  }
  return inner
}
```

代码解释：这次内部的递归函数的检查条件变为传入参数的长度，无参数传入时代表返回结果，下面看看测试用例

```js
// currying.test.js
test('test curryingInfinite', () => {
  function adder(...nums) {
    let res = 0
    nums.forEach((num) => (res += num))
    return res
  }
  const curried = curryingInfinite(adder)
  expect(curried(1, 2, 3)(4)(5)(6, 7)()).toBe(28)
  expect(curried(1, 2, 3, 4, 5, 6, 7)()).toBe(28)
})
```

我们可以看到，这次的柯里化实现版本从传入指定数量的参数，变为不断接受参数直到无参数调用(调用没有传入参数)时才返回结果

### 函数内部柯里化

从上面两个例子我们可以发现，一个通用的柯里化函数有时候并不是那么好用，可能需要根据需要柯里化的函数和使用的场景去做调整。所以实际上柯里化更多的只是一个 **思想**，我们可以将柯里化的行为内置到函数里面，或是说从一开始以柯里化的角度来定义函数。例如我们现在来简化第二个例子中的 `curryingInfinite + adder` 的组合：

```js
// currying.js
function curriedAdder() {
  let sum = 0
  const inner = (...nums) => {
    if (nums.length === 0) {
      return sum
    } else {
      nums.forEach((num) => (sum += num))
      return inner
    }
  }
  return inner
}
```

一样的测试用例

```js
// currying.test.js
test('test curriedAdder', () => {
  expect(curriedAdder()(1, 2, 3)(4)(5)(6, 7)()).toBe(28)
  expect(curriedAdder()(1, 2, 3, 4, 5, 6, 7)()).toBe(28)
})
```

### 柯里化的应用

#### 环境兼容性

有些时候我们的代码需要保证浏览器甚至运行环境的兼容性，我们需要对一些全局函数进行检查如下：

```js
var addEvent = function(ele, type, fn) {
    if (window.addEventListener) {
        return ele.addEventListener(type,fn,false);
    } else if (window.attachEvent) {
        return ele.attachEvent(type, fn);
    }
}
```

然而这样写有一个严重的缺陷就是，当我们每次调用这个兼容性的 `addEvent` 方法时，都必须经过一次 `if-else` 的判断。这时我们就可以使用柯里化的思想，**定制** 好一个环境相关的全局函数，往后直接调用已经绑定好的函数即可：

```js
var addEvent = function(ele, type, fn) {
    if (window.addEventListener) {
        addEvent = function(ele, type, fn) {
            ele.addEventListener(type,fn,false);
        } 
    } else if (window.attachEvent) {
        addEvent = function(ele, type, fn) {
            ele.attachEvent(type,fn);
        } 
    }
    //执行
    addEvent(ele, type, fn);
}
```

改写后的函数会在第一次调用的时候直接绑定与环境匹配的方法，往后的调用就能直接使用正确的方法而不再需要额外的条件判断

#### `Function.prototype.bind`

`Function.prototype.bind` 方法本身就是一种柯里化思想的体现。我们知道在 js 中一个函数会根据调用上下文的不同改变 `this` 关键字的指向。这时我们就能够使用 `bind` 方法绑定一个上下文，使得不管在哪里直接调用方法都能有一样的结果：

```js
function f() {
    console.log(this)
}
f() // window / global
const obj = { name: 'superfree' }
const bindingF = f.bind(obj)
bindingF() // { name: 'superfree' }
```

## 反柯里化 Uncurrying

第二个比较少听到的是一个叫 **反柯里化(uncurrying)** 的思想。这里容易产生的一个误解是，反柯里化并不是作为柯里化函数的反函数而存在，仅仅只是名字上存在关联。

反柯里化的作用类似于 **借用方法**。前面我们提到柯里化可以提前绑定函数调用的上下文(也就是 `this` 关键字的指向)，而反柯里化的作用之一就是解藕出一个绑定好的上下文的方法，听起来好像就是 `Function.prototype.call` 方法是不是！

下面我们给出三种反柯里化的实现方式，分别使用了 `Function.prototype.call`、`Function.prototype.apply`、`Reflect.apply`

### `Function.prototype.call` 实现

```js
// uncurrying.js
function uncurryingByCall(fn) {
  return function (ctx, ...args) {
    return fn.call(ctx, ...args)
  }
}
```

- 测试

```js
test('test uncurryingByCall', () => {
  const slice = uncurryingByCall(Array.prototype.slice)
  expect(slice([1, 2, 3, 4, 5], 1, 3)).toEqual([2, 3])
})
```

### `Function.prototype.apply` 实现

```js
// uncurrying.js
function uncurryingByApply(fn) {
  return function (ctx, ...args) {
    return fn.apply(ctx, args)
  }
}
```

- 测试

```js
test('test uncurryingByApply', () => {
  const slice = uncurryingByApply(Array.prototype.slice)
  expect(slice([1, 2, 3, 4, 5], 1, 3)).toEqual([2, 3])
})
```

### `Reflect.apply` 实现

```js
// uncurrying.js
function uncurryingByReflect(fn) {
  return (ctx, ...args) => Reflect.apply(fn, ctx, args)
}
```

- 测试

```js
test('test uncurryingByReflect', () => {
  const slice = uncurryingByReflect(Array.prototype.slice)
  expect(slice([1, 2, 3, 4, 5], 1, 3)).toEqual([2, 3])
})
```

# 结语

柯里化和反柯里化都是围绕着参数/上下文绑定在进行的，在实际的开发场景之中其实是非常有用的一个小技巧。由于函数在 js 语言之中属于一等公民，当我们发现总是在使用相同的参数重复调用同样的方法的时候，我们就可以考虑使用柯里化的思想来定制化(提前绑定参数/上下文)一个新的函数，不仅能够优化调用性能，代码的可读性也是 upup。
