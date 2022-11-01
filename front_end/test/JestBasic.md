# Jest 入门: Jest 核心 API & 多环境运行配置实现前端测试

@[TOC](文章目录)

<!-- TOC -->

- [Jest 入门: Jest 核心 API & 多环境运行配置实现前端测试](#jest-入门-jest-核心-api--多环境运行配置实现前端测试)
- [前言](#前言)
- [正文](#正文)
  - [0. 要测试什么？](#0-要测试什么)
  - [1. 启用 Jest 测试框架 & 多环境配置](#1-启用-jest-测试框架--多环境配置)
    - [1.1 安装依赖 & 初始化项目](#11-安装依赖--初始化项目)
    - [1.2 基础 NodeJS 环境测试](#12-基础-nodejs-环境测试)
    - [1.3 搭配 Babel 支持新语法](#13-搭配-babel-支持新语法)
    - [1.4 加上 TypeScript 实现类型检查](#14-加上-typescript-实现类型检查)
  - [2. Jest 核心 API](#2-jest-核心-api)
    - [2.1 匹配器篇](#21-匹配器篇)
      - [2.1.1 精确匹配](#211-精确匹配)
      - [2.1.2 真值匹配](#212-真值匹配)
      - [2.1.3 数字匹配](#213-数字匹配)
      - [2.1.4 字符串匹配](#214-字符串匹配)
      - [2.1.5 数组匹配](#215-数组匹配)
      - [2.1.6 异常匹配(异常捕获)](#216-异常匹配异常捕获)
      - [2.1.7 匹配器汇总](#217-匹配器汇总)
    - [2.2 异步函数测试](#22-异步函数测试)
      - [2.2.1 Callback 回调函数](#221-callback-回调函数)
      - [2.2.2 使用 Promise](#222-使用-promise)
      - [2.2.3 使用 async / await 关键字](#223-使用-async--await-关键字)
    - [2.3 其他 API](#23-其他-api)
  - [3. 关于前端测试的思考](#3-关于前端测试的思考)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

本篇要来介绍一个前端非常有名的 **轻量级测试框架：Jest**。本篇作为入门篇仅仅介绍 Jest 的基础 API，以及对于几个常见的基本场景/环境进行相应的配置(透过 Babel 支持 ES6+语法、支持 TypeScript)。后续根据需要会再另外写一篇用于测试 React 框架的 Jest 详细用法。

# 正文

## 0. 要测试什么？

首先在开始介绍之前，要先明确一下所谓的"前端"到底要测试什么？

- 功能性测试

对于后端来说，因为大部分后端业务属于功能性函数，或是对一些业务进行逻辑层的封装，所谓就衍生出了 **单元测试、集成测试** (这里就不区分到底是白盒还是黑盒了)，也就是说在服务端的项目中几乎都是功能函数或是流程逻辑的集合，我们就很清晰的知道要测试每个功能函数/模块的完整性和正确性。

- 视图 vs 业务逻辑

但是相较之下前端很多时候的作用在于 **从数据到视图的映射**，甚至不提 MVVM 框架之下，很多时候对于数据的处理都是与 dom 元素的操作紧密相关的。这就导致了前端对于测试边界的模糊，单纯的功能函数仅仅只占了整个前端逻辑中的一小部分，而视图也就是 dom 元素本身却又存在过于复杂的表现行为导致难以进行高覆盖性的单元测试，倒不如"直接操作页面成果"来得直接。

不过归功于当前流行的 MVVM 响应式框架，透过 ViewModel 的双向绑定，用户开始能够将前端的数据从视图中分离并独立成单独的 **组件**，同时基于 React 框架本身对于虚拟 DOM 的封装和测试接口的支持，除了用户自己编写的功能性测试之外，也能够轻松支持 **基于单个组件的单元测试**。

本篇将要介绍的是如何在不同环境下来编写 Jest 测试，以及一些 Jest 提功能的基本 API 用法，说白了其实还是关于传统功能性的测试的能力使用，关于 **视图和组件的快照测试** 我们摆到下一篇来说明。

## 1. 启用 Jest 测试框架 & 多环境配置

说了半天终于要进入正题了，下面我们先来学学怎么在前端项目中添加 Jest 框架并编写测试

### 1.1 安装依赖 & 初始化项目

首先是安装依赖，Jest 本身只有一个依赖就是 `jest`，这里我们使用 `yarn` 作为我们的包管理器

- 初始化 node 项目

```bash
$ mkdir jest_basic
$ cd jest_basic
$ yarn init -y
```

- 添加依赖

```bash
$ yarn add jest -D
```

添加好依赖之后我们可以透过 `yarn jest --init` 的指令来初始化关于 Jest 测试框架的基础配置

```bash
$ yarn jest --init
```

基本上就是选 node 的环境配置就行

### 1.2 基础 NodeJS 环境测试

接下来我们就可以来编写我们的第一个测试文件啦，在测试前首先当然要有一个被测试的东西

- `/src/index.js`

```js
function sum(a, b) {
  return a + b
}

module.exports = sum
```

下面我们另外再创建一个测试文件，从约束上来说所有 Jest 测试文件应该要以 `.test.js` 作为后缀，而按照惯例通常直接使用目标测试文件的名字，所以我们就可以为 `index.js` 创建一个测试文件 `index.text.js`

- `/src/index.test.js`

```js
const sum = require('./index.js')

test('test 1', () => {
  expect(sum(1, 2)).toBe(3)
})

test('test 2', () => {
  expect(sum(-1, 1)).toBe(0)
})

test('test 3', () => {
  expect(sum(Infinity, Infinity)).toBe(Infinity)
})
```

关于 Jest 的核心 API 我们下面会再说明，这里只要知道：`test` 创建出一个测试用例、而 `expect + toXxx` 方法组合出测试用例中的断言语句。也就是在这段代码中我们为 `index` 模块编写了三个测试用例，每个用例调用了一次 `sum` 方法并预期结果

如此一来就写好我们的测试文件了，下面在命令行输入指令 `yarn test` 执行 `jest` 指令(由前面 `yarn jest --init` 自动生成)就能够看到测试结果了

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jest_basic_index.png)

### 1.3 搭配 Babel 支持新语法

由于 NodeJS 环境下能够支持的语法其实有限，在项目中我们常常会用到 Babel 来支持新的 JS 语法特性。事实上 Jest 也提供了与 Babel 连用的方法！

首先一样要安装一下依赖

```bash
$ yarn add @babel/core @babel/preset-env -D  # babel 核心
$ yarn add @babel/plugin-transform-runtime -D  # 用于 babel 运行时转换
```

然后我们还需要再加一个 Babel 的配置文件

- `babel.config.json`

```json
{
  "presets": ["@babel/preset-env"],
  "plugins": ["@babel/plugin-transform-runtime"]
}
```

这样一来 Jest 就会自动在运行时自动侦测到 babel 的配置，并对代码进行编译以支持新的语法

下面我们再写一个测试文件试试

- `/src/babel.test.js`

```js
import sum from './index'

test('test babel', () => {
  expect(sum(2, 2)).toBe(4)
})
```

一样用上刚刚的 `index` 模块，下面是测试结果，看到没有报错，也就是成功支持了 ES6 的模块化语法(`import/export`)了

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jest_basic_babel.png)

### 1.4 加上 TypeScript 实现类型检查

第三种我们要在更进一步，在 Jest 的测试代码中同时用上 TypeScript，使得代码不仅仅完成功能性的测试，同时加上了类型的约束来提高函数的可用性

一样先安装依赖

```bash
$ yarn add @babel/preset-typescript @types/jest -D
```

我们在使用 Babel 的基础之上，直接添加对于 typescript 的转换支持(当然我们也可以另外构建出利用原本 `typescript` 库的 `tsc` 指令进行编译之后再投入 Jest 测试的工作流程，看项目需求)，同时加上 Jest 相关的类型库(`@types/jest`)

接下来略微修改一下 Babel 的配置文件，以支持对于 typescript 语法的转换

- `babel.config.json`

```json
{
  "presets": ["@babel/preset-env", "@babel/preset-typescript"],
  "plugins": ["@babel/plugin-transform-runtime"]
}
```

下面的测试后我们就可以使用 `.test.ts` 扩展名的测试文件了

- `/src/typescript.test.js`

```ts
import sum from './index'

type BinaryFn = (a: number, b: number) => number

test('test babel', () => {
  const fn: BinaryFn = sum
  expect(fn(2, 2)).toBe(4)
})
```

我们为 `sum` 函数添加一个 `BinaryFn` 的类型限制，由此来约束测试代码对于函数的调用，可以省去关于类型检查的逻辑代码，同时也使得函数的功能和参数更加明确

- 测试结果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jest_basic_ts.png)

## 2. Jest 核心 API

好了前面关于不同环境的 Jest 配置大概就告一段落，下面我们开始着重讲解 Jest 本身的核心 API 使用方式

### 2.1 匹配器篇

前面我们已经看到 `test、expect、toBe` 方法，那这些函数究竟是什么呢？

- `test(name, cb)`：定义一个测试用例，第一个参数 `name` 为用例名称，第二个 `cb` 为用例过程代码
- `expect(receiveValue)`：定义一个被测试的目标值，通常是由要测试的目标函数生成的某个值，后续调用匹配器来进行断言来完成测试
- `toBe(expectedValue)`：调用 `expect` 并接受真实结果之后，后面就可以调用 `toBe` 来指定该输出的期望值

接下来我们会介绍在 Jest 当中提供的各种匹配器的使用

#### 2.1.1 精确匹配

首先是第一种：**精确匹配**，简单来说就是要值完全一样才能生效，官方文档是这样说的：

> `toBe` uses `Object.is` to test exact equality.

测试文件如下

- `/src/simple.test.js`

```js
test('toBe', () => {
  expect(1 + 1).toBe(2)
  // expect('1').toBe(1)              // fail
  // expect({ a: 1 }).toBe({ a: 1 })  // fail
})
```

但是有个问题是，当我们想要比较对象的时候，则要使用 `toEqual` 方法，他会进入对象比较每个属性

```js
test('toEqual', () => {
  expect({ a: 1 }).toEqual({ a: 1 })
  expect([1, 2, 3]).toEqual([1, 2, 3])
})
```

当我们只是想要排除一些情况的时候，则可以调用 `not` 反向器

```js
test('not', () => {
  expect(1).not.toBe(0)
})
```

最终的测试结果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jest_basic_simple.png)

#### 2.1.2 真值匹配

第二种则是对于真/假值的匹配，我们知道 JS 中部分操作会引起隐式的类型转换，同时 `null、undefined` 类型又是独立于其他值类型的空类，这些都属于测试阶段需要能够确定的，所以 Jest 也提供了几个相应的方法

- `/src/bool.test.js`

```js
test('null', () => {
  const n = null
  expect(n).toBeNull() // toBeNull：匹配 null
  expect(n).toBeDefined() // toBeDefined：匹配非 undefined
  expect(n).not.toBeUndefined() // toBeUndefined：匹配 undefined
  expect(n).not.toBeTruthy() // toBeTruthy：匹配真值
  expect(n).toBeFalsy() // toBeFalsy：匹配假值
})

test('zero', () => {
  const zero = 0
  expect(zero).not.toBeNull()
  expect(zero).toBeDefined()
  expect(zero).not.toBeUndefined()
  expect(zero).not.toBeTruthy()
  expect(zero).toBeFalsy()
})
```

这边用到了几个方法稍微说明一下

| Method          | Description                               |
| --------------- | ----------------------------------------- |
| `toBeNull`      | 只匹配 `null`                             |
| `toBeUndefined` | 只匹配 `undefined`                        |
| `toBeDefined`   | `toBeUndefined` 的相反                    |
| `toBeFalsy`     | 匹配假值(`0、false、''、null、undefined`) |
| `toBeTruthy`    | 匹配真值(非假值)                          |

结果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jest_basic_bool.png)

#### 2.1.3 数字匹配

数字的部分通常并不会是一个指定的值作为结果，而是某个约束范围如某个数值区间等，Jest 当然也能做

- `/src/number.test.js`

```js
test('2 + 2', () => {
  const val = 2 + 2
  expect(val).toBe(4)
  expect(val).toEqual(4)

  expect(val).toBeGreaterThan(3)
  expect(val).toBeGreaterThanOrEqual(3.5)
  expect(val).toBeLessThan(5)
  expect(val).toBeLessThanOrEqual(4.5)
})
```

方法不想说了，方法名都说的很清楚hhh

另外值得一提的是，对于浮点数有时候会出现精度问题，使得 `toBe` 方法失效，这时候有一个专门由于此问题的 `toBeCloseTo` 方法

```js
test('0.1 + 0.2', () => {
  const val = 0.1 + 0.2
  expect(val).not.toBe(0.3)
  expect(val).toBeCloseTo(0.3)
})
```

结果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jest_basic_number.png)

#### 2.1.4 字符串匹配

下面一个是关于字符串类型的匹配。对于字符串来说实际上就是使用 `toBe` 进行精确匹配，或是使用 `toMatch` 进行正则表达式的匹配

- `/src/string.test.js`

```js
test('I not in time', () => {
  expect('time').not.toMatch(/I/)
  expect('time').toMatch(/.*/)
  expect('time').toMatch(/^t/)
})

test('stop is in Christoph', () => {
  expect('Christoph').toMatch(/stop/)
})
```

这里使用的主要还是部分匹配，而不用与正则表达式完全匹配，也就相当于 `String.prototype.match` 方法

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jest_basic_string.png)

#### 2.1.5 数组匹配

再来前面提过，对于对象类型我们可以使用 `toEqual` 进行精确匹配，而数组类型也是一种特殊的对象类型，除了使用 `toEqual` 进行精确匹配之外，还能提供了 `toContain` 方法检查是否包含目标元素

- `/src/array.test.js`

```js
test('toContain', () => {
  const arr = ['123', '456', '789']
  expect(arr).toContain('123')
  expect(arr).not.toContain('000')
  expect(new Set(arr)).not.toContain('000')

  const objList = [{ a: 1 }]
  expect(objList).not.toContain({ a: 1 })
})
```

结果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jest_basic_array.png)

#### 2.1.6 异常匹配(异常捕获)

前面已经大致涵盖 JS 的各种类型，下面一个来说说如果调用方法产生异常的时候，Jest 也能接到！

- `/src/exception.test.js`

```js
function f() {
  throw new Error('My Error')
}

test('toThrow', () => {
  expect(f).toThrow()
  expect(f).toThrow(Error)
  expect(f).not.toThrow('dont contain this message')
  expect(f).toThrow(/Error/)
})
```

我们将需要执行的函数直接放入 `expect`，然后使用 `toThrow` 来预期函数调用会抛出异常

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jest_basic_exception.png)

#### 2.1.7 匹配器汇总

下面我们总结一下前面使用到的所有匹配器函数和其作用

| Method                         | Description                 |
| ------------------------------ | --------------------------- |
| `.toBe(val)`                   | 严格等于                    |
| `.toEqual(val)`                | 比较对象内容                |
| `.not`                         | 取反                        |
| `.toBeNull()`                  | 匹配 `null`                 |
| `.toBeUndefined()`             | 匹配 `undefined`            |
| `.toBeDefined()`               | 等价于 `.not.toBeUndefined` |
| `.toBeTruthy()`                | 匹配真值                    |
| `.toBeFalsy()`                 | 匹配假值                    |
| `.toBeGreaterThan(val)`        | $\gt$                       |
| `.toBeGreaterThanOrEqual(val)` | $\ge$                       |
| `.toBeLessThan(val)`           | $\lt$                       |
| `.toBeLessThanOrEqual(val)`    | $\le$                       |
| `.toBeCloseTo(val)`            | 近似于(浮点数相等)          |
| `.toMatch(regexp)`             | 匹配正则表达式              |
| `.toContain(val)`              | 检查数组是否包含目标        |
| `.toThrow(err)`                | 检查函数是否抛出异常        |

### 2.2 异步函数测试

#### 2.2.1 Callback 回调函数

第二部分比较不同的是，有时候我们要测试的方法是一个异步函数，也就是执行过程会被放到事件循环队列里面，这就使得函数真正的结果在测试用例结束之后才被调用，如下代码

```js
function asyncFunction(cb, ms = 1000) {
  setTimeout(() => {
    cb('A Message')
  }, ms)
}

test('callback', () => {
  asyncFunction((msg) => {
    expect(msg).toBe('A Message')
    expect(msg).not.toBe('Other Message')
  })
})
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jest_basic_asynchronous1.png)

真正的测试用例再回调之前就结束了，这样不太对。实际上我们可以从 `test` 方法的第二个参数中的测试用例方法上接受一个参数 `done` 并显示的调用指定用例结束

- `/src/asynchronous.test.js`

```js
function asyncFunction(cb, ms = 1000) {
  setTimeout(() => {
    cb('A Message')
  }, ms)
}

test('callback', (done) => {
  asyncFunction((msg) => {
    expect(msg).toBe('A Message')
    expect(msg).not.toBe('Other Message')
    done()
  })
})
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jest_basic_asynchronous2.png)

#### 2.2.2 使用 Promise

使用回调的方法主要就是使用 `done` 方法来结束测试用例，然而我们更熟悉的是 ES6 的 Promise 作为回调函数的返回，Jest 也想到了，而且对 Promise 的支持更加友善

具体使用的方式就是我们可以将返回的 Promise 对象作为测试用例的返回值，Jest 会自动为我们检查并等待 Promise 的异步结果

首先我们先要将刚刚的回调函数封装成 Promise 对象

```js
function createPromise(fn) {
  return () =>
    new Promise((resolve, reject) => {
      try {
        fn((msg) => {
          resolve(msg)
        })
      } catch (e) {
        reject(e)
      }
    })
}

const asyncFunctionWithPromise = createPromise(asyncFunction)
```

接下来就可以直接在测试用例里面使用 Promise 对象了

```js
test('promise', () => {
  return asyncFunctionWithPromise().then((msg) => {
    expect(msg).toBe('A Message')
    expect(msg).not.toBe('Other Message')
  })
})
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jest_basic_asynchronous3.png)

除了使用 `then` 方法获取结果并在其中使用 `expect` 来创建测试目标之外，我们还可以使用 `resolves` 和 `rejects` 来直接检查结果

```js
const asyncPromise = (success = true) =>
  new Promise((resolve, reject) => {
    setTimeout(() => {
      if (success) {
        resolve('success')
      } else {
        reject('fail')
      }
    })
  })

test('promise resolves', () => {
  return expect(asyncPromise()).resolves.toBe('success')
})

test('promise rejects', () => {
  return expect(asyncPromise(false)).rejects.toBe('fail')
})
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jest_basic_asynchronous4.png)

#### 2.2.3 使用 async / await 关键字

能用 Promise 之后，我们还不满意，再进一步使用 ES8 的 `async/await` 关键字来进行同步化

- `/src/async_await.test.js`

```js
const asyncPromise = (success = true) =>
  new Promise((resolve, reject) => {
    setTimeout(() => {
      if (success) {
        resolve('success')
      } else {
        reject('fail')
      }
    })
  })

test('async / await resolve', async () => {
  const res = await asyncPromise(true)
  expect(res).toBe('success')
})

test('async / await reject', async () => {
  expect.assertions(1)
  try {
    const res = await asyncPromise(false)
    expect(res).toBe('success')
  } catch (e) {
    expect(e).toBe('fail')
  }
})

test('async / await with resolves/rejects', async () => {
  expect.assertions(2)
  await expect(asyncPromise(true)).resolves.toBe('success')
  await expect(asyncPromise(false)).rejects.toBe('fail')
})
```

使用 `async/await` 的时候，我们可以使用 `async` 将整个测试用例包成一个 Promise，在其中就能实现异步方法的同步化

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jest_basic_async_await.png)

### 2.3 其他 API

其他还有像是 `.beforeAll`、`.beforeEach`、`.afterAll`、`.afterEach` 等前后置回调，`expect.xxx` 等用例内部约束等，本篇不再详细说明，或许其他篇会用到，读者可以去查查看 Jest 官方的 API 文档更加全面

## 3. 关于前端测试的思考

最后一点想要说明的是，关于前端测试用例的思考。由于前端的数据和业务逻辑与视图是紧密相关的，同时随着前端能完成的操作越来越多，业务逻辑越来越复杂，数据和视图之间的界限就更需要明确区分。

前端测试的出现并不仅仅是提升代码的健壮性和可用性，同时也是帮助开发者重新思考 **数据和视图的绑定、模块化的界限，以及功能模块的划分**。当一个项目有明确且良好的划分方式和职责分离，不仅降低测试的复杂度和难度，同时也使得整个前端页面的逻辑更加明朗。

# 结语

本篇介绍了一个适合用于 JS 语言的测试框架，事实上它不仅仅作为前端测试框架，使用 NodeJS 开发的服务端框架也能够使用 Jest 来进行测试。就好比 JUnit 之于 Java 一般，Jest 也是作为 JS 语言的最泛用的基础测试框架之一。

具体针对个别大型项目的测试还能够利用 Jest 提供的自定义测试方法为基础，进行个别化的测试扩展，进而封装成项目通用的测试方法，简化测试的开发。

# 其他资源

## 参考连接

| Title     | Link                                                     |
| --------- | -------------------------------------------------------- |
| Jest 官方 | [https://jestjs.io/zh-Hans/](https://jestjs.io/zh-Hans/) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/jest/jest_basic](https://github.com/superfreeeee/Blog-code/tree/main/front_end/jest/jest_basic)
