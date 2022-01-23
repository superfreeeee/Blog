# HTML 踩坑笔记: video 标签 autoplay 属性失效(Error: Uncaught (in promise) DOMException: play() failed)

@[TOC](文章目录)

<!-- TOC -->

- [HTML 踩坑笔记: video 标签 autoplay 属性失效(Error: Uncaught (in promise) DOMException: play() failed)](#html-踩坑笔记-video-标签-autoplay-属性失效error-uncaught-in-promise-domexception-play-failed)
- [0. 项目背景](#0-项目背景)
- [1. 问题描述](#1-问题描述)
  - [尝试: 主动调用 play 方法（失败）](#尝试-主动调用-play-方法失败)
- [2. 解决方案](#2-解决方案)
  - [2.1 解决方案 1: 静音](#21-解决方案-1-静音)
  - [2.2 解决方案 2: 监听初次点击后播放](#22-解决方案-2-监听初次点击后播放)
  - [2.3 解决方案 3: 将视频放入用户操作链路后段(推荐)](#23-解决方案-3-将视频放入用户操作链路后段推荐)
- [参考连接](#参考连接)
- [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 0. 项目背景

- 浏览器：Chrome

在前端项目中使用 &lt;video /&gt; 标签

# 1. 问题描述

使用如下 html 元素

```html
<video autopaly />
```

- 核心问题：自动播放无效

## 尝试: 主动调用 play 方法（失败）

加入如下代码

```html
<video autopaly />

<script>
  document.querySelector('video').play()
<script/>
```

产生如下报错

![](https://picures.oss-cn-beijing.aliyuncs.com/img/html_note_video_autoplay_1_error.png)

```
Uncaught (in promise) DOMException: play() failed because the user didn't interact with the document first.
```

# 2. 解决方案

问题在于 Chrome 的视频政策：[传送门 - Autoplay policy in Chrome](https://developer.chrome.com/blog/autoplay/#developer-switches)

重点就是：

> 避免自动播放视屏，最小化用户安装 Ads Block 的想法hh

因此我们可以提出以下三种解决方案

## 2.1 解决方案 1: 静音

根据 Chrome 的规则，加上 `muted` 静音属性就允许自动播放了

```html
<video src="sample.mp4" autoplay muted></video>
```

然而这或许就失去播放的意义了hh，还要找个机会把静音取消掉

## 2.2 解决方案 2: 监听初次点击后播放

第二种办法就是等待用户进行第一次操作之后就能够播放了，代码如下

- `index.html`

```html
<video src="sample.mp4" controls autoplay></video>

<script>
  const video = document.querySelector('video');

  // auto play when script loaded
  video
    .play()
    .then(() => {
      console.log(`autoplay well`);
    })
    .catch((e) => {
      console.log(`autoplay fail, wait for first click`);
      if (e instanceof DOMException) {
        // play before user intersact
        const autoPlayAfterAnyClick = () => {
          video.play();
          document.removeEventListener('click', autoPlayAfterAnyClick);
        };

        document.addEventListener('click', autoPlayAfterAnyClick);
        throw e;
      } else {
        // or rethrow
        throw e;
      }
    });
</script>
```

我们先调用 `play` 方法检测是否调用成功，如果因为 autoplay policy 失败则监听 document 的点击事件

## 2.3 解决方案 3: 将视频放入用户操作链路后段(推荐)

最后一种应该也是最好的，那就是不要自动播放，或是说根据用户的操作/路由/导航，再进行视频的加载、播放，如此一来 autoplay 属性也不会受 policy 的影响

- `index2.html`

```html
<video controls autoplay></video>

<button id="load">Load video</button>

<script>
  const video = document.querySelector('video');

  document.querySelector('#load').addEventListener('click', () => {
    video.src = './sample.mp4';
  });
</script>
```

# 参考连接

| Title                                         | Link                                                                                                                                                   |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Autoplay policy in Chrome - Chrome Developers | [https://developer.chrome.com/blog/autoplay/#developer-switches](https://developer.chrome.com/blog/autoplay/#developer-switches)                       |
| what constitues user gesture - stackoverflow  | [https://stackoverflow.com/questions/56388258/what-constitues-user-gesture](https://stackoverflow.com/questions/56388258/what-constitues-user-gesture) |

# 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/html/html_video_autoplay](https://github.com/superfreeeee/Blog-code/tree/main/front_end/html/html_video_autoplay)
