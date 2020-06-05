# js-cookie：輕量級 cookie 框架

@[TOC](文章目錄)

<!-- TOC -->

- [js-cookie：輕量級 cookie 框架](#js-cookie輕量級-cookie-框架)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Import 引入方法](#import-引入方法)
    - [CDN](#cdn)
    - [NPM](#npm)
  - [js-cookie 接口說明](#js-cookie-接口說明)
    - [三種基本方法語法](#三種基本方法語法)
    - [避免衝突](#避免衝突)
  - [Cookie Attribute 屬性說明](#cookie-attribute-屬性說明)
    - [1. expires 屬性](#1-expires-屬性)
    - [2. path 屬性](#2-path-屬性)
    - [3. domain 屬性](#3-domain-屬性)
    - [4. secure 屬性](#4-secure-屬性)
    - [5. sameSite 屬性](#5-samesite-屬性)
  - [Converters 攔截器](#converters-攔截器)
  - [封裝](#封裝)
- [結語](#結語)

<!-- /TOC -->

## 簡介

當前前後端分離網頁常常會使用 http/https 協議搭配 RESTful 風格作為接口約定，由於 http 協議為無連接，這為前端網頁登入狀態保持來說是一個問題，有些後端會使用 session 來保持狀態，這位後端的加密和訪問控制帶來好處；同樣的，前端也需要一個手段來控制用戶訪問權限，同時又不想為服務器帶來過大的負擔，這時候就可以用上輕量方便的 cookie 技術。

cookie 就好像瀏覽器的緩存一樣，可以用來保存簡單的信息，如 session key 等，作為前端則可以用來保存用戶狀態信息、或是登入的 uuid 等。

接下來介紹一個超輕量的 cookie 控制庫，同時帶來一些簡單的封裝，讓你的網頁用戶不再需要每次刷新就重新登入一次。

## 參考

<table>
  <tr>
    <td>輕量級JS Cookie外掛js-cookie的使用方法</td>
    <td><a href="https://codertw.com/%E5%89%8D%E7%AB%AF%E9%96%8B%E7%99%BC/204962/">https://codertw.com/%E5%89%8D%E7%AB%AF%E9%96%8B%E7%99%BC/204962/</a></td>
  </tr>
  <tr>
    <td>js-cookie github</td>
    <td><a href="https://github.com/js-cookie/js-cookie">https://github.com/js-cookie/js-cookie</a></td>
  </tr>
</table>

# 正文

從官方 readme 裡面可以看到這個庫儘管極為輕量，依舊提供一些威力不俗的能力，然而本篇只介紹簡單的操作和封裝。

## Import 引入方法

### CDN

第一種可以透過 CDN 直接引入資源

```html
<script src="https://cdn.jsdelivr.net/npm/js-cookie@rc/dist/js.cookie.min.js"></script>
```

### NPM

第二種是更為常用的模塊化開發配合 NPM 包管理器

```bash
$ npm i js-cookie
```

```js
import Cookies from 'js-cookie'
```

## js-cookie 接口說明

### 三種基本方法語法

```js
// 設定 cookie
Cookies.set(name, value, attribute)

// 獲取 cookie
Cookies.get(name, attribute)

// 移除 cookie
Cookies.remove(name, attribute)
```

### 避免衝突

如果採用直接引入(CDN)的方式有可能產生命名衝突，這時可以透過 `noConflict` 方法來避免衝突：

```js
var _cookie = Cookies.noConlict()
```

## Cookie Attribute 屬性說明

基本方法的最後一個參數都提供了一個機會來指定一些屬性，以下一個一個介紹其用途

### 1. expires 屬性

number 類型（整數），指定 cookie 的存活時間（單位：天），默認存活期間為一個 session

```js
Cookies.set('name', 'value', { expires: 365 })
```

### 2. path 屬性

string 類型：指定 cookie 可見路徑，默認為 `/`

```js
Cookies.set('name', 'value', { path: '' })
```

### 3. domain 屬性

string 類型，指定對應主機域名，默認對所有域名可見

```js
Cookies.set('name', 'value', { domain: 'google.com' })
```

### 4. secure 屬性

boolean 類型，說明傳送 cookie 時是否需要用安全協議(https)，默認為 `false`

```js
Cookies.set('name', 'value', { secure: true })
```

### 5. sameSite 屬性

string 類型，指定傳送 cookie 時是否必須為同源（屏蔽跨域請求），可選值為 `lax`、`strict`，默認為空

```js
Cookies.set('name', 'value', { sameSite: 'lax' })
```

## Converters 攔截器

好比 axios 的 interceptor，可以在每次 cookie 操作前實現自定義操作，其中又可分為 `Read` 操作和 `Write` 操作兩種（代碼轉自官方示例）

```js
// Read
var _cookies = Cookies.withConverter({
  read: function (value, name) {
    if (name === 'escaped') {
      return unescape(value)
    }
    // Fall back to default for all other cookies
    return Cookies.converter.read(value, name)
  }
})

// Write
Cookies.withConverter({
  write: function (value, name) {
    return value.toUpperCase()
  }
})
```

## 封裝

以下提供一個最簡單的封裝案例，以模塊化開發為例，建立一個叫 `auth.js` 的獨立文件

```js
import Cookies from 'js-cookie'

const _cookie = Cookies.withConverter({
  write: function (value, name) {
    cosnole.log(`set cookie: { name=${name}, value=${value} }`)
    return value
  },
  read: function (value, name) {
    cosnole.log(`get cookie: { name=${name}, value=${value} }`)
    return value
  }
})

const tokenKey = 'SAMPLE-COOKIE-NAME'

export function getToken() {
  return _cookie.get(tokenKey)
}

export function setToken(value) {
  return _cookie.set(tokenKey, value)
}

export function removeToken() {
  return _cookies.remove(tokenKey)
}
```

# 結語

本篇介紹 js-cookie，輕量級的 cookie 操作庫。其實透過原生瀏覽器接口(document.cookie)同樣能實現上面這些操作，不過作為一個封裝好的第三方開源庫，可以省下程序員的精力能集中在業務功能上。
