# ES6: Promise 對象

@[TOC](文章目錄)

<!-- TOC -->

- [ES6: Promise 對象](#es6-promise-對象)
    - [簡介](#簡介)
    - [參考](#參考)
- [正文](#正文)
    - [Callback 回調函數](#callback-回調函數)
    - [Callback Hell 回調地獄](#callback-hell-回調地獄)
    - [Promise 對象](#promise-對象)
        - [Promise 創建](#promise-創建)
        - [應用：圖片加載](#應用圖片加載)
        - [應用：Ajax 操作](#應用ajax-操作)
        - [狀態傳遞](#狀態傳遞)
    - [Promise 實例方法](#promise-實例方法)
        - [`Promise.prototype.then()`](#promiseprototypethen)
            - [鏈式調用：直接返回值](#鏈式調用直接返回值)
            - [鏈式調用：返回新的 Promise](#鏈式調用返回新的-promise)
        - [`Promise.prototype.catch()`（推薦）](#promiseprototypecatch推薦)
            - [狀態更新](#狀態更新)
            - [異常冒泡](#異常冒泡)
            - [拋出異常順序](#拋出異常順序)
    - [Promise 靜態方法](#promise-靜態方法)
        - [`Promise.all()`](#promiseall)
        - [`Promise.race()`](#promiserace)
        - [`Promise.resolve()`](#promiseresolve)
            - [參數為 Promise 實例](#參數為-promise-實例)
            - [參數為 thenable 對象](#參數為-thenable-對象)
            - [其他任意參數](#其他任意參數)
            - [無參數](#無參數)
        - [`Promise.reject()`](#promisereject)
    - [Promise 自定義擴展（推薦）](#promise-自定義擴展推薦)
        - [`Promise.done()`](#promisedone)
        - [`Promise.finally()`](#promisefinally)
- [結語](#結語)

<!-- /TOC -->

## 簡介

遠古時代（ES5 乃至 ES3 以前)曾經有種現象叫做`回調地獄`，當時候如果出現異步函數需要處理，則必須將異步之後的操作透過回調函數的形式傳入，一層套一層變形成所謂的回調地獄。然而在 ES6 提出了 `Promise`，有人翻作承諾，好比這個函數對主線程的承諾會有回來的一天，必須為它回來的時候(then 之後)準備好後續操作。雖然 Promise 在一定程度上解決了回調地獄，將層層嵌套的 callback 轉成鏈式函數調用的形式，但卻依然沒有擺脫傳遞函數的雛形，或需 ES7 的 async/await 才是真正的終極解決之道。不過即便如此，ES7 的 async/await 也只是 Promise 的變體，完整的學習 Promise 依舊對於理解後續 ES7 的終極解法有一定的幫助，

## 參考

<table>
    <tr>
        <td>打不死的回调地狱？</td>
        <td><a href="https://zhuanlan.zhihu.com/p/39580112">https://zhuanlan.zhihu.com/p/39580112</a></td>
    </tr>
    <tr>
        <td>MDN:使用 Promise</td>
        <td><a href="https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Guide/Using_promises">https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Guide/Using_promises</a></td>
    </tr>
    <tr>
        <td>ES6 promise对象</td>
        <td><a href="http://caibaojian.com/es6/promise.html">http://caibaojian.com/es6/promise.html</a></td>
    </tr>
</table>

# 正文

## Callback 回調函數

在此之前，我們先來解釋一下何謂回調函數。通常回調函數的約定形式如下：

```js
function(error, result) {
    if(error) {
        // when error occur
    } else {
        // when function success
    }
}
```

回調函數作為一個參數傳遞到另一個主流程的方法裡面，它可以是一個同步也可以是一個異步的方法。回調函數在形成閉包(closure)的同時，也擴展了一個方法的後置流程的開放性，透過對一個方法傳入 callback 可以客製化後置方法行為。

常出現的場景通常是**讀寫文件**、**向遠端服務器發送請求**等異步方法，需要等到某個異步程序結束之後才繼續執行後置流程的方法。

## Callback Hell 回調地獄

有了回調函數，看似對一個異步方法提供開放性的能力，能夠客製化異步方法的後置流程，然而遠古時代大家就發現這種用法常常會形成一種回調地獄，直接看代碼感受最深刻：（引用參考條目一的代碼）

```js
fs.readdir(source, function (err, files) {
  if (err) {
    console.log('Error finding files: ' + err)
  } else {
    files.forEach(function (filename, fileIndex) {
      console.log(filename)
      gm(source + filename).size(function (err, values) {
        if (err) {
          console.log('Error identifying file size: ' + err)
        } else {
          console.log(filename + ' : ' + values)
          aspect = values.width / values.height
          widths.forEach(
            function (width, widthIndex) {
              height = Math.round(width / aspect)
              console.log(
                'resizing ' + filename + 'to ' + height + 'x' + height
              )
              this.resize(width, height).write(
                dest + 'w' + width + '_' + filename,
                function (err) {
                  if (err) console.log('Error writing file: ' + err)
                }
              )
            }.bind(this)
          )
        }
      })
    })
  }
})
```

我們先暫時忽略這段程序本身的邏輯細節，我們可以看到在第一行、第七行、第十五行都傳入了一個回調函數，看似方便的回調函數卻形成嵌套層數如次之多(尤其最後的`}`, `)`)的程序令人眼花撩亂。代碼是用來讀的，我們從邏輯的視角來看，回調函數似乎應該作為主流程的後置流程，應該屬於同個層級，卻由於傳遞方法的寫法，造成回調函數必須不斷的嵌套在另一個函數調用的參數之內，此時卻使得程序段的用意不夠明朗。因此在 ES6 制定了 `Promise`，作為回調地獄的解決辦法。

## Promise 對象

在開始使用 Promise 之前，我們先來明確一下 Promise 對象的狀態。由於回調函數往往需要作用於一個不確定結束時間的方法（異步函數），所以這時候就需要隱式的記錄一下主方法的處理狀態，分成三種：

1. `Pending 處理中`：主方法還在執行中途
2. `Resolved 成功`：主方法完成處理，返回一個結果(res)
3. `Rejected 失敗`：主方法處理失敗，返回一個錯誤(err)

了解了 Promise 對象的三種狀態，就可以開始創建自己的 Promise 對象了

### Promise 創建

Promise 的構造函數需要傳入一個擁有兩個參數(`resolve` 和 `reject`)的方法，作為參數的方法即為主流程方法，可以也通常是一個異步方法。`resolve` 和 `reject` 由 JS 引擎提供，作為處理成功或失敗時調用的方法，可以改變 Promise 對象的狀態：`resolve` 將狀態轉為 `Resolved`；而 `reject` 將狀態轉為 `reject`

```js
let promise = new Promise(function(resolve, reject) {
    // main process
    // ...
    if(/* process success */) {
        resolve(/* success result */)
    } else {
        reject(/* error */)
    }
})
```

resolve 將調用 then 方法中傳入的回調函數，並將傳入的結果作為回調函數的參數調用；相似的 reject 方法則是調用 then 方法的第二個回調函數或是 catch 方法的回調函數，不過通常傳入的是 Error 對象。調用形式如下：

```js
promise.then(
  function (res) {
    // when success
  },
  function (err) {
    // when error
  }
)
// 或是
promise.then(
  (res) => {
    // when success
  },
  (err) => {
    // when error
  }
)
```

### 應用：圖片加載

（引用參考三代碼）

```js
function loadImageAsync(url) {
  return new Promise(function (resolve, reject) {
    const image = new Image()

    image.onload = function () {
      resolve(image)
    }

    image.onerror = function () {
      reject(new Error(`Could not load image on ${url}`))
    }

    image.src = url
  })
}
```

### 應用：Ajax 操作

Ajax(Asynchronous JavaScrip and XML)，詳細可查閱其他文章

（引用參考三代碼）

```js
function getJSON(url) {
  const promise = new Promise(function (resolve, reject) {
    const client = new XMLHttpRequest()
    client.open('GET', url)
    client.onreadystatechange = handler
    client.responseType = 'json'
    client.setRequestHeader('Accept', 'application/json')
    client.send()

    function handler() {
      if (this.readyState !== 4) {
        return
      }
      if (this.status === 200) {
        resolve(this.response)
      } else {
        reject(new Error(this.statusText))
      }
    }
  })

  return promise
}

getJSON('/posts.json').then(
  (res) => {
    console.log(`content: ${res}`)
  },
  (err) => {
    console.log(`error occur: ${err.message}`)
  }
)
```

即透過 Promise 對象對 XMLHttpRequest 對象的封裝，並透過 resolve, reject 等方法將回調函數幫綁定到後置流程(onreadystatechange)的位置

### 狀態傳遞

reject 通常傳入 Error 對象；而 resolve 除了可以傳入正常值作為 then 所接受的回調函數的參數之外，還可以傳入另一個 Promise 對象，即一個 Promise 對象的結果為另一個 Promise 對象的結果，形成狀態傳遞（可限定異步順序）：

```js
let p1 = new Promise(function (resolve, reject) {
  setTimeout(() => reject(new Error('p1 error')), 3000)
})

let p2 = new Promise(function (resolve, reject) {
  setTimeout(() => resolve(p1), 1000)
})

p2.then((res) => {
  console.log(res)
}).catch((err) => {
  console.log(err.message)
})
```

p1 為一個 Promise 對象，作為 p2 的 resolve 的參數，因此一秒後 p2 的狀態取決於 p1 的狀態，也就是再兩秒後變為 `Rejected` 因而觸發 then 的第二個回調函數。

## Promise 實例方法

一個 Promise 對象擁有兩種實例方法

1. `Promise.prototype.then()`
2. `Promise.prototype.catch()`

### `Promise.prototype.then()`

`then` 定義在 Promise.prototype 上，作用為為 Promise 對象狀態改變添加回調函數。第一個參數為 `Resolved` 狀態下的回調函數，第二個參數(可選)為 `Rejected` 狀態下的回調函數，上面也提到過了：

```js
promise.then(
  (res) => {
    // callback when resolved
  },
  (err) => {
    // callback when rejected
  }
)
```

#### 鏈式調用：直接返回值

而 `then` 方法返回一個新的 Promise 對象，而 `then` 方法內部的返回值作為下一次調用的參數值。

```js
let promise = new Promise(function (resolve, reject) {
  let r = Math.floor(Math.random() * 100)
  console.log(`r = ${r}`)
  if (r >= 50) {
    resolve(r)
  } else {
    reject(new Error(`r = ${r} is lower than 50`))
  }
})

promise
  .then((res) => {
    console.log(`promise resolve with res = ${res}`)
    return (res -= 50)
  })
  .then((res) => {
    console.log(`next res = ${res}`)
  })
// output:
//   r = 70
//   promise resolve with res = 70
//   next res = 20
```

#### 鏈式調用：返回新的 Promise

再者，`then` 還能夠主動返回一個新的 promise 對象，提供鍊式操作(例如：連續請求或鏈式請求)

```js
function startRandomTest() {
  let promise = new Promise(function (resolve, reject) {
    let r = Math.floor(Math.random() * 100)
    console.log(`r = ${r}`)
    if (r >= 50) {
      resolve(r)
    } else {
      reject(new Error(`r = ${r} is lower than 50`))
    }
  })
  return promise
}

startRandomTest()
  .then((res) => {
    console.log(`promise resolve with res = ${res}`)
    return startRandomTest()
  })
  .then(
    (res) => {
      console.log(`promise 2 resolve with res = ${res}`)
    },
    (err) => {
      console.log(`unexpected number 2: ${err.message}`)
    }
  )
```

- 說明：第一次 then 方法指定第一次調用 startRandomTest 成功時則在觸發第二次 startRandomTest，實現鏈式的多個異步請求操作。

### `Promise.prototype.catch()`（推薦）

前面我們都透過向 then 方法傳遞兩個回調函數，但其實這樣是**不推薦**的。

Promise.prototype.catch 提供了另一種方法，作為 `.then(null, rejection)` 的別名。Promise 對象中的 `rejected` 狀態可以被視為一種拋出異常，而 catch 既能捕獲 `rejected` 狀態的回調也能捕獲一般的異常拋出，下面兩段代碼是等價的：

```js
// rejected 回調
let promise = new Promise(function (resolve, reject) {
  reject(new Error('rejected state'))
})
promise.catch((err) => {
  console.log(`catch: ${err.message}`)
})

// 一般異常
let promise = new Promise(function (resolve, reject) {
  throw new Error('normal Exception')
})
promise.catch((err) => {
  console.log(`catch: ${err.message}`)
})
```

#### 狀態更新

一個 Promsie 對象在 resolve 也就是狀態變成 `resolved` 之後將不再捕獲異常，同時也不會向全局拋出異常，錯誤將被隱式的吃掉：

```js
let promise = new Promise(function (resolve, reject) {
  resolve('success message')
  throw new Error('throw error')
})

promise
  .then((res) => {
    console.log(res)
  })
  .catch((err) => {
    console.log(err.message)
  })
// output:
//     success message
```

#### 異常冒泡

Promise 函數內部拋出的異常(包括 `rejected` 狀態下的回調)有類似冒泡的性質，會一直往後找直接找到最近的 catch 回調：

```js
function buildPromise(succeed) {
  let promise = new Promise(function (resolve, reject) {
    resolve('success')
  })
  return promise
}

buildPromise()
  .then((res) => {
    console.log('first then callback')
    return buildPromise()
  })
  .then((res) => {
    console.log('second then callback')
  })
  .catch((err) => {
    console.log('catch callback')
  })
// output:
//    catch callback
```

- 說明：第一個 buildPromise 返回的 Promise 對象直接拋出一個異常，所以跳過兩次 then 回調直接找到最近的 catch 回調。

#### 拋出異常順序

catch 捕獲的異常或是 `rejected` 回調跟方法調用的順序有關。

第一段代碼演示在 catch 調用後的 then 方法內部報錯，由於沒有 catch 捕獲也沒有被全局捕獲（嚴格來說其實還是會觸發全局的 `UnhandledRejection` 事件）。

```js
new Promise(function (resolve, reject) {
  reject(new Error('some error'))
})
  .catch((err) => {
    console.log(`err message: ${err.message}`)
    return 'after catch message'
  })
  .then((res) => {
    console.log(`res: ${res}`)
    throw new Error('throw in then with no catch after')
  })
// output:
// err message: some error
// res: after catch message
```

第二段代碼演示 catch 函數的鏈式調用，在回調函數內再次拋出異常：

```js
new Promise(function (resolve, reject) {
  reject(new Error('some error'))
})
  .catch((err) => {
    console.log(`err message: ${err.message}`)
    throw new Error('new error throw by catch callbcak')
  })
  .catch((err) => {
    console.log(`err message 2: ${err.message}`)
  })
// output:
// err message: some error
// err message 2: new error throw by catch callbcak
```

## Promise 靜態方法

除了透過構造函數 `new Promise()` 創建 Promise 對象之外，ES6 另外提供幾種靜態方法：透過組合多個 Promise 對象作為一個 Promise 對象；或是直接傳入值作為 `resolved` 或 `rejected` 狀態回調的參數。

靜態方法如下：

1. `Promise.all()`
2. `Promise.race()`
3. `Promise.resolve()`
4. `Promise.reject()`

### `Promise.all()`

all 可以看作將多個 Promise 對象打包成一個連續任務，只有當所有任務進入 `fulfilled` 狀態(也就是 `resolved` 狀態)的時候才會執行後面的 then 回調，而只要有一個 Promise 對象進入 `rejected` 狀態就跳到執行 catch 回調：

```js
function createPromise(ms, succeed) {
  if (succeed) {
    return new Promise(function (resolve, reject) {
      setTimeout(() => {
        resolve(`resolve in ${ms}`)
      }, ms)
    })
  } else {
    return new Promise(function (resolve, reject) {
      setTimeout(() => {
        reject(new Error(`reject in ${ms}`))
      }, ms)
    })
  }
}

Promise.all([
  createPromise(1000, true),
  createPromise(2000, true),
  createPromise(3000, true)
]).then((results) => {
  console.log(results)
})
// 3 秒後執行 resolved 回調
// results = [ 'resolve in 1000', 'resolve in 2000', 'resolve in 3000' ]

Promise.all([
  createPromise(1000, true),
  createPromise(2000, false),
  createPromise(3000, true)
])
  .then((results) => {
    console.log(results)
  })
  .catch((err) => {
    console.log(err.message)
  })
// 2 秒後跳過 resolved 回調直接執行 catch 回調
// output: reject in 2000
```

### `Promise.race()`

race 與 all 不同的是，all 可以看作多個 Promise 對象作為一個連續任務來看待；而 race 則像是多個任務之間的競爭，只有最先改變狀態的對象才能執行回調函數：

```js
Promise.race([
  createPromise(3000, true),
  createPromise(1000, true),
  createPromise(2000, true)
]).then((res) => {
  console.log(res)
})
// output: resolve in 1000

Promise.race([
  createPromise(3000, false),
  createPromise(1000, false),
  createPromise(2000, false)
]).catch((err) => {
  console.log(err.message)
})
// output: reject in 1000

Promise.race([
  createPromise(3000, false),
  createPromise(1000, true),
  createPromise(2000, false)
])
  .then((res) => {
    console.log(res)
  })
  .catch((err) => {
    console.log(err.message)
  })
// output: resolve in 1000
```

- 說明 1：第二個回調函數最早轉為 resolved 狀態，所以返回的 res 為第二個回調函數傳入的結果
- 說明 2：同理，最早轉為 rejected 狀態的對象執行回調函數
- 說明 3：由第三段代碼示例我們能看出 race 其實就是競爭出一個最早做出反應的 Promise 獲得回調函數的調用權，一但競爭出第一個其他的 Promise 對象將不再做任何處理。

### `Promise.resolve()`

Promise.resolve 需要根據傳入的參數做區分：

#### 參數為 Promise 實例

第一種是傳入一個 Promise 對象，則 Promise.resolve 不做任何行為直接返回傳入的對象：（這邊直接沿用上面所定義的 `createPromise` 方法）

```js
Promise.resolve(createPromise(1000, true)).then((res) => {
  console.log(res)
})
// 1 秒後執行 resolved 回調
// output: resolve in 1000

Promise.resolve(createPromise(1000, false)).catch((err) => {
  console.log(err.message)
})
// 1 秒後執行 rejected 回調
// output: reject in 1000
```

#### 參數為 thenable 對象

`thenable` 對象就是具有 then 方法的對象，此時 then 方法作為 Promise 構造函數參數傳入，相當於這個 Promise 對象的主流程

我們首先仿造 `createPromise` 定義一個返回 thenable 對象的函數：

```js
function createThenable(ms, succeed) {
  if (succeed) {
    const thenable = {
      then(resolve, reject) {
        setTimeout(() => {
          resolve(`resolve in ${ms}`)
        }, ms)
      }
    }
    return thenable
  } else {
    const thenable = {
      then(resolve, reject) {
        setTimeout(() => {
          reject(new Error(`reject in ${ms}`))
        }, ms)
      }
    }
    return thenable
  }
}
```

接下來調用 Promise.resolve 將 thenable 對象作為參數傳入，表現行為就好像 then 方法為 Promise 對象主流程

```js
Promise.resolve(createThenable(1000, true)).then((res) => {
  console.log(res)
})
// output: resolve in 1000

Promise.resolve(createThenable(1000, false)).catch((err) => {
  console.log(err.message)
})
// output: reject in 1000
```

#### 其他任意參數

如果傳入的參數不是 Promise 實例，也不是 thenable 對象，那就會創建一個空的 Promsie 對象並將狀態直接轉為 `resolved`，而參數直接作為 `resovled` 回調的參數傳入：

```js
let person = {
  id: 0,
  name: 'John'
}
Promise.resolve(person).then((res) => {
  console.log(res)
})
// output: { id: 0, name: 'John' }
```

也就是說傳入其他任意參數的時候的形式如下：

```js
Promise.resolve(something)
// 等價於
new Promise((resolve) => resolve(something))
```

#### 無參數

與傳入任意參數相似，不過 `resolved` 回調沒有接收參數

```js
Promise.resolve().then((res) => {
  console.log(res)
})
// output: undefined
```

### `Promise.reject()`

`Promise.reject()` 的參數用法與 `Promise.resolve()` 完全一致，狀態的改變從變為 `resolved` 變為 `rejected` 而已：

```js
Promise.reject().catch((err) => {
  console.log(`catch callback, err = ${err}`)
})
// output: catch callback, err = undefined
```

也就是說下面的兩段代碼等價：

```js
Promise.reject(something)
// 等價於
new Promise((_, reject) => reject(something))
```

## Promise 自定義擴展（推薦）

除了 ES6 添加的方法以外，另外有兩個推薦的字定義用法，分別是 `done` 和 `finally` 方法，這兩個方法需要透過程序員主動修改 Promise 原型。

### `Promise.done()`

Promise 的鏈式調用其實具有一定的風險，可能因為沒有正確結尾造成額外的調用如下：

```js
Promise.resolve()
  .then((res) => {
    console.log(`first then callback, res = ${res}`)
  })

  .then((res) => {
    console.log('unexpected action')
  })
```

或是 Promise 主流程中未被捕獲的異常將被隱式的忽略而為正常處理：

```js
Promise.reject()
// not Excetion
// only invoke `unhandleRejection` event at global
```

因此我們可以定義一個 Promise 對象的結尾方法，並使其調用的形式與 then 相仿：

```js
Promise.prototype.done = function (onResolved, onRejected) {
  this.then(onResolved, onRejected).catch((err) => {
    setTimeout(() => {
      throw err
    }, 0)
  })
}

Promise.resolve()
  .then((res) => {})
  .then((res) => {})
  .done()
```

- 說明：內部調用一次 then 之後補足一個向全局拋出異常的 catch 調用，然後不返回任何值即可作為 Promise 對象結束方法。

### `Promise.finally()`

與 `Promise.done()` 類似的是，finally 的用意在於不管狀態如何都必須執行傳入的回調方法，也就是 `resolved` 和 `rejected` 的回調方法最終都要執行同樣的 callback：

```js
Promise.prototype.finally = function (callback) {
  let P = this.constructor
  return this.then(
    (res) => P.resolve(callback()).then(() => value),
    (err) =>
      P.resolve(callback()).then(() => {
        throw err
      })
  )
}
```

# 結語

Promise 算是一半成功解決了回調地獄的窘境，但是過長的鏈式調用或是過多的回調包裝其實多少還是延續了回調函數的一貫風格，老實說我個人覺得 Promise 還是太過醜陋，下期我們介紹 async/await，個人認為這就是異步函數作為同步函數調用的終極解決辦法。
