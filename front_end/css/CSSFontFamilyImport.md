# CSS 实用工具: Google Fonts API 引入免费字体库

@[TOC](文章目录)

<!-- TOC -->

- [CSS 实用工具: Google Fonts API 引入免费字体库](#css-实用工具-google-fonts-api-引入免费字体库)
- [正文](#正文)
  - [1. Google Fonts 使用](#1-google-fonts-使用)
  - [2. 在代码中引入字体 & 效果](#2-在代码中引入字体--效果)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

今天给大家分享一个有趣的工具，写 Web 的时候常常找不到有什么字体是好看的或是要一直去找。本篇将给大家介绍一个免费的字体库，由 Google 提供的 API

## 1. Google Fonts 使用

地址：[https://fonts.google.com/](https://fonts.google.com/)

首先点击去上面那个网站

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_font_family_import_1_fonts1.png)

将会看到有很多字体可以选择，点进去你想要的字体

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_font_family_import_2_fonts2.png)

然后点击 `select this style`，你可以一次选中多个字体，然后在代码中一次导入

接下来在右侧选择 `@import` 复制下面的内容到 css 如下

```css
@import url('https://fonts.googleapis.com/css2?family=Space+Mono&family=Trispace&family=Zen+Antique+Soft&display=swap');
```

## 2. 在代码中引入字体 & 效果

最后你就可以在你的 css 文件中使用想相关字体了

- `index.css`

```css
@import url('https://fonts.googleapis.com/css2?family=Space+Mono&family=Trispace&family=Zen+Antique+Soft&display=swap');

.font1 {
  font-family: 'Space Mono', monospace;
}

.font2 {
  font-family: 'Trispace', sans-serif;
}

.font3 {
  font-family: 'Zen Antique Soft', serif;
}
```

- 效果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_font_family_import_3_sample.png)

# 其他资源

## 参考连接

| Title            | Link                                                   |
| ---------------- | ------------------------------------------------------ |
| Google Fonts API | [https://fonts.google.com/](https://fonts.google.com/) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/css/css_font_family_import](https://github.com/superfreeeee/Blog-code/tree/main/front_end/css/css_font_family_import)
