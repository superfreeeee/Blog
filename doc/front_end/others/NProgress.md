# NProgress：簡單進度條

@[TOC](文章目錄)

<!-- TOC -->

- [NProgress：簡單進度條](#nprogress簡單進度條)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Import 引入](#import-引入)
    - [CDN](#cdn)
    - [NPM](#npm)
  - [接口說明](#接口說明)
  - [Configuration 配置選項](#configuration-配置選項)
    - [1. minimum 最小值](#1-minimum-最小值)
    - [2. template 模板](#2-template-模板)
    - [3. easing & speed 移動模式](#3-easing--speed-移動模式)
    - [4. trickle & trickleSpeed & trickleRate 自動前進](#4-trickle--tricklespeed--tricklerate-自動前進)
    - [5. showSpinner 顯示加載圖樣](#5-showspinner-顯示加載圖樣)
    - [6. parent 父元素](#6-parent-父元素)
  - [Usage 使用](#usage-使用)
- [結語](#結語)

<!-- /TOC -->

## 簡介

有些時候網頁需要向後端或是其他第三方服務器拉取資源，甚至於某些內容"豐富"的頁面加載都是頗為耗時的行為，長時間的白屏對於用戶體驗是極差，所以 nprogress 提供了一個簡單實現的進度條，並可由程序員手動控制進度。

## 參考

<table>
  <tr>
    <td>nprogress github</td>
    <td><a href="https://github.com/rstacruz/nprogress">https://github.com/rstacruz/nprogress</a></td>
  </tr>
</table>

# 正文

## Import 引入

### CDN

```html
<script src="nprogress.js"></script>
<link rel="stylesheet" href="nprogress.css" />
```

### NPM

```bash
$ npm i nprogress --save
```

```js
import NProgress from 'nprogress'
import 'nprogress/nprogress.css'
```

- **注意！**：記得加 css！記得加 css！記得加 css！（因為很重要所以說三遍，忘了加你啥都看不到）

## 接口說明

這個庫實在是很簡單，基本上只有少少幾個基本操作

```js
NProgress.start() // 開始
NProgress.done(bool)) // 結束，true 時表示即使當前進度為 0 還是顯示一次
Nprogress.set(percent) // 設置進度條長度
Nprogress.inc(percent) // 前進一點點(前進指定長度)
```

## Configuration 配置選項

NProgress 同時提供一些配置選項可客製化進度條的屬性，示例代碼引用自官方示例

### 1. minimum 最小值

調用 `start` 方法時顯示進度條的最小值（初始狀態），默認為 `0.08`

```js
NProgress.configure({ minimum: 0.1 })
```

### 2. template 模板

抽換進度條的默認模板

```js
NProgress.configure({
  template: "<div class='....'>...</div>"
})
```

### 3. easing & speed 移動模式

進度條前進的動畫樣式，使用 CSS 的 ease 分類，速度單位為毫秒(ms)，默認值 `ease & 200`

```js
NProgress.configure({ easing: 'ease', speed: 500 })
```

### 4. trickle & trickleSpeed & trickleRate 自動前進

boolean 類型，決定進度條是否會自動緩慢前進，默認為 `true`

speed 表示自動前進間隔，單位為毫秒(ms)，默認為 `800`

rate 表示自動前進長度比例，默認為 `0.02`

```js
NProgress.configure({ trickle: false })
```

### 5. showSpinner 顯示加載圖樣

boolean 類型，決定是否顯示右側的小圈圈，默認為 true

```js
NProgress.configure({ showSpinner: false })
```

### 6. parent 父元素

Selector 選擇器字符串，決定進度條的父元素，也就是進度條顯示位置，默認為 `body`

```js
NProgress.configure({ parent: '#container' })
```

## Usage 使用

進度條使用可以用於圖片加在、文件加載或其他異步操作等需要等待的操作，以下範例為 vue-router 裡頁面加載的範例

```js
import router from './router'
import NProgress from 'nprogress'
import 'nprogress/nprogress.css'

router.beforeEach((to, from, next) => {
  NProgress.start()
  next()
})

router.afterEach(() => {
  // finish progress bar
  NProgress.done(true)
})
```

# 結語

本篇介紹一個簡單的進度條，看似多餘的樣式，但是對於需要較長時間等待的用戶體驗極為重要，使用者不仿自己實現看看自定義的加載進度條，也是能夠玩出很多花樣的。
