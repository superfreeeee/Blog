# JS 实战: 前端图片下载实现

@[TOC](文章目录)

<!-- TOC -->

- [JS 实战: 前端图片下载实现](#js-实战-前端图片下载实现)
- [正文](#正文)
  - [1. Image + canvas API 实现](#1-image--canvas-api-实现)
  - [2. Fetch API / XHR 实现](#2-fetch-api--xhr-实现)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

## 1. Image + canvas API 实现

- `/src/utils/index.ts`

第一步是构造 Image 对象，实际上就是 HTMLImageElement 类型的对象，也就等价于 `<img>` 标签的作用

```ts
export const downloadImgByImage = (imgSrc: string, name: string) => {
  const image = new Image();
  image.crossOrigin = 'anonymous';
```

接下来确定加载好图片之后，创建一个 canvas 元素

```ts
  image.onload = function () {
    // draw on canvas
    const canvas = document.createElement('canvas');
    canvas.width = image.width;
    canvas.height = image.height;
    const context = canvas.getContext('2d') as CanvasRenderingContext2D;
```

使用 canvas API `context.drawImage` 写到画布上，再使用 `canvas.toDataURL` 创建数据地址

```ts
    context.drawImage(image, 0, 0, image.width, image.height);

    // create dataUrl
    const dataUrl = canvas.toDataURL('image/png');
```

最后创建 `<a>` 标签并点击以触发下载事件

```ts
    const a = document.createElement('a');
    a.download = name;
    a.href = dataUrl;

    const event = new MouseEvent('click');
    a.dispatchEvent(event);
  };
  image.src = imgSrc;
};
```

## 2. Fetch API / XHR 实现

- `/src/utils/index.ts`

```ts
export const downloadImgByFetch = (imgSrc: string, name: string) => {
  fetch(imgSrc)
    .then((res) => res.blob())
    .then((blob) => {
      const dataUrl = window.URL.createObjectURL(blob);
      const link = document.createElement('a');
      link.download = name;
      link.href = dataUrl;
      document.body.appendChild(link);
      link.click();
      link.remove();
    });
};
```

使用 fetch API 会稍微简单一点，返回的 reponse 对象本身就提供了 `blob` 方法转换成 Blob 对象，然后再使用 `URL.createObjectURL(blob)` 构建数据地址并下载。

这边需要注意两个点，一个是有的浏览器对于下载的限制是 `<a>` 标签必须存在 dom 树当中，所以要先 `document.body.appendChild` 再用 `remove` 移除

另改就是如果使用的不是 fetch API 而是使用原本的 XHR 对象，则需要使用 `new Blob([res.data])` 自己进行数据类型的转换

# 其他资源

## 参考连接

| Title                                | Link                                                                                                                                                                   |
| ------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| JS 通过 url 下载图片                 | [https://segmentfault.com/a/1190000038747836](https://segmentfault.com/a/1190000038747836)                                                                             |
| HTMLCanvasElement.toDataURL() - MDN  | [https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/toDataURL](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/toDataURL)           |
| Tainted canvases may not be exported | [https://stackoverflow.com/questions/22710627/tainted-canvases-may-not-be-exported](https://stackoverflow.com/questions/22710627/tainted-canvases-may-not-be-exported) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_download_image](https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_download_image)
