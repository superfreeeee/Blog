# CSS 進階：flex 彈性佈局

@[TOC](文章目錄)

<!-- TOC -->

- [CSS 進階：flex 彈性佈局](#css-進階flex-彈性佈局)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Element 元素](#element-元素)
  - [Container 外容器屬性](#container-外容器屬性)
    - [display](#display)
      - [語法](#語法)
    - [flex-direction](#flex-direction)
      - [語法](#語法-1)
      - [Sample](#sample)
    - [flex-wrap](#flex-wrap)
      - [語法](#語法-2)
      - [Sample](#sample-1)
    - [flex-flow](#flex-flow)
      - [語法](#語法-3)
      - [Sample](#sample-2)
    - [justify-content](#justify-content)
      - [語法](#語法-4)
      - [Sample](#sample-3)
    - [align-items](#align-items)
      - [語法](#語法-5)
      - [Sample](#sample-4)
  - [Item 內元素屬性](#item-內元素屬性)
    - [flex-grow](#flex-grow)
      - [語法](#語法-6)
      - [Sample](#sample-5)
    - [flex-shrink](#flex-shrink)
      - [語法](#語法-7)
      - [Sample](#sample-6)
    - [flex-basis](#flex-basis)
      - [語法](#語法-8)
      - [Sample](#sample-7)
    - [flex](#flex)
      - [語法](#語法-9)
      - [Sample](#sample-8)
    - [align-self](#align-self)
      - [語法](#語法-10)
      - [Sample](#sample-9)
    - [order](#order)
      - [語法](#語法-11)
      - [Sample](#sample-10)
- [結語](#結語)

<!-- /TOC -->

## 簡介

我們都知道 html 文件裡面一個個標籤在解析的時候都需要區分 `display` 或是 `position` 以決定它在文檔流的位置，相對於最原始的 `block`、`inline` 等，CSS 提供了更多元的屬性選項，如 `flex`、`grid`、`table` 等。本篇就來簡單介紹一下 `flex` 彈性盒子的一些重要屬性，使頁面中塊元素的對齊方式更加方便簡潔。

## 參考

<table>
    <tr>
        <td>圖解：CSS Flex 屬性一點也不難</td>
        <td><a href="https://wcc723.github.io/css/2017/07/21/css-flex/">https://wcc723.github.io/css/2017/07/21/css-flex/</a></td>
    <tr>
    </tr>
        <td>MDN CSS彈性盒子用法</td>
        <td><a href="https://developer.mozilla.org/zh-TW/docs/Web/CSS/CSS_Flexible_Box_Layout/Using_CSS_flexible_boxes">https://developer.mozilla.org/zh-TW/docs/Web/CSS/CSS_Flexible_Box_Layout/Using_CSS_flexible_boxes</a></td>
    </tr>
</table>

# 正文

## Element 元素

總體來說 flex 佈局的元素可分為 `Container(外容器)`和 `Item(內元素)`。Container 大致上可決定內部元素的定位方式，而 Item 可以決定伸縮性和順序。

## Container 外容器屬性

外容器的屬性總覽：

- display
- flex-flow
  - flex-direction
  - flex-wrap
- justify-content
- align-items

### display

原容器的表現類型

#### 語法

```css
.flex-container {
  display: flex | inline-flex;
}
```

1. `flex`：塊級的彈性盒子
2. `inline-flex`：作為行內塊的彈性盒子，等價於 `inline-block` + `flex`

### flex-direction

決定內元素的排列方向，flex 彈性盒子有兩個主要的軸線：

1. main axis 主軸線：元素排列方向
2. cross axis 交錯軸線：換行方向

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_axis.png)

#### 語法

```css
.flex-container {
  flex-direction: row | row-reverse | column | column-reverse;
}
```

1. `row`：水平正向
2. `row-reverse`：水平反向
3. `column`：垂直正向
4. `column-reverse`：垂直反向

#### Sample

- row：水平正向

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_direction1.png)

- row-reverse：水平反向

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_direction2.png)

- column：垂直正向

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_direction3.png)

- column-reverse：垂直反向

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_direction4.png)

### flex-wrap

決定內元素超過容器時的換行方式，依據 cross-axis 的方向。

#### 語法

```css
.flex-container {
  flex-wrap: no-wrap | wrap | wrap-reverse;
}
```

1. `no-wrap`：不換行
2. `wrap`：換行
3. `wrap-reverse`：反向換行（朝 cross-axis 反向）

#### Sample

- no-wrap：不換行

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_wrap1.png)

- wrap：換行

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_wrap2.png)

- wrap-reverse：換行時反向

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_wrap3.png)

### flex-flow

為 `flex-direction` 和 `flex-wrap` 的組合縮寫，替換 `<>` 內為對應的值即可。

#### 語法

```css
.flex-container {
  flex-flow: <flex-direction> <flex-wrap>;
}
```

#### Sample

```css
.flex-container {
  flex-flow: column no-wrap;
}
```

### justify-content

內元素對主軸線的對齊方式，依據 `flex-direction` 方向可能為水平對齊或垂直對齊。

#### 語法

```css
.flex-container {
  justify-content: flex-start | flex-end | center | space-between | space-around
    | space-evenly;
}
```

1. `flex-start`：向主軸線起始點對齊(main-axis start)
2. `flex-end`：向主軸線結束點對齊(main-axis end)
3. `center`：向主軸線置中對齊
4. `space-between`：元素間空白
5. `space-aroung`：元素左右分別空白
6. `space-evenly`：元素間和軸線上兩側平均空白

#### Sample

- flex-start

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_justify_content1.png)

- flex-end

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_justify_content2.png)

- center

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_justify_content3.png)

- space-between

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_justify_content4.png)

- space-around

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_justify_content5.png)

- space-evenly

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_justify_content6.png)

### align-items

相對於 `justify-content` 是內元素相對於主軸的對齊方式，`align-items` 則是內元素對於交錯軸線(cross-axis) 的對齊方式。

#### 語法

```css
.flex-container {
  align-items: flex-start | flex-end | center | baseline | stretch;
}
```

1. `flex-start`：向交錯軸線起始點對齊(cross-axis start)
2. `flex-end`：向交錯軸線結束點對齊(cross-axis end)
3. `center`：向交錯軸線置中對齊
4. `baseline`：依據文字行對齊
5. `stretch`：伸展到最大

#### Sample

- flex-start

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_align_items1.png)

- flex-end

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_align_items2.png)

- center

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_align_items3.png)

- baseline

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_align_items4.png)

- stretch

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_align_items5.png)

## Item 內元素屬性

內元素的屬性總覽：

- flex
  - flex-grow
  - flex-shrink
  - flex-basis
- align-self
- order

### flex-grow

當容器內還有空間的時候（相對於同一條主線上），表示內元素的伸展性。

#### 語法

```css
.flex-item {
  flex-grow: <number>;
}
```

#### Sample

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_grow1.png)

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_grow2.png)

### flex-shrink

當容器空間不足的時候，表示內元素的收縮性。

#### 語法

```css
.flex-item {
  flex-shrink: <number>;
}
```

#### Sample

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_shrink1.png)

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_shrink2.png)

### flex-basis

表示內元素在主線上分配到的寬度的基準值，在此基礎上再根據 `grow`、`shrink` 伸縮

#### 語法

```css
.flex-item {
  flex-basis: <number>;
}
```

#### Sample

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_basis1.png)

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_basis2.png)

### flex

為 `flex-grow`、`flex-shrink`、`flex-basis` 的縮寫

#### 語法

```css
.flex-item {
  flex: <flex-grow> <flex-shrink> <flex-basis>;
}
```

#### Sample

```css
.flex-item {
  flex: 3 5 30%;
}
```

### align-self

跟容器的 `align-items` 相似，不過 `align-self` 只作用在單個內元素身上。

#### 語法

```css
.flex-item {
  align-self: flex-start | flex-end | center | baseline | stretch;
}
```

與 `align-items` 表現相同

#### Sample

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_align_self1.png)

### order

宣告內元素的顯示優先級

#### 語法

```css
.flex-item {
  order: <number>;
}
```

數值越小的先渲染

#### Sample

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/flex_order1.png)

# 結語

flex 彈性盒子提供了外容器、內元素，透過設置外容器和內元素的屬性可以使內元素依照規則排列、按比例在軸線上自由伸縮，對於響應式頁面是非常有用的屬性。靈活運用 flex 可以避免寫死長度單位造成內容顯示不全或標籤跑位等問題。
