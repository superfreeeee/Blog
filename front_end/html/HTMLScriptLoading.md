# `<script>` 标签的加载和执行时机

@[TOC](文章目录)

<!-- TOC -->

- [`<script>` 标签的加载和执行时机](#script-标签的加载和执行时机)
  - [简介](#简介)
  - [参考](#参考)
  - [完整示例代码](#完整示例代码)
- [正文](#正文)
  - [`<script>` 标签的 `defer`、`async` 属性](#script-标签的-deferasync-属性)
    - [1. 简单引入(不添加任何额外属性)：`<script>`](#1-简单引入不添加任何额外属性script)
    - [2. 使用 `defer` 属性：`<script defer>`](#2-使用-defer-属性script-defer)
    - [3. 使用 `async` 属性：`<script async>`](#3-使用-async-属性script-async)
    - [三种使用方式小结](#三种使用方式小结)
  - [代码实践测试](#代码实践测试)
    - [简单引入：`<script>`](#简单引入script)
      - [`DOMContentLoaded` 与 `load` 事件](#domcontentloaded-与-load-事件)
    - [`defer` 属性：`<script defer>`](#defer-属性script-defer)
    - [`async` 属性：`<script async>`](#async-属性script-async)
- [结语：到底什么时候用？](#结语到底什么时候用)
  - [决策条件(使用时机)](#决策条件使用时机)

<!-- /TOC -->

## 简介

在浏览器的运行环下，我们知道要想在网页中插入脚本，我们可以使用 `<script>` 标签来选择引入内联脚本/外部文件脚本的方式来为网页插入 js。但是脚本的加载和执行时机其实是与 HTML 文件的解析紧密关联的，本篇将详细介绍使用 `<script>` 标签时关于脚本加载的 `defer`、`async` 属性之间的区别。 

## 参考

<table>
  <tr>
    <td>defer和async的区别</td>
    <td><a href="https://segmentfault.com/q/1010000000640869">https://segmentfault.com/q/1010000000640869</a></td>
  </tr>
  <tr>
    <td>DOMContentLoaded与load的区别</td>
    <td><a href="https://segmentfault.com/a/1190000019249783">https://segmentfault.com/a/1190000019249783</a></td>
  </tr>
  <tr>
    <td>[JS]DOMContentLoaded和load的区别</td>
    <td><a href="https://blog.csdn.net/weixin_41650504/article/details/89379947">https://blog.csdn.net/weixin_41650504/article/details/89379947</a></td>
  </tr>
</table>

## 完整示例代码

<a href="https://github.com/superfreeeee/Blog-code/tree/main/front_end/html/html_script_loading">https://github.com/superfreeeee/Blog-code/tree/main/front_end/html/html_script_loading</a>

# 正文

## `<script>` 标签的 `defer`、`async` 属性

首先我们先从宏观的角度来说明 `<script>` 标签的属性意义，主要差别在于 `defer`、`async` 两个属性的使用，具体分成三种情况：

### 1. 简单引入(不添加任何额外属性)：`<script>`

HTML 解析的过程当中，内联脚本(脚本直接写在 `<script>` 标签之中)，和简单引入的外部脚本都会阻塞 HTML 文件的解析，等待脚本的获取(透过网路)、执行结束之后，才会继续 HTML 脚本的解析。所以执行顺序如下：

HTML 解析 $\to$ 发现 `<script>` 标签，阻塞 HTML 解析 $\to$ (非内联脚本)从网络获取外部脚本 $\to$ 执行 js 脚本 $\to$ 继续 HTML 解析

### 2. 使用 `defer` 属性：`<script defer>`

如果我们在 `<script>` 标签上使用了 `defer` 属性，那这个脚本就会被视为异步脚本，它的脚本加载将会与 HTML 解析并行，并在 HTML 完全解析完毕后执行脚本。实际的执行顺序如下：

HTML 解析 $\to$ 发现 `<script defer>` 标签，**不** 阻塞 HTML 解析 $\to$ 异步加载外部脚本 $\to$ 直到 HTML 解析完毕后执行 js 脚本

### 3. 使用 `async` 属性：`<script async>`

另一种情况我们选择使用 `async` 属性，与 `defer` 相似的是脚本的加载都是异步的，也就是与 HTML 解析并行的，差异在于 `async` 加载的脚本会在加载完毕后立即执行，也就是下列顺序：

HTML 解析 $\to$ 发现 `<script async>` 标签，**不** 阻塞 HTML 解析 $\to$ 异步加载外部脚本 $\to$ 当外部脚本加载完毕之后，立即阻塞并执行 js 脚本 $\to$ 脚本执行完毕之后继续未完的 HTML 解析

### 三种使用方式小结

看完三种使用 `<script>` 标签的方式后，这边给出一张简图来代表三种情况下 HTML 和 js 脚本的顺序关系(这边通常指的是外部文件方式引入的脚本)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/html_script_loading_comparison.png)

- 脚本加载：使用 `defer` 和 `async` 时脚本的加载(蓝色线段)能够与 HTML 解析(绿色线段)并行，也就是异步加载脚本
- 脚本执行：`defer` 脚本会在 HTML 解析完毕之后执行，而 `async` 则会在加载完毕之后立马阻塞 HTML 解析并开始执行

## 代码实践测试

光说不练，下面我们实际写几个简单的文件来检验上面解析、加载、执行的规则

### 简单引入：`<script>`

第一种情况是使用最基本的情况引入脚本，分别在 `<head>` 中引入内联脚本，并在 `<body>` 的最后引入一个外部脚本。

- `basic.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <script>
      document.addEventListener('DOMContentLoaded', () => {
        console.log('on DOMContentLoaded')
      })
      window.addEventListener('load', () => {
        console.log('on load')
      })
    </script>
  </head>
  <body>
    <h2>using script basic</h2>

    <script src="basic.js"></script>
  </body>
</html>
```

- `basic.js`

```js
console.log('loading script using basic')
```

- 加载顺序

![](https://picures.oss-cn-beijing.aliyuncs.com/img/html_script_loading_basic_performance.png)

在简单引入的情况下，我们可以看到在 main 标签下面的蓝色线段就是 HTML 解析的部分，顺序正好是：

1. HTML 文件的加载(network 下的蓝色线段)
2. HTML 文件的解析(main 下的蓝色线段)
3. js 文件的加载(network 下的黄色线段)
4. js 文件的执行(main 下的黄色线段)
5. HTML 的解析(main 下的蓝色线段)

是的与上面所说到的情况相符合；值得一提的是，在 Timings 标签下有好几个标签(时间点)：

- `L`： onLoad Event
- `DCL`：DOMContentLoaded Event
- `FP`：First Paint
- `FCP`：First Contentful Paint
- `LCP`：Largest Contentful Paint

这边将不会展开说明各个指标的意义，后续作者会再其他篇章详细说明。

#### `DOMContentLoaded` 与 `load` 事件

上面的指标中值得一提的是 `document.DOMContentLoaded` 和 `window.load` 两个事件

- `document.DOMContentLoaded` 代表的是当前 HTML 文档内容准备完毕的意思
- `window.load` 代表包括 HTML 文档、外部资源/脚本都加载准备完毕的意思

还记得刚刚提过在默认(简单引入)的情况下，`<script>` 脚本就好像一般的 HTML 标签，会被同步的加载、执行，然后继续 HTML 文档的解析，所以三个事件的顺序就是：

js 脚本执行 $\to$ `document.DOMContentLoaded` 事件 $\to$ `window.load` 事件

- 控制台输出

![](https://picures.oss-cn-beijing.aliyuncs.com/img/html_script_loading_basic_console.png)


### `defer` 属性：`<script defer>`

第二种我们在脚本标签加入 `defer` 属性

- `defer.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <script>
      document.addEventListener('DOMContentLoaded', () => {
        console.log('on DOMContentLoaded')
      })
      window.addEventListener('load', () => {
        console.log('on load')
      })
    </script>
  </head>
  <body>
    <h2>using script with defer</h2>

    <script defer src="defer.js"></script>
  </body>
</html>
```

- `defer.js`

```js
console.log('loading script using defer')
```

- 加载顺序

![](https://picures.oss-cn-beijing.aliyuncs.com/img/html_script_loading_defer_performance.png)

我们一样可以从 network 和 main 标签的蓝黄色看出文档解析和脚本加载的前后顺序。

由于使用了 `defer` 属性，js 文件的加载不再阻塞 HTML 文档的解析，但是 `defer` 属性又指定必须在 HTML 解析完毕而 `DOMContentLoaded` 事件之前执行，所以 DCL 事件的出现会在 `defer.js` 的脚本执行完毕之后

- 控制台输出

![](https://picures.oss-cn-beijing.aliyuncs.com/img/html_script_loading_defer_console.png)

事件顺序为：`defer.js` 脚本的执行 $\to$ `document.DOMContentLoaded` 事件 $\to$ `window.load` 事件

### `async` 属性：`<script async>`

最后一种使用了 `async` 属性来加载脚本

- `async.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <script>
      document.addEventListener('DOMContentLoaded', () => {
        console.log('on DOMContentLoaded')
      })
      window.addEventListener('load', () => {
        console.log('on load')
      })
    </script>
  </head>
  <body>
    <h2>using script with async</h2>

    <script async src="async.js"></script>
  </body>
</html>
```

- `async.js`

```js
console.log('loading script using async')
```

- 加载顺序

![](https://picures.oss-cn-beijing.aliyuncs.com/img/html_script_loading_async_performance.png)

前面提过 `async` 关键字所代表的意思是完全的异步，也就是说 `async.js` 被视为与 HTML 的解析完全不相关的脚本，所以我们看到在脚本执行甚至加载之前，DCL 事件早早就被触发了。

- 控制台输出

![](https://picures.oss-cn-beijing.aliyuncs.com/img/html_script_loading_async_console.png)

事件顺序：`document.DOMContentLoaded` 事件 $\to$ `async.js` 脚本的执行 $\to$ `window.load` 事件

注意：`load` 事件指的是所有外部资源/外部脚本加载 & 执行完毕之后，所以理所当然必须在 `async.js` 脚本执行完毕之后

# 结语：到底什么时候用？

我们已经把三种情况都看了一遍，看起来好像懂了什么，最后剩下的问题就是：到底什么时候该用哪个？

## 决策条件(使用时机)

- `async` 属性：DOM 无关的脚本

由于 `async` 属性会与异步加载脚本并立即执行，所以脚本看见的 DOM 结构不一定是完整的，所以这时候我们应该只在这里放入 **与 DOM 无关的脚本**，同时也要是 **希望能及早执行** 的脚本，如一些 **环境兼容性的 polyfill**、**用户行为收集/分析** 等脚本

- `defer` 属性：DOM 的补全

`defer` 属性虽然也会异步加载脚本，但是他会等到初步的 HTML 文件解析完毕之后才执行脚本，所以这时候我们可以放入一些 **增强/补全 DOM 的脚本**(但是要小心不一定能依赖原有的 DOM 结构)，例如向 vue、react 等框架都完全透过 js 脚本来掺入页面，原有的 HTML 非常的小，我们也可以使用 `defer` 的方式来异步加载 bundle 后的脚本，以加快页面的渲染。

- 默认(简单引入)：DOM 相关脚本

最后就是默认的脚本引入，一些 DOM 操作或需要与 HTML 解析存在特定顺序关联的脚本就可以使用简单引用的方式。
