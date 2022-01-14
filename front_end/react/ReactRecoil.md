# React 状态管理: Recoil - Facebook 状态管理

@[TOC](文章目录)

<!-- TOC -->

- [React 状态管理: Recoil - Facebook 状态管理](#react-状态管理-recoil---facebook-状态管理)
- [Recoil 概念](#recoil-概念)
- [Recoil 示例](#recoil-示例)
  - [0. RecoilRoot](#0-recoilroot)
  - [1. Atom 状态](#1-atom-状态)
  - [2. Selector 导出状态](#2-selector-导出状态)
  - [3. Async Selector 异步导出状态](#3-async-selector-异步导出状态)
  - [4. Atom Family 状态类](#4-atom-family-状态类)
  - [5. Selector Family 导出状态类](#5-selector-family-导出状态类)
  - [6. RecoilBridge: 跨 Root 通信](#6-recoilbridge-跨-root-通信)
- [小结](#小结)
- [参考连接](#参考连接)
- [完整代码示例](#完整代码示例)

<!-- /TOC -->

# Recoil 概念

第一次尝试 Recoil 让我非常惊艳，以前我都是用 redux 进行状态管理，但是多余的重新渲染总是让人不胜其扰，Recoil 可以说是彻底解决 redux 基于 Context API 进行状态管理的弊端，并借助了 React hooks 的优势，将状态更新推迟到一个个最接近底部的子节点上。

上一张图大家就能够感受到差异了（借用 [Dave McCabe 大佬](https://www.youtube.com/watch?v=_ISAA_Jt9kI)的图）

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_recoil_1_concept.png)

相当于是把每一个状态设计成一个单独的可订阅对象，然后使用 hooks 进行订阅最终状态更新就只会修改直接使用该状态的组件，同时避免了 Redux 将多个状态都堆放在同一个对象上造成使用同一个 Context 的组件触发多余的渲染

# Recoil 示例

下面我们透过实际操作来体验一下 Recoil 的便捷

## 0. RecoilRoot

下面 1-5 的例子我们都统一使用单一个 `RecoilRoot` 作为全局作用域，不论是 atom 还是 selector 都是基于 `RecoilRoot` 来获取当前值，对于不同 `RecoilRoot` 则会产生不同的状态实例，切记。

## 1. Atom 状态

第一个概念是状态，Recoil 定义了一个 `atom` 函数来声明一个状态，使用如下

- `/src/state/states.ts`

```ts
export const counterState = atom({
  key: 'counter',
  default: 0,
});
```

接下来在组件内我们就可以像原来使用 `useState` 一样来使用这个状态，只需要简单改成使用 `useRecoilState` 即可

`useRecoilState` 与 `useState` 类似，会返回一个 value 与一个 setter，Recoil 更进一步提供我们只需要其中一个的时候的钩子 `useRecoilValue` 与 `useSetRecoilState`

第一个组件我们使用 `useRecoilValue` 来获取状态

- `/src/components/Counter.tsx`

```ts
import React from 'react';
import { useRecoilValue } from 'recoil';

import { counterState } from '@/state/states';

const Counter = () => {
  const count = useRecoilValue(counterState);

  return (
    <>
      <h2>Atom - Counter</h2>
      <div>
        <h3>Counter in ComponentA</h3>
        <div>count: {count}</div>
      </div>
    </>
  );
};

export default Counter;
```

第二个组件我们则是透过 button 提供修改的入口

- `/src/components/CounterController.tsx`

```ts
const CounterController = () => {
  const [count, setCount] = useRecoilState(counterState);

  const increment = () => {
    setCount(count + 1);
  };

  const reset = useResetRecoilState(counterState);

  const setCount2 = useSetRecoilState(counterState);
  const addTen = () => setCount2(count + 10);

  return (
    <CounterControllerWrapper>
      <h3>Counter controller</h3>
      <div>
        <button onClick={increment}>+1</button>
        <button onClick={addTen}>+10</button>
        <button onClick={reset}>Reset</button>
      </div>
    </CounterControllerWrapper>
  );
};

export default CounterController;
```

我们可以看到，`useRecoilState` 的 setter 与 `useSetRecoilState` 返回的 setter 是等价的

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_recoil_2_atom.png)

## 2. Selector 导出状态

第二个概念非常有趣，与 vue 的 computed 非常相似，甚至还能透过 set 反过来更新依赖状态

首先我们先定义一个摄氏温度做为原始状态

- `/src/state/states.ts`

```ts
export const celsiusState = atom({
  key: 'celsius',
  default: 32,
});
```

接下来定义华氏温度与摄氏温度同步

- `/src/state/selectors.ts`

```ts
export const fahrenheitState = selector<number>({
  key: 'fahrenheit',
  get: ({ get }) => (get(celsiusState) * 9) / 5 + 32,
  set: ({ set }, newValue: number) =>
    set(celsiusState, ((newValue - 32) * 5) / 9),
});
```

get 方法透过内部的钩子函数来获取摄氏温度的状态，同时可选的 set 方法还能够反过来更新摄氏温度的状态

不管是 atom 还是 selector 其实都是返回一个 RecoilState 对象，所以在使用上是一致的

- `/src/components/Temperature.tsx`

```ts
const Temperature = () => {
  const [celsius, setCelsius] = useRecoilState(celsiusState);
  const [fahrenheit, setFahrenheit] = useRecoilState(fahrenheitState);

  const onCelsiusChange = (e) => {
    const val = Number(e.target.value);
    console.log(`new celsius ${val}`);
    setCelsius(val);
  };
  const onFahrenheitChange = (e) => {
    const val = Number(e.target.value);
    console.log(`new fahrenheit ${val}`);
    setFahrenheit(val);
  };

  return (
    <TemperatureWrapper>
      <h2>Selector - Temparature</h2>
      <div
        style={{
          display: 'flex',
          justifyContent: 'center',
          alignItems: 'center',
        }}
      >
        <label>
          <div>celsius</div>
          <input type="text" value={celsius} onChange={onCelsiusChange} />
        </label>{' '}
        ={' '}
        <label>
          <div>fahrenheit</div>
          <input type="text" value={fahrenheit} onChange={onFahrenheitChange} />
        </label>
      </div>
    </TemperatureWrapper>
  );
};

export default Temperature;
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_recoil_3_selectors.png)

不论输入哪一侧都能够同步两侧的温度

## 3. Async Selector 异步导出状态

透过上述的 selector 发现，他可以组合、选择、过滤任意多个状态进行导出，甚至作者还提供了异步的方式，让我们能够产生异步的状态。而这个异步状态则是被设计与 `Suspense` 协同工作的

- `/src/state/selectors.ts`

```ts
export interface BusinessCard {
  name: string;
  message: string;
}

export const businessCardState = selector<BusinessCard>({
  key: 'businessCard',
  get: async ({ get }) => {
    const [res] = await Promise.all([
      fetch('/businessCard.json'),
      new Promise((resolve) => setTimeout(resolve, 2000)),
    ]);
    return res.json();
  }
});
```

这里我们加上了一个 2s 的延迟，确保组件不会产生闪烁

- `/src/components/AsyncSelector.tsx`

使用的时候则需要透过组件将异步 selector 进行隔离

```ts
const BusinessCard = () => {
  const businessCard = useRecoilValue(businessCardState);

  return <div>data: {JSON.stringify(businessCard)}</div>;
};

const AsyncSelector = () => {
  return (
    <div>
      <h2>Async Selector</h2>
      <Suspense fallback={<div>Loading business card ...</div>}>
        <BusinessCard />
      </Suspense>
    </div>
  );
};
```

效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_recoil_4_async.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_recoil_5_async2.png)

感觉前面这三种已经足够满足大部分的使用场景了，同时也设计的足够简单，几乎可以一些现有业务进行无痛迁移（如果是使用 redux 那很不好意思了hhh）

## 4. Atom Family 状态类

下面更是出现所谓的 family 类型，使用下来的感觉就像是定义一个 atom 类型，然后根据参数来获取对应的状态。有点像是当你需要多个相同 atom 的时候，把它变成一个保存了 `key:value` 的对象，而使用时则是调用并传入对应的 key，返回 atom 实例的感觉

下面我们一样透过计数器来举例，只是这次我们将一次创造多个计数器

- `/src/state/states.ts`

```ts
export const counterFamily = atomFamily({
  key: 'counterFamily',
  default: 0,
});
```

使用的时候则是调用 `counterFamily(key)` 来获取具体的 atom 实例

- `/src/components/CounterItem.tsx`

```ts
const CounterItem: FC<CounterItemProps> = ({ id }) => {
  const counterState = counterFamily(id);
  const [count, setCount] = useRecoilState(counterState);
  const resetCount = useResetRecoilState(counterState);

  return (
    <div>
      <h2>Atom Family - Counter Item({id})</h2>
      <div>count: {count}</div>
      <div>
        <button onClick={() => setCount(count + 1)}>+1</button>
        <button onClick={() => setCount(count + 10)}>+10</button>
        <button onClick={resetCount}>reset</button>
      </div>
    </div>
  );
};
```

然后我们一次创建多个对象

- `/src/App.tsx`

```ts
{Array.from({ length: 3 }, (_, i) => (
  <CounterItem key={i} id={i} />
))}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_recoil_6_atom_family.png)

我们可以看到，三个组件都使用了相同的 atomFamily，但是基于不同的 id 产生了不同的状态实例

## 5. Selector Family 导出状态类

family 对于 selector 也是类似的，不同的 key 映射到不同的 selector 实例

首先定义基础状态

- `/src/state/states.ts`

```ts
export const baseNumberState = atom({
  key: 'baseNumberState',
  default: 1,
});
```

然后定义公用的导出状态类型

- `/src/state/selectors.ts`

```ts
export const multipliedNumberFamily = selectorFamily<number, number>({
  key: 'multipliedNumber',
  get:
    (multiplier: number) =>
    ({ get }) =>
      get(baseNumberState) * multiplier,
  set:
    (multiplier: number) =>
    ({ set }, newValue: number) =>
      set(baseNumberState, newValue / multiplier),
});
```

这里我们可以提供任意倍数来获取不同的导出状态量

- `/src/components/MultipliedSelector.tsx`

```ts
const MultipliedSelector = () => {
  const base = useRecoilValue(baseNumberState);
  const [tenTimes, setTenTimes] = useRecoilState(multipliedNumberFamily(10));
  const [hundredTimes, setHundredTimes] = useRecoilState(
    multipliedNumberFamily(100)
  );

  return (
    <div>
      <h2>Selector Family - Multiplier</h2>
      <div>base: {base}</div>
      <div>
        <h3>Multiplier - x10</h3>
        <input
          type="text"
          value={tenTimes}
          onChange={(e) => setTenTimes(Number(e.target.value))}
        />
      </div>
      <div>
        <h3>Multiplier - x100</h3>
        <input
          type="text"
          value={hundredTimes}
          onChange={(e) => setHundredTimes(Number(e.target.value))}
        />
      </div>
    </div>
  );
};
```

这里我们使用了一个 10 倍、一个 100 倍的导出状态，两个状态都是基于相同的 baseNumber 状态，因此下面不论修改哪个状态，三个数字都会同步保持倍数关系

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_recoil_7_selector_family.png)

## 6. RecoilBridge: 跨 Root 通信

最后我们回过头来谈谈 RecoilRoot 的问题，由于 atom 和 selector 都是基于 RecoilRoot 来保存状态，但是嵌套的 RecoilRoot 又会遮蔽掉外层的 RecoilRoot，造成我们非常难以划分 RecoilRoot 的界限，最后被迫只能全部写在同一个 RecoilRoot 之下。

未来或许可以透过为 RecoilRoot 附加 ignoreState 的方式来支持内部作用域访问外部 RecoilRoot 的方式。

官方目前则是透过提供 `useRecoilBridgeAcrossReactRoots` 的方式来支持跨 Root 访问

- `/src/components/NestedRoot.tsx`

首先我们先定义一个最外层的 RecoilRoot

```ts
const NestedRoot: FC = () => {
  return (
    <RecoilRoot>
      <NestedRootWrapper>
        <Outer />
      </NestedRootWrapper>
    </RecoilRoot>
  );
};
```

然后 Outer 组件内第一次访问状态

```ts
const useRandomNumber = (): [number, () => void] => {
  const [num, setNum] = useRecoilState(randomNumberState);
  const change = () => setNum(Math.random());

  return [num, change];
};

const Outer = () => {
  const [num, change] = useRandomNumber();
  const RecoilBridge = useRecoilBridgeAcrossReactRoots_UNSTABLE();

  return (
    <div className="outer">
      <h2>Outer State: {num}</h2>
      <button onClick={change}>change</button>
      <RecoilRoot>
        <Inner>
          <RecoilBridge>
            <Inner inBridge />
          </RecoilBridge>
        </Inner>
      </RecoilRoot>
    </div>
  );
};
```

并透过 `useRecoilBridgeAcrossReactRoots_UNSTABLE` 钩子来将当前的 RecoilRoot 保存下来。后面我们使用两次 `Inner` 组件，内部的 Inner 使用 `RecoilBridge` 包裹

```ts
const Inner: FC<{ inBridge?: boolean }> = ({ inBridge, children }) => {
  const [num, change] = useRandomNumber();

  return (
    <div className={inBridge ? 'bridge' : 'inner'}>
      <h2>Inner State: {num}</h2>
      <button onClick={change}>change</button>
      {children}
    </div>
  );
};
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_recoil_8_nested.png)

就能看到最内层的 Inner 与 Outer 所在的 Root 是相同的状态，而中间的 Inner 则是一个独立的作用域

# 小结

Recoil 作为继 Redux、MobX 之后的一个全新的状态管理库，与前两种的状态管理理念从根本上的不同，感觉还是与 React Hooks 非常搭的，同时提供 Async Selector 为我们开启了全新的概念，将异步远程数据纳入状态管理的范畴。

不过作为一个比较新的库还是存在许多不足，版本上也是还没完全稳定 API 的 0.x.x 版本

---

# 参考连接

| Title                                                                                      | Link                                                                                       |
| ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------ |
| Recoil                                                                                     | [https://recoiljs.org/](https://recoiljs.org/)                                             |
| Recoil: State Management for Today's React - Dave McCabe aka @mcc_abe at @ReactEurope 2020 | [https://www.youtube.com/watch?v=_ISAA_Jt9kI](https://www.youtube.com/watch?v=_ISAA_Jt9kI) |
| 关于Recoil的atom跨RecoilRoot交互的二三事                                                   | [https://juejin.cn/post/7004754412501467173](https://juejin.cn/post/7004754412501467173)   |

# 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_recoil](https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_recoil)
