# Zustand 源码解析(Npm library)

@[TOC](文章目录)

<!-- TOC -->

- [Zustand 源码解析(Npm library)](#zustand-源码解析npm-library)
- [前言](#前言)
- [基本信息](#基本信息)
- [入口：使用示例](#入口使用示例)
- [源码解析](#源码解析)
  - [源码主链路](#源码主链路)
  - [核心函数：createStore 构建状态管理中心](#核心函数createstore-构建状态管理中心)
  - [核心函数：Hook 绑定](#核心函数hook-绑定)
- [同场加映：中间件](#同场加映中间件)
- [感想 & 体悟](#感想--体悟)
  - [适用场景](#适用场景)
- [参考链接](#参考链接)

<!-- /TOC -->

# 前言

近两年，React 体系相关的状态管理层出不穷，可能适逢 React 近期版本的重大变化，包括 Hooks、Concurrent Mode 等，推动大家去对 React 的状态管理重新思考，不论是从使用层面的写法、API 简化等方面，亦或是从底层运行原理基于流式的 immutable 状态管理机制又或是使用 Proxy 构建响应式对象两大流派。

最近听到的新状态管理框架如本篇要介绍的 Zustand，还有如 Jotai、Valtio 等，大家有兴趣的也都可以自己去看一看。

与以往的源码解析不一样的是之后可能会比较少留下阅读笔记，看源码的时候更多的是学习思虑以及模型，单纯的记忆和参考代码写法对作者来说已经意义不是太大，直接对着源码仓库看效率更好。

# 基本信息

- version：`v4.1.2`
- 功能：前端状态管理，主要对接但不限于 React 框架

# 入口：使用示例

要了解一个库，最好的办法就是先从 readme 的推荐用法先照着写一遍，然后再慢慢加入自己的逻辑进行灵活运用&改造

- 基础用法

参考官方首页示例

```ts
import create from 'zustand'

const useStore = create(set => ({
  count: 1,
  inc: () => set(state => ({ count: state.count + 1 })),
}))

function Controls() {
  const inc = useStore(state => state.inc)
  return <button onClick={inc}>one up</button>
}

function Counter() {
  const count = useStore(state => state.count)
  return <h1>{count}</h1>  
}
```

我们可以看到形式上非常像简化版的 redux，不同之处在于它似乎是与 Hooks紧密关联的，当然细读能够知道他其实也能够脱离 react 独立使用。另一个最明显的优点就是他不依赖于 Context API，所以不用加 Provider，这我们在后续的源码解析会看到原因。

下面进入正题源码解析

# 源码解析

## 源码主链路

- `/src/react.ts`

首先我们从源码的主入口往下探，前面的 demo 我们可以看到 zustand 将暴露给使用者的 API 简化到只需要一个 create API

![](https://picures.oss-cn-beijing.aliyuncs.com/img/zustand_1_create.png)

通常我们会直接传入 `createState`，所以直接看到 `createImpl` 里面

![](https://picures.oss-cn-beijing.aliyuncs.com/img/zustand_2_createImpl.png)

我们可以看到整个 `createImpl` 相对简单，主要就是 `createStore` 与 `useStore` 两阶段

## 核心函数：createStore 构建状态管理中心

- `/src/vanilla.ts`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/zustand_3_createStore.png)

从 `createStoreImpl` 我们可以看到，zustand 采用了一种闭包的形式，将 state 藏在了函数之间，并将几个方法合并成一个 api 的对象后导出

- `/src/vanilla.ts > createStoreImpl > setState`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/zustand_4_createStoreImpl_setState.png)

来到 `setState` 我们可以看到，除了使用 `Object.is` 做极简的比较之外，其余的都会重新更新 state 并通知所有订阅对象

## 核心函数：Hook 绑定

- `/src/react.ts > useStore`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/zustand_5_useStore.png)

`useStore` 相对简单，就是将 `createImpl` 导出的 api 与 React 使用 `useSyncExternalStoreWithSelector` 进行对接，并返回 `useSelector` 钩子罢了

# 同场加映：中间件

Zustand 的中间件可以说是我看过的库里面最精简的一个，本质上其实根本没有任何规范。我们先看看最经典的 logger 中间件当例子

```ts
// Log every time state is changed
const log = (config) => (set, get, api) =>
  config(
    (...args) => {
      console.log('  applying', args)
      set(...args)
      console.log('  new state', get())
    },
    get,
    api
  )

const useBeeStore = create(
  log((set) => ({
    bees: false,
    setBees: (input) => set({ bees: input }),
  }))
)
```

其实本质上 Zustand 只要求我们去维护这个 `(set, get, api) => state` 的函数签名，作为参数传给 `create` 即可，Zustand 并不会做任何限制

# 感想 & 体悟

看完最近非常火的 Zustand 状态管理库，最大的特色就在于简单，足够简单的 API，加上提供非常精简而透明的机制，保证每个 state 状态的最小化与有效控制框架复杂度，我觉得在想法上是很好的。

不过有一个问题是里面为了做泛型，并没有按照常规的 TS 类型编写方式，而是用了很多类型声明 + 类型断言的组合技，其实就非常仰赖开发者团队去保证类型的正确性，同时也缺少很多推导类型的自动生成，相当于与 TS 体系的融合并没有那么完整与友善。

## 适用场景

由于 Zustand 本身的机制非常简单，同时也将状态与 action 一起塞入整个 state 当中，在性能优化上完全依赖 `useSyncExternalStoreWithSelector` 基于 selector 的重渲染优化。因此可以说 Zustand 更适合个人与一些小型项目，可能比较没办法承载大型业务的能力。

# 参考链接

| Title                   | Link                                                                   |
| ----------------------- | ---------------------------------------------------------------------- |
| Zustand                 | [https://zustand-demo.pmnd.rs/](https://zustand-demo.pmnd.rs/)         |
| pmndrs/zustand - Github | [https://github.com/pmndrs/zustand](https://github.com/pmndrs/zustand) |
