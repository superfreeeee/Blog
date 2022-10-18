# React 进阶: useSyncExternalStore API 外部状态管理

@[TOC](文章目录)

<!-- TOC -->

- [React 进阶: useSyncExternalStore API 外部状态管理](#react-进阶-usesyncexternalstore-api-外部状态管理)
- [完整代码示例](#完整代码示例)
- [动机 & 关于状态的思考](#动机--关于状态的思考)
- [方案一：自行接入外部状态](#方案一自行接入外部状态)
  - [外部状态接口定义](#外部状态接口定义)
- [方案二：useSyncExternalStore API](#方案二usesyncexternalstore-api)
- [方案三：useSyncExternalStoreWithSelector 进行 Selector 优化](#方案三usesyncexternalstorewithselector-进行-selector-优化)
- [参考连接](#参考连接)

<!-- /TOC -->

# 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_external_store](https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_external_store)

# 动机 & 关于状态的思考

现代 SPA 前端页面当中，面临的的一大挑战就是状态管理。社区里已经能看到大量关于状态管理的结局方案和思路，很多时候这些所谓的状态管理又是与网页所使用的 UI 框架无关的。

如流式思想的 Flux 状态管理模式产生的 Redux 既可以用在 React 也可以用到 Vue 当中；与之对应的义响应式对象为核心的如 MobX、RxJS 等框架一样可以适配到任意 UI 框架之中。当今的框架结构也慢慢的趋同与，管理状态+与UI框架结合的胶水代码的模式。

这时候我们就要回头去思考，到底什么是状态，我们为什么要去管理，最后又如何管理一个状态。本篇主要探讨那些需要持久化，或是至少在页面运行的某段时间内需要持续维护的状态，而不讨论一些瞬时状态的传递与接收。

在 React 开发当中，所有状态都可以分成三类：组件内部 state/props 状态、局部上下文 Context 内部的状态，以及来自于组件外部的独立状态。

本篇要特别针对第三种状态的接入方式进行分享，核心就是一个 useSyncExternalStore 的 React API，以及附带的 withSelector 版本的优化。

# 方案一：自行接入外部状态

第一种最容易想到的就是，在生命周期的挂载(onMount)与卸载(Unmount)进行外部状态的监听。本篇拿自定义 hook 来举例

## 外部状态接口定义

首先我们先定义好一个外部状态接入的通用接口

```ts
interface ExternalStore<T> {
  getState(): T;
  subscribe(listener: () => void): () => void; // return unmount
  dispatch(updater: T | ((prevState: T) => T)): void;
}
```

要有获取状态(不论是静态快照或是响应式动态对象都好)的`getState`方法；第二个是监听状态变化的`subscribe`方法；最后是更新状态的`dispatch`方法。

有些状态管理库会透过其他形式来提供这些接口，如响应式对象直接使用 `getter/setter` 作为状态的 input/output 就行了。

本篇基于上述接口的外部状态，我们就可以写出如下的自定义 Hook 来接入一个外部状态

```ts
export const useCustomStore = <T>(store: ExternalStore<T>) => {
  const [state, setState] = useState(store.getState());

  useEffect(() => {
    const unsubscribe = store.subscribe(() => {
      setState(store.getState());
    });

    return () => { unsubscribe(); };
  }, []);

  return state;
};
```

利用 useEffect 的生命周期特性来完成状态变化事件的监听，并使用 `setState` 方法来更新并引起组件的重渲染。

# 方案二：useSyncExternalStore API

上述方法虽然基础而简单，但是在实际项目中还是非常实用的。

接下来向读者介绍第二种也就是 React 官方提供的 API

该 API 的接口如下

```ts
export function useSyncExternalStore<Snapshot>(
  subscribe: (onStoreChange: () => void) => () => void,
  getSnapshot: () => Snapshot,
  getServerSnapshot?: () => Snapshot,
): Snapshot;
```

我们可以发现与我们定义的外部状态接口非常一致，几乎可以无缝接入，而这也是当今多数开源库采用的接入状态定义

```ts
export const useExternalStore = <T>(store: ExternalStore<T>) => {
  return useSyncExternalStore(
    store.subscribe,
    store.getState
  );
};
```

使用 `useSyncExternalStore` API 与自定义 Hook 最大的不同在于，使用 API 我们就是将状态更新如何触发重渲染的逻辑与时机交给 React 来负责。这样业务层面所提供的外部状态，在接入新的 React 渲染逻辑(如 18 之后出现的 Concurrent 模式)，从而能够获得更多的框架内部优化机会，不像自定义 Hook 一定只能基于 Effect 执行逻辑从而存在优化上限

# 方案三：useSyncExternalStoreWithSelector 进行 Selector 优化

除了基础的 `useSyncExternalStore` 之外，React 还额外提供了带 Selector 优化的 `useSyncExternalStoreWithSelector`。

什么叫 "带 Selector 优化" 呢？从第一个方案我们可以看到对于外部状态的变化感知力度是相对较粗的，例如我们的状态类型是

```ts
type State = {
  name: string;
  grade: number;
}
```

那么除非我们特别定制一个选择性监听变化的逻辑，否则我们只能在触发响应的时候基于返回的状态快照进行记忆和优化如下

```ts
let prevState; // ...

store.subscribe(() => {
  const newState = store.getState();
  if (!isEqual(prevState, newState)) {
    setState(newState);
  }
})
```

但是一方面这样的定制因状态的不同而异，要想要得到最大程度的优化就需要进行非常多的额外工作。

这时候 React 提供基于 Selector 的优化范式，声明了一种 `state => Selection` 的选择器函数，然后在 React 内部机制内针对选择器返回的取值进行优化

这个特制的 `useSyncExternalStoreWithSelector` API 放在了 `use-sync-external-store` 包之中，首先是类型定义

```ts
export function useSyncExternalStoreWithSelector<Snapshot, Selection>(
    subscribe: (onStoreChange: () => void) => () => void,
    getSnapshot: () => Snapshot,
    getServerSnapshot: undefined | null | (() => Snapshot),
    selector: (snapshot: Snapshot) => Selection,
    isEqual?: (a: Selection, b: Selection) => boolean,
): Selection;
```

我们可以看到相较于原来的 `useSyncExternalStore` 多了两个参数，一个是选择返回指定状态的 seletor 函数，以及决定是否更新的 isEqual 函数

这样一来我们就可以定制出针对细粒度状态的监听与更新优化钩子

```ts
export const createCustomStoreSelector =
  <T, S>(store: ExternalStore<T>) =>
  (selector: (snapshot: T) => S) => {
    return useSyncExternalStoreWithSelector(
      store.subscribe,
      store.getState,
      store.getState,
      selector
    );
  };
```

实际去扒如 react-redux、zustand 的源码的时候，就很容易看到类似上述形式的代码，也是帮助大家学习如何构建一个自己的状态管理库。

# 参考连接

| Title                                       | Link                                                                                                                                     |
| ------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| Library Hooks: useSyncExternalStore - React | [https://reactjs.org/docs/hooks-reference.html#usesyncexternalstore](https://reactjs.org/docs/hooks-reference.html#usesyncexternalstore) |
| pmndrs/zustand - Github                     | [https://github.com/pmndrs/zustand/blob/main/src/react.ts](https://github.com/pmndrs/zustand/blob/main/src/react.ts)                     |
