# JS 缓存: Service Worker 实现离线应用

@[TOC](文章目录)

<!-- TOC -->

- [JS 缓存: Service Worker 实现离线应用](#js-缓存-service-worker-实现离线应用)
- [Service Worker 概述](#service-worker-概述)
- [1. 加载 Service Worker](#1-加载-service-worker)
- [2. Service Worker 编程](#2-service-worker-编程)
  - [2.0 Service Worker 生命周期](#20-service-worker-生命周期)
  - [2.1 install 启用缓存](#21-install-启用缓存)
  - [2.2 fetch 缓存代理](#22-fetch-缓存代理)
  - [2.3 activate 清理资源](#23-activate-清理资源)
- [小结](#小结)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# Service Worker 概述

Service Worker 属于一种特殊的 Web Worker 之一，有兴趣可以参考[前篇：HTML5 新特性: Web Worker 的创建与使用(webpack + TS 环境)](https://blog.csdn.net/weixin_44691608/article/details/119839000)

今天要介绍的 Service Worker 的一个主要功能就是在于缓存整个应用，使我们能够在离线状态下也能使用已经访问过的应用

# 1. 加载 Service Worker

在开始具体的进行 Service Worker 编程之前，我们先来看看如何加载一个 Service Worker

- `index.js`

```js
const INDEX_PREFIX = `[index.js]`;

console.log(`${INDEX_PREFIX} load script sucess`);

// load service worker
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    const serviceWorkerUrl = '/sw.js';
    console.log(
      `${INDEX_PREFIX} try register serviceWorker: ${serviceWorkerUrl}`
    );

    navigator.serviceWorker
      .register(serviceWorkerUrl)
      .then((registration) => {
        console.log(`${INDEX_PREFIX} register success:`, registration);
      })
      .catch((reason) => {
        console.log(`${INDEX_PREFIX} register fail:`, reason);
      });
  });
}
```

加载一个 Service Worker 的关键点在 `navigator.serviceWorker` 指向的 `ServiceWorkerContainer` 对象上，然后我们使用 `ServiceWorkerContainer.prototype.register(path)` 进行注册。

注意这里传入的 path 是 Service Worker 的脚本路径，同时必须是同源的（关于 scope 这里就不扯了）

接下来则是写一个 Service Worker 用的脚本就行了

- `sw.js`

```js
const CURRENT_VERSION = 'v1';
const LIFECYCLE_PREFIX = `[serviceWorker: sw.js(${CURRENT_VERSION})]`;

console.log(`${LIFECYCLE_PREFIX} load serviceWorker`, this);
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_service_worker_1_load.png)

从上面我们就可以看到一个 Service Worker 被加载进来了

同时我们也可以从下面两种方式查看当前 Service Worker 的运行情况

- 从控制台 F12 `Application > Application > Service Workers` 下查看当前页面的 Service Worker
  - Service Worker 的注册是与域名相关的，因此我们在这里通常是查看当前域名下的所有 Service Worker

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_service_worker_2_app.png)

- 从 Chrome 浏览器下的 inspect 页面查看所有 Service Worker
  - [chrome://inspect/#service-workers](chrome://inspect/#service-workers)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_service_worker_3_inspect.png)

# 2. Service Worker 编程

接下来我们进入主要的 Service Worker 编程

## 2.0 Service Worker 生命周期

首先我们先来看 Service Worker 的生命周期

![](https://developers.google.com/web/fundamentals/primers/service-workers/images/sw-lifecycle.png?hl=zh-cn)

- 参考 Google 提供的 Service Worker 简介

实际上 Service Worker 属于一段运行在一个独立进程的上下文环境当中，因此即使我们关闭应用页面还是会独立存在。这时候如果我们重新打开页面，Service Worker 就能够监听到一些特殊的事件（如 `install、fetch` 等），然后进而对部分页面资源进行代理、缓存的作用。

不仅仅是在离线状态下能提供一定程度的页面保留，在正常连线的情况也能提供部分的页面缓存加速功能

## 2.1 install 启用缓存

我们要介绍的第一个事件是 `install` 事件，也就是 Service Worker 第一次启用的时候会调用的方法

- `sw.js`

```js
self.addEventListener('install', (event) => {
  console.log(`${LIFECYCLE_PREFIX} onInstall`, event);

  event.waitUntil(
    caches.open(CURRENT_VERSION).then((cache) => {
      return cache.addAll(['/', '/index.html', '/index.css', '/index.js']);
    })
  );
});
```

在 install 钩子下，我们使用 `caches.open + cache.addAll` 来声明我们需要缓存的资源路径，结果如下图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_service_worker_4_install.png)

## 2.2 fetch 缓存代理

在 install 钩子声明好 cache 的缓存资源之后，我们可以透过监听 fetch 事件来拦截页面的资源请求，然后透过返回缓存结果来进行加速/置换

- `sw.js`

```js
self.addEventListener('fetch', (event) => {
  console.log(`${LIFECYCLE_PREFIX} onFetch, url=${event.request.url}`, event);

  event.respondWith(
    caches.match(event.request).then((response) => {
      if (response) {
        console.log(`${LIFECYCLE_PREFIX} onFetch: pre cache`, response);
      }
      return (
        response ||
        fetch(event.request)
          .then(function (response) {
            let responseClone = response.clone();

            console.log(`${LIFECYCLE_PREFIX} onFetch: fetch and cache`, response);
            caches.open(CURRENT_VERSION).then(function (cache) {
              cache.put(event.request, responseClone);
            });
            return response;
          })
          .catch(function () {
            // return caches.match('/sw-test/gallery/myLittleVader.jpg');
            return new Response('fetch fail');
          })
      );
    })
  );
});
```

我们监听到 fetch 事件之后，判断是否经过缓存(`caches.match`)，决定是否使用 `fetch(event.request)` 方法重新发起请求，并且使用 `cache.push` 写入缓存当中，最后返回请求

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_service_worker_5_fetch.png)

## 2.3 activate 清理资源

到此为止整个 Service Worker 应该都能够正常运行了，最后我们考虑一个场景：当我们的应用进行更新之后，页面的缓存应该需要适当地刷新（也就是 `CURRENT_VERSION` 对应的版本号应该进行变化，也就是同时修改作为缓存 key 的吧版本号，因此我们需要透过监听 `activate` 事件来进行缓存资源的清理）

- `sw.js`

```js
self.addEventListener('activate', (event) => {
  console.log(`${LIFECYCLE_PREFIX} onActivate`, event);

  // clean none current version cache
  event.waitUntil(
    caches.keys().then((keys) => {
      console.log(`${LIFECYCLE_PREFIX} onActivate: caches keys`, keys);
      return Promise.all(
        keys.map((key) => {
          if (key !== CURRENT_VERSION) {
            return caches.delete(key);
          }
        })
      );
    })
  );
});
```

每次 Service Worker 重新启动的时候，我们去检查是否存在其他版本的缓存记录，并进行资源清理的工作

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_service_worker_6_cleanup.png)

这里关于为什么是在 `activate` 事件内进行清除，这里我理解的是因为每次加载新版本的 Service Worker 会进行一次新的 Install 事件，然而过程对于 `fetch` 资源的管理不一定成功，也就是说可以在此阶段进行版本回退/兼容等操作。

因此我们必须等到下一次 Service Worker 再加载，同时没有更新（现有版本 Service Worker 重新启动）的状态下，可以看做当前 Service Worker 的版本已经趋于稳定，才对先前的缓存资源进行清理

# 小结

Service Worker 不同于通用的 Web Worker，主要关注点在于页面资源的管理，当然或许还有其他的应用方式，欢迎读者与作者分享。

# 其他资源

## 参考连接

| Title                                   | Link                                                                                                                                                                                   |
| --------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Service Worker API - MDN                | [https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API](https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API)                                             |
| 使用 Service Workers - MDN              | [https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API/Using_Service_Workers](https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API/Using_Service_Workers) |
| Service Worker: 简介 - Web Fundamentals | [https://developers.google.com/web/fundamentals/primers/service-workers?hl=zh-cn](https://developers.google.com/web/fundamentals/primers/service-workers?hl=zh-cn)                     |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_service_worker](https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_service_worker)
