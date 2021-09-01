# Redux 源码解析(二): bindActionCreators & combineReducers API

@[TOC](文章目录)

<!-- TOC -->

- [Redux 源码解析(二): bindActionCreators & combineReducers API](#redux-源码解析二-bindactioncreators--combinereducers-api)
- [前言](#前言)
- [正文](#正文)
  - [1. bindActionCreators 源码解析](#1-bindactioncreators-源码解析)
    - [1.1 类型定义](#11-类型定义)
    - [1.2 导出方法签名](#12-导出方法签名)
    - [1.3 源码实现](#13-源码实现)
  - [2. combineReducers 源码解析](#2-combinereducers-源码解析)
    - [2.1 使用背景](#21-使用背景)
    - [2.2 类型定义](#22-类型定义)
    - [2.3 导出方法签名](#23-导出方法签名)
    - [2.4 源码实现](#24-源码实现)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 前言

今天是 Redux 源码解析的第二篇，本篇要介绍两个 API：bindActionCreators、combineReducers

# 正文

## 1. bindActionCreators 源码解析

用过 Redux 的应该知道，所谓的 bindActionCreators 就是为使用的 ActionCreator 自动绑定 dispatch 方法，使我们调用的时候会自动调用 dispatch，而不需要写成 `dispatch(createAction())` 的麻烦形式

### 1.1 类型定义

首先我们先来看到与 bindActionCreators 相关的类型定义

- `/src/types/actions.ts`（阅读笔记：`/src/types/actions.ts`）

首先是 Action 相关的，我们知道通常我们使用的时候如下

```ts
// 使用示例
dispatch({ type: 'xxxType', payload: {} })
```

实际上官方类型定义里面只写明要有 type，而并没有规定一定用 payload，只是定义成可索引属性如下

```ts
export interface Action<T = any> {
  type: T
}

export interface AnyAction extends Action {
  // Allows any extra properties to be defined in an action.
  [extraProps: string]: any
}
```

同时 Redux 推荐我们使用一种所谓的 ActionCreator，平常使用的时候我们是这样写的

```ts
// 使用示例
const doSomething = (params) => ({
  type: 'someAction',
  payload: {}
})
```

很棒的是 Redux 也提供了相关的类型定义

```ts
export interface ActionCreator<A, P extends any[] = any[]> {
  (...args: P): A
}

export interface ActionCreatorsMapObject<A = any, P extends any[] = any[]> {
  [key: string]: ActionCreator<A, P>
}
```

第一种是单个 ActionCreator、第二种 ActionCreatorsMapObject 则是一个关于 ActionCreator 的映射表

### 1.2 导出方法签名

看完几个基本类型定义之后，我们下面来看看 bindActionCreators 有哪些导出的方法签名

- `/src/bindActionCreators.ts`（阅读笔记：`/src/bindActionCreators.ts/exports.ts`）

```ts
import { Dispatch } from './types/store'
import {
  AnyAction,
  ActionCreator,
  ActionCreatorsMapObject
} from './types/actions'
import { kindOf } from './utils/kindOf'

export default function bindActionCreators<A, C extends ActionCreator<A>>(
  actionCreator: C,
  dispatch: Dispatch
): C;

export default function bindActionCreators<
  A extends ActionCreator<any>,
  B extends ActionCreator<any>
>(actionCreator: A, dispatch: Dispatch): B;

export default function bindActionCreators<
  A,
  M extends ActionCreatorsMapObject<A>
>(actionCreators: M, dispatch: Dispatch): M;

export default function bindActionCreators<
  M extends ActionCreatorsMapObject,
  N extends ActionCreatorsMapObject
>(actionCreators: M, dispatch: Dispatch): N;
```

我们看到一共重载了四种方法签名，但是实际上参数就是分成 `(actionCreator, dispatch)` 和 `(actionCreators, dispatch)` 两种，其他只是 TS 类型上细微的差异

下面我们就来看看实现

### 1.3 源码实现

- `/src/bindActionCreators.ts`（阅读笔记：`/src/bindActionCreators.ts/implements.ts`）

对于 bindActionCreators 的实现，其实要分成两种

第一种是实现单个 actionCreator 的绑定

```ts
function bindActionCreator<A extends AnyAction = AnyAction>(
  actionCreator: ActionCreator<A>,
  dispatch: Dispatch
) {
  return function (this: any, ...args: any[]) {
    return dispatch(actionCreator.apply(this, args))
  }
}
```

这里的实现就是为 actionCreator 套上一层壳，从 `actionCreator()` 变成 `dispatch(actionCreator())`

第二种则是 bindActionCreators 的完整实现，需要分别处理单个 actionCreator 和多个 actionCreator 两种场景

首先如果第一个参数是 function，则返回 bindActionCreator 的结果

```ts
export default function bindActionCreators(
  actionCreators: ActionCreator<any> | ActionCreatorsMapObject,
  dispatch: Dispatch
) {
  // bindActionCreators(fn, dispatch)
  if (typeof actionCreators === 'function') {
    return bindActionCreator(actionCreators, dispatch)
  }
```

否则则视为 ActionCreatorsMapObject 类型，将所有 key 对应的 actionCreator 分别绑定后返回

```ts
  // bindActionCreators(fnMap, dispatch)
  const boundActionCreators: ActionCreatorsMapObject = {}
  for (const key in actionCreators) {
    const actionCreator = actionCreators[key]
    if (typeof actionCreator === 'function') {
      boundActionCreators[key] = bindActionCreator(actionCreator, dispatch)
    }
  }
  return boundActionCreators
}
```

也就是说我们使用的时候可以如下两种用法

```ts
// 单个 creator
bindActionCreators(creator, dispatch)

// 多个 creator
bindActionCreators({ creatorA, creatorB, creatorC }, dispatch)
```

以上对于 bindActionCreators 的源码解析

## 2. combineReducers 源码解析

我们要介绍的第二个 API 则是 combineReducers

### 2.1 使用背景

一般情况下，我们创建 store 的时候可以创建一个 reducer，如下所示

```ts
const reducer = (state, action) => {}

const store = createStore(reducer)
```

然而当我们的 state 的数据结构足够复杂，同时我们希望区分为不同的模块的时候，我们则可以使用 combineReducers 来粘合多个 reducer

```ts
const rootReducer = combineReducers({ reducerA, reducerB, reducerC })

const store = createStore(rootReducer)
```

### 2.2 类型定义

稍微理解怎么用之后，我们就可以来看看 Redux 中定义的一些基础类型

- `/src/types/reducers.ts`（阅读笔记：`/src/types/reducers.ts`）

首先是基础的单个 reducer 的类型定义

```ts
export type Reducer<S = any, A extends Action = AnyAction> = (
  state: S | undefined,
  action: A
) => S
```

签名是接受状态 S 和操作 A 类型之后，返回新的状态 S

第二个则是我们刚刚提到的使用 combineReducers 时传入的参数是一个 reducer 的映射表，key 表示了每个 reducer 对应的 store.getState() 下的模块名称

```ts
export type ReducersMapObject<S = any, A extends Action = AnyAction> = {
  [K in keyof S]: Reducer<S[K], A>
}
```

如上，所谓的 ReducersMapObject 的 key 是大状态 S 的某个键 K，然后对应的 Reducer 类型为状态 S\[K\]

接下来有几个衍生类型，平常是比较少遇到的

```ts
export type StateFromReducersMapObject<M> = M extends ReducersMapObject
  ? { [P in keyof M]: M[P] extends Reducer<infer S, any> ? S : never }
  : never

export type ReducerFromReducersMapObject<M> = M extends {
  [P in keyof M]: infer R
}
  ? R extends Reducer<any, any>
    ? R
    : never
  : never
```

StateFromReducersMapObject 表示的是 ReducersMapObject 所对应的 State 类型

有关于 State 类型的定义，当然也有 Aciton 的定义咯

```ts
export type ActionFromReducer<R> = R extends Reducer<any, infer A> ? A : never

export type ActionFromReducersMapObject<M> = M extends ReducersMapObject
  ? ActionFromReducer<ReducerFromReducersMapObject<M>>
  : never
```

### 2.3 导出方法签名

看完基础类型定义之后，combineReducers 的方法签名也呼之欲出了

- `/src/combineReducers.ts`（阅读笔记：`/src/combineReducers.ts/exports.ts`）

```ts
import { AnyAction, Action } from './types/actions'
import {
  ActionFromReducersMapObject,
  Reducer,
  ReducersMapObject,
  StateFromReducersMapObject
} from './types/reducers'
import { CombinedState } from './types/store'

import ActionTypes from './utils/actionTypes'
import isPlainObject from './utils/isPlainObject'
import warning from './utils/warning'
import { kindOf } from './utils/kindOf'

export default function combineReducers<S>(
  reducers: ReducersMapObject<S, any>
): Reducer<CombinedState<S>>;

export default function combineReducers<S, A extends Action = AnyAction>(
  reducers: ReducersMapObject<S, A>
): Reducer<CombinedState<S>, A>;

export default function combineReducers<M extends ReducersMapObject>(
  reducers: M
): Reducer<
  CombinedState<StateFromReducersMapObject<M>>,
  ActionFromReducersMapObject<M>
>;
```

一共有三种方法签名，不过差别都是在于 `状态 S`、`操作 A` 等的类型定义，实际上三个重载的参数都一样，都是传入一个 reducers 对象

所以接下来我们就来看看 combineReducers 实现

### 2.4 源码实现

- `/src/combineReducers.ts`（阅读笔记：`/src/combineReducers.ts/implements.ts`）

```ts
export default function combineReducers(reducers: ReducersMapObject) {
  const reducerKeys = Object.keys(reducers)
  const finalReducers: ReducersMapObject = {}
  for (let i = 0; i < reducerKeys.length; i++) {
    const key = reducerKeys[i]

    if (typeof reducers[key] === 'function') {
      finalReducers[key] = reducers[key]
    }
  }
  const finalReducerKeys = Object.keys(finalReducers)
```

在实现方法中的前半，我们可以看到一开始的工作是先做出一个确保属性都是 function 的 finalReducers 以及其对应的键值 finalReducerKeys

第二步则是对 reducers 进行校验，也就是试着传入几个特定的 Action 来测试返回的值

```ts
export default function combineReducers(reducers: ReducersMapObject) {
  // ...

  let shapeAssertionError: Error
  try {
    assertReducerShape(finalReducers)
  } catch (e) {
    shapeAssertionError = e
  }
```

```ts
function assertReducerShape(reducers: ReducersMapObject) {
  Object.keys(reducers).forEach(key => {
    const reducer = reducers[key]
    const initialState = reducer(undefined, { type: ActionTypes.INIT })

    if (typeof initialState === 'undefined') {
      // throw error
    }

    if (
      typeof reducer(undefined, {
        type: ActionTypes.PROBE_UNKNOWN_ACTION()
      }) === 'undefined'
    ) {
      // throw error
    }
  })
}
```

也就是说 assertReducerShape 的作用就是对每个 reducer 进行初始化，然后检查返回的 State 是否符合预期

combineReducers 的函数主体到这里就完成了，最后返回一个合并后的 reducer，当然也必须符合 Reducer 的类型定义

```ts
export default function combineReducers(reducers: ReducersMapObject) {
  // ...
  
  return function combination(
    state: StateFromReducersMapObject<typeof reducers> = {},
    action: AnyAction
  ) {
    if (shapeAssertionError) {
      throw shapeAssertionError
    }

    if (process.env.NODE_ENV !== 'production') {
      const warningMessage = getUnexpectedStateShapeWarningMessage(
        state,
        finalReducers,
        action,
        unexpectedKeyCache
      )
      if (warningMessage) {
        warning(warningMessage)
      }
    }
```

函数的开头一样也是一些异常检查和报错提示

然后我们看到核心的部分

```ts
export default function combineReducers(reducers: ReducersMapObject) {
  // ...

  return function combination(
    state: StateFromReducersMapObject<typeof reducers> = {},
    action: AnyAction
  ) {
    // ...
    let hasChanged = false
    const nextState: StateFromReducersMapObject<typeof reducers> = {}
    for (let i = 0; i < finalReducerKeys.length; i++) {
      const key = finalReducerKeys[i]
      const reducer = finalReducers[key]
      const previousStateForKey = state[key]
      const nextStateForKey = reducer(previousStateForKey, action)
      if (typeof nextStateForKey === 'undefined') {
        // throw error
      }
      nextState[key] = nextStateForKey
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }
    hasChanged =
      hasChanged || finalReducerKeys.length !== Object.keys(state).length
    return hasChanged ? nextState : state
  }
}
```

combine 之后的 reducer 被执行的时候实际上就是做这样的一个工作：

遍历一边 finalReducers

```ts
    for (let i = 0; i < finalReducerKeys.length; i++) {
      const key = finalReducerKeys[i]
      const reducer = finalReducers[key]
```

然后分别获取执行前后的返回值

```ts
      const previousStateForKey = state[key]
      const nextStateForKey = reducer(previousStateForKey, action)
```

最后将结果放到新的对象上，并记录状态是否变化

```ts
      nextState[key] = nextStateForKey
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
```

最后返回之前检查是否所有 reducer 都没有变化决定直接返回旧的值

```ts
    hasChanged =
      hasChanged || finalReducerKeys.length !== Object.keys(state).length
    return hasChanged ? nextState : state
```

如此一来 combineReducers 的工作就完成了，对外部来说 combineReducers 返回的 reducer 与一般的 reducer 不同的是他返回的 state 状态多了一层罢了，而这个层级关系还是与 reducer 被合并的时候的结构相关的，也就是说当我使用如下的 reducer 的时候

```ts
const rootReducer = combineReducer({
  reducerA,
  reducerB: combineReducer({
    reducerB1,
    reducerB2,
  }),
  reducerC,
})
```

我就可以预期返回的 state 应该存在如下结构

```ts
{
  reducerA: {},
  reducerB: {
    reducerB1: {},
    reducerB2: {},
  },
  reducerC: {},
}
```

# 结语

本篇作为 Redux 源码解析的第二部，介绍了 bindActionCreators 和 combineReducers 两个 API，也是使用 Redux 时使用频率非常高的两个 API 供大家参考。

下一篇将为大家带来 Redux 源码解析的最后一篇，介绍 compose 和 applyMiddleware 两个 API，并稍微说明一下 Redux 所定义的中间件模式与应用模式

# 其他资源

## 参考连接

| Title                      | Link                                                                                       |
| -------------------------- | ------------------------------------------------------------------------------------------ |
| bindActionCreators - Redux | [https://redux.js.org/api/bindactioncreators](https://redux.js.org/api/bindactioncreators) |
| combinereducers - Redux    | [https://redux.js.org/api/combinereducers](https://redux.js.org/api/combinereducers)       |
| Redux - Github             | [https://github.com/reduxjs/redux](https://github.com/reduxjs/redux)                       |

## 阅读笔记参考

[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/redux-4.1.1](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/redux-4.1.1)
