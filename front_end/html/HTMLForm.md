# HTML 基礎：Form 表單

@[TOC](文章目錄)

<!-- TOC -->

- [HTML 基礎：Form 表單](#html-基礎form-表單)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Form Attribute 表單屬性](#form-attribute-表單屬性)
    - [Basic 基本款](#basic-基本款)
    - [裝飾一下](#裝飾一下)
    - [數據綁定](#數據綁定)
- [結語](#結語)

<!-- /TOC -->

## 簡介

在開始學網頁的時候，一定都會從 HTML 開始著手，加上 CSS 的樣式，以及動態操作 HTML 的 JavaScript。其中，`表單(Form)`可以說是前端網頁的靈魂，所謂的前端除了展示靜態資源之外，大多數都會有用戶與系統交互的部分，而`表單(Form)`則是收集用戶輸入的重要依據。本篇主要從 HTML 的角度看待表單的構成方式，以及相關元素和屬性的說明，而表單驗證和提交等由 JS 動態操作的部分將不會在這邊細說。

## 參考

<table>
  <tr>
    <td>form tag-MDN</td>
    <td><a href="https://developer.mozilla.org/zh-TW/docs/Web/HTML/Element/form">https://developer.mozilla.org/zh-TW/docs/Web/HTML/Element/form</a></td>
  </tr>
  <tr>
    <td>HTML form 标签</td>
    <td><a href="https://www.w3school.com.cn/tags/tag_form.asp">https://www.w3school.com.cn/tags/tag_form.asp</a></td>
  </tr>
</table>

# 正文

## Form Attribute 表單屬性

首先我們先列出 `<form>` 元素可用的相關屬性：

| Attribute    | Description                                                                                                    |
| ------------ | -------------------------------------------------------------------------------------------------------------- |
| action       | 指定提交表單的目標位置，JS 處理時較少用                                                                        |
| method       | 指定提交表單的 http 請求方法，JS 處理時較少用                                                                  |
| autocomplete | 規定是否開放自動填充的功能(瀏覽器緩存)，可選值：`on(默認)`、`off`                                              |
| enctype      | 指定表單提交的編碼類型，可選值：`application/x-www-form-urlencoded(默認)`、`multipart/form-data`、`text/plain` |
| novalidate   | 表示表單提交時默認不進行表單驗證                                                                               |
| target       | 表示在哪打開提交回覆的結果                                                                                     |

由於 HTML4 以後已經不推薦 form 元素的 `name` 屬性所以不使用，而由於使用 JS 來負責表單提交，所以 `action` 和 `method` 屬性也比較少用了。所以主要會使用到的屬性如下：

- `id`：表單唯一標示
- `enctype`：提交 file 時需要設為 `multipart/form-data`
- `novalidate`：取消默認表單驗證

由於大多數表單處理會由 JS 代理，所以一但使用 `e.preventDefault` 之後許多屬性就顯得多餘

### Basic 基本款

首先我們先來創建一個最最簡單的表單如下：

```html
<form id="form">
  <input type="text" name="account" />
  <input type="text" name="password" />
  <input type="submit" />
  <input type="reset" />
</form>
```

其中包含兩個輸入框，以及提交和重設按鈕各一個樣式如下：

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/form_basic.png)

這邊有三個要點：

1. `<input type="xxx"/>` 為表單的輸入框，用 `type` 來表示輸入類型，輸入過程會觸發 `input` 事件、完成修改觸發 `change` 事件
2. `<input type="submit"/>` 為默認的提交按鈕，將會觸發 `submit` 事件
3. `<input type="reset"/>` 為默認的重設按鈕，將會觸發 `reset` 事件

有關 `<input>` 輸入域的詳細介紹將留到下一篇解說，這邊主要關注整個表單的運作。由於往後都將透過 JS 來處理表單，`action`、`method`、`target` 等提交相關的屬性則可以捨棄不用。

使用 `<form>` 元素的好處在於，可透過 `document.forms[formId]` 來訪問表單元素，並透過 `name` 屬性來訪問表單各項輸入框元素。

### 裝飾一下

接下來我們加上一些 `form` 內部可用的元素：

- `<fieldset>`：外框
- `<legend>`：外框標題
- `<label>`：修飾 `<input>` 元素（輸入框說明、覆蓋樣式）
- `<menu>`：由於主流瀏覽器都不支持，也較少使用

```html
<form id="form" style="line-height: 30px; width: 350px;">
  <fieldset>
    <legend>Report</legend>
    <label>name: <input type="text" name="name" /></label><br />
    <label>title: <input type="text" name="title" /></label><br />
    <label style="vertical-align: top;" for="desc">description:</label>
    <textarea id="desc" name="description"></textarea><br />

    <input type="submit" />
    <input type="reset" />
  </fieldset>
</form>
```

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/form_decoration.png)

### 數據綁定

最後一部分我們將實現數據綁定。在 Vue 中我們看過 v-model 實現了數據和用戶輸入的綁定，這邊我們嘗試用原生 js 來實現數據綁定：

```js
// 首先是數據定義的部分
const formId = 'form'
const formData = {
  name: '',
  title: '',
  description: ''
}

// 接下來做數據和用戶輸入綁定
const form = document.forms[formId] // 使用 document.forms 提取表單
for (let key of Object.keys(formData)) {
  // 使用 formData 對表單初始化
  form[key].value = formData[key] // 使用 form.name 查找輸入元素
  // 監聽 input 事件，綁定到表單對象上
  form[key].addEventListener('input', function (e) {
    formData[key] = e.target.value
  })
}

// 監聽 submit 和 reset 事件
form.addEventListener('submit', function (e) {
  // 這邊接收一個 SubmitEvent 做參數
  console.log('submit')
  console.log(formData) // 查看表單內容
  // 取消默認行為（提交到 action、刷新頁面等）
  e.preventDefault() // 也可以使用 return false
})

form.addEventListener('reset', function (e) {
  console.log('reset')
  // 初始化數據
  for (let key of Object.keys(formData)) {
    formData[key] = ''
  }
})
```

如此一來我們就簡單使用 JS 綁定好用戶輸入數據啦

# 結語

`表單(Form)`一直是前端人機交互非常重要的一環，也因為 JS 的出現能夠更自由的提取和綁定數據項，而不需要寫死在 action 上。下一篇將會詳細介紹所有 `<input>` 元素可以使用的輸入類型和用法。
