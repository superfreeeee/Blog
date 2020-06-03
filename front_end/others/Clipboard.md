# Clipboard：黏貼簿操作

@[TOC](文章目錄)

<!-- TOC -->

- [Clipboard：黏貼簿操作](#clipboard黏貼簿操作)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Install 安裝](#install-安裝)
    - [CDN](#cdn)
    - [NPM](#npm)
  - [Create 創建對象(官方 Setup)](#create-創建對象官方-setup)
    - [參數說明](#參數說明)
    - [Sample](#sample)
  - [Usage in Html 標籤用法](#usage-in-html-標籤用法)
    - [1. Action 執行動作(`data-clipboard-action`)](#1-action-執行動作data-clipboard-action)
    - [2. Target 參照目標(`data-clipboard-target`)](#2-target-參照目標data-clipboard-target)
    - [3. Text 操作文本(`data-clipboard-text`)](#3-text-操作文本data-clipboard-text)
    - [Sample](#sample-1)
- [結語](#結語)

<!-- /TOC -->

## 簡介

我們常常會在一些網頁上看到"點擊複製鏈結"的按鈕（像是 github 的地址或是一些分享連結），本篇介紹一個輕量級而且封裝良好的第三方庫：<a href="https://clipboardjs.com/">clipboard.js</a>

自己也可以透過 `execCommand` 來實現點擊複製的操作，但由於自己寫的容易產生各式各樣的問題（封裝不完整、兼容性問題、內存泄露、焦點轉移），同時基於不重複造輪子的精神，有人開源當然是拿起來用啊！

## 參考

<table>
  <tr>
    <td>clipboard.js</td>
    <td><a href="https://clipboardjs.com/">https://clipboardjs.com/</a></td>
  </tr>
</table>

# 正文

本篇僅僅介紹 clipboard.js 庫的使用，其實現原理主要也是透過 `execCommand` 方法，這部分留到源碼解析的時候再說明

## Install 安裝

在開始使用前當然要先安裝（廢話）

### CDN

可以透過 CDN 引用：

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/clipboard.js/2.0.4/clipboard.min.js"></script>
```

<a href="https://github.com/zenorocha/clipboard.js/wiki/CDN-Providers">這裏</a>提供其他多個不同的靜態資源站點，不過都 2020 年了應該大家都引入模塊化開發了，所以筆者本身不推薦這種引用方式

### NPM

透過 `npm` 引入：

```bash
$ npm i clipboard --save
```

使用時：

```js
import ClipboardJS from 'clipboard'
```

## Create 創建對象(官方 Setup)

clipboard.js 的思想主要就是創建出一個對象負責管理"點擊複製"行為的按鈕：

```js
new ClipboardJS(<element>, <options>)
```

### 參數說明

```ts
element: String | HTMLElement | HTMLCollection | NodeList // Clipboard 對象的觸發目標
options: {
  target: Function // 返回一個 Element 對象，可動態選擇觸發對象，接收一個參數 triiger 即為 element 對象
  action: Function // 返回執行操作，可選值有 'copy'、'cut'
  text: Function // 返回操作的直接文本
  container: Object // Clipboard 對象掛載的容器，默認為 body
}
```

### Sample

```html
<input id="in" value="hello world" />
<button id="cp">Copy!</button>
```

```js
const _cp = new ClipboardJS('#id', {
  action: (trigger) => 'cut',
  target: (trigger) => document.querySelector('#in')
})
```

## Usage in Html 標籤用法

一種用法是全部都在創建 Clipboard 對象的時候根據 trigger 對象配置好所有行為（在 Create 大題），另一種用法是透過 html 標籤來配置目標對象和行為，屬性名都是 `data-clipboard-<option>`

### 1. Action 執行動作(`data-clipboard-action`)

配置執行動作，可選為 `copy(複製)`、`cut(剪下)`

### 2. Target 參照目標(`data-clipboard-target`)

執行動作的目標元素，可傳入選擇器字符串

### 3. Text 操作文本(`data-clipboard-text`)

可直接操作的文本字符串

### Sample

- cut + target

```html
<input id="input-1" value="hello world" />
<button id="cp1" data-clipboard-action="cut" data-clipboard-target="#input-1">
  Cut!
</button>
```

```js
const _cp1 = new ClipboardJS('#cp1')
```

- text

```html
<button id="cp2" data-clipboard-text="text: hello world">Copy!</button>
```

```js
const _cp2 = new ClipboardJS('#cp2')
```

# 結語

以上就是 clipboard.js 的使用方法，可以看得出來作者封裝的還挺不錯的，使用上非常便捷並且侵入性極小，不太容易發生命名衝突的問題，供大家參考。
