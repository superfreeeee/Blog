# HTML 基礎：Input 輸入框

@[TOC](文章目錄)

<!-- TOC -->

- [HTML 基礎：Input 輸入框](#html-基礎input-輸入框)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Types 類型](#types-類型)
    - [Common Property 通用屬性](#common-property-通用屬性)
  - [輸入類型](#輸入類型)
    - [Property 屬性](#property-屬性)
  - [選擇類型](#選擇類型)
    - [Property 屬性](#property-屬性-1)
  - [時間日期類型](#時間日期類型)
  - [本地資源](#本地資源)
    - [Property 屬性](#property-屬性-2)
  - [表單行為和其他](#表單行為和其他)
    - [Event 事件](#event-事件)
  - [Event 常用監聽事件](#event-常用監聽事件)
- [結語](#結語)

<!-- /TOC -->

## 簡介

<a href="https://blog.csdn.net/weixin_44691608/article/details/106803022">上一篇：HTML 基礎：Form 表單</a>，介紹了 HTML 中 `<form>` 元素的用法。接下來本篇將來詳細展開 `<input>` 輸入元素的所有類型以及用法。

## 參考

<table>
  <tr>
    <td>HTML input Tag-w3schools</td>
    <td><a href="https://www.w3schools.com/tags/tag_input.asp">https://www.w3schools.com/tags/tag_input.asp</a></td>
  </tr>
  <tr>
    <td>input-MDN</td>
    <td><a href="https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/input">https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/input</a></td>
  </tr>
</table>

# 正文

## Types 類型

首先我們先列出所有 `input` 可選的類型，主要可以分成五大類：

- 輸入類型：基本上就是輸入文字，不過在不同設備會默認出現與 `type` 對應的鍵盤
- 選擇類型：主要是寫好選項模板，在有限集合內選定單一或複數個值
- 時間日期類型：選擇與日期或時間相關的值
- 本地資源：需要讀取本地資源，如文件、圖片或其他任意類型的文件
- 表單行為：表單的提交、點擊、重置等功能性的交互元素

### Common Property 通用屬性

在開始介紹各種不同的 `type` 類型之前，我們先看看有哪些屬性是幾乎所有 `input` 都能共用的（無可選值則表示出現與否，如 `<input name="password" required />`）：

| Attr         | Description                                      | 可選值                |
| ------------ | ------------------------------------------------ | --------------------- |
| autocomplete | 是否允許自動填充                                 |
| autofocus    | 表單加載時自動聚焦(focus)                        |
| disabled     | 是否禁用                                         |
| list         | 自動填充選項                                     | `<datalist>` 的 id 值 |
| name         | 表示輸入框的名字，也就是表單的域(field)          | string                |
| type         | 標示輸入框類型                                   | string                |
| value        | 輸入框的值，寫在 html 則為初始值，隨用戶操作改變 | string                |

- **注意**！儘管輸入框有非常多種類型，但是不變的是所有 `value` 都是`字符串(string)` 的形式不變。

## 輸入類型

| Type         | Description |
| ------------ | ----------- |
| text         | 文本        |
| email        | 郵箱        |
| number       | 數字        |
| password     | 密碼        |
| url          | 網址        |
| search       | 搜索        |
| tel          | 電話        |
| `<textarea>` | 多行文本    |

首先是輸入類型，不同 `type` 值的區別在於在移動端或是其他會出現默認鍵盤的時候會出現與 `type` 相對應的鍵盤類型，而 `<textarea>` 則表示多行文本輸入框，詳細用法這邊就不展開了

下面為使用範例：

- index.html

```html
<fieldset>
  <legend>輸入類型</legend>
  <label>文本：<input type="text" /></label>
  <label>郵箱：<input type="email" /></label>
  <label>數字：<input type="number" /></label>
  <label>密碼：<input type="password" /></label>
  <label>網址：<input type="url" /></label>
  <label>搜索：<input type="search" /></label>
  <label>電話：<input type="tel" /></label>
</fieldset>
```

- web 下的樣式

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/input_input.png)

- 不同 `type` 在移動端上的鍵盤區別

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/input_text.png)

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/input_email.png)

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/input_number.png)

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/input_url.png)

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/input_phone.png)

### Property 屬性

這邊再列出與輸入類型的 `type` 相關的 `input` 屬性，主要用途在於表單驗證時候使用：

| Attr        | type                             | Description        |
| ----------- | -------------------------------- | ------------------ |
| minlength   | password, search, tel, text, url | 輸入最小長度       |
| maxlength   | password, search, tel, text, url | 輸入最大長度       |
| placeholder | password, search, tel, text, url | 輸入提示           |
| size        | email, password, tel, text       | 元素大小           |
| pattern     | password, text, tel              | 有效內容正則表達式 |
| min         | number                           | 最小值             |
| max         | number                           | 最大值             |

這邊就不上圖了，可以自己到 codepen 嘗試

## 選擇類型

| Type       | Description |
| ---------- | ----------- |
| radio      | 單選        |
| checkbox   | 多選        |
| color      | 顏色        |
| range      | 拉桿        |
| `<select>` | 下拉選單    |

再來是選擇類型，分成單選、多選、顏色、拉桿、下拉選單等。

單選、多選通過設置多個 `name` 相同的 `input` 來實現。

下拉選單則透過 `<select>` 或 `<input list>` 來指定下拉單，並透過 `<datalist>` 和 `<option>` 來填充可選選項，使用範例如下：

```html
<fieldset>
  <legend>選擇類型</legend>
  <label>
    單選 -
    <label>选项 1：<input type="radio" name="radio" value="1" /> </label>
    <label>选项 2：<input type="radio" name="radio" value="2" /> </label>
    <label>选项 3：<input type="radio" name="radio" value="3" /> </label>
  </label>

  <label>
    多選 -
    <label>选项 1：<input type="checkbox" name="checkbox" value="1" /> </label>
    <label>选项 2：<input type="checkbox" name="checkbox" value="2" /> </label>
    <label> 选项 3：<input type="checkbox" name="checkbox" value="3" /> </label>
  </label>

  <label>顏色：<input type="color" name="color" /></label>
  <label>拉桿：<input type="range" name="range" /></label>

  <input list="selectList" />
  <datalist id="selectList">
    <option value="1"></option>
    <option value="2"></option>
    <option value="3"></option>
  </datalist>

  <select>
    <!-- value 為選中時的值，a 為展示出來的值 -->
    <option value="1">a</option>
    <option value="2">b</option>
    <option value="3">c</option>
  </select>
</fieldset>
```

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/input_choose.png)

`<select>` 是在有限值內選擇，而 `<input list>` 則是提供可選選項也能自由填充內容

### Property 屬性

| Attr    | type            | Description        |
| ------- | --------------- | ------------------ |
| value   | radio, checkbox | 表示選中時代表的值 |
| checked | radio, checkbox | 表示此元素是否選中 |

## 時間日期類型

| Type           | Description | valu-format      |
| -------------- | ----------- | ---------------- |
| date           | 日期        | 2020-06-18       |
| datetime-local | 本地日期    | 2020-06-18T00:00 |
| month          | 月份        | 2020-06          |
| week           | 週          | 2020-W25         |
| time           | 時間        | 00:00            |

接下來是與時間日期相關的輸入，默認都會有日期選擇框，範例如下：

```html
<fieldset>
  <legend>日期時間相關</legend>
  <label>日期：<input type="date" /></label>
  <label>本地日期：<input type="datetime-local" /></label>
  <label>月份：<input type="month" /></label>
  <label>週：<input type="week" /></label>
  <label>時間：<input type="time" /></label>
</fieldset>
```

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/input_time_date.png)

- 不同類型的默認選擇樣式

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/input_date.png)

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/input_datetime_local.png)

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/input_month.png)

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/input_week.png)

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/input_time.png)

## 本地資源

| Type | Description |
| ---- | ----------- |
| file | 文件        |

```html
<fieldset>
  <legend>本地資源</legend>
  <label>文件：<input type="file" /></label>
</fieldset>
```

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/input_file.png)

### Property 屬性

| Attr     | Description              |
| -------- | ------------------------ |
| accept   | 接受的文件類型           |
| capture  | 捕獲資源的位置           |
| files    | 已選擇文件，`value` 無效 |
| multiple | 是否接受多個文件         |

**注意**！在 `type="file"` 的情況下，讀取 `value` 是沒有意義的，需要使用如下來獲得文件列表：

```js
const form = document.forms[formId]
const fileInput = form['file']
// files 為已選取的文件列表
console.log(fileInput.files)
```

## 表單行為和其他

| Type   | Description  |
| ------ | ------------ |
| hidden | 隱藏         |
| button | 按鈕         |
| image  | 帶圖樣的提交 |
| submit | 提交         |
| reset  | 重置         |

最後是與表單行為相關的輸入類型，`type="button"` 與 `<button>` 的結果是一模一樣的，直接看範例：

```html
<fieldset>
  <legend>表單行為相關和其他</legend>
  <label>隱藏：<input type="hidden" /></label>
  <label>按鈕：<input type="button" value="button" /></label>
  <label>提交：<input type="submit" /></label>
  <label>帶圖標提交：<input type="image" /></label>
  <label>重置：<input type="reset" /></label>
</fieldset>
```

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/input_actions.png)

`type="image"` 時，`src` 表示圖標路徑，`alt` 表示替代文字，比較少使用，這邊就不展開說明了

### Event 事件

對於這幾個元素來說屬性值本身並不是太重要，相對更重要的則是動作所觸發的事件：

| Actions/Event | Description                           |
| ------------- | ------------------------------------- |
| click         | 點擊行為                              |
| submit        | 提交動作（點擊 `type="submit"` 元素） |
| reset         | 重置動作（點擊 `type="reset"` 元素）  |

在 JS 使用時可以這樣綁定事件處理函數：

```js
const form = document.forms[formId]
form.addEventListener('submit', function (e) {
  // 提交事件 ...
})
form.addEventListener('reset', function (e) {
  // 重置事件 ...
})
```

## Event 常用監聽事件

有了 HTML 定義各樣的輸入框，這邊提出一些用戶輸入過程中可能觸發的事件，可用來綁定自定義處理函數：

| Event  | Actions  |
| ------ | -------- |
| click  | 點擊     |
| focus  | 聚焦     |
| blur   | 聚焦移除 |
| change | 值改變   |
| input  | 輸入行為 |
| submit | 表單提交 |
| reset  | 表單重置 |

後續將會為表單處理以及相關事件的觸發順序和應用等單獨說明

# 結語

本篇主要介紹 HTML 中有關 `<input>` 標籤的完整應用，透過指定不同的 `type` 可以有非常多種的輸入類型，豐富了原生 HTML 的人機交互能力。
