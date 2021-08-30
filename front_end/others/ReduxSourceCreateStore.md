# Redux 源码解析(一): createStore API

@[TOC](文章目录)

<!-- TOC -->

- [Redux 源码解析(一): createStore API](#redux-源码解析一-createstore-api)
- [前言](#前言)
- [正文](#正文)
  - [0. 背景知识](#0-背景知识)
    - [0.1 源代码版本：4.1.1, TS 实现](#01-源代码版本411-ts-实现)
    - [0.2 源码目录结构](#02-源码目录结构)
  - [1. createStore 源码解析](#1-createstore-源码解析)
    - [1.1 导出类型](#11-导出类型)
    - [1.2 状态 & 返回](#12-状态--返回)
    - [1.3 预处理](#13-预处理)
    - [1.4 获取状态 getState](#14-获取状态-getstate)
    - [1.5 订阅状态 subscribe](#15-订阅状态-subscribe)
    - [1.6 修改状态 dispatch](#16-修改状态-dispatch)
    - [1.7 置换核心函数 replaceReducer](#17-置换核心函数-replacereducer)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 前言

在 SPA 的前端架构底下，状态管理从来都是绕不开的话题。

本篇将要带大家来解析著名状态管理库 Redux 的源代码。与 Vuex 不同的是，Redux、MobX 这类的状态管理库是独立于任何前端框架的状态管理实现，例如 Redux 要利用 react-redux 作为桥梁才能更好地在 React 框架下使用。

Redux 的源码算是比较清晰容易理解的，为了充分理解状态管理的思想，同时更进一步的学习如何手写一个开源框架库，充分处理各种边界情况和编写容易使用的 api，我们的源码拆解将会分成更细的力度，细细品味每个部分的实现和思想

# 正文

本篇作为 Redux 源码解析的首篇，我们先来解析使用 Redux 创建 store 状态管理对象必用的 api 之一 - createStore

## 0. 背景知识

在开始之前我们先过一些基础的背景知识

### 0.1 源代码版本：4.1.1, TS 实现

由于 Redux 的 release 版本都是编译后的 js 文件，所以本篇使用 master 分支上的 `4.1.1` 版本的源代码，使用 TS 编写

### 0.2 源码目录结构

接下来是 Redux 源码的目录结构

```tree
redux-master/
└── src/
    ├── applyMiddleware.ts
    ├── bindActionCreators.ts
    ├── combineReducers.ts
    ├── compose.ts
    ├── createStore.ts
    ├── index.ts
    ├── types/
    │   ├── actions.ts
    │   ├── middleware.ts
    │   ├── reducers.ts
    │   └── store.ts
    └── utils/
        ├── actionTypes.ts
        ├── formatProdErrorMessage.ts
        ├── isPlainObject.ts
        ├── kindOf.ts
        ├── symbol-observable.ts
        └── warning.ts
```

- `types` 目录下是各个 api 涉及的类型定义
- `utils` 则是一些辅助的工具函数
- 剩下位于 src 根目录的就是 redux 的几个主要的 api 实现

本篇主要介绍 `createStore.ts` 导出的 `createStore` API

## 1. createStore 源码解析

### 1.1 导出类型

首先我们先看看 createStore 导出的类型有哪些

- `/src/createStore.ts`（源码笔记：`/src/createStore.ts/exports.ts`）

```ts
import $$observable from './utils/symbol-observable'

import {
  Store,
  PreloadedState,
  StoreEnhancer,
  Dispatch,
  Observer,
  ExtendState
} from './types/store'
import { Action } from './types/actions'
import { Reducer } from './types/reducers'
import ActionTypes from './utils/actionTypes'
import isPlainObject from './utils/isPlainObject'
import { kindOf } from './utils/kindOf'

export default function createStore<S, A extends Action, Ext = {}, StateExt = never>(
  reducer: Reducer<S, A>,
  enhancer?: StoreEnhancer<Ext, StateExt>
): Store<ExtendState<S, StateExt>, A, StateExt, Ext> & Ext;

export default function createStore<S, A extends Action, Ext = {}, StateExt = never>(
  reducer: Reducer<S, A>,
  preloadedState?: PreloadedState<S>,
  enhancer?: StoreEnhancer<Ext, StateExt>
): Store<ExtendState<S, StateExt>, A, StateExt, Ext> & Ext;
```

我们看到实际上一共导出了两种 createStore 的方法签名重载，分别对应两个和三个参数的调用方式

### 1.2 状态 & 返回

接下来我们来看看 createStore 方法内定义的一些局部变量和返回对象

- `/src/createStore.ts`（源码笔记：`/src/createStore.ts/state.ts`）

```ts
export default function createStore<
  S,
  A extends Action,
  Ext = {},
  StateExt = never
>(
  reducer: Reducer<S, A>,
  preloadedState?: PreloadedState<S> | StoreEnhancer<Ext, StateExt>,
  enhancer?: StoreEnhancer<Ext, StateExt>
): Store<ExtendState<S, StateExt>, A, StateExt, Ext> & Ext {
  // ...

  let currentReducer = reducer
  let currentState = preloadedState as S
  let currentListeners: (() => void)[] | null = []
  let nextListeners = currentListeners
  let isDispatching = false

  // ...

  dispatch({ type: ActionTypes.INIT } as A)

  const store = {
    dispatch: dispatch as Dispatch<A>,
    subscribe,
    getState,
    replaceReducer,
    [$$observable]: observable
  } as unknown as Store<ExtendState<S, StateExt>, A, StateExt, Ext> & Ext
  return store
}
```

变量名称还是比较好懂的，一共定义了五个局部变量

- `currentReducer` 保存当前的 reducer，为真正处理 action 并返回状态的核心函数
- `currentState` 则是保存的状态本身
- `currentListeners`、`nextListeners` 则是将监听函数分成两个队列，在保留旧的队列的同时可以对新的队列进行预处理
- `isDispatching` 标志则是规范在 redux 处理过程中必不可以同时请求状态，也就保证整个 redux 执行是同步不可中断的原子操作

最后我们看到返回的 store 对象总共给出了四个 api：

- `getState` 获取当前状态
- `subscribe` 订阅状态
- `dispatch` 修改状态
- `replaceReducer` 置换核心函数

### 1.3 预处理

接下来我们真正进入 createStore API 内部

- `/src/createStore.ts`（源码笔记：`/src/createStore.ts/precheck.ts`）

第一步骤当然是先对参数进行预处理，毕竟 js 实际上是不存在重载方法的，所以不同形式的调用必须自己处理输入参数才行

```ts
export default function createStore<
  S,
  A extends Action,
  Ext = {},
  StateExt = never
>(
  reducer: Reducer<S, A>,
  preloadedState?: PreloadedState<S> | StoreEnhancer<Ext, StateExt>,
  enhancer?: StoreEnhancer<Ext, StateExt>
): Store<ExtendState<S, StateExt>, A, StateExt, Ext> & Ext {
  // 传入参数校验
  if (
    (typeof preloadedState === 'function' && typeof enhancer === 'function') ||
    (typeof enhancer === 'function' && typeof arguments[3] === 'function')
  ) {
    throw new Error(
      'It looks like you are passing several store enhancers to ' +
        'createStore(). This is not supported. Instead, compose them ' +
        'together to a single function. See https://redux.js.org/tutorials/fundamentals/part-4-store#creating-a-store-with-enhancers for an example.'
    )
  }

  // createStore(reducer, enhancer)
  if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
    enhancer = preloadedState as StoreEnhancer<Ext, StateExt>
    preloadedState = undefined
  }

  if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
      throw new Error(
        `Expected the enhancer to be a function. Instead, received: '${kindOf(
          enhancer
        )}'`
      )
    }

    return enhancer(createStore)(
      reducer,
      preloadedState as PreloadedState<S>
    ) as Store<ExtendState<S, StateExt>, A, StateExt, Ext> & Ext
  }

  if (typeof reducer !== 'function') {
    throw new Error(
      `Expected the root reducer to be a function. Instead, received: '${kindOf(
        reducer
      )}'`
    )
  }

  // ...
```

这部分还是比较简单的，读者自己看看

值得注意的是其中当存在 enhancer 的时候会直接返回调用的结果，这部分我们就不做探讨，让我们放到解析 applyMiddleware 的时候一起说明

### 1.4 获取状态 getState

接下来我们进入重头戏，返回的 store 对象上对应的 api

- `/src/createStore.ts`（源码笔记：`/src/createStore.ts/getState.ts`）

第一个是获取当前状态的方法，实际上就是一个 getter

```ts
export default function createStore<
  S,
  A extends Action,
  Ext = {},
  StateExt = never
>(
  reducer: Reducer<S, A>,
  preloadedState?: PreloadedState<S> | StoreEnhancer<Ext, StateExt>,
  enhancer?: StoreEnhancer<Ext, StateExt>
): Store<ExtendState<S, StateExt>, A, StateExt, Ext> & Ext {
  // ...

  let currentReducer = reducer
  let currentState = preloadedState as S
  let currentListeners: (() => void)[] | null = []
  let nextListeners = currentListeners
  let isDispatching = false

  /**
   * Reads the state tree managed by the store.
   *
   * @returns The current state tree of your application.
   */
  function getState(): S {
    if (isDispatching) {
      throw new Error(
        'You may not call store.getState() while the reducer is executing. ' +
          'The reducer has already received the state as an argument. ' +
          'Pass it down from the top reducer instead of reading it from the store.'
      )
    }

    return currentState as S
  }

  const store = {
    dispatch: dispatch as Dispatch<A>,
    subscribe,
    getState,
    replaceReducer,
    [$$observable]: observable
  } as unknown as Store<ExtendState<S, StateExt>, A, StateExt, Ext> & Ext
  return store
}
```

### 1.5 订阅状态 subscribe

第二个 api 则是用于订阅 redux 的 store 状态

- `/src/createStore.ts`（源码笔记：`/src/createStore.ts/subscribe.ts`）（篇幅有限省略一些类型检查的警告）

```ts
export default function createStore<
  S,
  A extends Action,
  Ext = {},
  StateExt = never
>(
  reducer: Reducer<S, A>,
  preloadedState?: PreloadedState<S> | StoreEnhancer<Ext, StateExt>,
  enhancer?: StoreEnhancer<Ext, StateExt>
): Store<ExtendState<S, StateExt>, A, StateExt, Ext> & Ext {
  // ...

  let currentReducer = reducer
  let currentState = preloadedState as S
  let currentListeners: (() => void)[] | null = []
  let nextListeners = currentListeners
  let isDispatching = false

  function ensureCanMutateNextListeners() {
    if (nextListeners === currentListeners) {
      nextListeners = currentListeners.slice()
    }
  }

  function subscribe(listener: () => void) {
    let isSubscribed = true

    ensureCanMutateNextListeners()
    nextListeners.push(listener)

    return function unsubscribe() {
      if (!isSubscribed) {
        return
      }

      isSubscribed = false

      ensureCanMutateNextListeners()
      const index = nextListeners.indexOf(listener)
      nextListeners.splice(index, 1)
      currentListeners = null
    }
  }
  
  const store = {
    dispatch: dispatch as Dispatch<A>,
    subscribe,
    getState,
    replaceReducer,
    [$$observable]: observable
  } as unknown as Store<ExtendState<S, StateExt>, A, StateExt, Ext> & Ext
  return store
}
```

在这里我们就能看出监听函数分成两个队列的用意，首先定义一个确保两个队列相互独立的方法

```ts
  function ensureCanMutateNextListeners() {
    if (nextListeners === currentListeners) {
      nextListeners = currentListeners.slice()
    }
  }
```

接下来订阅的时候就是将监听函数放入新队列当中

```ts
  function subscribe(listener: () => void) {
    let isSubscribed = true

    ensureCanMutateNextListeners()
    nextListeners.push(listener)
```

然后最后返回能够解除订阅的方法

```ts
    return function unsubscribe() {
      if (!isSubscribed) {
        return
      }

      isSubscribed = false

      ensureCanMutateNextListeners()
      const index = nextListeners.indexOf(listener)
      nextListeners.splice(index, 1)
      currentListeners = null
    }
```

### 1.6 修改状态 dispatch

接下来就是 Redux 的重头戏，用于修改状态的 dispatch 函数

- `/src/createStore.ts`（源码笔记：`/src/createStore.ts/dispatch.ts`）（一样去除一些不必要的警告和检查）

```ts
export default function createStore<
  S,
  A extends Action,
  Ext = {},
  StateExt = never
>(
  reducer: Reducer<S, A>,
  preloadedState?: PreloadedState<S> | StoreEnhancer<Ext, StateExt>,
  enhancer?: StoreEnhancer<Ext, StateExt>
): Store<ExtendState<S, StateExt>, A, StateExt, Ext> & Ext {
  // ...

  let currentReducer = reducer
  let currentState = preloadedState as S
  let currentListeners: (() => void)[] | null = []
  let nextListeners = currentListeners
  let isDispatching = false

  function dispatch(action: A) {
    try {
      isDispatching = true
      currentState = currentReducer(currentState, action)
    } finally {
      isDispatching = false
    }

    const listeners = (currentListeners = nextListeners)
    for (let i = 0; i < listeners.length; i++) {
      const listener = listeners[i]
      listener()
    }

    return action
  }

  const store = {
    dispatch: dispatch as Dispatch<A>,
    subscribe,
    getState,
    replaceReducer,
    [$$observable]: observable
  } as unknown as Store<ExtendState<S, StateExt>, A, StateExt, Ext> & Ext
  return store
}
```

dispatch 的核心在于将前一次的状态和 action 作为参数传入 reducer 来获取下一次的状态

```ts
    try {
      isDispatching = true
      currentState = currentReducer(currentState, action)
    } finally {
      isDispatching = false
    }
```

接下来更新完毕之后就是调用以下所有的观察者

```ts
    const listeners = (currentListeners = nextListeners)
    for (let i = 0; i < listeners.length; i++) {
      const listener = listeners[i]
      listener()
    }

    return action
```

值得注意的有两个点

1. `(currentListeners = nextListeners)` 使得我们在真正 dispatch 的时候才会去使用新的队列，使得我们在操作 subscribe/unsubscribe 的时候能更直接的操作 nextListeners 而避免对 currentListeners 有过多过于复杂的修改
2. 返回 action 的目的是在于配合中间件的调用模式，保留 action 并继续向下传递

### 1.7 置换核心函数 replaceReducer

最后一个 api 就是用于置换 reducer 的方法

- `/src/createStore.ts`（源码笔记：`/src/createStore.ts/replaceReducer.ts`）

reducer 函数就好像整个 store 对象的命脉，决定了往后所有状态的变动的核心函数

不过 replaceReducer 方法的实现也异常简单

```ts
export default function createStore<
  S,
  A extends Action,
  Ext = {},
  StateExt = never
>(
  reducer: Reducer<S, A>,
  preloadedState?: PreloadedState<S> | StoreEnhancer<Ext, StateExt>,
  enhancer?: StoreEnhancer<Ext, StateExt>
): Store<ExtendState<S, StateExt>, A, StateExt, Ext> & Ext {
  // ...

  let currentReducer = reducer
  let currentState = preloadedState as S
  let currentListeners: (() => void)[] | null = []
  let nextListeners = currentListeners
  let isDispatching = false

  function replaceReducer<NewState, NewActions extends A>(
    nextReducer: Reducer<NewState, NewActions>
  ): Store<ExtendState<NewState, StateExt>, NewActions, StateExt, Ext> & Ext {
    // TODO: do this more elegantly
    ;(currentReducer as unknown as Reducer<NewState, NewActions>) = nextReducer

    dispatch({ type: ActionTypes.REPLACE } as A)
    // change the type of the store by casting it to the new store
    return store as unknown as Store<
      ExtendState<NewState, StateExt>,
      NewActions,
      StateExt,
      Ext
    > &
      Ext
  }

  const store = {
    dispatch: dispatch as Dispatch<A>,
    subscribe,
    getState,
    replaceReducer,
    [$$observable]: observable
  } as unknown as Store<ExtendState<S, StateExt>, A, StateExt, Ext> & Ext
  return store
}
```

其实就是将新的 reducer 保存

```ts
    ;(currentReducer as unknown as Reducer<NewState, NewActions>) = nextReducer
```

接下来再重新调用一个 dispatch 来对新的 reducer 进行初始化

```ts
    dispatch({ type: ActionTypes.REPLACE } as A)
    // change the type of the store by casting it to the new store
    return store as unknown as Store<
      ExtendState<NewState, StateExt>,
      NewActions,
      StateExt,
      Ext
    > &
      Ext
```

# 结语

本篇对于 Redux 的源码解析就先在这里告一段落，鉴于上次解析 Vue2 的时候产生超过 2000 行的长篇大论，我认为应该将整个源码进行适当的拆解之后逐个击破才是正确的攻略方式。

本篇介绍的 createStore 方法，相信大家都了解 store 对象究竟是怎么来的啦，同时也留下了许多谜团

- `reducer` 又有普通 reducer，还可以 combineReducer => 留到 **combineReducers API** 的篇幅再说明
- `dispatch` 的 middleware 模式又是如何实现的呢？这个则留到 **applyMiddleware API** 的部分来讲解
- 还有就是 `action` 的用法，我们常常看到会写成 actionCreator 的形式（这也是官方推荐的写法），这部分则留到 **bindActionCreators API** 的部分来讲解。

希望本篇能帮助大家更容易的理解 redux 源码

# 其他资源

## 参考连接

| Title          | Link                                                                 |
| -------------- | -------------------------------------------------------------------- |
| redux - Github | [https://github.com/reduxjs/redux](https://github.com/reduxjs/redux) |

## 阅读笔记参考

[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/redux-4.1.1](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/redux-4.1.1)
