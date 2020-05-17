# JS 基礎：DOM & BOM

@[TOC](文章目錄)

<!-- TOC -->

- [JS 基礎：DOM & BOM](#js-基礎dom--bom)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Overview 總覽](#overview-總覽)
  - [BOM(Browser Object Model)](#bombrowser-object-model)
    - [Window 對象](#window-對象)
    - [Window.document (Document 對象)](#windowdocument-document-對象)
    - [Window.history (History 對象)](#windowhistory-history-對象)
    - [Window.location (Location 對象)](#windowlocation-location-對象)
    - [Window.navigator (Navigator 對象)](#windownavigator-navigator-對象)
    - [Window.screen (Screen 對象)](#windowscreen-screen-對象)
    - [Window.localStorage (Storage 對象)](#windowlocalstorage-storage-對象)
  - [DOM(Document Object Model)](#domdocument-object-model)
    - [創建方法](#創建方法)
    - [查找方法](#查找方法)
- [結語](#結語)

<!-- /TOC -->

## 簡介

我們都知道 HTML、css、JavaScript 組合能生成千變萬化的頁面，今天我們來介紹瀏覽器環境下的 JS 對象。在瀏覽器環境下除了原生的標籤節(HTMLElement)點和樣式節點，瀏覽器還提供了兩種對象模型用來提供外部與原調用的接口，例如我們想調用瀏覽右鍵操作的列表、控制網頁內容的複製權限、對窗口歷史紀錄的回退等，這些都是 JS 原生能力以外由瀏覽器釋放出的接口，今天我們就介紹`瀏覽器對象模型(BOM)`和在此之下的`文件對象模型(DOM)`，以及其內部的一些重要對象簡述。

## 參考

<table>
  <tr>
    <td>Window-MDN</td>
    <td><a href="https://developer.mozilla.org/zh-TW/docs/Web/API/Window">https://developer.mozilla.org/zh-TW/docs/Web/API/Window</a></td>
  </tr>
  <tr>
    <td>Documnet-MDN</td>
    <td><a href="https://developer.mozilla.org/zh-TW/docs/Web/API/Document">https://developer.mozilla.org/zh-TW/docs/Web/API/Document</a></td>
  </tr>
  <tr>
    <td>JavaScript入門系列：BOM和DOM筆記</td>
    <td><a href="https://www.happycoding.today/posts/43">https://www.happycoding.today/posts/43</a></td>
  </tr>
  <tr>
    <td>最全的DOM和BOM的解释分析</td>
    <td><a href="https://juejin.im/post/5d7677b06fb9a06afd662d20">https://juejin.im/post/5d7677b06fb9a06afd662d20</a></td>
  </tr>
</table>

# 正文

## Overview 總覽

瀏覽器提供的對象模型主要分成兩大體系：

1. BOM(Browser Object Model)
2. DOM(Document Object Model)

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/bom&dom_model.png)

## BOM(Browser Object Model)

BOM 模型最主要的對象就是 `Window` 對象，也就是我們在寫 js 時使用到的全局對象。我們可以把 Window 對象想像成他代表一個`視窗`的上下文環境，視窗等級下的重要屬性和操作都是透過這個對象向使用者釋出。

### Window 對象

全局對象 `Window` 下存在一些重要屬性和方法，以下間單列出一些常用的方法，詳細屬性可以到參考一的 MDN 鏈結全面介紹 Window 對象的屬性和方法：

| Property                      | Description         |
| :----------------------------- | :------------------- |
| Window.document(readonly)     | 指向 Document 對象  |
| Window.history(readonly)      | 指向 History 對象   |
| Window.location               | 指向 Location 對象  |
| Window.navigator(readonly)    | 指向 Navigator 對象 |
| Window.screen(readonly)       | 指向 Screen 對象    |
| Window.localStorage(readonly) | 指向 Storage 對象   |

| Methods                                         | Description                                                          |
| :----------------------------------------------- | :-------------------------------------------------------------------- |
| Window.addEventListener(type, listener, option) | 添加事件監聽器                                                       |
| Window.setTimeout(callback, delay\[, args\])    | 設置定時器，延遲調用，調用邏輯詳見事件循環(Event Loop)               |
| Window.clearTimeout(timeoutID)                  | 清除定時器                                                           |
| Window.setInterval(callback, delay\[, args\])   | 設置定時循環，每隔一段時間重複調用，調用邏輯詳見事件循環(Event Loop) |
| Window.clearInterval(timeoutID)                 | 清除定時循環                                                         |

| Methods                               | Description                                               |
| :------------------------------------- | :--------------------------------------------------------- |
| Window.alert()                        | 彈出警告框                                                |
| Window.confirm()                      | 彈出確認框                                                |
| Window.focus()                        | 主動觸發視窗 focus                                        |
| Window.blur()                         | 主動觸發視窗 blur                                         |
| Window.open(url, name, windowFeature) | 開新分頁                                                  |
| Window.close()                        | 關閉分頁                                                  |
| Window.stop()                         | 停止頁面加載                                              |
| Window.scrollBy(x, y)                 | 滑動窗口（相對偏移量）                                    |
| Window.scrollTo([x, y] \| option)     | 滑動窗口（指定位置），behavior 可選值：`smooth | instant` |

### Window.document (Document 對象)

詳見下方 DOM

### Window.history (History 對象)

History 對象保存當前頁面的`瀏覽紀錄`和`控制頁面的跳轉或回退`，以下簡單介紹重要屬性和方法：

| Property                  | Description                                                               |
| :------------------------- | :------------------------------------------------------------------------- |
| History.length(readonly)  | 瀏覽紀錄深度，新開視窗的預設值為 1                                        |
| History.scrollRestoration | 允許 web 應用顯示的指定默認的回滾行為（上一頁），可選值為 `auto | manual` |
| History.state(readonly)   | 表示歷史紀錄棧頂（當前頁）的狀態                                          |

| Methods                                    | Description                                      |
| :------------------------------------------ | :------------------------------------------------ |
| History.back()                             | 回退一頁，相當於 History.go(-1)                  |
| History.forward()                          | 回退一頁，相當於 History.go(1)，以位於棧頂則無效 |
| History.go(step)                           | 於瀏覽紀錄列表移動，step 表示前後移動偏移量      |
| History.pushState(stateObj, title, url)    | 向 History 紀錄壓棧                              |
| History.replaceState(stateObj, title, url) | 修改 History 的某條紀錄                          |

### Window.location (Location 對象)

Location 對象紀錄當前頁面的`位置相關信息`，包括**url 相關信息**、**附帶參數**等，以下列出主要屬性和方法：

| Property          | Description                                 | Sample                                                         |
| :----------------- | :------------------------------------------- | :-------------------------------------------------------------- |
| Location.hash     | 路徑標示符，即 `#` 後的內容，可用來模擬路徑 | #/user/123                                                     |
| Location.host     | hostname + port                             | www.google.com                                                 |
| Location.hostname | 主機名                                      | www.google.com                                                 |
| Location.href     | 完整 url                                    | https://www.google.com.tw/?gfe_rd=cr&ei=u2VgWb2LG8748Aec_57ACw |
| Location.path     | url 中 host 後面的路由部分                  | /、/user                                                       |
| Location.port     | 端口號                                      | 80                                                             |
| Location.protocol | 通信協議                                    | http:                                                          |
| Location.search   | 搜索參數，即 `?` 後的內容                   | ?user=xxx&password=xxx                                         |

| Methods                      | Description                                                                        |
| :---------------------------- | :---------------------------------------------------------------------------------- |
| Location.assign(url)         | 加載給定 url 的資源，會創建一個新的 History 紀錄                                   |
| Location.reload(forceReload) | 重新加載資源，簡單來說就是刷新，參數決定是否強制刷新（false 時會先從緩存尋找資源） |
| Location.replace(url)        | 替換當前 url，修改當前 History 紀錄而不會新增瀏覽紀錄                              |
| Location.toString()          | 返回 Location 對象的 href                                                          |

### Window.navigator (Navigator 對象)

Navigator 對象表示`用戶的狀態和標示`，看了看屬性有點像`瀏覽器相關狀態`，包括**瀏覽器名稱**、**地理位置信息**、**連網信息**、**瀏覽器語言設定**，有點抽象，以下列出重要屬性可能比較有感：

| Property                          | Description                                                            |
| :--------------------------------- | :---------------------------------------------------------------------- |
| Navigator.appName(readonly)       | 瀏覽器官方名稱                                                         |
| Navigator.appVersion(readonly)    | 瀏覽器版本號                                                           |
| Navigator.battery(readonly)       | 可能要用 getBattery 訪問，返回一個 `BatteryManager`，保存電池相關信息  |
| Navigator.connection(readonly)    | 保存一個 `NetworkInformation` 對象，保存當前網路連線相關信息           |
| Navigator.cookieEnabled(readonly) | 保存當前 cookie 是否啟用                                               |
| Navigator.geolocation(readonly)   | 保存一個 `Geolocation 對象`，可訪問設備地理信息位置                    |
| Navigator.language(readonly)      | 保存當前瀏覽器首要使用語言                                             |
| Navigator.languages(readonly)     | 保存當前瀏覽器已知語言列表                                             |
| Navigator.onLine(readonly)        | 保存當前瀏覽器是否連網                                                 |
| Navigator.serviceWorker           | 保存一個 `ServiceWorkerContainer`，作為 ServiceWorker 池和控制管理模塊 |

### Window.screen (Screen 對象)

Screen 對象表示螢幕`窗口的相關信息`，窗口的**高度**、**寬度**、**像素**等，以下列出重要屬性和方法：

| Property           | Description                                               |
| :------------------ | :--------------------------------------------------------- |
| Screen.height      | 當前螢幕高度                                              |
| Screen.width       | 當前螢幕寬度                                              |
| Screen.availHeight | 瀏覽器可用螢幕高度                                        |
| Screen.availWidth  | 瀏覽器可用螢幕寬度                                        |
| Screen.orientation | 保存一個 `ScreenOrientation` 對象，紀錄當前螢幕的旋轉狀態 |
| Screen.colorDepth  | 表示當前瀏覽器色彩深度（？）                              |

| Methods                    | Description  |
| :-------------------------- | :------------ |
| Screen.lockOrientation()   | 鎖定屏幕旋轉 |
| Screen.unlockOrientation() | 解鎖屏幕旋轉 |

### Window.localStorage (Storage 對象)

Storage 可在本地以鍵值對的形式保存少量數據，為瀏覽器的本地緩存，不會隨著網頁的關閉清除：

| Property       | Description      |
| :-------------- | :---------------- |
| Storage.length | 保存鍵值對的數量 |

| Methods                     | Description                       |
| :--------------------------- | :--------------------------------- |
| Storage.key(val)            | 返回第 val 個 key 的值            |
| Storage.getItem(key)        | 查找 key 對應的值                 |
| Storage.setItem(key, value) | 設置鍵值對（新增 or 修改）        |
| Storage.removeItem(key)     | 移除鍵值對                        |
| Storage.claer()             | 清空 Storage 對象，移除所有鍵值對 |

## DOM(Document Object Model)

BOM 模型裡面提供了大多數與瀏覽器相關的接口，有**瀏覽紀錄相關的 History**、**位置信息相關的 Location**、**螢幕相關的 Screen**、**狀態標示和位置相關的 Navigator**，最後還有一個最重要的：**文檔內容相關的 Document**，也就是跟我們的網頁內容直接相關的對象，下列介紹 Document 的重要屬性和方法：

| Property                           | Description                                                           |
| :---------------------------------- | :--------------------------------------------------------------------- |
| Document.characterSet(readonly)    | 文檔的編碼類型                                                        |
| Document.doctype(readonly)         | 紀錄文檔類型(HTML 不同版本擁有不同 doctype、可區分 XML)               |
| Document.styleSheets(readonly)     | 返回文檔的樣式表                                                      |
| Document.visibilityState(readonly) | 返回文檔隱藏狀態，可能值為：`visible | hidden | prerender | unloaded` |
| Document.title                     | 返回 HTML 文檔的標題（分頁名）                                        |
| Document.head                      | 返回 HTML 文檔的 `<head>` 標籤                                        |
| Document.body                      | 返回 HTML 文檔的 `<body>` 標籤                                        |
| Document.cookie                    | 以 `key=value;key=value; ...` 的形式返回當前 cookie 的字符串          |
| Document.anchors                   | 返回文檔的錨點標籤（**廢棄**）                                        |
| Document.applets                   | 返回文檔的 applet 列表（**廢棄**）                                    |
| Document.forms(readonly)           | 返回當前文檔內的所有 `<form>` 標籤                                    |
| Document.embeds(readonly)          | 返回當前文檔內的所有 `<embed>` 標籤                                   |
| Document.images(readonly)          | 返回當前文檔內的所有 `<img>` 標籤                                     |
| Document.links(readonly)           | 返回當前文檔內的所有 `<a>` 標籤                                       |
| Document.scripts(readonly)         | 返回當前文檔內的所有 `<script>` 標籤                                  |
| Document.readyState(readonly)      | 表示文檔的加載狀態，可能值有：`loading | interactive | complete`      |

- 說明：嚴格來說我並沒有在 chrome 環境下找到 areas 和 layers 屬性，並不確定是 IE 專有還是已經被廢棄，所以開頭的圖中所示的 Layers[] 和 Areas[] 僅供參考

### 創建方法

Document 的屬性大多是一些內容，然而 Document 所有擁有的方法使用性更高，大多數 JS 框架如 Vue, React 在實現虛擬 DOM 的時候都借助到了 Document 對象主動創建和解析節點的能力

| Methods                           | Description                                                                                                  |
| :--------------------------------- | :------------------------------------------------------------------------------------------------------------ |
| Document.createAttribute(attr)    | 創建一個 `Attr` 屬性對象，用來添加在 `Element` 對象上                                                        |
| Document.createDocumentFragment() | 創建一個 `DocumentFragment` 文檔片段對象，相當於一個抽象的 Document 對象，插入真正的文檔後才會觸發頁面的重繪 |
| Document.createElement(tagname)   | 創建一個 `HTMLElement` 元素對象，可以是任意標籤對象(非標準標籤被定義成 `HTMLUnknownElement` 對象)            |
| Document.createComment(data)      | 創建一個 `Comment` 註解對象，相當於一個注釋標籤                                                              |
| Document.createTextNode(data)     | 創建一個 `Text` 文本對象，為一個文字節點                                                                     |
| Document.createEvent(type)        | 創建一個 `Event` 對象，調用 `Event.initEvent` 之後可作為標籤事件掛載目標                                     |
| Document.createRange()            | 創建一個 `Range` 對象                                                                                        |

### 查找方法

除了創建元素的方法之外，對於文檔結構中元素的查詢也是 JS 的 DOM 操作中非常重要的一塊

| Methods                                    | Description                                                               |
| :------------------------------------------ | :------------------------------------------------------------------------- |
| Document.getElementById(id)                | 依據 `id` 查找元素，返回一個 `Element` 對象                               |
| Document.getElementsByClassName(className) | 依據 `class` 查找元素，返回一個 `HTMLCollection` 對象，類似於一個元素列表 |
| Document.getElementsByTagName(tagName)     | 依據`標籤名`查找元素，返回一個 `HTMLCollection` 對象                      |
| Document.querySelector(selector)           | 使用選擇器查找元素，返回第一個 `Element` 對象                             |
| Document.querySelectorAll(selector)        | 使用選擇器查找元素，返回一個 `NodeList` 對象，保存所有符合選擇器的節點    |

# 結語

本篇本來只打算介紹一下 BOM 和 DOM 的結構，但是自己在網上查了好幾輪之後發現，不簡單瀏覽一遍屬性和方法真的是會搞不清楚到底在幹嘛。本篇洋洋灑灑寫了很多，基本上都列在 API 列表裡面，本篇只是抽取一些比較常用或是常見的屬性和方法做介紹。由於一個人的時間精力有限，對於技術應該只需要了解結構和大概，操作細節大多是到應用的時候才慢慢熟練出來的。希望讀者能透過本篇簡單理解 `Window 對象`和 `Document 對象`到底保存了哪些東西，具備了什麼樣的能力，往後有需要的時候就能知道要查哪些對象和用法（例如定位相關就需要用到 Navigator 對象）。
