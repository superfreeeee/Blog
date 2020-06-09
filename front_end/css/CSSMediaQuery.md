# CSS 進階：Media Query 媒體查詢

@[TOC](文章目錄)

<!-- TOC -->

- [CSS 進階：Media Query 媒體查詢](#css-進階media-query-媒體查詢)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Definition 使用語法](#definition-使用語法)
  - [Media Type 媒體類型](#media-type-媒體類型)
  - [Media Feature 媒體特徵](#media-feature-媒體特徵)
    - [視窗規格(Viewport/Page dimension)](#視窗規格viewportpage-dimension)
      - [Sample](#sample)
    - [顯示品質(Display Quality)](#顯示品質display-quality)
    - [色彩(Color)](#色彩color)
    - [交互能力(Interaction)](#交互能力interaction)
  - [邏輯運算符：OR、AND、NOT、ONLY](#邏輯運算符orandnotonly)
    - [OR 或](#or-或)
    - [AND 與](#and-與)
    - [NOT 非](#not-非)
    - [ONLY 只有](#only-只有)
    - [Sample](#sample-1)
- [結語](#結語)

<!-- /TOC -->

## 簡介

21 世紀之後網際網絡迅速發展，各式各樣的硬體設備都開始能夠連上網路、使用瀏覽器訪問網頁。在此場景之下前端網頁的螢幕適配的要求更高，如果還使用傳統甚至寫死的元素內容，會使除了一般電腦螢幕之外的設備閱讀困難，甚至還要考慮到無障礙設計等特殊場景。這樣的需求之下就出現了如 Bootstrap 等響應式(RWD)框架，而其中的實現精髓便是基於`媒體查詢(Media Query)`的技術之上。

## 參考

<table>
  <tr>
    <td>CSS Media Queries 詳細介紹</td>
    <td><a href="https://www.oxxostudio.tw/articles/201810/css-media-queries.html">https://www.oxxostudio.tw/articles/201810/css-media-queries.html</a></td>
  </tr>
  <tr>
    <td>CSS @media Rule-w3school</td>
    <td><a href="https://www.w3schools.com/cssref/css3_pr_mediaquery.asp">https://www.w3schools.com/cssref/css3_pr_mediaquery.asp</a></td>
  </tr>
</table>

# 正文

媒體查詢，顧名思依舊是查詢媒體的特性並套用設定好的對應的樣式表，這邊有一點條件分支的感覺，滿足某某媒體條件的套用哪些形式，語法如下：

## Definition 使用語法

使用上可以直接寫在 HTML 文件內的 `<link>` 標籤上：

```html
<link rel="stylesheet" href="..." media="<condition>" />
```

或是在外部樣式表中作為一個過濾條件，選擇是否引入樣式組：

```css
@media <condition> {
  /* styles */
  body {
    margin: 0;
  }
  div {
    display: block;
  }
  /* blablabla */
}
```

再或是將不同媒體條件的樣式表分離成一個獨立的 CSS 模塊，並在 `@import` 語句導入時過濾：

```css
@import 'index.css' <condition>;
```

其中 `<condition>` 的部分可能是查詢某種`媒體類型(Media Type)`或是某種`媒體特徵(Media Feature)`，然後再搭配`邏輯運算符組(OR,AND,NOT,ONLY)`合成一個表達式

## Media Type 媒體類型

我們可以透過查詢當前解析 HTML 的媒體類型為何，以下列出幾個常用的類型：

| Type   | Description                                |
| ------ | ------------------------------------------ |
| all    | 所有裝置                                   |
| print  | 印刷裝置，以及印刷預覽等                   |
| speech | 朗讀裝置，可以讀出頁面的裝置（無障礙設計） |
| screen | 螢幕裝置                                   |

- Sample

```css
@media screen {
  /* ... */
}
@media print {
  /* ... */
}
```

這樣就能夠根據解析設備的不同套用不同的樣式

## Media Feature 媒體特徵

除了檢驗媒體類型之外，還能夠檢驗許多媒體特徵，可能根據`視窗規格(Viewport/Page dimension)`、`顯示品質(Display Quality)`、`色彩(Color)`、`交互能力(Interaction)`等多種類型，以下分別列出個類型常見的屬性：

### 視窗規格(Viewport/Page dimension)

| Attribute    | Description                                                                      |
| ------------ | -------------------------------------------------------------------------------- |
| width        | 螢幕寬度，`min-width` 表示下限、`max-width` 為上限                               |
| height       | 螢幕高度，支持`min-height`、`max-height`                                         |
| aspect-ratio | 螢幕寬度/高度比（1/1、1680/720），同樣支持`min-aspect-ratio`、`max-aspect-ratio` |
| orientation  | 螢幕方向，兩個可選值：`landscape`為橫向、`portrait`為垂直                        |

#### Sample

```css
@media (max-width: 500px) {
}
@media (orientation: landscape) {
}
```

### 顯示品質(Display Quality)

| Attribute       | Description                                                                                                 |
| --------------- | ----------------------------------------------------------------------------------------------------------- |
| resolution      | 解析度（dpi、ppx），支持`min-resolution`、`max-resolution`                                                  |
| scan            | 電視掃描方式，可選值：`interlace` 交錯式掃描、`progressive` 漸進是掃描                                      |
| update          | 更新方式，可選值：`none`(印刷裝置)、`slow`(嵌入式設備)、`fast`(電腦螢幕)                                    |
| overflow-block  | 塊元素超出邊界的表現，可選值：`none`(隱藏)、`scroll`(滾動條)、`optional-paged`(手動查看)、`paged`(分頁顯示) |
| overflow-inline | 與 `overflow-block` 類似，不過作用於行內元素                                                                |
| grid            | 網格媒體，可選值 0 或 1                                                                                     |

### 色彩(Color)

| Attribute   | Description                                                         |
| ----------- | ------------------------------------------------------------------- |
| color       | 輸出裝置的色彩位元數，支持 `min-color`、`max-color`                 |
| color-index | 輸出裝置的色彩索引位元數，支持 `min-color-index`、`max-color-index` |
| monochrome  | 單色裝置，可選值：0(不是單色裝置)、其他                             |
| color-gamut | 輸出色域，可選值：`srgb`、`p3`、`rec2020`                           |

### 交互能力(Interaction)

| Attribute            | Description                                                                                                  |
| -------------------- | ------------------------------------------------------------------------------------------------------------ |
| pointer、any-pointer | 指標裝置，可選值表示精度：`none`(沒有指標裝置)、`coarse`(低精度，如觸控板)、`fine`(高精度，如滑鼠 or 手寫筆) |
| hover、any-hover     | 裝置具備懸浮(hover)能力，可選值：`none`、`hover`                                                             |

## 邏輯運算符：OR、AND、NOT、ONLY

以上透過指定媒體類型或是特定媒體特徵條件已經能夠建立大部分的條件，接下來就只缺將這些獨立的條件連接在一起的邏輯運算符，有 `OR`(或)、`AND`(與)、`NOT`(非)、`ONLY`(只有) 四種

### OR 或

用於連接可選條件，語法使用 `,` 連接多個條件：

```css
@media screen, (min-width: 1500px) {
}
```

### AND 與

用於連接多個必要條件，語法使用 `and` 連接多個條件：

```css
@media screen and (min-width: 1680px) {
}
```

### NOT 非

用於排除特定條件，語法使用 `not`，為一元運算符：

```css
@media not screen {
}
```

### ONLY 只有

其實這個運算符沒什麼用，有點像是 `NOT` 的相反，其實和默認表現差不多：

```html
<link rel="stylesheet" media="only screen and (color)" href="index.css" />
<!-- 其實兩者效果相同 -->
<link rel="stylesheet" media="screen and (color)" href="index.css" />
```

### Sample

這邊舉出一些複合使用的範例：

```css
@media not screen and (max-width: 500px), print and (orientation: landscape) {
}
```

# 結語

到此就結束啦！沒錯你沒有看錯，就這麼多，其實也沒什麼複雜嘛。然而真正困難的地方在於如何設計你的網頁以及樣式代碼，這邊就牽涉到網頁設計的部分，以確定如何設定媒體查詢的條件以及樣式的組織結構。從技術的層面來說，媒體查詢(Media Query)僅僅作為過濾樣式引入的條件設定罷了。
