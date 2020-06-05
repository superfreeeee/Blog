# Axios：前端 HTTP 請求

@[TOC](文章目錄)

<!-- TOC -->

- [Axios：前端 HTTP 請求](#axios前端-http-請求)
    - [簡介](#簡介)
    - [參考](#參考)
- [正文](#正文)
    - [Installing 安裝](#installing-安裝)
    - [Axios 實例](#axios-實例)
        - [全局 axios](#全局-axios)
        - [自定義實例](#自定義實例)
        - [默認值(修改配置)](#默認值修改配置)
    - [axios API](#axios-api)
        - [Sample 1: GET 請求](#sample-1-get-請求)
        - [Sample 2: POST 請求](#sample-2-post-請求)
        - [Sample 3: 并發請求](#sample-3-并發請求)
    - [Config 配置選項](#config-配置選項)
        - [Sample](#sample)
    - [Reponse 響應結構](#reponse-響應結構)
    - [Interceptors 攔截器](#interceptors-攔截器)
        - [Request Interceptors 請求攔截器](#request-interceptors-請求攔截器)
        - [Response Interceptors 響應攔截器](#response-interceptors-響應攔截器)
- [結語](#結語)

<!-- /TOC -->

## 簡介

前端作為客戶端常常需要向後端服務器發送請求，不論是同一個網站自己提供的服務器，還是調用其他服務器釋放出來的 API。前端發起 HTTP 請求的方式有很多：

- JQuery ajax：為了使用 ajax 引用整個 JQuery 的成本太高，針對 MVC 模式的設計已與當前 MVVM 框架浪潮不合
- fetch：由於 XHR(XMLHttpRequest)設計上的不清晰，fetch 作為新一代的原生替代方案被提出，但是實現和使用上還是存在許多麻煩

相較上述兩種技術，axios 算是一個相對成熟而穩定且被廣泛使用的前端 HTTP 請求庫，本篇簡單介紹 axios 的基本使用和配置選項，詳細特性可以參照參考鏈結。

## 參考

<table>
    <tr>
        <td>Jquery ajax, Axios, Fetch區別之我見</td>
        <td><a href="https://codertw.com/%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80/147000/">https://codertw.com/%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80/147000/</a></td>
    </tr>
    <tr>
        <td>axios</td>
        <td><a href="http://www.axios-js.com/docs/">http://www.axios-js.com/docs/</a></td>
    </tr>
</table>

# 正文

## Installing 安裝

- npm

```bash
$ npm install axios
```

- bower

```bash
$ bower install axios
```

## Axios 實例

相較於其他博客，我想從 Axios 的實例說起。

axios 提供了一個全局的 `axios`，也可以透過使用 `axios.create()` 創建自定義的實例。我們可以看過當一個前端項目需要向多個不同的服務器發送請求的時候，依據不同的服務器情況創建各自的 axios 實例用於設定不同的默認配置並作封裝，便可將默認配置與使用解耦合。

### 全局 axios

axios 默認提供了一個全局 axios 實例，可直接使用全局對象發起請求：

```js
axios('/user?id=123')
axios({
  method: 'POST',
  url: 'user/123',
  data: {
    id: 0,
    name: 'John'
  }
})
```

### 自定義實例

我們也可以透過 `axios.create()` 來創建自己的 axios 實例，從而將自定義配置選項與全局對象區隔開來：

```js
const instance = axios.create({
  baseURL: 'http://localhost:8000'
})
instance.defaults.method = 'POST'
```

### 默認值(修改配置)

不論是全局 axios 對象，或是自定義 axios 對象都可以在創建之後透過 `defaults` 重新修改配置選項：

- 全局 axios

```js
axios.defaults.baseURL = 'http://localhost:8999'
axios.defaults.headers.common['Authorization'] = AUTH_TOKEN
```

- 自定義 axios 實例

```js
// 創建實例
const instance = axios.create({
    baseURL = 'http://localhost:8080'
})
// 修改配置選項
instance.defaults.headers.common['Authorization'] = AUTH_TOKEN
```

## axios API

除了 axios 對象本身就能夠作為方法調用之外，axios 還為幾乎所有 HTTP 方法提供了別名

| Function                              |
| ------------------------------------- |
| axios(config)                         |
| axios.request(config)                 |
| axios.get(url\[, config\])            |
| axios.delete(url\[, config\])         |
| axios.head(url\[, config\])           |
| axios.post(url\[\, data[, config\]\]) |
| axios.put(url\[, config\])            |
| axios.patch(url\[, config\])          |
| axios.all(iterable)                   |
| axios.spread(iterable)                |

### Sample 1: GET 請求

```js
// 直接使用對象
axios({
  url: '/user',
  method: 'GET',
  params: {
    id: 123
  }
}).then((res) => {
  console.log(res)
})

// 使用 get 別名
axios.get('/user?id=123').then((res) => {
  console.log(res)
})

// url 和參數分離
axios
  .get('/user', {
    params: {
      id: 123
    }
  })
  .then((res) => {
    console.log(res)
  })
```

- 說明 1：第一個示例使用 axios 本身，傳入整個 config 對象
- 說明 2：第二個示例使用 get 別名，第一個參數為 url，第二個為可選的 config

### Sample 2: POST 請求

```js
// post 別名
axios
  .post('/user', {
    id: 0,
    name: 'John'
  })
  .then((res) => {
    console.log(res)
  })
```

- 說明：post 別名第一個也是 url，而第二個為 data 對象，將被序列化後作為 http 報文的 request body 傳輸

### Sample 3: 并發請求

與 `Promise.all` 相似，`axios.all` 同時發送多個請求，並等待到所有請求都回覆後，再透過 `axios.spread` 將多個請求的 Response 分離

```js
function getUser() {
  return axios.get('/user/123')
}

function getToken() {
  return axios.get('/user/123/token')
}

axios.all([getUser(), getToken()]).then(
  axios.spread(function (userRes, tokenRes) {
    console.log('deal after both response was got')
  })
)
```

## Config 配置選項

不管是創建 axios 實例或是發送請求時候的臨時配置，都提到了配置選項 config 對象：

| Option             | Description                                                                                 | values(type)                                                                |
| ------------------ | ------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| url                | 發送請求 url                                                                                | （必填）string                                                              |
| method             | 創建請求使用的方法                                                                          | GET \| POST \| DELETE \| HEAD \| PUT \| PATCH                               |
| baseURL            | 基地址，通常保存服務器和端口，將被添加在所有請求的 url 前面                                 | string                                                                      |
| transformRequest   | 用於 POST \| PUT \| PATCH，在發送請求前處理 data 數據                                       | Function[]                                                                  |
| transformResponse  | 在傳遞給 then, catch 之前對 response 數據處理                                               | Function[]                                                                  |
| headers            | 自定義請求頭(Request headers)                                                               | {\<header-key\>: \<header-value\>}                                          |
| params             | 添加到 url 裡的參數，與 data 分離                                                           | Object                                                                      | URLSearchParams |
| paramsSerializer   | 參數的序列化函數                                                                            | Function                                                                    |
| data               | 用於 POST \| PUT \| PATCH，作為 Request body 發送                                           | string \| plain object \| ArrayBuffer \| ArrayBufferView \| URLSearchParams |
| timeout            | 請求超時的毫秒數(ms)，0 表示不限制超時時間，超時就會中斷請求                                | number                                                                      |
| withCredentials    | 跨域請求時是否使用憑證                                                                      | boolean                                                                     |
| adapter            | 自定義處理請求方法，可用於測試                                                              | Function                                                                    |
| auth               | 使用 HTTP Basic 驗證，自動生成 `Authorization` 請求頭，並覆蓋自定義的 `Authorization`       | Object                                                                      |
| responseType       | 指定服務器響應的數據類型                                                                    | arraybuffer \| blob \| document \| json \| text \| stream                   |
| xsrfCookieName     | 用作 xsrf token 的值的 cookie 名稱                                                          | string                                                                      |
| xsrfHeaderName     | 乘載 xsrf token 的值的 HTTP 請求頭名稱                                                      | string                                                                      |
| onUploadProgress   | 上傳處理進度的事件的響應函數                                                                | Function(uploadEvent)                                                       |
| onDownloadProgress | 下載處理進度的事件的響應函數                                                                | Function(downloadEvent)                                                     |
| maxContentLength   | 響應內容的最大容量                                                                          | number                                                                      |
| validateStatus     | 定義針對 Response Status 決定 Promise 的狀態(返回 true 為 resolved，返回 false 為 rejected) | (status)=>boolean                                                           |
| maxRedirects       | 最大重定向次數                                                                              | number                                                                      |
| httpAgent          | 在 node.js 環境中自定義 http 服務器代理                                                     | http.Agent                                                                  |
| httpsAgent         | 在 node.js 環境中自定義 https 服務器代理                                                    | http.Agent                                                                  |
| proxy              | 指定代理服務器信息，將會自動生成 `Proxy-Authorization` 請求頭                               | { host, port, auth }                                                        |
| cancelToken        | 請求取消                                                                                    | CancelToken                                                                 |

### Sample

（代碼轉自參考二鏈接）

```js
const config = {
  // `url` 是用于请求的服务器 URL
  url: '/user',

  // `method` 是创建请求时使用的方法
  method: 'get', // 默认是 get

  // `baseURL` 将自动加在 `url` 前面，除非 `url` 是一个绝对 URL。
  // 它可以通过设置一个 `baseURL` 便于为 axios 实例的方法传递相对 URL
  baseURL: 'https://some-domain.com/api/',

  // `transformRequest` 允许在向服务器发送前，修改请求数据
  // 只能用在 'PUT', 'POST' 和 'PATCH' 这几个请求方法
  // 后面数组中的函数必须返回一个字符串，或 ArrayBuffer，或 Stream
  transformRequest: [function (data) {
    // 对 data 进行任意转换处理

    return data;
  }],

  // `transformResponse` 在传递给 then/catch 前，允许修改响应数据
  transformResponse: [function (data) {
    // 对 data 进行任意转换处理

    return data;
  }],

  // `headers` 是即将被发送的自定义请求头
  headers: {'X-Requested-With': 'XMLHttpRequest'},

  // `params` 是即将与请求一起发送的 URL 参数
  // 必须是一个无格式对象(plain object)或 URLSearchParams 对象
  params: {
    ID: 12345
  },

  // `paramsSerializer` 是一个负责 `params` 序列化的函数
  // (e.g. https://www.npmjs.com/package/qs, http://api.jquery.com/jquery.param/)
  paramsSerializer: function(params) {
    return Qs.stringify(params, {arrayFormat: 'brackets'})
  },

  // `data` 是作为请求主体被发送的数据
  // 只适用于这些请求方法 'PUT', 'POST', 和 'PATCH'
  // 在没有设置 `transformRequest` 时，必须是以下类型之一：
  // - string, plain object, ArrayBuffer, ArrayBufferView, URLSearchParams
  // - 浏览器专属：FormData, File, Blob
  // - Node 专属： Stream
  data: {
    firstName: 'Fred'
  },

  // `timeout` 指定请求超时的毫秒数(0 表示无超时时间)
  // 如果请求话费了超过 `timeout` 的时间，请求将被中断
  timeout: 1000,

  // `withCredentials` 表示跨域请求时是否需要使用凭证
  withCredentials: false, // 默认的

  // `adapter` 允许自定义处理请求，以使测试更轻松
  // 返回一个 promise 并应用一个有效的响应 (查阅 [response docs](#response-api)).
  adapter: function (config) {
    /* ... */
  },

  // `auth` 表示应该使用 HTTP 基础验证，并提供凭据
  // 这将设置一个 `Authorization` 头，覆写掉现有的任意使用 `headers` 设置的自定义 `Authorization`头
  auth: {
    username: 'janedoe',
    password: 's00pers3cret'
  },

  // `responseType` 表示服务器响应的数据类型，可以是 'arraybuffer', 'blob', 'document', 'json', 'text', 'stream'
  responseType: 'json', // 默认的

  // `xsrfCookieName` 是用作 xsrf token 的值的cookie的名称
  xsrfCookieName: 'XSRF-TOKEN', // default

  // `xsrfHeaderName` 是承载 xsrf token 的值的 HTTP 头的名称
  xsrfHeaderName: 'X-XSRF-TOKEN', // 默认的

  // `onUploadProgress` 允许为上传处理进度事件
  onUploadProgress: function (progressEvent) {
    // 对原生进度事件的处理
  },

  // `onDownloadProgress` 允许为下载处理进度事件
  onDownloadProgress: function (progressEvent) {
    // 对原生进度事件的处理
  },

  // `maxContentLength` 定义允许的响应内容的最大尺寸
  maxContentLength: 2000,

  // `validateStatus` 定义对于给定的HTTP 响应状态码是 resolve 或 reject  promise 。如果 `validateStatus` 返回 `true` (或者设置为 `null` 或 `undefined`)，promise 将被 resolve; 否则，promise 将被 rejecte
  validateStatus: function (status) {
    return status >= 200 && status < 300; // 默认的
  },

  // `maxRedirects` 定义在 node.js 中 follow 的最大重定向数目
  // 如果设置为0，将不会 follow 任何重定向
  maxRedirects: 5, // 默认的

  // `httpAgent` 和 `httpsAgent` 分别在 node.js 中用于定义在执行 http 和 https 时使用的自定义代理。允许像这样配置选项：
  // `keepAlive` 默认没有启用
  httpAgent: new http.Agent({ keepAlive: true }),
  httpsAgent: new https.Agent({ keepAlive: true }),

  // 'proxy' 定义代理服务器的主机名称和端口
  // `auth` 表示 HTTP 基础验证应当用于连接代理，并提供凭据
  // 这将会设置一个 `Proxy-Authorization` 头，覆写掉已有的通过使用 `header` 设置的自定义 `Proxy-Authorization` 头。
  proxy: {
    host: '127.0.0.1',
    port: 9000,
    auth: : {
      username: 'mikeymike',
      password: 'rapunz3l'
    }
  },

  // `cancelToken` 指定用于取消请求的 cancel token
  // （查看后面的 Cancellation 这节了解更多）
  cancelToken: new CancelToken(function (cancel) {
  })
}
```

## Reponse 響應結構

```js
{
  // Response 的狀態碼
  status: 200,

  // 服務器響應的狀態信息
  statusText: 'OK',

  // 服務器的響應頭(headers)
  headers: {},

  // 請求配置信息
  config: {}

    // 真正由服務器返回的響應信息
  data: {},
}
```

## Interceptors 攔截器

除了默認配置之外，axios 還提供了一個接口為攔截器，可以在 request、response 之前處理 config 和 response 選項

### Request Interceptors 請求攔截器

```js
axios.interceptors.request.use(
  function (config) {
    // 在發送請求前對 config 做處理
    return config
  },
  function (err) {
    // 在請求錯誤時做處理
    return Promise.reject(err)
  }
)
```

### Response Interceptors 響應攔截器

```js
axios.interceptors.response.use(
  function (response) {
    // 在響應回調前對 response 做處理
    return response
  },
  function (err) {
    // 在響應錯誤時做處理
    return Promise.reject(err)
  }
)
```

# 結語

axios 還有其他提供的能力如 cancelToken, typescript 支持等特性，可以到 axios 的 github 首頁上查詢細節。 通常 config 內部的很多選項其實並不需要特別配置。不過 axios 透過對 XHR 的封裝非常好，並提供許多較為細節的配置供不同層級的程序員選擇性的為 axios 實例配置需要的屬性。在前後端分離浪潮之下，前後端的交互更是一個 web 系統中非常重要的一環，熟悉使用和封裝 axios 請求能使前端開發更為便捷且能保障一定的安全性和可靠性。
