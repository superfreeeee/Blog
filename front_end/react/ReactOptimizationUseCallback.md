# React 优化: 到底怎么用 useCallback 才是正确的？

@[TOC](文章目录)

<!-- TOC -->

- [React 优化: 到底怎么用 useCallback 才是正确的？](#react-优化-到底怎么用-usecallback-才是正确的)
- [前言](#前言)
- [正文](#正文)
  - [0. 环境准备](#0-环境准备)
  - [1. 普通场景](#1-普通场景)
  - [2. 存在子组件](#2-存在子组件)
  - [3. 没有子组件的情况下使用 useCallback](#3-没有子组件的情况下使用-usecallback)
  - [4. 存在子组件的情况下使用 useCallback](#4-存在子组件的情况下使用-usecallback)
  - [React 组件概念 & 组件更新机制梳理](#react-组件概念--组件更新机制梳理)
  - [5. 使用 React.memo 与 useCallback 协同](#5-使用-reactmemo-与-usecallback-协同)
  - [6. 使用 useMemo 与 useCallback 协同](#6-使用-usememo-与-usecallback-协同)
  - [useCallback 与 useMemo 的好处](#usecallback-与-usememo-的好处)
  - [7. 总结 & 使用时机](#7-总结--使用时机)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

之前在 [React Hook: 高级 Hook API](https://blog.csdn.net/weixin_44691608/article/details/118719312) 中跟大家提过诸如 useCallback、useMemo 等钩子，其实他与原来 Class 组件时用的 React.PureComponent、shouldComponentUpdate、React.memo 都是密切相关的。

本篇我们就从 useCallback 的使用角度入手，看看到底我们套了这一层钩子能有什么作用，到底什么时候要套这一层钩子？

# 正文

## 0. 环境准备

首先我们先准备一下实验环境，我们需要几个钩子和一个用于检查的方法

- `/src/hooks/useInputs.ts`

首先是两个版本的 useInput 钩子，负责对应于一个 `<input>` 标签

```ts
/**
 * 基本用法
 * @returns
 */
const useInput = (): [
  string,
  ChangeEventHandler<HTMLInputElement>
] => {
  const [value, setValue] = useState('');

  const handleChange = (e: ChangeEvent<HTMLInputElement>) =>
    setValue(e.target.value);

  return [value, handleChange];
};
```

```ts
/**
 * 带 useCallback 用法
 * @returns
 */
const useInputWithCallback = (): [
  string,
  ChangeEventHandler<HTMLInputElement>
] => {
  const [value, setValue] = useState('');

  const handleChange = useCallback(
    (e: ChangeEvent<HTMLInputElement>) => setValue(e.target.value),
    []
  );

  return [value, handleChange];
};
```

- `/src/utils/record.ts`

接下来我们需要一个特别的函数，来检查组件的更新以及内部函数的替换与否

```ts
const funcsMap = new Map<string, Function[]>();

export const rec = (tag: string = 'default', fn: Function) => {
  if (!funcsMap.has(tag)) {
    funcsMap.set(tag, []);
  }
  const funcs = funcsMap.get(tag);
  if (funcs.includes(fn)) {
    console.log(`[${tag}] fn exists`);
  } else {
    funcs.push(fn);
    console.log(`[${tag}] funcs:`, funcs);
  }
};
```

之后我们在组件内部每次调用 `rec` 都会输出一次记录值(fn)是否存在

下面我们就可以开始实验场景了

## 1. 普通场景

- `/src/test/Test1.tsx`

第一个场景非常简单，就是啥也没用，只用了最基本的 useState 钩子来完成 input 事件

```ts
import React, { useEffect } from 'react';
import { useInput } from '../hooks/useInputs';
import { rec } from '../utils/record';

const Test1 = () => {
  console.group('render Test1');

  const [name, handleNameChange] = useInput();

  rec('Test1', handleNameChange);

  console.groupEnd();
  return (
    <div>
      <h2>Test 1: 测试基本情况(简单 inline 函数)</h2>
      <h3>name: {name}</h3>
      <input type="text" value={name} onChange={handleNameChange} />
    </div>
  );
};

export default Test1;
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_optimization_useCallback_test1.gif)

看起来没啥不对劲的，输入一次就要更新一次组件，所以就会调用一次方法嘛，挺正常的。我们接着看下去

## 2. 存在子组件

- `/src/test/Test2.tsx`

第二个场景比较特别了，现在我们在组件内部插入一个子组件

```ts
const Inner = (props) => {
  console.log('render Test2 Inner');
  return <input type="text" onChange={props.onChange} />;
};
```

然后原来的组件照旧

```ts
import React from 'react';
import { useInput } from '../hooks/useInputs';
import { rec } from '../utils/record';

const Test2 = () => {
  console.group('render Test2');

  const [name, handleNameChange] = useInput();

  rec('Test2', handleNameChange);

  console.groupEnd();
  return (
    <div>
      <h2>Test 2: 基本情况 & 子组件</h2>
      <h3>name: {name}</h3>
      <Inner onChange={handleNameChange} />
    </div>
  );
};

export default Test2;
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_optimization_useCallback_test2.gif)

看起来也没啥大问题。React 本身就是一个单向数据流，所以 Test2 更新 Inner 也跟着更新也是理所当然的。

到了这里你可能就会想到那是不是就可以用上 `useCallback` 了呢？

## 3. 没有子组件的情况下使用 useCallback

- `/src/test/Test3.tsx`

第三个实验我们先稍微回退一下，回到没有子组件的时候，我们换成用了 useCallback 版本的 useInput

```ts
import React from 'react';
import { useInputWithCallback } from '../hooks/useInputs';
import { rec } from '../utils/record';

const Test3 = () => {
  console.group('render Test3');

  const [name, handleNameChange] = useInputWithCallback();

  rec('Test3', handleNameChange);

  console.groupEnd();
  return (
    <div>
      <h2>Test 3: 使用 useCallback</h2>
      <h3>name: {name}</h3>
      <input type="text" value={name} onChange={handleNameChange} />
    </div>
  );
};

export default Test3;
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_optimization_useCallback_test3.gif)

看起来也跟第一次实验没什么区别嘛，但是细看输出我们会发现，原本实验一每次更新之后给到 rec 函数的都是新的函数，而这次用了 useCallback 之后，就变成都是同一个了，useCallback 就好像记忆了上次更新的那个函数，挪到下次更新的时候进行复用

下面我们看看这个概念到底能起到什么作用

## 4. 存在子组件的情况下使用 useCallback

- `/src/test/Test4.tsx`

既然上面看到 useCallback 能帮我们记住前一次组件更新时的函数，那是不是再加上子组件他就不会更新了呢？

```ts
import React from 'react';
import { useInputWithCallback } from '../hooks/useInputs';
import { rec } from '../utils/record';

const Inner = (props) => {
  console.log('render Test4 Inner');
  return <input type="text" onChange={props.onChange} />;
};

const Test4 = () => {
  console.group('render Test4');

  const [name, handleNameChange] = useInputWithCallback();

  rec('Test4', handleNameChange);

  console.groupEnd();
  return (
    <div>
      <h2>Test 4: useCallback 与子组件</h2>
      <h3>name: {name}</h3>
      <Inner onChange={handleNameChange} />
    </div>
  );
};

export default Test4;
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_optimization_useCallback_test4.gif)

结果事与愿违，子组件还是非常勤奋的每次都更新一次

## React 组件概念 & 组件更新机制梳理

- 组件与实例方法

这边就要提到 React 组件的更新机制了。我们知道 React 组件分成使用 Class 声明的类组件，以及使用 function 声明的函数组件。

对于类组件来说，所有方法都是绑定在一个叫做组件实例的对象上，所以每次组件更新的时候都是传入同一个函数实例；而对于函数组件来说就需要使用 `useCallback` 方法来确保传入同一个函数实例

- 组件的更新

不论是哪种组件的更新时机都是 props 与 state 的改变。默认情况下只要重新传递 props 或是 state 改变，组件就会直接更新。如果使用了 `React.pureComponent` 则会对 props 进行浅比较，选择性的更新；更新一步我们还可以使用类组件的 `shouldComponentUpdate` 钩子来自定义组件更新的时机。

而对于函数组件来说，组件本身就是一个 render 方法，也没有什么钩子方法，这时候我们就可以使用 `React.memo` 来实现类似的功能。

## 5. 使用 React.memo 与 useCallback 协同

- `/src/test/Test5.tsx`

了解上述更新机制之后，我们就能够借由 `useCallback` 来保留相同的函数，在传入 `React.memo` 包含的组件时实现子组件直接复用而不更新的功能咯

```ts
import React, { ChangeEventHandler } from 'react';
import { useInputWithCallback } from '../hooks/useInputs';
import { rec } from '../utils/record';

interface IInnerProps {
  onChange: ChangeEventHandler<HTMLInputElement>;
}

const Inner = React.memo<IInnerProps>((props) => {
  console.log('render Test5 Inner');
  return <input type="text" onChange={props.onChange} />;
});

const Test5 = () => {
  console.group('render Test5');

  const [name, handleNameChange] = useInputWithCallback();

  rec('Test5', handleNameChange);

  console.groupEnd();
  return (
    <div>
      <h2>Test 5: useCallback 与 React.memo 协同</h2>
      <h3>name: {name}</h3>
      <Inner onChange={handleNameChange} />
    </div>
  );
};

export default Test5;
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_optimization_useCallback_test5.gif)

## 6. 使用 useMemo 与 useCallback 协同

- `/src/test/Test6.tsx`

最后一个场景也很简单，就是使用 `useMemo` 来替换 `React.memo` 的用法

```ts
import React, { ChangeEventHandler, useMemo } from 'react';
import { useInputWithCallback } from '../hooks/useInputs';
import { rec } from '../utils/record';

interface IInnerProps {
  onChange: ChangeEventHandler<HTMLInputElement>;
}

const Inner = (props: IInnerProps) => {
  console.log('render Test6 Inner');
  return <input type="text" onChange={props.onChange} />;
};

const Test6 = () => {
  console.group('render Test6');

  const [name, handleNameChange] = useInputWithCallback();

  rec('Test6', handleNameChange);

  console.groupEnd();
  return (
    <div>
      <h2>Test 6: useMemo 实现局部刷新</h2>
      <h3>name: {name}</h3>
      {useMemo(
        () => (
          <Inner onChange={handleNameChange} />
        ),
        [handleNameChange]
      )}
    </div>
  );
};

export default Test6;
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_optimization_useCallback_test6.gif)

## useCallback 与 useMemo 的好处

由上述例子我们大概能够知道两个钩子的用法，下面说说这两个钩子的意义

- useCallback

    useCallback 本身可以看作类组件实例方法的替代方案，
    
    同时他又给我们开启一个视角：在调用前就能够绑定好变量(犹如闭包一般)，并且在依赖数据不改变的情况下保留相同的函数实例

- useMemo

    useMemo 可以分为两个角度来说明

    一方面可以将其看成就是 `React.memo` 的替代品，同时可以对任意大小的 JSX 片段进行缓存

    另一方面其实它让我们能够对组件内部的各个元素进行更细粒度的控制，让我们能够不只是利用 `React.memo` 粗暴的对整个组件进行记忆，而可以针对特定片段进行缓存与复用

## 7. 总结 & 使用时机

最后我们总结呼应一下标题：

`useCallback` 的作用在于保留相同的函数示例，其本身并不干涉组件更新的机制；与 `React.memo` 或是 `useMemo` 等方法联用时，才会获得保留相同组件示例的传入使局部片段能够进行复用而不会完全更新的优化好处。

# 结语

本篇透过针对 useCallback 的用法和各种场景进行探讨，尝试归纳出 useCallback 的效果和使用时机，供大家参考

# 其他资源

## 参考连接

| Title                                | Link                                                                                                                                   |
| ------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| useCallback - React官方文档          | [https://zh-hans.reactjs.org/docs/hooks-reference.html#usecallback](https://zh-hans.reactjs.org/docs/hooks-reference.html#usecallback) |
| useMemo - React官方文档              | [https://zh-hans.reactjs.org/docs/hooks-reference.html#usememo](https://zh-hans.reactjs.org/docs/hooks-reference.html#usememo)         |
| React Hooks 第一期：聊聊 useCallback | [https://zhuanlan.zhihu.com/p/56975681](https://zhuanlan.zhihu.com/p/56975681)                                                         |
| TypeScript and React: Events         | [https://fettblog.eu/typescript-react/events/](https://fettblog.eu/typescript-react/events/)                                           |
| React.memo 与 useMemo                | [https://zhuanlan.zhihu.com/p/105940433](https://zhuanlan.zhihu.com/p/105940433)                                                       |
| React.Component - React官方文档      | [https://zh-hans.reactjs.org/docs/react-component.html](https://zh-hans.reactjs.org/docs/react-component.html)                         |


## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_optimization_useCallback](https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_optimization_useCallback)
