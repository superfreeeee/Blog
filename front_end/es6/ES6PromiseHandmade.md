# ES6 实战: 手写 Promise

@[TOC](文章目录)

<!-- TOC -->

- [ES6 实战: 手写 Promise](#es6-实战-手写-promise)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [类结构和对外接口](#类结构和对外接口)
    - [重要属性](#重要属性)
    - [回调函数](#回调函数)
    - [静态方法](#静态方法)
    - [最终类结构](#最终类结构)
  - [简单同步版本](#简单同步版本)
  - [Promise 返回值打包](#promise-返回值打包)
  - [接受异步任务](#接受异步任务)
  - [添加静态方法](#添加静态方法)
- [结语](#结语)

<!-- /TOC -->

## 简介

前一篇：<a href="https://blog.csdn.net/weixin_44691608/article/details/106035331">ES6特性：Promise異步函數</a>，介绍过了一些 Promise 的基础概念和用法，本篇将要来尝试手写自己的 Promise 类。

Promise 属于 ES6 新增的原生函数，因此在 ES5 甚至更早的版本环境里面我们需要透过 babel 或是其他 polyfill 来额外注入 Promise 类，本篇会带读者一步步完成一个 Promise 的雏形，更多的类型检查和写法支持可以由读者参考规范慢慢加上去。

## 参考

<table>
  <tr>
    <td>可能是目前最易理解的手写promise</td>
    <td><a href="https://mp.weixin.qq.com/s/oURuka-Qgbbj8JKtlYNMaw">https://mp.weixin.qq.com/s/oURuka-Qgbbj8JKtlYNMaw</a></td>
  </tr>
  <tr>
    <td>Promise-MDN</td>
    <td><a href="https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise">https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise</a></td>
  </tr>
  <tr>
    <td>JavaScript Promise 对象-菜鸟教程</td>
    <td><a href="https://www.runoob.com/w3cnote/javascript-promise-object.html">https://www.runoob.com/w3cnote/javascript-promise-object.html</a></td>
  </tr>
  <tr>
    <td>剖析Promise内部结构</td>
    <td><a href="https://github.com/xieranmaya/blog/issues/3">https://github.com/xieranmaya/blog/issues/3</a></td>
  </tr>
  <tr>
    <td>Promises/A+ Compliance Test Suite</td>
    <td><a href="https://github.com/promises-aplus/promises-tests">https://github.com/promises-aplus/promises-tests</a></td>
  </tr>
</table>

# 正文

## 类结构和对外接口

### 重要属性

首先我们先来看看 Promise 的概念，Promise 相当于承诺使用者必定能够在可见的未来接收到返回，Promise 对象的三个状态也应运而生：

- `pending` 等待状态：Promise 对象初始化后到任务执行结束前的状态
- `resolved(fulfilled)` 接受状态：Promise 对象接受任务结果的状态
- `rejected` 拒绝状态：Promise 对象拒绝任务结果的状态

同时我们的对象需要添加三个相关属性：

- `state` 状态：用于保存当前 Promise 对象的状态
- `value` 接受值：保存接受状态下的任务结果
- `reason` 拒绝理由：保存拒绝时的理由（异常信息）

### 回调函数

用过 Promise 都知道我们初始化 Promise 时传入主任务后，要使用 `then` 方法和 `catch` 方法来接住`接受(resolve)`或`拒绝(reject)`后的结果并传入相应的回调函数，形式如下：

```js
const p = new Promise(function (resolve, reject) {
    // main task ...
})
p.then(value => {
    // handle result when resolved
}).catch(reason => {
    // handle reason when rejected
})
```

比较正式的函数接口（链式调用）定义如下：

- `Promise.prototype.then(onFulfilled, onRejected)`：`onFulfilled` 为接受状态的回调函数、`onRejected` 传入拒绝状态的回调函数
- `Promise.prototype.catch(onRejected)`：拦截拒绝状态下的 Promise 对象
- `*Promise.prototype.finally(onFinally)`：不管接受或拒绝都执行的回调函数，本篇不做实现

这边我们发现要接住 Promise 对象返回的结果通常必须传入回调函数（由于 Promise 内部方法可能是同步也可以是异步），所以我们的类也必须保存两个回调函数队列，一个是`接受状态的回调函数队列(resolveCallbacks)`以及`拒绝状态的回调函数队列(rejectCallbacks)`

### 静态方法

Promise 类本身还提供几个静态方法：

- `Promise.all(iterable)`：一次执行多个 Promise 任务，直到全部都进入 `fulfilled` 接受状态，或第一个拒绝任务
- `*Promise.any(iterable)`：也是一次执行多个 Promise 任务，只要接受其中一个就返回，或是直到全部拒绝
- `Promise.retry(executor, times, delay)`：允许执行有限次数的尝试，每次尝试间隔 `delay` 毫秒

### 最终类结构

对应上述说明和接口，下面先列出接下来我们要实现的 Promise 类架构：

```js
function Promise (executor) {
  const self = this // 保留 this
  this.state = 'pending' // 初始状态为 pending
  this.resolveCallbacks = [] // resolve 回调函数队列
  this.rejectCallbacks = [] // reject 回调函数队列
  this.value = undefined // resolved 状态时保存接受值
  this.reason = undefined // rejected 状态时返回拒绝理由

  function resolve (value) {} // 接受函数
  function reject (reason) {} // 拒绝函数

  try {
    executor(resolve, reject) // 执行主任务
  } catch (reason) {
    reject(reason)
  }
}

// 链式调用
//   fn1 为接受状态的回调函数
//   fn2 为拒绝拒绝状态的回调函数，可选
Promise.prototype.then = function (fn1, fn2) {}

// 链式调用
//   用于捕获拒绝状态的
Promise.prototype.catch = function (fn) {}
```

接下来我们就一步步的来完善我们自己的 Promise 类吧

## 简单同步版本

第一个版本我们先假设传入 Promise 的任务(executor)都是同步方法，也就是说在执行 `then` 或 `catch` 方法之前，Promise 对象已经转变成 `resolved(fulfilled)` 或 `rejected` 状态了，先上代码：

```js
function Promise (executor) {
  // 主任务必须是一个函数
  //   主任务函数形式：function (resolve, reject) {}
  if (typeof executor !== 'function') {
    throw new TypeError(`Promise Constructor expect function, but get ${typeof executor}`)
  }
  const self = this
  this.state = 'pending'
  this.resolveCallbacks = []
  this.rejectCallbacks = []
  this.value = undefined
  this.reason = undefined

  function resolve (value) {
    // 等待状态 -> 接受状态
    if (self.state === 'pending') {
      self.state = 'resolved'
      self.value = value
    }
  }
  function reject (reason) {
    // 等待状态 -> 拒绝状态
    if (self.state === 'pending') {
      self.state = 'rejected'
      self.reason = reason
    }
  }

  try {
    executor(resolve, reject)
  } catch (err) {
    reject(err)
  }
}

Promise.prototype.then = function (fn1, fn2) {
  const self = this
  let p2

  // fn1 为接受状态时的回调函数，提供默认函数实现值穿透
  fn1 = typeof fn1 === 'function' ? fn1 : v => v
  // fn2 为拒绝状态时的回调函数，提供默认函数实现值穿透
  fn2 = typeof fn2 === 'function' ? fn2 : r => {throw r}

  if (self.state === 'resolved') {
    // 1. 接受状态
    return p2 = new Promise((resolve, reject) => {
      try {
        // 执行接受状态回调函数 fn1
        const x = fn1(self.value)
        // 回调函数返回值 x 将作为返回的 Promise 的接受值，使用 resolve 接受
        resolve(x)
      } catch (err) {
        reject(err)
      }
    })
  } else if (self.state === 'rejected') {
    // 2. 拒绝状态
    if (fn2 == null) {
      // 若未提供 fn2 则按原样传递下去
      return self
    }
    return p2 = new Promise((resolve, reject) => {
      try {
        // 执行拒绝状态回调函数 fn2
        const x = fn2(self.reason)
        // 维持原状态，返回的 Promise 再拒绝返回值
        reject(x)
      } catch (err) {
        reject(err)
      }
    })
  } else {
    // 3. 其他状态
    throw new TypeError(`unknown Promise state: ${self.state}`)
  }
}

Promise.prototype.catch = function (fn) {
  // 拒绝回调函数作为第二个参数
  return this.then(null, fn)
}

const p = new Promise((resolve, reject) => {
  console.log('main task start')
  resolve(1) // 同步函数任务
}).then(value => {
  console.log(`first then callback, with value = ${value}`)
  return 2
}).then(value => {
  console.log(`second then callback, with value = ${value}`)
  throw new Error('intentionally error occur')
}).then(value => {
  console.log(`third then callback, which should be skipped until meet rejected callback`)
}).catch(err => {
  console.log(`error occur, with message = "${err.message}"`)
}).catch(value => {
  console.log(`fourth then callback, with value = ${value}`)
})
```

- output 输出

```
main task start
first then callback, with value = 1
second then callback, with value = 2
error occur, with message = "intentionally error occur"
fourth then callback, with value = undefined
```

当我们构造 Promise 对象时，初始化好所有对象属性之后就会立即执行`任务(executor)`，传入任务的方法形式需要固定接受两个参数，用于`接受(resolve)`或`拒绝(reject)`任务结果，调用的同时会改变 Promise 的状态。

由于现阶段的任务假定是同步方法，所以 `then` 方法内可以直接根据当前状态直接执行回调函数并返回一个新的 Promise 对象。

最后，在第一代的同步版本里面有一个很重要的实现，甚至大大影响后面的异步版本：

```js
// fn1 为接受状态时的回调函数，提供默认函数实现值穿透
fn1 = typeof fn1 === 'function' ? fn1 : v => v
// fn2 为拒绝状态时的回调函数，提供默认函数实现值穿透
fn2 = typeof fn2 === 'function' ? fn2 : r => {throw r}
```

`then` 方法的这两句话实现的 Promise 对象的`值穿透`。当我们使用 `then` 和 `catch` 实现链式调用时：

```js
new Promise(function (resolve, reject) {})
  .then(res => {})
  .catch(err => {})
```

有时候拒绝任务时我们需要跳过几个 `then` 的调用直接跳到下一个 `catch` 方法，我们提供了默认的 `fn1`、`fn2` 允许我们的 Promise 类真正在运行时能够直接跳过当前 Promise 对象并直接包装继承变成下一个 Promise 对象。

```js
try {
  // 执行接受状态回调函数 fn1
  const x = fn1(self.value)
  // 回调函数返回值 x 将作为返回的 Promise 的接受值，使用 resolve 接受
  resolve(x)
} catch (err) {
  reject(err)
}
```

## Promise 返回值打包

基于第一个同步版本，我们注意到回调函数的返回值 x (`const x = fn1(self.value)`)可能是各种千奇百怪的值，也可能是另一个 Promise 对象，这时候我们可以实现一个 `resolvePromise` 用于解析并将回调函数的返回结果包装回下一个 Promise（`p2` 作为下一个 Promise 对象的句柄）

```js
function Promise (executor) {
  if (typeof executor !== 'function') {
    throw new TypeError(`Promise Constructor expect function, but get ${typeof executor}`)
  }
  const self = this
  this.state = 'pending'
  this.resolveCallbacks = []
  this.rejectCallbacks = []
  this.value = undefined
  this.reason = undefined

  function resolve (value) {
    if (self.state === 'pending') {
      self.state = 'resolved'
      self.value = value
    }
  }
  function reject (reason) {
    if (self.state === 'pending') {
      self.state = 'rejected'
      self.reason = reason
    }
  }

  try {
    executor(resolve, reject)
  } catch (err) {
    reject(err)
  }
}

Promise.prototype.then = function (fn1, fn2) {
  const self = this
  let p2

  fn1 = typeof fn1 === 'function' ? fn1 : v => v
  fn2 = typeof fn2 === 'function' ? fn2 : r => {throw r}

  if (self.state === 'resolved') {
    return p2 = new Promise((resolve, reject) => {
      try {
        const x = fn1(self.value)
        // resolve(x)
        // 使用 resolvePromise 重新包装返回值到 p2
        resolvePromise(p2, x, resolve, reject)
      } catch (err) {
        reject(err)
      }
    })
  } else if (self.state === 'rejected') {
    if (fn2 == null) {
      return self
    }
    return p2 = new Promise((resolve, reject) => {
      try {
        const x = fn2(self.reason)
        // reject(x)
        // 使用 resolvePromise 重新包装返回值到 p2
        resolvePromise(p2, x, resolve, reject)
      } catch (err) {
        reject(err)
      }
    })
  } else {
    throw new TypeError(`unknown Promise state: ${self.state}`)
  }
}

Promise.prototype.catch = function (fn) {
  return this.then(null, fn)
}

// 将返回值 x 重新包装成 Promise
function resolvePromise (p2, x, resolve, reject) {
  // 避免循环调用（等待自己）
  if (p2 === x) {
    return reject(new TypeError('Chaining cycle detected for Promise'))
  }
  // 如果返回值是一个 Promise
  if (x instanceof Promise) {
    // 则直接使用 x 的接受/拒绝回调函数
    x.then(value => {
      resolve(value)
    }).catch(err => {
      reject(err)
    })
    return
  }

  if ((x != null) && ((typeof x === 'object') || typeof x === 'function')) {
    // 1. 如果是对象 object 或是方法 funciton
    let called = false // 确保只有一个回调被触发
    try { // 访问 x.then 可能产生异常
      const then = x.then
      if (typeof then === 'function') {
        // 1.1 x.then 为 function，表示 x 是一个 thenable 对象/函数
        // 使用 Function.prototype.call 方法调用 then，语法 call(this, fn1, fn2)
        //   此时 x 作为回调的上下文，传入 then 方法需要的 fn1, fn2
        then.call(x, value => {
          // 检查 called，并将结果 x.then 接受状态下的结果再包装成 Promise
          if (called) {
            return
          }
          called = true
          return resolvePromise(p2, value, resolve, reject)
        }, err => {
          // 检查 called，x.then 被拒绝，则直接拒绝当前 x
          if (called) {
            return
          }
          called = true
          return reject(err)
        })
      } else {
        // 1.2 x 不是 thenable，直接接受
        resolve(x)
      }
    } catch (err) {
      if (called) {
        return
      }
      reject(err)
    }
  } else {
    // 2. 一般返回值，直接接受
    resolve(x)
  }
}

const p = new Promise((resolve, reject) => {
  console.log('main task start')
  resolve(1)
}).then(value => {
  console.log(`first then callback, with value = ${value}`)
  // 模拟返回值为 Promise 对象
  return new Promise((resolve, reject) => {
    resolve(2)
  })
}).then(value => {
  console.log(`second then callback, get value = ${value}`)
  throw new Error('intentionally error occur')
}).catch(err => {
  console.log(`error occur, with message = "${err.message}"`)
})
```

- output 输出

```
main task start
first then callback, with value = 1
second then callback, get value = 2
error occur, with message = "intentionally error occur"
```

我们定义好了 `resolvePromise` 函数用于封装回调函数的返回结果（详细步骤可以看代码中的注释），我们将 `then` 方法中的处理改为使用 `resolvePromise` 来将回调函数返回值 $x$ 包装到下一个 Promise 对象 $p2$ 内部

## 接受异步任务

上面的第二个版本实现了对回调函数返回值的包装作用，接下来我们要使 Promise 能够支持异步任务：

```js
function Promise (executor) {
  if (typeof executor !== 'function') {
    throw new TypeError(`Promise Constructor expect function, but get ${typeof executor}`)
  }
  const self = this
  this.state = 'pending'
  this.resolveCallbacks = []
  this.rejectCallbacks = []
  this.value = undefined
  this.reason = undefined

  function resolve (value) {
    if (value instanceof Promise) {
      return value.then(resolve, reject)
    }
    setTimeout(() => {
      if (self.state === 'pending') {
        self.state = 'resolved'
        self.value = value
        // 异步调用所有接受回调函数
        self.resolveCallbacks.map(resolve => resolve(value))
      }
    })
  }
  function reject (reason) {
    setTimeout(() => {
      if (self.state === 'pending') {
        self.state = 'rejected'
        self.reason = reason
        // 异步调用所有拒绝回调函数
        self.rejectCallbacks.map(reject => reject(reason))
      }
    })
  }

  try {
    executor(resolve, reject)
  } catch (err) {
    reject(err)
  }
}

Promise.prototype.then = function (fn1, fn2) {
  const self = this
  let p2

  fn1 = typeof fn1 === 'function' ? fn1 : v => v
  fn2 = typeof fn2 === 'function' ? fn2 : r => {throw r}

  if (self.state === 'resolved') {
    return p2 = new Promise((resolve, reject) => {
      // 改成异步执行回调
      setTimeout(() => {
        try {
          const x = fn1(self.value)
          resolvePromise(p2, x, resolve, reject)
        } catch (err) {
          reject(err)
        }
      })
    })
  } else if (self.state === 'rejected') {
    if (fn2 == null) {
      return self
    }
    return p2 = new Promise((resolve, reject) => {
      // 改成异步执行回调
      setTimeout(() => {
        try {
          const x = fn2(self.reason)
          resolvePromise(p2, x, resolve, reject)
        } catch (err) {
          reject(err)
        }
      })
    })
  } else if (self.state === 'pending') {
    // 新增 pending 状态，即调用 then 时 Promise 尚未完成
    // -> 向两种回调队列注册回调函数，resolve/reject 时异步调用
    return p2 = new Promise((resolve, reject) => {
      // 注册接受状态回调函数
      self.resolveCallbacks.push(value => {
        try {
          const x = fn1(value)
          resolvePromise(p2, x, resolve, reject)
        } catch (err) {
          reject(err)
        }
      })
      if (fn2 == null) {
        return
      }
      // 注册拒绝状态回调函数
      self.rejectCallbacks.push(reason => {
        try {
          const x = fn2(reason)
          resolvePromise(p2, x, resolve, reject)
        } catch (err) {
          reject(err)
        }
      })
    })
  } else {
    throw new TypeError(`unknown Promise state: ${self.state}`)
  }
}

Promise.prototype.catch = function (fn) {
  return this.then(null, fn)
}

function resolvePromise (p2, x, resolve, reject) {
  if (p2 === x) {
    return reject(new TypeError('Chaining cycle detected for Promise'))
  }

  if (x instanceof Promise) {
    x.then(value => {
      resolve(value)
    }).catch(err => {
      reject(err)
    })
    return
  }

  if ((x != null) && ((typeof x === 'object') || typeof x === 'function')) {
    let called = false
    try {
      const then = x.then
      if (typeof then === 'function') {
        then.call(x, value => {
          if (called) {
            return
          }
          called = true
          return resolvePromise(p2, value, resolve, reject)
        }, err => {
          if (called) {
            return
          }
          called = true
          return reject(err)
        })
      } else {
        resolve(x)
      }
    } catch (err) {
      if (called) {
        return
      }
      reject(err)
    }
  } else {
    resolve(x)
  }
}

const p = new Promise((resolve, reject) => {
  console.log('main task start')
  // 模拟异步函数，延迟 1s
  setTimeout(() => {
    resolve(1)
  }, 1000)
}).then(value => {
  console.log(`first then callback, with value = ${value}`)
  return new Promise((resolve, reject) => {
    resolve(2)
  })
}).then(value => {
  console.log(`second then callback, get value = ${value}`)
  throw new Error('intentionally error occur')
}).catch(err => {
  console.log(`error occur, with message = "${err.message}"`)
})
```

- output 输出

```
main task start
first then callback, with value = 1
second then callback, get value = 2
error occur, with message = "intentionally error occur"
```

我们透过将 `resolve` 和 `reject` 方法内部包装到一个 `setTimeout` 方法中，将整个接受/拒绝的方法延迟到下一个异步点(这边可以参考<a href="https://blog.csdn.net/weixin_44691608/article/details/106485760">JS基礎：Event Loop事件循環機制</a>)才执行；同时我们也将 `then` 方法中对于已接受/拒绝状态的处理也都包装到异步函数 `setTimeout` 里面，另外多加一个当执行 `then` 方法是状态还在 `pending` 时（表示主任务为异步尚未完成），我们就将两个回调函数先放入队列中（`resolveCallbacks`、`rejectCallbacks`），等到 `resolve/reject` 的时候会执行所有队列中的回调函数

到此我们的手写 Promise 对象方法都完成啦，已经可以完成常见的链式调用，同时兼容同步/异步任务的回调等，现在就差 Promise 类的静态方法了

## 添加静态方法

接下来我们实现两个静态方法，相关接口可以回到上面再看一眼：

- `Promise.all(iterable)`
- `Promise.retry(task, times, delay)`

```js
function Promise (executor) {
  if (typeof executor !== 'function') {
    throw new TypeError(`Promise Constructor expect function, but get ${typeof executor}`)
  }
  const self = this
  this.state = 'pending'
  this.resolveCallbacks = []
  this.rejectCallbacks = []
  this.value = undefined
  this.reason = undefined

  function resolve (value) {
    if (value instanceof Promise) {
      return value.then(resolve, reject)
    }
    setTimeout(() => {
      if (self.state === 'pending') {
        self.state = 'resolved'
        self.value = value
        self.resolveCallbacks.map(resolve => resolve(value))
      }
    })
  }
  function reject (reason) {
    setTimeout(() => {
      if (self.state === 'pending') {
        self.state = 'rejected'
        self.reason = reason
        self.rejectCallbacks.map(reject => reject(reason))
      }
    })
  }

  try {
    executor(resolve, reject)
  } catch (err) {
    reject(err)
  }
}

Promise.prototype.then = function (fn1, fn2) {
  const self = this
  let p2

  fn1 = typeof fn1 === 'function' ? fn1 : v => v
  fn2 = typeof fn2 === 'function' ? fn2 : r => {throw r}

  if (self.state === 'resolved') {
    return p2 = new Promise((resolve, reject) => {
      setTimeout(() => {
        try {
          const x = fn1(self.value)
          resolvePromise(p2, x, resolve, reject)
        } catch (err) {
          reject(err)
        }
      })
    })
  } else if (self.state === 'rejected') {
    return p2 = new Promise((resolve, reject) => {
      setTimeout(() => {
        try {
          const x = fn2(self.reason)
          resolvePromise(p2, x, resolve, reject)
        } catch (err) {
          reject(err)
        }
      })
    })
  } else if (self.state === 'pending') {
    return p2 = new Promise((resolve, reject) => {
      self.resolveCallbacks.push(value => {
        try {
          const x = fn1(value)
          resolvePromise(p2, x, resolve, reject)
        } catch (err) {
          reject(err)
        }
      })
      self.rejectCallbacks.push(reason => {
        try {
          const x = fn2(reason)
          resolvePromise(p2, x, resolve, reject)
        } catch (err) {
          reject(err)
        }
      })
    })
  } else {
    throw new TypeError(`unknown Promise state: ${self.state}`)
  }
}

Promise.prototype.catch = function (fn) {
  return this.then(null, fn)
}

function resolvePromise (p2, x, resolve, reject) {
  if (p2 === x) {
    return reject(new TypeError('Chaining cycle detected for Promise'))
  }

  if (x instanceof Promise) {
    x.then(value => {
      resolve(value)
    }).catch(err => {
      reject(err)
    })
    return
  }

  if ((x != null) && ((typeof x === 'object') || typeof x === 'function')) {
    let called = false
    try {
      const then = x.then
      if (typeof then === 'function') {
        then.call(x, value => {
          if (called) {
            return
          }
          called = true
          return resolvePromise(p2, value, resolve, reject)
        }, err => {
          if (called) {
            return
          }
          called = true
          return reject(err)
        })
      } else {
        resolve(x)
      }
    } catch (err) {
      if (called) {
        return
      }
      reject(err)
    }
  } else {
    resolve(x)
  }
}

Promise.all = (arr = []) => {
  if (!Array.isArray(arr)) {
    throw new TypeError(`arguments for Promise.all must be an Array`)
  }
  return new Promise((resolve, reject) => {
    const values = []
    arr.map(p => {
      p.then(value => {
        values.push(value)
        if (values.length === arr.length) {
          return resolve(values)
        }
      }).catch(err => {
        reject(err)
      })
    })
  })
}

Promise.retry = (p, times, delay) => {
  return new Promise((resolve, reject) => {
    function attempt () {
      p().then(value => {
        resolve(value)
      }).catch(err => {
        if (times === 0) {
          reject(err)
        } else {
          times--;
          setTimeout(attempt, delay)
        }
      })
    }
    attempt()
  })
}

const task1 = new Promise((resolve, reject) => {resolve(1)})
const task2 = new Promise((resolve, reject) => {resolve(2)})
const task3 = new Promise((resolve, reject) => {resolve(3)})
const task4 = new Promise((resolve, reject) => {reject(new Error('intentionally error'))})

const p = Promise.all([task1, task2, task3])
p.then(values => {
  console.log(values)
})

const p2 = Promise.all([task2, task3, task4])
p2.then(values => {
  console.log('It will print if all tasks resolved')
}).catch(err => {
  console.log('some tasks were rejected')
  console.log(`error occur with message: ${err.message}`)
})

const p3 = Promise.retry(() => {
  console.log('try')
  return new Promise((resolve, reject) => {
    reject('always reject')
  })
}, 1, 1000)

p3.then(value => {
  console.log(`retry for 3 times ${value}`)
}).catch(err => {
  console.log(`retry still reject, with message: ${err}`)
})
```

- output 输出

```
try
[ 1, 2, 3 ]
some tasks were rejected
error occur with message: intentionally error
try
retry still reject, with message: always reject
```

# 结语

到此我们就全部完成啦，网上还有相关的<a href="https://github.com/promises-aplus/promises-tests">手写 Promise 测试</a>，读者可以在自己实现之后使用官方测试。
