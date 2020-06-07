# CSS 基礎：Selector 選擇器

@[TOC](文章目錄)

<!-- TOC -->

- [CSS 基礎：Selector 選擇器](#css-基礎selector-選擇器)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Overview 概述](#overview-概述)
  - [基礎選擇器(Basic Selectors)](#基礎選擇器basic-selectors)
    - [標籤選擇器(Type/Tag Selectors)：`tagName`](#標籤選擇器typetag-selectorstagname)
      - [Sample](#sample)
    - [類選擇器(Class Selectors)：`.className`](#類選擇器class-selectorsclassname)
      - [Sample](#sample-1)
    - [ID 選擇器(ID Selectors)：`#idName`](#id-選擇器id-selectorsidname)
      - [Sample](#sample-2)
    - [通配符(Universal Selectors)：`*`](#通配符universal-selectors)
      - [Sample](#sample-3)
    - [屬性選擇器(Attribute Selectors)：`selector[attr="value"]`](#屬性選擇器attribute-selectorsselectorattrvalue)
      - [Sample](#sample-4)
    - [狀態選擇器(State Selectors)：`selector:state`](#狀態選擇器state-selectorsselectorstate)
      - [Sample](#sample-5)
    - [組選擇器(Selector list)：`selector, selector, ...`](#組選擇器selector-listselector-selector-)
      - [Sample](#sample-6)
      - [相似比較：`AB`、`A B`、`A,B`](#相似比較aba-bab)
  - [複合選擇器(Multiple Selector)](#複合選擇器multiple-selector)
    - [相鄰兄弟選擇器(Adjacent Sibling Selectors)：`A + B`](#相鄰兄弟選擇器adjacent-sibling-selectorsa--b)
      - [Sample](#sample-7)
    - [兄弟選擇器(General Sibling Selectors)：`A ~ B`](#兄弟選擇器general-sibling-selectorsa--b)
      - [Sample](#sample-8)
    - [直接子代選擇器(Child Selectors)：`A > B`](#直接子代選擇器child-selectorsa--b)
      - [Sample](#sample-9)
    - [後代選擇器(Descendant Selectors)：`A B`](#後代選擇器descendant-selectorsa-b)
      - [Sample](#sample-10)
  - [偽選擇器(Pseudo Selectors)](#偽選擇器pseudo-selectors)
    - [偽類(Pseudo Classes)：`selector:pseudo-class`](#偽類pseudo-classesselectorpseudo-class)
      - [Sample](#sample-11)
    - [偽元素(Pseudo Elements)：`selector::pseudo-element`](#偽元素pseudo-elementsselectorpseudo-element)
      - [Sample](#sample-12)
- [結語](#結語)

<!-- /TOC -->

## 簡介

網頁三大件 HTML、CSS、JavaScript，其中 HTML 負責網頁模版的部分（即便到後面使用 JS 框架建造虛擬 DOM，開發者們依舊保留了 HTML 的模板樣式，如 Vue 的 template、React 的 JSX 等）；而 JavaScript 則負責網頁的動態部分，如操作 dom、抽換 class、監聽事件處理等。

而本篇的主題：CSS 則是負責網頁標籤的樣式部分。CSS 的盒子模型能夠處理元素的覆蓋區域大小，控制元素的表現形式（flex、grid、table、block、inline），還有其他如背景、字體操作。CSS 幾乎囊括了一切開發者能想到對元素樣式的修改，合理靈活的運用 CSS 幾乎能做出任何你想得到的效果。

然而在聲明樣式表的同時有一個最重要的能力，也就是**選擇器(Selector)**，CSS 需要透過選擇器材能夠定位到樣式真正生效的元素上。本篇就來全盤解析一下 CSS 的選擇器到底有哪些，以及如何使用。

## 參考

<table>
  <tr>
    <td>CSS Selectors MDN</td>
    <td><a href="https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Selectors">https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Selectors</a></td>
  </tr>
</table>

# 正文

## Overview 概述

在開始介紹一個個選擇器之前，我們需要先來將選擇器分類，如下表：

- 基礎選擇器(Basic Selectors)
  - 標籤選擇器(Type/Tag Selectors)
  - 類選擇器(Class Selectors)
  - ID 選擇器(ID Selectors)
  - 通配符(Universal Selectors)
  - 屬性選擇器(Attribute Selectors)
  - 狀態選擇器(State Selectors)
  - 組選擇器(Selector list)
- 複合選擇器(Multiple Selector)
  - 相鄰兄弟選擇器(Adjacent Sibling Selectors)
  - 兄弟選擇器(General Sibling Selectors)
  - 直接子代選擇器(Child Selectors)
  - 後代選擇器(Descendant Selectors)
- 偽選擇器(Pseudo Selectors)
  - 偽類(Pseudo Classes)
  - 偽元素(Pseudo Elements)

其中`基礎選擇器`定義了最基本形式的選擇器類型，可以用於指定特定目標元素特徵；`複合選擇器`則定義那些多個元素相關連的，由元素間的關聯性來決定的樣式；最後一種`偽選擇器`也就是狀態選擇器，再加上一些特殊的複合選擇器，詳細直接進入說明會更清楚明瞭。

## 基礎選擇器(Basic Selectors)

### 標籤選擇器(Type/Tag Selectors)：`tagName`

 由於 CSS 標籤都是套用在 DOM 樹上，標籤選擇器則可以透過直接指定標籤來選定這些元素，語法如下：

```css
tagName {
  /* attributes */
}
```

#### Sample

- html

```html
<body>
  <h1>Title</h1>
  <ul>
    <li>item1</li>
    <li>item2</li>
    <li>item3</li>
    <li>item4</li>
  </ul>
  <p>Paragraph 1</p>
</body>
```

- css

```css
body {
  margin: 0;
  height: 100vw;
}

li {
  line-height: 30px;
  font-size: 15px;
}

p {
  color: red;
}
```

### 類選擇器(Class Selectors)：`.className`

類選擇器透過指定標籤的 `class` 屬性中的字符串來選定元素，語法如下：

```css
.className {
  /* attributes */
}
```

#### Sample

- html

```html
<div class="wrapper">
  <div class="list">
    <div class="item">1</div>
    <div class="item">2</div>
    <div class="item">3</div>
    <div class="item">4</div>
  </div>
</div>
```

- css

```css
.wrapper {
  width: 1680px;
  margin: 0 auto;
}

.list {
  max-width: 500px;
  padding: 10px 40px;
}

.item {
  color: red;
  font-size: 15px;
}
```

### ID 選擇器(ID Selectors)：`#idName`

類選擇器透過 `class` 屬性來指定元素，而 ID 選擇器則透過 `id` 屬性來指定元素。與 `class` 不同的是，`id` 屬性同時只會在一個元素上作用，也就是指定特定元素的唯一標示符，語法如下：

```css
#idName {
  /* attributes */
}
```

#### Sample

- html

```html
<div id="sample"></div>
```

- css

```css
#sample {
  border-bottom: 1px solid red;
}
```

### 通配符(Universal Selectors)：`*`

有時候我們可能需要將某些樣式套用到所有標籤上，就可以使用通配符 `*`，將會匹配所有元素標籤。

**注意！**然而並不推薦這樣的使用方式，因為會使 CSSOM 選染樹的開銷急劇增加，CSS 選擇器的使用應能越接近渲染數底部越好。通常通配符應該與其他選擇器共同作用來縮減目標元素的範圍，以減輕渲染數的負擔

- 語法

```css
* {
  /* attributes */
}
```

#### Sample

```css
* {
  margin: 0;
  font-size: 12px;
}
```

### 屬性選擇器(Attribute Selectors)：`selector[attr="value"]`

有時候我們想選擇的對象並沒有特定的 `class` 或 `id` 值，而是想透過標籤屬性的特徵來指定元素，這時候就可以使用屬性選擇器，屬性選擇器內部又可以分成多種屬性值的判別方式

- 語法

```css
/* 指定屬性存在 */
selector [attr] {
}

/* 指定屬性值 */
selector [attr='value'] {
}

/* 屬性值為空格隔開的字符串列表，其中存在指定屬性值 */
selector [attr~='value'] {
}

/* 指定屬性值或以指定屬性值為前綴，如 value 或是 value- */
selector [attr|='value'] {
}

/* 以指定屬性值開頭 */
selector [attr^='value'] {
}

/* 以指定屬性值結尾 */
selector [attr$='value'] {
}

/* 包含指定屬性值 */
selector [attr*='value'] {
}

/* operator、value 可為上述形式，i 表示忽略大小寫 */
selector [attr operator value i] {
}
```

- `selector`：為三種基本形式 `tagName`、`.className`、`#idName` 之一

#### Sample

- html

```html
<div title="a b c"></div>
```

- css

```css
div[title~='b'] {
}
```

### 狀態選擇器(State Selectors)：`selector:state`

HTML 元素通常會自帶一些狀態，而狀態選擇器可以選擇指定狀態下的元素形式，如焦點(focus)、訪問(visited)、激活(active)等，以下列出選擇器語法和常見狀態

- 語法

```css
selector :state {
}
```

- 常見狀態

| State    | 狀態描述                                     |
| -------- | -------------------------------------------- |
| active   | 激活狀態（例如鼠標點擊下的狀態）             |
| checked  | radio 選中時的狀態                           |
| disabled | 禁用狀態（輸入框、選擇器）                   |
| enabled  | 啟用狀態                                     |
| focus    | 獲得焦點的狀態                               |
| hover    | 鼠標懸浮在元素上的狀態（觸控屏的判定有問題） |
| valid    | 表單元素未通過驗證的狀態                     |
| link     | 尚未訪問的鏈結                               |
| visited  | 已訪問過的鏈結                               |
| optional | 非必填的表單元素                             |
| required | 必填的表單元素                               |
| target   | id 與當前 URL 錨點相同的元素                 |

#### Sample

- html

```html
<div id="root"></div>
<input id="input-name"></input>
<a id="to-github"></a>
```

- css

```css
#root:hover {
  outline: 1px solid red;
}

#input-name:disabled {
  background-color: lightgray;
}
```

### 組選擇器(Selector list)：`selector, selector, ...`

有時候我們同一組樣式想要套用在多個擇器上，透過 `,` 來連接多個選擇器

- 語法

```css
selector,
selector,
... {
}
```

#### Sample

- html

```html
<div class="item1"></div>
<div class="item2"></div>
<div class="item3"></div>
<div class="item4"></div>
```

- css

```css
.item1,
.item2,
.item3,
.item4 {
  font-size: 15px;
}
```

#### 相似比較：`AB`、`A B`、`A,B`

這邊可能會出現三種容易搞混的選擇器（後代選擇器下面有介紹），分別是 `AB`、`A B`、`A,B`，直接上代碼：

```html
<div class="a b">1</div>
<div class="a">
  <div>
    <div class="b">2</div>
  </div>
</div>
```

```css
.a.b {
  color: red;
}

.a .b {
  color: orange;
}

.a,
.b {
  font-size: 20px;
}
```

- 說明 1：第一個選擇器會作用在內容為 1 的標籤上，`AB` 的含義是匹配同時滿足 A 和 B 兩個選擇器的元素
- 說明 2：第二個選擇器會作用在內容為 2 的標籤上，`A B` 為後代選擇器，在後面的段落有說明
- 說明 3：第三個選擇器會同時作用在兩個標籤上，`A,B` 表示匹配 A 或 B 的所有元素

## 複合選擇器(Multiple Selector)

### 相鄰兄弟選擇器(Adjacent Sibling Selectors)：`A + B`

這個選擇器用於匹配直接相鄰的兄弟元素（後面的一個），語法如下

- 語法

```css
selector + selector {
}
```

#### Sample

- html

```html
<ul>
  <li>item1</li>
  <li>item2</li>
  <li>item3</li>
  <li>item4</li>
</ul>

<div style="display: flex">
  <div class="tag">tag1</div>
  <div class="tag">tag2</div>
  <div class="tag">tag3</div>
</div>
```

- css

```css
li + li {
  margin-top: 5px;
}

.tag + .tag {
  margin-left: 10px;
}
```

- 說明：只會匹配 item2、item3、item4、tag2、tag3，而不會匹配 item1、tag1

### 兄弟選擇器(General Sibling Selectors)：`A ~ B`

這個與上一個選擇器相比沒有那麼嚴格，只要是同層級的兄弟元素都能夠匹配，不一定要直接相鄰

- 語法

```css
selector ~ selector {
}
```

#### Sample

- html

```html
<div>
  <div class="a">1</div>
  <div class="b">b-1</div>
  <div class="a">2</div>
  <div class="b">b-2</div>
  <div class="a">3</div>
  <div class="b">b-3</div>
  <div class="a">4</div>
</div>
```

- css

```css
.a ~ .a {
  background-color: red;
}

.b ~ .b {
  background-color: orange;
}
```

### 直接子代選擇器(Child Selectors)：`A > B`

直接子代選擇器需要滿足 A 匹配父元素而 B 匹配直接子代的子代元素

- 語法

```css
selector > selector {
}
```

#### Sample

- html

```html
<div class="a">
  <div class="b">1</div>
</div>

<div class="a">
  <div>
    <div class="b">2</div>
  </div>
</div>
```

- css

```css
.a > .b {
  font-size: 30px;
}
```

- 說明：只有內容為 1 的標籤會套用樣式，因為內容為 2 的標籤並不是 `class="a"` 的直接子代

這邊對於直接子代的限定比較嚴格，也是邏輯上限制較大的匹配模式，相較於後面要介紹的後代選擇器，在開銷的表現上比較好，也應該優先選擇直接子代選擇器

### 後代選擇器(Descendant Selectors)：`A B`

相較於上面的直接子代選擇器，後代選擇器的條件比較寬鬆一些，只要匹配 A 的元素的任意子代匹配 B 就能夠套用樣式

- 語法

```css
selector selector {
}
```

**注意！**這邊容易跟組選擇器(`A, B`)和聯合選擇器(`AB`)搞混，在上面的組選擇器中有說明

#### Sample

- html

```html
<div class="a">
  <div class="b">1</div>
</div>

<div class="a">
  <div>
    <div class="b">2</div>
  </div>
</div>
```

- css

```css
.a > .b {
  font-size: 30px;
}
```

- 說明：與直接子代選擇器比較，這邊 1、2 都會套用樣式，因為 `class="b"` 的元素都是 `class="a"` 的元素的後代

## 偽選擇器(Pseudo Selectors)

### 偽類(Pseudo Classes)：`selector:pseudo-class`

上面介紹過了狀態選擇器，而"狀態"類是偽類子級，這邊將介紹更完整的偽類。

- 語法

```css
selector :pseudo-class {
}
```

除了上面介紹過的一些狀態選擇器之外，還存在一些根據元素順序匹配的偽類：

| 偽類                   | 描述                                                            |
| ---------------------- | --------------------------------------------------------------- |
| first                  | 與 `@page` 連用，指打印時第一頁的元素                           |
| left                   | 與 `@page` 連用，指打印時頁面左側的樣式                         |
| right                  | 與 `@page` 連用，指打印時頁面右側的樣式                         |
| first-child            | 一組兄弟元素內的第一個元素                                      |
| first-of-type          | 一組兄弟元素內的第一個符合類型的元素                            |
| last-child             | 一組兄弟元素內的最後一個元素                                    |
| last-of-type           | 一組兄弟元素內的第一個符合類型的元素                            |
| nth-child(an + b)      | 一組兄弟元素內中第 an + b 個元素（n 為任意數，a、b 為指定常數） |
| nth-last-child(an + b) | 與 nth-child 類似，由後往前數                                   |
| nth-of-type(n)         | 一組兄弟元素內中第 n 個匹配的元素                               |
| nth-last-of-type(n)    | 一組兄弟元素內中第 n 個匹配的元素                               |
| only-child             | 一組兄弟元素內中唯一的元素                                      |
| only-of-type           | 一組兄弟元素內中唯一匹配的元素                                  |

#### Sample

- html

```html
<div>
  <p>p1</p>
  <ul>
    <li>li1</li>
    <li>li2</li>
    <li>li3</li>
    <li>li4</li>
  </ul>
  <p>p2</p>
  <ul>
    <li>li5</li>
    <li>li6</li>
    <li>li7</li>
    <li>li8</li>
  </ul>
  <p>p3</p>
</div>
```

- css

```css
ul > li:nth-child(2) {
  background-color: red;
}

div > p:nth-child(2n + 1) {
  background-color: orange;
}
```

- 說明 1：第一個選擇器會匹配 li2、li6，使背景為紅色
- 說明 2：第二個選擇器會匹配三個 p，使背景為橘色

### 偽元素(Pseudo Elements)：`selector::pseudo-element`

偽元素則是在選擇器中比較特別的存在，他不像是一般的選擇器改變已有標籤的樣式，反而更像是往匹配的元素的指定位置添加一個元素樣式，可能是提示文字或是模板文字，不過這種情況通常比較難以控制，所以需要小心使用

- 語法

```css
selector::pseudo-element {
}
```

以下列出常用的標準偽元素表：

| 偽元素       | 說明                                               |
| ------------ | -------------------------------------------------- |
| before       | 將偽元素添加到匹配元素之前                         |
| after        | 將偽元素添加到匹配元素之後                         |
| first-letter | 匹配元素的第一個字符，可用於首字符放大             |
| first-letter | 匹配元素的第一行文字                               |
| selection    | 匹配被用戶高亮的部分(就是用滑鼠框起來的部分啦 hhh) |

#### Sample

- html

```html
<div class="subtitle">跳轉頁面一</div>
<a href="">link1</a>
<div class="subtitle">跳轉頁面二</div>
<a href="">link2</a>
```

- css

```css
.subtitle::before {
  content: '- ';
}

a::before {
  content: '鏈結：';
  color: lightgray;
  font-size: 8px;
}
```

- 說明：第一個選擇器會在 `class="subtitle"` 的元素前加上 `-`，而第二個選擇器為每個鏈結加上前導文字

# 結語

以上就是 CSS 選擇器的完整介紹，基本上能夠熟練運用上述的選擇器，99% 的元素特徵任你組合，這些也基本就是選擇器的全部了。或許你會問，阿還有 `@page`、`@media` 啊，那些就是另一種通屬於 `@` 的語法了，將在其他篇詳細解說。
