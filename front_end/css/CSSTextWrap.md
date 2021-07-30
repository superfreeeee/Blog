# CSS 应用: 文本溢出省略效果(单行/多行)

@[TOC](文章目录)

<!-- TOC -->

- [CSS 应用: 文本溢出省略效果(单行/多行)](#css-应用-文本溢出省略效果单行多行)
- [前言](#前言)
- [正文](#正文)
  - [使用属性介绍](#使用属性介绍)
  - [1. 单行文本溢出](#1-单行文本溢出)
  - [2. 多行文本溢出](#2-多行文本溢出)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

本篇主要介绍一个在 css 中还挺常见的文本溢出省略效果，也就是指定行数之后就会在最后一行最后变成删节号表示有更多文本。

# 正文

## 使用属性介绍

首先我们介绍几个待会会用到的 css 属性

- `overflow` [MDN reference](https://developer.mozilla.org/zh-CN/docs/Web/CSS/overflow)

    主要处理任何内容在块元素中溢出时效果，常见的值有

    - `overflow: visible`：显示溢出内容
    - `overflow: hidden`：隐藏溢出内容
    - `overflow: scroll`：滚动条
    - `overflow: auto`：有需要自动出现滚动条

- `white-space` [MDN reference](https://developer.mozilla.org/zh-CN/docs/Web/CSS/white-space)

    空白内容在元素中的处理机制，包括**连续空白的合并、换行符、\<br/\> 元素的处理**

- `word-break` [MDN reference](https://developer.mozilla.org/zh-CN/docs/Web/CSS/word-break)

    指定换行时文本断开的机制，尤其在处理长英文单词时需要特别处理

- `text-overflow` [MDN reference](https://developer.mozilla.org/zh-CN/docs/Web/CSS/text-overflow)

    指定文本溢出时的效果，与 overflow 不同的是，text-overflow 只关心文本，也就是提供文本删节号的核心属性

接下来直接看例子

## 1. 单行文本溢出

```css
.single-line {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
```

单行文本溢出很简单，`text-overflow: ellipsis` 给出溢出时的删节号；`white-space: nowrap` 指定文本不换行。

## 2. 多行文本溢出

```css
.multiple-line {
  display: -webkit-box;
  overflow: hidden;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
}
```

多行文本唯一不同的是，使用 `display: -webkit-box` 指定为特殊的块元素，`-webkit-box-orient: vertical` 指定文本排列交错轴方向，最后 `-webkit-line-clamp: 3` 指定最大文本高度同时自带 `text-overflow: ellipsis` 的效果。

效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_text_wrap_result.png)

# 结语

文本换行是一个挺常见的需求，供大家参考

# 其他资源

## 参考连接

| Title                                   | Link                                                                                                                             |
| --------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| overflow - MDN                          | [https://developer.mozilla.org/zh-CN/docs/Web/CSS/overflow](https://developer.mozilla.org/zh-CN/docs/Web/CSS/overflow)           |
| white-space - MDN                       | [https://developer.mozilla.org/zh-CN/docs/Web/CSS/white-space](https://developer.mozilla.org/zh-CN/docs/Web/CSS/white-space)     |
| word-break - MDN                        | [https://developer.mozilla.org/zh-CN/docs/Web/CSS/word-break](https://developer.mozilla.org/zh-CN/docs/Web/CSS/word-break)       |
| text-overflow - MDN                     | [https://developer.mozilla.org/zh-CN/docs/Web/CSS/text-overflow](https://developer.mozilla.org/zh-CN/docs/Web/CSS/text-overflow) |
| CSS 实现文本的单行和多行溢出省略效 #130 | [https://github.com/sisterAn/JavaScript-Algorithms/issues/130](https://github.com/sisterAn/JavaScript-Algorithms/issues/130)     |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/css/css_word_wrap](https://github.com/superfreeeee/Blog-code/tree/main/front_end/css/css_word_wrap)
