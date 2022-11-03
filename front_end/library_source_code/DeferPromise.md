# defer-promise 源码解析(Npm library)

@[TOC](文章目录)

<!-- TOC -->

- [defer-promise 源码解析(Npm library)](#defer-promise-源码解析npm-library)
- [基本信息](#基本信息)
- [源码解析](#源码解析)
- [使用](#使用)
- [参考链接](#参考链接)

<!-- /TOC -->

# 基本信息

- version：`v3.0.0`
- 功能：利用 Promise 的特性实现两块异步代码同步化
- [传送门：75lb/defer-promise - Github](https://github.com/75lb/defer-promise/blob/v3.0.0/index.js)

# 源码解析

我们先来看看源码，非常简短

```js
/**
 * @module defer-promise
 */

function defer () {
  if (typeof Promise !== 'undefined' && Promise.defer) {
    return Promise.defer()
  } else {
    var deferred = {}
    deferred.promise = new Promise(function (resolve, reject) {
      deferred.resolve = resolve
      deferred.reject = reject
    })
    return deferred
  }
}

export default defer
```


先构建一个 Promise 对象，然后将 `resolve`、`reject` 方法透出并返回，如此一来我们就可以如下去使用

# 使用

```ts
let solver;

function invoker() {
  solver = defer(); // start task

  // ... do something async

  // setTimeout for example
  setTimeout(() => {
    solver.resolve();
  }, 10 * 1000);
}

// other place
function otherTask() {
  if (solver) {
    solver.then(() => {
      // ...
    })
  }
}

// use async await
async function asyncTask() {
  if (solver) {
    // wait for solver resolved
    await solver;
  }
}
```

这是一种非常好的异步代码同步化的一种实践，将异步任务以 Promise 来表示，并透过将 resolve、reject 方法在外部显示的指定任务的完成与否。

相信大家后续在写更复杂的系统的时候总有一天会用到，源码实际上也非常简单，不引库自己造一个也是非常方便的。

# 参考链接

| Title                       | Link                                                                                                                     |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| 75lb/defer-promise - Github | [https://github.com/75lb/defer-promise/blob/v3.0.0/index.js](https://github.com/75lb/defer-promise/blob/v3.0.0/index.js) |
