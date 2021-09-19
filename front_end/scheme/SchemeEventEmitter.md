# 技术方案实践: EventEmitter 和 Observable 对象实现

@[TOC](文章目录)

<!-- TOC -->

- [技术方案实践: EventEmitter 和 Observable 对象实现](#技术方案实践-eventemitter-和-observable-对象实现)
- [前言](#前言)
- [正文](#正文)
  - [1. EventEmitter 实现](#1-eventemitter-实现)
    - [1.1 类型定义](#11-类型定义)
    - [1.2 on、off 注册/取消监听函数实现](#12-onoff-注册取消监听函数实现)
    - [1.3 emit 触发事件实现](#13-emit-触发事件实现)
    - [1.4 once 只触发一次实现](#14-once-只触发一次实现)
    - [1.5 测试](#15-测试)
  - [2. Observable 实现](#2-observable-实现)
    - [2.1 类型定义](#21-类型定义)
    - [2.2 代码实现](#22-代码实现)
    - [2.3 测试](#23-测试)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

在前端领域用得最多的设计模式离不开观察者模式，Nodejs 中也在 events 模块提供了一个功能强大的事件触发器 Emitter，如果你真的很在意你的项目打包大小的话，不妨自己实现一个 EventEmitter，也就是许多面试题经常出现的那一个，只实现最基本的 on、off、emit、once 等方法只需要个位数 KB 的大小

本篇除了介绍 EventEmitter 的实现方案之外，还给各位带来一个 emitter 的应用场景：Observable 对象，利用 emitter 作为事件分发中心，与 Proxy 结合创造出一个可观察对象，并在对象更新的时候自动通知所有观察者。

# 正文

## 1. EventEmitter 实现

面试多几次的大概都被问过，或是使用 Node 的时候也用过，这里就不再对 EventEmitter 做过分的解说，本质上我们就是要来实现一个事件触发类就对了

### 1.1 类型定义

由于我们的目标只是实现一个基本款，有基本功能的触发器就好了，不需要像 nodejs 那样全面，所以我们先来定义一下类型

- `/types/emitter/interface.d.ts`

```ts
export interface EventEmitterListener<P = void> {
    (param: P): void;
}
```

- `/types/emitter/index.d.ts`

```ts
import { EventEmitterListener } from './interface';
/**
 * 事件触发器
 * E 为事件类型，默认 string
 * T 为传递数据类型，默认为 void
 */
declare class EventEmitter<E extends string, T = void> {
    private listenersMap;
    private ensureListenersMap;
    on(event: E, listener: EventEmitterListener<T>): void;
    off(event: E, listener: EventEmitterListener<T>): void;
    emit(event: E, param: T): void;
    once(event: E, listener: EventEmitterListener<T>): void;
}
export default EventEmitter;
```

这次的实现我们目标就是实现四个方法：

- on 监听事件
- off 取消监听
- emit 触发事件
- once 注册只触发一次的监听函数

### 1.2 on、off 注册/取消监听函数实现

接下来第一个我们先来看关于监听函数的注册和取消的实现

- `/src/emitter/index.ts`

首先我们先有一个方法确保监听队列（监听函数的集合 listeners）的存在

```ts
class EventEmitter<E extends string, T = void> {
  private listenersMap: Map<E, Set<EventEmitterListener<T>>> = null;

  private ensureListenersMap(event: E): void {
    if (!this.listenersMap) {
      this.listenersMap = new Map();
    }
    if (!this.listenersMap.has(event)) {
      this.listenersMap.set(event, new Set());
    }
  }

  // ...
```

接下来注册方法就是往队列中添加一个监听函数

```ts
  on(event: E, listener: EventEmitterListener<T>) {
    this.ensureListenersMap(event);
    const listeners = this.listenersMap.get(event);
    if (!listeners.has(listener)) {
      listeners.add(listener);
    }
  }
```

而取消监听则是将监听函数从这个集合中移除

```ts
  off(event: E, listener: EventEmitterListener<T>) {
    if (this.listenersMap.has(event)) {
      const listeners = this.listenersMap.get(event);
      if (listeners.has(listener)) {
        listeners.delete(listener);
      }
    }
  }
```

### 1.3 emit 触发事件实现

当我们把监听函数处理好了，触发事件也就简单了

- `/src/emitter/index.ts`

```ts
  emit(event: E, param: T) {
    if (this.listenersMap.has(event)) {
      const listeners = this.listenersMap.get(event);
      listeners.forEach((listener) => {
        listener(param);
      });
    }
  }
```

### 1.4 once 只触发一次实现

实现 once 的时候一开始我是想着去改动 on、off 的方法，但是实际上我们可以稍微包装一下监听函数，拓展成执行后自动取消监听的方法，就能够实现我们的目标

- `/src/emitter/index.ts`

```ts
  once(event: E, listener: EventEmitterListener<T>) {
    const wrappedListener: EventEmitterListener<T> = (param: T) => {
      listener(param);
      this.off(event, wrappedListener);
    };
    this.on(event, wrappedListener);
  }
```

当然，这样写会造成一个问题是往后没办法直接使用 off 取消这个监听函数，不过我们依旧可以透过一个新的 Map(listener => wrappedListener) 的映射来满足这个需求

### 1.5 测试

最后都写好之后我们稍微测试一下是否符合预期

- `/src/tests/emitter.test.ts`

首先我们定义一个事件类型的集合

```ts
import EventEmitter from '../emitter';

export enum EventEmitterEvent {
  EventA = 'EventA',
  EventB = 'EventB',
}
```

接下来我们分成两块输出

第一块注册了四个监听函数、其中一个用的是 once 方法

```ts
const emitter = new EventEmitter<EventEmitterEvent>();

const listenerA1 = () => {
  console.log('listenerA1 listen for EventEmitterEvent.EventA');
};

const listenerA2 = () => {
  console.log('listenerA2 listen for EventEmitterEvent.EventA');
};

const listenerB1 = () => {
  console.log('listenerB1 listen for EventEmitterEvent.EventB');
};

const listenerB2 = () => {
  console.log('listenerB2 listen for EventEmitterEvent.EventB');
};

console.group('>>>>> group 1');
emitter.on(EventEmitterEvent.EventA, listenerA1);
console.log('[log] on listenerA1');
emitter.on(EventEmitterEvent.EventA, listenerA2);
console.log('[log] on listenerA2');
emitter.on(EventEmitterEvent.EventB, listenerB1);
console.log('[log] on listenerB');
emitter.once(EventEmitterEvent.EventB, listenerB2);
console.log('[log] once listenerB2');
console.log('emitter info', emitter);

console.log('[log] invoke EventA');
emitter.emit(EventEmitterEvent.EventA);
console.log('[log] invoke EventB');
emitter.emit(EventEmitterEvent.EventB);

console.log('emitter info', emitter);
console.groupEnd();
```

第二块我们取消一个监听函数

```ts
console.group('>>>>> group 2');
emitter.off(EventEmitterEvent.EventA, listenerA1);
console.log('[log] off listenerA1');
console.log('emitter info', emitter);

console.log('[log] invoke EventA');
emitter.emit(EventEmitterEvent.EventA);
console.log('[log] invoke EventB');
emitter.emit(EventEmitterEvent.EventB);
console.groupEnd();
```

输出结果如下，是符合预期的

![](https://picures.oss-cn-beijing.aliyuncs.com/img/scheme_event_emitter_1_event_emitter.png)

## 2. Observable 实现

实现了 EventEmitter，或是学会了 Node 提供的版本，有的时候我们还是不知道要怎么用，接下来我们就给出本篇的第二个实现，应用 emitter 对象，创建一个可观察的 Observable 对象实现

### 2.1 类型定义

一样在开始之前我们先明确我们的目标，我们到底想创建出什么样的东西

- `/types/observable/interface.d.ts`

```ts
export interface ObservableListener<T> {
    (data: T): void;
}

export interface Observable<T> {
    subscribe(listener: ObservableListener<T>): void;
}
```

我们的目标是想创建出一个 Observable\<T\> 类型的对象，它能够额外提供一个 subscribe 方法用于监听数据的变化

- `/types/observable/index.d.ts`

```ts
import { Observable } from './interface';
/**
 * 创建一个可观察对象
 * 内部使用 Proxy 代理
 * @param obj
 * @returns
 */
declare const createObservable: <T extends Object>(obj: T) => T & Observable<T>;
export default createObservable;
```

而我们的核心目标在于实现这一个 createObservable 方法来创建一个可观察对象

### 2.2 代码实现

实现上还是比较简单的，一口气贴出来

- `/src/observable/index.ts`

```ts
import EventEmitter from '../emitter';
import { Observable, ObservableListener } from './interface';

/**
 * 创建一个可观察对象
 * 内部使用 Proxy 代理
 * @param obj
 * @returns
 */
const createObservable = <T extends Object>(obj: T): T & Observable<T> => {
  const ON_DATA_UPDATE = 'ON_DATA_UPDATE';
  const emitter = new EventEmitter<string, T>();

  const proxy = new Proxy(
    Object.assign(obj, {
      subscribe(listener: ObservableListener<T>) {
        emitter.on(ON_DATA_UPDATE, listener);
      },
    }),
    {
      set(target, propKey, value, receiver) {
        const res = Reflect.set(target, propKey, value, receiver);
        emitter.emit(ON_DATA_UPDATE, target);
        return res;
      },
    }
  );
  return proxy;
};

export default createObservable;
```

关于 Proxy 的原理就不再详细赘述，这个 Observable 的实现目的在于让大家直观的去感受一个 EventEmitter 的作用，不仅仅能使用 Proxy 作为触发源，EventEmitter 本质上就是提供一个事件收发的管理中心，我们就可以在其他任意地方将事件的管理委托给一个 EventEmitter 的实例

### 2.3 测试

- `/src/tests/observable.test.ts`

```ts
import createObservable from '../observable';

const obj = createObservable({ name: 'Jason' });

console.log('obj', obj);

obj.subscribe((obj) => {
  console.log('on obj update', obj);
});

obj.name = 'Jack';
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/scheme_event_emitter_2_observable.png)

# 结语

本篇的实现还是比较简单的，主要的目的在于自己整理一下 EventEmitter 的实践过程与思考（之前面试被问了好几次都是临时瞎写hh），同时也尝试的实现了一下 Observable 对象，也是看到曾经在某个库里面实现了，也算是练习将 EventEmitter 应用到代码里面去。

# 其他资源

## 参考连接

| Title | Link |
| ----- | ---- |
|       | []() |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/scheme/scheme_event_emitter](https://github.com/superfreeeee/Blog-code/tree/main/front_end/scheme/scheme_event_emitter)
