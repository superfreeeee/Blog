# CSS 滚动条: 自定义滚动条样式

@[TOC](文章目录)

<!-- TOC -->

- [CSS 滚动条: 自定义滚动条样式](#css-滚动条-自定义滚动条样式)
- [前言](#前言)
- [正文](#正文)
  - [overflow & ::-webkit-scrollbar](#overflow---webkit-scrollbar)
  - [实际效果（自定义滚动条、隐藏滚动条）](#实际效果自定义滚动条隐藏滚动条)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

本篇来介绍如何为自己的网页定制化自己的滚动条

# 正文

由于滚动条大部分时候其实是属于浏览器管控的元素，所以我们只能透过一些伪类的方式来影响它的样式。本篇介绍的是专用于 webkit 引擎上生效的写法

## overflow & ::-webkit-scrollbar

与滚动条相关的属性无非就是 `overflow` 和 `::-webkit-scrollbar` 两个

- `overflow` [MDN-reference](https://developer.mozilla.org/zh-CN/docs/Web/CSS/overflow) 属性的作用在于指定内容溢出的场景，实际上我们可以再细分为 `overflow-x`、`overflow-y` 分别指定两个方向的溢出效果。本篇中我们要讨论的主要就是以下几个用法

    - `overflow: hidden` 或 `overflow: visible`：就是单纯的显示 or 隐藏内容
    - `overflow: scroll`：出现滚动条
    - `overflow: auto`：自动在需要的方向出现滚动条

    这边要注意的是 auto 的意思并不是滚动时才出现，而是只在对应方向出现，我们平常看到的 chrome 里面的那个滚动条实际上是内置的所以才可以在滚动时才出现

- `::-webkit-scrollbar` [MDN-reference](https://developer.mozilla.org/zh-CN/docs/Web/CSS/::-webkit-scrollbar) 则是一系列在 webkit 环境下允许我们自定义滚动条样式的选择器，分别有以下几种

    - `::-webkit-scrollbar`：整个滚动条
    - `::-webkit-scrollbar-button`：滚动条上的按钮 (上下箭头)
    - `::-webkit-scrollbar-thumb`：滚动条上的滚动滑块
    - `::-webkit-scrollbar-track`：滚动条轨道
    - `::-webkit-scrollbar-track-piece`：滚动条没有滑块的轨道部分
    - `::-webkit-scrollbar-corner`：当同时有垂直滚动条和水平滚动条时交汇的部分
    - `::-webkit-resizer`：某些元素的corner部分的部分样式(例:textarea的可拖动按钮)

## 实际效果（自定义滚动条、隐藏滚动条）

接下来我们看看实际效果

第一个是我们平常看到的滚动条

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_scroll_bar_sample1.gif)

第二种我们加上自己的滚动条样式

```css
.block::-webkit-scrollbar {
  width: 0.5em;
  background-color: #d9d9d9;
}

.block::-webkit-scrollbar-thumb {
  border-radius: 0.25em;
  background-color: #b9b9b9;
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_scroll_bar_sample2.gif)

实际上 auto 也不会自动隐藏滚动条的，最后我们看看 auto 跟 scroll 差别在哪里，下面我们这样改

```css
.block {
    overflow: scroll;
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_scroll_bar_sample3.png)

我们发现本来 x 方向是没有滚动条的，但还是一并出现了

最后你真的看 windows 的滚动条不爽，实际上我们也可以直接隐藏滚动条

```css
.block::-webkit-scrollbar {
    display: none;
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_scroll_bar_sample4.gif)

# 结语

本篇介绍关于滚动条的样式设计，如果你的设计师真的很挑，那可能还是需要学一下hhh

# 其他资源

## 参考连接

| Title                     | Link                                                                                                                                         |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| overflow - MDN            | [https://developer.mozilla.org/zh-CN/docs/Web/CSS/overflow](https://developer.mozilla.org/zh-CN/docs/Web/CSS/overflow)                       |
| ::-webkit-scrollbar - MDN | [https://developer.mozilla.org/zh-CN/docs/Web/CSS/::-webkit-scrollbar](https://developer.mozilla.org/zh-CN/docs/Web/CSS/::-webkit-scrollbar) |
| CSS进阶篇--设置滚动条样式 | [https://segmentfault.com/a/1190000003708894](https://segmentfault.com/a/1190000003708894)                                                   |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/css/css_scroll_bar](https://github.com/superfreeeee/Blog-code/tree/main/front_end/css/css_scroll_bar)
