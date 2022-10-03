# 细说 Redux Toolkit 1 : 重新回顾 Redux

@[TOC](文章目录)

<!-- TOC -->

- [细说 Redux Toolkit 1 : 重新回顾 Redux](#细说-redux-toolkit-1--重新回顾-redux)
- [前言](#前言)
- [Redux 数据流 & 核心概念](#redux-数据流--核心概念)
- [实现核心示例](#实现核心示例)
  - [State、Reducer](#statereducer)
  - [Store](#store)
  - [Action、ActionCreator](#actionactioncreator)
  - [Middleware、Thunk](#middlewarethunk)
- [Redux Toolkit 概述](#redux-toolkit-概述)
- [参考连接](#参考连接)

<!-- /TOC -->

# 前言

接下来几篇我们要来说说关于 redux 官方提供的新工具 Redux Toolkit。开篇第一篇我们先来重新梳理一下 Redux 的基本概念。

# Redux 数据流 & 核心概念

本篇默认读者已经对于 Redux 有一些基本的认识，如果是零基础也建议读者先去看看 Redux 官方的介绍

- [Redux Essentials, Part 1: Redux Overview and Concepts](https://redux.js.org/tutorials/essentials/part-1-overview-concepts)

接下来我们从 Redux 官方给出的数据流图来重新梳理 Redux 状态管理的流程

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_redux_toolkit_1_overview_1_dataflow.png)

- 全局状态

整个 Redux 的全局数据都存储在 state 对象里面，然后通常我们会透过 `Provider` 的方式基于 Context API 注入到 React 组件当中；当然我们也可以直接使用 store API 的 `getState`、`subscribe`、`dispatch` 自行订阅并管理状态的变化。

- 处理变更

在 view 层基于状态进行展示之后，不论是透过用户操作还是其他外部事件，我们在控制层需要透过所谓的 dispatch 方法提交一个 Action 对象，其中 Action 通常使用 `{ type, payload }` 的数据结构包含操作的类型与参数。我们也可以透过利用中间件来识别更多不同数据结构的 Action 对象（例如 redux-thunk 接受的 Action 实际上是一个 function）

- 中间件

Redux 所提出的中间件模式大大地提高了 Redux 架构的灵活性，我们可以利用中间件进行如过滤、监控、打点、校验，甚至是对于 Action 结构的扩展。经过中间件链的处理才将最终返回的 Action 对象提交到 store 中的 Reducer 重新计算下一次的 state 状态。

到此 Redux 的单向数据流结构就完成闭环，下面我们分别看到每个核心概念的简单实现示例。

# 实现核心示例

## State、Reducer

Redux 当中的全局状态实际上是 Redux 自身在管理的，我们需要做的只是定义好 State 的数据结构，然后透过实现 Reducer 来明确每一次更新时该如何计算下一个 State 状态。

```ts
interface CounterState {
  value: number;
}

const initialState: CounterState = {
  value: 0,
};

function counterReducer(state: CounterState = initialState, action): CounterState {
  // counting next state base on action
  return newState;
}
```

在每次使用 dispatch 提交变更的时候都会触发 reducer 的调用，并计算出下一个 state 的快照。

## Store

建立好 reducer 之后就可以创建完整的状态管理中心 store

```ts
const store = createStore(counterReducer);
```

如果我们有多个 reducer 的时候可以使用 combineReducers 进行合并

```ts
const rootReducer = combineReducers({
  counter: counterReducer,
  meta: metaReducer,
})
const store = createStore(rootReducer);
```

注意当我们使用多个 reducer 的时候，每个 reducer 返回的状态就会分别对应 state 下不同字段，例如 `state.counter.value`、`state.meta.xxx` 等。

- 消费 store

第一种 store 的使用方式就是直接掉用 store 的 API

```ts
store.getState()
store.subscribe(cb)
store.dispatch(action)
```

第二种就是我们在 React 里面最常用的基于 Context API 的 react-redux 库

```ts
function App() {
  return (
    <Provider store={store}>
      <Body/>
    </Provider>
  )
}
```

组件内部可以使用 react-redux 提供的 `connect` HOC 组件将状态映射到 props 之中；或是使用 `<Consumer>` 配合 render children 的方式实现

```ts
function Body() {/*...*/}
export default connect(mapStateToProps, mapDispatchToProps)(Body)
```

```ts
function App() {
  return (
    <Provider store={store}>
      <Consumer>
        {(props) => <Body/>}
      </Consumer>
    </Provider>
  )
}
```

## Action、ActionCreator

前面在数据流的部分我们提过，要修改 Redux 的状态只能透过 dispatch 提交变更，并在 action 内附带相关信息。

```ts
store.dispatch({ type: 'counter/increment' });
store.dispatch({ type: 'counter/setNumber', payload: 10 });
```

标准情况下我们会传递一个带 type 字段的简单对象来表达操作类型。其他情况我们也可以配合中间件的扩展来定义新的变更信息数据结构。下面的例子是使用 redux-thunk 中间件来实现异步状态修改的示例

```ts
store.dispatch((dispatch, getState) => {
  queryAPI().then((res) => {
    dispatch({ type: 'counter/init', payload: res.data.count });
  })
});
```

下面我们在详细说明中间件是如何运作的。

- Action Creator

我们可以看到每次提交变更总要写出完整的 action 对象非常麻烦，所以在大多数的实践中我们会定义一个 actionCreator，将 action 的生成转换成函数的掉用如下

```ts
const incrementAction = () => ({ type: 'counter/increment' });

store.dispatch(incrementAction())
```

## Middleware、Thunk

在基础版本的数据流中，dispatch 的 action 对象会直接传递给 reducer 进行解析并计算下一个 state。然而在传递给 reducer 之前我们可以透过添加中间件来进行额外的操作。基础的中间件函数标签如下

```ts
const middlerware = store => next => action => {
  // logic
  next();
}
```

中间件会像链表一样将多个中间件处理过程串联起来，然后使用 `next` 方法来将 action 传递给下一个中间件。最经典的例子就是日志中间件

```ts
const loggerMiddleware = store => next => action => {
  logger.log('dispatch start', action);
  const res = next(action);
  logger.log('dispatch end', store.getState());
  return res;
};
```

我们需要注意的是 next 返回的是上一个中间件的返回产物，通常但不一定就是 reducer 的 state 对象，所以我们如果不做任何处理于修改的话直接返回上一个中间件的结果保证数据正确透传即可。

# Redux Toolkit 概述

我们回顾完 Redux 的数据流与所有核心概念的代码使用案例，不难发现 Redux 的实践过程其实是相当冗长的。

构建一个基于 Redux 的全局状态管理，我们需要经历以下步骤：
1. 定义全局状态数据结构
2. 定义 Reducer 如何计算下一次 state 的逻辑
   1. 多 Reducer 需要使用 CombineReducers
3. 使用 applyMiddleware 或是其他 enhancers 构建中间件链表
4. 最后基于 reducers 和所有 enhancers 创建 store 对象
5. 在 React 组件中使用 Provider 与 connect HOC 将全局状态注入。

每次我们想要新增新的 action 操作类型的时候：
1. 定义新的 action type，并明确 payload 数据结构
2. 往 Reducer 中添加新的 type 判断分支，以及新的状态计算逻辑
3. 创建新的 actionCreator 加速 dispatch 掉用过程

以上步骤反复执行的时候不免感到异常乏味而且重复，同时 type、actionCreator、reducers 之间又会存在隐式的字面量关联，很多时候需要开发者非常小心，否则就会产生变更提交无效的情形。

Redux Toolkit 的出现就是提供一些对于 Redux API 的封装和语法糖，帮助使用者从重复劳动中解放，能够更好的专注在业务逻辑以及状态转换逻辑本身。

接下来几篇我们会将几个 Redux Toolkit 提供的经典 API 与旧有的 Redux 用法作比较，不管是从旧版本 Redux 迁移过来的用户，或是全新的 Redux 使用者来说，能够更加细致的理解每一个封装背后的构想。

# 参考连接

| Title                                                 | Link                                                                                                                                     |
| ----------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| Redux Essentials, Part 1: Redux Overview and Concepts | [https://redux.js.org/tutorials/essentials/part-1-overview-concepts](https://redux.js.org/tutorials/essentials/part-1-overview-concepts) |
| Redux Toolkit                                         | [https://redux-toolkit.js.org/](https://redux-toolkit.js.org/)                                                                           |
