# Redux 源码解析(三): compose & applyMiddleware API

@[TOC](文章目录)

<!-- TOC -->

- [Redux 源码解析(三): compose & applyMiddleware API](#redux-源码解析三-compose--applymiddleware-api)
- [前言](#前言)
- [正文](#正文)
  - [1. compose 源码解析](#1-compose-源码解析)
    - [1.1 方法签名](#11-方法签名)
    - [1.2 源码实现](#12-源码实现)
  - [2. applyMiddleware 源码解析](#2-applymiddleware-源码解析)
    - [2.1 类型定义](#21-类型定义)
    - [2.2 方法签名](#22-方法签名)
    - [2.3 源码实现](#23-源码实现)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 前言

Redux 源码系列转眼间就来到最后一篇了，本篇要来介绍最后剩下的两个 API：compose 和 applyMiddleware

# 正文

## 1. compose 源码解析

compose 函数还是比较常见的，属于函数式编程的标配，作用是将

```ts
compose(f1, f2, f3, f4)
```

变成

```ts
f1(f2(f3(f4())))
```

的调用

然而在 Redux 源码当中其实这个 compose 主要是用于实现 applyMiddleware 时对多个 middleware 的包装

### 1.1 方法签名

所以接下来我们先看看 compose 的方法签名

- `/src/compose.ts`（阅读笔记：`/src/compose.ts/exports.ts`）

```ts
type Func<T extends any[], R> = (...a: T) => R

export default function compose(): <R>(a: R) => R;

export default function compose<F extends Function>(f: F): F;

/* two functions */
export default function compose<A, T extends any[], R>(
  f1: (a: A) => R,
  f2: Func<T, A>
): Func<T, R>;

/* three functions */
export default function compose<A, B, T extends any[], R>(
  f1: (b: B) => R,
  f2: (a: A) => B,
  f3: Func<T, A>
): Func<T, R>;

/* four functions */
export default function compose<A, B, C, T extends any[], R>(
  f1: (c: C) => R,
  f2: (b: B) => C,
  f3: (a: A) => B,
  f4: Func<T, A>
): Func<T, R>;

/* rest */
export default function compose<R>(
  f1: (a: any) => R,
  ...funcs: Function[]
): (...args: any[]) => R;

export default function compose<R>(...funcs: Function[]): (...args: any[]) => R;
```

这里可以看出 compose 的核心在于组合多个单参数的函数，然后以最后一个函数的方法签名作为导出函数的方法签名

### 1.2 源码实现

compose 的源码还是比较简单的，过一下就行了

- `/src/compose.ts`（阅读笔记：`/src/compose.ts/implements.ts`）

```ts
export default function compose(...funcs: Function[]) {
  if (funcs.length === 0) {
    // infer the argument type so it is usable in inference down the line
    return <T>(arg: T) => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce(
    (a, b) =>
      (...args: any) =>
        a(b(...args))
  )
}
```

## 2. applyMiddleware 源码解析

相对于 compose 来说，applyMiddleware 才是我们更关注的 api，用于为 store 添加中间件的方法

### 2.1 类型定义

接下来我们先来看看相关的类型定义

- `/src/types/middleware.ts`（阅读笔记：`/src/types/middleware.ts`）

```ts
import { Dispatch } from './store'

export interface MiddlewareAPI<D extends Dispatch = Dispatch, S = any> {
  dispatch: D
  getState(): S
}

export interface Middleware<
  _DispatchExt = {}, // TODO: remove unused component (breaking change)
  S = any,
  D extends Dispatch = Dispatch
> {
  (api: MiddlewareAPI<D, S>): (
    next: D
  ) => (action: D extends Dispatch<infer A> ? A : never) => any
}
```

很简单，一个是中间件的第一个参数的类型，一个则是中间件的方法签名

### 2.2 方法签名

接下来我们看看 applyMiddleware 的方法签名

- `/src/applyMiddleware.ts`（阅读笔记：`/src/applyMiddleware.ts/exports.ts`）

```ts
import compose from './compose'
import { Middleware, MiddlewareAPI } from './types/middleware'
import { AnyAction } from './types/actions'
import {
  StoreEnhancer,
  Dispatch,
  PreloadedState,
  StoreEnhancerStoreCreator
} from './types/store'
import { Reducer } from './types/reducers'

export default function applyMiddleware(): StoreEnhancer;

export default function applyMiddleware<Ext1, S>(
  middleware1: Middleware<Ext1, S, any>
): StoreEnhancer<{ dispatch: Ext1 }>;

export default function applyMiddleware<Ext1, Ext2, S>(
  middleware1: Middleware<Ext1, S, any>,
  middleware2: Middleware<Ext2, S, any>
): StoreEnhancer<{ dispatch: Ext1 & Ext2 }>;

export default function applyMiddleware<Ext1, Ext2, Ext3, S>(
  middleware1: Middleware<Ext1, S, any>,
  middleware2: Middleware<Ext2, S, any>,
  middleware3: Middleware<Ext3, S, any>
): StoreEnhancer<{ dispatch: Ext1 & Ext2 & Ext3 }>;

export default function applyMiddleware<Ext1, Ext2, Ext3, Ext4, S>(
  middleware1: Middleware<Ext1, S, any>,
  middleware2: Middleware<Ext2, S, any>,
  middleware3: Middleware<Ext3, S, any>,
  middleware4: Middleware<Ext4, S, any>
): StoreEnhancer<{ dispatch: Ext1 & Ext2 & Ext3 & Ext4 }>;

export default function applyMiddleware<Ext1, Ext2, Ext3, Ext4, Ext5, S>(
  middleware1: Middleware<Ext1, S, any>,
  middleware2: Middleware<Ext2, S, any>,
  middleware3: Middleware<Ext3, S, any>,
  middleware4: Middleware<Ext4, S, any>,
  middleware5: Middleware<Ext5, S, any>
): StoreEnhancer<{ dispatch: Ext1 & Ext2 & Ext3 & Ext4 & Ext5 }>;

export default function applyMiddleware<Ext, S = any>(
  ...middlewares: Middleware<any, S, any>[]
): StoreEnhancer<{ dispatch: Ext }>;
```

看起来很复杂，其实本质上就是可以传入多个中间件并封装成一个 enhancer 就对了

### 2.3 源码实现

实际上，enhancer 名字看起来很高大上，其实没有那么复杂，且听我娓娓道来

- `/src/applyMiddleware.ts`（阅读笔记：`/src/applyMiddleware.ts/implements.ts`）

```ts
export default function applyMiddleware(...middlewares: Middleware[]): StoreEnhancer<any> {
  return (createStore: StoreEnhancerStoreCreator) =>
    <S, A extends AnyAction>(reducer: Reducer<S, A>, preloadedState?: PreloadedState<S>) => {
      const store = createStore(reducer, preloadedState)
      let dispatch: Dispatch = () => {
        // throw error
      }

      const middlewareAPI: MiddlewareAPI = {
        getState: store.getState,
        dispatch: (action, ...args) => dispatch(action, ...args),
      }
      const chain = middlewares.map((middleware) => middleware(middlewareAPI))
      dispatch = compose<typeof dispatch>(...chain)(store.dispatch)

      return {
        ...store,
        dispatch,
      }
    }
}
```

首先定义一个基础的 dispatch 方法，但是禁止中间件在创建的过程就立即调用

```ts
      let dispatch: Dispatch = () => {
        // throw error
      }
```

接下来定义一个 api，负责传递 getState 和 dispatch 方法给每个中间件

```ts
      const middlewareAPI: MiddlewareAPI = {
        getState: store.getState,
        dispatch: (action, ...args) => dispatch(action, ...args),
      }
```

接下来就是将 api 传入中间件并用 compose 串起来

```ts
      const chain = middlewares.map((middleware) => middleware(middlewareAPI))
      dispatch = compose<typeof dispatch>(...chain)(store.dispatch)
```

也就是说当我们执行新的 dispatch 的时候，会由前至后，调用第一个 middleware 的 dispatch，而如果中间件调用 next 才会将 action 传递给下一个中间件来调用

# 结语

redux 的源码真的是相对简单很多，与网上说的一样非常适合初学者来学习，不过本篇也算是一个特别难的逻辑在于使用 compose 组合中间件的部分，因为中间件是一个有三层参数的科里化函数，还是比较复杂的。

Redux 源码系列就到这里结束啦，下面作者会在继续找找一些常用非常重要的库来带大家解析源码啦～

# 其他资源

## 参考连接

| Title                   | Link                                                                                 |
| ----------------------- | ------------------------------------------------------------------------------------ |
| applyMiddleware - Redux | [https://redux.js.org/api/applymiddleware](https://redux.js.org/api/applymiddleware) |
| redux - Github          | [https://github.com/reduxjs/redux](https://github.com/reduxjs/redux)                 |

## 阅读笔记参考

[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/redux-4.1.1](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/redux-4.1.1)
