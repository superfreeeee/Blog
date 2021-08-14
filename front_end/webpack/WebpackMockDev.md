# Webpack 实战: 本地 mock 开发模式实践

@[TOC](文章目录)

<!-- TOC -->

- [Webpack 实战: 本地 mock 开发模式实践](#webpack-实战-本地-mock-开发模式实践)
- [前言](#前言)
- [正文](#正文)
  - [1. 环境准备](#1-环境准备)
    - [1.1 客户端环境：搭建 webpack 项目](#11-客户端环境搭建-webpack-项目)
    - [1.2 服务端环境：搭建 express 项目](#12-服务端环境搭建-express-项目)
  - [2. 接入 API](#2-接入-api)
    - [2.1 基础配置信息 & mock 数据](#21-基础配置信息--mock-数据)
    - [2.2 接口配置定义](#22-接口配置定义)
    - [2.3 请求方法封装](#23-请求方法封装)
    - [2.4 测试](#24-测试)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

本篇要带来的并不是什么新技术或是三方库的用法，而是一种项目上的开发模式实践分享

本篇要介绍的是一种 mock 开发模式，我们知道在一般的 web 系统中，多数会有一个后端来为自己提供服务，然而真正进入开发的时候我们可能会遇到后端尚未准备好接口，堵塞了自己的开发进度；这时候我们就可以使用 mock 开发，透过建立 mock 接口的形式来模拟后端返回数据的情况。

下面我们就来看看是如何实践的

# 正文

## 1. 环境准备

### 1.1 客户端环境：搭建 webpack 项目

首先我们要准备一个 webpack 项目的环境（当然其他环境也行，不过本篇的重点是这个环境启动开发服务器的时候要能够访问到本地的静态资源

相关工具可以参考作者自己写的一个脚手架：[superfreeeee/general-cli - Github](https://github.com/superfreeeee/general-cli)

然后是 webpack 的配置项里头需要加入配置

- `webpack.config.js`

```js
{
    // ...
    devServer: {
        contentBase: '.',
    },
    // ...
}
```

这个 contentBase 的作用在于当我们启动 webpack-dev-server 的时候同时将本地目录作为根目录，我们后面就可以直接透过根目录访问静态资源，也就是我们的 mock 文件

### 1.2 服务端环境：搭建 express 项目

接下来是搭建一个服务端环境来提供我们的服务，本篇选用 express，你可以用官方脚手架搭也可以自己撸，当然我的脚手架也提供了其中一个选项

然后我们的后端需要提供 test1、2、3 三个接口来做示例

- `/be/src/index.ts`

```ts
import chalk from 'chalk';
import express from 'express';
import cors from 'cors';
import bodyParser from 'body-parser';

const app = express();

app.use(cors());
app.use(bodyParser.urlencoded({ extended: false }));
app.use(bodyParser.json());

app.get('/test1', (req, res, next) => {
  res.send({ code: 1, msg: 'test 1 response' });
});

app.get('/test2', (req, res, next) => {
  res.send({ code: 2, msg: 'test 2 response' });
});

app.post('/test3', (req, res, next) => {
  res.send({ code: 3, msg: 'test 3 response', body: req.body });
});

const PORT = 3001;

app.listen(PORT, () => {
  console.clear();
  console.log(chalk.bold('start express server ...'));
  console.log(`express listen at http://localhost:${PORT}`);
});
```

## 2. 接入 API

接下来就可以开始正式的开发

首先我们要接入后端提供的 api 时，通常会先将接口描述封装成一些方法，然后直接调用；更进一步，我们希望透过更简单的配置来提高这些方法的可扩展性

### 2.1 基础配置信息 & mock 数据

首先我们写个 config 来配置一些全局相关的信息

- `/src/api/config.ts`

```ts
export default {
  mock: {
    use: false,
    host: '/mock',
  },
  host: 'http://localhost:3001',
};
```

我们的后端启动在 3001，然后 mock 资源放在项目根目录之下如下图，对应了三个接口返回的数据

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_mock_dev_0_files.png)

### 2.2 接口配置定义

接下来则是项目内定义接口

- `/src/api/index.ts`

首先是三个接口的枚举类型和相关信息

```ts

enum EApiMethod {
  GET = 'GET',
  POST = 'POST',
}

interface IRouteInfo {
  method: EApiMethod;
  path: string;
}

type IApiInfo = {
  [route in API_ROUTE]: IRouteInfo;
};

export enum API_ROUTE {
  TEST1 = 'test1',
  TEST2 = 'test2',
  TEST3 = 'test3',
}

/**
 * 服务端 API 注册表
 */
const API_INFO: IApiInfo = {
  [API_ROUTE.TEST1]: {
    method: EApiMethod.GET,
    path: '/test1',
  },
  [API_ROUTE.TEST2]: {
    method: EApiMethod.GET,
    path: '/test2',
  },
  [API_ROUTE.TEST3]: {
    method: EApiMethod.POST,
    path: '/test3',
  },
};
```

### 2.3 请求方法封装

接下来则是请求方法的封装

- `/src/api/index.ts`

首先是我们重载两个方法签名，分别对应 GET、POST 两种请求方法

```ts
/**
 * GET 请求
 * @param route
 */
export function apiRequest(route: API_ROUTE): Promise<any>;

/**
 * POST 请求
 * @param route
 * @param body
 */
export function apiRequest(route: API_ROUTE, body: any): Promise<any>;
```

然后在具体实现的方法体里面判断是否开启 mock 模式（本篇使用 superagent 库来发起 http 请求的

```ts
/**
 * apiRequest 实现
 * @param route
 * @param body
 */
export function apiRequest(route: API_ROUTE, body?: any): Promise<any> {
  const { method, path } = API_INFO[route];

  const useMock = config.mock.use;
  const host = useMock ? config.mock.host : config.host;

  /**
   * 本地 mock
   */
  if (useMock) {
    const mockUrl = `${host}${path}.json`;
    return superagent.get(mockUrl).send();
  }

  const url = `${host}${path}`;

  switch (method) {
    case EApiMethod.POST:
      return superagent.post(url).send(body);
    case EApiMethod.GET:
    default:
      return superagent.get(url).send();
  }
}
```

### 2.4 测试

最后我们写一个页面用于测试

- `/src/App.tsx`

```ts
import styles from './index.module.scss';

import React, { useState } from 'react';
import classNames from 'classnames';

import useDocumentTitle from '@hooks/useDocumentTitle';
import { apiRequest, API_ROUTE } from '@api';
import config from '@api/config';

const useApi = () => {
  const [useMock, setUseMock] = useState(config.mock.use);

  const toggleUseMock = () => {
    setUseMock(!useMock);
    config.mock.use = !useMock;
  };

  const apiLog = (testName: string, p: Promise<any>) => {
    p.then((res) => {
      console.log(`[api ${testName} success]`, res.body);
    }).catch((err) => {
      console.log(`[api ${testName} error]`, err);
    });
  };

  const test1 = () => {
    apiLog(API_ROUTE.TEST1, apiRequest(API_ROUTE.TEST1));
  };

  const test2 = () => {
    apiLog(API_ROUTE.TEST2, apiRequest(API_ROUTE.TEST2));
  };

  const test3 = () => {
    apiLog(API_ROUTE.TEST3, apiRequest(API_ROUTE.TEST3, { from: 'test3' }));
  };

  return {
    useMock,
    toggleUseMock,
    test1,
    test2,
    test3,
  };
};

const App: React.FC<{}> = () => {
  useDocumentTitle('Webpack Mock');

  const { useMock, toggleUseMock, test1, test2, test3 } = useApi();

  return (
    <div className={classNames(styles.app)}>
      <h1>Webpack Development in Mock mode</h1>
      <div>
        <h3 style={{ margin: '1em 0' }}>use mock: {`${useMock}`}</h3>
      </div>
      <div className={styles.btns}>
        <button onClick={toggleUseMock}>toggle mock</button>
        <button onClick={test1}>Test1</button>
        <button onClick={test2}>Test2</button>
        <button onClick={test3}>Test3</button>
      </div>
    </div>
  );
};

export default App;
```

首先是正常请求后端的场景

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_mock_dev_1_before.png)

接下来是开启 mock 模式后的请求结果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_mock_dev_2_after.png)

# 结语

主要就是提供大家一个思路，用于封装各自的 api，同时可以考虑到在切换模式的时候一键切换数据源。

# 其他资源

## 参考连接

| Title                                | Link                                                                                                                                                               |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| SuperAgent                           | [https://visionmedia.github.io/superagent/#post--put-requests](https://visionmedia.github.io/superagent/#post--put-requests)                                       |
| webpack简单搭建localhost访问静态资源 | [https://cloud.tencent.com/developer/article/1619966](https://cloud.tencent.com/developer/article/1619966)                                                         |
| TypeScript 中的方法重载              | [https://www.cnblogs.com/Wayou/p/function_overload_in_typescript.html](https://www.cnblogs.com/Wayou/p/function_overload_in_typescript.html)                       |
| bodyParser is deprecated express 4   | [https://stackoverflow.com/questions/24330014/bodyparser-is-deprecated-express-4](https://stackoverflow.com/questions/24330014/bodyparser-is-deprecated-express-4) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/webpack/webpack_mock_dev](https://github.com/superfreeeee/Blog-code/tree/main/front_end/webpack/webpack_mock_dev)
