# JS 基础: 取消 Ajax 请求(fetch abort)

@[TOC](文章目录)

<!-- TOC -->

- [JS 基础: 取消 Ajax 请求(fetch abort)](#js-基础-取消-ajax-请求fetch-abort)
- [完整代码示例](#完整代码示例)
- [背景](#背景)
- [代码实现](#代码实现)
  - [0. 没有正确 cancel](#0-没有正确-cancel)
  - [1. 添加序列号](#1-添加序列号)
  - [2. fetch 取消请求](#2-fetch-取消请求)
- [更多思考](#更多思考)
  - [RxJS 实现](#rxjs-实现)
- [参考链接](#参考链接)

<!-- /TOC -->

# 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_fetch_abort](https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_fetch_abort)

# 背景

前端同学一定绕不开的一个基础能力便是向后端发起请求，最通用的便是基于 HTTP 协议与后端进行交互。不论是直接使用原生的 XHR API、axios，还是后来的 fetch API 都好。

这些请求都会会返回所谓的 Promise(或是基于 callback)的形式，然而有时候请求与请求间的响应时序是程序员所不能控制的，因此在回调函数、then 的处理上我们应该更加小心，对于同一种请求的重复发送应该进行正确的 cancellation，来保证最终更新到前端的状态永远是最新的数据。

# 代码实现

## 0. 没有正确 cancel

我们先来看看一般场景中常见的写法，没有考虑请求的 cancellation

```ts
useEffect(() => {
  getUserInfoAPI(URL, { body: { id } })
    .then((userInfo) => {
      setUserInfo(userInfo)
    })
}, [id])
```

如上代码，乍看之下没啥问题，但是实际上存在一种隐患是，当 id 在短时间内频繁变换的时候，`getUserInfoAPI` 会被调用多次，这时候多次请求之间的顺序实际上是不一定被保证的(这里的例子比较极端，常见场景之下也容易出现多个地方发起请求并同时掉用相同 API 并且修改相同状态的场景)。

这时候我们对于老旧需要废弃的请求就应该进行所谓的 cancellation，不去响应时序上已经无效的请求数据

## 1. 添加序列号

最直接的做法，那我们就为每一个请求添加序号，保证只更新最新状态咯

```ts
const userInfoAPISeq = useRef(0);
useEffect(() => {
  const seq = userInfoAPISeq.current = userInfoAPISeq.current + 1;
  getUserInfoAPI(URL, { body: { id } })
    .then((userInfo) => {
      if (seq === userInfoAPISeq.current) {
        setUserInfo(userInfo)
      }
    })
}, [id])
```

但是这样看起来实在不是非常优雅

## 2. fetch 取消请求

接下来我们介绍 fetch API 所提供的取消请求的能力，就是利用 `AbortController` 类来实现

```ts
useEffect(() => {
  const controller = new AbortController();
  getUserInfoAPI(URL, {
    body: { id },
    signal: controller.signal,
  })
    .then((userInfo) => {
      if (id === userInfoAPISeq.current) {
        setUserInfo(userInfo)
      }
    })
    .catch((error) => {
      if (
        error.name === 'AbortError' &&
        error instanceof DOMException
      ) {
        // Response of abort
      }
    });

  // at some point
  controller.abort();
}, [id])
```

我们只需要在 fetch API 的 option 里面传入一个 signal，然后后面使用 `controller.abort` 方法就能够取消请求，fetch 方法会自动将 response 置为 error 并且返回指定类型的 Exception 和 name。

# 更多思考

实际上经过测试之后发现，这些类似的 cancel 方法实际上仅仅只是前端层面上的取消，请求只要抵达后端就会进行相应的处理，因此在 cancel 上有几个点可以思考

1. Cancel 只对拿数据的 get 方法有实际的意义，对于 post 请求相关的操作实际上还是会影响后端部分的持久化，这里再做 cancel 就显得没什么意义
2. Abort vs Ignore：这时候到底是调用指定的 cancel 方法(abort 或是 axios 也提供了一个 cancel 的方式)；又或是像第一种方法仅仅通过忽略非最后一次的请求结果即可。两种方法实际上后面看来区别并不大

## RxJS 实现

这时候作者发现如果仅仅只是依赖忽略请求结果也能达到相同效果的话，使用 RxJS 将请求封装成一个 Observable 会看起来更加优雅

```ts
const getUserInfo = (() => {
  subject
    .pipe(
      // debounce(100)
      map(() => from(getUserInfoAPI())),
      switchAll()
    )
    .subscribe((userIfno) => {
      setUserInfo(userInfo)
    });

  return () => {
    subject.next();
  }
})()

getUserInfo();
```

利用 RxJS 的 Observable 模型，将每一个请求作为一个 Observable 来管理，我们就可以很轻易的利用 RxJS 提供的工具链进行 debounce、switchAll 来聚合代码逻辑

# 参考链接

| Title                 | Link                                                                                                                                 |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| Fetch: Abort          | [https://javascript.info/fetch-abort](https://javascript.info/fetch-abort)                                                           |
| AbortController - MDN | [https://developer.mozilla.org/en-US/docs/Web/API/AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) |
| Subject - Rxjs        | [https://rxjs.dev/guide/subject](https://rxjs.dev/guide/subject)                                                                     |
