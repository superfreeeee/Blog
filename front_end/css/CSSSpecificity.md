# CSS 基礎：Specificity 權重

@[TOC](文章目錄)

<!-- TOC -->

- [CSS 基礎：Specificity 權重](#css-基礎specificity-權重)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Type 類型](#type-類型)
    - [引入方法](#引入方法)
    - [選擇器類型](#選擇器類型)
  - [權重計算](#權重計算)
    - [Level 5](#level-5)
    - [Level 4](#level-4)
    - [Level 3](#level-3)
    - [Level 2](#level-2)
    - [Level 1](#level-1)
    - [`!important`](#important)
    - [Sample](#sample)
- [結語](#結語)

<!-- /TOC -->

## 簡介

前面我已經寫過一個<a href="https://blog.csdn.net/weixin_44691608/article/details/106603985">CSS 基礎：Selector 選擇器</a>，介紹了 CSS 裡面最重要的選擇器，接下來本篇將要介紹 CSS 是如何計算選擇器的權重。

有時候你寫 CSS 的時候會發現，我寫的樣式怎麼沒有生效？原來是同時存在多個同樣的定義，那為什麼是另一個覆蓋這一個呢？瀏覽器是如何決定套用哪個樣式呢？其實瀏覽器在解析 CSS 樣式表的時候會為每個樣式前的選擇器計算一個權重，如果同個元素出現相同的屬性定義，就會比較權重，套用權重較高的一方的樣式；而當權重相同的時候，後面聲明的樣式將會覆蓋前面的樣式。接下來就讓我們實際來看看 CSS 的權重是如何計算的吧。

## 參考

<table>
  <tr>
    <td>Day20：小事之 CSS 權重 (css specificity)</td>
    <td><a href="https://ithelp.ithome.com.tw/articles/10196454">https://ithelp.ithome.com.tw/articles/10196454</a></td>
  </tr>
  <tr>
    <td>优先级</td>
    <td><a href="https://developer.mozilla.org/zh-CN/docs/Web/CSS/Specificity">https://developer.mozilla.org/zh-CN/docs/Web/CSS/Specificity</a></td>
  </tr>
</table>

# 正文

## Type 類型

我們可以依據引入方法和選擇器類型進行兩種分級

### 引入方法

CSS 引入的方法可以分成三種：

1. 內聯樣式(inline)：寫在標籤的 `style` 屬性內

```html
<div style="display: block;"></div>
```

2. 內部樣式(internal)：寫在 HTML 文件的 `<style>` 標籤內

```html
<div></div>

<style>
  div {
    display: block;
  }
</style>
```

3. 外部樣式(external)：寫在外部 `.css` 文件，並透過 `<link>` 標籤引入

```html
<!-- index.html -->
<link rel="stylesheet" href="index.css"></link>
```

```css
/* index.css */
div {
  display: block;
}
```

三種樣式的優先級順序是：`內聯(inline)` > `內部(internal)` > `外部(external)`

不過都 2020 年了，應該也很少人使用內部樣式表來編寫 css，因此我們可以將所有樣式表粗略的分為內聯和外部樣式表兩種

### 選擇器類型

這邊使用的選擇器類型請參考我的前一篇<a href="https://blog.csdn.net/weixin_44691608/article/details/106603985">CSS 基礎：Selector 選擇器</a>

在以引入方式計算優先級之前，會先比較由選擇器類型決定的權重(specificity)。請原諒我這邊決定使用`權重(specificity)`，因為比起`優先級(priority)`通常只有一個值來比大小，CSS 權重的計算更像是計算各部位的特徵做加總後，最後得到的一個總和。

選擇器類型的權重計算有四個等級加上一個特例(`!important`)：

0. `!important`
1. 內聯樣式(inline)(`style="..."`)
2. ID 選擇器(`#idName`)
3. class 選擇器(`.className`)、屬性選擇器(`[attr="value"]`)、偽類(`:pseudo-class`)
4. 元素選擇器(`tagName`)、偽元素(`::pseudo-element`)

以下我們使用這樣的形式來表示計算後的權重，四個值分別表示四個等級的選擇器類型：

```
0.0.0.0
高 -> 低
```

## 權重計算

這邊主要計算上面提到的選擇器類型的權重計算，注意 `>`、`~`、`+`、`` 等運算符是不影響權重計算的。

### Level 5

在介紹四個等級的權重值之前，我們先來考慮全局樣式（也就是使用通配符 `*`）。由於通配符的目的在於匹配任何元素，所以他的權重需要比任何樣式都低（也就是全部為 0）：

```
0.0.0.0
```

### Level 4

- 選擇器類型

第四級的優先級有兩種選擇器：

1. 元素選擇器(`tagName`)
2. 偽元素(`::pseudo-element`)

- 權重

```
0.0.0.1
```

### Level 3

- 選擇器類型

第三級的優先級有三種選擇器：

1. class 選擇器(`.className`)
2. 屬性選擇器(`[attr="value"]`)
3. 偽類(`:pseudo-class`)

- 權重

```
0.0.1.0
```

### Level 2

- 選擇器類型

第二級的優先級只有一種選擇器：

1. ID 選擇器(`#idName`)

- 權重

```
0.1.0.0
```

### Level 1

- 選擇器類型

這邊我們也把內聯樣式單獨劃為一個等級：

1. 內聯樣式(inline)(`style="..."`)

- 權重

```
1.0.0.0
```

### `!important`

這個類型屬於比較特殊的等級，他擁有最高的優先級，會覆蓋所有其他四種等級的選擇器類型，用法如下：

```css
div {
  display: block !important;
}
```

但是在此基礎之上還是能夠比較其他四個等級的優先級，因此我們可以把它的權重看成：

```
1.0.0.0.0
```

**注意！**總是使用 `!important` 來覆蓋優先級是強烈不推薦的，相反的應該優先使用更具體的聲明；也不建議在自定義的組件庫或全局樣式使用 `!important`，因為這樣往後的使用者想要覆蓋屬性的時候也必須加上 `!important`，將會強烈破壞樣式聲明結構和可讀性。

### Sample

其實很簡單吧，簡單來說越具體（同一個樣式指定越多限制），權重當然就會越大。以下舉一些選擇器的權重計算的例子：

```css
ul > li {
}
/* 2 個 level4，權重為 0.0.0.2 */

body div ul li a span {
}
/* 6 個 leve4，權重為 0.0.0.6 */

li.myclass {
}
/* level4 * 1 + level3 * 1 = 0.0.1.1 */

li.myclass ~ li {
}
/* level4 * 2 + leve3 * 1 = 0.0.1.2 */
/* 兩個 element 加上一個 class ，所以是 0-0-1-2 */

form input[type='email'] {
}
/* leve4 * 2 + level3 * 1 = 0.0.1.2 */

#app > .layout > .main .title > p::first-letter {
  font-size: 30px !important;
}
/* level4 * 2 + level3 * 3 + level2 * 1 + !important = 1.0.1.3.2 */
```

就這樣，沒了，真的沒了。

# 結語

其實 CSS 權重的計算真的沒有很複雜，有時候你可以在引用 UI 組件庫的時候為了調樣式調到頭破血流。這時候你可能要反思一下是 UI 組件庫本身限制了一些屬性的使用，還是你的 CSS 使用方法或是觀念出現問題。Debug 的時候從來都是先重現、定位問題再針對性地解決，尤其是在 CSS 這種並不那麼明顯的表現出生效的樣式（當然有可能有些工具提供了支持）。**切記！**絕對不可以盲目的調試 CSS 代碼，看起來沒事了就當沒問題了，這樣極有可能在不同瀏覽器、不同解析度、甚至僅僅只是不同的縮放情況下出現樣式大幅度跑位的嚴重問題！
