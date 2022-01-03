# CSS 实战: 纯 CSS 画三角形

@[TOC](文章目录)

<!-- TOC -->

- [CSS 实战: 纯 CSS 画三角形](#css-实战-纯-css-画三角形)
- [实现技术核心: Border 属性基础](#实现技术核心-border-属性基础)
- [改变高度、宽度](#改变高度宽度)
- [扭曲变形](#扭曲变形)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 实现技术核心: Border 属性基础

使用 CSS 实现三角形的基础在于 Border 属性

首先我们先来看基础的 border 使用

```css
#t1 {
  border-top: 20px solid blue;
  border-right: 20px solid green;
  border-bottom: 20px solid yellow;
  border-left: 20px solid red;
}
#t11 {
  border: 20px solid transparent;
  border-top: 20px solid blue;
}
#t12 {
  border: 20px solid transparent;
  border-right: 20px solid green;
}
#t13 {
  border: 20px solid transparent;
  border-bottom: 20px solid yellow;
}
#t14 {
  border: 20px solid transparent;
  border-left: 20px solid red;
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_triangle_1_basic.png)

border 属性实际上是填满由 content 内容部分到外边框的四个梯形所组成，因此如果将 width、height 属性置为 0，就可以产生三角形的效果。

记得其他三个边一样必须设置宽度，否则宽、高塌陷之后就不会出现图形

# 改变高度、宽度

如果我们想要自由控制高度，则是改变某一边 border 的宽度

```css
#t21 {
  border: 20px solid transparent;
  border-top: 50px solid transparent;
  border-bottom: 10px solid green;
}
#t22 {
  border: 20px solid transparent;
  border-top: 40px solid transparent;
  border-bottom: 20px solid green;
}
#t23 {
  border: 20px solid transparent;
  border-top: 30px solid transparent;
  border-bottom: 30px solid green;
}
#t24 {
  border: 20px solid transparent;
  border-top: 20px solid transparent;
  border-bottom: 40px solid green;
}
#t25 {
  border: 20px solid transparent;
  border-top: 10px solid transparent;
  border-bottom: 50px solid green;
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_triangle_2_height.png)

改变宽度则比较不一样，我们则是要改变与三角形高度垂直的两个方向的透明 border 的高度

```css
#t31 {
  border: 20px solid transparent;
  border-left: 10px solid transparent;
  border-right: 10px solid transparent;
  border-bottom: 20px solid green;
}
#t32 {
  border: 20px solid transparent;
  border-left: 15px solid transparent;
  border-right: 15px solid transparent;
  border-bottom: 20px solid green;
}
#t33 {
  border: 20px solid transparent;
  border-left: 20px solid transparent;
  border-right: 20px solid transparent;
  border-bottom: 20px solid green;
}
#t34 {
  border: 20px solid transparent;
  border-left: 25px solid transparent;
  border-right: 25px solid transparent;
  border-bottom: 20px solid green;
}
#t35 {
  border: 20px solid transparent;
  border-left: 30px solid transparent;
  border-right: 30px solid transparent;
  border-bottom: 20px solid green;
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_triangle_3_width.png)

# 扭曲变形

最后一种，我们前面的三角形都是正中，如果想要出现直角三角形或是其他角度的三角形，则要同时使用上面提过的垂直方向的透明 Border 高度

```css
#t41 {
  border-top: 0;
  border-left: 0;
  border-right: 40px solid transparent;
  border-bottom: 40px solid green;
}
#t42 {
  border-top: 0;
  border-left: 10px solid transparent;
  border-right: 30px solid transparent;
  border-bottom: 40px solid green;
}
#t43 {
  border-top: 0;
  border-left: 20px solid transparent;
  border-right: 20px solid transparent;
  border-bottom: 40px solid green;
}
#t44 {
  border-top: 0;
  border-left: 30px solid transparent;
  border-right: 10px solid transparent;
  border-bottom: 40px solid green;
}
#t45 {
  border-top: 0;
  border-left: 40px solid transparent;
  border-right: 0;
  border-bottom: 40px solid green;
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_triangle_4_transform.png)

# 其他资源

## 参考连接

| Title             | Link                                                                                       |
| ----------------- | ------------------------------------------------------------------------------------------ |
| 如何用css画三角形 | [https://segmentfault.com/a/1190000005715074](https://segmentfault.com/a/1190000005715074) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/css/css_triangle](https://github.com/superfreeeee/Blog-code/tree/main/front_end/css/css_triangle)
