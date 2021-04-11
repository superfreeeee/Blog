# JS 基礎：事件流

@[TOC](文章目錄)

<!-- TOC -->

- [JS 基礎：事件流](#js-基礎事件流)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [EventTarget 事件目標對象](#eventtarget-事件目標對象)
    - [Methods 方法](#methods-方法)
      - [EventTarget.addEventListener](#eventtargetaddeventlistener)
      - [EventTarget.removeEventListener](#eventtargetremoveeventlistener)
      - [EventTarget.dispatchEvent](#eventtargetdispatchevent)
  - [Event 事件對象](#event-事件對象)
    - [Properties 屬性](#properties-屬性)
    - [Methods 方法](#methods-方法-1)
  - [Capture & Bubbling 捕獲和冒泡階段](#capture--bubbling-捕獲和冒泡階段)
    - [`preventDefault()` & `stopPropagation()`](#preventdefault--stoppropagation)
  - [實踐：事件委派](#實踐事件委派)
- [結語](#結語)

<!-- /TOC -->

## 簡介

前一篇我們介紹過 JS 引擎的事件觸發機制：點擊鏈結：<a href="https://blog.csdn.net/weixin_44691608/article/details/106485760">JS 基礎：Event Loop 事件循環機制</a>

本篇來介紹 JS 的"事件"觸發機制。作為整個 web 系統中與用戶交互最頻繁的客戶端，網頁必須提供一種機制來"響應"或是"回饋"用戶的操作，而這種響應觸發的時間是不確定的，又由於 JS 為單線程語言，因此瀏覽器就不得不提供內置的 WebAPI，來把事件監聽事件回調函數作為異步函數處理。本篇就來簡單介紹一下 DOM 的世界裡面，到底誰接下了這個事件觸發的任務，使用者又要如何來為裡的元素添加事件。

## 參考

<table>
  <tr>
    <td>EventTarget MDN</td>
    <td><a href="https://developer.mozilla.org/zh-TW/docs/Web/API/EventTarget">https://developer.mozilla.org/zh-TW/docs/Web/API/EventTarget</a></td>
  </tr>
  <tr>
    <td>事件 (Event) 的註冊、觸發與傳遞</td>
    <td><a href="https://medium.com/@realdennis/javascript-%E4%BA%8B%E4%BB%B6-event-da8104c5c98c">https://medium.com/@realdennis/javascript-%E4%BA%8B%E4%BB%B6-event-da8104c5c98c</a></td>
  </tr>
</table>

# 正文

## EventTarget 事件目標對象

首先在我們開始研究事件觸發的機制之前，我們需要先搞清楚到底真正接收事件並觸發監聽函數的對象是誰，也就是所謂的 `EventTarget` 事件目標對象。

 我們都知道 html 內所有的標籤都是一個個的 `Node` 對象，而 `Node` 對象的原型就是所謂的 `EventTarget`（本篇不詳細介紹 JS 的原型鏈繼承機制），如下圖：

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/EventTarget_inherit.png)

也就是說所有的 html 元素都是可以觸發事件的目標對象，另外還有 XMLHttpRequest、Document、Window、AudioNode 等也都繼承 EventTarget 對象(或是說接口)，在此就不一一贅述。

### Methods 方法

事件目標對象總共提供了三個方法來控制一個事件目標的監聽函數，分別是`創建(addEventListener)`、`移除(removeEventListener)`、`觸發(dispatchEvent)`

#### EventTarget.addEventListener

addEventListener 能夠向事件目標添加監聽函數，語法如下：

- 語法

```js
target.addEventListener(type, listener [, options])
target.addEventListener(type, listener [, useCapture])
```

- 參數說明：

```ts
interface Param {
  type: string // 事件名稱（類型）
  listener: EventListener | Function // 事件處理函數，接收一個 Event 對象作參數
  options?: {
    capture: boolean // 是否作為捕獲事件(true)，默認為冒泡事件(false)
    passive: boolean // 事件是否會繼續傳遞
  }
  useCapture?: boolean // 與 options.capture 一致
}
```

這邊的 capture 跟 useCapture 會牽扯到`捕獲(capture)`和`冒泡(bubble)`事件，後面段落將會說明

#### EventTarget.removeEventListener

- 語法

```js
target.removeEventListener(type, listener[, options])
target.removeEventListener(type, listener[, useCapture])
```

- 參數說明：

這邊的參數與 add 基本上一致，唯一不同的是這裡的 options 和 useCapture 更像是特徵說明，removeEventListener 將會移除所有符合特徵說明的監聽函數

#### EventTarget.dispatchEvent

- 語法

```js
target.dispatchEvent(event) => canceled
```

- 參數說明：

```ts
event: Event // 為 DOM 模型中繼承 Event 接口的事件對象，返回值表示事件是否正常結束
```

## Event 事件對象

有了事件觸發對象(EventTarget)，接下來我們來了解一下事件對象(Event)本身。

`事件(Event)`通常是由使用者的操作觸發的行為，而程序員也能透過 EventTarget.dispatchEvent 來主動觸發事件。常見的事件有點擊、鍵盤、滾動條事件等，詳細的事件列表可以查詢<a href="https://developer.mozilla.org/zh-TW/docs/Web/API/Event">相關 API</a>，這邊僅僅列出幾個簡單子接口：

| Event         | 事件名   |
| ------------- | -------- |
| MouseEvent    | 滑鼠事件 |
| KeyboardEvent | 鍵盤事件 |
| MessageEvent  | 信息事件 |
| WheelEvent    | 滾動事件 |
| FocusEvent    | 關注事件 |
| InputEvent    | 輸入事件 |

不同元素和不同事件類型觸發的事件對象不同，須依據不同事件名稱正確的處理事件對象

### Properties 屬性

這邊介紹事件對象的主要屬性

| Property                         | Description                                  |
| -------------------------------- | -------------------------------------------- |
| Event.bubbles(readonly)          | boolean 類型，表示事件是否向上傳遞（冒泡）   |
| Event.currentTarget(readonly)    | 指向當前監聽器函數所屬的 DOM 對象            |
| Event.target(readonly)           | 永遠指向最初觸發事件的 DOM 對象              |
| Event.defaultPrevented(readonly) | 表示預設行為是否被 `preventDefault` 方法取消 |
| Event.eventPhase(readonly)       | 表示當前事件的處理階段                       |
| Event.type(readonly)             | 事件類型                                     |

### Methods 方法

事件的主要方法，由於事件是經觸發後產生的對象，多數操作都是利用事件的屬性，相對的事件的方法就少很多，主要在控制事件的傳遞行為(capture & bubble)

| Mehtods                          | Description                                          |
| -------------------------------- | ---------------------------------------------------- |
| Event.preventDefault()           | 取消默認行為(頁面跳轉、表單提交)，但是事件將繼續傳遞 |
| Event.stopImmediatePropagation() | 取消傳遞同時取消觸發同一事件類型的其他監聽函數       |
| Event.stopPropagation()          | 取消事件的傳遞(capture & bubbling)                   |

## Capture & Bubbling 捕獲和冒泡階段

接下來就進入我們的重頭戲了，事件的捕獲(capture)和冒泡(bubbling)階段

我們先看一個總覽圖，表示事件觸發的冒泡和補貨

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/capture&bubbling.png)

首先先給出一段 html 的模板代碼

```html
<div id="app">
  <ul id="list">
    <li id="item1">row 1</li>
    <li id="item2">row 2</li>
    <li id="item3">row 3</li>
    <li id="item4">row 4</li>
  </ul>
</div>
```

再來向各個元素註冊監聽事件：

**注意！**

這裡必需使用 `getElementById` / `getElementByTagName`，而不能使用 `querySelector`（因為 querySelector 返回的是元素的靜態副本），詳細差異請自行查詢相關解釋本篇不贅述。

同時因為註冊函數默認註冊的函數為冒泡，因此捕獲事件需要將 `useCapture` 設置為 `true`

```js
const app = document.getElementById('app')
const list = document.getElementById('list')
const item1 = document.getElementById('item1')
const item2 = document.getElementById('item2')
const item3 = document.getElementById('item3')
const item4 = document.getElementById('item4')
// 註冊函數定義
function registerClick(target, name, useCapture) {
  if (useCapture === undefined) {
    useCapture = false
  }
  target.addEventListener(
    'click',
    () => {
      console.log(
        `invoke ${useCapture ? 'capture' : 'bubble'} event at ${name}`
      )
    },
    useCapture
  )
}
// 註冊監聽函數
registerClick(app, 'app')
registerClick(app, 'app', true)
registerClick(list, 'list')
registerClick(list, 'list', true)
registerClick(item1, 'item1')
registerClick(item1, 'item1', true)
registerClick(item2, 'item2')
registerClick(item2, 'item2', true)
registerClick(item3, 'item3')
registerClick(item3, 'item3', true)
registerClick(item4, 'item4')
registerClick(item4, 'item4', true)
```

這時候如果我們點擊 ul 中的第二個 li 元素的輸出結果應該如下：

```
invoke capture event at app
invoke capture event at list
invoke capture event at item2
invoke bubble event at item2
invoke bubble event at list
invoke bubble event at app
```

這時候我們就能夠看出事件的觸發會從根元素一路向下傳遞到目標元素(因為通常越接近底部的元素處於越上層的位置)，一路捕獲到目標對象後在反向往根部冒泡，也就是說執行順序如下：

```
app(capture)
-> list(capture)
-> item2(capture)
-> item2(bubble)
-> list(bubble)
-> app(bubble)
```

- 備註：我在 chrome 環境下測試的時候會長這樣，目前還不清楚為啥目標對象的捕獲和冒泡順序異常，希望有小夥伴能幫我解答 qq

```js
invoke capture event at app
invoke capture event at list
invoke bubble event at item1
invoke capture event at item1
invoke bubble event at list
invoke bubble event at app
```

### `preventDefault()` & `stopPropagation()`

現在我們稍微修改一下上面的監聽函數註冊，來檢驗`preventDefault` 和 `stopPropagation` 的作用：

```js
// 重新定義註冊函數
function registerClick(target, name, useCapture, prev, stop) {
  if (useCapture === undefined) {
    useCapture = false
  }
  target.addEventListener(
    'click',
    (e) => {
      if (prev) {
        e.preventDefault()
      }
      if (stop) {
        e.stopPropagation()
      }
      console.log(
        `invoke ${useCapture ? 'capture' : 'bubble'} event at ${name}`
      )
    },
    useCapture
  )
}
// 註冊監聽函數
registerClick(app, 'app')
registerClick(app, 'app', true)
registerClick(list, 'list')
registerClick(list, 'list', true, true)
registerClick(item1, 'item1')
registerClick(item1, 'item1', true)
registerClick(item2, 'item2')
registerClick(item2, 'item2', true)
registerClick(item3, 'item3')
registerClick(item3, 'item3', true)
registerClick(item4, 'item4')
registerClick(item4, 'item4', true)
```

輸出：

```
invoke capture event at app
invoke capture event at list
invoke capture event at item2
invoke bubble event at item2
invoke bubble event at list
invoke bubble event at app
```

- 說明：這裡在 list 的冒泡階段調用 `preventDefault()` 函數，輸出一樣，表示事件依舊正常的捕獲和冒泡，也就是說事件還是正常地走完整個傳遞流程

---

接下來我們改成在 list 的捕獲階段調用 `stopPropagation()` 函數：

```js
registerClick(app, 'app')
registerClick(app, 'app', true)
registerClick(list, 'list')
registerClick(list, 'list', true, false, true)
registerClick(item1, 'item1')
registerClick(item1, 'item1', true)
registerClick(item2, 'item2')
registerClick(item2, 'item2', true)
registerClick(item3, 'item3')
registerClick(item3, 'item3', true)
registerClick(item4, 'item4')
registerClick(item4, 'item4', true)
```

輸出：

```
invoke capture event at app
invoke capture event at list
```

- 說明：這時候我們就發現事件傳遞被截止了，也就是當事件被 list 捕獲之後，就終止了傳遞，傳遞路徑變成如下：

```
app(capture)
-> list(capture)
-> X
```

## 實踐：事件委派

到此我們都瞭解了事件的傳遞機制，以及捕獲和冒泡的順序。

現在我們假設一個場景：我們有一個列表，裡面的元素是不固定，且有可能非常巨大(超過一千個列表項)，html 模板如下：

```html
<ul id="list">
  <li id="item1">row 1</li>
  <li id="item2">row 2</li>
  <li id="item3">row 3</li>
  <li id="item4">row 4</li>
  <!-- ... -->
  <li id="item1000">row 1000</li>
</ul>
```

我們當然可以透過一個 for 循環為每個子元素註冊監聽函數，並在添加元素或刪除元素的時候移除監聽函數：

```js
const ul = document.getElementById('list')
for (let li of ul.children) {
  li.addEventListener('click', (e) => {
    console.log(e.target.innerHTML)
  })
}
```

雖然說這樣是可以，但這樣會添加大量的監聽函數，然而他的處理方法都是一模一樣的，這時候我們可以將觸發事件"委派"(dispatch)給其父元素，我們可以像下面這樣寫：

```js
const ul = document.getElementById('list')
ul.addEventListener('click', (e) => {
  console.log(e.target.innerHTML)
})
```

- 說明：由於 Event.target 永遠指向被觸發的最底層元素，所以我們可以藉由事件傳遞機制在冒泡或捕獲的過程中提前處理事件，並透過 Event.target 得知事件觸發的真正目標（也可以透過 path 屬性來檢查事件傳遞路徑）。

如此就能夠做到只註冊了一個監聽函數就能操作整個列表的點擊事件，大大減少了運行時開銷。

# 結語

本篇先介紹事件觸發相關的`事件目標對象(EventTarget)`以及`事件對象(Event)`，並描述了事件`捕獲(capture)`和`冒泡(bubbling)`的過程，並可透過 `Event.stopPropagation()` 方法來控制事件的傳遞。熟悉 JS 的事件處理機制能夠使開發者能更有效的運用監聽函數並避免不必要的開銷，本篇就到此為止。
