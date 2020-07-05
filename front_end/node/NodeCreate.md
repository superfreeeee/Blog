# Node：入門 + 建構 Web server

@[TOC](文章目錄)

<!-- TOC -->

- [Node：入門 + 建構 Web server](#node入門--建構-web-server)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [使用 Node 環境](#使用-node-環境)
    - [原來的 JavaScript](#原來的-javascript)
    - [Node 環境下的 JavaScript](#node-環境下的-javascript)
      - [Install 安裝](#install-安裝)
      - [Run 運行](#run-運行)
      - [npm(Node package manager) 包管理器](#npmnode-package-manager-包管理器)
  - [Node 環境](#node-環境)
    - [全局 Global 對象](#全局-global-對象)
    - [全局屬性](#全局屬性)
      - [所有屬性](#所有屬性)
    - [模塊化](#模塊化)
      - [內置模塊](#內置模塊)
  - [使用 Node 建構一個 Web server](#使用-node-建構一個-web-server)
- [結語](#結語)

<!-- /TOC -->

## 簡介

原來的 JS 是為了服務 HTML 而誕生的，不過經過多年的發展，JS 開發者已經不滿足於僅僅只存在於瀏覽器(browser)上的服務。

Node.js 就是為此而誕生的，他是一個能夠在`本地(localhost)`運行的 JavaScript 環境，採用 V8 引擎來解析並運行 JavaScript，並且採用 CommonJS 模塊化協議，將一個個 `.js` 文件視為一個個`模塊(module)`，與`全局對象(global)`分離，避免命名空間和作用域的污染。

在 Node.js 的基礎之上已經出現大量的框架和應用，有 `npm` 作為 JS 項目的包管理系統工具、`Bootstrap`, `Vue`, `React`, `Angular` 等響應式框架以及以此為基礎開發的延伸框架。當今的 JS 開發環境已經離不開 node 環境，接下來就讓我們來一探究竟吧。

## 參考

<table>
  <tr>
    <td>Node.js-wiki</td>
    <td><a href="https://zh.wikipedia.org/wiki/Node.js">https://zh.wikipedia.org/wiki/Node.js</a></td>
  </tr>
  <tr>
    <td>Node.js 全局對象</td>
    <td><a href="http://www.w3big.com/zh-TW/nodejs/nodejs-global-object.html">http://www.w3big.com/zh-TW/nodejs/nodejs-global-object.html</a></td>
  </tr>
  <tr>
    <td>nodejs内置模块有哪些？</td>
    <td><a href="https://m.html.cn/qa/node-js/10830.html">https://m.html.cn/qa/node-js/10830.html</a></td>
  </tr>
  <tr>
    <td>第一次建置node.js開發環境和安裝npm就上手！</td>
    <td><a href="https://ithelp.ithome.com.tw/m/articles/10199058">https://ithelp.ithome.com.tw/m/articles/10199058</a></td>
  </tr>
</table>

# 正文

## 使用 Node 環境

在一開始，我們先不進入 node 環境的內容，我們先來看看原本的 JS 使用方式，以及如何啟動 node 環境來運行 JS

### 原來的 JavaScript

最原初的 JavaScript 作為 HTML 的附屬，一開始僅僅起到一些輔助作用，動態的操作 DOM 等，充其量是作為 HTML 的裝飾和簡單的互動。因此原本的 JavaScript 只能透過`瀏覽器`解析 HTML 時引入 JS 代碼。

我們首先需要準備兩個文件：`index.html`、`index.js`

- index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <div id="app">Hello World</div>

    <script src="index.js"></script>
  </body>
</html>
```

- index.js

```js
console.log('hello')
```

接下來我們只需要使用瀏覽器打開 HTML 文件，就會自動引入並執行 JS 代碼，結果如下圖：

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/origin_js.png)

### Node 環境下的 JavaScript

然而隨著時間遷移，開發者們開始意識到 JavaScript 的潛力和應用面的廣泛。我們開始需要一個獨立於`瀏覽器(browser)`之外的運行環境，以便能夠運行在本地(native)、移動端(App 應用、小程序)、服務器(後端)等，而 Node.js 就是為此而誕生的，它使 JavaScript 代碼能夠運行在任何地方，只要能夠運行 node 甚至將 node 編譯成`本地代碼(native code)`來運行。

#### Install 安裝

首先我們需要先安裝 node 環境：<a href="https://nodejs.org/en/">node 官網</a>。

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/node_download.png)

通常建議安裝 LTS(Long-term support) 版較為穩定，安裝 node 會同時安裝 npm 套件，在命令行檢查版本（windows 需要新增環境變量）：

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/node_version.png)

#### Run 運行

使用 node 環境執行 JS 非常簡單，只需要敲上 `node 入口文件` 命令即可，我們沿用剛剛的 `index.js` 文件，使用命令啟動：

```bash
$ node index
# $ node index.js
# 後綴可以忽略，當然 node 也只認識 .js 文件
```

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/node_run.png)

#### npm(Node package manager) 包管理器

通常我們會使用 `npm(Node package manager)包管理器` 來建立並管理我們的 node 項目，先在目標資料夾使用 `npm init` 初始化根目錄：

```bash
$ npm init -y # -y 使用默認配置初始化
```

我們會發現根目錄下多出了一個 `package.json` 文件（可能還會有 `package-lock.json`），它是用於各個包的版本管理控制，紀錄了安裝的其他的`包(package)`，以及一些包的配置選項（如 babel、jslint、webpack、jest）

## Node 環境

到此我們已經可以透過 `npm` 來初始化項目，並使用 `node` 指令來運行（當然你也可以寫在 `npm script` 中並使用 `npm run xxx` 來運行）。接下來我們就來看看相較於原始的`瀏覽器(browser)`環境，Node 環境有什麼不同。

### 全局 Global 對象

首先最重要的就是`全局對象`的改變，所謂的全局對象就是整個 JS 代碼運行的最底層的對象，一切底層的變量和方法聲明都作為全局對象的一個屬性存在(當然自從 ES6 之後並不那麼單純)

我們可以透過在最底層使用 `this` 關鍵字，或是使用 `window` 來訪問瀏覽器的全局對象 window

- 瀏覽器的全局對象：`Window 對象`

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/window.png)

然而在 Node 環境之下並不是這麼一回事的。還記得我們前面提過 Node 採用 CommonJS 模塊化方案，因此每個 `.js` 文件將被視為一個 JS 模塊(也就是一個對象)，我們只能夠透過 `global` 關鍵字來訪問。

首先我們先使用 npm 創建一個新的項目擁有如下結構：

```
node-demo/
|- src/
   |- index.js
|- package.json
```

並修改 `index.js` 的內容

- index.js

```js
console.log(this)
console.log(global)
```

然後使用 `node src/index` 運行

```bash
$ node src/index
{} # this 指向當前模塊
Object [global] { # global 為全局對象
  global: [Circular],
  clearInterval: [Function: clearInterval],
  clearTimeout: [Function: clearTimeout],
  setInterval: [Function: setInterval],
  setTimeout: [Function: setTimeout] {
    [Symbol(nodejs.util.promisify.custom)]: [Function]
  },
  queueMicrotask: [Function: queueMicrotask],
  clearImmediate: [Function: clearImmediate],
  setImmediate: [Function: setImmediate] {
    [Symbol(nodejs.util.promisify.custom)]: [Function]
  }
}
```

### 全局屬性

接下來介紹一些 Node.js 環境下的全局屬性比一般瀏覽器多出來的額外屬性以及比較重要的屬性：

- `__dirname`：當前執行文件目錄
- `__filename`：當前執行文件
- `console`：控制台輸出方法
  - `log`、`info`、`error`、`warn`：輸出
  - `time`、`timeEnd`：計時
  - `trace`：棧追蹤
  - `assert`：斷言
- `process`：表示當前進程對
  - `argv`：執行腳本參數
  - `env`：當前環境變量
  - `stdout`、`stderr`、`stdin`：標準輸出、錯誤、輸入流
  - `pid`：進程號
  - `config`：配置選項
  - `platform`：運行平台
  - `onexit`：退出事件
  - `onbeforeExit`：退出前事件
  - `uncaughtException`：全局異常捕獲
  - `pwd()`：返回當前目錄
  - `kill(pid)`：終止進程
  - `memoryUsage()`：返回內存使用情形
  - `nextTick(callback)`：等待事件循環結束

#### 所有屬性

這邊就不演示所以功能的使用，主要列出 `global`、`process` 的所有屬性，以及 `console` 的所有類型

```js
console.log('----- global properties -----')
console.log(Object.getOwnPropertyNames(global))
// ----- global properties -----
// [
//   'Object',             'Function',       'Array',
//   'Number',             'parseFloat',     'parseInt',
//   'Infinity',           'NaN',            'undefined',
//   'Boolean',            'String',         'Symbol',
//   'Date',               'Promise',        'RegExp',
//   'Error',              'EvalError',      'RangeError',
//   'ReferenceError',     'SyntaxError',    'TypeError',
//   'URIError',           'globalThis',     'JSON',
//   'Math',               'console',        'Intl',
//   'ArrayBuffer',        'Uint8Array',     'Int8Array',
//   'Uint16Array',        'Int16Array',     'Uint32Array',
//   'Int32Array',         'Float32Array',   'Float64Array',
//   'Uint8ClampedArray',  'BigUint64Array', 'BigInt64Array',
//   'DataView',           'Map',            'BigInt',
//   'Set',                'WeakMap',        'WeakSet',
//   'Proxy',              'Reflect',        'decodeURI',
//   'decodeURIComponent', 'encodeURI',      'encodeURIComponent',
//   'escape',             'unescape',       'eval',
//   'isFinite',           'isNaN',          'SharedArrayBuffer',
//   'Atomics',            'WebAssembly',    'global',
//   'process',            'GLOBAL',         'root',
//   'Buffer',             'URL',            'URLSearchParams',
//   'TextEncoder',        'TextDecoder',    'clearInterval',
//   'clearTimeout',       'setInterval',    'setTimeout',
//   'queueMicrotask',     'clearImmediate', 'setImmediate'
// ]

console.log(`\n__dirname: ${__dirname}`)
console.log(`__filename: ${__filename}`)
// __dirname: /Users/superfree/Desktop/Blog/code/front_end/node-demo/src
// __filename: /Users/superfree/Desktop/Blog/code/front_end/node-demo/src/index.js

console.log('\n----- console types -----')
console.log(console)
// ----- console types -----
// {
//   log: [Function: bound consoleCall],
//   warn: [Function: bound consoleCall],
//   dir: [Function: bound consoleCall],
//   time: [Function: bound consoleCall],
//   timeEnd: [Function: bound consoleCall],
//   timeLog: [Function: bound consoleCall],
//   trace: [Function: bound consoleCall],
//   assert: [Function: bound consoleCall],
//   clear: [Function: bound consoleCall],
//   count: [Function: bound consoleCall],
//   countReset: [Function: bound consoleCall],
//   group: [Function: bound consoleCall],
//   groupEnd: [Function: bound consoleCall],
//   table: [Function: bound consoleCall],
//   debug: [Function: bound consoleCall],
//   info: [Function: bound consoleCall],
//   dirxml: [Function: bound consoleCall],
//   error: [Function: bound consoleCall],
//   groupCollapsed: [Function: bound consoleCall],
//   Console: [Function: Console],
//   profile: [Function: profile],
//   profileEnd: [Function: profileEnd],
//   timeStamp: [Function: timeStamp],
//   context: [Function: context],
//   [Symbol(kBindStreamsEager)]: [Function: bound ],
//   [Symbol(kBindStreamsLazy)]: [Function: bound ],
//   [Symbol(kBindProperties)]: [Function: bound ],
//   [Symbol(kWriteToConsole)]: [Function: bound ],
//   [Symbol(kGetInspectOptions)]: [Function: bound ],
//   [Symbol(kFormatForStdout)]: [Function: bound ],
//   [Symbol(kFormatForStderr)]: [Function: bound ]
// }

console.log('\n----- process properties -----')
console.log(Object.getOwnPropertyNames(process))
// ----- process properties -----
// [
//   'version',
//   'versions',
//   'arch',
//   'platform',
//   'release',
//   '_rawDebug',
//   'moduleLoadList',
//   'binding',
//   '_linkedBinding',
//   '_events',
//   '_eventsCount',
//   '_maxListeners',
//   'domain',
//   '_exiting',
//   'config',
//   'dlopen',
//   'uptime',
//   '_getActiveRequests',
//   '_getActiveHandles',
//   'reallyExit',
//   '_kill',
//   'hrtime',
//   'cpuUsage',
//   'resourceUsage',
//   'memoryUsage',
//   'kill',
//   'exit',
//   'openStdin',
//   'getuid',
//   'geteuid',
//   'getgid',
//   'getegid',
//   'getgroups',
//   'allowedNodeEnvironmentFlags',
//   'assert',
//   'features',
//   '_fatalException',
//   'setUncaughtExceptionCaptureCallback',
//   'hasUncaughtExceptionCaptureCallback',
//   'emitWarning',
//   'nextTick',
//   '_tickCallback',
//   '_debugProcess',
//   '_debugEnd',
//   '_startProfilerIdleNotifier',
//   '_stopProfilerIdleNotifier',
//   'stdout',
//   'stdin',
//   'stderr',
//   'abort',
//   'umask',
//   'chdir',
//   'cwd',
//   'initgroups',
//   'setgroups',
//   'setegid',
//   'seteuid',
//   'setgid',
//   'setuid',
//   'env',
//   'title',
//   'argv',
//   'execArgv',
//   'pid',
//   'ppid',
//   'execPath',
//   'debugPort',
//   'argv0',
//   '_preload_modules',
//   'mainModule'
// ]
```

### 模塊化

前面也提過了 Node 採用 CommonJS 的模塊化協議，我們使用 `module.exports` 向外部導出一個`對象(object)`，在其他模塊使用 `require` 導入外部模塊，使用 `npm install` 安裝的模塊也是一樣的用法

我們沿用前面的 npm 項目，並加入一個 `other.js`，項目結構如下

```
node-demo/
|- src/
   |- index.js
   |- other.js
|- package.json
```

並修改兩個文件的內容

- other.js

```js
module.exports = {
  user: {
    name: 'John'
  },
  greeting: function (name) {
    console.log(`Hi! I am ${name}`)
  }
}
```

- index.js

```js
const other = require('./other')
const { user, greeting } = other
greeting(user.name)
```

最後一樣輸入 `node src/index` 執行：

```
$ node src/index
Hi! I am John
```

#### 內置模塊

`require` 能夠引入外部模塊，除了我們自定義的模塊以及使用 `npm install` 安裝的第三方模塊之外，還有一些是 node 環境內置的模塊（不需要 `npm install` 可以直接 `require`）：

| Module | Description        |
| ------ | ------------------ |
| path   | 處理文件路徑的模塊 |
| util   | 更豐富的功能性 API |
| fs     | 文件操作相關 API   |
| events | 事件相關 API       |
| http   | Web 服務器相關 API |

下面我們就使用 http 模塊來製作一個極簡的 Web 服務器做結尾

## 使用 Node 建構一個 Web server

- 項目結構

```
node-demo/
|- src/
   |- index.js
   |- server.js
|- package.json
```

- server.js

```js
const http = require('http')

const serverStartPrefix = '[httpServer][Start]'
const serverHandlerPrefix = '[httpServer][Handler]'

module.exports = {
  config: {
    port: 8080,
    ip: 'localhost',
    server: null
  },
  start: function () {
    if (this.config.server != null) {
      console.log(`${serverStartPrefix}: restart server`)
    }
    console.time(serverStartPrefix)
    const { ip, port } = this.config
    // 建立服務器
    const server = http.createServer(this.requestHandler.bind(this))
    server.listen(port, function () {
      console.log(`${serverStartPrefix}: running at http://${ip}:${port}`)
      console.timeEnd(serverStartPrefix)
    })
    this.config.server = server
  },
  // 請求處理函數
  requestHandler: function (request, response) {
    console.time(serverHandlerPrefix)
    // 請求路徑
    const requestURL = request.url
    console.log(`request with path: ${requestURL}`)
    // 處理返回，使用 end 結束並發送
    response.writeHeader(200, { 'Content-Type': 'text/plain' })
    response.end('Hello World\n')

    console.timeEnd(serverHandlerPrefix)
  }
}
```

- index.js

```js
const server = require('./server')
server.start()
```

- 運行截圖

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/sample_console.png)

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/sample_page.png)

# 結語

本篇簡單介紹了整個 Node.js 的環境以及全局屬性，並且使用 http 模塊創建一個極簡版的 Web 服務器。後面將單獨介紹各個內置模塊的屬性和方法，以及用法示例，供各位參考。
