# UI 实现分享: Setting 页面

@[TOC](文章目录)

<!-- TOC -->

- [UI 实现分享: Setting 页面](#ui-实现分享-setting-页面)
- [正文](#正文)
  - [0. 简介](#0-简介)
  - [1. 实现思路 & 布局规划](#1-实现思路--布局规划)
    - [1.1 整体布局](#11-整体布局)
    - [1.2 侧边栏](#12-侧边栏)
    - [1.3 主页面](#13-主页面)
  - [2. 设计原 & 还原效果对比](#2-设计原--还原效果对比)
  - [3. 实现小结](#3-实现小结)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

## 0. 简介

今天要来分享的东西看起来有点傻，但是对于前端新手或是样式的布局比较不熟悉的人来说（例如我）应该还是有点帮助的。

很多时候我们花很多时间在布局和样式调整上，是因为我们不知道到底该怎么选择样式的调整方式，到底什么时候用 flex、什么时候用 grid、position，居中的时候应该用 flex 还是用 position + transform 或是简单的 text-align 就好。这些东西和决策都是在练习中慢慢养成的基本意识。

因此往后我们会开启几篇关于 UI 样式实现的分享。对标实际工作场景，通常我们都会看着设计图来实现，或是去找一些样板。

本篇参照的样板在 [https://www.uidesigndaily.com/posts/figma-settings-menu-radio-button-day-1467](https://www.uidesigndaily.com/posts/figma-settings-menu-radio-button-day-1467) 有兴趣的可以自己把设计图下载下来试试看

![](https://picures.oss-cn-beijing.aliyuncs.com/img/daily_ui_design_setting_4_origin.png)

## 1. 实现思路 & 布局规划

本篇不会贴什么代码，默认你已经熟悉基础的 HTML、CSS 知识了，这里仅仅给出一些写样式的想法

### 1.1 整体布局

![](https://picures.oss-cn-beijing.aliyuncs.com/img/daily_ui_design_setting_1_layout.png)

根据上面的原图，我们很容易想到将整个布局分成 Header + Sidebar + Main 三块的布局

这里的实现我们可以运用下面的代码实现

```html
<div class="setting">
  <div class="header">
    Header
  </div>
  <div class="content">
    <div class="sidebar">
      Sidebar
    </div>
    <div class="main">
      Main
    </div>
  </div>
</div>
```

这里利用 div 属于块元素会占据整行的特性，将其分成 header、content 上下两块；content 则可以使用 flex 布局，分成 sidebar、main 的左右两块，并利用 sidebar 定宽而 main 实现 `flex: 1` 来撑满屏幕宽度

### 1.2 侧边栏

![](https://picures.oss-cn-beijing.aliyuncs.com/img/daily_ui_design_setting_2_sidebar.png)

对于侧边栏的实现我们分成外元素 Menu 组件，还有导航栏中的列表项 MenuItem，这里一样我们也不需要加上太多的 flex，可以简单使用 ul + li 的排列

而对于 MenuItem 内部如果只有文字本身，我们可以使用 height、line-height 等高的方式来时文字垂直居中；如果是 icon + 文字的组合我们则可以使用 flex + align-items: center 的方式来使两者处于同一条水平线上

### 1.3 主页面

![](https://picures.oss-cn-beijing.aliyuncs.com/img/daily_ui_design_setting_3_main.png)

主页面我们分成 Question、Selections、Answer、Options 几块分别进行样式的调整，这里我们可以选择两种方案

1. Main 组件使用 flex + flex-direciton: column 对齐，并默认使用 align-items: stretch 使内元素默认撑开，然后子元素再使用 align-self 进行单独调整
2. 第二种实际上我们就把他们都当作普通 div 元素就好，Options 这种可以在里面多套一层 span + display: inline-block + text-align: right 的方式实现内容靠右对齐

## 2. 设计原 & 还原效果对比

- 设计图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/daily_ui_design_setting_4_origin.png)

- 还原效果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/daily_ui_design_setting_5_implement.png)

## 3. 实现小结

- 内容垂直对齐
  - 纯文字：height、line-height 设置等高
  - icon + 文字：flex + align-items: center
- 内容水平对齐
  - text-align
  - flex
  - width: 100%
- 高层次块元素规划
  - 尽量少依赖 flex 以减少文档流的排列复杂度

# 其他资源

## 参考连接

| Title                  | Link                                                                                                                                                           |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Design Daily - Setting | [https://www.uidesigndaily.com/posts/figma-settings-menu-radio-button-day-1467](https://www.uidesigndaily.com/posts/figma-settings-menu-radio-button-day-1467) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/daily_ui_design/daily_ui_design_setting](https://github.com/superfreeeee/Blog-code/tree/main/front_end/daily_ui_design/daily_ui_design_setting)
