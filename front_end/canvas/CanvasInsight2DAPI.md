# 深入 Canvas: 基础 2D API

@[TOC](文章目录)

<!-- TOC -->

- [深入 Canvas: 基础 2D API](#深入-canvas-基础-2d-api)
- [正文](#正文)
  - [1. 基础 2D API 总览](#1-基础-2d-api-总览)
    - [1.1 Canvas 标签](#11-canvas-标签)
    - [1.2 渲染上下文](#12-渲染上下文)
    - [1.3 CanvasRenderingContext2D API](#13-canvasrenderingcontext2d-api)
  - [2. 绘制矩阵](#2-绘制矩阵)
  - [3. 绘制路径](#3-绘制路径)
    - [3.1 直线路径](#31-直线路径)
    - [3.2 曲线路径](#32-曲线路径)
    - [3.3 二次/三次贝塞尔曲线路径](#33-二次三次贝塞尔曲线路径)
  - [4. 绘制样式](#4-绘制样式)
    - [4.1 填充颜色](#41-填充颜色)
    - [4.2 描边颜色](#42-描边颜色)
    - [4.3 绘制透明度](#43-绘制透明度)
    - [4.4 线段样式](#44-线段样式)
    - [4.5 渐变颜色](#45-渐变颜色)
    - [4.6 绘制阴影](#46-绘制阴影)
  - [5. 绘制文字](#5-绘制文字)
    - [5.1 填充/描边文字](#51-填充描边文字)
    - [5.2 文字垂直基准线](#52-文字垂直基准线)
    - [5.3 测量文字宽度](#53-测量文字宽度)
  - [6. 绘制变形](#6-绘制变形)
    - [6.1 保存/恢复状态](#61-保存恢复状态)
    - [6.2 平移](#62-平移)
    - [6.3 旋转](#63-旋转)
    - [6.4 缩放](#64-缩放)
    - [6.5 变形转移矩阵](#65-变形转移矩阵)
  - [7. 图像操作/像素操作](#7-图像操作像素操作)
  - [8. 实践应用](#8-实践应用)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

## 1. 基础 2D API 总览

### 1.1 Canvas 标签

- width 表示像素宽度
- height 表示像素高度

```html
<canvas id="canvas" width="300" height="300"></canvas>
```

### 1.2 渲染上下文
  - `getContext` 方法
  - 参数 `2d` 表示获取 2D 渲染的上下文

```js
const ctx = document.querySelector('#canvas').getContext('2d')
```

### 1.3 CanvasRenderingContext2D API

- 矩形绘制相关

| 方法/属性                         | 描述           |
| --------------------------------- | -------------- |
| `fillRect(x, y, width, height)`   | 填充矩形       |
| `strokeRect(x, y, width, height)` | 绘制矩形(边框) |
| `rect(x, y, width, height)`       | ..             |
| `clearRect(x, y, width, height)`  | 清空矩形       |

- 路径绘制相关

| 方法/属性                                            | 描述                       |
| ---------------------------------------------------- | -------------------------- |
| `beginPath()`                                        | 开启路径                   |
| `closePath()`                                        | 闭合路径                   |
| `stroke()`                                           | 绘制路径                   |
| `fill(mode?)`                                        | 填充路径                   |
| `clip()`                                             | 裁切路径                   |
| `moveTo(x, y)`                                       | 移动画笔（不留下任何路径） |
| `lineTo(x, y)`                                       | 直线路径                   |
| `arc(cx, cy, r, startAngle, endAngle, counterClock)` | 圆形曲线                   |
| `quadraticCurveTo(cp1x, cp1y, x, y)`                 | 二次贝塞尔曲线             |
| `bezierCurveTo(cp1x, cp1y, cp2x, cp2y, x, y)`        | 三次贝塞尔曲线             |
| `Path2D`                                             | 创建路径记录               |

- 绘制样式相关

| 方法/属性                                      | 描述             |
| ---------------------------------------------- | ---------------- |
| `fillStyle = color`                            | 填充颜色         |
| `strokeStyle = color`                          | 描边/路径颜色    |
| `lineWidth = number`                           | 路径宽度         |
| `lineCap = type`                               | 线条末端样式     |
| `lineJoin = type`                              | 线条转折样式     |
| `miterLimit = number`                          | 线条转折限制     |
| `getLineDash()`                                | 获取线段虚线样式 |
| `setLineDash(segments)`                        | 设置线段虚线样式 |
| `createLinearGradient(x1, y1, x2, y2)`         | 线性渐变         |
| `createRadialGradient(x1, y1, r1, x2, y2, r2)` | 辐射渐变         |
| `gradient.addColorStop(position, color)`       | 新增颜色断点     |
| `createPattern(img, mode)`                     | 图片填色模式     |
| `globalAlpha = transparencyValue`              | 透明度           |
| `shadowOffsetX = float`                        | 阴影 x 偏移量    |
| `shadowOffsetY = float`                        | 阴影 y 偏移量    |
| `shadowBlur = float`                           | 阴影扩散宽度     |
| `shadowColor = color`                          | 阴影颜色         |
| `globalCompositeOperation = type`              | 图形覆盖样式     |

- 文字绘制相关

| 方法/属性                                     | 描述                 |
| --------------------------------------------- | -------------------- |
| `function fillText(text, x, y , maxWidth?)`   | 绘制实心文字         |
| `function strokeText(text, x, y , maxWidth?)` | 绘制描边文字         |
| `font = value`                                | 字体样式             |
| `textAlign = value`                           | 文字水平对齐方式     |
| `textBaseline = value`                        | 文字垂直基准线       |
| `direction = value`                           | 文字方向             |
| `measureText(text)`                           | 测量文字所需像素宽度 |

- 变形

| 方法/属性                        | 描述         |
| -------------------------------- | ------------ |
| `save()`                         | 保存画布状态 |
| `restore()`                      | 恢复画布状态 |
| `translate(x, y)`                | 平移         |
| `rotate(angle)`                  | 旋转         |
| `scale(x, y)`                    | 缩放         |
| `transform(a, b, c, d, e, f)`    | 修改变形矩阵 |
| `setTransform(a, b, c, d, e, f)` | 设置绝对变形 |
| `resetTransform()`               | 重制变形     |

- 图形填充/像素操作

| 方法/属性                                                            | 描述               |
| -------------------------------------------------------------------- | ------------------ |
| `drawImage(image, x, y)`                                             | 绘制图片           |
| `drawImage(image, x, y, width, height)`                              | 缩放               |
| `drawImage(image, sx, sy, sWidth, sHeight, dx, dy, dWidth, dHeight)` | 切片               |
| `mozImageSmoothingEnabled = boolean`                                 | 图片像素平滑效果   |
| `webkitImageSmoothingEnabled = boolean`                              | ..                 |
| `msImageSmoothingEnabled = boolean`                                  | ..                 |
| `imageSmoothingEnabled = boolean`                                    | ..                 |
| `createImageData(width, height)`                                     | 创建数据对象       |
| `createImageData(anotherImageData)`                                  | ..                 |
| `getImageData(left, top, width, height)`                             | 获取上下文数据对象 |
| `putImageData(myImageData, dx, dy)`                                  | 写入数据对象       |
| `canvas.toDataURL(MIME_TYPE)`                                        | 转为数据 base64    |
| `canvas.toDataURL(MIME_TYPE, quality)`                               | ..                 |
| `canvas.toBlob(callback, type, encoderOptions)`                      | 转为 Blob          |

API 还是比较多的，下面一个个尝试效果

## 2. 绘制矩阵

简单绘制矩阵就不多说了

```js
function drawRect() {
  ctx.fillRect(25, 25, 100, 100);
  ctx.clearRect(45, 45, 60, 60);
  ctx.strokeRect(50, 50, 50, 50);
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_2.png)

## 3. 绘制路径

路径通常由 `beginPath` 方法开始，同时会使用 `moveTo` 初始化绘制画笔坐标，然后使用 `fill` 或 `stroke` 方法绘制图形，可以使用 `closePath` 闭合路径

### 3.1 直线路径

`lineTo` 方法绘制直线路径

```js
function drawLine() {
  // 填充三角形
  ctx.beginPath();
  ctx.moveTo(25, 25);
  ctx.lineTo(105, 25);
  ctx.lineTo(25, 105);
  ctx.fill();

  // 描边三角形
  ctx.beginPath();
  ctx.moveTo(125, 125);
  ctx.lineTo(125, 45);
  ctx.lineTo(45, 125);
  ctx.closePath();
  ctx.stroke();
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_3_1.png)

### 3.2 曲线路径

`arc` 方法绘制曲线路径

```js
function drawArc() {
  for (var i = 0; i < 4; i++) {
    for (var j = 0; j < 3; j++) {
      ctx.beginPath();
      var x = 25 + j * 50; // x 坐标值
      var y = 25 + i * 50; // y 坐标值
      var radius = 20; // 圆弧半径
      var startAngle = 0; // 开始点
      var endAngle = Math.PI + (Math.PI * j) / 2; // 结束点
      var anticlockwise = i % 2 == 0 ? false : true; // 顺时针或逆时针

      ctx.arc(x, y, radius, startAngle, endAngle, anticlockwise);

      if (i > 1) {
        ctx.fill();
      } else {
        ctx.stroke();
      }
    }
  }
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_3_2.png)

### 3.3 二次/三次贝塞尔曲线路径

`quadraticCurveTo` 绘制二次曲线、`bezierCurveTo` 绘制三次曲线

```js
function* drawCurve() {
  resetCanvas();
  // 二次贝塞尔曲线
  ctx.beginPath();
  ctx.moveTo(75, 25);
  ctx.quadraticCurveTo(25, 25, 25, 62.5);
  ctx.quadraticCurveTo(25, 100, 50, 100);
  ctx.quadraticCurveTo(50, 120, 30, 125);
  ctx.quadraticCurveTo(60, 120, 65, 100);
  ctx.quadraticCurveTo(125, 100, 125, 62.5);
  ctx.quadraticCurveTo(125, 25, 75, 25);
  ctx.stroke();

  yield;
  resetCanvas();

  //三次贝塞尔曲线
  ctx.beginPath();
  ctx.moveTo(75, 40);
  ctx.bezierCurveTo(75, 37, 70, 25, 50, 25);
  ctx.bezierCurveTo(20, 25, 20, 62.5, 20, 62.5);
  ctx.bezierCurveTo(20, 80, 40, 102, 75, 120);
  ctx.bezierCurveTo(110, 102, 130, 80, 130, 62.5);
  ctx.bezierCurveTo(130, 62.5, 130, 25, 100, 25);
  ctx.bezierCurveTo(85, 25, 75, 37, 75, 40);
  ctx.fill();
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_3_3_1.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_3_3_2.png)

## 4. 绘制样式

### 4.1 填充颜色

`fillStyle` 决定填充颜色

```js
function drawColor(stroke = false) {
  const width = 50;
  const radius = width / 2 - 10;
  for (var i = 0; i < 6; i++) {
    for (var j = 0; j < 6; j++) {
      const c1 = Math.floor(255 - (255 / 6) * i);
      const c2 = Math.floor(255 - (255 / 6) * j);
      ctx.fillStyle = `rgb(${c1},${c2},0)`;
      ctx.fillRect(j * width, i * width, width, width);
    }
  }
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_4_1.png)

### 4.2 描边颜色

`strokeStyle` 决定描边颜色

```js
function drawColor(stroke = false) {
  const width = 50;
  const radius = width / 2 - 10;
  for (var i = 0; i < 6; i++) {
    for (var j = 0; j < 6; j++) {
      const c1 = Math.floor(255 - (255 / 6) * i);
      const c2 = Math.floor(255 - (255 / 6) * j);
      ctx.strokeStyle = `rgb(0,${c1},${c2})`;
      ctx.beginPath();
      ctx.arc(width / 2 + j * width, width / 2 + i * width, radius, 0, Math.PI * 2, true);
      ctx.stroke();
    }
  }
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_4_2.png)

### 4.3 绘制透明度

`globalAlpha` 表示透明度，可以使用白色半透明作为遮罩

```js
function drawTransparency() {
  const width = 150;

  // 画背景
  ctx.fillStyle = '#FD0';
  ctx.fillRect(0, 0, width, width);
  ctx.fillStyle = '#6C0';
  ctx.fillRect(width, 0, width, width);
  ctx.fillStyle = '#09F';
  ctx.fillRect(0, width, width, width);
  ctx.fillStyle = '#F30';
  ctx.fillRect(width, width, width, width);

  // 设置透明度值
  ctx.fillStyle = '#FFF';
  ctx.globalAlpha = 0.2;

  // 画半透明圆
  for (var i = 0; i < 7; i++) {
    ctx.beginPath();
    ctx.arc(width, width, 20 + 20 * i, 0, Math.PI * 2, true);
    ctx.fill();
  }
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_4_3.png)

### 4.4 线段样式

`lineWidth` 为线段宽度

```js
function drawLineWidth() {
  for (var i = 0; i < 10; i++) {
    ctx.lineWidth = 1 + i;
    ctx.beginPath();
    ctx.moveTo(5 + i * 14, 5);
    ctx.lineTo(5 + i * 14, 140);
    ctx.stroke();
  }
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_4_4_1.png)

`lineCap` 为线段末端样式

```js
function drawLineCap() {
  var lineCap = ['butt', 'round', 'square'];

  const offset = 30;
  const width = 300;

  // 创建路径
  ctx.strokeStyle = '#09f';
  ctx.beginPath();
  ctx.moveTo(offset, offset);
  ctx.lineTo(width - offset, offset);
  ctx.moveTo(offset, width - offset);
  ctx.lineTo(width - offset, width - offset);
  ctx.stroke();

  // 画线条
  ctx.strokeStyle = 'black';
  for (var i = 0; i < lineCap.length; i++) {
    const lineWidth = (ctx.lineWidth = offset);
    ctx.lineCap = lineCap[i];
    ctx.beginPath();
    ctx.moveTo(offset + lineWidth + (i * (width - (offset + lineWidth) * 2)) / 2, offset);
    ctx.lineTo(offset + lineWidth + (i * (width - (offset + lineWidth) * 2)) / 2, width - offset);
    ctx.stroke();
  }
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_4_4_2.png)

`lineJoin` 为线段转折样式

```js
function drawLineJoin() {
  var lineJoin = ['round', 'bevel', 'miter'];
  const width = (ctx.lineWidth = 20);
  for (var i = 0; i < lineJoin.length; i++) {
    ctx.lineJoin = lineJoin[i];
    ctx.beginPath();
    ctx.moveTo(-5, 5 + i * width * 4);
    ctx.lineTo(-5 + width * 2, 5 + width * 4 + i * width * 4);
    ctx.lineTo(-5 + width * 4, 5 + i * width * 4);
    ctx.lineTo(-5 + width * 6, 5 + width * 4 + i * width * 4);
    ctx.lineTo(-5 + width * 8, 5 + i * width * 4);
    ctx.lineTo(-5 + width * 10, 5 + width * 4 + i * width * 4);
    ctx.lineTo(-5 + width * 12, 5 + i * width * 4);
    ctx.lineTo(-5 + width * 14, 5 + width * 4 + i * width * 4);
    ctx.lineTo(-5 + width * 16, 5 + i * width * 4);
    ctx.stroke();
  }
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_4_4_3.png)

`setLineDash` 设置虚线样式

```js
let offset = 0,
  timer = null;
function drawAnt() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  ctx.setLineDash([4, 2]);
  ctx.lineDashOffset = -offset;
  ctx.strokeRect(10, 10, 100, 100);
}

function antAnime() {
  offset = (offset + 1) % 24;
  drawAnt();
  timer = setTimeout(antAnime, 100);
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_4_4_4.png)

### 4.5 渐变颜色

`createLinearGradient` 创建线性渐变

```js
function drawLinearGradient() {
  // Create gradients
  var lingrad = ctx.createLinearGradient(0, 0, 0, 150);
  lingrad.addColorStop(0, '#00ABEB');
  lingrad.addColorStop(0.5, '#fff');
  lingrad.addColorStop(0.5, '#26C000');
  lingrad.addColorStop(1, '#fff');

  var lingrad2 = ctx.createLinearGradient(0, 50, 0, 95);
  lingrad2.addColorStop(0.5, '#000');
  lingrad2.addColorStop(1, 'rgba(0,0,0,0)');

  // assign gradients to fill and stroke styles
  ctx.fillStyle = lingrad;
  ctx.strokeStyle = lingrad2;

  // draw shapes
  ctx.fillRect(10, 10, 130, 130);
  ctx.strokeRect(50, 50, 50, 50);
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_4_5_1.png)

`createRadialGradient` 创建辐射渐变

```js
function drawRadialGradient() {
  // 创建渐变
  var radgrad = ctx.createRadialGradient(45, 45, 10, 52, 50, 30);
  radgrad.addColorStop(0, '#A7D30C');
  radgrad.addColorStop(0.9, '#019F62');
  radgrad.addColorStop(1, 'rgba(1,159,98,0)');

  var radgrad2 = ctx.createRadialGradient(105, 105, 20, 112, 120, 50);
  radgrad2.addColorStop(0, '#FF5F98');
  radgrad2.addColorStop(0.75, '#FF0188');
  radgrad2.addColorStop(1, 'rgba(255,1,136,0)');

  var radgrad3 = ctx.createRadialGradient(95, 15, 15, 102, 20, 40);
  radgrad3.addColorStop(0, '#00C9FF');
  radgrad3.addColorStop(0.8, '#00B5E2');
  radgrad3.addColorStop(1, 'rgba(0,201,255,0)');

  var radgrad4 = ctx.createRadialGradient(0, 150, 50, 0, 140, 90);
  radgrad4.addColorStop(0, '#F4F201');
  radgrad4.addColorStop(0.8, '#E4C700');
  radgrad4.addColorStop(1, 'rgba(228,199,0,0)');

  const radgrad5 = ctx.createRadialGradient(75, 75, 5, 75, 75, 25);
  radgrad5.addColorStop(0, '#000');
  radgrad5.addColorStop(0.7, '#777');
  radgrad5.addColorStop(1, '#fff');

  // 画图形
  ctx.fillStyle = radgrad5;
  ctx.fillRect(0, 0, 150, 150);
  ctx.fillStyle = radgrad4;
  ctx.fillRect(0, 0, 150, 150);
  ctx.fillStyle = radgrad3;
  ctx.fillRect(0, 0, 150, 150);
  ctx.fillStyle = radgrad;
  ctx.fillRect(0, 0, 150, 150);
  ctx.fillStyle = radgrad2;
  ctx.fillRect(0, 0, 150, 150);
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_4_5_2.png)

### 4.6 绘制阴影

```js
function drawShadow() {
  ctx.shadowOffsetX = 2;
  ctx.shadowOffsetY = 2;
  ctx.shadowBlur = 2;
  ctx.shadowColor = 'rgba(0, 0, 0, 0.5)';

  ctx.fillStyle = 'steelblue';
  ctx.fillRect(100, 100, 50, 30);
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_4_6.png)

## 5. 绘制文字

### 5.1 填充/描边文字

```js
function drawText() {
  ctx.font = '48px serif';
  ctx.fillText('Hello world', 10, 50);
  ctx.strokeText('Hello world', 10, 100);
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_5_1.png)

### 5.2 文字垂直基准线

```js
function drawTextBaseline() {
  ctx.font = '48px serif';

  const lines = [
    { y: 10, baseline: 'top' },
    { y: 60, baseline: 'hanging' },
    { y: 110, baseline: 'middle' },
    { y: 160, baseline: 'alphabetic' },
    { y: 210, baseline: 'ideographic' },
    { y: 260, baseline: 'bottom' },
  ];

  lines.forEach(({ y, baseline }, i) => {
    ctx.beginPath();
    ctx.moveTo(0, y);
    ctx.lineTo(250, y);
    ctx.stroke();

    ctx.textBaseline = baseline;
    ctx.strokeText('Hello world' + (i + 1), 0, y);
  });
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_5_2.png)

### 5.3 测量文字宽度

需要注意的是调用 `measureText` 之前需要明确绘制样式，否则测量后才修改绘制样式所计算的宽度是不一样的

```js
function drawMeasureText() {
  const text = 'foo';
  ctx.font = '48px serif';
  const measurement = ctx.measureText(text); // TextMetrics object
  console.log('measurement', measurement);
  ctx.fillRect(10, 100, measurement.width, 5);
  ctx.fillText(text, 10, 100);
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_5_3.png)

## 6. 绘制变形

需要注意的是，这里的变形并不是将整个画布变形，而是对当前的绘制坐标系进行变形。

如果需要对当前画布整体变形可以使用 CSS 的 `transform` 属性。

### 6.1 保存/恢复状态

```js
function* drawState() {
  ctx.fillRect(0, 0, 150, 150); // 使用默认设置绘制一个矩形
  ctx.save(); // 保存默认状态

  yield;

  ctx.fillStyle = '#09F'; // 在原有配置基础上对颜色做改变
  ctx.fillRect(15, 15, 120, 120); // 使用新的设置绘制一个矩形

  yield;

  ctx.save(); // 保存当前状态
  ctx.fillStyle = '#FFF'; // 再次改变颜色配置
  ctx.globalAlpha = 0.5;
  ctx.fillRect(30, 30, 90, 90); // 使用新的配置绘制一个矩形

  yield;

  ctx.restore(); // 重新加载之前的颜色状态
  ctx.fillRect(45, 45, 60, 60); // 使用上一次的配置绘制一个矩形

  yield;

  ctx.restore(); // 加载默认颜色配置
  ctx.fillRect(60, 60, 30, 30); // 使用加载的配置绘制一个矩形
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_6_1.png)

### 6.2 平移

```js
function* drawTranslate() {
  for (var i = 0; i < 3; i++) {
    for (var j = 0; j < 3; j++) {
      ctx.save();
      ctx.fillStyle = 'rgb(' + 51 * i + ', ' + (255 - 51 * i) + ', 255)';
      ctx.translate(10 + j * 50, 10 + i * 50);
      ctx.fillRect(0, 0, 25, 25);
      ctx.restore();

      yield [i, j];
    }
  }
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_6_2.png)

### 6.3 旋转

```js
function drawRotate() {
  const width = 300;

  ctx.save();
  ctx.translate(width / 2, width / 2);

  // 遍历每一层环
  for (var i = 1; i < 6; i++) {
    // Loop through rings (from inside to out)
    ctx.save();
    const R = 51 * i;
    const G = 255 - R;
    ctx.fillStyle = `rgb(${R},${G},255)`;

    // 遍历环上的点
    for (var j = 0; j < i * 6; j++) {
      // draw individual dots
      ctx.rotate((Math.PI * 2) / (i * 6));
      ctx.beginPath();
      ctx.arc(0, i * 25, 10, 0, Math.PI * 2, true);
      ctx.fill();
    }

    ctx.restore();
  }
  ctx.restore();
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_6_3.png)

### 6.4 缩放

缩放使用 -1 的时候就好像是对某个坐标轴进行镜像映射一般

```js
function drawScale() {
  // draw a simple rectangle, but scale it.
  ctx.save();
  ctx.scale(10, 3);
  ctx.fillRect(1, 10, 10, 10);
  ctx.restore();

  // mirror horizontally
  ctx.save();
  ctx.scale(-1, 1);
  ctx.font = '48px serif';
  ctx.fillText('MDN', -135, 120);
  ctx.restore();
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_6_4.png)

### 6.5 变形转移矩阵

```js
function drawTransform() {
  var sin30 = Math.sin(Math.PI / 6);
  var cos30 = Math.cos(Math.PI / 6);

  ctx.save();
  ctx.translate(100, 100);
  var c = 0;
  for (var i = 0; i <= 12; i++) {
    ctx.fillStyle = `rgba(0,0,0,${1 - i / 12})`;
    ctx.fillRect(0, 0, 100, 10);
    ctx.transform(cos30, sin30, -sin30, cos30, 0, 0);
  }

  ctx.setTransform(-1, 0, 0, 1, 100, 100);
  ctx.fillStyle = 'rgba(255, 128, 255, 0.5)';
  ctx.fillRect(0, 50, 100, 100);
  ctx.restore();
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_6_5.png)

## 7. 图像操作/像素操作

```js
function loadImg(src, width, height) {
  return new Promise((resolve, reject) => {
    const img = new Image();
    width && (img.width = width);
    height && (img.height = height);
    img.onload = function () {
      resolve(img);
    };
    img.onerror = function (err) {
      reject(err);
    };
    img.src = src;
  });
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_7.png)

## 8. 实践应用

API 实在太多了，需要一个个去试才能真正掌握，跟着 MDN 的教程其实非常不错，下面有几个 MDN 上的例子也都可以看看

1. 颜色选择器：[https://github.com/superfreeeee/Blog-code/blob/main/front_end/html/canvas_insight_2d_api/colorPicker/index.html](https://github.com/superfreeeee/Blog-code/blob/main/front_end/html/canvas_insight_2d_api/colorPicker/index.html)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_8_1.png)

2. 鼠标跟踪：[https://github.com/superfreeeee/Blog-code/blob/main/front_end/html/canvas_insight_2d_api/mouse/mine.html](https://github.com/superfreeeee/Blog-code/blob/main/front_end/html/canvas_insight_2d_api/mouse/mine.html)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_8_2.png)

3. 贪吃蛇(带自动跟踪外挂hh)：[https://github.com/superfreeeee/Blog-code/blob/main/front_end/html/canvas_insight_2d_api/snake/snake.html](https://github.com/superfreeeee/Blog-code/blob/main/front_end/html/canvas_insight_2d_api/snake/snake.html)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/canvas_insight_2d_api_8_3.png)

# 其他资源

## 参考连接

| Title                 | Link                                                                                                                                         |
| --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| Canvas教程 - MDN      | [https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial) |
| HTML CANVAS DEEP DIVE | [https://joshondesign.com/p/books/canvasdeepdive/toc.html](https://joshondesign.com/p/books/canvasdeepdive/toc.html)                         |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/html/canvas_insight_2d_api](https://github.com/superfreeeee/Blog-code/tree/main/front_end/html/canvas_insight_2d_api)
