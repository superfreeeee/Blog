# JS 模块化: CommonJS 与 ESM(ECMAScript Module) 的引用机制比较 & 循环依赖解决方式

@[TOC](文章目录)

<!-- TOC -->

- [JS 模块化: CommonJS 与 ESM(ECMAScript Module) 的引用机制比较 & 循环依赖解决方式](#js-模块化-commonjs-与-esmecmascript-module-的引用机制比较--循环依赖解决方式)
- [前言](#前言)
  - [CommonJS & ECMAScript Module](#commonjs--ecmascript-module)
- [正文](#正文)
  - [1. 引入机制比较](#1-引入机制比较)
    - [1.1 导出 / 引入语法](#11-导出--引入语法)
    - [1.2 原始类型的导出/引入(primitive variable)](#12-原始类型的导出引入primitive-variable)
      - [1.2.1 CommonJS 方案](#121-commonjs-方案)
      - [1.2.2 ESM 方案](#122-esm-方案)
    - [1.3 引用类型的导出/导入(reference variable)](#13-引用类型的导出导入reference-variable)
      - [1.3.1 CommonJS 方案](#131-commonjs-方案)
      - [1.3.2 ESM 方案](#132-esm-方案)
    - [1.4 引用机制小结 & 图解](#14-引用机制小结--图解)
  - [2. 循环依赖测试](#2-循环依赖测试)
    - [2.1 测试代码 & 运行输出](#21-测试代码--运行输出)
      - [2.1.1 CommonJS 循环依赖](#211-commonjs-循环依赖)
      - [2.1.2 ESM 循环依赖](#212-esm-循环依赖)
    - [2.2 循环依赖运行过程详解(图解)](#22-循环依赖运行过程详解图解)
      - [2.2.1 CommonJS 过程](#221-commonjs-过程)
      - [2.2.2 ESM 过程](#222-esm-过程)
    - [2.3 循环依赖小结](#23-循环依赖小结)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

由于 CommonJS 和后来出现的 ESM(ECMAScript) 是大家比较熟悉也是比较常用的模块化方案，所以[前一篇](https://blog.csdn.net/weixin_44691608/article/details/117675558?spm=1001.2014.3001.5501)特地将 ADM 模块化方案提出来解说了一番并演示了应用方案

## CommonJS & ECMAScript Module

而本篇则要回来谈谈 CommonJS 和 ESM 两种模块化方案的差异，以及两种方案对于循环依赖的解决方式

虽然 ESM 方案在 ES6 已经通过提案成为标准，但是不同浏览器、不同环境对于模块化的支持依旧参差不齐，而且 NodeJS 环境下原生只支持 CommonJS 的标准，所以使得当前大环境下 CommonJS 依旧存在不小的生存空间，不会轻易的被 ESM 取代

下面我们就进入正题，说说两种模块化方案的引入机制以及对于循环依赖的解决方案

# 正文

## 1. 引入机制比较

首先第一节我们先来看看两种模块化方案在引入机制上的差异

### 1.1 导出 / 引入语法

首先先看看两种方案的变量导出和引入的语法

- CommonJS

```js
exports.a = a  // 导出变量 a
module.exports = { c: b } // 将变量 b 导出为 c 变量

const a = require('./a') // 引入整个 a 模块，并赋值给变量 a
const { c } = require('./a') // 引入 a 模块，并使用解构赋值只抽取导出模块中的 c 变量
```

- ESM(ECMAScript Module)

```js
export const a = 'xxx' // 导出变量 a
export { a, b } // 导出变量 a, b
export default {} // 模块默认导出

import a from './a' // 从模块 a 导入默认导出变量引用
import { a, b } from './a' // 从模块 a 导入 a, b 变量的引用
```

我们注意到 ESM 模块中我们使用"引用"一词，而 CommonJS 则是使用"赋值"来描述变量的导入和导出，下面我们就来看看具体差异

### 1.2 原始类型的导出/引入(primitive variable)

第一种我们先来看看使用原始类型变量的场景

#### 1.2.1 CommonJS 方案

我们先从 CommonJS 方案看起，首先先定义一个 CommonJS 的模块

- `src/reference/other_with_commonjs.js`

```js
let a = 1
const fa = () => { console.log(`origin a = ${a}`) }
module.exports = { a, fa }
```

首先我们现在该模块中定义一个变量 `a`，在定义一个打印该变量的函数 `fa`，确保打印的是 `other_with_commonjs` 模块的变量 a

接下来我们就在主入口中引入并在修改前后打印变量值

- `src/reference/index.js`

```js
let { a, fa } = require('./other_with_commonjs')

group('commonjs', () => {
  fa()
  log(`current a = ${a}`)

  log('\n>>> invoke a *= 10\n'), (a *= 10)

  fa()
  log(`current a = ${a}`)
})
```

我们先使用 `require` 方法引入变量

```js
let { a, fa } = require('./other_with_commonjs')
```

然后分别打印在原始模块中的变量 a 以及当前模块的变量 a

```js
fa()
log(`current a = ${a}`)
```

接下来我们修改当前模块的变量 a

```js
log('\n>>> invoke a *= 10\n'), (a *= 10)
```

然后再打印一次，结果如下

```
reference
  commonjs
    origin a = 1
    current a = 1
    
    >>> invoke a *= 10
    
    origin a = 1
    current a = 10
```

我们可以看到在变量值修改之后当前模块的变量 a 发生了改变，而原始模块的变量 a 维持着原本的值 `1`

这时候我们已经能够看出来实际上 `require` 引用不过是读取并复制了原本模块中的变量 a 的值，然后赋值给当前模块的变量 a，也就是所谓的 **值传递**，实际上两个模块中分别各存在一个独立的变量 a

#### 1.2.2 ESM 方案

下面我们来看看 ESM 模块，一样先定义一个模块

- `src/reference/other_with_es6.js`

```js
export let b = 2

export const fb = () => {
  console.log(`origin b = ${b}`)
}

export const setb = () => {
  console.log('\n>>> invoke setb b *= 10\n'), (b *= 10)
}
```

不同的是我们这里多定义一个 `setb` 方法用于修改原始模块中的变量值

下面我们仿造上面的测试过程，不同的是使用 ESM 的语法引入变量

- `src/reference/index.js`

```js
import { b, fb, setb } from './other_with_es6'

group('esm', () => {
  fb()
  log(`current b = ${b}`)

  // b *= 10;  // import b is read-only
  setb()

  fb()
  log(`current b = ${b}`)
}
```

由于使用 ESM 的 `import` 关键字进行引入的变量是一个只读类型的变量，所以与前一个例子不同的是，这次我们透过调用 `setb` 改变的是原始模块中的变量值，结果如下

```
reference
  esm
    origin b = 2
    current b = 2
    
    >>> invoke setb b *= 10
    
    origin b = 20
    current b = 20
```

我们发现即便我们调用 `setb` 改变的是原始模块的变量，当前模块的 b 的值打印出来也是最新的值！实际上我们透过 `import` 关键字导入的变量不仅仅只是一个 **只读(read-only)** 的变量，实际上他还是是原本模块中的变量的一种引用，也就是所谓的 **引用传递**。实际上我们在当前模块操作的导入变量只是对于原始模块的一个引用，所以实际上两个模块是真正共有这一个变量的，这个与 CommonJS 是不一样的

### 1.3 引用类型的导出/导入(reference variable)

理解在原始类型上的 **值传递** 和 **变量引用** 之后，对于引用类型的变量(如 object, array 等)就更好理解了，因为实际上变量的值只是对堆中对象的指针，这时候实际上是值传递还是变量引用关系就不是那么大了，反正都是指向同样一个指针

#### 1.3.1 CommonJS 方案

一样我们先定义一个模块

- `src/reference/other_with_commonjs.js`

```js
let oa = { a: 3 }

const foa = () => {
  console.log('origin oa = ', oa)
}

module.exports = {
  oa,
  foa,
}
```

接下来是引入模块的变量并在修改前后打印变量的值

- `src/reference/index.js`

```js
let { oa, foa } = require('./other_with_commonjs')

group('commonjs', () => {
  foa()
  log(`current oa = `, oa)

  log('\n>>> invoke oa.a *= 10\n'), (oa.a *= 10)

  foa()
  log(`current oa = `, oa)
})
```

- 运行输出

```
reference
  commonjs
    origin oa =  { a: 3 }
    current oa =  { a: 3 }
    
    >>> invoke oa.a *= 10
    
    origin oa =  { a: 30 }
    current oa =  { a: 30 }
```

这边需要注意的点就只有对于两个模块的 oa 变量是分离的，只是指向同一个对象而已；所以当原始模块的 `oa` 改变的话，当前模块还是会指向原来的对象，这点需要注意

#### 1.3.2 ESM 方案

ESM 方案也测试一下

- `src/reference/other_with_es6.js`

```js
export const ob = { b: 4 }

export const fob = () => {
  console.log('origin ob = ', ob)
}
```

- `src/reference/index.js`

```js
import { ob, fob } from './other_with_es6'

group('esm', () => {
  fob()
  log(`current ob = `, ob)

  log('\n>>> invoke ob.b *= 10\n'), (ob.b *= 10)

  fob()
  log(`current ob = `, ob)
})
```

- 运行输出

```
reference
  esm
    origin ob =  { b: 4 }
    current ob =  { b: 4 }
    
    >>> invoke ob.b *= 10
    
    origin ob =  { b: 40 }
    current ob =  { b: 40 }
```

这边需要注意的是，虽然 `import` 进来的变量 `ob` 是无法改变指向的，但是还是可以直接修改对象的属性，这点需要注意

```js
ob.b *= 10
```

### 1.4 引用机制小结 & 图解

最后做一个小结论

- CommonJS
  - **值传递** (赋值变量值到外部模块)
  - 实际上对于外部模块和原始模块一共存在两个变量，只是在引入的当下存在相同的值或指向相同的对象
- ESM
  - **变量引用** (引入对于原始模块中的变量的引用)
  - 实际上不论在外部模块引入几次，永远都只存在唯一的变量值，所有其他模块引入的都只是对目标变量的引用(所以即便是原始类型的改变，外部模块也能同步访问到最新的数据)

也就是两种方案在引入机制上的规则如下图示：

- CommonJS

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_module_cjs_esm_compare_reference_cjs.png)

- ESM

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_module_cjs_esm_compare_reference_esm.png)

## 2. 循环依赖测试

第二节我们来看看在前面的引入机制之下，两种模块化方案对于循环依赖的使用是如何进行处理的

### 2.1 测试代码 & 运行输出

由于代码比较简单，所以这边一次给出所有代码，后面慢慢用图解的形式说明

两种方案的测试都是分别定义 `a、b` 模块，再互相引用对方导出的变量，然后进行输出；而模块引用的顺序都是由入口模块 `index` 引入 `a` 模块开始

#### 2.1.1 CommonJS 循环依赖

- `src/cycle_dependency/cjs/a.js`

```js
const { b } = require('./b')

log('load a.js')

log(`b = ${b}`)

let a = 1

exports.a = a
```

- `src/cycle_dependency/cjs/b.js`

```js
const { a } = require('./a')

log('load b.js')

log(`a = ${a}`)

let b = 2

exports.b = b
```

- `src/cycle_dependency/cjs/index.js`

```js
import { group } from '../../utils'

group('cjs', () => {
  require('./a')
})
```

- 运行输出

```
cycle_dependency
  cjs
    load b.js
    a = undefined
    load a.js
    b = 2
```

#### 2.1.2 ESM 循环依赖

- `src/cycle_dependency/esm/a.js`

```js
import { b } from './b'

log('load a.js')

log(`b = ${b}`)

let a = 1

export { a }
```

- `src/cycle_dependency/esm/b.js`

```js
import { a } from './a'

log('load b.js')

log(`a = ${a}`)

let b = 2

export { b }
```

- `src/cycle_dependency/esm/index.js`

```js
import { group } from '../../utils'

group('esm', () => {
  require('./a')
})
```

- 运行输出

```
cycle_dependency
  esm
    load b.js
    a = undefined
    load a.js
    b = 2
```

### 2.2 循环依赖运行过程详解(图解)

下面我们用两张图来辅助我们解释循环依赖的具体运行流程和最终结果

#### 2.2.1 CommonJS 过程

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_module_cjs_esm_compare_cycle_dependency_cjs.png)

1. 引入 `a` 模块后的第一步就是引入 `b` 模块，所以会直接加载并运行 `b` 模块，这时候 `a` 模块会停在第一句，直到 `b` 模块加载完毕后拿到 `b` 的值
2. `b` 模块的第一句又是反过来加载 `a` 模块，但是这时候 `a` 模块已经被加载过第一次了，所以 `b` 模块会直接从 `a` 模块中提取 a 变量，然而未完成的 `a` 模块还没导出名为 a 的变量，所以 `b` 模块拿到的是 `a = undefined`
3. 第三步打印的时候就会打印出

```
load b.js
a = undefined
```

4. 第四和第五步则是定义 `b` 模块内部的变量 b 并导出(`exports.b = b`)
5. 到这里 `b` 模块加载完毕了，所以 `a` 模块就能够继续运行了，拿到 `b` 模块返回的 `b = 2` 的值，同时由于前面说过的 CommonJS 采取的是 **值传递** 的方式，所以实际上是在 `a` 模块中产生一个新的变量 b，并将 `b` 模块中 b 变量的值拷贝到 `a` 模块的 b 变量上
6. 第六步打印的结果就是

```
load a.js
b = 2
```

7. 最后第七和第八步的时候才定义了 `a` 模块中的 a 变量并导出，但是实际上 `b` 模块拿到的 a 变量并不会更新，所以最后一共存在四个变量，如图最后一步

#### 2.2.2 ESM 过程

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_module_cjs_esm_compare_cycle_dependency_esm.png)

然而对于 ESM 方案来说，整体流程类似，就不再一步一步说明了。

不同的地方在于，由于 ESM 的 `import` 关键字引入的是对目标变量的引用，所以实际上在脚本的最后只存在两个变量：`a` 模块的  a 变量、`b` 模块的 b 变量，而对于另一个模块变量的导入是对于原始变量的引用，也就是说在脚本的末尾，两个模块的 `a、b` 变量事实上是同步的，这点就与 CommonJS 不同了。

### 2.3 循环依赖小结

两种模块化方案对于循环依赖的处理其实很相似，都是会 **优先加载引入的模块**，如果模块已经存在就会提取当前已经定义的变量值(CommonJS 使用值传递，而 ESM 则是引用传递)，由于两个方案的引入机制不同，所以会造成引入变量在原始模块的变量改变之后，产生不一样的结果(CommonJS 维持引入当下的值，ESM 的引用会正确的与原始变量同步更新)

实际上要使两种方案都有一样的表现并且同步变量的解决方案也很简单，我们就只需要定义一个不会改变的对象

```js
const obj = {}
```

不管是 CommonJS 的值传递或是 ESM 的引用传递，反正最后使用的变量总是会指向同一个对象，也就不存在变量值不同步的问题。

# 结语

本篇对两种模块化方案的使用进行了比较详细的说明和比较，同时也帮助读者对于 JS 的模块化机制包括循环依赖有更深的认识，供大家参考。

# 其他资源

## 参考连接

| Title                                            | Link                                                                                                         |
| ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------ |
| 1. 模块化的引入与导出 （commonJS规范 和ES6规范） | [https://www.cnblogs.com/-constructor/p/11810237.html](https://www.cnblogs.com/-constructor/p/11810237.html) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_module_cjs_esm_compare](https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_module_cjs_esm_compare)
