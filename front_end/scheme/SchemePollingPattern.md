# 技术方案实践: 前端轮询方案实现 & 思考

@[TOC](文章目录)

<!-- TOC -->

- [技术方案实践: 前端轮询方案实现 & 思考](#技术方案实践-前端轮询方案实现--思考)
- [前言](#前言)
- [正文](#正文)
  - [0. 什么叫轮询？](#0-什么叫轮询)
  - [1. 轮询接口定义 & 数据结构](#1-轮询接口定义--数据结构)
  - [2. 轮询方案 1: 使用定时器](#2-轮询方案-1-使用定时器)
  - [3. 轮询方案 2: 使用尾递归](#3-轮询方案-2-使用尾递归)
    - [3.1 增加取消机制](#31-增加取消机制)
    - [3.2 增加组件卸载时终止轮询机制](#32-增加组件卸载时终止轮询机制)
    - [3.3 防止并发场景](#33-防止并发场景)
  - [4. 轮询方案 3: 定制轮询引擎](#4-轮询方案-3-定制轮询引擎)
    - [4.1 核心轮询方法](#41-核心轮询方法)
    - [4.2 constructor 构造函数 & start、pause、continue 操作](#42-constructor-构造函数--startpausecontinue-操作)
    - [4.3 回调注册 onDataRecieve & 状态通知 notify](#43-回调注册-ondatarecieve--状态通知-notify)
    - [4.4 自定义钩子](#44-自定义钩子)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

在普通业务场景中，前端查询后端状态的场景并不少见，然后并不是所有场景都适合建立长链接来进行服务端推送，因此我们需要结合实际业务场景来决定适合的技术方案。从低版本或是较早期的方案实现，我们发现前端主动轮询后端状态是非常常见而有时甚至是必要的手段。

本篇将带着读者一起思考前端轮询到底怎么写比较好，不过记住：**没有银弹**，只有更适合的解决方案，接下来提到的几个方案不能说哪一个一定是错的，这就需要开发人员自己根据实际的应用场景来判断了。

# 正文

## 0. 什么叫轮询？

所谓的轮询就是，由后端维护某个 **状态**，或是一种连续多篇的数据（如分页、分段），由前端决定按序访问的方式将所有片段依次查询，直到后端给出终止状态的响应（结束状态、分页的最后一页等）

## 1. 轮询接口定义 & 数据结构

本篇以分页为例，由前端依次轮询后端的分页数据知道最后一页停止

请求参数如下

- `/fe/src/api/interface.ts`

```ts
export interface IPollingParams {
  pageNo: number; // 页码
  pageSize: number; // 页大小
}
```

而后端给出的响应数据包括一个数据项（data）和一个分页描述对象（pagination）

```ts
export interface IPollingResponse {
  data: number[];
  pagination: IPollingPagination;
}

// 页描述对象
export interface IPollingPagination {
  pageNo: number;
  pageSize: number;
  total: number;
  hasFinish: boolean;
}
```

也就是说后端给出的接口定义如下

- `/fe/src/api/index.ts`

```idl
path:     /polling
method:   POST
body:     { pageNo, pageSize }
response: { data: [], pagination: { pageNo, pageSize, total, hasFinish } }
```

下面我们就可以开始尝试找出前端轮询方案的通用模式

## 2. 轮询方案 1: 使用定时器

首先第一个直觉一定是想到使用 `setInterval` 这个 api 进行定时检查数据（本篇写在了一个 React hook 当中，不过这块的写法可以忽略，主要核心在于轮询方法 doPolling 上）

- `/fe/src/components/Polling/hooks/usePollingType2.ts`

```ts
import { useCallback, useRef } from 'react';

import { polling } from '@api';

const usePollingType2 = () => {
  const doPollingType2 = useCallback(async () => {
    let currentPageNo = 1;
    const pageSize = 10;

    const timer = setInterval(() => {
      console.log(`[doPollingType1] ${currentPageNo} ${pageSize}`);
      polling(currentPageNo, pageSize).then(({ data, pagination }) => {
        console.log(`[doPollingType2] recieve data`, data);

        const { pageNo, pageSize, hasFinish } = pagination;
        if (!hasFinish) {
          console.log(`[doPollingType2] ready to do next ${pageNo + 1} ${pageSize}`);
          currentPageNo++;
        }
      });
    }, 1000);
  }, []);

  return doPollingType2;
};

export default usePollingType2;
```

实现上确实看起来简洁有力，但是这其实存在很大的问题：实际上所谓的“轮询”并不像直观上就是一个单纯的定时方法，相反的每一次的轮询都依赖前一次的结果来决定，也就是说更偏向于一种连续的链式调用方法。第二种方法我们就尝试使用尾递归的方式来实现所谓的轮询。

## 3. 轮询方案 2: 使用尾递归

第二种轮询方案我们改成使用 `setTimeout` 来进行延时的尾递归调用

- `/fe/src/components/Polling/hooks/usePollingType1.ts`

```ts
import { useRef, useCallback, useEffect } from 'react';

import { polling } from '@api';
import { wait } from '@utils';
import useUnMount from '@hooks/useUnMount';

const usePollingType1 = () => {
  const doPollingType1 = useCallback(async () => {
    const startPageNo = 1;
    const pageSize = 10;

    const pollNext = async (pageNo: number, pageSize: number) => {
      console.log(`[doPollingType1] ${pageNo} ${pageSize}`);
      const { data, pagination } = await polling(pageNo, pageSize);
      const { hasFinish } = pagination;
      console.log(`[doPollingType1] recieve data`, data);

      if (!hasFinish) {
        const { pageNo, pageSize } = pagination;
        console.log(`[doPollingType1] ready to do next ${pageNo + 1} ${pageSize}`);
        await wait(1000);
        pollNext(pageNo + 1, pageSize);
      }
    };

    pollNext(startPageNo, pageSize);
  }, []);

  return doPollingType1;
};

export default usePollingType1;
```

我们从一开始的 (1, 10) 开始调用 polling 进行请求，然后每次检查返回的 hasFinish，并调用下一次的 pollNext 方法进行递归，以此完成分页轮询的任务

然而这样的实现过于简单而且存在巨大的问题，下面我们一步步加上一些边界条件和异常链路处理

### 3.1 增加取消机制

第一个是我们可能会需要增加一个取消的机制，来中断轮询操作

- `/fe/src/components/Polling/hooks/usePollingType1.ts`

这时候我们需要加两个变量，作为标记

```ts
const usePollingType1 = () => {
  const isPollingRef = useRef(false);
  const cancelRef = useRef(false);

  // ...
```

一个用来标记轮询状态，一个用来标记是否需要中断轮询

然后我们在每次轮询的开头检查是否需要取消轮询

```ts
    const pollNext = async (pageNo: number, pageSize: number) => {
      if (cancelRef.current) {
        console.log(`[doPollingType1] canceled`);
        cancelRef.current = false;
        isPollingRef.current = false;
        return;
      }
```

最后我们对外透出一个方法，用来设置这个标记

```ts
  const cancelPolling = useCallback(() => {
    if (isPollingRef.current) {
      cancelRef.current = true;
    }
  }, []);

  return [doPollingType1, cancelPolling];
```

如此一来我们就可以调用 cancelPolling 来中止轮询

### 3.2 增加组件卸载时终止轮询机制

第二个特性我们有可能在轮询的过程中卸载了 React 组件，然而对轮询结果的处理往往涉及到组件状态的更新，如此一来就会造成轮询方法去更新已经卸载的 React 组件而产生异常，这时候我们就需要加一个在组件卸载的时候自动取消轮询的机制

- `/fe/src/components/Polling/hooks/usePollingType1.ts`

```ts
const usePollingType1 = () => {
  // ...

  useUnMount(cancelPolling);

  // ...
}
```

这里的 useUnMount 是实现了 componentWillUnMount 生命周期钩子的 hook，具体实现如下

- `/fe/src/hooks/useUnMount.ts`

```ts
import { useEffect } from 'react';

const useUnMount = (fn: () => any) => {
  useEffect(() => fn, []);
};

export default useUnMount;
```

### 3.3 防止并发场景

第三个特性是触发机制的完善，往往触发轮询的方法不只一个入口，而且可能存在多次触发的情况，这就会造成数据更新顺序上的错乱，因为对于同一次的轮询任务一次存在多个异步任务在处理，这时候我们需要加一个类似锁的机制来防止一次触发多个异步任务（我们可以复用刚刚已经加入的 isPollingRef

- `/fe/src/components/Polling/hooks/usePollingType1.ts`

```ts
  const doPollingType1 = useCallback(async () => {
    if (isPollingRef.current) {
      console.log(`[doPollingType1] isPolling, return immediately`);
      return;
    }
```

## 4. 轮询方案 3: 定制轮询引擎

前一种使用 setTimeout 定制轮询用的 hook 已经是一个不错的解决方案，同时对边界条件和异常链路的处理也算是可接受了，不过接下来我们可以再进一步将轮询任务封装成一个轮询引擎，实际上其内部的实现机制并没有改变（换汤不换药hh），不过算是一种不依赖于 React 的更新方案，使其具备更好的框架层级上的可移植性，但是多少也让整个轮询引擎变得更加厚重

- `/fe/src/components/Polling/hooks/usePollingType3.ts`

首先我们先来看看轮询引擎的接口定义

```ts
class PollingEngine {
  constructor(startPageNo: number, pageSize: number, fn: IPollingEngineFn) {
      
  }
  onDataRecieve(callback: IPollingEngineCallback): () => void {}
  start(): void {}
  pause(): void {}
  continue(): void {}
  hasFinish(): boolean {}
}
```

首先构造函数对首页、分页大小、轮询方法进行初始化；然后我们能够使用 start、pause、continue 来控制轮询的开始、暂停、继续等行为；最后可以使用 hasFinish 来检查是否结束、onDataRecieve 来注册数据获取的回调

下面我们来看看轮询引擎的具体实现

### 4.1 核心轮询方法

轮询轮询，最重要的其实就是请求嘛，我们的引擎最重要的就是这个请求方法

- `/fe/src/components/Polling/hooks/usePollingType3.ts`

```ts
class PollingEngine {
  // ...

  private _pasued: boolean = false;
  private _hasFinish: boolean = false;

  private async doPolling() {
    // check pause
    if (this._pasued) {
      console.log('[PollingEngine.doPolling] paused and return');
      this._pasued = false;
      return;
    }

    // polling
    const res = await this._pollingFn(this._pageNo, this._pageSize);

    // notify
    this.notify(res);

    // do next
    if (res.pagination.hasFinish) {
      this._hasFinish = true;
    } else {
      this._pageNo++;
      setTimeout(() => {
        this.doPolling();
      }, 1000);
    }
  }
```

由于我们的引擎支持暂停，所以再加上了一个 _paused 标记，接下来是调用构造函数注入的请求方法 _pollingFn，最后根据返回结果决定是否调用下次轮询

### 4.2 constructor 构造函数 & start、pause、continue 操作

第二部分就是整个轮询引擎对外的接口实现，包括构造函数的状态初始化

- `/fe/src/components/Polling/hooks/usePollingType3.ts`

```ts
class PollingEngine {
  private readonly _startPageNo: number;
  private _pageNo: number;
  private readonly _pageSize: number;
  private readonly _pollingFn: IPollingEngineFn;

  constructor(startPageNo: number, pageSize: number, fn: IPollingEngineFn) {
    this._startPageNo = startPageNo;
    this._pageSize = pageSize;
    this._pollingFn = fn;
  }

  start(): void {
    if (this._isPolling) {
      console.log(`[doPollingType3] isPolling, return immediately`);
      return;
    }
    this._isPolling = true;

    this._pageNo = this._startPageNo;
    this.doPolling();
  }

  pause(): void {
    if (this._isPolling) {
      this._pasued = true;
      this._isPolling = false;
    }
  }

  continue(): void {
    if (this._isPolling) {
      console.log(`[doPollingType3] isPolling, return immediately`);
      return;
    }
    this._isPolling = true;

    this.doPolling();
  }
}
```

### 4.3 回调注册 onDataRecieve & 状态通知 notify

前面 4.1 中其实有个细节，就是每次获取数据的时候，会调用 notify 方法通知所有观察者，也就是依次调用所有回调方法

- `/fe/src/components/Polling/hooks/usePollingType3.ts`

```ts
class PollingEngine {
  // ...

  private callbackList: Set<IPollingEngineCallback> = new Set();

  private notify(res: IPollingResponse) {
    this.callbackList.forEach((callback) => {
      callback(res);
    });
  }
}
```

而这里的观察者则是透过最后一个对外接口提供的注册方法

```ts
class PollingEngine {
  // ...
  
  onDataRecieve(callback: IPollingEngineCallback): () => void {
    this.callbackList.add(callback);

    let cleared = false;
    return () => {
      if (cleared) {
        return;
      }

      this.callbackList.delete(callback);
      cleared = true;
    };
  }
}
```

而这个注册方法直接将取消注册的方法作为返回值返回，省去多定义一个接口的麻烦

### 4.4 自定义钩子

最后要在 React 中使用，再定义一个钩子还是比较方便的，同时用上轮询引擎之后一切的实现就是如此的简单

- `/fe/src/components/Polling/hooks/usePollingType3.ts`

```ts
const usePollingType3 = () => {
  const pollingEngineRef = useRef<PollingEngine>(null);

  useMount(() => {
    const startPageNo = 1;
    const pageSize = 10;

    pollingEngineRef.current = new PollingEngine(startPageNo, pageSize, polling);

    pollingEngineRef.current.onDataRecieve((res) => {
      console.log(`[doPollingType3] recieve data`, res.data);
    });
  });

  const doPollingType3 = useCallback(() => {
    pollingEngineRef.current.start();
  }, []);

  const cancelPolling = useCallback(() => {
    pollingEngineRef.current.pause();
  }, []);

  useUnMount(cancelPolling);

  return [doPollingType3, cancelPolling];
};

export default usePollingType3;
```

就不展示更多的实现细节了，以及实现结果，有兴趣可以下个代码来玩玩哈

# 结语

本篇尝试寻找一个通用的前端轮询解决方案，试图找出轮询的通用写法，然而结论上还是各有千秋。不过在总结的过程中我们注意到，轮询最重要的就是实现方法的链式调用（或是使用定时器也并无不可），封装上无非作为一个普通的方法（如 pollingNext），或是更进一步封装成一个轮询引擎（如 PollingEngine），其实这几种方法都可以。

同时除了核心的轮询手段之外，还有像是取消轮询、防止多个异步任务并发轮询、轮询过程组件卸载等异常链路也是需要注意的点。供大家参考哈。

# 其他资源

## 参考连接

| Title                                                    | Link                                                                                                                               |
| -------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| fetch获取后台的数据，解析Response中的body信息解决方案    | [https://blog.csdn.net/oopxiajun2011/article/details/117123843](https://blog.csdn.net/oopxiajun2011/article/details/117123843)     |
| \[转\] 学会fetch的用法                                   | [https://www.cnblogs.com/chris-oil/p/8430447.html](https://www.cnblogs.com/chris-oil/p/8430447.html)                               |
| Fetch使用方法                                            | [https://www.cnblogs.com/zhuzhenwei918/p/7249691.html](https://www.cnblogs.com/zhuzhenwei918/p/7249691.html)                       |
| JS 实战: 一文了解 5 种文件上传场景(React + Koa 实现)     | [https://blog.csdn.net/weixin_44691608/article/details/119046366](https://blog.csdn.net/weixin_44691608/article/details/119046366) |
| Koa 提交和接收 JSON 表单数据                             | [https://blog.csdn.net/weixin_30367873/article/details/97888705](https://blog.csdn.net/weixin_30367873/article/details/97888705)   |
| fetch请求设置Content-Type失败？                          | [https://segmentfault.com/q/1010000011796178/a-1020000011805467](https://segmentfault.com/q/1010000011796178/a-1020000011805467)   |
| koa2 使用 koa-body 代替 koa-bodyparser 和 koa-multer     | [http://www.ptbird.cn/koa-body.html](http://www.ptbird.cn/koa-body.html)                                                           |
| fetch 发送post请求， 后台接收不到数据，body是undefined。 | [https://www.imooc.com/wenda/detail/534185](https://www.imooc.com/wenda/detail/534185)                                             |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/scheme/scheme_polling_pattern](https://github.com/superfreeeee/Blog-code/tree/main/front_end/scheme/scheme_polling_pattern)
