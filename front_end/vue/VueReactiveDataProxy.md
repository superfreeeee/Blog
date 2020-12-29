# Vue 源码实现: Reactive Data 响应式对象 Vue3 实现（使用 Proxy 实现）

@[TOC](文章目录)

<!-- TOC -->

- [Vue 源码实现: Reactive Data 响应式对象 Vue3 实现（使用 Proxy 实现）](#vue-源码实现-reactive-data-响应式对象-vue3-实现使用-proxy-实现)
  - [简介](#简介)
  - [参考](#参考)
  - [完整示例代码](#完整示例代码)
- [正文](#正文)
  - [响应式数据对象 Reactive Data](#响应式数据对象-reactive-data)
    - [创建响应式数据对象（reactive 方法）](#创建响应式数据对象reactive-方法)
    - [创建响应式回调函数（effect 方法）](#创建响应式回调函数effect-方法)
  - [代码实现](#代码实现)
    - [实现用例](#实现用例)
    - [项目结构](#项目结构)
    - [交互逻辑](#交互逻辑)
      - [重要对象](#重要对象)
      - [对象间的重要联系](#对象间的重要联系)
    - [详细代码实现](#详细代码实现)
    - [测试结果](#测试结果)
- [结语](#结语)

<!-- /TOC -->

## 简介

Vue3 最近愈来越火，从 Vue2 到 Vue3 最大的改革便是响应式数据的实现方式从原来的 `Object.defineProperty` 改为使用 ES6 新特性 `Proxy` 代理对象来实现。由于响应式数据几乎在整个 MVVM 框架中无处不在，透过改用 Proxy 能大大提升整个框架的运行时性能，同时 Proxy 相对于 Object.defineProperty 提供了更多操作代理方法，能够更全面的发挥`元编程(meta-programming)`的能力。相关代理操作的全面讲解可以查看<a href="https://blog.csdn.net/weixin_44691608/article/details/107025482">ES6特性：Proxy 代理</a>，接下来我们就来自己实现一遍使用 Proxy 的响应式库吧。

## 参考

<table>
  <tr>
    <td>从零实现Vue3的响应式库(1)</td>
    <td><a href="https://segmentfault.com/a/1190000038681994">https://segmentfault.com/a/1190000038681994</a></td>
  </tr>
  <tr>
    <td>Hook 简介-React</td>
    <td><a href="https://react.docschina.org/docs/hooks-intro.html">https://react.docschina.org/docs/hooks-intro.html</a></td>
  </tr>
</table>

## 完整示例代码

<a href="https://github.com/superfreeeee/Blog-code/tree/main/front_end/vue:react:angular/vue_reactive_data_proxy">https://github.com/superfreeeee/Blog-code/tree/main/front_end/vue:react:angular/vue_reactive_data_proxy</a>

# 正文

## 响应式数据对象 Reactive Data

Vue3 在响应式对象的创建上非常接近于 React Hook 的形式，相关可以查看参考链接二。

使用 React Hooks 的形式如下（React 官方示例）：

```js
import React, { useState, useEffect } from 'react';

function Example() {
  // 创建响应式对象
  const [count, setCount] = useState(0);

  // 创建响应式回调
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

而在 Vue3 的使用形式如下：

```js
// 创建响应式对象
const state = reactive({ count: 0 })
// 创建响应式回调
effect(() => {
    document.title = `You clicked ${state.count} times`
})

document.querySelector('button').onclick = () => {
    state.count++
}
```

我们可以发现 Vue3 中的 `reactive`、`effect` 方法几乎就是 React 中的 `useState`、`useEffect` 方法。一个用于创建`响应式数据对象(reactiveObject)`，一个用于创建`响应式回调函数(reactiveEffect)`，接下来我们详细说明两个函数具体需要完成的职责

### 创建响应式数据对象（reactive 方法）

在 Vue3 中我们需要使用 `reactive` 方法传入 data 对象，并返回代理后的响应式对象，如下

```js
const data = reactive({ count: 0 })
```

reactive 方法的职责是创建响应式对象，而所谓响应式对象的具体实现就是透过 `Proxy` 代理来使得对象具备以下特征：

- 数据`访问`操作代理（`getter`）：除了原本的取值操作，需要跟踪数据使用情形（追踪并记录所有相关的响应式回调函数）
- 数据`赋值/删除属性`操作代理（`setter/deleteProperty`）：除了原本的赋值/删除属性操作，还要调用所有相关的响应式回调（effect 回调）

当然 Vue3 或许还代理并追踪了更多东西，但是本篇暂且就只代理这三个操作作为核心

### 创建响应式回调函数（effect 方法）

使用了 `reactive` 创建响应式数据对象之后，我们还可以利用 `effect` 创建响应式回调函数。

```js
effect(() => {
    console.log(`data.count updated: ${data.count}`)
})
```

所谓的响应式回调指的就是当回调函数内使用的任何响应式数据发生改变（`set`、`deleteProperty`）时，在改变完成后会重新执行一次的回调函数，相当于是定义关于数据的`副作用(side-effect)`。

这种回调的意义在于：定义好相应的响应式回调函数之后，当数据发生`改变(update)`时都会自动调用定义好的`副作用操作(effect)`（可能是 dom 操作、计算属性、状态同步等）。相当于一切都自动化了，一切都会跟着数据自动调整自动更新。

## 代码实现

接下来我们就尝试着自己来实现一个响应式库

### 实现用例

首先先看看我们使用响应式对象的形式，也就是最终的测试用例的样子

- `index.js`

```js
import { reactive } from './reactive.js'
import { effect } from './effect.js'

// 创建响应式状态
const state = reactive({ count: 1 })

// 创建副作用，当 state.count 修改时会重新调用
effect(() => {
  console.log(`state.count = ${state.count}`)
})

// 修改 state.count，触发上面的 effect 回调
state.count = 2

// 删除 state.count，一样也会触发回调
delete state.count
```

1. 我们首先创建一个响应式 state 对象
2. 定义一个副作用函数：每次 state.count 更新时就要调用一次（打印新值）
3. 执行赋值操作
4. 执行删除属性操作

这时候我们期望的输出应该是长成下面这样

```
state.count = 1          // 初始化时首次调用
state.count = 2          // state.count = 2 的副作用
state.count = undefined  // delete state.count 的副作用
```

接下来就进入真正的实现环节

### 项目结构

首先向给出完整的项目结构

```
/vue_reactive_data_proxy
├── package.json
├── src
│   ├── effect.js   // 响应式回调相关函数
│   ├── handlers.js // 代理操作定义
│   ├── index.js    // 主要测试入口
│   ├── reactive.js // 响应式对象创建方法
│   └── utils.js    // 工具函数
├── start.js
└── yarn.lock
```

### 交互逻辑

在开始看代码之前我们先来看看整个响应式数据/回调的逻辑中，我们需要创建并保存哪些对象，以及这些对象之间的交互逻辑，这边我们直接以[实现用例](#实现用例)为例进行说明

![](https://picures.oss-cn-beijing.aliyuncs.com/img/vue_reactive_effect_relationship.png)

#### 重要对象

在实现用例的场景下会产生几个重要的数据对象(object)

- 原始数据对象 data：实际存放数据内容的对象
- 代理对象 proxy：响应式数据对象，代理原始对象的操作
- 代理对象映射表 proxyMap：保存`原始对象`到`代理对象`的映射，使用 WeakMap 实现（资源不足时会自动释放不常用的代理对象空间）
- 响应式回调函数集合 Set\<activeEffect\>：响应式对象的 effect 函数集合
- 响应式对象/effect 函数映射表 targetMap：以 `target => key => Set<activeEffect>` 两层 Map 的形式保存不同 `target[key]` 的响应函数集合
- 当前正在运行的回调函数 activeEffect：保存当前正在执行的响应式回调函数

#### 对象间的重要联系

上述的对象透过下列的几个关键交互逻辑联系在一起的

- 代理对象执行 `get` 操作是会`追踪(track 方法)`当前 effect 函数：如果当前 get 方法是由某个 effect 回调函数所触发的，说明该 effect 函数是与当前 `target[key]` 属性相关联的，则将该 effect 函数加入对应的 Set 集合当中
- 代理对象执行 `set` 操作时相当于`更新(update)`该属性值，则需要在修改后需要重新`调用所有相关联的回调函数(activeEffect)(trigger 方法)`
- 代理对象执行 `deleteProperty` 时需要检查属性是否存在以及删除操作是否成功，两个都成功才代表该属性值被成功更新，一样也要`调用关联回调函数(trigger 方法)`

### 详细代码实现

最后给出实际代码实现，相关操作都写在代码注释中了

- `utils.js`：工具函数集合

```js
// object 类型检查
export function isObject (obj) {
  return Object.prototype.toString.call(obj) === '[object Object]'
}
```

- `reactive.js`：响应式对象创建相关函数

```js
import { isObject } from './utils'
import { baseHandlers } from './handlers'

// 保存`原始对象 -> 代理对象`的映射表
const proxyMap = new WeakMap()

/* 响应式对象（Proxy 实现），返回代理对象 */
export function reactive (target) {
  return createReactiveObject(target)
}

/* 创建响应式对象 */
function createReactiveObject (target) {
  // 目标必须是 object 类型
  if (!isObject(target)) return target
  // 同样的目标只需要创建一个代理
  if (proxyMap.has(target)) return proxyMap.get(target)
  
  // 创建代理对象
  const proxy = new Proxy(target, baseHandlers)
  proxyMap.set(target, proxy) // 使用 WeakMap 保存弱引用
  // 返回代理对象
  return proxy
}
```

- `handlers.js`：代理操作定义

```js
import { isObject } from './utils'
import { reactive } from './reactive'
import { track, trigger } from './effect'

/* Proxy 代理方法 */
export const baseHandlers = {
  // 代理属性`访问`操作（例如：state.count）
  get (target, key, receiver) {
    track(target, key) // 追踪相关 effect 回调
    const res = Reflect.get(target, key, receiver)
    if (isObject(res)) {
      // 如果属性值也是对象，则返回响应式对象
      return reactive(res)
    } else {
      return res
    }
  },
  // 代理属性`赋值`操作（例如：state.count = 2）
  set (target, key, value, receiver) {
    const res = Reflect.set(target, key, value, receiver)
    // 赋值操作后触发所有相关 effect 回调
    trigger(target, key, value)
    return res
  },
  // 代理属性`删除`操作（例如：delete state.count）
  deleteProperty (target, key) {
    const hasKey = target.hasOwnProperty(key) // 检查属性是否存在
    const res = Reflect.deleteProperty(target, key) // 删除操作结果
    if (hasKey && res) {
      // 属性存在 且 删除成功才触发回调
      trigger(target, key, undefined)
    }
    return res
  }
}
```

- `effect.js`：响应式回调函数相关

```js
/*
以 target => key => Set<activeEffect> 的形式
保存所有与 target[key] 相关联的 effect 回调
*/
const targetMap = new WeakMap()
let activeEffect

/* 追踪相关 effect 回调 */
export function track (target, key) {
  // 当前并不在任何 effect 回调之内，直接返回
  if (!activeEffect) return

  // 保证 targetMap[target] 存在（为一个 Map<key, Set<activeEffect>>）
  let depsMap = targetMap.get(target)
  if (!depsMap) {
    targetMap.set(target, (depsMap = new Map()))
  }

  // 保证 targetMap[target][key] 存在（为一个 Set<activeEffect>）
  let deps = depsMap.get(key)
  if (!deps) {
    depsMap.set(key, (deps = new Set()))
  }

  // 如果当前 effect 回调不存在，则加入
  if (!deps.has(activeEffect)) {
    deps.add(activeEffect)
  }
}

/* 触发所有相关 effect 回调 */
export function trigger (target, key, newValue) {
  const depsMap = targetMap.get(target)
  if (!depsMap) return

  const effects = depsMap.get(key)
  // 找到 target[key] 相关的所有 effect 回调
  if (effects) {
    // 每个都要调用一次
    effects.forEach(effect => effect())
  }
}

/* 添加 effect 回调 */
export function effect (fn) {
  const effect = createReactiveEffect(fn)
  // 首次调用时直接触发第一次响应式回调
  return effect() // 参考代码这一行写错了，简直天坑hhh
}

// effect 回调调用栈
const effectStack = []

/* 触发所有相关 effect 回调 */
function createReactiveEffect (fn) {
  // 创建响应式回调函数
  const effect = function reactiveEffect () {
    if (!effectStack.includes(effect)) {
      // 防止递归调用，利用调用栈
      try {
        effectStack.push(effect)
        activeEffect = effect // activeEffect 表示当前正在执行的 effect 回调
        return fn() // 实际执行回调
      } finally {
        effectStack.pop()
        activeEffect = effectStack[effectStack.length - 1] //恢复 activeEffect 标志位
      }
    }
  }
  return effect
}
```

- `index.js`：实现用例

```js
import { reactive } from './reactive.js'
import { effect } from './effect.js'

// 创建响应式状态
const state = reactive({ count: 1 })

// 创建副作用，当 state.count 修改时会重新调用
effect(() => {
  console.log(`state.count = ${state.count}`)
})

// 修改 state.count，触发上面的 effect 回调
state.count = 2

// 删除 state.count，一样也会触发回调
delete state.count
```

### 测试结果

```
state.count = 1
state.count = 2
state.count = undefined
```

正确输出三条结果，第一条是第一次调用 `effect` 时会首次执行回调以进行`回调函数关联/追踪(track)`；第二条则是因为 `state.count` 的 set 操作`触发 effect 回调(trigger)`；第三次则是删除属性时引发的 effect 回调

# 结语

Vue3 透过 Proxy 建立的响应式逻辑相对于 Vue2 使用的 Object.defineProperty 更加简便，Proxy 提供的操作代理选项非常清楚的体现出透过`介入原生行为(元编程，即代理操作)`，MVVM 能够轻易达成基于数据的自动驱动过程，进行数据的更新、同步、DOM 操作等，也将程序员从编程式的数据更新通知的问题上解放出来，提供近似与声明式的编程体验（透过声明副作用 effect 达成基于数据驱动的自动状态更新）。
