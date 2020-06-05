# JS 基礎：Event Loop 事件循環機制

@[TOC](文章目錄)

<!-- TOC -->

- [JS 基礎：Event Loop 事件循環機制](#js-基礎event-loop-事件循環機制)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Structure 結構](#structure-結構)
    - [Stack 棧](#stack-棧)
    - [Heap 堆](#heap-堆)
    - [WebAPIs 接口](#webapis-接口)
    - [Callback Queue 回調函數隊列](#callback-queue-回調函數隊列)
  - [Main Procedure 主要流程](#main-procedure-主要流程)
  - [Problem 問題](#problem-問題)
    - [Blocking 阻塞](#blocking-阻塞)
      - [Sample](#sample)
      - [Solution 解決辦法](#solution-解決辦法)
    - [Render Blocking 瀏覽器渲染阻塞](#render-blocking-瀏覽器渲染阻塞)
      - [Sample](#sample-1)
      - [Solution 解決方法](#solution-解決方法)
    - [Callback Queue Blocking 回調函數隊列阻塞](#callback-queue-blocking-回調函數隊列阻塞)
- [結語](#結語)

<!-- /TOC -->

## 簡介

前言：一個國外開發者非常經典的關於事件循環機制的分享，講者為 Philip Roberts，相關連結

眾所皆知 JavaScript 是一個單線程的語言，也就是所有代碼都必須順序執行不存在多線程併發(如 Java 的時間切分機制)，然而程序中免不了必定會有一些異步任務，則 JS 引擎就必須區分對待，大多數引擎所採納的標準大同小異也就是 **Event Loop 事件循環機制**，而本地 node 環境和 V8 引擎的表現又會有些許的差異（檢查順序或任務切分等，因此程序邏輯不可依賴於這些順序）可以忽略不計。

## 參考

<table>
  <tr>
    <td>What the heck is the event loop anyway?</td>
    <td><a href="https://www.youtube.com/watch?v=8aGhZQkoFbQ&index=42&list=WL&t=32s">https://www.youtube.com/watch?v=8aGhZQkoFbQ&index=42&list=WL&t=32s</a></td>
  </tr>
  <tr>
    <td>Javascript 的 Event Loop</td>
    <td><a href="https://medium.com/hobo-engineer/ricky%E7%AD%86%E8%A8%98-javascript-%E7%9A%84-event-loop-c17a0a49d6e4">https://medium.com/hobo-engineer/ricky%E7%AD%86%E8%A8%98-javascript-%E7%9A%84-event-loop-c17a0a49d6e4</a></td>
  </tr>
</table>

# 正文

首先談到異步操作就必須先瞭解有哪些操作被稱為"異步"：

1. setTimeout / setInterval / setImmediate
2. ajax(XMLHttpRequest)
3. Promise
4. mutation observer(setter/getter)

異步往往是一些需要等待的任務，或是需要花費大量時間的任務有時候也會被放到異步任務當中。

這邊就不贅述異步任務的意義了，我們主要要解析的是瀏覽器到底怎麼處理"異步"任務，使其看起來像是同時進行多項任務(multi-task)。

由於 Event Loop 的細節原理根據實際的 JS 引擎會有所差異，所以這裡只是介紹 JS 引擎的抽象概念，各種引擎的表現細節需要個別查詢使用手冊。

## Structure 結構

我們先來看一下 JS 引擎執行 JavaScript 代碼的抽象結構：

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/eventloop_main.png)

### Stack 棧

現代計算機的程序執行邏輯大多都使用棧結構，函數調用則作為一個一個的幀(frame)被壓到棧裡面，而作為單線程語言的 JS 對於同一個進程(progress)同時只會存在唯一的棧結構。詳細的棧說明可從參考一的影片中得到更形象化的解釋。

### Heap 堆

除了棧保存了當前的函數調用鏈之外，有其他被創建的對象、函數閉包等會在棧幀彈出之後依舊保持他的生命週期，這類型的數據就會被放置在堆裡面

### WebAPIs 接口

由於對使用者來說，組成程序的方法集合無非就是自定義的方法和瀏覽器提供的內置方法，也就是所謂的 API(Application Programming Interface)。而異步方法作為一個瀏覽器提供的內置方法由使用者來調用，以此來達成異步請求。

歸屬於異步方法的 API 在被調用之後，會將處理好的邏輯以及回調函數一併放到一個回調函數隊列之中(Callback queue)，而當棧中的方法被清空之後，JS 引擎就會查詢回調函數隊列中是否存在待執行的方法，開始真正的異步處理程序。

### Callback Queue 回調函數隊列

這邊保存了一個異步操作結束後的回調函數隊列，實際上在某些引擎會將 Callback Queue 分成兩個隊列(Task Queue & Microtask Queue)，並且擁有不同的調用順序等，具體細節一樣需要查詢具體引擎的說明，這邊我們只需要知道回調函數通常是在棧清空之後才會執行的任務隊列。

## Main Procedure 主要流程

其實我們已經可以從上面的圖中清楚的看到 JS 引擎大致的結構，這邊開始描述 JS 異步函數的執行流程：

一般的 JS 同步方法執行時都是直接往 stack 壓棧執行函數，然後執行完畢之後再出棧。然而當遇到異步函數時候就不同了，這時候調用了 WebAPI 的異步方法，將會先執行方法的函數主體，然後將處理完之後需要調用的回調邏輯在恰當的時刻(setTimeout 時間到、後端請求返回)，將回調函數放入回調函數隊列之中。而 Callback Queue 中的方法則會等待當前方法棧(stack)中的同步方法執行完畢之後，將回調函數隊列中的回調函數一次一個押入棧中真正執行回調，並且如果同時觸發大量的回調函數事件並不會阻塞同步方法的執行，在異步方法的回調函數之間將會檢查是否有同步方法等待執行，瀏覽器選染也可以在異步方法的調用空擋繼續渲染而不會造成網頁停頓。

以上就是 JS 的異步函數的調用機制。

## Problem 問題

JS 引擎提供這樣的異步操作接口，到底能為程序設計帶來怎麼樣的好處呢，又能夠解決哪些同步函數無能為力的死局？

### Blocking 阻塞

第一個可能的問題是阻塞

場景說明：當所有程序都是同步執行的時候，如果出現一些響應速度很慢或需要極長時間來執行的任務，會造成當前程序阻塞(blocking)，看起來就像是當機或卡頓一樣，這時用戶體驗會變得極差。

同時也不利於程序流程控制，因為程序將停頓在不可預期的位置而使瀏覽器根本無暇響應用戶操作。

#### Sample

```js
function pause(ms) {
  const target = Date.now() + ms
  while (true) {
    const cur = Date.now()
    if (cur >= target) {
      return
    }
  }
}

function task1() {
  // do something
  pause(1000)
}

function task2() {
  // do something
  pause(2000)
}

function task1() {
  // do something
  pause(3000)
}

const res1 = task1()
const res2 = task2()
const res3 = task3()

console.log(res1)
console.log(res2)
console.log(res3)
```

- 說明：task1,2,3 用於模擬耗時長的同步方法調用，由於 task1,2,3 總共將會阻塞 6 秒的程序執行時間，因此在這段期間內瀏覽器完全無法進行任何其他的操作，便將這個網頁"阻塞"起來了

通常發送 http 請求、加載圖片、加載文件等都是耗時操作，並且不確定返回時機，對於這種方法如果使用同步邏輯非常容易造成程序阻塞。

#### Solution 解決辦法

上述阻塞代碼則可以透過 setTimeout 異步方法來解決，如下

```js
setTimeout(() => {
  const res1 = task1()
  console.log(res1)
}, 0)

setTimeout(() => {
  const res2 = task2()
  console.log(res2)
}, 0)

setTimeout(() => {
  const res3 = task3()
  console.log(res3)
}, 0)
```

- 說明：當把 task1,2,3 換成異步方法調用的時候，程序就會優先調用棧上的同步方法，等站上清空（也就是瀏覽器"有空"的時候）才會來執行異步調用的方法。

同時，標籤上的監聽函數也是一種異步調用邏輯。由於 JS 並不能向其他語言(如 C、Java)等擁有中斷處理機制，用戶操作所觸發的事件回調函數只能透過回調函數隊列，讓觸發事件先"排隊"等候一下，等 JS 引擎"有空"(Stack 清空)的時候才開始調用回調函數。

這時候我們會發現，`setTimeout` 方法的第二個參數 `delay` 不一定是真正函數的延遲時間，而只能保證中間間隔至少多少毫秒。

### Render Blocking 瀏覽器渲染阻塞

由於渲染也是瀏覽器引擎所需要執行的動作之一，對於一個 60 fps 的畫面渲染，大約每 16.67 毫秒就需要選染一次，也就是說當一段連續的同步方法調用超過這個間隔的話，就會造成渲染延遲（可以再參考一的演講中看到示例）

#### Sample

```js
;[1, 2, 3, 4, 5].map((val) => {
  pause(2000)
  console.log(val)
})
```

- 說明：上述代碼調用一個 map 方法將會遍歷一遍數組，並執行相應操作，然而這個同步方法將阻塞引擎約十秒的時間，這就有可能造成也面渲染延遲

#### Solution 解決方法

將其轉換成異步代碼，同時設置 `setTimeout 0` 表示立即執行的異步函數，將被立即添加到回調函數隊列

```js
;[1, 2, 3, 4, 5].map((val) => {
  setTimeout(() => {
    pause(2000)
    console.log(val)
  }, 0)
})
```

- 說明：由於 setTimeout 本身並不是一個耗時的操作，僅僅是將回調函數"委託"到隊列中，也就是說這段代碼就不再阻塞程序控制流，而將實際操作延遲到 Callback Queue 內等待調用

### Callback Queue Blocking 回調函數隊列阻塞

這時雖然解決的同步方法阻塞的問題，然而還是有可能出現阻塞，這時不是出現在棧裡面，而是出現在回調函數隊列裡面，場景如下

現在為 HTML 元素綁定一個事件監聽函數

```js
const target = document.getElementById(selector)
target.addEventListener('click', function () {
  console.log('invoke target click')
})
```

這時如果出現左鍵連打(短時間大量觸發事件)，就累積大量回調函數在 Callback Queue 當中，這時其他後勁來的回調函數將會大大推遲，造成響應無效的假象。這種問題可以透過段時間內阻斷按鈕監聽來避免。

# 結語

本篇介紹有關事件循環機制的基本原理概念，實際上 JS 引擎真正的調用順序應該查詢具體引擎說明。同時強烈建議讀者把參考一的演講細細的品嘗一遍，同時搭配他所提供的事件循環機制的模擬器實際操作一遍，會對 JS 引擎的運行有更深刻的了解。
