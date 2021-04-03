# Http 缓存: 强缓存与协商缓存

@[TOC](文章目录)

<!-- TOC -->

- [Http 缓存: 强缓存与协商缓存](#http-缓存-强缓存与协商缓存)
  - [简介](#简介)
  - [参考](#参考)
  - [完整示例代码](#完整示例代码)
- [正文](#正文)
  - [Http 缓存机制：强缓存 & 协商缓存](#http-缓存机制强缓存--协商缓存)
    - [强缓存：`Expires` & `Cache-Control`](#强缓存expires--cache-control)
      - [Http 1.0：`Expires`](#http-10expires)
      - [Http 1.1：`Cache-Control`](#http-11cache-control)
    - [协商缓存：`Last-Modified/If-Modified-Since` & `Etag/If-None-Match`](#协商缓存last-modifiedif-modified-since--etagif-none-match)
      - [Http 1.0：`Last-Modified/If-Modified-Since`](#http-10last-modifiedif-modified-since)
      - [Http 1.1：`Etag/If-None-Match`](#http-11etagif-none-match)
  - [Http 缓存实验](#http-缓存实验)
    - [项目结构 & 基础代码](#项目结构--基础代码)
      - [Express Static 缓存配置](#express-static-缓存配置)
    - [默认无缓存](#默认无缓存)
    - [`Cache-Control` 测试(Http 1.1 强缓存实现)](#cache-control-测试http-11-强缓存实现)
    - [`Etag/If-None-Match` 测试(Http 1.1 协商缓存实现)](#etagif-none-match-测试http-11-协商缓存实现)
- [结语](#结语)

<!-- /TOC -->

## 简介

Http 协议是一个非常强大的应用层协议，我们日常生活中用到的大部分的网页几乎都是透过 Http(或是 Https) 协议提供的页面和服务。本篇将要来介绍在 Http 协议中提供的缓存机制(请求/响应头的设置)，并透过简单的 demo 页面来检验浏览器的实际反应。

## 参考

<table>
  <tr>
    <td>浏览器的缓存机制cache-control</td>
    <td><a href="https://blog.csdn.net/weixin_43801564/article/details/103050667">https://blog.csdn.net/weixin_43801564/article/details/103050667</a></td>
  </tr>
  <tr>
    <td>10分钟彻底搞懂Http的强制缓存和协商缓存</td>
    <td><a href="https://segmentfault.com/a/1190000016199807">https://segmentfault.com/a/1190000016199807</a></td>
  </tr>
  <tr>
    <td>HTTP 304状态码的详细讲解</td>
    <td><a href="https://blog.csdn.net/huwei2003/article/details/70139062">https://blog.csdn.net/huwei2003/article/details/70139062</a></td>
  </tr>
  <tr>
    <td>Express 中设置缓存Cache-control的maxAge</td>
    <td><a href="https://blog.csdn.net/liudabaozhangxiaobei/article/details/50956346">https://blog.csdn.net/liudabaozhangxiaobei/article/details/50956346</a></td>
  </tr>
  <tr>
    <td>【nodejs】express框架(一) express的使用以及express.static的使用，顺带解决一些服务器路径问题和浏览器解析过程</td>
    <td><a href="https://blog.csdn.net/weixin_42476799/article/details/89296873">https://blog.csdn.net/weixin_42476799/article/details/89296873</a></td>
  </tr>
  <tr>
    <td>前端性能优化 —— 添加Expires头</td>
    <td><a href="https://www.cnblogs.com/zhutao/p/6690198.html">https://www.cnblogs.com/zhutao/p/6690198.html</a></td>
  </tr>
  <tr>
    <td>ETag - HTTP from MDN</td>
    <td><a href="https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/ETag#Browser_compatibility">https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/ETag#Browser_compatibility</a></td>
  </tr>
  <tr>
    <td>If-None-Match - HTTP from MDN</td>
    <td><a href="https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/If-None-Match">https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/If-None-Match</a></td>
  </tr>
</table>

## 完整示例代码

<a href="https://github.com/superfreeeee/Blog-code/tree/main/others/network/http_cache">https://github.com/superfreeeee/Blog-code/tree/main/others/network/http_cache</a>

# 正文

## Http 缓存机制：强缓存 & 协商缓存

在 Http 协议中，缓存的规则分为 **强缓存** 和 **协商缓存** 两种。用一句话来说明的话就是：

- 强缓存：时限内 **不再向服务端请求资源**，而是直接使用浏览器缓存的资源
- 协商缓存：**每次访问服务端确定资源是否改变**，改变则返回新的资源；未改变则返回 `304 Not Modified` 状态码

### 强缓存：`Expires` & `Cache-Control`

强缓存在 Http 中的实际应用为 `Expires`、`Cache-Control` 两个响应头

#### Http 1.0：`Expires`

- Http 1.0：使用 `Expires` 记录 **资源有效期限**

    服务端返回资源的时候在响应头中加入 `Expires` 请求头，其值为 **GMT 格式的时间**，表示资源的 **有效期限**。浏览器收到有效期限之后，每次再向同样的资源发起请求的时候会先检查上次的有效期限：

    - 在期限内：直接使用缓存资源
    - 资源过期：重新发起请求

    由于 `Expires` 直接记录有效期限的方式存在很大的缺陷：**缓存时间严格依赖于客户端的时间**，也就是服务端传送的是固定时间，假设客户端的时间不与服务端同步或是差异过大，该缓存可能就无效。因此在 Http 1.1 我们就要使用另一个响应头来实现强缓存

#### Http 1.1：`Cache-Control`

- Http 1.1：使用 `Cache-Control` 记录 **缓存模式**

    `Cache-Control` 除了将 `Expires` 的固定时间改成使用缓存时长(`max-age` 记录缓存有效时长，取代固定时间)，还记录了缓存策略(模式)的值

    | Cache-Directive | Description                                                     |
    | --------------- | --------------------------------------------------------------- |
    | public          | 内容都可被任何人缓存                                            |
    | private         | 内容可只可被浏览器缓存(代理服务器不可缓存)                      |
    | no-cache        | 不可立即使用缓存(即 **启用协商缓存**)，需要向服务器进行缓存验证 |
    | no-store        | 所有内容都不被缓存                                              |
    | max-age         | 缓存在 xxx 毫秒后失效，优先级高于 `Last-Modified`               |

    `Cache-Control` 中多个值可以连用，例如设置为：`Cache-Control: public, max-age=0`

    同时，浏览器根据 `Cache-Control` 的设置需要遵循下列的响应原则

    | Cache-Directive       | 新的浏览器窗口   | 原窗口发起请求                  | 刷新     | 路由回退(Back 按钮) |
    | --------------------- | ---------------- | ------------------------------- | -------- | ------------------- |
    | public                | 使用缓存         | 使用缓存                        | 重发请求 | 使用缓存            |
    | private               | 重发请求         | 第一次重发请求<br/>之后使用缓存 | 重发请求 | 使用缓存            |
    | no-cache<br/>no-store | 重发请求         | 重发请求                        | 重发请求 | 重发请求            |
    | max-age               | xxx 秒后重发请求 | xxx 秒后重发请求                | 重发请求 | xxx 秒后重发请求    |

    注意，这边如果使用了 `no-cache` 就会走 **协商缓存** 的路线，所以即便重发请求也不一定会每次都返回目标资源

### 协商缓存：`Last-Modified/If-Modified-Since` & `Etag/If-None-Match`

协商缓存的部分与强缓存的不同，即便浏览器缓存了资源但不会马山使用，而是每次要使用的时候都会 **重新与服务端验证缓存资源的有效性**：

- 资源未改变：则返回 `status 304` 并直接使用缓存
- 资源改变：返回 `status 200` 并返回最新资源

同时也代表协商缓存的 header 是成对的存在：

- 一个为前一次 **响应** 时返回的 **标志**(`Last-Modified`、`Etag`)
- 一个为下一次 **请求** 时加入的标志(`If-Modified-Since`、`If-None-Match`)

下面我们就来看看两对协商缓存的报头是如何交互的

#### Http 1.0：`Last-Modified/If-Modified-Since`

`Last-Modified`、`If-Modified-Since` 两个请求头保存的都是一个 **GMT 格式的时间**：

- `Last-Modified`：代表服务端里资源最后更新的时间
- `If-Modified-Since`：代表客户端上次收到的资源最后更新的时间

知道两个报头的含义，交互方式也就显而易见：

- 第一次请求，服务端返回资源的同时，附带 `Last-Modified` 的报头记录 **资源最后修改时间**
- 第二次请求，客户端将前一次的 `Last-Modified` 放入 `If-Modified-Since` 并发送，服务端检查该请求头
  - 如果对应资源最后修改时间 **相同**：返回 `304 Not Modified`，告知浏览器可直接使用上次缓存资源
  - 如果对应资源最后修改时间 **不同(更晚)**：直接返回最新的资源，并附带新的 `Last-Modified` 时间

#### Http 1.1：`Etag/If-None-Match`

相较于 Http 1.0 的 `Last-Modified/If-Modified-Since`，`Etag/If-None-Match` 选择保存的是一个针对资源生成(特征提取 or 其他手段)的一个 **标识**。其交互顺序与 `Last-Modified/If-Modified-Since` 类似：

- 第一次请求，服务端返回资源，同时为该资源生成一个 **标识(etag)**，并放入 `Etag` 响应头返回
- 第二次请求，客户端将前一次的 `Etag` 放入 `If-None-Match` 并发送，服务端检查该请求头
  - 如果标识  **相同**：返回 `304 Not Modified`，告知浏览器可直接使用上次缓存资源
  - 如果标识  **不同**：直接返回最新的资源，并附带新的 `Etag` 标识

## Http 缓存实验

下面我们创建一个简单的 Express 服务端来测试 Http 的缓存机制，使用的是 Http 1.1 版本的协议

### 项目结构 & 基础代码

首先我们创建一个 Express 项目，项目结构如下

```
http-cache
|- /node-modules
|- /public
  |- index.html
|- /src
  |- app.js
|- package.json
|- yarn.lock
```

核心的文件有两个，一个是 `index.html` 表示客户端，一个是 `app.js` 表示服务端。同时本项目中的客户端以 Express 静态资源的方式访问 

- 客户端：`index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Http Cache</title>
    <style>
      * {
        text-align: center;
      }
    </style>
  </head>
  <body>
    <h1>Http Cache Test Client</h1>
    <div>Here is content</div>
  </body>
</html>
```

- 服务端：`app.js`

```js
const express = require('express')
const app = express()

// basic
app.use(
  express.static('public', {
    lastModified: false,
    etag: false,
  })
)

app.get('/hello', (req, res, next) => {
  res.send('Hello World!')
})

const port = 3000

app.listen(port, () => {
  console.log(`server listen at http://localhost:${port}`)
})
```

由于 Express 默认是启用 `Last-Modified` 和 `Etag` 的，所以我们一开始先都设置为 `false` 关闭其特性。

#### Express Static 缓存配置

`express.static(root, [options])`，除了传入静态资源的根目录 `root` 之外，我们还可以传入配置参数来实现不同的 Http 缓存策略(这边只列出与缓存相关的参数，其他参数详见[参考](#参考)链接)

| options 属性 | 用途                                    | 类型                                                                                 | 默认值 |
| ------------ | --------------------------------------- | ------------------------------------------------------------------------------------ | ------ |
| maxAge       | 设置响应头 `Cache-Control` 的 `max-age` | Number/String(单位：毫秒)                                                            | 0      |
| lastModified | 设置响应头 `Last-Modified`              | Boolean                                                                              | true   |
| etag         | 设置响应头 `Etag`                       | Boolean                                                                              | true   |
| setHeaders   | 设置响应头                              | Function(res, path, stat)<br/>- res 响应对象<br/>- path 资源路径<br/>- stat 资源对象 |        |

### 默认无缓存

第一种情况我们不添加任何缓存策略，则每次请求页面就返回页面

![](https://picures.oss-cn-beijing.aliyuncs.com/img/http_cache_basic.png)

### `Cache-Control` 测试(Http 1.1 强缓存实现)

第二种情况，我们修改服务端，使用 `Cache-Control` 的 `max-age` 属性来实现强缓存策略。修改服务端代码：

```js
// 强缓存：Cache-Control
app.use(
  express.static('public', {
    maxAge: 5000,
    lastModified: false,
    etag: false,
  })
)
```

同时我们在客户端添加一个按钮重新请求 `index.html` 页面

```html
<button onclick="request()">index.html</button>

<script>
  function request() {
    fetch('http://localhost:3000')
      .then((res) => res.text)
      .then((text) => console.log(text))
  }
</script>
```

接下来我们一共向服务端请求三次资源：

- 第一次：输入页面 url 首次加载页面
- 第二次：加载页面后 5s 内再次请求页面(透过点击按钮发起请求)
- 第三次：加载页面后 5s 后再发起一次请求

结果如下；

![](https://picures.oss-cn-beijing.aliyuncs.com/img/http_cache_cache_control1.png)

- 第一次请求：与先前一样就是正常加载页面
- 第二次请求：由于在强缓存生效时限内，所以不会向服务端发起请求，而是直接返回浏览器缓存(`memory cache`/`disk cache`)的资源，如下图所示

![](https://picures.oss-cn-beijing.aliyuncs.com/img/http_cache_cache_control2.png)

- 第三次请求：由于已经过了 `max-age` 的有效期，所以重新向服务端发起请求，并开始新一轮的强缓存计时

### `Etag/If-None-Match` 测试(Http 1.1 协商缓存实现)

第三种情况我们使用 `Etag/If-None-Match` 实现协商缓存，修改服务端代码：

```js
app.use(
  express.static('public', {
    lastModified: true,
    etag: true,
  })
)
```

接下来根据下面步骤进行实验：

- 第一次访问页面
- 刷新页面(第二次访问页面)
- 服务端修改 `index.html` 后再刷新页面(第三次访问页面)

结果如下：

![](https://picures.oss-cn-beijing.aliyuncs.com/img/http_cache_etag1.png)

第一次访问页面：返回页面资源(`status 200`)，并返回第一次的 `Etag`：`W/"1fa-1789863b316"`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/http_cache_etag2.png)

第二次访问页面：由于资源未修改，所以返回 `status 304 Not Modified`，浏览器直接使用缓存页面

![](https://picures.oss-cn-beijing.aliyuncs.com/img/http_cache_etag3.png)

第三次访问页面：由于资源经过修改，所以返回新的页面与新的 `Etag`：`1fb-17898673aa0`

# 结语

不论是在面试的时候或是在真正的项目实践当中，Http 缓存都是非常重要的性能优化基础。透过协议本身直接支持的缓存策略能初步提升页面加载的性能，同时对于不同的静态资源可以设置不同的缓存策略，只需要简单设置一下 Http 响应头就行了，非常便捷且直观，能初步剩下不少的资源传输上的网络开销。
