# Express 实战: 使用 express-static 处理静态资源

@[TOC](文章目录)

<!-- TOC -->

- [Express 实战: 使用 express-static 处理静态资源](#express-实战-使用-express-static-处理静态资源)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [一切的起点](#一切的起点)
  - [项目结构 & 初始化](#项目结构--初始化)
  - [静态资源服务器](#静态资源服务器)
  - [浏览器支持 ES6 模块化](#浏览器支持-es6-模块化)
- [结语](#结语)

<!-- /TOC -->

## 简介

本篇的出发点在于一直想测试 HTML 提供的 `<script type="module"></script>` 的功能，但是如果直接使用 `files` 协议访问本地的 html 文件，是不支持跨域 cors 请求的，在下面正文中会说明。

因此本篇介绍如何使用 express + express-static 直接实现一个简单的静态资源分发的简易前端服务器，马上来看看。

## 参考

<table>
  <tr>
    <td>node.js express 返回一个静态页面</td>
    <td><a href="https://blog.csdn.net/purple_lumpy/article/details/88569478">https://blog.csdn.net/purple_lumpy/article/details/88569478</a></td>
  </tr>
</table>

# 正文

## 一切的起点

会诞生这一篇主要是在于作者研究原生的 html 引入 js 的机制时遇到的问题，原来我们想引入外部 js 文件可能会这样写：

- `index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <div id="app">Hello World</div>

    <script src="index.js"></script>
</body>
</html>
```

然而这样引入的 js 文件仅仅只是一个单一的脚本，后来在网上查到 HTML 原生可以透过设置 `type="module"` 来引入 ES6 的模块化 js 文件，所以改成下面这样：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <div id="app">Hello World</div>

    <script src="index.js" type="module"></script>
</body>
</html>
```

但是浏览器不高兴了，由于我们直接访问本地 html 默认使用的是 `file` 协议，不支持跨域 CORS，如下面这样：

![](https://picures.oss-cn-beijing.aliyuncs.com/img/20201212160832.png)

最后我决定透过自己构建基于 express 的静态资源服务器来返回本地文件（对有点绕，不过也是模拟前端服务器的一种手段之一）

## 项目结构 & 初始化

首先先给出整个项目的结构：

```
/express-static
|- node_module
|- src/
    |- index.html
    |- index.js
|- package.json
|- yarn.lock
|- server.js
```

- 使用 `yarn` 作为包管理工具
- `server.js` 为服务器设置
- `src/` 目录为源码位置，即我们将要分发的静态资源

> 初始化项目并引入依赖

```bash
$ mkdir express-static
$ cd express-static
$ yarn init -y
$ yarn add express express-static
```

> 启动配置

- `package.json`

```jsonc
{
    //...
    "scripts": {
        "start": "node server.js"
    }
    //...
}
```

> 静态文件内容

我们先给出静态文件内容（本篇重点放在 express-static 的使用）

- `src/index.html`：引入 `index.js` 并设置 `type="module"`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>script-module</title>
    <link rel="icon" href="favicon32.ico">
</head>
<body>
    <div id="app">Sample</div>

    <script src="index.js" type="module"></script>
</body>
</html>
```

- `src/index.js`

```js
console.log('load index.js')
```

## 静态资源服务器

最后我们加入 `server.js` 的内容完成静态资源服务器的创建

- `server.js`

```js
const express = require('express')
const expressStatic = require('express-static')

const app = express()

// 将 src 目录下的文件作为静态资源向外暴露
app.use(expressStatic('./src'))

const port = 3000

app.listen(port, () => {
  console.log(`server listen at: http://localhost:${port}`)
})
```

使用 `express-static` 的核心就是 `app.use(expressStatic('./src'))` 这一行，传入你想暴露的静态资源目录路径即可

接下来启动服务器

```bash
$ yarn start
```

并访问 `http://localhost:3000`（默认寻找 `index.html` 文件），得到如下结果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/20201212162750.png)

> 浏览器 & 服务器交互顺序

浏览器访问 `http://localhost:3000` 后的交互顺序如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/express_static_access_orders.png)

## 浏览器支持 ES6 模块化

好了建好了极简的服务器，终于可以来用用看 ES6 的模块化机制了，相关特性可以查看<a href="https://blog.csdn.net/weixin_44691608/article/details/106628905">ES6特性：Module模塊化</a>

我们在 `src/` 下加入新的 js 文件 `tools.js` 作为新的模块并修改 `index.js` 文件内容

- 项目结构

```
/express-static
|- node_module
|- src/
    |- index.html
    |- index.js
    |- tools.js
|- package.json
|- yarn.lock
|- server.js
```

- `src/tools.js`

```js
export function f () {
  console.log('invoke function f')
}

export function g () {
  console.log('invoke function g')
}

export default function h () {
  console.log('invoke default function h')
}
```

- `src/index.js`

```js
console.log('load index.js')

import h, { f, g } from './tools.js'
console.log(f)
console.log(g)
console.log(h)
f()
g()
h()
```

运行结果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/express_static_access_sample2.png)

大功告成！

# 结语

本来是想着来测试测试浏览器中能不能直接使用 ES6 模块，本来以为加个 `type="module"` 就完事了，没想到 CORS 出来搞事，最后引出这一篇的诞生，供大家参考hhh。
