# CSS 实战: Switch 按钮开关(checkbox 实现)

@[TOC](文章目录)

<!-- TOC -->

- [CSS 实战: Switch 按钮开关(checkbox 实现)](#css-实战-switch-按钮开关checkbox-实现)
- [正文](#正文)
  - [1. 效果](#1-效果)
  - [2. 代码实现](#2-代码实现)
    - [2.1 html 结构](#21-html-结构)
    - [2.2 外框 wrapper + 底座 box 样式](#22-外框-wrapper--底座-box-样式)
    - [2.3 白色按钮 but](#23-白色按钮-but)
    - [2.4 表情符号](#24-表情符号)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

## 1. 效果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_checkbox_design_1_unchecked.png) ![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_checkbox_design_2_checked.png)

## 2. 代码实现

### 2.1 html 结构

```html
<label class="box-wrapper">
  <input type="checkbox" />
  <div class="box">
    <div class="but"></div>
  </div>
</label>
```

### 2.2 外框 wrapper + 底座 box 样式 

```css
/* 外框 */
.box-wrapper {
  cursor: pointer;
}

/* 输入框 */
.box-wrapper > input {
  display: none;
}

/* 底座 */
.box-wrapper > .box {
  position: relative;
  width: 70px;
  height: 40px;
  border-radius: 20px;
  background: var(--color_red);
  box-shadow: 0 10px 20px var(--shadow_red);
  transition: background 0.3s ease, box-shadow 0.3s ease;
}

.box-wrapper > input:checked + .box {
  background: var(--color_green);
  box-shadow: 0 10px 20px var(--shadow_green);
}
```

整个 label 使用 `cursor: pointer;`；隐藏 input 元素；底座使用圆角矩形 + 开关变色 + 渐变动画

### 2.3 白色按钮 but

```css
/* 按钮 */
.box-wrapper > .box > .but {
  position: absolute;
  top: 4px;
  left: 4px;
  width: 32px;
  height: 32px;
  border-radius: 50%;
  background: #fff;
  transition: left 0.45s ease;
}

.box-wrapper > input:checked + .box > .but {
  left: 34px;
}
```

按钮就是一个白色圆形，按选中状态决定定位，并加上 transition 渐变

### 2.4 表情符号

表情是比较有趣的部分，使用 before 伪元素的 background + box-shadow 实现眼睛

```css
/* 按钮表情 */
.box-wrapper > .box > .but::before {
  content: '';
  position: absolute;
  top: 8px;
  left: 7px;
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: var(--color_red);
  box-shadow: 12px 0 0 var(--color_red);
  transition: background 0.3s ease, box-shadow 0.3s ease;
}

.box-wrapper > input:checked + .box > .but::before {
  background: var(--color_green);
  box-shadow: 12px 0 0 var(--color_green);
}
```

然后使用 after 伪元素实现嘴巴

```css
.box-wrapper > .box > .but::after {
  content: '';
  position: absolute;
  top: 21px;
  left: 9px;
  width: 14px;
  height: 4px;
  border-radius: 4px;
  background: var(--color_red);
  transition: background 0.3s ease, top 0.45s ease, height 0.45s ease, border-radius 0.45s ease;
}

.box-wrapper > input:checked + .box > .but::after {
  top: 20px;
  height: 6px;
  border-radius: 0;
  border-bottom-left-radius: 6px;
  border-bottom-right-radius: 6px;
  background: var(--color_green);
}
```

# 其他资源

## 参考连接

| Title                      | Link                                |
| -------------------------- | ----------------------------------- |
| CSS Custom Checkbox Design | Html CSS Tutorial @Online Tutorials | [https://www.youtube.com/watch?v=Lc1Neq59c-A](https://www.youtube.com/watch?v=Lc1Neq59c-A) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/css/css_checkbox_design](https://github.com/superfreeeee/Blog-code/tree/main/front_end/css/css_checkbox_design)
