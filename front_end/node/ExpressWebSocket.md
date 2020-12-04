# Express 实战: 使用 express-ws 实现 WebSocket 协议

@[TOC](文章目录)

<!-- TOC -->

- [Express 实战: 使用 express-ws 实现 WebSocket 协议](#express-实战-使用-express-ws-实现-websocket-协议)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [什么是 WebSocket？](#什么是-websocket)
  - [服务端 WebSocket](#服务端-websocket)
    - [`express`、`express-ws`](#expressexpress-ws)
    - [服务端结构](#服务端结构)
    - [初始化和配置服务端](#初始化和配置服务端)
    - [搭建一般 Express 服务](#搭建一般-express-服务)
    - [添加 WebSocket 服务](#添加-websocket-服务)
  - [客户端 WebSocket](#客户端-websocket)
    - [客户端结构](#客户端结构)
    - [WebSocket API](#websocket-api)
      - [Creation 建立连接](#creation-建立连接)
      - [Properties 重要属性](#properties-重要属性)
      - [Methods 方法](#methods-方法)
      - [Event 事件监听](#event-事件监听)
    - [包装 WebSocket](#包装-websocket)
      - [WebSocketProxy 结构](#websocketproxy-结构)
      - [建立连接：`createSocket`](#建立连接createsocket)
      - [发送消息：`sendMessage`](#发送消息sendmessage)
      - [关闭连接和日志输出：`closeSocket`、`log`](#关闭连接和日志输出closesocketlog)
    - [客户端完整代码](#客户端完整代码)
- [结语](#结语)

<!-- /TOC -->

## 简介

在普通的前后端场景中，HTTP 是我们最常用的通信协议之一，但是 HTTP 总是一问一答，并且总是由客户端向服务端请求。这时候有这么一个业务场景：

后端的某部分数据更新或是接收到某个消息时，才主动更新前端更新。

在这样的场景之下，很明显 HTTP 协议并不能满足我们的需求，这时候就要换新的协议登场啦：WebSocket（标识符 `ws`）

## 参考

<table>
  <tr>
    <td>WebSocket 教程-阮一峰</td>
    <td><a href="http://www.ruanyifeng.com/blog/2017/05/websocket.html">http://www.ruanyifeng.com/blog/2017/05/websocket.html</a></td>
  </tr>
  <tr>
    <td>WebSocket-MDN</td>
    <td><a href="https://developer.mozilla.org/en-US/docs/Web/API/WebSocket">https://developer.mozilla.org/en-US/docs/Web/API/WebSocket</a></td>
  </tr>
  <tr>
    <td>WebSocket协议入门介绍</td>
    <td><a href="https://www.cnblogs.com/nuccch/p/10947256.html">https://www.cnblogs.com/nuccch/p/10947256.html</a></td>
  </tr>
  <tr>
    <td>NodeJs实现WebSocket——express-ws</td>
    <td><a href="https://www.jianshu.com/p/b0700d4162e7">https://www.jianshu.com/p/b0700d4162e7</a></td>
  </tr>
</table>

# 正文

## 什么是 WebSocket？

WebSocket 是一个隶属于应用层的网络通信协议，与 HTTP 同层并且同样是`基于传输层的 TCP 协议`；而与 HTTP 不同的是，WebSocket 提供`双向传输能力`，可以从服务端主动向客户端推送消息(数据)。

这边给出 WS 和 HTTP 的比较图，关于协议的详细解说可以查看参考链接或是其他资料，这边就不做赘述。

![](https://picures.oss-cn-beijing.aliyuncs.com/img/http_websocket_ryf.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/http_websocket_ryf2.png)
（图自参考一-阮一峰老师的教程）

## 服务端 WebSocket

到目前为止我们就只知道 WS 在 C/S 架构下，提供了客户端和服务端双向传输的能力，我们先来建设好服务端部分的 WebSocket 服务。

### `express`、`express-ws`

本篇选用 Express 框架以及框架相关的 `express-ws` 来构建 WebSocket 服务端。

有关构建 Express 应用可以查看我之前写过的 <a href="https://blog.csdn.net/weixin_44691608/article/details/109371958">Express 项目启动</a>

### 服务端结构

```
/websocket-demo-be
|- node_module/
|- src/
    |- index.js
|- package.json
|- yarn.lock
```

### 初始化和配置服务端

本篇选用 yarn 工具来作为服务端的包管理工具

```bash
$ yarn init -y
$ yarn add express express-ws
```

使用 `yarn init` 会自动建立 `package.json`，并在其中添加 script 后续将会用到

并且安装 `express`、`express-ws` 包，也是后续会用到的服务的基础，下面给出 `package.json` 的内容

- `package.json`

```json
{
  "name": "websocket-demo-be",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "scripts": {
    "start": "node src/index.js"
  },
  "dependencies": {
    "express": "^4.17.1",
    "express-ws": "^4.0.0"
  }
}
```

### 搭建一般 Express 服务

首先我们先写出最基本的 Express 服务端，相关步骤和内容说明可以查看前一篇：<a href="https://blog.csdn.net/weixin_44691608/article/details/109371958">Express 项目启动</a>，这边直接给出项目内容

- `index.js`

```js
const express = require('express')

const app = express()

app.get('/', function (req, res, next) {
  res.send('Hello World!')
})

const port = 3000
app.listen(port, () => {console.log(`express server listen at http://localhost:${port}`)})
```

- 输出结果

```
$ yarn start
yarn run v1.22.10
node src/index.js
express server listen at http://localhost:3000
```

使用浏览器访问 `http://localhost:3000` 看到 Hello World! 表示启动成功

### 添加 WebSocket 服务

上面我们搭建出原本一般使用 HTTP 的 express 服务，接下来我们加入 `express-ws` 来启用 WebSocket 服务：

- `index.js`

```js
const express = require('express')
const expressWs = require('express-ws') // 引入 WebSocket 包

const app = express()
expressWs(app) // 将 WebSocket 服务混入 app，相当于为 app 添加 .ws 方法

app.get('/', function (req, res, next) {
  res.send('Hello World!')
})

// 建立 WebSocket 服务
// 
// 第一个参数为服务路径： /basic
// 第二个参数为与前端建立连接时会调用的回调函数
//   ws 相当于建立 WebSocket 的实例
//   req 为建立连接的请求
app.ws('/basic', function (ws, req) {
  console.log('connect success')
  console.log(ws)
  
  // 使用 ws 的 send 方法向连接另一端的客户端发送数据
  ws.send('connect to express server with WebSocket success')

  // 使用 on 方法监听事件
  //   message 事件表示从另一段（服务端）传入的数据
  ws.on('message', function (msg) {
    console.log(`receive message ${msg}`)
    ws.send('default response')
  })

  // 设置定时发送消息
  let timer = setInterval(() => {
    ws.send(`interval message ${new Date()}`)
  }, 2000)

  // close 事件表示客户端断开连接时执行的回调函数
  ws.on('close', function (e) {
    console.log('close connection')
    clearInterval(timer)
    timer = undefined
  })
})

const port = 3000
app.listen(port, () => {console.log(`express server listen at http://localhost:${port}`)})
```

上面的代码完成了下面几件事：

- `app.ws` 用于声明 WebSocket 服务器
  - WebSocket 服务挂载在 `http://localhost:3000/basic` 路径下
  - 每次建立连接时会调用回调函数（第二个参数），并可以拿到 WebSocket 连接的实例(`ws` 对象)
- `ws.send` 方法可以向另一端发送数据
- `ws.on` 可以监听特定事件
  - `message` 事件：接收到消息的时间
  - `close` 事件：连接关闭事件

关于其他 ws 的方法或是 on 能够监听的事件类型可以查阅相关文档，这边只提到几个比较常用的便能够完成一般的业务逻辑

## 客户端 WebSocket

我们已经搭建好一个 WebSocket 的后端服务了，接下来要建立一个前端应用来与后端做对接

### 客户端结构

由于客户端我想尽可能简化，并且 HTML5 原生就提供了 `WebSocket` 类来支持 ws 服务，所以我们之间使用最原本的三个独立文件的项目结构：

```
/websocket-demo-fe
|- index.html
|- index.css
|- index.js
```

- `index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <link rel="stylesheet" href="index.css">
</head>
<body>
    <div id="app">
        <button id="ws-create">Create</button>
        <button id="ws-send">Send</button>
        <button id="ws-close">Close</button>
        <button id="ws-show">Show</button>
    </div>

    <script src="index.js"></script>
</body>
</html>
```

- `index.css`

```css
body {
    margin: 0;
}

#app {
    display: grid;
    place-items: center;
    height: 500px;
}
```

### WebSocket API

我们先来介绍 HTML5 提供的 WebSocket 类，用于建立 WebSocket 连接。并获取连接实例，觉得太冗长可以跳到下面的完成示例再回头来看看 API

#### Creation 建立连接

对应于上面服务端 WebSocket，我们可以使用下面代码来建立连接：

```js
// 先检查当前浏览器是否支持 WebSocket
if (!WebSocket) {
  console.log('Sorry! Your browser doesn\'t support WebSocket')
  return
}
// 透过 url 建立连接并获取连接实例
const socket = new WebSocket('ws://localhost:3000/basic')
```

#### Properties 重要属性

这边获取的 `socket` 便是建立 WebSocket 连接之后的实例，有下面几个重要属性：

| Property       | Description                                                                                                   |
| -------------- | ------------------------------------------------------------------------------------------------------------- |
| binaryType     | 用于传输数据的二进制类型（类似编码啥的，不用动到）                                                            |
| bufferedAmount | 当前传输的缓冲区，可用于检查是否传输完成                                                                      |
| readyState     | 当前实例的状态<br/>0=CONNECTING：连接中<br/>1=OPEN：已建立<br/>2=CLOSING：断开中<br/>3=CLOSED：已断开（关闭） |
| url            | 建立 WebSocket 连接的路径                                                                                     |

#### Methods 方法

我们使用 `new WebSocket(url)` 能够建立 WebSocket 连接并获取实例，实例提供两种方法供我们操作：

- `WebSocket.prototype.send`：发送消息
- `WebSocket.prototype.cloase`：关闭 WebSocket 连接

怎么样没想到吧，就是这么简单hh

#### Event 事件监听

同时与后端相似的是，我们也是透过监听的特定事件来透过 WebSocket 收发消息：

| Event   | Description          |
| ------- | -------------------- |
| open    | 建立连接时触发的事件 |
| close   | 关闭连接时触发的事件 |
| error   | 出现异常时触发的事件 |
| message | 收到消息时触发的事件 |

### 包装 WebSocket

好介绍了这么久的 API，我们赶紧把它拿起来用吧！首先我们先来封装以下 WebSocket 类，每个方法各自调用显得杂乱：

#### WebSocketProxy 结构

我们的目标是将 WebSocket 服务包装成一个代理，拥有如下结构：

```js

function WebSocketProxy (url) {
  this.url = ''            // 服务器路径
  this.socket = undefined  // 连接实例
  this.messages = []       // 收到的消息列表
}

// 用于建立 WebSocket 连接
// 同时注册接收消息事件的处理函数
WebSocketProxy.prototype.create = function createSocket () {}
// 用于向服务端发送消息
WebSocketProxy.prototype.send = function sendMessage (msg) {}
// 用于关闭 WebSocket 连接
WebSocketProxy.prototype.close = function closeSocket () {}
// 用于包装相关输出，也可特化成日志系统
WebSocketProxy.prototype.log = function (msg) {}
```

下面我们一个个介绍方法细节，相关说明可以看代码注释

#### 建立连接：`createSocket`

```js
WebSocketProxy.prototype.create = function createSocket () {
  if (!WebSocket) {
    console.log('Sorry! Your browser doesn\'t support WebSocket')
    return
  }
  // 检查是否已经有示例存在
  if (this.socket) {
    console.log('Connection already exist')
    console.log(this.socket)
    return
  }

  try {
    this.log(`create socket with url: ${this.url}`)
    this.socket = new WebSocket(this.url)

    const self = this
    console.log(this.socket)
    // 连接开启
    this.socket.onopen = function (e) {
      console.log('on open')
    }
    // 连接错误
    this.socket.onerror = function (e) {
      console.log('on error')
      self.close()
    }
    // 消息通知
    this.socket.onmessage = function ({data: msg}) {
      self.messages.push(msg) // 记录消息
      self.log('receive message') // 向后端回复（发送回复消息）
      console.log(msg)
    }

  } catch (err) {
    console.log(err)
    this.close()
  }
}
```

#### 发送消息：`sendMessage`

```js
WebSocketProxy.prototype.send = function sendMessage (msg) {
  if (!this.socket) {
    this.log('socket doesn\'t exist')
    return
  }
  msg = msg || 'default message'
  this.socket.send(msg) // 透过实例发送消息
  this.log('message sent')
}
```

#### 关闭连接和日志输出：`closeSocket`、`log`

```js
WebSocketProxy.prototype.close = function closeSocket () {
  if (!this.socket) {
    this.log('socket doesn\'t exist')
    return
  }
  this.socket.close() // 关闭连接
  this.socket = undefined // 清空当前实例
  this.log('socket close')
}

WebSocketProxy.prototype.log = function (msg) {
  const prefix = '[WebSocketProxy]'
  console.log(`${prefix}${msg}`)
}
```

### 客户端完整代码

最后我们将上面封装好的代码加入到项目里面，完成客户端的最终版本：

- `index.js`

```js
window.onload = function () {
  console.log('window load')
  // 建立代理对象
  const proxy = new WebSocketProxy()

  // 将对象的各个方法绑定到按钮方法，注意方法内部 this 的指向问题
  document.getElementById('ws-create').onclick = () => proxy.create()
  document.getElementById('ws-send').onclick = () => proxy.send()
  document.getElementById('ws-close').onclick = () => proxy.close()
  document.getElementById('ws-show').onclick = () => {
    proxy.log('show messages')
    console.log(proxy.messages)
  }
}

function WebSocketProxy (url) {
  this.url = url || 'ws://localhost:3000/basic'
  this.socket = undefined
  this.messages = []
}

WebSocketProxy.prototype.create = function createSocket () {
  if (!WebSocket) {
    console.log('Sorry! Your browser doesn\'t support WebSocket')
    return
  }
  if (this.socket) {
    console.log('Connection already exist')
    console.log(this.socket)
    return
  }

  try {
    this.log(`create socket with url: ${this.url}`)
    this.socket = new WebSocket(this.url)

    const self = this
    console.log(this.socket)
    // 连接开启
    this.socket.onopen = function (e) {
      console.log('on open')
    }
    // 连接错误
    this.socket.onerror = function (e) {
      console.log('on error')
      self.close()
    }
    // 消息通知
    this.socket.onmessage = function ({data: msg}) {
      self.messages.push(msg)
      self.log('receive message')
      console.log(msg)
    }

  } catch (err) {
    console.log(err)
    this.close()
  }
}

WebSocketProxy.prototype.send = function sendMessage (msg) {
  if (!this.socket) {
    this.log('socket doesn\'t exist')
    return
  }
  msg = msg || 'default message'
  this.socket.send(msg)
  this.log('message sent')
}

WebSocketProxy.prototype.close = function closeSocket () {
  if (!this.socket) {
    this.log('socket doesn\'t exist')
    return
  }
  this.socket.close()
  this.socket = undefined
  this.log('socket close')
}

WebSocketProxy.prototype.log = function (msg) {
  const prefix = '[WebSocketProxy]'
  console.log(`${prefix}${msg}`)
}
```

- 输出结果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/websocket_fe_sample.png)

# 结语

到此我们完成使用 Express 建立的 WebSocket 服务端实例啦，好几次写业务都常常遇到需要从服务端主动发起消息的业务需求，现在才真正的把 WebSocket 捡起来用hh。其实对于其他语言或后端框架如 Springboot、Flask 等，都已经存在一些库提供了对于 WebSocket 的支持，所以读者也不一定要坚持用 express 实现，可以根据不同的业务场景来实现。后续有机会作者也会试着在不同框架和语言下用用看 WebSocket。