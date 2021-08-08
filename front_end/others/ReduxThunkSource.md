# Redux 源码解析: 从源码的角度了解 redux-thunk 到底怎么用

@[TOC](文章目录)

<!-- TOC -->

- [Title](#title)
- [前言](#前言)
- [正文](#正文)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

Redux 虽然说概念上是个还算单纯的东西，就是透过 Action 改变状态，而用户自动去监听 State 的状态变化就好。然而实际上在使用的时候 reducer 越来越复杂，同时 actions 的定义也越来越复杂，加上异步 Actions、thunk 等更多 Actions 定义的花样，常常用一用就晕头转向。

本篇带大家从源码的角度来看看到底该怎么用，第二部分则带大家重新认识我们平常到底都在用什么东西

# 正文

本篇分成两个部分：

- 第一部分：主要着重于源码 redux-thunk 源码的解析，同时会加入一点点 redux 中的相关类型和方法说明

- 第二部分：实战代码到底该怎么写？

## 1. 源码解析

### 1.1 （复习）Redux 中间件

首先第一件事我们先来复习一下 Redux 的中间件该怎么写

一个中间件的方法签名如下

```ts
export interface Middleware<
  _DispatchExt = {}, // TODO: remove unused component (breaking change)
  S = any,
  D extends Dispatch = Dispatch
> {
  (api: MiddlewareAPI<D, S>): (
    next: D
  ) => (action: D extends Dispatch<infer A> ? A : never) => any;
}
```

看起来有些复杂，换一个简单版

```ts
const middleware = (store) => (next) => (action) => {};
```

 一个中间件需要经过三次的参数绑定

- `store` 也就是绑定的状态管理对象 store
- `next` 实际上就是原本的 store.dispatch 方法，中间件透过替换 dispatch 方法的方式来为 dispatch 提供更强的处理能力，大概如下（官方称为 Monkeypatching）

```ts
function middleware(store) {
  const next = store.dispatch;

  return function dispatch(action) {
    // do something
    next(action);
  };
}
```

实际上就是让我们得以在 action 进来的时候先进入中间件处理，然后再透过 next 调用原来的 dispatch 方法。

官方给出了好多中间件的[例子](https://redux.js.org/understanding/history-and-design/middleware)，可以上去参考参考

### 1.2 redux-thunk 源码

中间件就先谈到这里，接下来我们来看看 redux-thunk 到底是个什么玩意儿。用过的人隐约知道它使 redux 能够支持异步操作，而且就是通过中间件实现。其实它的源码真的非常简单，看过就知道了

- redux-thunk **全部源码**（真的是全部）

```ts
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) =>
    (next) =>
    (action) => {
      if (typeof action === 'function') {
        return action(dispatch, getState, extraArgument);
      }

      return next(action);
    };
}

const thunk = createThunkMiddleware();
thunk.withExtraArgument = createThunkMiddleware;

export default thunk;
```

redux-thunk 先定义一个中间件的工厂函数 createThunkMiddleware，返回的函数也符合上面提过的中间件方法签名

我们看到实际上就两句话

1. 如果是函数，就调用这个函数，并传入三个参数

```ts
if (typeof action === 'function') {
  return action(dispatch, getState, extraArgument);
}
```

2. 否则直接将 action 传递下去

```ts
return next(action);
```

对就是这么简单，也就是说实际上当我们将中间件如下面一样加入 store 的时候

```js
const store = createStore(rootReducer, applyMiddleware(thunk));
```

我们就可以向 dispatch 方法传入一个函数，其中他接受三个参数：dispatch, getState, extraArgument

```js
const asyncAction = (dispatch, getState, extraArgument) => {
  // do something
};

store.dispatch(asyncAction);
```

也就是说我们既可以在函数的最后自己调用 `dispatch(action)`，也可以选择 return 返回一个 action，由中间件自己传递下去。

### 1.3 Action、ActionCreator 类型定义

其实就单纯上述的几种用法还是比较单纯，无非就是从传入一个 aciton 对象变成传入一个方法罢了。

这时候我们再回头来看看所谓的 action 都是些什么类型

#### 1.3.1 Action 类型（redux 源码）

```ts
export interface Action<T = any> {
  type: T;
}

export interface AnyAction extends Action {
  // Allows any extra properties to be defined in an action.
  [extraProps: string]: any;
}
```

我们可以看到 redux 中定义的 action 很简单，有个 type 属性就算你对了，如果想要加入更多的属性还可以继承 AnyAction 类型来进行扩展（大多时候我们可以约定都叫做 payload 来避免大量的类型定义）

#### 1.3.2 ActionCreator 类型（redux 源码）

第二个也是 redux 官方推荐的写法，可以写一个用于生产 action 对象的工厂方法

```ts
export interface ActionCreator<A, P extends any[] = any[]> {
  (...args: P): A;
}

export interface ActionCreatorsMapObject<A = any, P extends any[] = any[]> {
  [key: string]: ActionCreator<A, P>;
}
```

我们可以看到实际上所谓的 ActionCreator 就是接受任意参数（`...args`），然后返回一个 Action 类型（`A`）的函数

此外我们还可以定义一个拥有多个 ActionCreator 作为键值的对象，用于对应下面的用法

```ts
import * as SomeActions from 'SomeActions.ts';
```

这时候的 SomeActions 就会是一堆 ActionCreator 的集合

### 1.4 bindActionCreators 类型定义 & 源码

有了 ActionCreator 之后，我们可能会常常需要这样使用它

```ts
store.dispatch(someActionCreator(args));
```

当这个 someActionCreator 被调用很多次的时候，每次都要带上 store.dispatch 就显得很麻烦，所以 redux 还提供了所谓的 bindActionCreators 方法来简化我们的操作

#### 1.4.1 bindActionCreator 源码

我们先看基础款的 bindActionCreator，也就是绑定的单一个 ActionCreator 的场景

```ts
function bindActionCreator<A extends AnyAction = AnyAction>(
  actionCreator: ActionCreator<A>,
  dispatch: Dispatch
) {
  return function (this: any, ...args: any[]) {
    return dispatch(actionCreator.apply(this, args));
  };
}
```

P.S. 这里使用 this 作为参数的用法，不懂得可以参考一下 Typescript 手册，还蛮有趣的用法，当初在这里卡了好久

我们使用的时候传入

```ts
const someAction = bindActionCreator(someActionCreator, store.dispatch);
```

之后，我们就可以直接调用

```ts
// no need
// store.dispatch(someActionCreator(args))

// better
const someAction = bindActionCreator(someActionCreator, store.dispatch);
someAction(args);
```

然后前面 bind 过的方法就会自动调用绑定好的 dispatch 函数，然后将 someActionCreator 生产出来的 action 传入

#### 1.4.2 bindActionCreators 源码

有了单一基础版本的 bindActionCreator，我们来看看完整版的实现

```ts
// v1
export default function bindActionCreators<A, C extends ActionCreator<A>>(
  actionCreator: C,
  dispatch: Dispatch
): C;

// v2
export default function bindActionCreators<
  A extends ActionCreator<any>,
  B extends ActionCreator<any>
>(actionCreator: A, dispatch: Dispatch): B;

// v3
export default function bindActionCreators<
  A,
  M extends ActionCreatorsMapObject<A>
>(actionCreators: M, dispatch: Dispatch): M;

// v4
export default function bindActionCreators<
  M extends ActionCreatorsMapObject,
  N extends ActionCreatorsMapObject
>(actionCreators: M, dispatch: Dispatch): N;

// real function
export default function bindActionCreators(
  actionCreators: ActionCreator<any> | ActionCreatorsMapObject,
  dispatch: Dispatch
) {
  if (typeof actionCreators === 'function') {
    return bindActionCreator(actionCreators, dispatch);
  }

  if (typeof actionCreators !== 'object' || actionCreators === null) {
    throw new Error(
      `bindActionCreators expected an object or a function, but instead received: '${kindOf(
        actionCreators
      )}'. ` +
        `Did you write "import ActionCreators from" instead of "import * as ActionCreators from"?`
    );
  }

  const boundActionCreators: ActionCreatorsMapObject = {};
  for (const key in actionCreators) {
    const actionCreator = actionCreators[key];
    if (typeof actionCreator === 'function') {
      boundActionCreators[key] = bindActionCreator(actionCreator, dispatch);
    }
  }
  return boundActionCreators;
}
```

我们看到实际上源码中定义了四种重载方法，能够满足各种 actionCreator 的定义形式，实际上就是区分两种：

1. 如果是函数，就当作 actionCreator 进行绑定后返回
2. 如果是对象，就当作一个 actionCreator 的集合，一个个绑定后返回整个对象

也就是对应下面两种用法

```ts
// 单个 actionCreator
import { someActionCreator } from 'someActions';

const someAction = bindActionCreators(someActionCreator, store.dispatch);
```

```ts
// 多个个 actionCreator 的集合
import * as someActionCreators from 'someActions';

const someActions = bindActionCreators(someActionCreators, store.dispatch);
```

## 2. 重新回到实战

源码都看完了，我们来回想一下实战中我们都是怎么用的

### 2.0 环境准备

一开始我们先准备一下实验用的 store 定义、reducer 等

#### 2.0.1 store

这边直接加上 thunk 中间件，因为我们已经知道 thunk 会根据传入的 action 进行分发：

1. 如果传入的是对象：当成普通的 action 向下传递
2. 如果传入的是函数：用户自定义的详细 action，调用并传入 dispatch, getState 等信息

- `/src/createStore.ts`

```ts
import { applyMiddleware, createStore } from 'redux';
import thunk from 'redux-thunk';
import { timerReducer } from './timer/reducers';

export default () => createStore(timerReducer, applyMiddleware(thunk));
```

#### 2.0.2 timerReducer

这里我们定义一个简单的计时器 reducer

- `/src/timer/reducers.ts`

```ts
import { Action } from 'redux';
import { ThunkAction } from 'redux-thunk';

export interface ITimerState {
  count: number;
}

export enum ETimerActionType {
  INCREMENT = 'INCREMENT',
  RESET = 'RESET',
}

export interface ITimerAction extends Action<ETimerActionType> {}

export const increment: ActionCreator<ITimerAction> = () => ({
  type: ETimerActionType.INCREMENT,
});

export const reset: ActionCreator<ITimerAction> = () => ({
  type: ETimerActionType.RESET,
});

/**
 * 计数器
 * @param state
 * @param action
 * @returns
 */
const timerReducer = (
  state: ITimerState = initTimerState,
  action: ITimerAction
) => {
  switch (action.type) {
    case ETimerActionType.INCREMENT:
      return { count: state.count + 1 };
    case ETimerActionType.RESET:
      return { count: 0 };
    default:
      return state;
  }
};

export { timerReducer };
```

count 属性表示当前计数，有两个可选操作（INCREMENT 递增、RESET 重置）

#### 2.0.3 bindLogStore

最后我们定义一个自动打印当前状态的方法，否则一个一个写用起来真的挺麻烦

- `/src/utils.ts`

```ts
export const bindLogStore =
  (store: Store<ITimerState, ITimerAction>) =>
  (tag: string = '') => {
    const state = store.getState();
    const prefix = tag ? `[${tag}] ` : '';
    console.log(`${prefix}state`, state);
  };
```

### 2.1 基本用法：store API

一开始最基础的用法就是直接使用 store 的 API 来操作，什么都要自己来，自己生成 action、自己调用 store.dispatch

- `/src/tests/test1_basic.ts`

```ts
import createStore from '../createStore';
import { increment, reset } from '../timer/actions';
import { bindLogStore } from '../utils';

const store = createStore();

const logStore = bindLogStore(store);

logStore('init');

store.dispatch(increment());

logStore();

store.dispatch(increment());
store.dispatch(increment());
store.dispatch(increment());

logStore();

store.dispatch(reset());

logStore();
```

输出：

```
>>>>> test1_basic.ts <<<<<
  [init] state { count: 0 }
  state { count: 1 }
  state { count: 4 }
  state { count: 0 }
```

还挺正常的，我们接着看下去

### 2.2 进阶用法：配合 bindActionCreators

- `/src/tests/test2_bind.ts`

接下来我们就可以用上 bindActionCreators 来简化

```ts
import { bindActionCreators } from 'redux';
import createStore from '../createStore';
import * as timerActions from '../timer/actions';
import { bindLogStore } from '../utils';

const store = createStore();

const logStore = bindLogStore(store);

logStore('init');

const { increment, reset } = bindActionCreators(timerActions, store.dispatch);

increment();

logStore();

increment();
increment();
increment();

logStore();

reset();

logStore();
```

用了之后代码看起来简洁多了，输出如下：

```
>>>>> test2_bind.ts <<<<<
  [init] state { count: 0 }
  state { count: 1 }
  state { count: 4 }
  state { count: 0 }
```

与第一个一样，表示运行逻辑是正确的

### 2.3 异步 Action：使用 redux-thunk

最后就是我们本篇的主角，异步 aciton

前面已经加过 redux-thunk 的中间件了，所以我们现在已经可以向 dispatch 传入一个我们自定义的函数

所以首先我们先定义两个新的异步方法

- `/src/timer/actions.ts`

```ts
export const incrementAsync: ActionCreator<ITimerAsyncAction> =
  (delay: number) => (dispatch, getState, args) =>
    new Promise((resolve, reject) => {
      setTimeout(() => {
        dispatch(increment());
        resolve(getState());
      }, delay);
    });

export const resetAsync: ActionCreator<ITimerAsyncAction> =
  (delay: number) => (dispatch, getState, args) =>
    new Promise((resolve, reject) => {
      setTimeout(() => {
        dispatch(reset());
        resolve(getState());
      }, delay);
    });
```

注意这里的结构，实际上这两个新的方法还是一种 actionCreator，差别在于创建出来的新的 "action" 是一个有如下函数签名的方法

```ts
type AsyncAction = (dispatch, getState, extraArguments) => {};
```

还记得上面 redux-thunks 源码告诉我们的，就是传入这三个参数

- `/src/tests/test3_async.ts`

```ts
import { bindActionCreators } from 'redux';
import createStore from '../createStore';
import * as timerActions from '../timer/actions';
import { bindLogStore } from '../utils';

const store = createStore();

const logStore = bindLogStore(store);

logStore('init');

const { incrementAsync, resetAsync } = bindActionCreators(
  timerActions,
  store.dispatch
);

async function task() {
  const DELAY = 1000;
  await incrementAsync(DELAY);

  logStore();

  await incrementAsync(DELAY / 3);
  await incrementAsync(DELAY / 3);
  await incrementAsync(DELAY / 3);

  logStore();

  await resetAsync(DELAY);

  logStore();
}

task();
```

前面我们定义的两个异步方法都把 promise 对象返回，所以最后的用例我们就可以透过外部的 async/await 来实现同步化

输出：

```
>>>>> test3_async.ts <<<<<
  [init] state { count: 0 }

state { count: 1 }
state { count: 4 }
state { count: 0 }
```

# 结语

本篇到这里就结束了，之前被面试官问过有没有看过 redux-thunk 源码，最近终于腾出时间来看看，还真的是很短，不过对于整个 redux 的运行机制有非常大的帮助，供大家参考。

# 其他资源

## 参考连接

| Title                                          | Link                                                                                                                                                                                                               |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Middleware - Redux 官方                        | [https://redux.js.org/understanding/history-and-design/middleware](https://redux.js.org/understanding/history-and-design/middleware)                                                                               |
| redux - Github                                 | [https://github.com/reduxjs/redux](https://github.com/reduxjs/redux)                                                                                                                                               |
| redux-thunk - Github                           | [https://github.com/reduxjs/redux-thunk](https://github.com/reduxjs/redux-thunk)                                                                                                                                   |
| declaring-this-in-a-function - TypeScript 官方 | [https://www.typescriptlang.org/docs/handbook/2/functions.html#declaring-this-in-a-function](https://www.typescriptlang.org/docs/handbook/2/functions.html#declaring-this-in-a-function)                           |
| 具有泛型的 Typescript 箭头函数的语法是什么？   | [https://qastack.cn/programming/32308370/what-is-the-syntax-for-typescript-arrow-functions-with-generics](https://qastack.cn/programming/32308370/what-is-the-syntax-for-typescript-arrow-functions-with-generics) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/others/redux_thunk_source](https://github.com/superfreeeee/Blog-code/tree/main/front_end/others/redux_thunk_source)
