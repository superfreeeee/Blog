# JS API: Fullscreen 全屏 API

@[TOC](文章目录)

<!-- TOC -->

- [JS API: Fullscreen 全屏 API](#js-api-fullscreen-全屏-api)
- [正文](#正文)
  - [1. 相关 API](#1-相关-api)
  - [2. 代码示例](#2-代码示例)
    - [2.1 一般用法](#21-一般用法)
    - [2.2 封装成钩子](#22-封装成钩子)
    - [2.3 效果](#23-效果)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

今天来介绍一下全屏模式 API 使用

## 1. 相关 API

- 属性

| Properties                 | 用法                     |
| -------------------------- | ------------------------ |
| Document.fullscreenElement | 当前处于全屏模式的根元素 |
| Document.fullscreenEnabled | 检查全屏模式是否可用     |

- 方法

| Method                      | 用法                                                |
| --------------------------- | --------------------------------------------------- |
| Document.exitFullscreen()   | 退出全屏模式                                        |
| Element.requestFullscreen() | 以 Element 为底触发全屏模式，通常可以使用 body 元素 |

- 事件

| Event                                    | 用法         |
| ---------------------------------------- | ------------ |
| (Document \| Element).onfullscreenchange | 全屏模式改变 |
| (Document \| Element).onfullscreenerror  | 全屏模式异常 |

## 2. 代码示例

### 2.1 一般用法

一般流程我们就是使用 `requestFullscreen` 方法进入全屏模式，使用 `exitFullscreen` 退出，并用 `onfullscreenchange` 监听状态的修改

### 2.2 封装成钩子

以上的用法我们可以封装成一个自定义的 Hook

- `/src/hooks/useFullScreen.ts`

```ts
import React, { useCallback, useEffect, useRef, useState } from 'react';

type PlainFunction = () => void;

type FullScreenHookRes<T extends HTMLElement> = [
  React.RefObject<T>,
  {
    requestFullScreen: PlainFunction;
    exitFullScreen: PlainFunction;
    toggleFullScreen: PlainFunction;
    isFullScreen: boolean;
  }
];

const useFullScreen = <T extends HTMLElement = HTMLBodyElement>(): FullScreenHookRes<T> => {
  const elementRef = useRef<T>(null);
  const [isFullScreen, setIsFullScreen] = useState(!!document.fullscreenElement);
```

首先是一些状态的保留

```ts
  useEffect(() => {
    document.body.addEventListener('fullscreenchange', () => {
      setIsFullScreen(!!document.fullscreenElement);
    });
    return () => {
      isFullScreen && document.exitFullscreen();
    };
  }, []);
```

Mount/Unmount 的时候进行状态的更新、退出全屏

```ts
  const requestFullScreen = useCallback(() => {
    (elementRef.current || (elementRef.current = document.body as T)).requestFullscreen();
  }, []);

  const exitFullScreen = useCallback(() => {
    document.fullscreenElement && document.exitFullscreen();
  }, []);

  const toggleFullScreen = useCallback(() => {
    document.fullscreenElement ? exitFullScreen() : requestFullScreen();
  }, []);

  return [elementRef, { requestFullScreen, exitFullScreen, toggleFullScreen, isFullScreen }];
};

export default useFullScreen;
```

最后其实就是简单封装方法，由于我们可以直接使用 `document.fullscreenElement` 来判断实时的全屏状态，所以我们可以看到这三个方法都是无依赖列表的方法

### 2.3 效果

写一个简单页面

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_full_screen_1_origin.png)

下面我们可以看到分别是以 body 为底的全屏

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_full_screen_2_body.png)

另一个则是以 AppRoot 为底

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_full_screen_3_appRoot.png)

可以看到根元素之外的并不会出现在全屏模式当中

# 其他资源

## 参考连接

| Title          | Link                                                                                                                               |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| 全屏 API - MDN | [https://developer.mozilla.org/zh-CN/docs/Web/API/Fullscreen_API](https://developer.mozilla.org/zh-CN/docs/Web/API/Fullscreen_API) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_full_screen](https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_full_screen)
