# CSS 实战: Loading 动画

@[TOC](文章目录)

<!-- TOC -->

- [CSS 实战: Loading 动画](#css-实战-loading-动画)
- [正文](#正文)
  - [1. html 结构](#1-html-结构)
  - [2. div 实现样式](#2-div-实现样式)
  - [3. svg 实现样式](#3-svg-实现样式)
  - [4. 实现效果](#4-实现效果)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

## 1. html 结构

这里一共实现两种 loading 图样，一个使用 `<div>` 标签，一个则是 svg 的 `<circle>` 元素

```html
<div class="loading-css"></div>
<svg viewBox="0 0 50 50" class="loading-svg">
  <circle cx="25" cy="25" r="20" fill="none" class="path"></circle>
</svg>
```

区别在于对于普通 div 元素我们只能使用 `border` 属性来控制四个方位的边界，使用 svg 标签我们则可以利用 `strokeArray` 来构建任意造型的转菊花线条

## 2. div 实现样式

关于 div 实现的样式可想而知就是 border + animation 的组合

```css
.loading-css {
  box-sizing: border-box;
  width: 50px;
  height: 50px;
  display: inline-block;
  border: 5px solid #f3f3f3;
  border-top: 5px solid red;
  border-radius: 50%;
  /* 动画旋转效果 */
  animation: rotate-360 1s infinite linear;
}

@keyframes rotate-360 {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
```

## 3. svg 实现样式

svg 就比较有趣了，分为 svg 根元素整体旋转，以及里面的圆圈随时间修改虚线样式

```css
.loading-svg {
  width: 50px;
  height: 50px;
  animation: loading-rotate 1.5s infinite ease-in-out;
}

@keyframes loading-rotate {
  to {
    transform: rotate(1turn);
  }
}

.loading-svg > .path {
  stroke: #409eff;
  stroke-width: 5;
  animation: loading-dash 1.5s infinite ease-in-out;
}

@keyframes loading-dash {
  0% {
    stroke-dasharray: 1, 125;
    stroke-dashoffset: 0;
  }

  50% {
    stroke-dasharray: 95, 31;
    stroke-dashoffset: -31px;
  }

  to {
    stroke-dasharray: 6, 120;
    stroke-dashoffset: -120px;
  }
}
```

## 4. 实现效果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_loading_result.gif)

# 其他资源

## 参考连接

| Title                                                            | Link |
| ---------------------------------------------------------------- | ---- |
| 很久以前看到的，忘记参考的大哥的链接了，看到的话留个言给你补上hh | []() |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/css/css_loading](https://github.com/superfreeeee/Blog-code/tree/main/front_end/css/css_loading)
