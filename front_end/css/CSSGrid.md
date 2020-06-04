# CSS 進階：Grid 格線佈局

@[TOC](文章目錄)

<!-- TOC -->

- [CSS 進階：Grid 格線佈局](#css-進階grid-格線佈局)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Grid Element 格線佈局元素](#grid-element-格線佈局元素)
  - [Container 外容器](#container-外容器)
    - [1. display](#1-display)
    - [2. grid-template-rows / grid-template-columns](#2-grid-template-rows--grid-template-columns)
      - [Sample](#sample)
      - [repeat & fr](#repeat--fr)
    - [3. grid-template-areas](#3-grid-template-areas)
      - [Sample](#sample-1)
      - [Sample-empty](#sample-empty)
    - [4. grid-template](#4-grid-template)
      - [Sample](#sample-2)
    - [5. grid-auto-rows / grid-auto-columns](#5-grid-auto-rows--grid-auto-columns)
      - [Sample](#sample-3)
    - [6. grid-auto-flow](#6-grid-auto-flow)
      - [Sample](#sample-4)
    - [7. column-gap / row-gap](#7-column-gap--row-gap)
    - [Sample](#sample-5)
    - [8. gap](#8-gap)
      - [Sapmle](#sapmle)
    - [9. justify-items / align-items](#9-justify-items--align-items)
      - [Sample](#sample-6)
    - [10. place-items](#10-place-items)
      - [Sapmle](#sapmle-1)
    - [11. justify-content / align-content](#11-justify-content--align-content)
      - [Sample](#sample-7)
    - [12. place-content](#12-place-content)
      - [Sample](#sample-8)
  - [Item 內元素](#item-內元素)
    - [1. grid-column-start / grid-column-end / grid-row-start / grid-row-end](#1-grid-column-start--grid-column-end--grid-row-start--grid-row-end)
      - [Sample](#sample-9)
    - [2. grid-column / grid-row](#2-grid-column--grid-row)
      - [Sample](#sample-10)
    - [3. grid-area](#3-grid-area)
      - [Sample](#sample-11)
    - [4. justify-self / align-self](#4-justify-self--align-self)
      - [Sample](#sample-12)
    - [5. place-self](#5-place-self)
      - [Sample](#sample-13)
- [結語](#結語)

<!-- /TOC -->

## 簡介

之前我們介紹過了 CSS 的 <a href="https://blog.csdn.net/weixin_44691608/article/details/105980289">flex 彈性盒子</a>。通常使用 flex 的場景多用於需要排列多個元素，並設定位於軸線上的對齊方式等，相較於 flex 較為流動的排版樣式，grid 的排版則顯得稍微固定一點。grid 的概念是先將外容器劃分隔線(rows/columns)，將一塊區域劃分成類似表格的形狀，再將各區域(area)分配給不同的內元素，我們也可以說是由內元素來指定想要佔有的格線區域。如此一來作爲整體頁面的佈局形式，能使的內容顯得更有章法，例如一種經典的 960gs 格線系統（查看參考三的鏈結），使用 grid 配合媒體查詢(@media)，將作為響應式網頁的開發是非常有用的一個屬性。

## 參考

<table>
  <tr>
    <td>CSS Grid 屬性介紹</td>
    <td><a href="https://wcc723.github.io/css/2017/03/22/css-grid-layout/">https://wcc723.github.io/css/2017/03/22/css-grid-layout/</a></td>
  </tr>
  <tr>
    <td>A Complete Guide to Grid</td>
    <td><a href="https://css-tricks.com/snippets/css/complete-guide-grid/">https://css-tricks.com/snippets/css/complete-guide-grid/</a></td>
  </tr>
  <tr>
    <td>鐵人賽：網頁設計常用格線系統(上)</td>
    <td><a href="https://wcc723.github.io/design/2018/10/18/grid-system/">https://wcc723.github.io/design/2018/10/18/grid-system/</a></td>
  </tr>
</table>

# 正文

## Grid Element 格線佈局元素

與 flex 佈局相似，grid 佈局也分成外容器(Container)和內元素(Item)，接下來將分別介紹兩者的屬性。

## Container 外容器

外容器可以看做是畫表格的底版，可以設定以下屬性：

1. `display`：容器類型
2. `grid-template-rows` / `grid-template-columns`：格線分佈模板(顯式聲明)
3. `grid-template-areas`：格線區域模板
4. `grid-template`：grid-template-rows、grid-template-columns、grid-template-areas 合併的縮寫，建議還是分開寫
5. `grid-auto-rows` / `grid-auto-columns`：格線分佈模板(隱式聲明，可超線)
6. `grid-auto-flow`：內元素自動填充的方向
7. `column-gap` / `row-gap`：格線間空格(gutter)
8. `gap`：column-gap、row-gap 合併的縮寫
9. `justify-items` / `align-items`：內元素對齊
10. `place-items`：justify-items、align-items 合併的縮寫
11. `justify-content` / `align-content`：整個表格相對於外容器對齊
12. `place-content`：justify-content、align-content 合併的縮寫

### 1. display

外容器的顯示類型，語法如下：

```css
.container {
  display: grid | inline-grid;
}
```

- `grid`：格線佈局
- `inline-gird`：行內格線佈局

很好理解，直接進入下一項

### 2. grid-template-rows / grid-template-columns

這兩個屬性聲明了格線的模板，聲明每條線的名字和各區域所佔的寬度

```css
.container {
  grid-template-rows: <track-size> ... | [<line-name>] <track-size> ...;
  grid-template-columns: <track-size> ... | [<line-name>] <track-size> ...;
}
```

- `<track-size>`：表示格線間距的寬度
- `<line-name>`：表示格線名（用於 start / end），需要用方括號 `[]` 與 `<track-size>` 區隔開來

#### Sample

```css
.container {
  grid-template-columns: 200px 50px auto 50px 200px;
  grid-template-rows: 150px auto 150px;
}
```

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_template_columns_rows.png)

#### repeat & fr

有時候我們可能會需要固定寬度或是固定比例，這時候就可以使用 repeat 跟 fr

```css
.container {
  grid-template-columns: repeat(2, 1fr 2fr) 300px;
}
/* 等價於 */
.container {
  grid-template-columns: 1fr 2fr 1fr 2fr 300px;
}
```

這時候就會先賦予最右邊一列 300px 的寬度，前四列則分別是剩餘寬度的 1/6、2/6、1/6、2/6，如下圖所示

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_template_repeat.png)

### 3. grid-template-areas

有了格線之後，就可以來劃分區域了。grid 的規則限定，同名的區域必須處於相鄰的位置(不可分割且唯一的)，否則不生效，語法如下

```css
.container {
  grid-template-areas:
    '<area-name> ...'
    '<area-name> ...'
    ...;
}
```

每個字符串表示一行，其中的 `<area-name>` 表示此格聲明的區域名

- `<area-name>` 除了使用一般字符串表示名字，還可以是 `. (未分配)`、`none (塊不存在)`

#### Sample

```css
.container {
  grid-template-areas:
    'header header header header header'
    'aside main main main main'
    'aside footer footer footer footer';
}
```

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_template_areas.png)

#### Sample-empty

```css
.container {
  grid-template-areas:
    'header header header header header'
    'aside . . main main'
    'aside footer footer footer .';
}
```

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_template_areas_empty.png)

### 4. grid-template

為 grid-template-rows、grid-template-columns、grid-template-areas 合併的縮寫，語法如下：

```css
.container {
  grid-template: none | <grid-template-rows> / <grid-template-columns>;
}
```

#### Sample

```css
.container {
  grid-template:
    [row1-start] 'header header header header header' 150px [row1-end]
    [row2-start] 'aside main main main main' auto [row2-end]
    [row3-start] 'aside footer footer footer footer' 150px [row3-end]
    / 200px 50px auto 50px 200px;
}
/* 等價於 */
.container {
  grid-template-columns: 200px 50px auto 50px 200px;
  grid-template-rows: [row1-start] 150px [row1-end row2-start] auto [row2-end row3-start] 150px [row3-end];
  grid-template-areas:
    'header header header header header'
    'aside main main main main'
    'aside footer footer footer footer';
}
```

### 5. grid-auto-rows / grid-auto-columns

上面提到的 grid-template 是顯式的聲明所有格線，但如果想要隱式的聲明固定樣式，並讓格線數量自由改變的話，就要使用 grid-auto 來聲明模板，並透過 grid-row、grid-column 來指定實際擺放的位置。

不過從網頁設計的角度來說，多數時候需要固定格數，比較少用到這種超出格線的問題，所以實際使用上多數還是以 template 為主。

語法如下：

```css
.container {
  grid-auto-columns: <track-size> ...;
  grid-auto-rows: <track-size> ...;
}
```

#### Sample

```css
.container {
  grid-template-columns: repeat(3, 200px);
  grid-template-rows: repeat(3, 160px);
  grid-auto-columns: 100px;
  grid-auto-rows: 150px;
}

.item1 {
  grid-row: 4 / 5;
  grid-column: 5 / 6;
}
```

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_auto_columns_rows.png)

`grid-row` 與 `grid-column` 屬性後續會介紹

### 6. grid-auto-flow

這個屬性指定了當內元素未指定區域時自動填充的方向，語法如下

```css
.container {
  grid-auto-flow: row | column | row dense | column dense;
}
```

- `row`：表示優先填充行
- `column`：表示優先填充列
- `dense`：表示後續元素能自動填充到前面的空隙，有可能會破壞元素定義順序和提高可理解性成本，須謹慎使用

#### Sample

- row

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_auto_flow_row.png)

- column

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_auto_flow_column.png)

### 7. column-gap / row-gap

這個屬性可以分別指定列之間(columns)和行之間(rows)的間隙(gap 也稱為 gutter)

```css
.container {
  column-gap: <line-size>;
  row-gap: <line-size>;
}
```

- `<line-size>`：為間隙的長度

### Sample

```css
.container {
  column-gap: 15px;
  row-gap: 30px;
}
```

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_gap.png)

### 8. gap

`gap` 為 `column-gap`、`row-gap` 合併後的縮寫，語法如下

```css
.container {
  gap: <row-gap> <column-gap>;
}
```

- 注意：這裡值得一提的是，`gap`、`column-gap`、`row-gap` 都是後來修改的提案，如果瀏覽器版本不支援的話可以加上 `grid-` 前綴來適用舊的屬性名

#### Sapmle

```css
.container {
  gap: 30px 15px;
}
/* 等價於 */
.container {
  column-gap: 15px;
  row-gap: 30px;
}
```

### 9. justify-items / align-items

這邊就類似於 flex 佈局的屬性命名啦，`items` 指所有內元素，而 `justify`、`align` 分別指主軸和交錯軸的方向（水平和垂直），語法如下：

```css
.container {
  justify-items: start | end | center | stretch;
  align-items: start | end | center | stretch;
}
```

- `start`：對齊區域的軸線起始點
- `end`：對齊區域的軸線結束點
- `center`：對齊區域的軸線中央
- `stretch`：伸展直到填滿整條軸線

#### Sample

- jusfity-items：start

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_justify_items1.png)

- jusfity-items：end

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_justify_items2.png)

- jusfity-items：center

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_justify_items3.png)

- jusfity-items：stretch

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_justify_items4.png)

- align-items：start

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_align_items1.png)

- align-items：end

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_align_items2.png)

- align-items：center

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_align_items3.png)

- align-items：stretch

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_align_items4.png)

### 10. place-items

看名字很容易猜到，就是 `justify-items`、`align-items` 的縮寫

```css
.container {
  place-items: <align-items> <justify-items>;
}
```

#### Sapmle

```css
.container {
  place-items: start center;
}
/* 等價於 */
.container {
  align-items: start;
  justify-items: center;
}
```

### 11. justify-content / align-content

相對於 `items` 指所有內元素，`content` 則是指內元素的內容分布，在這裡指的就是列或是行（水平或垂直），`justify`、`align` 一樣指主軸和交錯軸的方向（水平和垂直），語法如下：

```css
.container {
  justify-content: start | end | center | stretch | space-around | space-between | space-evenly;
  align-content: start | end | center | stretch | space-around | space-between | space-evenly;
}
```

- `start`：對齊外容器的軸線起始點
- `end`：對齊外容器的軸線結束點
- `center`：對齊外容器的軸線中央
- `stretch`：伸展直到填滿外容器的軸線
- `space-around`：各區域左右各自插入空隙
- `space-between`：各區域之間插入空隙
- `space-evenly`：各區域之間及與外容器間均分空隙

#### Sample

- justify-content：start

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_justify_content1.png)

- justify-content：end

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_justify_content2.png)

- justify-content：center

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_justify_content3.png)

- justify-content：stretch

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_justify_content4.png)

- justify-content：space-around

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_justify_content5.png)

- justify-content：space-between

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_justify_content6.png)

- justify-content：space-evenly

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_justify_content7.png)

- align-content：start

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_align_content1.png)

- align-content：end

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_align_content2.png)

- align-content：center

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_align_content3.png)

- align-content：stretch

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_align_content4.png)

- align-content：space-around

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_align_content5.png)

- align-content：space-between

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_align_content6.png)

- align-content：space-evenly

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_align_content7.png)

### 12. place-content

與 `place-items` 相似，它是 `justify-content`、`align-content` 的縮寫，語法如下：

```css
.container {
  place-content: <align-content> <justify-content>;
}
```

#### Sample

```css
.container {
  place-content: space-between space-evenly;
}
/* 等價於 */
.container {
  justify-content: space-between;
  align-content: space-evenly;
}
```

## Item 內元素

有了格線佈局的外容器，我們已經定義好格線和區塊了，接下來就是定義內元素所佔用的區塊，以及各自的對齊方式，內元素可以設定的屬性如下：

1. `grid-column-start` / `grid-column-end` / `grid-row-start` / `grid-row-end`：指定內元素行與列的開始和結束線段
2. `grid-column` / `grid-row`：為 `start`、`end` 的縮寫
3. `grid-area`：為 `grid-column`、`grid-row` 的縮寫
4. `justify-self` / `align-self`：自定義對齊（覆蓋外元素定義的 `items` 屬性）
5. `place-self`：為 `justify-self`、`align-self` 的縮寫

### 1. grid-column-start / grid-column-end / grid-row-start / grid-row-end

分別定義內元素的 `起始列格線(column-start)`、`結束列格線(column-end)`、`起始行(row-start)`、`結束行格線(row-end)`，語法如下：

```css
.item {
  grid-column-start: <number> | <line-name> | span <number> | span <line-name> | auto;
  grid-column-end: <number> | <line-name> | span <number> | span <line-name> | auto;
  grid-row-start: <number> | <line-name> | span <number> | span <line-name> | auto;
  grid-row-end: <number> | <line-name> | span <number> | span <line-name> | auto;
}
```

- `<number>`：格線的序號（從 1 開始，反向從 -1 開始）
- `<line-name>`：格線的名稱，在 template 裡面定義的
- `span <number>`：跨越固定數量的行或列格線
- `span <line-name>`：跨越直到遇到指定名稱的格線
- `auto`：自動填入（方向由外容器的 `grid-auto-flow` 決定），通常一個元素佔用一格

#### Sample

```css
.item {
  grid-row-start: 2;
  grid-row-end: 4;
  grid-column-start: 3;
  grid-column-end: 5;
}
/* 等價於 */
.item {
  grid-row: 2 / 4;
  grid-column: 3 / 5;
}
```

例圖放到下一個屬性說明

### 2. grid-column / grid-row

上個屬性僅僅為了聲明頭尾就分成了四個屬性有點麻煩，這兩個屬性則是上面四個屬性的縮寫，中間用 `/` 隔開，語法如下：

```css
.item {
  grid-column: <start-line> / <end-line> | <start-line> span <value>;
  grid-row: <start-line> / <end-line> | <start-line> span <value>;
}
```

- `<start-line>`：開始的格線，序號或名字
- `<end-line>`：結束的格線
- `span <value>`：伸展的格線數或目標格線名

#### Sample

```css
.item {
  grid-row: 2 / 4;
  grid-column: 3 / 5;
}
```

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_column_row.png)

### 3. grid-area

這個屬性則是將 `grid-column`、`grid-row` 合併後再簡化，語法如下：

```css
.item {
  grid-area: <area-name> | <row-start> / <column-start> / <row-end> / <column-end>;
}
```

- `<area-name>`：區塊名，在 template 中定義的
- `<row-start>`：行的起始格線
- `<column-start>`：列的起始格線
- `<row-end>`：行的結束格線
- `<column-end>`：列的結束格線

**注意！**由於四個屬性的順序容易搞混且可讀性不高，所以建議除非使用 `<area-name>`，不然還是使用 `grid-column`、`grid-row` 比較好

#### Sample

```css
.item {
  grid-area: 2 / 3 / 4 / 5; /* 看不懂在幹嘛吧hhh */
}
/* 等價於 */
.item {
  grid-row: 2 / 4;
  grid-column: 3 / 5;
}
```

### 4. justify-self / align-self

外容器可以透過定義 `justify-items`、`align-items` 來定義內元素在區塊內的對齊方式，而內元素可以透過定義 `justify-self`、`align-self` 來覆蓋這個屬性，用於處理特例，語法如下：

```css
.item {
  justify-self: start | end | center | stretch;
  align-self: start | end | center | stretch;
}
```

可選值說明與 `items` 相同，就不贅述

#### Sample

- justify-self：start

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_justify_self1.png)

- justify-self：end

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_justify_self2.png)

- justify-self：center

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_justify_self3.png)

- justify-self：stretch

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_justify_self4.png)

- align-self：start

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_align_self1.png)

- align-self：end

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_align_self2.png)

- align-self：center

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_align_self3.png)

- align-self：stretch

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/grid_align_self4.png)

### 5. place-self

這個屬性是 `justify-self`、`align-self` 合併後的縮寫，語法如下：

```css
.item {
  place-self: <align-self> <justify-self>;
}
```

#### Sample

```css
.item {
  place-self: center center;
}
/* 等價於 */
.item {
  justify-self: center;
  align-self: center;
}
```

# 結語

以上就是 grid 的所有可用屬性，在視需求設定好外容器和內元素格線定義後，你的網頁想不整齊都難！下一篇會來介紹媒體查詢相關的技術，透過格線佈局和媒體查詢組合使用，就可以讓你的網頁漂漂亮亮又不用一直修 css。grid 也是許多 UI 框架的格線佈局的基礎，就算不直接操作修改相關屬性，理解 grid 的概念和屬性含義有助於更快上手其他 UI 框架的組件。
