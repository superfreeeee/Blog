# Koa 项目启动: 从脚手架到自定义项目（&连接 mysql 数据库）

@[TOC](文章目录)

<!-- TOC -->

- [Koa 项目启动: 从脚手架到自定义项目（&连接 mysql 数据库）](#koa-项目启动-从脚手架到自定义项目连接-mysql-数据库)
- [前言](#前言)
- [正文](#正文)
  - [1. 官方脚手架生成项目解析](#1-官方脚手架生成项目解析)
    - [1.1 安装脚手架 + 生成项目](#11-安装脚手架--生成项目)
    - [1.2 目录结构](#12-目录结构)
    - [1.3 中间件配置](#13-中间件配置)
    - [1.4 路由配置](#14-路由配置)
    - [1.5 启动 http 服务器](#15-启动-http-服务器)
    - [1.6 启动截图](#16-启动截图)
  - [2. 定制服务端（自定义 Koa 应用配置）](#2-定制服务端自定义-koa-应用配置)
    - [2.1 识别依赖](#21-识别依赖)
    - [2.2 目录结构](#22-目录结构)
    - [2.3 工具封装](#23-工具封装)
    - [2.4 路由配置](#24-路由配置)
    - [2.5 app 服务](#25-app-服务)
    - [2.6 配置文件](#26-配置文件)
    - [2.7 指令 & 启动](#27-指令--启动)
  - [3. 配置数据库（连接 mysql）](#3-配置数据库连接-mysql)
    - [3.1 安装依赖](#31-安装依赖)
    - [3.2 mysql 连接封装](#32-mysql-连接封装)
    - [3.3 log 路由](#33-log-路由)
    - [3.4 运行结果](#34-运行结果)
  - [4. 讨论](#4-讨论)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

之前用过 express 作为项目的服务端框架。最近想着尝试来用用看更新的 Koa，由 Express 原班团队打造的，适应 async/await 写法的新的服务端框架。整体的架构和思想与 express 几乎一致（连脚手架的注释都没改hh）。

本篇分为三个阶段，第一节我们先利用官方提供的脚手架创建一个项目，然后观察他的内部结构；第二部分我们去除不需要的部分，自己手动搭建一个基于 koa 的后端服务；最后一部分则是基于前一节自定义项目结构下添加 mysql 的数据库连接。

# 正文

## 1. 官方脚手架生成项目解析

第一部分我们先用官方脚手架生成一个项目然后稍微解析一下他都干了啥

### 1.1 安装脚手架 + 生成项目

首先我们可以利用 npm 将命令安装到全局（这里有个bug，如果使用 `yarn global add` 的话会造成缓存错误）

```bash
$ npm install -g koa-generator
```

安装完成之后就可以利用指令 + 项目名称生成项目啦

```bash
$ koa2 koa_launch  # 后续迁移到 default/ 子目录下了哈
```

### 1.2 目录结构

首先先来看看目录结构

```tree
default/
├── app.js
├── bin
│   └── www
├── package.json
├── public
│   ├── images
│   ├── javascripts
│   └── stylesheets
│       └── style.css
├── routes
│   ├── index.js
│   └── users.js
├── views
│   ├── error.pug
│   ├── index.pug
│   └── layout.pug
└── yarn.lock
```

与 exrpess 脚手架几乎一样：

1. 入口在 `/bin/www` 下，然后使用 `app.js` 导出的模块启用 http 服务器
2. `routes` 目录下则是按路由分化各个接口
3. `views` 使用了 pug 的模版引擎，也可以使用 `-e` 参数改为使用 ejs 的模版引擎

下面我们就一个个来看每个模块具体是如何使用 koa 相关的库来搭建

### 1.3 中间件配置

首先我们先来看看 app.js 中的内容，这边我们只关心相关依赖的使用，具体业务逻辑就不放进来了

- `/default/app.js`

先利用 koa 生成 app

```js
const Koa = require('koa')
const app = new Koa()
```

接下来是中间件的相关配置

```js
const views = require('koa-views')
const json = require('koa-json')
const onerror = require('koa-onerror')
const bodyparser = require('koa-bodyparser')
const logger = require('koa-logger')

// error handler
onerror(app)

// middlewares
app.use(bodyparser({
  enableTypes:['json', 'form', 'text']
}))
app.use(json())
app.use(logger())
app.use(require('koa-static')(__dirname + '/public'))

app.use(views(__dirname + '/views', {
  extension: 'pug'
}))
```

- `koa-onerror` 像是全局的异常处理，可以进行重定向或是其他回调
- `koa-json` 则是对输出 json 进行格式化（注意这里是输出 json 不是 request.body）
- `koa-bodyparser` 这个才是对输入请求的请求体进行解析的库
- `koa-logger` 这个库会自动打印比较好看的日志到默认输出
- `koa-static` 这个与 express.static 类似，一样是提供静态资源目录
- `koa-views` 这个提供了模版解析，使后端本身也能提供一定是程度上的页面渲染

### 1.4 路由配置

接下来是路由配置，也就是与每个请求路径一一对应的"路由"，我们以默认的 user 为例

- `/default/routes/users.js`

```js
const router = require('koa-router')()

router.prefix('/users')

router.get('/', function (ctx, next) {
  ctx.body = 'this is a users response!'
})

// ...

module.exports = router
```

我们利用 `koa-router` 这个库提供的构造函数来创建一个路由，然后我们可以设置一个前缀 prefix，他会对请求路径进行拼接

- `/default/app.js`

最后我们只需要在 app.js 里面配置路由就行了

```js
const index = require('./routes/index')
const users = require('./routes/users')

// routes
app.use(index.routes(), index.allowedMethods())
app.use(users.routes(), users.allowedMethods())
```

这里的 allowedMethods 指的是支持如同路由下的 OPTIONS 请求

### 1.5 启动 http 服务器

最后则是启动入口，利用 app 启用一个 http 服务器

- `/default/bin/www`

```js
var app = require('../app');
var http = require('http');

var port = normalizePort(process.env.PORT || '3000');
var server = http.createServer(app.callback());
server.listen(port);
server.on('error', onError);
server.on('listening', onListening);
```

本质上就是透过调用 koa 提供的 `app.callback()` 作为参数，传递给 `http.createServer` 构建出一个新的 http 服务器

### 1.6 启动截图

最后看一下启动之后的样子哈

![](https://picures.oss-cn-beijing.aliyuncs.com/img/koa_launch_1_start_default.png)

## 2. 定制服务端（自定义 Koa 应用配置）

稍微了解 koa 脚手架为我们配置的内容之后，接下来我们要自定义自己的服务端项目，其实本质上就是稍微做一点修改而已，大体的结构不变

### 2.1 识别依赖

开始之前我们先来看要用上哪些依赖

- `/custom/packag.json`

首先是中间件的部分，我们可以直接使用脚手架涵盖的那些依赖：

- 用 `koa-body` 替代 `kao-bodyparser`
- 用 `@koa/router` 替代 `koa-router`
- 添加 `koa-cors` 来支持跨域请求等

```json
{
    "dependencies": {
        "@koa/cors": "^3.1.0",
        "@koa/router": "^10.1.1",
        "debug": "^4.3.2",
        "koa": "^2.13.1",
        "koa-body": "^4.2.0",
        "koa-json": "^2.0.2",
        "koa-logger": "^3.2.1",
        "koa-onerror": "^4.1.0",
        "koa-static": "^5.0.0",
    }
}
```

然后我们希望服务端也能使用 typescript 进行开发，为此我们需要添加 babel 相关的依赖，以及上面中间件的类型说明；同时我们可以使用 nodemon 来作为开发工具，实时监控文件变化并重新启动应用

```json
{
    "devDependencies": {
        "@babel/core": "^7.15.5",
        "@babel/node": "^7.15.4",
        "@babel/preset-env": "^7.15.6",
        "@babel/preset-typescript": "^7.15.0",
        "@types/debug": "^4.1.7",
        "@types/koa": "^2.13.4",
        "@types/koa-json": "^2.0.20",
        "@types/koa-logger": "^3.1.1",
        "@types/koa-static": "^4.0.2",
        "@types/koa__cors": "^3.0.3",
        "@types/koa__router": "^8.0.8",
        "@types/node": "^16.9.4",
        "nodemon": "^2.0.12"
    }
}
```

### 2.2 目录结构

接下来我们其实就跟脚手架搭建的内容没什么两样，稍微移动一下目录结构而已

```tree
/custom
├── babel.config.json
├── nodemon.json
├── package.json
├── public
├── src
│   ├── app.ts
│   ├── index.ts
│   ├── libs
│   │   ├── logger
│   │   │   └── index.ts
│   │   └── router
│   │       └── index.ts
│   └── routes
│       └── index.ts
├── tsconfig.json
└── yarn.lock
```

### 2.3 工具封装

首先我们先对 debug 也就是控制台输出，以及 router 库进行一点封装

- `/custom/src/libs/logger/index.ts`

```ts
import debug from 'debug';

export const createDebugger = (namespace: string) => debug(namespace);

export const logDevServer = debug('server:dev');
```

- `/custom/src/libs/router/index.ts`

```ts
import Router from '@koa/router';

export const createRouter = () => new Router();
```

### 2.4 路由配置

然后我们一样写上路由

- `/custom/src/routes/index.ts`

```ts
import { createRouter } from '../libs/router';

const router = createRouter();

router.get('/', async (ctx, next) => {
  ctx.body = 'welcome koa2';
});

router.get('/string', async (ctx) => {
  ctx.body = 'welcome koa2 in string';
});

router.get('/json', async (ctx, next) => {
  ctx.body = {
    tag: 'welcome koa2 in json',
  };
});

export default router;
```

### 2.5 app 服务

- `/custom/src/app.ts`

接下来一样在 app 中构建服务并配置中间件

```ts
import path from 'path';
import Koa from 'koa';
import koaCors from '@koa/cors';
import koaBody from 'koa-body';
import koaJson from 'koa-json';
import koaLogger from 'koa-logger';
import koaStatic from 'koa-static';

import indexRouter from './routes';
import logRouter from './routes/log';

const app = new Koa();

// middlewares
app
  .use(koaCors())
  .use(
    koaBody({
      multipart: true,
      // encoding: 'gzip',
      formidable: {
        uploadDir: path.resolve(__dirname, '/public/upload/'),
        keepExtensions: true,
        maxFieldsSize: 2 * 1024 * 1024, // 2MB
      },
    })
  )
  .use(koaJson())
  .use(koaLogger())
  .use(koaStatic(path.resolve(__dirname, '/public')));

// routes
app.use(indexRouter.routes()).use(logRouter.routes());

app.on('error', (err, ctx) => {
  console.error('server error', err, ctx);
});

export default app;
```

### 2.6 配置文件

最后我们一共有三个配置文件要搞一下

- `/custom/babel.config.json`

首先是 babel 的配置文件，主要就是配置 es6 的语法跟 typescirpt 编译

```json
{
  "presets": ["@babel/preset-env", "@babel/preset-typescript"]
}
```

- `/custom/tsconfig.json`

第二个则是 typescript 的编译配置文件

```json
{
  "compilerOptions": {
    "esModuleInterop": true
  }
}
```

- `/custom/nodemon.json`

第三个则是配置 nodemon，才能够正确的监听对的文件

```json
{
  "verbose": true,
  "watch": ["src/"],
  "ext": "js,ts,json"
}
```

### 2.7 指令 & 启动

最后我们在 package.json 里面配置一下启动命令就能够正确启动啦

- `/custom/package.json`

```json
{
  "scripts": {
    "start": "DEBUG=server:dev babel-node src/index -x .ts",
    "dev": "DEBUG=server:dev nodemon --exec \"babel-node src/index -x .ts\""
  },
}
```

我们在 dev 开发模式下使用 nodemon 启动监听模式实时重启服务，而正常启动 start 则是使用 babel-node 的方法启动一次就行啦

![](https://picures.oss-cn-beijing.aliyuncs.com/img/koa_launch_2_start_custom.png)

## 3. 配置数据库（连接 mysql）

最后第三阶段我们尝试在我们的 koa 服务器中加上 mysql 的连接，使用数据库进行持久化

### 3.1 安装依赖

本篇使用最原始的 mysql 库直接进行连接，所以需要的库只有一个

```bash
$ yarn add mysql
$ yarn add @types/mysql -D
```

### 3.2 mysql 连接封装

接下来我们要使用 mysql 进行数据库操作，我们可以稍微封装一个类，来屏蔽数据库连接相关的操作

- `/custom/src/libs/mysql/MySQLAdapter.ts`

```ts
import { Connection, ConnectionConfig, createConnection } from 'mysql';
import { logDevServer } from '../logger';

export default class MySQLAdapter {
  private readonly config: ConnectionConfig;
  private connection: Connection | null = null;

  constructor(config: ConnectionConfig) {
    this.config = config;
    this.connect();
  }

  connect(): boolean {
    try {
      this.connection = createConnection(this.config);
      logDevServer('connect success');
      return true;
    } catch (e) {
      logDevServer('connect error', e);
      return false;
    }
  }

  private ensureConnection(): boolean {
    return this.connection !== null || this.connect();
  }

  query<T>(stmt: string): Promise<T> {
    return new Promise((resolve, reject) => {
      if (!this.ensureConnection()) {
        reject(new Error('connect mysql fail'));
        return;
      }

      this.connection.query(stmt, (err, data) => {
        err ? reject(err) : resolve(data);
      });
    });
  }
}
```

首先参数接受连接需要的 config，也就是在初始化的时候配置好，接下来提供两个接口

- `connect` 为进行数据库连接的接口
- `query` 则为实际进行数据库查询的接口
- 私有方法 `ensureConnection` 则是确保每次查询前要连接成功

- `/custom/src/libs/mysql/index.ts`

接下来则是创建一个初始化的实例供外部进行操作

```ts
import MySQLAdapter from './MySQLAdapter';

const db = new MySQLAdapter({
  host: 'localhost',
  user: 'root',
  password: 'xxxxxxxxxx',
  database: 'exampledb',
});

export default db;
```

### 3.3 log 路由

接下来我们开一个新的路由创建两个新的接口，而这两个接口则是利用数据库的查询/插入等操作进行

- `/custom/src/routes/log.ts`


```ts
import { logDevServer } from '../libs/logger';
import db from '../libs/mysql';
import { createRouter } from '../libs/router';

const router = createRouter();

router.prefix('/logger');

// routes ...

export default router;
```

第一个接口是查询接口，查询 log 表的所有数据

```ts
router.get('/all', async (ctx, next) => {
  const dataList = await db.query('select * from log');
  (dataList as any[]).forEach((data) => {
    data.create_time = new Date(data.create_time).getTime();
  });
  ctx.body = dataList;
});
```

第二个接口则是往数据库添加一条数据

```ts
router.post('/add', async (ctx, next) => {
  const model = ctx.request.body;
  logDevServer('model', model);
  const { app, env } = model;
  try {
    const stmt = `insert into log (app, env) values ("${app}", "${env}")`;
    logDevServer('stmt', stmt);
    await db.query(stmt);
    ctx.body = { success: true };
  } catch (e) {
    ctx.body = {
      success: false,
      msg: e,
    };
  }
});
```

最后一样在 app.ts 中注册路由就行啦

```ts
import logRouter from './routes/log';

app.use(logRouter.routes());
```

### 3.4 运行结果

最后我们可以搞一个小前端来测试接口就行啦（当然其实用一个 apifox、postman 等工具也行）

![](https://picures.oss-cn-beijing.aliyuncs.com/img/koa_launch_3_logger_result.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/koa_launch_4_logger_fe.png)

## 4. 讨论

本篇采用直接使用 mysql 连接，使用 connection.query 的方式进行数据库操作，实际上是一个数据类型非常不安全的做法，这可能也是弱类型语言进行数据库连接的通病。更好的做法是为数据库操作都再套上一层类型和数据安全的过滤层，甚至还需要加上如 Sequelize 的 ORM 框架，但是实际上依旧弥补不了弱类型的劣势，同时加上这些配置实际上也不比 java 要简单多少了。

而这可能也是 Node.js 被定位成中台而没办法真正取代后端地位的原因，毕竟 JS 的弱类型优势天生与数据库操作需要的类型稳定性相互冲突

# 结语

本篇就到这里了，算是在探索使用 Koa 开发服务端的同时，对于 Nodejs 做服务端的优劣进行比较，也在尝试的过程中感受出 JS 在数据库操作上的不足。

后期作者可能就比较少对 Nodejs 的后端进行探索，可能还是继续使用如 Java 的 SpringBoot 框架才是正道。不过透过 Koa 来实现数据中台等方法依旧是可以探索的方向之一。

# 其他资源

## 参考连接

| Title                                                                                                             | Link                                                                                                                                   |
| ----------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| node工具之nodemon                                                                                                 | [https://www.jianshu.com/p/f60e14db0b4e](https://www.jianshu.com/p/f60e14db0b4e)                                                       |
| nodemon添加babel支持                                                                                              | [https://www.cnblogs.com/Peter2014/p/13154034.html](https://www.cnblogs.com/Peter2014/p/13154034.html)                                 |
| nodemon - Github                                                                                                  | [https://github.com/remy/nodemon/blob/master/doc/sample-nodemon.md](https://github.com/remy/nodemon/blob/master/doc/sample-nodemon.md) |
| babel-node 和 nodemon                                                                                             | [https://www.cnblogs.com/yangzhou33/p/11504313.html](https://www.cnblogs.com/yangzhou33/p/11504313.html)                               |
| Koa                                                                                                               | [https://koajs.com/](https://koajs.com/)                                                                                               |
| Koa - Github                                                                                                      | [https://github.com/koajs/koa](https://github.com/koajs/koa)                                                                           |
| koa-body - Github                                                                                                 | [https://github.com/koajs/koa-body](https://github.com/koajs/koa-body)                                                                 |
| koa-bodyparser - Github                                                                                           | [https://github.com/koajs/koa-body](https://github.com/koajs/koa-body)                                                                 |
| koa2 使用 koa-body 代替 koa-bodyparser 和 koa-multer                                                              | [http://www.ptbird.cn/koa-body.html](http://www.ptbird.cn/koa-body.html)                                                               |
| koa-logger - Github                                                                                               | [https://github.com/koajs/logger](https://github.com/koajs/logger)                                                                     |
| koa-json - Github                                                                                                 | [https://github.com/koajs/json](https://github.com/koajs/json)                                                                         |
| koa-router - Github                                                                                               | [https://github.com/koajs/router/tree/master/lib](https://github.com/koajs/router/tree/master/lib)                                     |
| koa-onerror 使用教程                                                                                              | [https://blog.csdn.net/nullccc/article/details/113851337](https://blog.csdn.net/nullccc/article/details/113851337)                     |
| 每天一个npm包：koa-convert                                                                                        | [https://zhuanlan.zhihu.com/p/150867763](https://zhuanlan.zhihu.com/p/150867763)                                                       |
| HTTP的options方法和 koa-router 的 allowedmethods 方法                                                             | [https://blog.csdn.net/m0_48446542/article/details/109113438](https://blog.csdn.net/m0_48446542/article/details/109113438)             |
| koa+mysql 使用教程                                                                                                | [https://www.jianshu.com/p/b251792f747f](https://www.jianshu.com/p/b251792f747f)                                                       |
| debug - Github                                                                                                    | [https://github.com/visionmedia/debug](https://github.com/visionmedia/debug)                                                           |
| debug - npm                                                                                                       | [https://www.npmjs.com/package/debug](https://www.npmjs.com/package/debug)                                                             |
| supports-color - npm                                                                                              | [https://www.npmjs.com/package/supports-color](https://www.npmjs.com/package/supports-color)                                           |
| MySQL ALTER命令 - 菜鸟教程                                                                                        | [https://www.runoob.com/mysql/mysql-alter.html](https://www.runoob.com/mysql/mysql-alter.html)                                         |
| 解决Navicat连接不上MySql服务器报错：Client does not support authentication protocol requested by server； conside | [https://blog.csdn.net/weixin_43111077/article/details/108811949](https://blog.csdn.net/weixin_43111077/article/details/108811949)     |
| koa框架连接MySQL数据库并使用session                                                                               | [https://www.cnblogs.com/huibaoqiang/p/12103675.html](https://www.cnblogs.com/huibaoqiang/p/12103675.html)                             |
| Postman报错：“error“: “Unsupported Media Type“的解决方法                                                          | [https://blog.csdn.net/weixin_44532671/article/details/114576639](https://blog.csdn.net/weixin_44532671/article/details/114576639)     |
| koa-body unsupported media type                                                                                   | [https://www.jianshu.com/p/e53492b0d163](https://www.jianshu.com/p/e53492b0d163)                                                       |
| Nodejs之ORM框架                                                                                                   | [https://www.jianshu.com/p/0738e29d8af3](https://www.jianshu.com/p/0738e29d8af3)                                                       |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/express/koa_launch](https://github.com/superfreeeee/Blog-code/tree/main/front_end/express/koa_launch)
