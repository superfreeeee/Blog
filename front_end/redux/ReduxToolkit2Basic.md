# 细说 Redux Toolkit 2 : 基础用法

@[TOC](文章目录)

<!-- TOC -->

- [细说 Redux Toolkit 2 : 基础用法](#细说-redux-toolkit-2--基础用法)
- [完整代码示例](#完整代码示例)
- [概述](#概述)
- [Reducer vs Slice](#reducer-vs-slice)
- [createStore vs configureStore](#createstore-vs-configurestore)
- [redux-thunk vs createAsyncThunk](#redux-thunk-vs-createasyncthunk)
- [结语](#结语)
- [参考连接](#参考连接)

<!-- /TOC -->

# 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_redux_toolkit](https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_redux_toolkit)

# 概述

本片要来正式介绍 Redux Toolkit 的具体使用方法。本篇会先介绍使用 Redux 时候的基础流程，并对比使用原生 Redux 跟 Toolkit 的区别。

# Reducer vs Slice

我们知道 Redux 状态管理的核心在于 Reducer 函数，根据传入的 action 来计算下一个 state 快照

- 原生 Redux

在原生 Redux 里面我们会需要写一个 reducer 函数

```ts
const counterReducer: Reducer<ICounterState, Action<ECounterAction>> = (
  prevState = initialState,
  action
) => {
  switch (action.type) {
    case ECounterAction.Increment:
      return { ...prevState, value: prevState.value + 1 };
    case ECounterAction.Decrement:
      return { ...prevState, value: prevState.value - 1 };
    default:
      return prevState;
  }
};
```

然后我们通常也会为不同 action 定制一个 actionCreator

```ts
const incrementAction = () => ({ type: ECounterAction.Increment });
const decrementAction = () => ({ type: ECounterAction.Decrement });
// ...
```

但是这样会产生几个问题

1. reducer 内部的 switch 容易变成异常臃肿，最后还要再单独抽离成不同的方法
2. 需要开发者自己去维护 action type 与 switch case 的映射和一致性，虽然使用 TS 基本上不太容易出现拼写错误的问题但是非常麻烦
3. 创建 action creator 是非常冗余且重复的工作，降低开发效率

- 使用 Redux Toolkit : `createSlice`

新的 Redux Toolkit 提供了一个 API，帮我们一次解决所有问题，同时还将不同 aciton 的逻辑很好的隔离

```ts
const counterSlice = createSlice({
  name: 'counter',
  initialState,
  reducers: {
    incremented(state) {
      state.value++;
    },
    amountAdded(state, action: PayloadAction<number>) {
      state.value += action.payload;
    },
    reset(state) {
      state.value = 0;
    },
  },
});

export const { incremented, amountAdded } = counterSlice.actions;
export default counterSlice.reducer;
```

我们可以看到，使用 createSlice 之后代码非常简洁有力，不同 action 对应的操作直接对应了 reducers 内部的方法（内部状态修改使用了 immer 库的能力，避免传统用法需要一直使用 `...` 运算符的问题）。

同时 toolkit 会为我们自动生成 reducers 对应的 actionCreator，直接从 `xxxSlice.actions` 里面就能够取得，后续就跟原生 Redux 一样如下进行掉用就可以了，体验非常爽

```ts
dispatch(incremented());
```

# createStore vs configureStore

创建好 reducer 之后，就是要建立状态管理中心 store

- 原生 Redux

原生的 Redux 我们会使用 `createStore` API 如下

```ts
const store = createStore(counterReducer);
```

当我们需要使用多个 reducer 的时候则会变成

```ts
const rootReducer = combineReducers({
  counter: counterReducer,
});
const store = createStore(rootReducer);
```

如果要加上中间件还要自行进行组合

```ts
const composedEnhancer = applyMiddleware(thunkMiddleware);
// combineReducers ...
const store = createStore(rootReducer, composedEnhancer);
```

可以看到代码非常冗长，同时还有机会存在 store 组合错误。在使用 TS 的时候还需要为每个单位单独配置 type 类型信息，非常琐碎。

- Redux Toolkit

Toolkit 这次提供了一个 `configureStore` API 为我们一次解决所有问题，采用配置的编写形式，简化使用者的理解成本，能够更直观的表达 store 的组合逻辑

```ts
export const store = configureStore({
  reducer: {
    counter: counterSlice,
    [apiSlice.reducerPath]: apiSlice.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(apiSlice.middleware).concat(loggerMiddleware),
});
```

所有 reducer 直接写在 reducer 字段下的对象当中，中间件则是使用类似创建者模式的方式衔接成一个链表

甚至关于状态/操作类型，我们可以直接利用 TS 的内置特性获得组合后的 store 导出类型

```ts
export type AppDispatch = typeof store.dispatch;
export type IRootState = ReturnType<typeof store.getState>;
```

我们就不需要再自己维护组合后的总状态类型描述，并能够直接从方法上获取对应的函数签名，非常方便

# redux-thunk vs createAsyncThunk

最后一个段落则是介绍关于 redux-thunk 用法上的改进

- 原生 Redux

在原生 Redux 中使用 redux-thunk，我们需要先向 store 加入中间件

```ts
const composedEnhancer = applyMiddleware(thunkMiddleware);
const store = createStore(rootReducer, composedEnhancer);
```

接下来我们需要写一个 thunk 传入 redux

```ts
export async function fetchTodos(dispatch, getState) {
  const todos = await fetch('/todos.json').then((res) => res.json());
  dispatch({ type: ETodoActionType.TodoLoaded, payload: todos });
}
```

最后往 reducer 中添加一个新的 case 进行处理

- Redux Toolkit

但是在实践中我们会发现，对于异步的 thunk 状态，很多时候我们还需要区分 pending、fulfilled 等不同的状态，就变成针对一个 action 逻辑我们需要写好多次的 dispatch 逻辑以及对应的 case 状态变化如下

```ts
export async function fetchTodos(dispatch, getState) {
  dispatch({ type: ETodoActionType.TodoLoadedPending });
  try {
    const todos = await fetch('/todos.json').then((res) => res.json());
    dispatch({ type: ETodoActionType.TodoLoadedFulfilled, payload: todos });
  } catch (e) {
    dispatch({ type: ETodoActionType.TodoLoadedRejected, payload: e });
  }
}
```

Toolkit 提供了 createAsyncThunk API，为我们自动区分异步操作的不同阶段，我们可以直接根据不同阶段的钩子定义状态变化

首先我们先定义好原始的 thunk 操作

```ts
export const asyncInitCounter = createAsyncThunk<number>(
  'counter/initCounter',
  (params, thunksAPI) =>
    new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve(100);
      }, 1000);
    })
);
```

然后 toolkit 会自动生成对应的 pending、fulfilled action，我们只需要分别添加到 slice 内部的 extraReducers 即可

```ts
const counterSlice = createSlice({
  name: 'counter',
  initialState,
  reducers: {
    // ...
  },
  extraReducers: (builder) => {
    builder
      .addCase(asyncInitCounter.pending, (state, action) => {
        state.hasInit = false;
      })
      .addCase(asyncInitCounter.fulfilled, (state, action) => {
        state.hasInit = true;
        state.value = action.payload;
      });
  },
});
```

我们可以看到整个代码非常的清爽，每一个 case 分成一个个独立的方法，事件名称也由 createAsyncThunk 自动生成

最后掉用方式就跟原来的 thunk 一样，直接 dispatch 提交即可

```ts
dispatch(asyncInitCounter());
```

# 结语

本片介绍并比较原生 Redux 的用法以及 Toolkit 的写法，以及带来的好处，不管是逻辑的梳理更加清晰、重复性劳动更少，以及更好的类型支持，使开发者能更加专注在状态的业务逻辑本身。

下一篇也是最后一篇，将简单介绍 Redux Toolkit 另一个杀手级特性：RTK Query，类似我们以前用过的 React Query、SWR 等，都是关于与服务端进行状态同步管理的解决方案。RTK Query 很好的实现与 Redux 体系结合，同时良好的 API 和类型支持，使得新的 Redux Toolkit 用户能够无缝将服务端状态同步管理与前端全局状态管理进行融合。

# 参考连接

| Title                                                            | Link                                                                                             |
| ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| Usage Guide - Redux Toolkit                                      | [https://redux-toolkit.js.org/usage/usage-guide](https://redux-toolkit.js.org/usage/usage-guide) |
| Let’s Learn Modern Redux! (with Mark Erikson) — Learn With Jason | [https://youtu.be/9zySeP5vH9c](https://youtu.be/9zySeP5vH9c)                                     |
