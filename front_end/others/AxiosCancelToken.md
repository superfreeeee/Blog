# Axios 应用: 实现后端接口封装 & 重复请求回避(撤销请求)

@[TOC](文章目录)

<!-- TOC -->

- [Axios 应用: 实现后端接口封装 & 重复请求回避(撤销请求)](#axios-应用-实现后端接口封装--重复请求回避撤销请求)
  - [简介](#简介)
  - [参考](#参考)
  - [完整示例代码](#完整示例代码)
- [正文](#正文)
  - [目标](#目标)
  - [axios 中的请求撤销](#axios-中的请求撤销)
    - [1. CacelToken.source 工厂方法](#1-caceltokensource-工厂方法)
    - [2. CancelToken 构造函数(参数为接受一个 cancel 方法的回调)](#2-canceltoken-构造函数参数为接受一个-cancel-方法的回调)
  - [代码示例](#代码示例)
    - [axios 对象封装](#axios-对象封装)
    - [后端 API 封装](#后端-api-封装)
    - [测试](#测试)
- [结语](#结语)

<!-- /TOC -->

## 简介

之前的<a href="https://blog.csdn.net/weixin_44691608/article/details/106077323">Axios：前端 HTTP 請求</a>介绍了 js 的第三库 Axios，封装前端 AJAX 请求的 API 和详细的使用参数。但是相信大部分的人，包括正在做项目而需要直接上手的开发者，肯定没有耐心也没有时间细细的品味哪些参数能用在哪里。

因此本篇将要从封装后端 API 接口以及实现避免多次点击的重复请求问题的角度进行整理，为大家提供一种 Axios 的使用模型，相当于是对 Axios 进行再封装，马上来看看。

## 参考

<table>
  <tr>
    <td>axios中的取消请求</td>
    <td><a href="https://www.cnblogs.com/yancyzheng/p/12576329.html">https://www.cnblogs.com/yancyzheng/p/12576329.html</a></td>
  </tr>
</table>

## 完整示例代码

<a href="https://github.com/superfreeeee/Blog-code/tree/main/front_end/others/axios-encapsulation">https://github.com/superfreeeee/Blog-code/tree/main/front_end/others/axios-encapsulation</a>

# 正文

## 目标

首先我们先明确接下来要完成的模型目标：

1. 封装 axios 和后端的接口，使用者不需要自行管理 axios 相关配置，能专注在业务数据上
2. 限制重复的 http 请求，短时间内发起相同请求则撤销前一次请求(重复触发的逻辑可根据业务需要进行更改)

## axios 中的请求撤销

在开始实现 axios 的再封装之前，我们先来搞清除 axios 提供的撤销请求的接口有哪些

### 1. CacelToken.source 工厂方法

第一种是使用 `axios.CancelToken` 提供的工厂方法 `source` 生成一个 cancelToken，并可透过 `cancel` 方法来撤销请求

```js
import axios from 'axios'
const CancelToken = axios.CancelToken

// 生成 cancelToekn 实例
const source = CancelToken.source()

axios.get('/xxx/yyy', {
    cancelToken: source.token
}).catch((err) => {
    if (axios.isCancel(err)) {
        console.log('Request cancel')
    } else {
        // 其他错误处理
        // 例：return Promise.reject(err)
    }
})
```

### 2. CancelToken 构造函数(参数为接受一个 cancel 方法的回调)

第二种则是直接使用 CancelToken 的构造函数，参数为一个回调函数，回调函数接受一个 cancel 方法作参数

```js
import axios from 'axios'
const CancelToken = axios.CancelToken

let cancel

axios.get('/xxx/yyy', {
    cancelToken: new CancelToken((c) => {
        // c 为该请求对应的撤销函数
        cancel = c
    })
})

// 撤销请求
cancel()
```

## 代码示例

好了了解了 axios 的撤销请求方法之后，就能够开始我们的项目用 axios 再封装了。完整的示例项目实现请到[完整示例代码](#完整示例代码)，这边只列出关键的核心代码。

### axios 对象封装

首先第一步我们要创建自己的 axios 对象，取代全局 axios 对象作为我们的请求主体

```js
/* axios.js */
import axios from 'axios'

const CancelToken = axios.CancelToken

const instance = axios.create({
  baseURL: 'http://localhost:3000',
  timeout: 10000,
})

// 请求中队列
const requests = []

// 撤销请求
const cancelRequest = (config) => {
  const target = `${config.url}&${config.method}`
  for (let idx in requests) {
    if (requests[idx].umet === target) {
      requests[idx].cancel(`撤销请求: ${target}`)
      requests.splice(idx, 1)
    }
  }
}

// 请求拦截器
instance.interceptors.request.use(
  (config) => {
    config.headers = {
      'content-type': 'application/json; charset=utf-8',
    }
    cancelRequest(config)
    config.cancelToken = new CancelToken((c) => {
      requests.push({ umet: `${config.url}&${config.method}`, cancel: c })
    })
    return config
  },
  (error) => {
    return Promise.reject(error)
  }
)

// 响应拦截器
instance.interceptors.response.use(
  (response) => {
    cancelRequest(response.config)
    return response.data
  },
  (error) => {
    return Promise.reject(error)
  }
)

export default instance
```

1. 首先第一步我们使用 `axios.create` 创建新的 axios 实例(通常如果整个前端需要请求多个后端服务，可以考虑不设置 baseUrl；或是选择为每个服务端创建一个专用的 axios 实例)
2. 设置**请求中队列(`requests`)**和**撤销请求的函数(`cancelRequest`)**
3. 设置请求/响应的拦截器：请求时先撤销前一次相同的请求，并设置新的 `cancelToekn`；响应时删除对应的撤销函数
4. 最后返回我们自己创建的 axios 实例 `instance`，不使用全局的 axios 实例

### 后端 API 封装

在封装后端 API 的时候通常我们可以将不同业务(好的命名风格应该按前缀划分)分装在不同的文件之下，并对每个后端方法进行对应的封装

- 后端接口

```js
app.get('/test/1', (req, res, next) => {
  setTimeout(() => {
    res.send(`response1: ${new Date()}`)
  }, 3000)
})

app.get('/test/2', (req, res, next) => {
  setTimeout(() => {
    res.send(`response2: ${new Date()}`)
  }, 3000)
})
```

后端接口的实现我们使用 express 简单实现，并设定适当的延迟方便测试的时候进行重复请求。下面我们就可以针对后端的接口进行对应的封装

```js
/* testAPI.js */
import axios from './axios.js'

const prefix = '/test'

function queryParamToStr(obj = {}) {
  const params = []
  for (let prop in obj) {
    params.push(`${prop}=${obj[prop]}`)
  }
  return params.join('&')
}

export function test1API(queryParam) {
  const queryStr = queryParamToStr(queryParam)
  return axios({
    url: `${prefix}/1?${queryStr}`,
    method: 'GET',
  })
}

export function test2API(queryParam) {
  const queryStr = queryParamToStr(queryParam)
  return axios({
    url: `${prefix}/2?${queryStr}`,
    method: 'GET',
  })
}
```

除了对后端接口进行封装之外，还另外设置了一个用于处理查询参数的方法，使得调用接口的时候能传入简单对象而不需要自己组装成 query 参数

### 测试

最后给出调用代码和测试结果输出

```js
/* index.js */
import { test1API, test2API } from './api/test.js'

document.getElementById('btn-test1').addEventListener('click', (e) => {
  console.log('click test1')
  test1API().then(res => {
    console.log(res)
  })
})
document.getElementById('btn-test2').addEventListener('click', (e) => {
  console.log('click test2')
  test2API().then(res => {
    console.log(res)
  })
})
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/axios_encapsulation_test.png)

首先我们先分别执行一次两个请求的正常调用，之后再连续请求两次第一个接口，我们可以看到第二次重复调用第一个接口之后撤销了一次，并且最后只返回一次结果，大功告成！

# 结语

本篇实现了对 axios 进一步的封装，以及重复请求的避免。然而在实际业务场景之下我们还可以透过使用其他手段来避免重复请求，例如：点击按钮后取消事件监听、请求状态跟踪等，需要根据具体的业务目标和需要作调整，这边提供的是一种根据 axios 提供的接口(`CancelToekn` 对象)和拦截器(`interceptors`)来实现的，供大家参考。
