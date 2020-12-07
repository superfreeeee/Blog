# JS 实战: Drag 点击拖曳效果

@[TOC](文章目录)

<!-- TOC -->

- [JS 实战: Drag 点击拖曳效果](#js-实战-drag-点击拖曳效果)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [项目结构 & 静态模版](#项目结构--静态模版)
    - [添加元素](#添加元素)
    - [添加 position](#添加-position)
  - [主要逻辑片段](#主要逻辑片段)
    - [事件响应结构](#事件响应结构)
    - [移动元素](#移动元素)
    - [限制可移动范围](#限制可移动范围)
    - [最终版本](#最终版本)
- [结语](#结语)

<!-- /TOC -->

## 简介

一直以来都觉得网页中的点击拖曳效果很酷，本篇就来尝试看看实现使用原生 JS 来实现点击拖曳元素的效果。

## 参考

<table>
  <tr>
    <td>DIV 点击拖动，纯JS 实现</td>
    <td><a href="https://blog.csdn.net/qq_32214375/article/details/82387482">https://blog.csdn.net/qq_32214375/article/details/82387482</a></td>
  </tr>
  <tr>
    <td>鼠标事件以及clientX、offsetX、screenX、pageX、x的区别</td>
    <td><a href="https://blog.csdn.net/weixin_41342585/article/details/80659736">https://blog.csdn.net/weixin_41342585/article/details/80659736</a></td>
  </tr>
  <tr>
    <td>Window.getComputedStyle()-MDN</td>
    <td><a href="https://developer.mozilla.org/zh-CN/docs/Web/API/Window/getComputedStyle">https://developer.mozilla.org/zh-CN/docs/Web/API/Window/getComputedStyle</a></td>
  </tr>
  <tr>
    <td>js原生获取元素的css属性</td>
    <td><a href="https://www.cnblogs.com/leaf930814/p/6985017.html?utm_source=itdadao&utm_medium=referral">https://www.cnblogs.com/leaf930814/p/6985017.html?utm_source=itdadao&utm_medium=referral</a></td>
  </tr>
  <tr>
    <td>CSS z-index 属性</td>
    <td><a href="https://www.w3school.com.cn/cssref/pr_pos_z-index.asp">https://www.w3school.com.cn/cssref/pr_pos_z-index.asp</a></td>
  </tr>
</table>

# 正文

## 项目结构 & 静态模版

在开始编写 js 程序之前，我们先来创建一个接下来要操作的元素模版，同时给出我们的项目结构：

```
/js_drag
|- index.html
|- index.css
|- index.js
```

### 添加元素

首先我们先定义好 html 和 css 文件的内容：

- `index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <link rel="stylesheet" href="index.css">
</head>
<body>
    <div class="container">
        <div id="box1" class="box"></div>
        <div id="box2" class="box"></div>
        <div id="box3" class="box"></div>
        <div id="box4" class="box"></div>
    </div>
</body>
</html>
```

- `index.css`

```css
body {
    margin: 0;
    background-color: gray;
}

.container {
    background-color: #fff;
    margin: 40px auto 0;
    width: 800px;
    height: 800px;
}

.box {
    width: 100px;
    height: 100px;
}

#box1 { background-color: blue; }
#box2 { background-color: red; }
#box3 { background-color: green; }
#box4 { background-color: lightgray; }
```

效果如下：

![](https://picures.oss-cn-beijing.aliyuncs.com/img/drag_element_1.png)

### 添加 position

然而这样的元素其实是不能拖曳的，我们将 `.container` 元素作为我们拖动的背景，而里面的每个 `.box` 作为我们想要拖动的元素，就需要使用 `position` 属性的 `relative` 和 `absolute` 值，使得 box 相对于 container 进行移动

- `index.css`

```css
body {
    margin: 0;
    background-color: gray;
}

.container {
    background-color: #fff;
    margin: 40px auto 0;
    width: 800px;
    height: 800px;
    position: relative;
}

.box {
    width: 100px;
    height: 100px;
}

#box1 {
    background-color: blue;
    position: absolute;
    left: 200px;
    top: 100px;
}

#box2 {
    background-color: red;
    position: absolute;
    top: 200px;
    right: 100px;
}

#box3 {
    background-color: green;
    position: absolute;
    right: 200px;
    bottom: 100px;
}

#box4 {
    background-color: lightgray;
    position: absolute;
    bottom: 200px;
    left: 100px;
}
```

效果如下：

![](https://picures.oss-cn-beijing.aliyuncs.com/img/drag_element_2.png)

## 主要逻辑片段

完成好我们的静态元素模版后，我们就可以开始来编写拖曳元素的逻辑代码了

### 事件响应结构

首先我们先建立整个拖曳事件的状态转移图：

![](https://picures.oss-cn-beijing.aliyuncs.com/img/drag_event_state.png)

我们判断鼠标落下(mousedown)时记录当前鼠标坐标(clientX、clientY)，若在放开前进行拖曳则根据鼠标偏移量改变元素位置，放开时则取消对移动事件的监听，我们首先给出事件响应结构的代码：

- `index.html`


```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <link rel="stylesheet" href="index.css">
</head>
<body>
    <div class="container">
        <!-- 在 mousedown 事件绑定拖曳方法，传入 event 和 id 作为参数 -->
        <div onmousedown="dragStart(event, this.id)" id="box1" class="box"></div>
        <div onmousedown="dragStart(event, this.id)" id="box2" class="box"></div>
        <div onmousedown="dragStart(event, this.id)" id="box3" class="box"></div>
        <div onmousedown="dragStart(event, this.id)" id="box4" class="box"></div>
    </div>

    <script src="index.js"></script>
</body>
</html>
```

- `index.js`

```js
function dragStart (e, id) {
  // mousedown 事件，拖曳开始
  console.log('mouse down -> drag start')
  const el = document.getElementById(id)
  console.log(this)
  console.log(el)
  
  const getMove = function (e) {
    // mousemove 事件，表示拖曳移动阶段
    console.log('mouse move')
  }
  this.addEventListener('mousemove', getMove)
  this.onmouseup = function (e) {
    // mouseup 事件，表示结束拖曳，移除 mousemove 的监听函数
    console.log('mouse up -> drag over')
    this.removeEventListener('mousemove', getMove)
  }
}
```

点击、移动、并放开鼠标的示例如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/drag_element_2.png)

### 移动元素

接下来，为了要移动元素，我们首先要先拿到元素原本的位置（由于使用 `position` 属性，所以需要拿到 `top`、`left` 的值）

> `window.getComputedStyle`

这边需要注意的一点在于，当我们直接使用 `el.style.attr` 只能拿到透过 `style` 设置的值。为了拿到一开始的初始值我们需要使用 `window.getComputedStyle` 方法来动态的取值，为此我们需要抽象出 `getStyleVal` 方法来封装 `getComputedStyle` 方法：

- `getComputedStyle`

```js
function getStyleVal (el, attr) {
  const s = getComputedStyle(el)[attr]
  return Number(s.substring(0, s.indexOf('p')))
}
```

同时我们在每次的 `mousemove` 事件拿到鼠标的新位置并计算偏移量之后，改变元素的位置：

- `index.js`

```js
// 获取目标元素的目标值（封装 getComputedStyle 方法）
function getStyleVal (el, attr) {
  const s = getComputedStyle(el)[attr]
  return Number(s.substring(0, s.indexOf('p')))
}

function dragStart (e, id) {
  console.log('mouse down -> drag start')
  const el = document.getElementById(id)
  // 获取当前位置
  const [top, left] = [getStyleVal(el, 'top'), getStyleVal(el, 'left')]
  // 获取鼠标落下时的坐标
  const [cursorX, cursorY] = [e.clientX, e.clientY]

  const getMove = function (e) {
    console.log('mouse move')
    // 鼠标新坐标
    const [x, y] = [e.clientX, e.clientY]
    // 鼠标相对偏移量
    const [offsetX, offsetY] = [x - cursorX, y - cursorY]
    // 元素新位置
    let [nextLeft, nextTop] = [left + offsetX, top + offsetY]
    // 移动元素
    el.style.left = `${nextLeft}px`
    el.style.top = `${nextTop}px`
  }
  this.addEventListener('mousemove', getMove)
  this.onmouseup = function (e) {
    console.log('mouse up -> drag over')
    this.removeEventListener('mousemove', getMove)
  }
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/drag_sample_1.gif)

### 限制可移动范围

我们可以看到，上面的例子已经可以自由的拖动元素了，但是却飞出了白色的范围。

接下来我们需要限制元素可以移动的范围，避免里面的 box 拖一拖飞出 container 的范围，首先我们将获取可移动范围封装成一个方法：

```js
function maxOffset (el) {
  const p = el.parentNode
  const outerW = getStyleVal(p, 'width')
  const outerH = getStyleVal(p, 'height')
  const innerW = getStyleVal(el, 'width')
  const innerH = getStyleVal(el, 'height')
  // return [maxW, maxH]
  return [outerW - innerW, outerH - innerH]
}
```

并在 mousemove 事件中限制元素的目标位置

- `index.js`

```js
function getStyleVal (el, attr) {
  const s = getComputedStyle(el)[attr]
  return Number(s.substring(0, s.indexOf('p')))
}

function maxOffset (el) {
  const p = el.parentNode
  const outerW = getStyleVal(p, 'width')
  const outerH = getStyleVal(p, 'height')
  const innerW = getStyleVal(el, 'width')
  const innerH = getStyleVal(el, 'height')
  return [outerW - innerW, outerH - innerH]
}

function dragStart (e, id) {
  console.log('mouse down -> drag start')
  const el = document.getElementById(id)

  const [top, left] = [getStyleVal(el, 'top'), getStyleVal(el, 'left')]
  const [cursorX, cursorY] = [e.clientX, e.clientY]
  const [maxW, maxH] = maxOffset(el)

  const getMove = function (e) {
    console.log('mouse move')
    const [x, y] = [e.clientX, e.clientY]
    const [offsetX, offsetY] = [x - cursorX, y - cursorY]
    let [nextLeft, nextTop] = [left + offsetX, top + offsetY]
    // left 和 top 必须分别限制在 [0, maxW], [0, maxH] 的范围之内
    nextLeft = nextLeft < 0 ? 0 : nextLeft > maxW ? maxW : nextLeft
    nextTop = nextTop < 0 ? 0 : nextTop > maxH ? maxH : nextTop
    el.style.left = `${nextLeft}px`
    el.style.top = `${nextTop}px`
  }
  this.addEventListener('mousemove', getMove)
  this.onmouseup = function (e) {
    console.log('mouse up -> drag over')
    this.removeEventListener('mousemove', getMove)
  }
}
```

效果如下：

![](https://picures.oss-cn-beijing.aliyuncs.com/img/drag_sample_2.gif)

### 最终版本

最后我们加上一个小功能，每次点击元素时将其提到最上方（使用 `z-index` 属性实现），代码如下：

- `index.js`

```js
function getStyleVal (el, attr) {
  const s = getComputedStyle(el)[attr]
  return Number(s.substring(0, s.indexOf('p')))
}

function maxOffset (el) {
  const p = el.parentNode
  const outerW = getStyleVal(p, 'width')
  const outerH = getStyleVal(p, 'height')
  const innerW = getStyleVal(el, 'width')
  const innerH = getStyleVal(el, 'height')
  return [outerW - innerW, outerH - innerH]
}

const data = {
  zIndex: 0
}

function dragStart (e, id) {
  const el = document.getElementById(id)
  const [top, left] = [getStyleVal(el, 'top'), getStyleVal(el, 'left')]
  const [cursorX, cursorY] = [e.clientX, e.clientY]
  const [maxW, maxH] = maxOffset(el)

  // 每次 mousedown 将该元素的 zIndex 设为全局最大
  el.style.zIndex = ++data.zIndex
  
  const getMove = function (e) {
    const [x, y] = [e.clientX, e.clientY]
    const [offsetX, offsetY] = [x - cursorX, y - cursorY]
    let [nextTop, nextLeft] = [top + offsetY, left + offsetX]
    nextLeft = nextLeft < 0 ? 0 : nextLeft > maxW ? maxW : nextLeft
    nextTop = nextTop < 0 ? 0 : nextTop > maxH ? maxH : nextTop
    el.style.left = `${nextLeft}px`
    el.style.top = `${nextTop}px`
  }
  this.addEventListener('mousemove', getMove)
  this.onmouseup = function (e) {
    this.removeEventListener('mousemove', getMove)
  }
}
```

最终版本的效果如下：

![](https://picures.oss-cn-beijing.aliyuncs.com/img/drag_sample_3.gif)

# 结语

其实很简单吧，核心功能实现在于：根据鼠标偏移量改变元素当前位置，使用到的相关属性和方法注意点如下：

- 监听 `mousedown`、`mousemove`、`mouseup` 实现状态的轮转
- 使用 `getComputedStyle` 动态获取 css 属性值
- `clientX`、`clientY` 表示相对于可视区域的偏移量（参考链接二）
- 直接设置 `el.style.attr` 改变元素样式

本篇就到此为止啦（终于写出来了hh）
