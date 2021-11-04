# react-intersection-observer 源码解析二连发

@[TOC](文章目录)

<!-- TOC -->

- [react-intersection-observer 源码解析二连发](#react-intersection-observer-源码解析二连发)
- [正文](#正文)
  - [1. IntersectionObserver API](#1-intersectionobserver-api)
  - [2. @researchgate/react-intersection-observer@1.3.5](#2-researchgatereact-intersection-observer135)
    - [2.1 类型定义 types](#21-类型定义-types)
    - [2.2 观察者封装 observer](#22-观察者封装-observer)
      - [2.2.1 模块结构](#221-模块结构)
      - [2.2.2 观察者实例缓存/记录](#222-观察者实例缓存记录)
      - [2.2.3 查找观察目标](#223-查找观察目标)
      - [2.2.4 创建观察者](#224-创建观察者)
      - [2.2.5 观察/取消观察方法](#225-观察取消观察方法)
    - [2.3 hook 版本 - useIntersectionObserver](#23-hook-版本---useintersectionobserver)
      - [2.3.1 状态定义](#231-状态定义)
      - [2.3.2 取消观察 & 回调函数包装](#232-取消观察--回调函数包装)
      - [2.3.3 观察方法](#233-观察方法)
      - [2.3.4 Observer 实例](#234-observer-实例)
      - [2.3.5 观察目标 ref](#235-观察目标-ref)
    - [2.4 class 组件版本 - IntersectionObserver](#24-class-组件版本---intersectionobserver)
      - [2.4.0 组件结构](#240-组件结构)
      - [2.4.1 生命周期相关](#241-生命周期相关)
      - [2.4.2 回调函数包装](#242-回调函数包装)
      - [2.4.3 callback ref 方法](#243-callback-ref-方法)
      - [2.4.4 观察/取消观察方法](#244-观察取消观察方法)
  - [3. react-intersection-observer@8.26.1（官方推荐）](#3-react-intersection-observer8261官方推荐)
    - [3.1 类型定义 index](#31-类型定义-index)
    - [3.2 观察者封装 observe](#32-观察者封装-observe)
      - [3.2.0 模块结构](#320-模块结构)
      - [3.2.1 root 参数缓存 key](#321-root-参数缓存-key)
      - [3.2.2 options 参数缓存 key](#322-options-参数缓存-key)
      - [3.2.3 Observer 实例创建](#323-observer-实例创建)
      - [3.2.4 observe 观察目标](#324-observe-观察目标)
    - [3.3 hook 版本 - useInView](#33-hook-版本---useinview)
      - [3.3.1 状态保存](#331-状态保存)
      - [3.3.2 callback ref](#332-callback-ref)
      - [3.3.3 状态清理 & 返回结果](#333-状态清理--返回结果)
    - [3.4 class 组件版本 - InView](#34-class-组件版本---inview)
      - [3.4.0 组件结构](#340-组件结构)
      - [3.4.1 生命周期相关](#341-生命周期相关)
      - [3.4.2 callback ref](#342-callback-ref)
      - [3.4.3 观察方法](#343-观察方法)
      - [3.4.4 回调函数包装](#344-回调函数包装)
      - [3.4.5 取消监听](#345-取消监听)
  - [4. 总结](#4-总结)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 正文

## 1. IntersectionObserver API

本篇要介绍的两个库都围绕着一个 WebAPI - IntersectionObserver，这个 API 本质上能实现的是检测目标元素是否出现在视图当中

- 基础用法 & 对象类型定义

请自行查阅 MDN 文档，这里就不当搬运工了：[Intersection Observer - MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver)

下面我们先从非官方推荐的库开始看起

## 2. @researchgate/react-intersection-observer@1.3.5

- 目录结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_intersection_observer_library_1_researchgate_modules.png)

- 阅读笔记：[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/%40researchgate__react-intersection-observer-1.3.5](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/%40researchgate__react-intersection-observer-1.3.5)

### 2.1 类型定义 types

首先先来看类型定义

- `/src/types.ts`（阅读笔记：`/src/types.ts`）

```ts
// 取消观察方法
export type Unobserve = () => void;

// 回调函数
export type ChangeHandler = (entry: IntersectionObserverEntry, unobserve: Unobserve) => void;

// 目标元素
export type TargetNode = Element;

// 配置选项
export interface Options {
  root?: string | Element | null;
  rootMargin?: string;
  threshold?: number | number[];
  disabled?: boolean;
}

/**
 * 观察目标实例
 *   handleChange 响应回调
 *   observer IntersectionObserver实例
 *   target 观察目标
 */
export interface Instance {
  handleChange: (event: IntersectionObserverEntry) => void;
  observer?: IntersectionObserver;
  target?: TargetNode;
}
```

类型还是比较直接好懂的，下面我们看代码慢慢理解

### 2.2 观察者封装 observer

首先第一个文件要来看的是关于观察者封装的文件

#### 2.2.1 模块结构

- `/src/observer.ts`（阅读笔记：`/src/observer.ts/0_structure.ts`）

```ts
import { parseRootMargin, shallowCompare } from './utils';
import { Instance, TargetNode } from './types';

export const observerElementsMap = new Map<IntersectionObserver | undefined, Set<Instance>>();

export function getPooled(options: IntersectionObserverInit = {}) {}

export function findObserverElement(observer: IntersectionObserver, entry: IntersectionObserverEntry) {}

export function callback(entries: IntersectionObserverEntry[], observer: IntersectionObserver) {}

export function createObserver(options: IntersectionObserverInit): IntersectionObserver {}

export function observeElement(element: Instance) {}

export function unobserveElement(element: Instance, target: TargetNode) {}
```

大致上分为三区

1. 观察者 IntersectionObserver 实例 => 观察目标数组 Instance[] 缓存 Map 维护(pool)
2. 观察者的创建
3. 观察/取消观察目标

#### 2.2.2 观察者实例缓存/记录

第一块先来看观察者实例的管理

- `/src/observer.ts`（阅读笔记：`/src/observer.ts/1_pool.ts`）

```ts
export const observerElementsMap = new Map<IntersectionObserver | undefined, Set<Instance>>();

/**
 * 获取 IntersectionObserver 实例
 * @param options
 * @returns
 */
export function getPooled(options: IntersectionObserverInit = {}) {
  // 参数处理
  const root = options.root || null;
  const rootMargin = parseRootMargin(options.rootMargin);
  const threshold = Array.isArray(options.threshold)
    ? options.threshold
    : [options.threshold != null ? options.threshold : 0];

  // 查找符合条件的实例
  const observers = observerElementsMap.keys();
  let observer;
  while ((observer = observers.next().value)) {
    const unmatched =
      root !== observer.root || rootMargin !== observer.rootMargin || shallowCompare(threshold, observer.thresholds);

    if (!unmatched) {
      return observer;
    }
  }
  return null;
}
```

这里唯一的重点在于 Map 对象的结构，是 `IntersectionObserver => Set<Instance>` 的映射，也就是一个观察者对应一个观察目标集合

`getPooled` 方法的本质上在于，基于 options 查找已经存在符合条件的 Observer

#### 2.2.3 查找观察目标

有了 pool（observerElementsMap 对象），他还定义了一个基于 Observer 查找观察目标元素的方法

- `/src/observer.ts`（阅读笔记：`/src/observer.ts/2_findObserverElement.ts`）

```ts
/**
 * 查找观察目标 element: Instance
 * @param observer
 * @param entry
 * @returns
 */
export function findObserverElement(observer: IntersectionObserver, entry: IntersectionObserverEntry) {
  const elements = observerElementsMap.get(observer);
  if (elements) {
    const values = elements.values();
    let element: Instance;
    while ((element = values.next().value)) {
      if (element.target === entry.target) {
        // observerElementsMap[observer].element: Instance
        return element;
      }
    }
  }
  return null;
}
```

但是实际上回调函数触发的时候我们已经能够从 entry 上找到观察目标，这里的作用相当于一种 double check，同时出现在 Observer 回调的 entries 和 Map 记录中的元素才会被返回

#### 2.2.4 创建观察者

上面完成了 Pool 相关的 Observer 查找和观察目标 Instance 的查找，最后利用一个核心的 `createObserver` 函数维护/限制 Observer 的创建生命周期

- `/src/observer.ts`（阅读笔记：`/src/observer.ts/3_createObserver.ts`）

```ts
/**
 * 创建/缓存实例
 * @param options
 * @returns
 */
export function createObserver(options: IntersectionObserverInit): IntersectionObserver {
  const pooled = getPooled(options);

  if (pooled) {
    return pooled;
  }

  const observer = new IntersectionObserver(callback, options);
  observerElementsMap.set(observer, new Set<Instance>());
  return observer;
}
```

先用 `getPooled` 查，没有才创建新的

而关于回调函数的 callback 方法

```ts
/**
 * 固定回调函数引用
 *   使用 findObserverElement 动态查找响应元素
 * @param entries
 * @param observer
 */
export function callback(entries: IntersectionObserverEntry[], observer: IntersectionObserver) {
  for (let i = 0; i < entries.length; i++) {
    const element = findObserverElement(observer, entries[i]);
    if (element) {
      element.handleChange(entries[i]);
    }
  }
}
```

这里比较有趣的是这个 callback 其实就固定了，所有 Observer 用的都是同一个 callback，而当 callback 触发的时候才 `findObserverElement` 动态查找 double check 下符合条件的观察目标，最后调用 `handleChange` 方法触发用户回调

#### 2.2.5 观察/取消观察方法

定义好 Observer 实例的生命周期，以及回调函数代理的 callback、观察目标 target 的动态查找，最后只要提供两个用户方法：观察、取消观察目标

- `/src/observer.ts`（阅读笔记：`/src/observer.ts/4_observeElement.ts`）

```ts
/**
 * 观察目标
 * @param element
 */
export function observeElement(element: Instance) {
  if (element.observer && !observerElementsMap.has(element.observer)) {
    observerElementsMap.set(element.observer, new Set<Instance>());
  }
  // observerElementsMap[observer].add(element) 加入缓存池
  observerElementsMap.get(element.observer)?.add(element);
  // observer.observe(element.target) 观察实例
  element.observer!.observe(element.target!);
}
```

- 步骤
  - 确保 Set 存在
  - 把观察目标加入 Set 队列
  - observer.observe 实际观察方法

- `/src/observer.ts`（阅读笔记：`/src/observer.ts/5_unobserveElement.ts`）

```ts
/**
 * 取消观察
 * @param element
 * @param target
 */
export function unobserveElement(element: Instance, target: TargetNode) {
  if (observerElementsMap.has(element.observer)) {
    const targets = observerElementsMap.get(element.observer);
    // observerElementsMap[observer].delete(element) 从缓存池移除
    if (targets?.delete(element)) {
      // observer.unobserve / observer.disconnect 取消观察/删除观察者对象
      if (targets.size > 0) {
        element.observer!.unobserve(target);
      } else {
        element.observer!.disconnect();
        observerElementsMap.delete(element.observer);
      }
    }
  }
}
```

取消观察稍微复杂一点，因为涉及到 Set 队列的释放，同时对应 IntersectionObserver 实例的两种方法（`unobserve` 取消观察一个目标、`disconnect` 卸载整个 Observer 观察）

这个库将 Observer 实例统一封装到 `observer.ts` 模块之中，然后再分别提供两种组件形式供用户使用

### 2.3 hook 版本 - useIntersectionObserver

#### 2.3.1 状态定义

- `/src/hook.ts`（阅读笔记：`/src/hook.ts/1_flag_instance.ts`）

```ts
import { useRef, useCallback, useMemo } from 'react';
import { createObserver, observeElement, unobserveElement } from './observer';
import { ChangeHandler, Options, Unobserve, Instance } from './types';
import { thresholdCacheKey } from './utils';

const noop = () => {};

export const useIntersectionObserver = (
  onChange: ChangeHandler,
  { root, rootMargin, threshold, disabled }: Options = {}
): [React.RefCallback<any>, Unobserve] => {
  // 标志：是否正在观察
  const observingRef = useRef(false);

  // 观察目标实例
  const instanceRef = useRef<Instance>({
    handleChange(event) {
      onChange(event, noop);
    },
  });

  // ...
};
```

useIntersectionObserver 定义了两个状态，分别是
- observingRef：是否正在观察标志
- instanceRef：观察目标实例

#### 2.3.2 取消观察 & 回调函数包装

第二步则是包装一下取消观察和实例上的回调函数 `handleChange` 方法

- `/src/hook.ts`（阅读笔记：`/src/hook.ts/2_unobserve_onChange.ts`）

```ts
import { useRef, useCallback, useMemo } from 'react';
import { createObserver, observeElement, unobserveElement } from './observer';
import { ChangeHandler, Options, Unobserve, Instance } from './types';
import { thresholdCacheKey } from './utils';

export const useIntersectionObserver = (
  onChange: ChangeHandler,
  { root, rootMargin, threshold, disabled }: Options = {}
): [React.RefCallback<any>, Unobserve] => {
  // ...

  // 取消观察方法（代理 unobserveElement 函数）
  const unobserve = useCallback(() => {
    if (instanceRef.current.target && observingRef.current) {
      unobserveElement(instanceRef.current, instanceRef.current.target);
      observingRef.current = false;
    }
  }, []);

  /**
   * 回调函数绑定
   *   避免写在 useEffect 里导致 onChange 改变时滞后
   */
  instanceRef.current.handleChange = function handleChange(event: IntersectionObserverEntry) {
    onChange(event, unobserve);
  };

  // ...
};
```

#### 2.3.3 观察方法

接下来是 hook 内部将要使用到的 observe 方法，在参数更新、元素替换的时候用于重新观察

- `/src/hook.ts`（阅读笔记：`/src/hook.ts/3_observe.ts`）

```ts
import { useRef, useCallback, useMemo } from 'react';
import { createObserver, observeElement, unobserveElement } from './observer';
import { ChangeHandler, Options, Unobserve, Instance } from './types';
import { thresholdCacheKey } from './utils';

export const useIntersectionObserver = (
  onChange: ChangeHandler,
  { root, rootMargin, threshold, disabled }: Options = {}
): [React.RefCallback<any>, Unobserve] => {
  // ...

  // 观察方法
  const observe = () => {
    // 保证 observer、target 皆存在
    if (instanceRef.current.observer && instanceRef.current.target && !observingRef.current) {
      observeElement(instanceRef.current);
      observingRef.current = true;
    }
  };

  // ...
};
```

#### 2.3.4 Observer 实例

接下来很重要的一步是要在钩子里面维护一个 Observer 实例的引用，由于前面 observer.ts 模块已经封装好 Observer 实例的管理，因此这里相当于是消费，并在每次参数修改的时候重新去获取这个 observer 实例

- `/src/hook.ts`（阅读笔记：`/src/hook.ts/4_observer.ts`）

```ts
import { useRef, useCallback, useMemo } from 'react';
import { createObserver, observeElement, unobserveElement } from './observer';
import { ChangeHandler, Options, Unobserve, Instance } from './types';
import { thresholdCacheKey } from './utils';

export const useIntersectionObserver = (
  onChange: ChangeHandler,
  { root, rootMargin, threshold, disabled }: Options = {}
): [React.RefCallback<any>, Unobserve] => {
  // ...

  // threshold 参数缓存
  const memoizedThreshold = useMemo(() => threshold, [thresholdCacheKey(threshold)]);

  /**
   * 维护 instance.observer: IntersectionObserver 实例
   */
  const observer = useMemo(
    () => {
      if (disabled) {
        unobserve();
        instanceRef.current.observer = undefined;
        return undefined;
      }

      // root 参数
      const rootOption = typeof root === 'string' ? document.querySelector(root) : root;

      const obs = createObserver({
        root: rootOption,
        rootMargin,
        threshold: memoizedThreshold,
      });

      instanceRef.current.observer = obs;

      // 重新观察
      unobserve(); // 保证 observing 标记已经清空
      observe();

      return obs;
    },
    // 构建参数是否改变
    [root, rootMargin, memoizedThreshold, disabled]
  );

  // ...
};
```

这里作者取名为 createObserver，实际上如果存在已经符合条件的 observer 就不会创建新的，所以命名上有点歧义，整体逻辑上步骤为

1. 检查 disabled 参数
2. 调用 `createObserver` 获取 Observer 实例
3. 重新观察目标

#### 2.3.5 观察目标 ref

最后我们要怎么获取观察目标呢？作者使用了 callback ref 的实现方式，提供 callback ref 让用户将 ref 标记到目标元素上来获取观察目标

- `/src/hook.ts`（阅读笔记：`/src/hook.ts/5_setRef.ts`）

```ts
import { useRef, useCallback, useMemo } from 'react';
import { createObserver, observeElement, unobserveElement } from './observer';
import { ChangeHandler, Options, Unobserve, Instance } from './types';
import { thresholdCacheKey } from './utils';

export const useIntersectionObserver = (
  onChange: ChangeHandler,
  { root, rootMargin, threshold, disabled }: Options = {}
): [React.RefCallback<any>, Unobserve] => {
  // ...

  /**
   * 置于目标组件上的 callback ref（会调用两次）
   *   node = null   => unobserve
   *   node = target => unobserve + target = node + observe
   */
  const setRef = useCallback<React.RefCallback<any>>(
    (node) => {
      const isNewNode = node != null && instanceRef.current.target !== node;

      // observer 实例不存在
      if (!observer) {
        unobserve();
      }

      // 返回新的 targetNode
      if (isNewNode) {
        unobserve();
        instanceRef.current.target = node;
        observe();
      }

      if (!node) {
        unobserve();
        instanceRef.current.target = undefined;
      }
    },
    [observer]
  );

  return [setRef, unobserve];
};
```

关于 callback ref 的细节请自行查阅文档：[https://zh-hans.reactjs.org/docs/refs-and-the-dom.html#callback-refs](https://zh-hans.reactjs.org/docs/refs-and-the-dom.html#callback-refs)

这里本质上就是检查 node 的更新、进行重新观察（`unobserve + observe`）

最后返回 ref 和主动取消观察的方法

### 2.4 class 组件版本 - IntersectionObserver

#### 2.4.0 组件结构

class 组件对标 hook 版本，同时作者直接让组件实现 Instance 接口，也就是整个组件对象也能够作为 Instance 被观察实例来访问

- `/src/IntersectionObserver.ts`（阅读笔记：`/src/IntersectionObserver.ts/0_structure.ts`）

```ts
import React from 'react';
import { findDOMNode } from 'react-dom';
import { createObserver, observeElement, unobserveElement } from './observer';
import { shallowCompare, isChildrenWithRef, hasOwnProperty, toString } from './utils';
import { ChangeHandler, Options, Instance, TargetNode } from './types';

interface Props extends Options {
  children?: React.ReactElement | null;
  onChange: ChangeHandler;
}

export default class ReactIntersectionObserver extends React.Component<Props, {}> implements Instance {
  static displayName = 'IntersectionObserver';

  private targetNode?: TargetNode;
  private prevTargetNode?: TargetNode;
  public target?: TargetNode;
  public observer?: IntersectionObserver;

  handleChange = (event: IntersectionObserverEntry) => {};

  handleNode = <T extends React.ReactInstance | null | undefined>(target: T) => {};

  observe = () => {};

  unobserve = (target: TargetNode) => {};

  externalUnobserve = () => {};

  getSnapshotBeforeUpdate(prevProps: Props) {}

  componentDidUpdate(_: any, __: any, relatedPropsChanged: boolean) {}

  componentDidMount() {}

  componentWillUnmount() {}

  render() {}
}

export * from './types';
```

下面我们按访问顺序一个个来看代码实现

#### 2.4.1 生命周期相关

首先我们跟着组件的生命周期来看整个组件是怎么运行的

- `/src/IntersectionObserver.ts`（阅读笔记：`/src/IntersectionObserver.ts/1_lifecycle_render.ts`）

```ts
import React from 'react';
import { findDOMNode } from 'react-dom';
import { createObserver, observeElement, unobserveElement } from './observer';
import { shallowCompare, isChildrenWithRef, hasOwnProperty, toString } from './utils';
import { ChangeHandler, Options, Instance, TargetNode } from './types';

export default class ReactIntersectionObserver extends React.Component<Props, {}> implements Instance {
  // ...

  /**
   * 捕获前一次的 targetNode
   * 比较 props 是否改变
   * @param prevProps
   * @returns
   */
  getSnapshotBeforeUpdate(prevProps: Props) {
    this.prevTargetNode = this.targetNode;

    // 浅比较 props
    const relatedPropsChanged = observableProps.some((prop: typeof observableProps[number]) =>
      shallowCompare(this.props[prop], prevProps[prop])
    );
    if (relatedPropsChanged) {
      if (this.prevTargetNode) {
        if (!prevProps.disabled) {
          // props 改变、存在 targetNode、前一次非 disabled 时取消观察
          this.unobserve(this.prevTargetNode);
        }
      }
    }

    return relatedPropsChanged;
  }
```

首先是 update 钩子之前，利用 `getSnapshotBeforeUpdate` 生命周期函数来计算比较 props 是否改变，返回 `relatedPropsChanged`，并记录前一次的观察目标 prevTargetNode

```ts
  /**
   * update 生命周期
   * @param _
   * @param __
   * @param relatedPropsChanged
   */
  componentDidUpdate(_: any, __: any, relatedPropsChanged: boolean) {
    let targetNodeChanged = false;
    // props 没有改变的时候，检查 targetNode 是否改变
    if (!relatedPropsChanged) {
      targetNodeChanged = this.prevTargetNode !== this.targetNode;
      if (targetNodeChanged && this.prevTargetNode != null) {
        this.unobserve(this.prevTargetNode);
      }
    }

    // 重新观察目标
    if (relatedPropsChanged || targetNodeChanged) {
      this.observe();
    }
  }
```

接下来在 update 周期内检查是否需要重新观察

```ts
  // mount 挂载时观察目标
  componentDidMount() {
    this.observe();
  }

  // unmount 卸载时取消观察
  componentWillUnmount() {
    if (this.targetNode) {
      this.unobserve(this.targetNode);
    }
  }
```

当然在挂载、卸载组件的时候也要进行观察的管理

```ts
  // 渲染时传入 callback ref（handleNode 函数）
  render() {
    const { children } = this.props;

    return children != null
      ? React.cloneElement(React.Children.only(children), {
          ref: this.handleNode,
        })
      : null;
  }

  // ...
}

export * from './types';
```

最后在渲染的时候将 callback ref 传入子组件，`React.cloneElement`、`React.Children` 查 API 吧：[https://zh-hans.reactjs.org/docs/react-api.html#cloneelement](https://zh-hans.reactjs.org/docs/react-api.html#cloneelement)

#### 2.4.2 回调函数包装

接下来我们先来看看 Observer 实例会用到的 handleChange 方法

- `/src/IntersectionObserver.ts`（阅读笔记：`/src/IntersectionObserver.ts/2_handleChange.ts`）

```ts
import React from 'react';
import { findDOMNode } from 'react-dom';
import { createObserver, observeElement, unobserveElement } from './observer';
import { shallowCompare, isChildrenWithRef, hasOwnProperty, toString } from './utils';
import { ChangeHandler, Options, Instance, TargetNode } from './types';

export default class ReactIntersectionObserver extends React.Component<Props, {}> implements Instance {
  // ...

  // Instance 回调函数 handleChange 定义
  handleChange = (event: IntersectionObserverEntry) => {
    this.props.onChange(event, this.externalUnobserve);
  };

  // 供外部使用者调用的卸载方法
  externalUnobserve = () => {
    if (this.targetNode) {
      this.unobserve(this.targetNode);
    }
  };

  // ...
}

export * from './types';
```

本质上就是直接代理用户的 onChange 方法，同时提供了一个用户可用的 `externalUnobserve` 方法

#### 2.4.3 callback ref 方法

接下来看到传入子组件的 ref

- `/src/IntersectionObserver.ts`（阅读笔记：`/src/IntersectionObserver.ts/3_handleNode.ts`）

```ts
import React from 'react';
import { findDOMNode } from 'react-dom';
import { createObserver, observeElement, unobserveElement } from './observer';
import { shallowCompare, isChildrenWithRef, hasOwnProperty, toString } from './utils';
import { ChangeHandler, Options, Instance, TargetNode } from './types';

export default class ReactIntersectionObserver extends React.Component<Props, {}> implements Instance {
  // ...

  /**
   * callback ref
   * @param target
   */
  handleNode = <T extends React.ReactInstance | null | undefined>(target: T) => {
    const { children } = this.props;

    if (isChildrenWithRef<T>(children)) {
      // 重新绑定子节点 ref
      const childenRef = children.ref;
      if (typeof childenRef === 'function') {
        childenRef(target);
      } else if (childenRef && hasOwnProperty.call(childenRef, 'current')) {
        (childenRef as React.MutableRefObject<T>).current = target;
      }
    }

    // 维护目标 dom 元素到 targetNode
    this.targetNode = undefined;
    if (target) {
      // 重新获取目标节点元素
      const targetNode = findDOMNode(target);
      if (targetNode && targetNode.nodeType === 1) {
        this.targetNode = targetNode as Element;
      }
    }
  };

  // ...
}

export * from './types';
```

这里作者写了一个有点侵入性的代码，强制覆盖了 childrenRef；后面再重新绑定观察目标 targetNode

#### 2.4.4 观察/取消观察方法

最后是组件内部使用的观察/取消观察方法

- `/src/IntersectionObserver.ts`（阅读笔记：`/src/IntersectionObserver.ts/4_observe.ts`）

```ts
import React from 'react';
import { findDOMNode } from 'react-dom';
import { createObserver, observeElement, unobserveElement } from './observer';
import { shallowCompare, isChildrenWithRef, hasOwnProperty, toString } from './utils';
import { ChangeHandler, Options, Instance, TargetNode } from './types';

const observerOptions = <const>['root', 'rootMargin', 'threshold'];

/**
 * options 参数预处理
 * @param props
 * @returns
 */
export const getOptions = (props: Props) => {
  return observerOptions.reduce<IntersectionObserverInit>((options, key) => {
    const isRootString = key === 'root' && toString.call(props.root) === '[object String]';

    return Object.assign(options, {
      [key]: isRootString ? document.querySelector(props[key] as string) : props[key],
    });
  }, {});
};

export default class ReactIntersectionObserver extends React.Component<Props, {}> implements Instance {
  // ...

  /**
   * 观察方法
   * @returns
   */
  observe = () => {
    // 不存在子节点 or disabled
    if (this.props.children == null || this.props.disabled) {
      return false;
    }
    if (!this.targetNode) {
      throw new Error(
        "ReactIntersectionObserver: Can't find DOM node in the provided children. Make sure to render at least one DOM node in the tree."
      );
    }

    // 获取 observer 实例
    this.observer = createObserver(getOptions(this.props));
    this.target = this.targetNode;
    observeElement(this);

    return true;
  };

  // ...
}

export * from './types';
```

- `/src/IntersectionObserver.ts`（阅读笔记：`/src/IntersectionObserver.ts/5_unobserve.ts`）

```ts
import React from 'react';
import { findDOMNode } from 'react-dom';
import { createObserver, observeElement, unobserveElement } from './observer';
import { shallowCompare, isChildrenWithRef, hasOwnProperty, toString } from './utils';
import { ChangeHandler, Options, Instance, TargetNode } from './types';

export default class ReactIntersectionObserver extends React.Component<Props, {}> implements Instance {
  // ...

  unobserve = (target: TargetNode) => {
    unobserveElement(this, target);
  };

  // ...
}

export * from './types';
```

本质上就是对 `observeElement`、`unobserveElement` 再包装

## 3. react-intersection-observer@8.26.1（官方推荐）

第二个才是官方推荐的库，看过源码之后确实也是比较简洁，语法上更为紧凑

- 目录结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_intersection_observer_library_2_official_modules.png)

两个库的实现模式简直不要太像，我怀疑前面那个库的作者估计参考了这个官方推荐的实现；关于 WebAPI 的其他可用对象其实也可以使用类似的封装模式，值得读者另作思考

- 阅读笔记：[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/react-intersection-observer-8.26.1](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/react-intersection-observer-8.26.1)

下面马上开始

### 3.1 类型定义 index

首先是类型定义，作者直接就写在 index 里面hh

- `/src/index.ts`（阅读笔记：`/src/index.ts`）

```ts
export type ObserverInstanceCallback = (inView: boolean, entry: IntersectionObserverEntry) => void;

export type ObserverInstance = {
  inView: boolean;
  readonly callback: ObserverInstanceCallback;
  readonly element: Element;
  readonly observerId: string;
  readonly observer: IntersectionObserver;
  readonly thresholds: ReadonlyArray<number>;
};
```

官方库也有一个观察对象实例

```ts
interface RenderProps {
  inView: boolean;
  entry: IntersectionObserverEntry | undefined;
  ref: React.RefObject<any> | ((node?: Element | null) => void);
}

export interface IntersectionOptions extends IntersectionObserverInit {
  root?: Element | null;
  rootMargin?: string;
  threshold?: number | number[];
  triggerOnce?: boolean;
  skip?: boolean;
  initialInView?: boolean;
  trackVisibility?: boolean;
  delay?: number;
}

export interface IntersectionObserverProps extends IntersectionOptions {
  children: (fields: RenderProps) => React.ReactNode;
  onChange?: (inView: boolean, entry: IntersectionObserverEntry) => void;
}
```

两个配置参数选项

```ts
export type PlainChildrenProps = IntersectionOptions & {
  children?: React.ReactNode;
  as?: React.ReactType<any>;
  tag?: React.ReactType<any>;
  onChange?: (inView: boolean, entry: IntersectionObserverEntry) => void;
} & Omit<React.HTMLProps<HTMLElement>, 'onChange'>;

export type InViewHookResponse = [(node?: Element | null) => void, boolean, IntersectionObserverEntry | undefined] & {
  ref: (node?: Element | null) => void;
  inView: boolean;
  entry?: IntersectionObserverEntry;
};
```

还有分别为 render props 模式和 hook 版本提供的类型定义

### 3.2 观察者封装 observe

接下来一样，有一个观察者实例的封装模块

#### 3.2.0 模块结构

这个库的封装比较紧凑一些，方法也比较少

- `/src/observe.ts`（阅读笔记：`/src/observe.ts/0_structure.ts`）

```ts
import { ObserverInstanceCallback } from './index';

function getRootId(root: IntersectionObserverInit['root']) {}

export function optionsToId(options: IntersectionObserverInit) {}

function createObserver(options: IntersectionObserverInit) {}

export function observe(element: Element, callback: ObserverInstanceCallback, options: IntersectionObserverInit = {}) {}
```

#### 3.2.1 root 参数缓存 key

首先第一个方法是为 root 元素创建唯一 id

- `/src/observe.ts`（阅读笔记：`/src/observe.ts/1_getRootId.ts`）

```ts
const RootIds: WeakMap<Element | Document, string> = new WeakMap();
let rootId = 0;

/**
 * 为 root 元素创建唯一 id 并缓存
 * @param root 
 * @returns 
 */
function getRootId(root: IntersectionObserverInit['root']) {
  if (!root) return '0';
  if (RootIds.has(root)) return RootIds.get(root);
  rootId += 1;
  RootIds.set(root, rootId.toString());
  return RootIds.get(root);
}
```

#### 3.2.2 options 参数缓存 key

第二个方法还是创建缓存 key，也就是用于唯一标识并区分不同 Observer 实例，同一组 options 配置将指向同一个 Observer 实例

- `/src/observe.ts`（阅读笔记：`/src/observe.ts/2_optionsToId.ts`）

```ts
/**
 * 为 options 创建 id(按值特征)
 * @param options
 * @returns
 */
export function optionsToId(options: IntersectionObserverInit) {
  return Object.keys(options)
    .sort()
    .filter((key) => options[key] !== undefined)
    .map((key) => {
      return `${key}_${key === 'root' ? getRootId(options.root) : options[key]}`;
    })
    .toString();
}
```

#### 3.2.3 Observer 实例创建

- `/src/observe.ts`（阅读笔记：`/src/observe.ts/3_createObserver.ts`）

```ts
const ObserverMap = new Map<
  string,
  {
    id: string;
    observer: IntersectionObserver;
    elements: Map<Element, Array<ObserverInstanceCallback>>;
  }
>();

function createObserver(options: IntersectionObserverInit) {
  // optionID 唯一标识 Observer 实例
  let id = optionsToId(options);
  let instance = ObserverMap.get(id);

  if (!instance) {
    // 不存在 => 创建一个新的
    const elements = new Map<Element, Array<ObserverInstanceCallback>>();
    let thresholds: number[] | readonly number[];

    const observer = new IntersectionObserver((entries) => {
      entries.forEach((entry) => {
        // 计算是否出现 inView
        const inView = entry.isIntersecting && thresholds.some((threshold) => entry.intersectionRatio >= threshold);

        // trackVisibility 决定是否使用 isVisible
        if (options.trackVisibility && typeof entry.isVisible === 'undefined') {
          entry.isVisible = inView;
        }

        // 调用回调
        elements.get(entry.target)?.forEach((callback) => {
          callback(inView, entry);
        });
      });
    }, options);

    // thresholds 参数
    thresholds =
      observer.thresholds || (Array.isArray(options.threshold) ? options.threshold : [options.threshold || 0]);

    // 创建实例（id、observer、观察目标）
    instance = {
      id,
      observer,
      elements,
    };

    ObserverMap.set(id, instance);
  }

  return instance;
}
```

- 步骤
  - 检查缓存 Map 是否存在同 key 的实例
  - 创建 Observer 对象，并用闭包绑定 thresholds 参数、观察目标 elements
  - 将实例写入缓存 map 当中

#### 3.2.4 observe 观察目标

最后提供了一个观察目标的方法，并将取消方法作为返回值返回

- `/src/observe.ts`（阅读笔记：`/src/observe.ts/4_observe.ts`）

```ts
/**
 * 观察目标
 * @param element
 * @param callback
 * @param options
 * @returns
 */
export function observe(element: Element, callback: ObserverInstanceCallback, options: IntersectionObserverInit = {}) {
  // 无目标
  if (!element) return () => {};

  // 按 options 获取 observer 实例
  const { id, observer, elements } = createObserver(options);

  // 注册回调函数
  let callbacks = elements.get(element) || [];
  if (!elements.has(element)) {
    elements.set(element, callbacks);
  }

  callbacks.push(callback);
  observer.observe(element); // 观察目标

  // 返回卸载方法
  return function unobserve() {
    // 从记录中移除
    callbacks.splice(callbacks.indexOf(callback), 1);

    // 一个 element 对应一个 callback 列表
    if (callbacks.length === 0) {
      // callbacks 为空
      elements.delete(element);
      observer.unobserve(element);
    }

    if (elements.size === 0) {
      // elements 为空
      observer.disconnect();
      ObserverMap.delete(id);
    }
  };
}
```

一样取消之后也会有缓存 Map 的资源释放问题

### 3.3 hook 版本 - useInView

看完 Observer 实例的封装，我们一样先来看 hook 版本的实现

比起前面那个库确实是少了很多不必要的逻辑，代码更为紧凑，提高了理解深度但是实际上降低整体的理解复杂度

#### 3.3.1 状态保存

- `/src/useInView.ts`（阅读笔记：`/src/useInView.tsx/1_state.ts`）

```ts
import * as React from 'react';
import { InViewHookResponse, IntersectionOptions } from './index';
import { useEffect } from 'react';
import { observe } from './observe';

type State = {
  inView: boolean;
  entry?: IntersectionObserverEntry;
};

export function useInView({
  threshold,
  delay,
  trackVisibility,
  rootMargin,
  root,
  triggerOnce,
  skip,
  initialInView,
}: IntersectionOptions = {}): InViewHookResponse {
  // 卸载方法
  const unobserve = React.useRef<Function>();
  // 数据状态
  const [state, setState] = React.useState<State>({
    inView: !!initialInView,
  });

  // ...
}
```

`useInView` 钩子保存了两种状态：一个是卸载方法，一个是回调函数的响应绑定

这里有一个很有意思的部分，作者在这里实现了一个从回调 callback 到 React state 的映射，也就是说对于响应函数的操作从 callback 的调用变成根据 state 的更新来触发回调

也就是说使用的时候我们变成像是

```ts
const { entry } = useInView()
useEffect(() => {
  // callback here
}, [entry])
```

#### 3.3.2 callback ref

接下来是 callback ref，也就是用于定位观察目标的，同时作者将观察的逻辑都封装到这个方法里面，避免了多余复杂的状态判断

- `/src/useInView.ts`（阅读笔记：`/src/useInView.tsx/2_setRef.ts`）

```ts
import * as React from 'react';
import { InViewHookResponse, IntersectionOptions } from './index';
import { useEffect } from 'react';
import { observe } from './observe';

type State = {
  inView: boolean;
  entry?: IntersectionObserverEntry;
};

export function useInView({
  threshold,
  delay,
  trackVisibility,
  rootMargin,
  root,
  triggerOnce,
  skip,
  initialInView,
}: IntersectionOptions = {}): InViewHookResponse {
  // ...

  /**
   * ref callback
   */
  const setRef = React.useCallback(
    (node) => {
      // node 更新时（调用 setRef callback）取消观察
      if (unobserve.current !== undefined) {
        unobserve.current();
        unobserve.current = undefined;
      }

      // 跳过
      if (skip) return;

      // node 存在 => 重新观察
      if (node) {
        unobserve.current = observe(
          node,
          (inView, entry) => {
            setState({ inView, entry });

            // 触发一次
            if (entry.isIntersecting && triggerOnce && unobserve.current) {
              unobserve.current();
              unobserve.current = undefined;
            }
          },
          {
            root,
            rootMargin,
            threshold,
            trackVisibility,
            delay,
          }
        );
      }
    },
    [
      Array.isArray(threshold) ? threshold.toString() : threshold,
      root,
      rootMargin,
      triggerOnce,
      skip,
      trackVisibility,
      delay,
    ]
  );

  // ...
}
```

#### 3.3.3 状态清理 & 返回结果

- `/src/useInView.ts`（阅读笔记：`/src/useInView.tsx/3_result.ts`）

```ts
import * as React from 'react';
import { InViewHookResponse, IntersectionOptions } from './index';
import { useEffect } from 'react';
import { observe } from './observe';

type State = {
  inView: boolean;
  entry?: IntersectionObserverEntry;
};

export function useInView({
  threshold,
  delay,
  trackVisibility,
  rootMargin,
  root,
  triggerOnce,
  skip,
  initialInView,
}: IntersectionOptions = {}): InViewHookResponse {
  // ...

  useEffect(() => {
    // update 时重置 inView
    if (!unobserve.current && state.entry && !triggerOnce && !skip) {
      setState({
        inView: !!initialInView,
      });
    }
  });

  // 返回结果
  const result = [setRef, state.inView, state.entry] as InViewHookResponse;

  result.ref = result[0];
  result.inView = result[1];
  result.entry = result[2];

  return result;
}
```

最后一部分是关于 state 的状态清理，由于 node 不存在时会忽略不处理，所以需要额外的 useEffect 来重置状态

最后又是一个小妙手，利用解构赋值的特性，同时支持数组解构和对象解构

### 3.4 class 组件版本 - InView

看完 hook 版本，class 版本就简单啦；比较复杂的部分在于 class 组件还分成一般 Wrapper 组件和 render props 两种用法

#### 3.4.0 组件结构

先来看下组件结构

- `/src/InView.tsx`（阅读笔记：`/src/InView.tsx/0_structure.ts`）

```ts
import * as React from 'react';
import { IntersectionObserverProps, PlainChildrenProps } from './index';
import { observe } from './observe';

type State = {
  inView: boolean;
  entry?: IntersectionObserverEntry;
};

export class InView extends React.Component<IntersectionObserverProps | PlainChildrenProps, State> {
  static displayName = 'InView';
  static defaultProps = {
    threshold: 0,
    triggerOnce: false,
    initialInView: false,
  };

  constructor(props: IntersectionObserverProps | PlainChildrenProps) {}

  componentDidUpdate(prevProps: IntersectionObserverProps) {}

  componentWillUnmount() {}

  node: Element | null = null;
  _unobserveCb: (() => void) | null = null;

  observeNode() {}

  unobserve() {}

  handleNode = (node?: Element | null) => {};

  handleChange = (inView: boolean, entry: IntersectionObserverEntry) => {};

  render() {}
}
```

#### 3.4.1 生命周期相关

首先一样先按组件生命周期过一遍

- `/src/InView.tsx`（阅读笔记：`/src/InView.tsx/1_lifecycle.ts`）

```ts
import * as React from 'react';
import { IntersectionObserverProps, PlainChildrenProps } from './index';
import { observe } from './observe';

type State = {
  inView: boolean;
  entry?: IntersectionObserverEntry;
};

export class InView extends React.Component<IntersectionObserverProps | PlainChildrenProps, State> {
  // ...

  // 构造时初始化 state
  constructor(props: IntersectionObserverProps | PlainChildrenProps) {
    super(props);
    this.state = {
      inView: !!props.initialInView,
      entry: undefined,
    };
  }
```

构造函数，初始化 state，标准的 React class 组件写法

```ts
  // 更新时 props 改变则重新观察
  componentDidUpdate(prevProps: IntersectionObserverProps) {
    if (
      prevProps.rootMargin !== this.props.rootMargin ||
      prevProps.root !== this.props.root ||
      prevProps.threshold !== this.props.threshold ||
      prevProps.skip !== this.props.skip ||
      prevProps.trackVisibility !== this.props.trackVisibility ||
      prevProps.delay !== this.props.delay
    ) {
      this.unobserve();
      this.observeNode();
    }
  }
```

update 生命周期则是判断 props 是否修改，是则重新观察目标

```ts
  // 卸载时取消观察
  componentWillUnmount() {
    this.unobserve();
    this.node = null;
  }
```

卸载没啥好说了，取消观察然后清理引用

```ts
  // 渲染
  render() {
    // props render
    if (!isPlainChildren(this.props)) {
      const { inView, entry } = this.state;
      return this.props.children({ inView, entry, ref: this.handleNode });
    }

    const {
      children,
      as,
      tag,
      triggerOnce,
      threshold,
      root,
      rootMargin,
      onChange,
      skip,
      trackVisibility,
      delay,
      initialInView,
      ...props
    } = this.props;

    return React.createElement(as || tag || 'div', { ref: this.handleNode, ...props }, children);
  }

  // ...
}
```

渲染的时候会利用下面这个 `isPlainChildren` 判断是不是使用 render props 的模式，否则使用原本的 `React.createElement` 函数创建组件

```ts
/**
 * 非 props render
 * @param props
 * @returns
 */
function isPlainChildren(props: IntersectionObserverProps | PlainChildrenProps): props is PlainChildrenProps {
  return typeof props.children !== 'function';
}
```

#### 3.4.2 callback ref

看完整个状态的生命周期，接下来看看观察目标的 ref 怎么管理

- `/src/InView.tsx`（阅读笔记：`/src/InView.tsx/2_handleNode.ts`）

```ts
import * as React from 'react';
import { IntersectionObserverProps, PlainChildrenProps } from './index';
import { observe } from './observe';

type State = {
  inView: boolean;
  entry?: IntersectionObserverEntry;
};

export class InView extends React.Component<IntersectionObserverProps | PlainChildrenProps, State> {
  // ...

  /**
   * targetNode callback ref
   * @param node
   */
  handleNode = (node?: Element | null) => {
    if (this.node) {
      // 清理旧观察着
      this.unobserve();

      // 为下一次重新观察初始化状态
      if (!node && !this.props.triggerOnce && !this.props.skip) {
        this.setState({ inView: !!this.props.initialInView, entry: undefined });
      }
    }
    this.node = node ? node : null;
    this.observeNode();
  };

  // ...
}
```

相比于前一个库，这个库的逻辑就清晰很多，本质上就是 unobserve、observe 重新绑定，没有过多的复杂状态逻辑

#### 3.4.3 观察方法

观察目标的时候就是简单调用 observe 方法，然后保留一下取消函数就行了

- `/src/InView.tsx`（阅读笔记：`/src/InView.tsx/3_observeNode.ts`）

```ts
import * as React from 'react';
import { IntersectionObserverProps, PlainChildrenProps } from './index';
import { observe } from './observe';

type State = {
  inView: boolean;
  entry?: IntersectionObserverEntry;
};

export class InView extends React.Component<IntersectionObserverProps | PlainChildrenProps, State> {
  // ...

  node: Element | null = null;
  _unobserveCb: (() => void) | null = null;

  /**
   * 观察节点
   * @returns
   */
  observeNode() {
    if (!this.node || this.props.skip) return;
    const { threshold, root, rootMargin, trackVisibility, delay } = this.props;

    this._unobserveCb = observe(this.node, this.handleChange, {
      threshold,
      root,
      rootMargin,
      trackVisibility,
      delay,
    });
  }

  // ...
}
```

#### 3.4.4 回调函数包装

- `/src/InView.tsx`（阅读笔记：`/src/InView.tsx/4_handleChange.ts`）

```ts
import * as React from 'react';
import { IntersectionObserverProps, PlainChildrenProps } from './index';
import { observe } from './observe';

type State = {
  inView: boolean;
  entry?: IntersectionObserverEntry;
};

export class InView extends React.Component<IntersectionObserverProps | PlainChildrenProps, State> {
  // ...

  handleChange = (inView: boolean, entry: IntersectionObserverEntry) => {
    // triggerOnce 只触发一次
    if (inView && this.props.triggerOnce) {
      this.unobserve();
    }

    // render props
    if (!isPlainChildren(this.props)) {
      this.setState({ inView, entry });
    }

    // 调用回调
    if (this.props.onChange) {
      this.props.onChange(inView, entry);
    }
  };

  // ...
}
```

以上个库的逻辑来说，用户回传入回调函数；但是这个库又分成两种用法：

- 简单包裹组件的话就是调用 onChange
- render props 模式则是利用 state 状态来标记，供 render 方法渲染时传入

#### 3.4.5 取消监听

取消监听就简单了，调用一下 unobserve，染灰清理一下引用就行了

- `/src/InView.tsx`（阅读笔记：`/src/InView.tsx/5_unobserve.ts`）

```ts
import * as React from 'react';
import { IntersectionObserverProps, PlainChildrenProps } from './index';
import { observe } from './observe';

type State = {
  inView: boolean;
  entry?: IntersectionObserverEntry;
};

export class InView extends React.Component<IntersectionObserverProps | PlainChildrenProps, State> {
  // ...

  // 取消观察
  unobserve() {
    if (this._unobserveCb) {
      this._unobserveCb();
      this._unobserveCb = null;
    }
  }

  // ...
}
```

## 4. 总结

本篇介绍的两个库都是对于 IntersectionObserver API 的封装，因此实际上并不算是什么特别大的库，甚至都比 redux 要小得多；同时我们可以从两个库的实现看出一些逻辑：

- 对于核心 API 的封装
- 面对不同使用场景提供客制化方法

两个库都实现了类似两层模型，下层封装 IntersectionObserver 实例与观察目标队列的生命周期管理，上层根据用户需要提供 hook、class 组件等方式接入

# 其他资源

## 参考连接

| Title                                             | Link                                                                                                                                                                                             |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| react-intersection-observer - npm                 | [https://www.npmjs.com/package/react-intersection-observer](https://www.npmjs.com/package/react-intersection-observer)                                                                           |
| thebuilder/react-intersection-observer - Github   | [https://github.com/thebuilder/react-intersection-observer](https://github.com/thebuilder/react-intersection-observer)                                                                           |
| react-intersection-observer - storybook           | [https://react-intersection-observer.vercel.app/?path=/story/introduction--page](https://react-intersection-observer.vercel.app/?path=/story/introduction--page)                                 |
| @researchgate/react-intersection-observer - npm   | [https://www.npmjs.com/package/@researchgate/react-intersection-observer](https://www.npmjs.com/package/@researchgate/react-intersection-observer)                                               |
| researchgate/react-intersection-observer - Github | [https://github.com/researchgate/react-intersection-observer](https://github.com/researchgate/react-intersection-observer)                                                                       |
| React Intersection Observer - storybook           | [https://researchgate.github.io/react-intersection-observer/?path=/story/playground--rootmargin](https://researchgate.github.io/react-intersection-observer/?path=/story/playground--rootmargin) |
| Intersection Observer - MDN                       | [https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver)                                                   |
| IntersectionObserverEntry - MDN                   | [https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserverEntry](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserverEntry)                                         |
| 回调 Refs - React 官方                            | [https://zh-hans.reactjs.org/docs/refs-and-the-dom.html#callback-refs](https://zh-hans.reactjs.org/docs/refs-and-the-dom.html#callback-refs)                                                     |
| findDOMNode() - React 官方                        | [https://zh-hans.reactjs.org/docs/react-dom.html#finddomnode](https://zh-hans.reactjs.org/docs/react-dom.html#finddomnode)                                                                       |
| cloneElement() - React 官方                       | [https://zh-hans.reactjs.org/docs/react-api.html#cloneelement](https://zh-hans.reactjs.org/docs/react-api.html#cloneelement)                                                                     |
| React.Children - React 官方                       | [https://zh-hans.reactjs.org/docs/react-api.html#reactchildren](https://zh-hans.reactjs.org/docs/react-api.html#reactchildren)                                                                   |
| getSnapshotBeforeUpdate() - React 官方            | [https://zh-hans.reactjs.org/docs/react-component.html#getsnapshotbeforeupdate](https://zh-hans.reactjs.org/docs/react-component.html#getsnapshotbeforeupdate)                                   |
| React Ref 其实是这样的                            | [https://www.cnblogs.com/zhongmeizhi/p/13819158.html](https://www.cnblogs.com/zhongmeizhi/p/13819158.html)                                                                                       |
| react中findDomNode的作用                          | [https://blog.csdn.net/margin_0px/article/details/81331159](https://blog.csdn.net/margin_0px/article/details/81331159)                                                                           |
| react之React.cloneElement()                       | [https://www.jianshu.com/p/be2f22e6bb78](https://www.jianshu.com/p/be2f22e6bb78)                                                                                                                 |

## 阅读笔记参考

- @researchgate/react-intersection-observer@1.3.5 阅读笔记：[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/%40researchgate__react-intersection-observer-1.3.5](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/%40researchgate__react-intersection-observer-1.3.5)
- react-intersection-observer@8.26.1 阅读笔记：[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/react-intersection-observer-8.26.1](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/react-intersection-observer-8.26.1)
