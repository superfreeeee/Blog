# ES6 Promise 应用: 回调函数方法封装成 Promise + async/await 同步化

@[TOC](文章目录)

<!-- TOC -->

- [ES6 Promise 应用: 回调函数方法封装成 Promise + async/await 同步化](#es6-promise-应用-回调函数方法封装成-promise--asyncawait-同步化)
  - [简介](#简介)
  - [参考](#参考)
  - [完整示例代码](#完整示例代码)
- [正文](#正文)
  - [什么是"接受回调函数的方法"？](#什么是接受回调函数的方法)
    - [示例一：http 请求](#示例一http-请求)
    - [示例二：mysql 命令](#示例二mysql-命令)
    - [示例三：同步方法](#示例三同步方法)
    - [面临问题：回调地狱](#面临问题回调地狱)
  - [封装开始](#封装开始)
    - [回调函数方法准备（接受回调函数的方法）](#回调函数方法准备接受回调函数的方法)
    - [使用 Promise 进行封装](#使用-promise-进行封装)
    - [使用 async/await 关键字同步化](#使用-asyncawait-关键字同步化)
- [结语](#结语)

<!-- /TOC -->

## 简介

前一篇<a href="https://blog.csdn.net/weixin_44691608/article/details/106035331">ES6特性：Promise異步函數</a>介绍了 ES6 的新特性 Promise 对象，而<a href="https://blog.csdn.net/weixin_44691608/article/details/110474497">ES6 实战: 手写 Promise</a>则是尝试使用 ES5 以下的语法手撸一个 Promise 出来。

本篇则是要来介绍一种 Promise 对象的应用：将接受回调函数的方法包装成 Promise 对象，同时透过 async/await 关键字同步化。属于实际开发时非常常见的需求，也是非常实用的技能。

## 参考

<table>
  <tr>
    <td></td>
    <td><a href=""></a></td>
  </tr>
</table>

## 完整示例代码

<a href="https://github.com/superfreeeee/Blog-code/tree/main/front_end/es6/es6_promise_encapsulation_callback">https://github.com/superfreeeee/Blog-code/tree/main/front_end/es6/es6_promise_encapsulation_callback</a>

# 正文

## 什么是"接受回调函数的方法"？

我们在开发时使用一些 ES5 以前的库或是不支持 ES6 以上的版本环境时，常常需要用到接受回调函数的方法调用，常见的例子有以下几个：

### 示例一：http 请求

- http 请求（使用 `request` 第三方库）

```js
const request = require('request')
request.get('http://localhost:3000', (err, res, body) => {/* http 请求结果处理 */})
```

`request.get` 方法会发起一个 http 请求，当请求返回时回调用第二个参数的处理函数对返回结果进行处理

### 示例二：mysql 命令

- mysql 连接、查询（使用 `mysql` 库）

```js
const mysql = require('mysql')
const connection = mysql.createConnection({/* 配置信息省略 */})
connection.query('select * from a_table', (err, data) => {/* sql 查询结果处理 */})
```

类似的，透过 `mysql` 库能够执行 SQL 命令，并且在命令执行完毕后调用第二个参数的回调函数进行处理

### 示例三：同步方法

这边我们要解开一个常见的迷思：`接受回调函数的方法并不一定要是异步函数`，前两个例子确实会进行异步调用后才执行回调函数处理异步调用返回的结果，但是像下面这个函数也是一种回调函数方法

- 同步方法接受回调函数

```js
const syncFunction = (x, cb) => {
    console.log(`param x = ${x}`)
    cb()
}
syncFunction(1, () => {/* do something */})
```

### 面临问题：回调地狱

乍看之下，其实回调函数好像还挺合理的，透过接受回调函数的方法将方法`结果处理的逻辑`交给调用者决定(借由回调函数 callback)。然而这样会引发回调地狱的惨剧，试想如果我们调用 sql 查询时想要根据不同的结果再进行二次甚至三次的查询会发生什么事：

```js
connection.query('' /* SQL命令1 */, (err, data1) => {
  /* judge or do something else */
  connection.query('' /* SQL命令2 */, (err, data2) => {
    /* judge or do something else */
    connection.query('' /* SQL命令3 */, (err, data3) => {
      /* judge or do something else */
      console.log(data3)
    })
  })
})
```

这样一来不仅代码结构会变得异常复杂而且丑陋，同时业务的逻辑也会显得混乱不堪，容易忘记哪些指令生处于哪一层的查询之中，第三个问题是查询结果的处理也异常困难。由于 data 都是`属于回调函`数的`局部变量`，所以透过 return 返回结果是没有意义的，而回调函数以外的部分也没办法获取到查询的结果（因为调用逻辑可能是异步的，而查询语句前后的上下文为同步调用的环境）。

为此，我们就能够利用 ES6 提供的 Promise 对象来进行回调函数的封装，将层层包裹的 callback 函数解放，变为使用 `then` 方法的`链式调用`形式

## 封装开始

接下来我们就要来介绍如何将回调函数方法封装成 Promise 对象，并透过 async/await 将异步方法同步化

### 回调函数方法准备（接受回调函数的方法）

这边我们使用 `request` 第三方库来发起 http 请求作为异步方法的代表

```js
const request = require('request')

// callback function
function normalCallback (test, cb) {
  console.log(`invoke function f from test ${test}`)
  request.get('http://localhost:3000', (err, res, body) => {
    cb(err, body)
  })
}
```

`normalCallback` 方法主要功能是向 3000 端口的默认路由请求服务，收到结果后执行回调函数处理结果

一般情况下我们使用上面这种回调函数方法最直白的方式就是按参数传入回调函数（结果处理函数），如下

```js
const test1 = () => {
  // 最原始的回调函数使用方式
  normalCallback('test1', (err, data) => {
    if (err) {
      console.log('err occur')
    } else {
      console.log(`receive data ${data}`)
    }
  })
}

test1()
```

输出

```
invoke function f from test test1
receive data Hello World
```

### 使用 Promise 进行封装

第二步我们要将回调函数方法封装成 Promise 对象

```js
const generatePromise = (test) => {
  return new Promise(function (resolve, reject) {
    normalCallback(test, (err, data) => {
      if (err) {
        reject(err)
      } else {
        resolve(data)
      }
    })
  })
}
```

透过将原方法作为 Promise 任务的主体，并在回调函数中调用 resolve/reject 方法改变 Promise 的状态。

如此一来我们就可以像下面这样透过 then 函数来依序传入结果处理函数

```js
const test2 = () => {
  // 调用封装好的方法返回 Promise 对象
  generatePromise('test2')
    .then(res => {
      console.log(`receive data ${res}`)
    })
    .catch(err => {
      console.log('err occur')
    })
}
test2()
```

输出

```
invoke function f from test test2
receive data Hello World
```

### 使用 async/await 关键字同步化

到此我们还不满足，Proimse 的链式调用是比 callback 的使用形式好看不少，但是与一般的同步方法还是存在一定的差异，接下来我们使用 async/await 关键字进行方法同步化

```js
async function asyncUsage () {
  const res = await generatePromise('test3')
  console.log(`receive data ${res}`)
}
```

透过在外再包裹一层异步方法(async 关键字)，异步方法内部就能够使用 await 方法进行同步化，直接获取 Promise 对象的返回值 res，如此一来这样的调用形式就跟一般的同步方法一样，这就是我们想要的最终形态。

最后看一下调用结果

```js
const test3 = () => {
  asyncUsage()
}
test3()
```

输出

```
invoke function f from test test3
receive data Hello World
```

注意点：如果异步方法报错，`await` 关键字是没办法处理会直接再向外抛出异常，所以通常需要在某个层次上使用 `try...catch` 块来捕捉异常，避免程序退出

# 结语

本篇的核心目标在于将 callback 的调用形式封装成 Promise 的链式调用(then、catch)，最后同步化，是实战开发时不可或缺的能力。
