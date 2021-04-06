# JS 前端路由：单页面应用的路由原理和实现

@[TOC](文章目录)

<!-- TOC -->

- [JS 前端路由：单页面应用的路由原理和实现](#js-前端路由单页面应用的路由原理和实现)
  - [简介](#简介)
  - [参考](#参考)
  - [完整示例代码](#完整示例代码)
- [正文](#正文)
  - [从页面刷新到路由跳转](#从页面刷新到路由跳转)
  - [单页面应用的优劣](#单页面应用的优劣)
  - [前端路由分类](#前端路由分类)
  - [前端路由实现](#前端路由实现)
    - [项目架构](#项目架构)
    - [Hash 实现](#hash-实现)
    - [History 实现](#history-实现)
- [结语](#结语)

<!-- /TOC -->

## 简介

在浏览器的默认行为中，不同的 url 就对应着不同的资源。这就表示当我们使用路径来区别一个网页的不同页面的时候，实际上浏览器就必须从服务端将不同页面加载下来解析然后展示，而这也是传统页面的做法。

下面我们来介绍一种当前更流行，并将异步交互体验发挥到极致的网页：**单页面应用(SPA = Single Page Application)**，以及作为实现基础的 **前端路由** 技巧。

## 参考

<table>
  <tr>
    <td>前端路由原理解析和实现</td>
    <td><a href="https://www.cnblogs.com/lguow/p/10921564.html">https://www.cnblogs.com/lguow/p/10921564.html</a></td>
  </tr>
  <tr>
    <td>什么是前端路由？</td>
    <td><a href="https://blog.csdn.net/weixin_40851188/article/details/90377025?utm_source=app&app_version=4.5.7">https://blog.csdn.net/weixin_40851188/article/details/90377025?utm_source=app&app_version=4.5.7</a></td>
  </tr>
  <tr>
    <td>react-router V4中三种router区别？</td>
    <td><a href="https://www.zhihu.com/question/63662664?sort=created">https://www.zhihu.com/question/63662664?sort=created</a></td>
  </tr>
  <tr>
    <td>浅谈前端SPA（单页面应用）</td>
    <td><a href="https://blog.csdn.net/cmzhuang/article/details/94334619">https://blog.csdn.net/cmzhuang/article/details/94334619</a></td>
  </tr>
  <tr>
    <td>SPA 单页面应用</td>
    <td><a href="https://www.cnblogs.com/thonrt/p/5995856.html">https://www.cnblogs.com/thonrt/p/5995856.html</a></td>
  </tr>
</table>

## 完整示例代码

<a href="https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_router">https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_router</a>

# 正文

## 从页面刷新到路由跳转

我们在简介中提到，传统网页的每个分页(每一个单独的 url)都对应一个独立的页面，即便他们都属于同一个网页，在页面跳转时都必须重新向服务器请求页面，这其实极大的影响用户体验：每次换页都会产生短暂的白屏和请求等待。

AJAX 由此而诞生，透过浏览器后台异步发请求并将响应数据对页面进行部分刷新，透过异步请求和 DOM 操作，我们可以实现不刷新页面只更新数据的 web 交互模式。但是这样还是没有解决分页之间的页面刷新问题。

后来 **前端路由** 技术的出现彻底解决多页面之间的刷新问题，同时用此技术设计的网页被称为 **单页面应用(SPA)**。用户将在首次访问网页的时候，一次拿到整个页面的完整数据，并依赖后台对页面跳转行为的拦截、阻断，将默认的页面刷新行为变成 JS 脚本动态的部分刷新页面。

## 单页面应用的优劣

前面提到单页面应用解决了页面刷新的体验问题，但是反过来说第一次请求页面的负担增加了，另外其实还会造成其他的副作用，下面列出单页面应用的优缺点：

- 优点
  - 提升交互体验：撇除初次加载的弊端来看，用户在访问网页的整个交互流程中，是有 JS 脚本来响应并处理用户的请求，比起发起网络请求要来的敏捷迅速，进而提升用户的交互体验
  - 前后端分离：所有页面的跳转/路由逻辑都交由 JS 来负责，就能够实现彻底的前后端分离架构，后端使用 RESTful 风格提供纯粹的服务接口，而 SPA 应用发起 AJAX 请求并全权负责数据的展示和页面的跳转。
  - 状态的维持：用户使用页面的时候总是会进行一系列的操作，如用户注册/登入、页面跳转等逻辑，但是在传统的多页面场景之下，页面见的跳转就会重新解析文档，同时用户的状态就丢失了，需要进行额外的前后端交互才能维持用户的操作状态；而 SPA 都是由 JS 来负责页面的转换，也就不存在状态的丢失。
- 缺点
  - SEO 搜索引擎优化：由于 SPA 页面的内容都保存在 JS 代码中同时也由 JS 来负责视图选择，所以实际上对于浏览器来说静态页面的部分极少、或是说静态页面内容是不可见的。这样对于搜索引擎的页面爬虫是非常不友好的，没办法获取足够的信息来认识页面的内容，同时也就没办法对页面进行正确的搜索推荐。
  - 首屏加载：由于 SPA 应用相当于是将所有页面作为 JS 脚本加入到首屏页面当中，理所当然就会造成首页面过于肥大，差生白屏、延迟等严重影响用户体验观感的问题。

## 前端路由分类

下面我们就来谈谈前端路由的实现方式，本文参考 Vue、React 等框架的实现方式总结出几种前端路由的实现方案

| 实现方案(模式) | 核心 API/事件     | 应用场景                  |
| -------------- | ----------------- | ------------------------- |
| Hash           | hashchange        | 网页                      |
| History        | History、popstate | 网页                      |
| memory         | 独立实现          | 移动端(无明显路由/无路由) |
| static         | 独立实现          | SSR 服务端渲染            |

- Hash 模式

    Hash 模式主要是利用 url 格式中一种特别的标记称为 **锚点(anchor)**，一般的 url 中 `#` 符号后的内容就是锚点的标记目标。原来锚点的作用是用于定位网页中特定元素的方法，这里利用锚点来记录页面下的路由。

    利用页面锚点改变的时候触发的 `hashchange` 事件来动态的更新页面 **预设点**(例如 Vue 中的 `<router-view>` 标签)的内容。

- History 模式

    History 模式则是利用了 HTML5 提供的 **History API**。在 BOM 模型下的 History 对象新增 `pushState`、`replaceState` 等 API，允许页面透过脚本操作浏览器的浏览记录而不需要真正的进行页面请求/刷新。

    除了 History API 之外，浏览器的可视化按钮还会触发 `popstate` 操作，也是 History 模式实现路由的时候需要进行拦截的路由点。

- memory 模式

    memory 模式的意思是在内存中独立维护一个浏览记录的栈，从页面的跳转到映射为记录栈的操作，都由应用的后台脚本独立维护。通常常见与移动端等不存在真实路由的场景，以隐式路由的方式管理不同页面之间的协作也有利于应用逻辑的梳理。

- static 模式

    静态路由的场景出现在服务端渲染的架构之下，本文不加以讨论。

## 前端路由实现

本篇选择 Hash 模式、History 模式，使用原生 js 来实现两个前端路由的模式。

### 项目架构

首先给出整个测试项目的架构和一些基础内容

- 目录结构

```tree
js-router
|- node_modules/
|- public/
   |- hash/     # hash 模式
      |- index.html
      |- HashRouter.js
   |- history/  # history 模式
      |- index.html
      |- HistoryRouter.js
|- src/
   |- app.js
|- package.json
|- yarn.lock
```

- express 实现静态服务器

```js
const express = require('express')
const app = express()

app.use(express.static('public'))
/* hash 模式使用目录 */
app.use(express.static('public/hash'))
/* history 模式使用目录 */
// app.use(express.static('public/history'))

const port = 3000

app.listen(port, () => {
  console.log(`server listen at http://localhost:${port}`)
})
```

### Hash 实现

- 模式流程图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_router_hash_states.png)

- index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>JS Router-hash</title>
    <style>
      body {
        margin: 0;
        padding: 0 40px;
      }
    </style>
  </head>
  <body>
    <h1>前端路由实现 - Hash 实现</h1>
    <ul>
      <li><a href="#/">default</a></li>
      <li><a href="#/home">home</a></li>
      <li><a href="#/about">about</a></li>
    </ul>

    <h3 id="router-view">default content</h3>

    <script src="routes.js"></script>
    <script src="HashRouter.js"></script>
  </body>
</html>
```

- HashRouter.js

```js
class HashRouter {
  constructor(routes) {
    this.el = document.querySelector('#router-view')
    this.init(routes)
  }

  init(routes) {
    console.log('HashRouter init')
    const onChange = this.change.bind(this)
    // reset hash when reload
    const old = window.onload
    window.onload = function () {
      onChange()
      window.onload = old
    }
    // create page mapper
    const mapper = {}
    for (const route of routes) {
      mapper[route.path] = route
    }
    this.mapper = mapper
    // add hashchange listener
    window.addEventListener('hashchange', onChange)
  }

  change() {
    const hash = location.hash
    const path = hash ? hash.substring(1) : '/'
    console.log(`hash path: ${path}`)
    this.el.innerHTML = this.mapper[path].content
  }
}

const router = new HashRouter(routes)
console.log(router)
```

我们可以看到整段脚本就是建立一个 `HashRouter` 的实例。整段代码的核心在于：

```js
window.addEventListener('hashchange', onChange)
```

`onChange` 方法是对于路由改变后的页面更新，这边直接改变 innerHTML 作示例，代码在 `init` 方法中利用 `load` 事件先对首次访问/页面刷新进行拦截检验，后面在 window 对象加上对 `hashchange` 事件的处理来实现前端路由的跳转。

- 实现效果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_router_hash1.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_router_hash2.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_router_hash3.png)

我们可以看到点击不同链接之后改变 url 锚点，进而触发页面组件的更新

### History 实现

- 模式流程图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_router_history_states.png)

- index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>JS Router-history</title>
    <style>
      body {
        margin: 0;
        padding: 0 40px;
      }
    </style>
  </head>
  <body>
    <h1>前端路由实现 - History 实现</h1>
    <ul>
      <li><a href="/">default</a></li>
      <li><a href="/home">home</a></li>
      <li><a href="/about">about</a></li>
    </ul>

    <h3 id="router-view">default content</h3>

    <script src="routes.js"></script>
    <script src="HistoryRouter.js"></script>
  </body>
</html>
```

- HistoryRouter.js

```js
class HistoryRouter {
  constructor(routes) {
    this.el = document.querySelector('#router-view')
    this.init(routes)
  }

  init(routes) {
    console.log('HistoryRouter init')
    const onChange = this.change.bind(this)
    // reset hash when reload
    const old = window.onload
    window.onload = function () {
      onChange()
      const linkList = document.querySelectorAll('a[href]')
      linkList.forEach((link) => {
        link.addEventListener('click', (e) => {
          e.preventDefault()
          history.pushState(null, '', link.getAttribute('href'))
          onChange()
        })
      })
      window.onload = old
    }
    // create page mapper
    const mapper = {}
    for (const route of routes) {
      mapper[route.path] = route
    }
    this.mapper = mapper
    // add hashchange listener
    window.addEventListener('popstate', onChange)
  }

  change() {
    const path = location.pathname
    console.log(`hash path: ${path}`)
    this.el.innerHTML = this.mapper[path].content
  }
}

const router = new HistoryRouter(routes)
console.log(router)
```

与 hash 模式类似，对首次访问的路由进行更新(`change` 方法)，为 `popstate` 添加监听函数(hash 模式监听的是 `hashchange`，而 history 模式透过 `popstate` 来监听 url 路径 `pathname` 的改变)；比较特别的是 History 模式还要阻断页面上所有 `<a>` 标签的默认跳转行为，改为使用 `pushState` 来避免页面的刷新和重新加载

- 实现效果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_router_history1.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_router_history2.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_router_history3.png)

# 结语

前端路由是一个简单但是非常重要的知识点，他是与用户操作直接相关的核心部分，也是整个 SPA 应用的实现基础，同时将页面的跳转缩小化成为一个 JS 状态的转移。

本篇实现了前端路由最最基础的部分(仅仅只有状态的更新 `pushState` 之类的)，其他还有像是浏览器的前进/后端，历史记录是进行 push 还是 replace 等根据不同操作进行状态管理的区分，这就是各大框架提供的 Router 库在做的事(如 `vue-router`、`react-router`、...)。
