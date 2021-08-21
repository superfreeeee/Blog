# HTML5 新特性: Web Worker 的创建与使用(webpack + TS 环境)

@[TOC](文章目录)

<!-- TOC -->

- [HTML5 新特性: Web Worker 的创建与使用(webpack + TS 环境)](#html5-新特性-web-worker-的创建与使用webpack--ts-环境)
- [前言](#前言)
- [正文](#正文)
  - [1. 基础使用](#1-基础使用)
    - [1.1 Worker 定义](#11-worker-定义)
    - [1.2 Worker 使用](#12-worker-使用)
  - [2. worker-loader](#2-worker-loader)
    - [2.1 webpack 配置](#21-webpack-配置)
    - [2.2 Worker 定义](#22-worker-定义)
    - [2.3 Worker 使用](#23-worker-使用)
    - [2.4 TS 环境下的配置](#24-ts-环境下的配置)
  - [3. 实践示例：计时器](#3-实践示例计时器)
    - [3.1 Worker 定义](#31-worker-定义)
    - [3.2 Worker 使用](#32-worker-使用)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

我们都知道 JS 一直都是单线程的语言，并透过事件循环来提供异步操作的方法避免堵塞。

本篇来介绍一个 HTML5 新出的特性 **Web Worker**，它能够真正的为 JS 带来多线程的特性的一套规范，后续还有更多的妙用。本篇主要介绍基础的使用方式，以及在 TS 环境下的一些配置问题。

本篇不会对 Worker 的一些定义做过多的描述，主要偏重在实践并写到真实项目里头，相关的概念还是先参考一些博客会好一些

# 正文

## 1. 基础使用

首先第一版我们先搞最基础的版本，也就是直接调用浏览器最原始的 API 的使用方式

```js
const worker = new Worker('sample.js')
```

实际上他会以同源的方式请求一个 js 文件，并加载作为 Worker 来执行

### 1.1 Worker 定义

接下来我们要定义一个 Worker 线程内要执行的脚本

- `/src/workers/test1.worker.js`

```js
const PREFIX_WORKER1 = '[Worker1]';

self.onmessage = (event) => {
  const msg = event.data;
  console.log(`${PREFIX_WORKER1} receive msg in worker: ${msg}`);

  const greeting = `${msg} from test1.worker.js`;
  postMessage(greeting);
};
```

### 1.2 Worker 使用

下面我们看看项目内的用法

- `/src/layouts/Test1.tsx`

```ts
const Test1 = () => {
  const createWorker = () => {
    const worker = new Worker('workers/test1.worker.js');

    worker.onmessage = (event) => {
      const msg = event.data;
      console.log(`${PREFIX_TEST1} worker.onmessage: ${msg}`);
      worker.terminate();
      console.log(`${PREFIX_TEST1} worker finished`);
    };

    const msg = 'Hello World';
    console.log(`${PREFIX_TEST1} worker.postMessage: ${msg}`);
    worker.postMessage('Hello World');
  };

  return (
    <div>
      <h2>Test1 - Basic Worker</h2>
      <button onClick={createWorker}>createWorker</button>
    </div>
  );
};
```

本质上就是 1 个创建和 3 个函数的操作

1. 创建 Worker

```ts
const worker = new Worker('workers/test1.worker.js');
```

2. 发送消息

```ts
worker.postMessage('Hello World');
```

3. 监听消息

```ts
worker.onmessage = (event) => {/* ... */}
```

4. 终止 Worker

```ts
worker.terminate();
```

实现效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/html5_web_worker_sample1.png)

## 2. worker-loader

第二种场景我们还可以在 webpack 中引入 Worker 的特性，甚至用起来比原生的更优雅

### 2.1 webpack 配置

首先是 `webpack.config.js` 配置要加上

```js
module.exports = {
  // ...
  module: {
    rules: [
     {
        test: /\.worker\.(js|jsx|ts|tsx)$/,
        exclude: /node_modules/,
        use: ['worker-loader', 'ts-loader'],
      },
    ]
  },
  // ...
}
```

### 2.2 Worker 定义

接下来我们在定义一个新的 Worker

- `/src/workers/test2.worker.ts`

```ts
const PREFIX_WORKER2 = '[Worker2]';

self.onmessage = (event) => {
  const msg = event.data;
  console.log(`${PREFIX_WORKER2} receive msg in worker: ${msg}`);

  const greeting = `${msg} from test2.worker.ts`;
  postMessage(greeting);
};
```

### 2.3 Worker 使用

然后我们就可以像引入一个模块一样引入一个 Worker 脚本

- `/src/layouts/Test2.tsx`

```ts
import React from 'react';

import { PREFIX_TEST2 } from '@utils/prefixs';
import Worker from '@workers/test2.worker.ts';

const Test2 = () => {
  const createWorker = () => {
    const worker = new Worker();

    worker.onmessage = (event) => {
      const msg = event.data;
      console.log(`${PREFIX_TEST2} worker.onmessage: ${msg}`);
      worker.terminate();
      console.log(`${PREFIX_TEST2} worker finished`);
    };

    const msg = 'Hello World';
    console.log(`${PREFIX_TEST2} worker.postMessage: ${msg}`);
    worker.postMessage('Hello World');
  };

  return (
    <div>
      <h2>Test2 - Worker Loader</h2>
      <button onClick={createWorker}>createWorker</button>
    </div>
  );
};

export default Test2;
```

与第一次的差别在于

```ts
import Worker from '@workers/test2.worker.ts';

const worker = new Worker();
```

也就是说接下来 webpack 可以将 Worker 用的脚本一并处理打包起来，而不需要我们额外再去管理 worker 的部署

### 2.4 TS 环境下的配置

然而上述的写法 TS 会给出一大堆报错，这时候需要补上一些配置

- `tsconfig.json`

```ts
{
  "compilerOptions": {
    "lib": ["WebWorker", "ScriptHost", "DOM"],
    "allowJs": false,
  }
}
```

除此之外，默认的 Worker 构造函数是需要传入一个脚本路径名的，但是在 webpack 下我们直接 import 然后就使用无参数构造函数了，所以我们需要额外建立一个类型声明

- `/src/types/worker.d.ts`

```ts
declare module '*.worker.ts' {
  class WebpackWorker extends Worker {
    constructor();
  }
  export default WebpackWorker;
}
```

最终效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/html5_web_worker_sample2.png)

## 3. 实践示例：计时器

最后我们摆上一个用 Web Worker 做的时钟范例

### 3.1 Worker 定义

- `/src/workers/test3.worker.ts`

```ts
type MessageType = 'RESET' | 'SKIP';

let count = 0;
let skipOnce = false;

self.onmessage = (e: MessageEvent<{ type: MessageType }>) => {
  const { type } = e.data;
  switch (type) {
    case 'RESET':
      count = 0;
      break;
    case 'SKIP':
      skipOnce = true;
      break;
  }
};

const SEC = 1000;
setInterval(() => {
  if (skipOnce) {
    skipOnce = false;
  } else {
    count++;
  }
  self.postMessage({ currentTime: new Date(), count });
}, SEC);
```

### 3.2 Worker 使用

- `/src/layouts/Test3.tsx`

```ts
import React, { useEffect, useRef, useState } from 'react';

import Worker from '@workers/test3.worker.ts';

const useWorker = () => {
  const [currentTime, setCurrentTime] = useState(new Date());
  const [count, setCount] = useState(0);

  const workerRef = useRef(null);

  useEffect(() => {
    const worker = new Worker();

    worker.onmessage = (e) => {
      const { currentTime, count } = e.data;
      setCurrentTime(currentTime);
      setCount(count);
    };

    workerRef.current = worker;
  }, []);

  const reset = () => {
    workerRef.current?.postMessage({ type: 'RESET' });
    setCount(0);
    console.log(workerRef.current);
  };

  const skip = () => {
    workerRef.current?.postMessage({ type: 'SKIP' });
    console.log(workerRef.current);
  };

  const terminate = () => {
    workerRef.current?.terminate();
    console.log(workerRef.current);
  };

  return [
    { currentTime, count },
    { reset, skip, terminate },
  ];
};

const Test3 = () => {
  const [{ currentTime, count }, { reset, skip, terminate }] = useWorker();

  return (
    <div>
      <h2>Test3 - Timer by Worker</h2>
      <h3>currentTime: {currentTime.toString()}</h3>
      <h3>count: {count}</h3>
      <button onClick={reset}>reset</button>
      <button onClick={skip}>skip</button>
      <button onClick={terminate}>terminate</button>
    </div>
  );
};

export default Test3;
```

最终效果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/html5_web_worker_sample3.gif)

# 结语

Web Worker 的最大好处就是在于他真正的创建一个独立于 JS 主线程的新线程，能真正实现并行运行。

# 其他资源

## 参考连接

| Title                                                                         | Link                                                                                                                               |
| ----------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| Web Worker 使用教程 - 阮一峰                                                  | [https://www.ruanyifeng.com/blog/2018/07/web-worker.html](https://www.ruanyifeng.com/blog/2018/07/web-worker.html)                 |
| worker-loader                                                                 | [https://cloud.tencent.com/developer/section/1477547](https://cloud.tencent.com/developer/section/1477547)                         |
| The import path cannot end with a ‘.ts‘ extension                             | [https://blog.csdn.net/peade/article/details/117534994](https://blog.csdn.net/peade/article/details/117534994)                     |
| Typescript error “Cannot write file xxx because it would overwrite input file | [https://blog.csdn.net/weixin_43459866/article/details/116356968](https://blog.csdn.net/weixin_43459866/article/details/116356968) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/html/html5_web_worker](https://github.com/superfreeeee/Blog-code/tree/main/front_end/html/html5_web_worker)
