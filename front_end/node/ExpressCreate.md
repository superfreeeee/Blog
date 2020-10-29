# Express 项目启动

@[TOC](文章目錄)

<!-- TOC -->

- [Express 项目启动](#express-项目启动)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [搭建环境](#搭建环境)
  - [创建服务器](#创建服务器)
- [結語](#結語)

<!-- /TOC -->

## 簡介

前段时间接触到 Node 中台的技术，从技术层面来看就好像用js写后端一样；而从架构的角度来看就是一个后台的缓存和消息转发的服务，透过中台将前段和后端适当隔离，从而减轻后端对请求反馈的实时性要求。从而提高后端的可用性和灵活性，也可以藉由中台区分不同服务的转发。

本篇使用 Express 快速搭建一个极简的服务器，五分钟内搞定。 

## 參考

<table>
  <tr>
    <td>Express 官方</td>
    <td><a href="https://www.expressjs.com.cn/">https://www.expressjs.com.cn/</a></td>
  </tr>
</table>

# 正文

## 搭建环境

首先建立一个空的目录，并用初始化为 npm 项目，然后加入 `express` 依赖

```
$ mkdir express-demo
$ cd express-demo
$ npm init -7
$ npm i -S express
```

- 项目结构

```
/express-demo
    package.json
    package-lock.json
    /node_module
    /src
        app.js
```

并在 `package.json` 里面加入启动用的脚本

```json
{
    // ...
    "scripts": {
      + "start": "node src/app.js",
    },
    // ...
}
```

## 创建服务器

- `app.js`

```js
const express = require('express')

const app = express()
const port = 3000

app.get('/', (req, res, next) => {
  console.log(Object.keys(req))
  console.log(req.headers)
  console.log(req.query)
  console.log(Object.keys(res))
  res.send('Hello World')
})

app.listen(port, () => {
  console.log(`express server start at http://localhost:${port}`)
})
```

这时在命令行输入 `npm start` 启动服务器并在浏览器访问 3000 端口

```bash
$ npm start
> express-demo@1.0.0 start .../express-demo
> node src/app.js

express server start at http://localhost:3000
```

在浏览器看到 `Hello World` 就代表启动成功了

# 結語

就这么简单～使用 express 中台的好处在于，相对于 SpringBoot 等重量级的后端框架（连接数据库需要使用中间件还有一大堆dao层的配置和实体声明），如果仅仅需要简单的数据库访问就可以使用 express 极简的配置，同时也提供可扩展性，较为复杂的逻辑或是对数据的预处理都能透过中台来处理过滤。
