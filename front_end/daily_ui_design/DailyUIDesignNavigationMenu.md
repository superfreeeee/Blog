# UI 实现分享: 动态导航栏

@[TOC](文章目录)

<!-- TOC -->

- [UI 实现分享: 动态导航栏](#ui-实现分享-动态导航栏)
- [正文](#正文)
  - [1. 实现效果](#1-实现效果)
  - [2. 实现细节](#2-实现细节)
    - [2.1 html 布局](#21-html-布局)
    - [2.2 元素定位](#22-元素定位)
    - [2.3 导航栏列表项](#23-导航栏列表项)
    - [2.4 动画效果](#24-动画效果)
    - [2.5 hover 变色](#25-hover-变色)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

## 1. 实现效果

先来看看最终实现效果

- 静态图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/daily_ui_design_navigation_menu_1_img.png)

- 动态效果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/daily_ui_design_navigation_menu_2_record.GIF)

## 2. 实现细节

### 2.1 html 布局

元素布局上比较简单，分成中心的 `.toggle` 元素，以及周围的 `li` 元素列表

- `index.html`

```html
    <div id="app">
      <ul class="menu">
        <div class="toggle">
          <ion-icon name="add-outline"></ion-icon>
        </div>
      </ul>
    </div>
```

`li` 列表的部分我们稍微用一下 JavaScript 进行列表渲染

- `index.js`

```js
const menu = document.querySelector('.menu');

const items = [
  'home-outline',
  'person-outline',
  'settings-outline',
  'mail-outline',
  'key-outline',
  'videocam-outline',
  'game-controller-outline',
  'camera-outline',
];

items.forEach((type, i) => {
  const li = document.createElement('li');
  li.style = `--i: ${i}`;
  li.innerHTML = `<a><ion-icon name="${type}"></ion-icon></a>`;

  menu.appendChild(li);
});
```

注意这里的 `<ion-icon>` 使用 cdn 的 webcomponent 图标导入

在 `<head>` 里面贴上这两句就行了

```html
<script
  type="module"
  src="https://unpkg.com/ionicons@5.5.2/dist/ionicons/ionicons.esm.js"
></script>
<script
  nomodule
  src="https://unpkg.com/ionicons@5.5.2/dist/ionicons/ionicons.js"
></script>
```

### 2.2 元素定位

对于 UI 实现比较重要的部分在于 CSS 属性的设置，首先我们先来看布局、定位相关的

- `index.html`

body 的部分设置一下渐层背景色

```css
body {
  width: 100vw;
  height: 100vh;
  margin: 0;
  font-family: 'Poppins', sans-serif;
  background: linear-gradient(45deg, #8460ed, #ff1252);
  display: flex;
  justify-content: center;
  align-items: center;
}
```

接下来分别定为一下整个导航栏的结构

Menu 为 200 * 200、然后使用 flex 水平垂直居中

```css
.menu {
  position: relative;
  width: 200px;
  height: 200px;
  display: flex;
  justify-content: center;
  align-items: center;
}
```

中间的按钮为 60 * 60，然后使用 flex 使图标相对居中

```css
.toggle {
  width: 60px;
  height: 60px;
  font-size: 2em;
  z-index: 100;
}
```

### 2.3 导航栏列表项

接下来比较难的就是外围的导航栏列表项，一个一个定位未免太傻，所以我们使用 `transform-origin + transform: rotate` 的效果，制造出多个图标围绕同样一个中心进行旋转的效果（其中还使用 `i` 变量辅助计算）

```css
.menu li {
  position: absolute;
  left: 0;
  list-style: none;
  transform-origin: 100px;
  transform: rotate(calc(360deg / 8 * var(--i)));
  z-index: 1;
}
```

同时由于每个小项是以外部点为中心旋转，自身还需要转回来来保证图标正向朝上

```css
.menu li a {
  width: 40px;
  height: 40px;
  font-size: 1.3em;
  transform: rotate(calc(360deg / -8 * var(--i)));
  color: #111;
}
```

### 2.4 动画效果

最后再来实现动画效果

首先先对中心点加个点击事件来开关 `active` 属性

```js
const menu = document.querySelector('.menu');
const toggle = document.querySelector('.toggle');

let disable = false;

toggle.addEventListener('click', () => {
  // menu.classList.toggle('active');
  if (!disable) {
    disable = true;
    menu.classList.toggle('active');
    setTimeout(() => {
      disable = false;
    }, 1300);
  }
});
```

这里加了个计时器，整个动画大概会持续 1.3 秒，动画结束前不让再次点击

中心点旋转 315 度，使 `+` 符号旋转后变成 `x`

```css
.menu.active .toggle {
  transform: rotate(315deg);
}
```

接下来是列表项，一开始我们先将每个项隐藏在中间的大按钮后面，然后点击后旋转透出

```css
.menu li {
  transform-origin: 100px;
  transform: rotate(0) translateX(80px);
  transition-delay: calc(0.1s * var(--i));
  transition-duration: 0.5s;
  z-index: 1;
}

.menu.active li {
  transform: rotate(calc(360deg / 8 * var(--i)));
}
```

### 2.5 hover 变色

hover 之后改变图标颜色就不用说啦

```css
.menu.active li a:hover {
  color: #ff1252;
}
```

# 其他资源

## 参考连接

| Title                                                                 | Link               |
| --------------------------------------------------------------------- | ------------------ |
| Animated Circular Navigation Menu using Html CSS & Vanilla Javascript | Simple Radial Menu | [https://www.youtube.com/watch?v=ShPPkZEeLPo](https://www.youtube.com/watch?v=ShPPkZEeLPo) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/daily_ui_design/daily_ui_design_navigation_menu](https://github.com/superfreeeee/Blog-code/tree/main/front_end/daily_ui_design/daily_ui_design_navigation_menu)
