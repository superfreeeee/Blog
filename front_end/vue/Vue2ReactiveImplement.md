# Vue 源码解析: MVVM 双向绑定1 - 响应式原理(数据观测/响应)

@[TOC](文章目录)

<!-- TOC -->

- [Vue 源码解析: MVVM 双向绑定1 - 响应式原理(数据观测/响应)](#vue-源码解析-mvvm-双向绑定1---响应式原理数据观测响应)
- [前言](#前言)
  - [深入 MVVM](#深入-mvvm)
  - [细化 ViewModel](#细化-viewmodel)
  - [MVVM 实现三阶段](#mvvm-实现三阶段)
- [正文](#正文)
  - [回顾：观察者模式](#回顾观察者模式)
  - [1. 数据观测](#1-数据观测)
    - [1.1 被观测数据代理 Observer](#11-被观测数据代理-observer)
      - [1.1.1 观测对象 Object](#111-观测对象-object)
      - [1.1.2 观测数组 Array](#112-观测数组-array)
      - [1.1.3 代理数组方法 arrayMethods](#113-代理数组方法-arraymethods)
    - [1.2 数据观测 observe](#12-数据观测-observe)
    - [1.3 转化为响应式对象 defineReactive](#13-转化为响应式对象-definereactive)
      - [1.3.1 访问数据 Object.defineProperty.getter](#131-访问数据-objectdefinepropertygetter)
      - [1.3.2 更新数据 Object.defineProperty.setter](#132-更新数据-objectdefinepropertysetter)
    - [1.4 数据观测小结](#14-数据观测小结)
  - [2. 依赖收集](#2-依赖收集)
    - [2.1 依赖管理器 Dep](#21-依赖管理器-dep)
    - [2.2 观察者对象 Watcher](#22-观察者对象-watcher)
      - [2.2.1 构造函数 constructor](#221-构造函数-constructor)
      - [2.2.2 获取观察值并绑定依赖关系 get](#222-获取观察值并绑定依赖关系-get)
    - [2.3 收集依赖](#23-收集依赖)
    - [2.4 响应式原理完整流程总结](#24-响应式原理完整流程总结)
      - [2.4.1 观察数据示例：initData](#241-观察数据示例initdata)
  - [3. 实现细节补充 & 弥补不足之处](#3-实现细节补充--弥补不足之处)
    - [3.1 观察者如何注册并管理依赖？](#31-观察者如何注册并管理依赖)
      - [3.1.1 图解 Watcher.get 方法](#311-图解-watcherget-方法)
    - [3.2 Vue 实例方法补充](#32-vue-实例方法补充)
      - [3.2.1 Vue.prototype.$set](#321-vueprototypeset)
      - [3.2.2 Vue.prototype.$del](#322-vueprototypedel)
      - [3.2.3 Vue.prototype.$watch](#323-vueprototypewatch)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 前言

前一篇讲了 <a href="https://blog.csdn.net/weixin_44691608/article/details/116576241">Vue 源码解析: 生命周期钩子全面剖析</a>，按 Vue 的生命周期顺序过了一遍整个 Vue 实例的行为和顺序，接下来我们就来深入 Vue 框架内部 MVVM 实现原理的各个阶段。

## 深入 MVVM

在开始正文之前，我们先来回顾一下 MVVM 架构的抽象逻辑

![](https://picures.oss-cn-beijing.aliyuncs.com/img/vue2_reactive_implement_mvvm.png)

ViewModel 同时担任了 **维护数据** 和 **选择并渲染视图** 的角色，也就是完成 $Model \leftrightarrows View$ 的双向绑定任务。那么在 Vue 框架之下又是如何细化并实现所谓的 ViewModel 呢？

## 细化 ViewModel

![](https://picures.oss-cn-beijing.aliyuncs.com/img/vue2_reactive_implement_mvvm2.png)

第一阶段我们可以先将整个 ViewModel 分成两个部分：`Reactive Proxy` 和 `Render Proxy`。`Reactive Proxy` 相当于是我们的 **响应式数据**，其实可以视为原始数据的一个代理，为原始的数据对象套上一层 **观察者模式** 的壳，而 `Render Proxy` 则是负责进行模版渲染的代理对象，利用 `Reactive Proxy` 提供的数据对象生成最终视图。

## MVVM 实现三阶段

接下来会将整个 MVVM 的实现流程分成三个阶段来解说：

1. 响应式原理
2. 虚拟 DOM 原理
3. 模版编译/渲染原理

本篇先来介绍第一阶段：**响应式原理**(解析版本：Vue-2.6.12)

# 正文

## 回顾：观察者模式

![](https://picures.oss-cn-beijing.aliyuncs.com/img/observer_pattern_structure.png)

观察者模式的核心角色就是两个：

- **Subject 观察目标(被观察对象)**
- **Watcher(Observer) 观察者**

而按应用于 Vue 的 MVVM 实现来说，**数据(Data)** 就是我们的观察目标，而我们会另外创建不同的 **观察者(Watcher)** 来使用和响应数据的变化(组件渲染 render、计算属性 computed、观察属性 watch 等都需要建立观察者对象)

接下来我们就来看看 Vue 是如何应用观察者模式来实现响应式数据绑定的。

## 1. 数据观测

实现响应式数据绑定的第一步要先能够观测数据。

### 1.1 被观测数据代理 Observer

首先我们先定义一个类 **Observer**，它的意义一个是 **深度观察** 数据对象(or 数组)；第二个则是起到一个 **标志** 的作用，我们会将 Observer 对象绑定到观测目标的 `__ob__` 属性上，看源码定义：

- `/src/core/observer/index.js`(阅读笔记文件路径：`/src/core/observer/index/Observer.js`)

```js
// 响应式对象代理

export class Observer {
  value: any;
  dep: Dep;
  vmCount: number; // number of vms that have this object as root $data

  constructor (value: any) {
    this.value = value
    this.dep = new Dep()
    this.vmCount = 0
    /* 将 Observer 对象绑定到 __ob__ 属性上 */
    def(value, '__ob__', this)

    if (Array.isArray(value)) {
      /* 数组则先代理数组基本方法 */
      if (hasProto) {
        protoAugment(value, arrayMethods)
      } else {
        copyAugment(value, arrayMethods, arrayKeys)
      }
      /* 后遍历观察数组内元素 */
      this.observeArray(value)
    } else {
      /* 深度遍历对象属性并转为响应式数据 */
      this.walk(value)
    }
  }

  // 遍历 "对象属性" 并转变为响应式数据
  walk (obj: Object) {
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i])
    }
  }

  // 遍历 "数组元素" 并观察
  observeArray (items: Array<any>) {
    for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i])
    }
  }
}
```

`value` 属性保存观测目标的原始对象；`dep` 为依赖管理器，后面会在提到；而 `vmCount` 则是记录有多少个 Vue 实例以此数据对象作为 data 的根对象。

注意：能被观测的对象类型只有 **Object** 和 **Array** 两种，其他基本类型(如 Number、String、Boolean)是不能够被观测的

```js
def(value, '__ob__', this)
```

这句话表示将此 Observer 实例绑定到观测目标的 `__ob__` 属性上

#### 1.1.1 观测对象 Object

第一种情形是我们观测的目标是一个 **Object**：

```js
if (Array.isArray(value)) {
  /* when value is array */
} else {
  /* 深度遍历对象属性并转为响应式数据 */
  this.walk(value)
}

// ...

// 遍历 "对象属性" 并转变为响应式数据
walk (obj: Object) {
  const keys = Object.keys(obj)
  for (let i = 0; i < keys.length; i++) {
    defineReactive(obj, keys[i])
  }
}
```

这时候会调用 `walk` 方法，即遍历 value 的所有 key 并调用 `defineReactive` 将该 key 的访问转变为响应式数据，这个方法下面会再提到

#### 1.1.2 观测数组 Array

第二种情形，我们的观测目标是一个 **Array**：

```js
if (Array.isArray(value)) {
  /* 数组则先代理数组基本方法 */
  if (hasProto) {
    protoAugment(value, arrayMethods)
  } else {
    copyAugment(value, arrayMethods, arrayKeys)
  }
  /* 后遍历观察数组内元素 */
  this.observeArray(value)
} else {
  /* when value is object */
}
```

我们可以看到在观测目标是数组的情况下，会先用新的数组方法 `arrayMethods` 替换/代理原始的数组方法，这里有两种策略：

- 策略一：在支持 `__proto__` 属性的环境内，直接以新的数组方法 `arrayMethods` 替换观测目标的原型

```js
protoAugment(value, arrayMethods)

// 重置原型对象(重新赋值 __proto__ 属性)
function protoAugment (target, src: Object) {
  /* eslint-disable no-proto */
  target.__proto__ = src
  /* eslint-enable no-proto */
}
```

- 策略二：不支持 `__protot__` 属性的环境下，则需要一个个的将新的数组方法写到数组对象中

```js
copyAugment(value, arrayMethods, arrayKeys)

// 覆盖数组原方法
function copyAugment (target: Object, src: Object, keys: Array<string>) {
  for (let i = 0, l = keys.length; i < l; i++) {
    const key = keys[i]
    def(target, key, src[key])
  }
}
```

将数组方法都代理好了之后，再调用 `observeArray` 深度遍历数组对象

```js
this.observeArray(value)

// 遍历 "数组元素" 并观察
observeArray (items: Array<any>) {
  for (let i = 0, l = items.length; i < l; i++) {
    observe(items[i])
  }
}
```

#### 1.1.3 代理数组方法 arrayMethods

这边插播一段，上面对于数组对象的操作代理用到了 `arrayMethods`，而这个 arrayMethods 则是这么来的

- `/src/core/observer/array.js`(阅读笔记文件路径：`/src/core/observer/array.js`)

```js
import { def } from '../util/index'

const arrayProto = Array.prototype
export const arrayMethods = Object.create(arrayProto)

const methodsToPatch = [
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
]

/* 代理数组对象 */
methodsToPatch.forEach(function (method) {
  // 原始数组方法
  const original = arrayProto[method]
  def(arrayMethods, method, function mutator (...args) {
    // 代理数组方法
    const result = original.apply(this, args)
    const ob = this.__ob__
    let inserted
    switch (method) {
      case 'push':
      case 'unshift':
        inserted = args
        break
      case 'splice':
        inserted = args.slice(2)
        break
    }
    // 递归观察插入内容
    if (inserted) ob.observeArray(inserted)
    // 通知数组依赖管理器进行更新
    ob.dep.notify()
    return result
  })
})
```

我们可以看到其实 arrayMethods 也只是调用了原来的数组方法

```js
const result = original.apply(this, args)
```

差别在于会对新插入的对象进行深度观察(push、unshift、splice 方法)

```js
if (inserted) ob.observeArray(inserted)
```

并在调用任何数组方法的时候就通知依赖进行更新

```js
ob.dep.notify()
```

### 1.2 数据观测 observe

在看 `defineReactive` 方法之前，我们先来看看观测数组的时候递归调用的 `observe` 方法是如何进行深度观察的

- `/src/core/observer/index.js`(阅读笔记文件路径：`/src/core/observer/index/observe.js`)

```js
// 观察目标(创建 Observer 对象，并更新 vmCount 值)

export function observe (value: any, asRootData: ?boolean): Observer | void {
  if (!isObject(value) || value instanceof VNode) {
    return
  }
  let ob: Observer | void
  if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) {
    // Observer 已经存在
    ob = value.__ob__
  } else if (
    shouldObserve &&
    !isServerRendering() &&
    (Array.isArray(value) || isPlainObject(value)) &&
    Object.isExtensible(value) &&
    !value._isVue
  ) {
    // 重新创建
    ob = new Observer(value)
  }
  if (asRootData && ob) {
    // 更新以该实例为根的 Vue 实例数量
    ob.vmCount++
  }
  return ob
}
```

我们可以看到 `observe` 方法的核心逻辑就是拿到观测目标的 `__ob__` 属性(没有就创建一个)并返回。所以这样看来剩下真正的将对象属性转换为响应式数据的方法就是 `defineReactive` 方法了。

### 1.3 转化为响应式对象 defineReactive

还记得在观测目标为对象的时候，我们调用 `walk` 方法遍历观测目标并将每个属性(key)转换为响应式数据

```js
// 遍历 "对象属性" 并转变为响应式数据
walk (obj: Object) {
  const keys = Object.keys(obj)
  for (let i = 0; i < keys.length; i++) {
    defineReactive(obj, keys[i])
  }
}
```

现在我们就来看看 `defineReactive` 实际上做了哪些操作

- `/src/core/observer/index.js`(阅读笔记文件路径：`/src/core/observer/index/defineReactive.flat2.js`)

```js
// 将对象属性转化为响应式(Object.defineProperty.getter/setter)

export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function,
  shallow?: boolean
) {
  const dep = new Dep()

  // configurable === false 时不作为响应式属性
  const property = Object.getOwnPropertyDescriptor(obj, key)
  if (property && property.configurable === false) {
    return
  }

  // 获取原来的访问器(getter/setter)方法
  const getter = property && property.get
  const setter = property && property.set
  if ((!getter || setter) && arguments.length === 2) {
    val = obj[key]
  }

  // 递归观察属性值
  let childOb = !shallow && observe(val)

  // 重新设置访问器属性 getter/setter
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      // 根据原访问器获得值
      const value = getter ? getter.call(obj) : val
      if (Dep.target) {
        // 将依赖加入当前运行的观察者
        dep.depend()
        if (childOb) {
          // 将对象值的依赖也加入当前观察者
          childOb.dep.depend()
          if (Array.isArray(value)) {
            dependArray(value)
          }
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val
      // 新旧值比较
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }

      // customSetter ...

      if (getter && !setter) return // read-only property
      // 设置新值
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }

      // 观察新值
      childOb = !shallow && observe(newVal)
      // 依赖通知所有观察者进行更新
      dep.notify()
    }
  })
}

// 遍历数组将对象值作为依赖加入当前观察者
function dependArray (value: Array<any>) {
  for (let e, i = 0, l = value.length; i < l; i++) {
    e = value[i]
    e && e.__ob__ && e.__ob__.dep.depend()
    if (Array.isArray(e)) {
      // 递归添加依赖
      dependArray(e)
    }
  }
}
```

我们看到 `defineReactive` 总共做了以下几件事情

- 建立一个 **依赖收集器(dep)**，后面会再说明依赖收集器的作用
- 使用 `Object.getOwnPropertyDescriptor` 保留原始的 getter/setter
- `observe(val)` 递归观察 value
- 设置新的 getter/setter

前三个步骤就是字面意思，代码也很简单，下面直接针对第四步骤展开说明

#### 1.3.1 访问数据 Object.defineProperty.getter

```js
// 重新设置访问器属性 getter/setter
Object.defineProperty(obj, key, {
  enumerable: true,
  configurable: true,
  get: function reactiveGetter () {
    // 根据原访问器获得值
    const value = getter ? getter.call(obj) : val
    if (Dep.target) {
      // 将依赖加入当前运行的观察者
      dep.depend()
      if (childOb) {
        // 将对象值的依赖也加入当前观察者
        childOb.dep.depend()
        if (Array.isArray(value)) {
          dependArray(value)
        }
      }
    }
    return value
  },
  set: function reactiveSetter (newVal) {/* ... */}
})
```

使用 `defineProperty` 描述符定义的 `get(reactiveGetter)` 方法就是每次访问这个对象的这个属性的时候就会调用的方法。

我们可以看到新的 getter 内部

1. 首先现根据原本的 getter 去获取值
2. 接下来则是检查是否存在 `Dep.target` 也就是我们要收集的目标依赖，有的话就调用 `dep.depend()` 让当前的依赖管理器去收集依赖
3. 接下来检查是否存在子属性依赖(childOb)，有则也加入当前依赖，并递归访问(收集)数组元素依赖。

后面我们会再提到到底在依赖什么hhh。

#### 1.3.2 更新数据 Object.defineProperty.setter

```js
set: function reactiveSetter (newVal) {
  const value = getter ? getter.call(obj) : val
  // 新旧值比较
  if (newVal === value || (newVal !== newVal && value !== value)) {
    return
  }
  // customSetter ...
  if (getter && !setter) return // read-only property
  // 设置新值
  if (setter) {
    setter.call(obj, newVal)
  } else {
    val = newVal
  }
  // 观察新值
  childOb = !shallow && observe(newVal)
  // 依赖通知所有观察者进行更新
  dep.notify()
}
```

`set(reacitveSetter)` 则是每次对数据进行赋值如 `obj[key] = xxx` 的时候会调用的方法

1. 首先透过 getter 获取旧的值
2. 做新旧值的比较
3. 不同时调用原来的 setter 更新数据 `setter.call(obj, newVal)`
4. 最后递归观察新的值 `observe(newVal)` 后，通知依赖进行更新 `dep.notify()`

### 1.4 数据观测小结

到此其实我们已经完成响应式数据的构建了：

1. 首先构建置于 `__ob__` 属性上的 Observer 对象并递归深度观察对象/数组
2. 递归观察的过程分为对象(Object)/数组(Array)
   1. 对于 Object 对象：遍历所有 key 属性调用 `defineReactive` 转变为响应式数据
   2. 对于 Array 对象：代理数组方法(arrayMethods)后，遍历添加观察目标代理(Observer)
3. 将观察目标 Object 的所有 key(深度观察) 转变为响应式的 `defineReactive`，利用 `Object.defineProperty` 的属性描述符(getter/setter)来代理原本属性值访问/更新的行为

下面我们再来谈谈前面提到的所谓的 **依赖管理器(Dep)**、**依赖** 又是些什么东西。

## 2. 依赖收集

到此为止其实我们已经完成响应式对象的创建了，问题是在哪里用(谁要观察数据、数据更新又要通知谁)，我们再回到开头的一张图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/vue2_reactive_implement_mvvm2.png)

前面我们完成的部分就是所谓的 Reactive Proxy 的部分，透过建立 Observer 对象以及 `defineReactive` 方法来深度观察并代理所有对象属性的 getter 和 setter。

现在剩下的问题是 Render Proxy 需要根据 Reactive Proxy 提供的数据来进行视图渲染，同时要向 Reactive Proxy 订阅数据，才能在数据更新的时候实时的重新渲染(也就是所谓的 **数据绑定** 的部分)，这时候我们就可以说：**Render Proxy 依赖于某个 Reactive Proxy**

下面我们就来看看到底该如何处理与绑定这种依赖关系

### 2.1 依赖管理器 Dep

首先我们先来看看前面好多次提到的 **依赖管理器(Dep)** (在 Observer 对象、`defineReactive` 方法里面都有提到)，先来看看定义

- `/src/core/observer/dep.js`(阅读笔记文件路径：`/src/core/observer/dep/Dep.flat2.js`)

```js
export default class Dep {
  static target: ?Watcher;
  id: number;
  subs: Array<Watcher>;

  constructor () {
    /* 初始化 uid & 观察者列表 */
    this.id = uid++
    this.subs = []
  }

  // 添加观察者
  addSub (sub: Watcher) {
    this.subs.push(sub)
  }

  // 移除观察者
  removeSub (sub: Watcher) {
    remove(this.subs, sub)
  }

  // 根据 Dep.target 添加观察者
  depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }

  // 通知观察者更新
  notify () {
    const subs = this.subs.slice()
    if (process.env.NODE_ENV !== 'production' && !config.async) {
      // async mode sorting
      subs.sort((a, b) => a.id - b.id)
    }
    for (let i = 0, l = subs.length; i < l; i++) {
      /* 逐一通知更新 */
      subs[i].update()
    }
  }
}
```

首先我们先要明确一个大前提：**依赖管理器(Dep)负责为响应式对象管理依赖**

这里所谓的 **依赖** 其实就是某个会用到这个响应式数据的对象，我们后面会再提到。这里我们先来看看 Dep 类里面都有哪些方法：

- `addSub`、`removeSub` 用于添加、移除依赖对象
- `notify` 用于通知所有依赖进行更新(调用 `update` 方法)
- `depend` 将依赖管理器加入 **Dep.target** 中

### 2.2 观察者对象 Watcher

在开始 Dep 对象的运作模式之前我们再来谈谈 **依赖** 这回事。我们前面已经提到：

1. Observer 负责代理被观察对象(观察目标)的一切观察行为并递归深度观察目标
2. Dep 负责观察目标的依赖管理，保留使用了该数据(依赖于该响应式对象)的观察者

所以其实我们就能够知道所谓的"依赖"，其实就是一个 **观察者(Watcher)**。

- `/src/core/observer/watcher.js`(阅读笔记文件路径：`/src/core/observer/watcher/constructor_get.js`)

```js
/* @flow */

import {
  warn,
  remove,
  isObject,
  parsePath,
  _Set as Set,
  handleError,
  noop
} from '../util/index'

import { traverse } from './traverse'
import { queueWatcher } from './scheduler'
import Dep, { pushTarget, popTarget } from './dep'

import type { SimpleSet } from '../util/index'

let uid = 0

/* 观察者对象 */
export default class Watcher {
  vm: Component;
  expression: string;
  cb: Function;
  id: number;
  deep: boolean;
  user: boolean;
  lazy: boolean;
  sync: boolean;
  dirty: boolean;
  active: boolean;
  deps: Array<Dep>;
  newDeps: Array<Dep>;
  depIds: SimpleSet;
  newDepIds: SimpleSet;
  before: ?Function;
  getter: Function;
  value: any;

  constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: ?Object,
    isRenderWatcher?: boolean
  ) {
    /** 
     * 实例属性初始化：
     * vm 观察目标Vue实例
     * options: deep, lazy, sync, before
     */
    this.vm = vm
    if (isRenderWatcher) {
      vm._watcher = this
    }
    vm._watchers.push(this)
    // options
    if (options) {
      this.deep = !!options.deep
      this.user = !!options.user
      this.lazy = !!options.lazy
      this.sync = !!options.sync
      this.before = options.before
    } else {
      this.deep = this.user = this.lazy = this.sync = false
    }
    /**
     * cb, id, active, dirty, 
     * deps, newDpes, depIds, newDepIds,
     * expression
     */
    this.cb = cb
    this.id = ++uid // uid for batching
    this.active = true
    this.dirty = this.lazy // for lazy watchers
    this.deps = []
    this.newDeps = []
    this.depIds = new Set()
    this.newDepIds = new Set()
    this.expression = process.env.NODE_ENV !== 'production'
      ? expOrFn.toString()
      : ''
    /**
     * getter
     */
    if (typeof expOrFn === 'function') {
      this.getter = expOrFn
    } else {
      this.getter = parsePath(expOrFn)
      if (!this.getter) {
        this.getter = noop
        process.env.NODE_ENV !== 'production' && warn(
          `Failed watching path: "${expOrFn}" ` +
          'Watcher only accepts simple dot-delimited paths. ' +
          'For full control, use a function instead.',
          vm
        )
      }
    }
    /**
     * invoke getter
     */
    this.value = this.lazy
      ? undefined
      : this.get()
  }

  /**
   * 取值：
   *   调用 getter
   *   traverse 递归遍历
   */ 
  get () {
    pushTarget(this)
    let value
    const vm = this.vm
    try {
      value = this.getter.call(vm, vm)
    } catch (e) {
      if (this.user) {
        handleError(e, vm, `getter for watcher "${this.expression}"`)
      } else {
        throw e
      }
    } finally {
      // "touch" every property so they are all tracked as
      // dependencies for deep watching
      if (this.deep) {
        traverse(value)
      }
      popTarget()
      this.cleanupDeps()
    }
    return value
  }

}
```

#### 2.2.1 构造函数 constructor

首先我们看到观察者被创建的时候(constructor)会

1. 初始化内部参数

```js
// options
if (options) {
  this.deep = !!options.deep
  this.user = !!options.user
  this.lazy = !!options.lazy
  this.sync = !!options.sync
  this.before = options.before
} else {
  this.deep = this.user = this.lazy = this.sync = false
}
```

2. 初始化实例变量

```js
/**
 * cb, id, active, dirty, 
 * deps, newDpes, depIds, newDepIds,
 * expression
 */
this.cb = cb
this.id = ++uid // uid for batching
this.active = true
this.dirty = this.lazy // for lazy watchers
this.deps = []
this.newDeps = []
this.depIds = new Set()
this.newDepIds = new Set()
this.expression = process.env.NODE_ENV !== 'production'
  ? expOrFn.toString()
  : ''
/**
 * getter
 */
this.getter = expOrFn
```

3. 调用 `get` 方法获取初始化依赖值(首次调用 `get`)

```js
/**
* invoke getter
*/
this.value = this.lazy
  ? undefined
  : this.get()
```

#### 2.2.2 获取观察值并绑定依赖关系 get

`get` 方法可以说是整个观察者对象的核心中的核心

```js
/**
 * 取值：
 *   调用 getter
 *   traverse 递归遍历
 */ 
get () {
  pushTarget(this)
  let value
  const vm = this.vm
  try {
    value = this.getter.call(vm, vm)
  } catch (e) {
    if (this.user) {
      handleError(e, vm, `getter for watcher "${this.expression}"`)
    } else {
      throw e
    }
  } finally {
    // "touch" every property so they are all tracked as
    // dependencies for deep watching
    if (this.deep) {
      traverse(value)
    }
    popTarget()
    this.cleanupDeps()
  }
  return value
}
```

首先先看到 `pushTarget`、`popTarget` 两个方法

- `/src/core/observer/dep.js`(阅读笔记文件路径：`/src/core/observer/dep/Dep.target.js`)

```js
// Dep.target 当前观察者

Dep.target = null
const targetStack = []

export function pushTarget (target: ?Watcher) {
  targetStack.push(target)
  Dep.target = target
}

export function popTarget () {
  targetStack.pop()
  Dep.target = targetStack[targetStack.length - 1]
}
```

`Dep.target` 其实就是当前正在运行 `get` 方法取值(访问数据)的观察者对象，每一个时刻只可能存在唯一一个对象(观察者可能递归调用或是深度遍历所以做成栈 `targetStack` 的形式)

我们可以看到整个 `get` 方法前后就是用 `pushTarget`、`popTarget`，说明在整个观察值的期间访问的所有可观察数据(存在 `__ob__` 属性的数据对象)都作为该观察者的依赖。

整个 `get` 方法的核心就是下面两句

```js
value = this.getter.call(vm, vm)

// ...

traverse(value)
```

首先利用构造函数绑定的 getter 获取观察值，然后递归访问该观察值的所有深度属性。

### 2.3 收集依赖

还记得我们前面已经在 `defineReactive` 的 getter 访问器属性上是这么写的

- `/src/core/observer/index.js`(阅读笔记文件路径：`/src/core/observer/index/defineReactive.flat2.js`)

```js
// ...

get: function reactiveGetter () {
  // 根据原访问器获得值
  const value = getter ? getter.call(obj) : val
  if (Dep.target) {
    // 将依赖加入当前运行的观察者
    dep.depend()
    if (childOb) {
      // 将对象值的依赖也加入当前观察者
      childOb.dep.depend()
      if (Array.isArray(value)) {
        dependArray(value)
      }
    }
  }
  return value
},

// ...
```

现在我们就能知道，其实就是在 Watcher 的 `get` 方法执行的时候，会作为 `Dep.target` 的角色去访问各个响应式数据，响应式数据的 getter 就会使用 `dep.depend()` 方法来将 `Dep.target` 注册到当前的依赖收集器当中

```js
// ...

set: function reactiveSetter (newVal) {
  const value = getter ? getter.call(obj) : val
  // 新旧值比较
  if (newVal === value || (newVal !== newVal && value !== value)) {
    return
  }

  // customSetter ...

  if (getter && !setter) return // read-only property
  // 设置新值
  if (setter) {
    setter.call(obj, newVal)
  } else {
    val = newVal
  }

  // 观察新值
  childOb = !shallow && observe(newVal)
  // 依赖通知所有观察者进行更新
  dep.notify()
}

// ...
```

而在响应式数据更新的时候，就会调用 `dep.notify()` 来通知所有先前访问过(getter 中收集的依赖)的 Watcher 进行更新。

### 2.4 响应式原理完整流程总结

到此就完成整个响应式数据的构建了，主要分成一下步骤/阶段

1. 观察数据：使用 `observe` 方法为观察目标(被观察对象)建立 **观察目标代理(Observer)**，放到 `__ob__` 属性上
2. 依赖管理：观察目标代理还维护了一个 **依赖管理器(Dep)**，用于维护依赖于该数据的观察者和通知观察者进行更新
3. 依赖收集：在 `defineReactive` 的 `getter` 方法中，将每个访问过该数据的观察者(访问的当下会作为 `Dep.target` 存在)加入依赖管理器(`dep.depend()`)
4. 数据更新：每当响应式对象进行更新的时候(`setter` 调用)，就会通知依赖管理器(dep)通知观察者进行更新(`dep.notify()`)

下图是一个响应式数据对象的实例依赖关系

![](https://picures.oss-cn-beijing.aliyuncs.com/img/vue2_reactive_implement_structure.png)

#### 2.4.1 观察数据示例：initData

还记得在 Vue 实例初始化的时候有这么一个阶段：`initState` 初始化所有实例变量。我们拿其中的 `initData` 做示例

- `/src/core/instance/state.js`(阅读笔记文件路径：`/src/core/instance/state/initState.js`)

```js
function initData (vm: Component) {
  let data = vm.$options.data
  data = vm._data = typeof data === 'function'
    ? getData(data, vm)
    : data || {}
  // non-object data warning ...
  // proxy data on instance
  const keys = Object.keys(data)
  const props = vm.$options.props
  const methods = vm.$options.methods
  let i = keys.length
  while (i--) {
    const key = keys[i]
    // conflict with methods warning ...
    if (props && hasOwn(props, key)) {
      // conflict with props warning ...
    } else if (!isReserved(key)) {
      /* 对每个属性建立 vm._data.key 的代理 */
      proxy(vm, `_data`, key)
    }
  }
  /* 转变为响应式数据 */
  observe(data, true /* asRootData */)
}
```

我们可以看到其实整个 `initData` 方法除了检查与 props、methods 啥的进行命名冲突检查之外，其实就是调用 `observe` 方法来将数据对象作为 Vue 实例的根数据对象并 **转化为响应式对象**(经过 `observe -> Observer.constructor -> Observer.walk -> defineReactive -> Object.defineProperty` 的流程来完成响应式数据对象的创建)

## 3. 实现细节补充 & 弥补不足之处

下面再来补充一些实现细节以及一些 Vue.prototype 的方法用于补足响应式对象实现原理的不足。

### 3.1 观察者如何注册并管理依赖？

首先第一个要说的是 `dep.depend()` 方法。我们在 `defineReactive` 的 getter 定义中看到 `dep.depend()` 方法的调用，知道它是将当前 `Dep.target` 的对象注册到 `dep` 对象当中，但其实这其中还使用了新旧两个依赖队列来进行依赖管理。我们先看到几个方法定义

- `/src/core/observer/dep.js`(阅读笔记文件路径：`/src/core/observer/dep/Dep.flat2.js`)

```js
export default class Dep {

  // ...

  // 根据 Dep.target 添加观察者
  depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }

  // ...
}
```

我们已经知道这里所谓的 `Dep.target` 其实就是某个 Watcher 对象，再接着看 `addDep` 方法：

- `/src/core/observer/watcher.js`(阅读笔记文件路径：`/src/core/observer/watcher/addDep_cleanupDeps.js`)

```js
// 观察者对象

export default class Watcher {

  // instance props ...

  // 添加依赖
  addDep (dep: Dep) {
    const id = dep.id
    if (!this.newDepIds.has(id)) {
      this.newDepIds.add(id)
      this.newDeps.push(dep)
      // 队列中不存在表示尚未注册到 dep
      if (!this.depIds.has(id)) {
        dep.addSub(this)
      }
    }
  }
}
```

我们可以看到其实 `addDep` 是将依赖管理器加入到一个 `newDeps` 的队列当中，并检查如果原队列 `deps` 没有该管理器时才向该管理器注册观察者自身(`dep.addSub(this)`)。

我们再注意到 Watcher 对象的 `get` 方法末尾，其实还多了句：

- `/src/core/observer/watcher.js`(阅读笔记文件路径：`/src/core/observer/watcher/constructor_get.js`)

```js
// ...
popTarget()
this.cleanupDeps()

// ...
```

也就是说在 `get` 结束的时候其实调用了一个 `cleanupDeps` 方法

- `/src/core/observer/watcher.js`(阅读笔记文件路径：`/src/core/observer/watcher/addDep_cleanupDeps.js`)

```js
// 刷新依赖队列

cleanupDeps () {
  let i = this.deps.length
  while (i--) {
    const dep = this.deps[i]
    // 清理下一轮未 dep
    if (!this.newDepIds.has(dep.id)) {
      dep.removeSub(this)
    }
  }
  // 以 newDepIds 替换 depIds
  let tmp = this.depIds
  this.depIds = this.newDepIds
  this.newDepIds = tmp
  this.newDepIds.clear()
  // 以 newDeps 替换 deps
  tmp = this.deps
  this.deps = this.newDeps
  this.newDeps = tmp
  this.newDeps.length = 0
}
```

我们可以看到 `cleanupDeps` 方法完成了这么一个操作：以 `newDeps` 替换 `deps`，并对 `newDpes` 中不存在的 `dep` 取消订阅

#### 3.1.1 图解 Watcher.get 方法

也就是说其实在一次的获取观察值(访问响应式数据对象)的过程结束后，都会进行一次下图的操作

![](https://picures.oss-cn-beijing.aliyuncs.com/img/vue2_reactive_implement_watcher_deps.png)

来更新观察者对象的依赖队列

### 3.2 Vue 实例方法补充

在上面提到的响应式数据实现原理中其实还存在几个很严重的缺陷：

- Object 对象数据
  - 无法观测到新属性的添加：`obj[newKey] = value`
  - 无法观测到属性被删除：`delete obj[oldKey]`
- Array 数组数据
  - 直接透过下标的数据赋值：`arr[index] = value`

为了解决该问题，Vue 就提供了 `Vue.prototype.$set`、`Vue.prototype.$del` 两个方法来填补这个漏洞；最后我们会再补充一个 `Vue.prototype.$watch` 方法，它让我们可以主动的观察一个与普通实例变量使用无关的数据对象。

上面三个方法都是在 `stateMixin` 的时候被混入 `Vue.prototype` 对象中的

- `/src/core/instance/state.js`(阅读笔记文件路径：`/src/core/instance/state/initState.js`)

```js
// 注入
//   Vue.prototype.$data  -> dataDef  -> this._data
//   Vue.prototype.$props -> propsDef -> this._props
//   Vue.prototype.$set
//   Vue.prototype.$delete
//   Vue.prototype.$watch

export function stateMixin (Vue: Class<Component>) {
  const dataDef = {}
  dataDef.get = function () { return this._data }
  const propsDef = {}
  propsDef.get = function () { return this._props }
  if (process.env.NODE_ENV !== 'production') {
    dataDef.set = function () {
      // direct set root $data warning ...
    }
    propsDef.set = function () {
      // set $props warning
    }
  }

  Object.defineProperty(Vue.prototype, '$data', dataDef)
  Object.defineProperty(Vue.prototype, '$props', propsDef)

  Vue.prototype.$set = set
  Vue.prototype.$delete = del

  Vue.prototype.$watch = function (
    expOrFn: string | Function,
    cb: any,
    options?: Object
  ): Function {/* ... */}
}
```

#### 3.2.1 Vue.prototype.$set

首先第一个我们先来看看 `set` 方法的实现

- `/src/core/observer/index.js`(阅读笔记文件路径：`/src/core/observer/index/set_del.js`)

```js
// 用于注入
//     Vue.prototype.$set

export function set (target: Array<any> | Object, key: any, val: any): any {
  // target undefined, null, primitive warning ... 

  // set(array, key, val)
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    target.length = Math.max(target.length, key)
    target.splice(key, 1, val)
    return val
  }
  // key already exists
  if (key in target && !(key in Object.prototype)) {
    target[key] = val
    return val
  }

  const ob = (target: any).__ob__
  // target as _data warning ...
  // target 不是观察目标
  if (!ob) {
    target[key] = val
    return val
  }
  // target 是观察目标 -> 设置为响应式属性
  defineReactive(ob.value, key, val)
  // 通知依赖于 target 的进行更新
  ob.dep.notify()
  return val
}
```

我们可以看到其实整个步骤/逻辑非常清楚

1. 检查是否为数组操作：使用 `splice` 方法取代，就能够被前面替换的 arrayMethods 代理观测到
2. 检查属性是否已存在
   1. 非响应式对象：则直接赋值并返回，不做额外行为
   2. 响应式对象：一样直接赋值，会走已经代理过的 getter，自动触发依赖收集
3. 检查观测目标代理是否存在(即表示该数据对象是否为响应式对象，以 `__ob__` 对象的存在与否作为标志)
4. 使用 `defineReactive` 对响应式数据对象进行赋值，并通知该对象的依赖管理器通知依赖进行更新

#### 3.2.2 Vue.prototype.$del

- `/src/core/observer/index.js`(阅读笔记文件路径：`/src/core/observer/index/set_del.js`)

```js
// 用于注入
//     Vue.prototype.$del

export function del (target: Array<any> | Object, key: any) {
  // target undefined, null, primitive warning ... 

  // del(array, key, val)
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    target.splice(key, 1)
    return
  }
  const ob = (target: any).__ob__
  // target as _data warning ...

  // key not exists
  if (!hasOwn(target, key)) {
    return
  }
  delete target[key]
  if (!ob) {
    return
  }
  // 是观察目标则通知依赖于 target 的进行更新
  ob.dep.notify()
}
```

`del` 的逻辑也很简单

1. 一样将下标访问数组的行为用 `splice` 替换
2. 检查属性是否存在(`hasOwn`)
3. 对响应式数据对象(`__ob__` 存在)通知目标进行更新(`ob.dep.notify()`)

#### 3.2.3 Vue.prototype.$watch

- `/src/core/instance/state.js`(阅读笔记文件路径：`/src/core/instance/state/stateMixin.flat2.$watch.js`)

```js
// 注入 Vue.prototype.$watch

export function stateMixin (Vue: Class<Component>) {
  // dataDef, propsDef ...

  // set Vue.prototype.{$data, $props, $set, $delete} ...

  Vue.prototype.$watch = function (
    expOrFn: string | Function,
    cb: any,
    options?: Object
  ): Function {
    const vm: Component = this
    if (isPlainObject(cb)) {
      return createWatcher(vm, expOrFn, cb, options)
    }
    options = options || {}
    options.user = true
    /* 创建 Watcher 对象观察 expOrFn 对象 */
    const watcher = new Watcher(vm, expOrFn, cb, options)
    if (options.immediate) {
      try {
        cb.call(vm, watcher.value)
      } catch (error) {
        handleError(error, vm, `callback for immediate watcher "${watcher.expression}"`)
      }
    }
    // 返回解除观察方法回调
    return function unwatchFn () {
      watcher.teardown()
    }
  }
}
```

我们可以看到其实实例上的 `Vue.prototype.$watch` 方法就做了一件事：**创建一个 Watcher 来观察对象**，接下来就会透过一系列的步骤与响应式数据形成绑定(`Watcher.constructor -> Watcher.get -> Watcher.getter -> defineReactive.getter -> dep.depend -> dep.addSub` 完成实例对象对响应式数据的订阅)。其中每个步骤在上面都提过了，可以往回翻一翻再复习复习

# 结语

本篇完成了 Vue2 中对于响应式对象实现原理的源码解析，比较重要的是 Vue2 中使用的底层机制为 `Object.defineProperty` 相对来说是一个比较老的特性，造成后面还需要增加 `set`、`del` 来弥补 getter/setter 无法触及的部分；到了 Vue3 则使用了 ES6 的 Proxy，其中的 `handler` 描述符提供了更全面的响应式数据访问代理，后面找个机会会去爬爬看 Vue3 的源码。

# 其他资源

## 参考连接

<table>
  <tr>
    <td>变化侦测篇综述 | Vue源码系列-Vue中文社区</td>
    <td><a href="https://vue-js.com/learn-vue/reactive/">https://vue-js.com/learn-vue/reactive/</a></td>
  </tr>
  <tr>
    <td>vuejs/vue-2.6.12-Github</td>
    <td><a href="https://github.com/vuejs/vue/tree/v2.6.12">https://github.com/vuejs/vue/tree/v2.6.12</a></td>
  </tr>
  <tr>
    <td>设计模式: Observer 观察者模式</td>
    <td><a href="https://blog.csdn.net/weixin_44691608/article/details/113757262">https://blog.csdn.net/weixin_44691608/article/details/113757262</a></td>
  </tr>
</table>

## 阅读笔记参考

<a href="https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/vue-2.6.12">https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/vue-2.6.12</a>
