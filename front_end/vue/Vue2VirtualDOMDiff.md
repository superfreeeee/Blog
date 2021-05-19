# Vue2 源码解析: MVVM 双向绑定2 - 虚拟 DOM & diff 算法原理深度解析(长文慎入！)

@[TOC](文章目录)

<!-- TOC -->

- [Vue2 源码解析: MVVM 双向绑定2 - 虚拟 DOM & diff 算法原理深度解析(长文慎入！)](#vue2-源码解析-mvvm-双向绑定2---虚拟-dom--diff-算法原理深度解析长文慎入)
- [前言](#前言)
  - [回顾：MVVM 实现细化](#回顾mvvm-实现细化)
  - [Render Proxy 渲染代理](#render-proxy-渲染代理)
  - [Render Proxy 实现方案：虚拟 DOM](#render-proxy-实现方案虚拟-dom)
  - [虚拟 DOM 的好处 & 误区：平台无关](#虚拟-dom-的好处--误区平台无关)
- [正文](#正文)
  - [1. VNode 虚拟节点](#1-vnode-虚拟节点)
    - [1.1 VNodeData](#11-vnodedata)
    - [1.2 VNodeComponentOptions](#12-vnodecomponentoptions)
  - [2. diff 算法](#2-diff-算法)
    - [回顾：什么时候进行更新并重新渲染节点？](#回顾什么时候进行更新并重新渲染节点)
      - [2.0.1 _render 构建vnode虚拟节点](#201-_render-构建vnode虚拟节点)
        - [2.0.1.1 注入 Vue.prototype.$createElement](#2011-注入-vueprototypecreateelement)
        - [2.0.1.2 _createElement 细节](#2012-_createelement-细节)
      - [2.0.2 _update 比较并更新 DOM 后进行渲染](#202-_update-比较并更新-dom-后进行渲染)
      - [2.0.3 diff 流程小结](#203-diff-流程小结)
    - [2.1 注入 Vue.prototype.\_\_patch\_\_](#21-注入-vueprototype__patch__)
    - [2.2 patch 方法的构成](#22-patch-方法的构成)
      - [2.2.1 node-ops 平台相关具体节点方法](#221-node-ops-平台相关具体节点方法)
      - [2.2.2 modules 节点管理模块](#222-modules-节点管理模块)
        - [2.2.2.1 baseModules 核心模块：refs、directives](#2221-basemodules-核心模块refsdirectives)
        - [2.2.2.2 platformModules 平台模块：attrs、klass、events、domProps、style、transition](#2222-platformmodules-平台模块attrsklasseventsdompropsstyletransition)
        - [2.2.2.3 hook 模块钩子](#2223-hook-模块钩子)
      - [2.2.3 createPatchFunction 构建patch函数](#223-createpatchfunction-构建patch函数)
    - [2.3 明确 patch 目标 & 区分操作类型](#23-明确-patch-目标--区分操作类型)
      - [2.3.1 createElm 创建 DOM 节点](#231-createelm-创建-dom-节点)
        - [2.3.1.1 createComponent 尝试创建组件节点](#2311-createcomponent-尝试创建组件节点)
        - [2.3.1.2 initComponent 初始化组件节点](#2312-initcomponent-初始化组件节点)
        - [2.3.1.3 invokeCreateHooks 调用节点模块 create 钩子](#2313-invokecreatehooks-调用节点模块-create-钩子)
        - [2.3.1.4 createChildren 创建子组件列表](#2314-createchildren-创建子组件列表)
      - [2.3.2 removeNode 删除 DOM 节点](#232-removenode-删除-dom-节点)
      - [2.3.3 patch 比较并更新 DOM 树(\_\_patch\_\_ 方法的本体)](#233-patch-比较并更新-dom-树__patch__-方法的本体)
        - [2.3.3.1 invokeDestroyHook 销毁节点并调用 destroy 钩子](#2331-invokedestroyhook-销毁节点并调用-destroy-钩子)
        - [2.3.3.2 patchVnode 比较并更新新旧节点](#2332-patchvnode-比较并更新新旧节点)
        - [2.3.3.3 addVnodes 添加多个节点](#2333-addvnodes-添加多个节点)
        - [2.3.3.4 removeVnodes 移除多个节点](#2334-removevnodes-移除多个节点)
        - [2.3.3.5 updateChildren 更新子节点数组](#2335-updatechildren-更新子节点数组)
        - [2.3.3.6 更新子节点策略代码 & 图解](#2336-更新子节点策略代码--图解)
        - [2.3.3.7 invokeInsertHook 完成节点插入调用 insert 钩子](#2337-invokeinserthook-完成节点插入调用-insert-钩子)
  - [3. 虚拟 DOM 与 diff 算法总结](#3-虚拟-dom-与-diff-算法总结)
    - [3.1 实现选择和主体流程](#31-实现选择和主体流程)
    - [3.2 比较更新 VNode 节点树：__patch__ 方法](#32-比较更新-vnode-节点树patch-方法)
    - [3.3 比较更新操作的五种情形：patch 方法](#33-比较更新操作的五种情形patch-方法)
    - [3.4 深度比较更新：patchVNode 方法](#34-深度比较更新patchvnode-方法)
    - [3.5 深度比较子节点：updateChildren 方法](#35-深度比较子节点updatechildren-方法)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 前言

前一篇 <a href="https://blog.csdn.net/weixin_44691608/article/details/116776465">Vue2 源码解析: MVVM 双向绑定1 - 响应式原理(数据观测/响应)</a> 解析 Vue2 使用 `Object.defineProperty` 为基础构建的响应式数据原理具体实现机制，本篇将来说明 MVVM 实现的第二阶段：**虚拟 DOM(Virtual DOM)**

## 回顾：MVVM 实现细化

首先我们先来看看前一篇提到的对于 MVVM 架构的细化

![](https://picures.oss-cn-beijing.aliyuncs.com/img/vue2_reactive_implement_mvvm2.png)

根据上一篇我们已经了解右侧的 Reactive Proxy 也就是我们的响应式数据部分的实现，本篇就要来说说左侧的 Render Proxy 的实现。

## Render Proxy 渲染代理

首先我们先来说明一下为啥要取名为渲染代理，这是因为在 MVVM 的架构中有一个非常关键的步骤：**将来自于 Model 数据自动与 View 视图进行绑定**。也就是说在构建响应式数据的基础之上，势必存在某一个渲染代理，作为 **响应式对象的消费者**，负责订阅响应式数据的改变并自动绑定、生成真实的 DOM 节点并对视图进行修改。

## Render Proxy 实现方案：虚拟 DOM

这时候 Render Proxy 其实存在多种可选的实现方案，不管使用哪一种方案，最终目标都是：**随时响应数据的变化，并自动更新绑定数据元素节点**

1. 使用真实 DOM

第一种实现方案使用真实 DOM 作为我们的渲染更新蓝本。我们可以直接为每个绑定数据的 DOM 元素生成观察者(或是简单的监听函数)，并在数据修改的时候直接与真实的 DOM 元素进行比较、更新。然而这会带来一个问题：**大量的 DOM 操作**。

每次数据的修改都会直接产生 DOM 元素的查询、修改，甚至进行节点的增删，这些都是昂贵的性能开销(当然还是存在很多优化的空间，甚至能超越虚拟 DOM 实现的性能，同时不需要虚拟 DOM 那般巨大的内存开销，不过选用哪种技术实现这不是本篇的讨论重点)。也因此如 React、Vue 等框架选择的是第二种的实现方案：**虚拟 DOM**。

2. 虚拟 DOM

虚拟 DOM 的核心思想在于，**使用 JS 构建一套虚拟的 DOM 树结构**，并在基于 JS 对象操作的基础下完成 DOM 树更新前后的 **差异比较(diff)**，最后将多个 DOM 操作进行合并，合成最小的 DOM 操作集合一次性进行修改，这就是虚拟 DOM 从运行角度的核心目标。

## 虚拟 DOM 的好处 & 误区：平台无关

值得一提的是，很多人在刚接触虚拟 DOM 的实现模式的时候会踏入一个误区：**使用虚拟 DOM 合并 DOM 操作能够提高性能**。然而真实情况下由于 DOM 节点的复杂性，额外的 JS 对象操作代价对于真实 DOM 操作的优化其实有限，同时还会额外 **占用多余的内存空间(保存虚拟节点信息)**，所以实际上虚拟 DOM 最大的优势并不在于 DOM 操作性能，而是：**平台无关的 DOM 渲染树结构**

借由透过 MVVM 框架实现虚拟 DOM 结构，我们可以构建出一个独立于平台 DOM 的渲染树结构，由框架来帮我们将虚拟 DOM 的结构映射成具体平台上的元素节点(如小程序、其他非 web 环境)

下面回到本篇的核心：**Vue2 的虚拟 DOM 实现与比较更新算法(DOM-diff)的具体实现**

# 正文

## 1. VNode 虚拟节点

首先一开始我们先看看 Vue 中虚拟 DOM 的节点类型 `VNode` 的定义

- `/src/core/vdom/vnode.js`(阅读笔记文件路径：`/src/core/vdom/vnode/VNode.js`)

```js
// 虚拟 DOM 元素节点

export default class VNode {
  tag: string | void;           // 元素标签
  data: VNodeData | void;       // 元素数据
  children: ?Array<VNode>;      // 子节点数组
  text: string | void;          // 节点内文本
  elm: Node | void;             // 对应真实 dom 节点
  ns: string | void;            // 当前节点内命名空间
  context: Component | void;    // 组件上下文(Vue 实例)
  key: string | number | void;  // 节点特征标识符(用于 v-for 判断)

  componentOptions: VNodeComponentOptions | void;  // 组件 options 选项
  componentInstance: Component | void;             // 组件对应的 Vue 实例
  parent: VNode | void;       // 父节点

  // strictly internal
  raw: boolean;          // 是否为纯 HTML 文本
  isStatic: boolean;     // 是否为静态节点(不重复渲染)
  isRootInsert: boolean; // 是否作为根节点
  isComment: boolean;    // 是否为注释
  isCloned: boolean;     // 是否为拷贝节点
  isOnce: boolean;       // 是否为 v-once(只渲染一次，不动态绑定数据)
  asyncFactory: Function | void;  // 异步组件工厂函数
  asyncMeta: Object | void;       // 异步元数据
  isAsyncPlaceholder: boolean;
  ssrContext: Object | void;
  fnContext: Component | void;    // 函数式组件对应的 Vue 实例
  fnOptions: ?ComponentOptions;   // 函数式组件 options 选项
  fnScopeId: ?string;             // 函数式组件作用域 id
  devtoolsMeta: ?Object; // used to store functional render context for devtools

  constructor (
    tag?: string,
    data?: VNodeData,
    children?: ?Array<VNode>,
    text?: string,
    elm?: Node,
    context?: Component,
    componentOptions?: VNodeComponentOptions,
    asyncFactory?: Function
  ) {
    this.tag = tag
    this.data = data
    this.children = children
    this.text = text
    this.elm = elm
    this.ns = undefined
    this.context = context
    this.fnContext = undefined
    this.fnOptions = undefined
    this.fnScopeId = undefined
    this.key = data && data.key
    this.componentOptions = componentOptions
    this.componentInstance = undefined
    this.parent = undefined
    this.raw = false
    this.isStatic = false
    this.isRootInsert = true
    this.isComment = false
    this.isCloned = false
    this.isOnce = false
    this.asyncFactory = asyncFactory
    this.asyncMeta = undefined
    this.isAsyncPlaceholder = false
  }

  // DEPRECATED: alias for componentInstance for backwards compat.
  /* istanbul ignore next */
  get child (): Component | void {
    return this.componentInstance
  }
}
```

我们可以看到 VNode 的类型定义就是一大堆用于描述原始 DOM 节点的属性，如 `tag` 表示标签名称、`children` 表示子节点列表等，详细含义都写在代码注释里面了。

另外由于 Vue2 源码中还另外添加了 `.d.ts` 类型声明文件来增加对 typescript 的支持，所以我们还可以在 `/types` 目录下看到许多相关类型定义

- `/types/vnode.d.ts`(阅读笔记文件路径：`/types/vnode/VNode.ts`)

```ts
// 虚拟 dom 节点

export interface VNode {
  tag?: string;
  data?: VNodeData;
  children?: VNode[];
  text?: string;
  elm?: Node;
  ns?: string;
  context?: Vue;
  key?: string | number;
  componentOptions?: VNodeComponentOptions;
  componentInstance?: Vue;
  parent?: VNode;
  raw?: boolean;
  isStatic?: boolean;
  isRootInsert: boolean;
  isComment: boolean;
}
```

与上面的原始定义类似，这边就不对属性含义再做解释

### 1.1 VNodeData

另外我们还可以看到其他类型像是 VNode 虚拟节点的数据类型(用于节点比较和渲染)

- `/types/vnode.d.ts`(阅读笔记文件路径：`/types/vnode/VNodeData.ts`)

```ts
// 虚拟 dom 节点数据

export interface VNodeData {
  key?: string | number;
  slot?: string;
  scopedSlots?: { [key: string]: ScopedSlot | undefined };
  ref?: string;
  refInFor?: boolean;
  tag?: string;
  staticClass?: string;
  class?: any;
  staticStyle?: { [key: string]: any };
  style?: string | object[] | object;
  props?: { [key: string]: any };
  attrs?: { [key: string]: any };
  domProps?: { [key: string]: any };
  hook?: { [key: string]: Function };
  on?: { [key: string]: Function | Function[] };
  nativeOn?: { [key: string]: Function | Function[] };
  transition?: object;
  show?: boolean;
  inlineTemplate?: {
    render: Function;
    staticRenderFns: Function[];
  };
  directives?: VNodeDirective[];
  keepAlive?: boolean;
}
```

### 1.2 VNodeComponentOptions

以及一些组件选项

- `/types/vnode.d.ts`(阅读笔记文件路径：`/types/vnode/VNodeComponentOptions.ts`)

```ts
// 虚拟 dom 组件选项

export interface VNodeComponentOptions {
  Ctor: typeof Vue;
  propsData?: object;
  listeners?: object;
  children?: VNode[];
  tag?: string;
}
```

## 2. diff 算法

接下来就正式进入虚拟 DOM 的 diff 算法，不过在此之前，我们先来回顾以下 VNode 虚拟节点是在什么时候被创建以及被渲染到真实 DOM 上的

### 回顾：什么时候进行更新并重新渲染节点？

回到我们首次进行实例挂载(`mountComponent`)的时候：

- `/src/core/instance/lifecycle.js`(阅读笔记文件路径：`/src/core/instance/lifecycle/mountComponent.js`)

```js
// 挂载实例

export function mountComponent (
  vm: Component,
  el: ?Element,
  hydrating?: boolean
): Component {
  vm.$el = el
  if (!vm.$options.render) {
    vm.$options.render = createEmptyVNode
    // missing template or render warning ...
  }
  callHook(vm, 'beforeMount')

  let updateComponent
  if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
    // production.version update ...
  } else {
    updateComponent = () => {
      vm._update(vm._render(), hydrating)
    }
  }

  // 创建观察者并挂载到 vm._watcher 上
  new Watcher(vm, updateComponent, noop, {
    before () {
      if (vm._isMounted && !vm._isDestroyed) {
        callHook(vm, 'beforeUpdate')
      }
    }
  }, true /* isRenderWatcher */)
  hydrating = false

  // 调用 mounted 生命周期钩子
  if (vm.$vnode == null) {
    vm._isMounted = true
    callHook(vm, 'mounted')
  }
  return vm
}
```

其中有一个方法特别值得我们注意

```js
updateComponent = () => {
  vm._update(vm._render(), hydrating)
}
```

这个方法会在首次进行渲染以及数据进行更新的时候被调用(其实就是作为 Watcher 的 `get` 方法中的 `getter` 方法被调用，[上一篇](https://blog.csdn.net/weixin_44691608/article/details/116776465)解说过了这边就不赘述)。也就是说每次的更新/首次渲染会先调用 `_render` 方法再调用 `_update` 方法，下面我们一个个来看

#### 2.0.1 _render 构建vnode虚拟节点

首先第一步骤的 `_render` 方法其实是用于生成当前数据对应的 vnode 节点树，我们看到初始化 Vue 实例时混入的 `Vue.prototype._render` 方法(在 `renderMixin` 方法中混入)

- `/src/core/instance/render.js`(阅读笔记文件路径：`/src/core/instance/render/renderMixin.flat2._render.js`)

```js
// Vue.prototype._render 渲染细节

export function renderMixin (Vue: Class<Component>) {
  /* 注入 render helpers */
  installRenderHelpers(Vue.prototype)

  Vue.prototype.$nextTick = function (fn: Function) {
    return nextTick(fn, this)
  }

  Vue.prototype._render = function (): VNode {
    const vm: Component = this
    const { render, _parentVnode } = vm.$options

    if (_parentVnode) {
      vm.$scopedSlots = normalizeScopedSlots(
        _parentVnode.data.scopedSlots,
        vm.$slots,
        vm.$scopedSlots
      )
    }

    // 保留父节点接入点
    vm.$vnode = _parentVnode
    // render self
    let vnode
    try {
      /* 递归创建 VNode */
      currentRenderingInstance = vm
      vnode = render.call(vm._renderProxy, vm.$createElement)
    } catch (e) {
      // render exception handling ...
      // ensure vnode exists
      vnode = vm._vnode
    } finally {
      currentRenderingInstance = null
    }
    // 接受包含唯一一个 VNode 的数组
    if (Array.isArray(vnode) && vnode.length === 1) {
      vnode = vnode[0]
    }
    // 保证 vnoe 存在
    if (!(vnode instanceof VNode)) {
      // multiple root node warning ...
      vnode = createEmptyVNode()
    }
    // set parent
    vnode.parent = _parentVnode
    return vnode
  }
}
```

忽略其他类型和环境检查以及其他辅助方法/属性的定义设置，用于创建虚拟 DOM 的方法是下面这一句

```js
vnode = render.call(vm._renderProxy, vm.$createElement)
```

还记得我们在创建实例的时候都会这么写

```js
new Vue({
    render: h => h(App)
})
```

这里的 `vm._renderProxy` 我们可以看作 vm 实例本身，也就是说实际上我们是透过 `h` 也就是所谓的 `vm.$createElemnt` 来创建我们虚拟 DOM 节点，下面我们来看看 `vm.$createElement` 方法具体是如何创建虚拟节点的。

##### 2.0.1.1 注入 Vue.prototype.$createElement

要使用 `vm.$createElement` 方法首先当然要先注入，看到 `initRender` 方法：

- `/src/core/instance/render.js`(阅读笔记文件路径：`/src/core/instance/render/initRender.flat2.js`)

```js
export function initRender (vm: Component) {
  // ...

  /* 挂载创建虚拟 dom 节点的方法 */
  // args order: tag, data, children, normalizationType, alwaysNormalize
  vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false)
  // 暴露接口
  vm.$createElement = (a, b, c, d) => createElement(vm, a, b, c, d, true)

  // ...
}
```

我们可以看到 `vm.$createElement` 其实是对 `createElement` 方法又套了层壳，继续看下去

- `/src/core/vdom/create-element.js`(阅读笔记文件路径：`/src/core/vdom/create-element/createElement.js`)

```js
// 创建虚拟节点 VNode 包装方法

export function createElement (
  context: Component,
  tag: any,
  data: any,
  children: any,
  normalizationType: any,
  alwaysNormalize: boolean
): VNode | Array<VNode> {
  // 检查 data 是否为数组，改成子节点数组
  if (Array.isArray(data) || isPrimitive(data)) {
    normalizationType = children
    children = data
    data = undefined
  }
  if (isTrue(alwaysNormalize)) {
    normalizationType = ALWAYS_NORMALIZE
  }
  return _createElement(context, tag, data, children, normalizationType)
}
```

我们可以看到其实 `createElement` 也只是对 `_createElement` 进行封装，再接着看。

##### 2.0.1.2 _createElement 细节

- `/src/core/vdom/create-element.js`(阅读笔记文件路径：`/src/core/vdom/create-element/_createElement.js`)

```js

export function _createElement (
  context: Component,
  tag?: string | Class<Component> | Function | Object,
  data?: VNodeData,
  children?: any,
  normalizationType?: number
): VNode | Array<VNode> {
  // observed data object as vnode data warning ...

  // data.is 替换 tag
  if (isDef(data) && isDef(data.is)) {
    tag = data.is
  }
  if (!tag) {
    // is 指向不合法元素
    return createEmptyVNode()
  }

  // non-primitive key warning ...

  // 还没搞懂。。。
  // support single function children as default scoped slot
  if (Array.isArray(children) &&
    typeof children[0] === 'function'
  ) {
    data = data || {}
    data.scopedSlots = { default: children[0] }
    children.length = 0
  }
  // 子节点数组标准化
  if (normalizationType === ALWAYS_NORMALIZE) {
    children = normalizeChildren(children)
  } else if (normalizationType === SIMPLE_NORMALIZE) {
    children = simpleNormalizeChildren(children)
  }
  let vnode, ns
  if (typeof tag === 'string') {
    let Ctor
    ns = (context.$vnode && context.$vnode.ns) || config.getTagNamespace(tag)
    if (config.isReservedTag(tag)) {
      /* 平台内置元素节点 */
      
      // .native for built-in elements warning ...

      vnode = new VNode(
        config.parsePlatformTagName(tag), data, children,
        undefined, undefined, context
      )
    } else if ((!data || !data.pre) && isDef(Ctor = resolveAsset(context.$options, 'components', tag))) {
      /* 组件节点 */
      vnode = createComponent(Ctor, data, context, children, tag)
    } else {
      /* 未知节点类型，运行时从命名空间(ns)查找 */
      vnode = new VNode(
        tag, data, children,
        undefined, undefined, context
      )
    }
  } else {
    /* 组件节点(直接使用组件 options & constructor 构建) */
    vnode = createComponent(tag, data, context, children)
  }
  if (Array.isArray(vnode)) {
    /* 返回节点数组 */
    return vnode
  } else if (isDef(vnode)) {
    /* 返回单根节点 */
    if (isDef(ns)) applyNS(vnode, ns)  // 申请命名空间
    if (isDef(data)) registerDeepBindings(data)  // 数据绑定
    return vnode
  } else {
    /* 返回空节点 */
    return createEmptyVNode()
  }
}
```

第一步骤首先会检查有没有使用 `:is="Xxx"`，替换为 `tag`

```js
// data.is 替换 tag
if (isDef(data) && isDef(data.is)) {
  tag = data.is
}
if (!tag) {
  // is 指向不合法元素
  return createEmptyVNode()
}
```

再来就是根据 tag 区分为组件节点 or 平台内置元素节点来创建 VNode 虚拟节点

```js
if (typeof tag === 'string') {
  let Ctor
  ns = (context.$vnode && context.$vnode.ns) || config.getTagNamespace(tag)
  if (config.isReservedTag(tag)) {
    /* 平台内置元素节点 */
    
    // .native for built-in elements warning ...

    vnode = new VNode(
      config.parsePlatformTagName(tag), data, children,
      undefined, undefined, context
    )
  } else if ((!data || !data.pre) && isDef(Ctor = resolveAsset(context.$options, 'components', tag))) {
    /* 组件节点 */
    vnode = createComponent(Ctor, data, context, children, tag)
  } else {
    /* 未知节点类型，运行时从命名空间(ns)查找 */
    vnode = new VNode(
      tag, data, children,
      undefined, undefined, context
    )
  }
} else {
  /* 组件节点(直接使用组件 options & constructor 构建) */
  vnode = createComponent(tag, data, context, children)
}
```

最后根据节点类型会再申请命名空间或是进行数据对象的绑定

```js
if (Array.isArray(vnode)) {
  /* 返回节点数组 */
  return vnode
} else if (isDef(vnode)) {
  /* 返回单根节点 */
  if (isDef(ns)) applyNS(vnode, ns)  // 申请命名空间
  if (isDef(data)) registerDeepBindings(data)  // 数据绑定
  return vnode
} else {
  /* 返回空节点 */
  return createEmptyVNode()
}
```

与 VNode 的创建相关的我会摆到下一篇模版编译的部分再详细说明，这边有个大概就行了(VNode 节点上的属性对应真实写法的部分可能需要猜一下，不过到下一篇的模版编译篇就能够明白 Vue 的完整样貌了)。

#### 2.0.2 _update 比较并更新 DOM 后进行渲染

我们透过 `_render` 方法拿到当前数据对应的虚拟 DOM 之后，接下来就会调用 `_update` 方法前面提过的：

```js
updateComponent = () => {
  vm._update(vm._render(), hydrating)
}
```

接着我们看看 `_update` 内部具体干了啥(`_update` 方法是在 `lifecycleMixin` 方法调用的时候注入的)

- `/src/core/instance/lifecycle.js`(阅读笔记文件路径：`/src/core/instance/lifecycle/lifecycleMixin.flat2._update.js`)

```js
// Vue.prototype._update 更新 vdom 方法

export function lifecycleMixin (Vue: Class<Component>) {
  Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {
    const vm: Component = this
    const prevEl = vm.$el
    const prevVnode = vm._vnode
    const restoreActiveInstance = setActiveInstance(vm)
    vm._vnode = vnode

    /* __patch__ 方法比较并更新渲染树 */
    if (!prevVnode) {
      // 首次渲染
      vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */)
    } else {
      // 后续更新
      vm.$el = vm.__patch__(prevVnode, vnode)
    }
    restoreActiveInstance()
    /* 更新 __vue__ 标志 */
    if (prevEl) {
      prevEl.__vue__ = null
    }
    if (vm.$el) {
      vm.$el.__vue__ = vm
    }
    /* 更新高阶组件 */
    if (vm.$vnode && vm.$parent && vm.$vnode === vm.$parent._vnode) {
      vm.$parent.$el = vm.$el
    }
    /* call updated hook by scheduler */
  }

  Vue.prototype.$forceUpdate = function () {
    const vm: Component = this
    if (vm._watcher) {
      vm._watcher.update()
    }
  }

  Vue.prototype.$destroy = function () {/* ... */}
}
```

我们可以看到 `_update` 的核心为下列这一句：

```js
/* __patch__ 方法比较并更新渲染树 */
if (!prevVnode) {
  // 首次渲染
  vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */)
} else {
  // 后续更新
  vm.$el = vm.__patch__(prevVnode, vnode)
}
```

我们看到 Vue 其实还针对第一次渲染跟后续更新进行区分，一个传入原始模版(`vm.$el`)，一个直接传入旧的虚拟节点树。两种都是使用了一个叫 `vm.__patch__` 方法来进行比较之后更新真实 DOM，最后返回的 DOM 树放回 `vm.$el` 上

#### 2.0.3 diff 流程小结

到此我们可以做一个小结：Vue 每次更新组件(`updateComponent`)的时候的大致流程如下：

![](https://picures.oss-cn-beijing.aliyuncs.com/img/vue2_virtual_dom_diff_updateComponent.png)

- `_render` 创建新的虚拟 DOM 树
- `_update` 比较新旧 DOM 树并更新真实 DOM 节点

### 2.1 注入 Vue.prototype.\_\_patch\_\_

下面我们来看看最核心的 `Vue.prototype.__patch__` 方法的注入

- `/src/platforms/web/runtime/index.js`(阅读笔记文件路径：`/src/platforms/web/runtime/index.js`)

```js
/* @flow */

import Vue from 'core/index'
import config from 'core/config'
import { extend, noop } from 'shared/util'
import { mountComponent } from 'core/instance/lifecycle'
import { devtools, inBrowser } from 'core/util/index'

import {
  query,
  mustUseProp,
  isReservedTag,
  isReservedAttr,
  getTagNamespace,
  isUnknownElement
} from 'web/util/index'

import { patch } from './patch'
import platformDirectives from './directives/index'
import platformComponents from './components/index'

// 挂载运行时环境相关工具
Vue.config.mustUseProp = mustUseProp
Vue.config.isReservedTag = isReservedTag
Vue.config.isReservedAttr = isReservedAttr
Vue.config.getTagNamespace = getTagNamespace
Vue.config.isUnknownElement = isUnknownElement

// 添加命令、组件扩展
extend(Vue.options.directives, platformDirectives)
extend(Vue.options.components, platformComponents)

// 注入 Vue.prototype.__patch__ 实例更新方法
Vue.prototype.__patch__ = inBrowser ? patch : noop

// 注入 Vue.prototype.$mount
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && inBrowser ? query(el) : undefined
  return mountComponent(this, el, hydrating)
}

// web 版本提示：devtools、development ...

export default Vue
```

Vue 将 `Vue.prototype.__patch__` 方法在 `web/runtime/` 包下引入，因为这个方法是需要更真实 DOM 强相关的，所以需要根据具体平台绑定不同的方法和模块

### 2.2 patch 方法的构成

而挂载到 `Vue.prototype.__patch__` 的 `patch` 方法则是透过下列的方式创建的

- `/src/platforms/web/runtime/patch.js`(阅读笔记文件路径：`/src/platforms/web/runtime/patch.js`)

```js
/* @flow */

import * as nodeOps from 'web/runtime/node-ops'
import { createPatchFunction } from 'core/vdom/patch'
import baseModules from 'core/vdom/modules/index'
import platformModules from 'web/runtime/modules/index'

// 收集所有依赖模块
const modules = platformModules.concat(baseModules)

// 创建 web 版本的运行时 patch
export const patch: Function = createPatchFunction({ nodeOps, modules })
```

我们可以看到这个 `patch` 方法是透过一个 `createPatchFunction` 方法创建的，该方法会根据传入的 `nodeOps` 与 `modules` 来绑定(使用闭包)具体的 DOM 操作方法和模块更新方法(下面会再提到这里所谓的模块指的是什么)

#### 2.2.1 node-ops 平台相关具体节点方法

在看到 `createPatchFunction` 方法具体是如何创建 `patch` 方法之前，我们先来看看他依赖的 `nodeOps, moudles` 是什么东西

- `/src/platforms/web/runtime/node-ops.js`(阅读笔记文件路径：`/src/platforms/web/runtime/node-ops.js`)

```js
/* @flow */
/* 真实 dom 节点工厂方法 */

import { namespaceMap } from 'web/util/index'

/* 创建元素节点 */
export function createElement (tagName: string, vnode: VNode): Element {
  const elm = document.createElement(tagName)
  if (tagName !== 'select') {
    return elm
  }
  // false or null will remove the attribute but undefined will not
  if (vnode.data && vnode.data.attrs && vnode.data.attrs.multiple !== undefined) {
    elm.setAttribute('multiple', 'multiple')
  }
  return elm
}

/* 创建元素命名空间 */
export function createElementNS (namespace: string, tagName: string): Element {
  return document.createElementNS(namespaceMap[namespace], tagName)
}

/* 创建文本节点 */
export function createTextNode (text: string): Text {
  return document.createTextNode(text)
}

/* 创建注释节点 */
export function createComment (text: string): Comment {
  return document.createComment(text)
}

/* 插入到节点之前 */
export function insertBefore (parentNode: Node, newNode: Node, referenceNode: Node) {
  parentNode.insertBefore(newNode, referenceNode)
}

/* 移除子节点 */
export function removeChild (node: Node, child: Node) {
  node.removeChild(child)
}

/* 添加子节点 */
export function appendChild (node: Node, child: Node) {
  node.appendChild(child)
}

/* 获取父节点 */
export function parentNode (node: Node): ?Node {
  return node.parentNode
}

/* 获取下一个兄弟节点 */
export function nextSibling (node: Node): ?Node {
  return node.nextSibling
}

/* 节点标签 */
export function tagName (node: Element): string {
  return node.tagName
}

/* 设置节点文本内容 */
export function setTextContent (node: Node, text: string) {
  node.textContent = text
}

/* 设置节点属性 scopeId */
export function setStyleScope (node: Element, scopeId: string) {
  node.setAttribute(scopeId, '')
}
```

首先我们以 `web/runtime` 举例，`nodeOps` 对浏览器平台下的 DOM 操作方法进行封装(也就是说使用相同命名的基础下可以更改每个对真实 DOM 操作细节)

后面我们就会看到 `patch` 方法内部经常会使用 `nodeOps.xXxx` 来对真实 DOM 进行操作/更新

#### 2.2.2 modules 节点管理模块

第二个依赖参数则是 modules 模块，我们可以看到么模块又分为 baseModules 与 platformModules

```js
// 收集所有依赖模块
const modules = platformModules.concat(baseModules)
```

这里所谓的模块是 Vue 将 DOM 节点上的特殊属性、指令的处理特别区分成多个模块，由不同模块共同合作来完成整个虚拟 DOM 的构建和更新(具体更新机制是下面会再提到的 **hook 钩子** )

##### 2.2.2.1 baseModules 核心模块：refs、directives

第一个 baseModules 核心模块是与平台无关的，可以看作就是 Vue 提供的核心能力，分为 `refs, directives` 也就是对应 Vue 的 `ref="xxx"` 与 `Vue.directive()` 两个能力

- `/src/core/vdom/modules/index.js`(阅读笔记文件路径：`/src/core/vdom/modules/index.js`)

```js
import directives from './directives'
import ref from './ref'

// ref, directives 模块
export default [
  ref,
  directives
]
```

我们可以看到整个 baseModules 其实是一个数组，包含 ref、directives 两个模块

我们拿 ref 模块来举例

- `/src/core/vdom/modules/ref.js`(阅读笔记文件路径：`/src/core/vdom/modules/ref.flat1.js`)

```js
/* @flow */

import { remove, isDef } from 'shared/util'

/* ref 模块(ref 引用属性处理) */
export default {
  // 创建周期处理 ref 引用
  create (_: any, vnode: VNodeWithData) {
    registerRef(vnode)
  },
  // 更新周期处理 ref 引用
  update (oldVnode: VNodeWithData, vnode: VNodeWithData) {
    if (oldVnode.data.ref !== vnode.data.ref) {
      registerRef(oldVnode, true)
      registerRef(vnode)
    }
  },
  // 销毁周期处理 ref 引用
  destroy (vnode: VNodeWithData) {
    registerRef(vnode, true)
  }
}

/* 注册 ref 引用 */
export function registerRef (vnode: VNodeWithData, isRemoval: ?boolean) {/* ... */}
```

我们可以看到 ref 的核心其实对应了三个方法 `create, update, destroy`，而三个方法其实都对应一些 `registerRef` 方法调用的组合，透过些方法来完成对于 DOM 节点的 `ref` 属性的维护

##### 2.2.2.2 platformModules 平台模块：attrs、klass、events、domProps、style、transition

而平台相关的 platformModules 模块则是由以下模块组成

- `/src/platforms/web/runtime/modules/index.js`(阅读笔记文件路径：`/src/platforms/web/runtime/modules/index.js`)

```js
import attrs from './attrs'
import klass from './class'
import events from './events'
import domProps from './dom-props'
import style from './style'
import transition from './transition'

/* platformModules 模块 (作为 patch 函数的 modules 模块选项) */
export default [
  attrs,
  klass,
  events,
  domProps,
  style,
  transition
]
```

我们可以看到平台相关的模块就是一些 DOM 上常见的属性

- `attrs` 对应其他自定义属性
- `klass` 对应 class 属性
- `events` 负责事件处理器的维护
- ... 

那么现在的问题是，不论是 baseModules 还是 platformModules 提供了很多模块，在 Vue 中是什么时候调用这些模块呢？

##### 2.2.2.3 hook 模块钩子

这里 Vue 作者利用了一个 **hook 钩子** 的机制。由于如果我们在具体 DOM 上一个个插入模块的调用更新会显得非常臃肿，而且极容易产生漏洞，也难以定位/维护更新问题(方法调用非常多同时也容易漏写)，因此透过定义一个 hook 的钩子

- `/src/core/vdom/patch.js`(阅读笔记文件路径：`/src/core/vdom/patch/createPatchFunction.flat2.js`)

```js
// vnode 生命周期钩子
const hooks = ['create', 'activate', 'update', 'remove', 'destroy']
```

并且在创建 `patch` 方法的时候先收集所有模块的各个钩子函数

```js
// 收集 vnode 生命周期相关回调
let i, j
const cbs = {}

const { modules, nodeOps } = backend

for (i = 0; i < hooks.length; ++i) {
  cbs[hooks[i]] = []
  for (j = 0; j < modules.length; ++j) {
    if (isDef(modules[j][hooks[i]])) {
      cbs[hooks[i]].push(modules[j][hooks[i]])
    }
  }
}
```

这样在具体 `patch` 的过程中，我们只要在正确的未知调用所有钩子(`cbs`)，就能够确保各个模块都进行相应的更新而不用修改具体的模块更新方法调用。

#### 2.2.3 createPatchFunction 构建patch函数

我们再回到 `patch` 方法被创建的时候

- `/src/platforms/web/runtime/patch.js`(阅读笔记文件路径：`/src/platforms/web/runtime/patch.js`)

```js
/* @flow */

import * as nodeOps from 'web/runtime/node-ops'
import { createPatchFunction } from 'core/vdom/patch'
import baseModules from 'core/vdom/modules/index'
import platformModules from 'web/runtime/modules/index'

// 收集所有依赖模块
const modules = platformModules.concat(baseModules)

// 创建 web 版本的运行时 patch
export const patch: Function = createPatchFunction({ nodeOps, modules })
```

我们已经知道 `createPatchFunction` 依赖的两个对象各是什么意思：

- nodeOps: 平台相关的具体节点操作方法
- modules: 各个节点部件操作模块

下面我们先来看看 `createPatchFunction` 方法的架构

- `/src/core/vdom/patch.js`(阅读笔记文件路径：`/src/core/vdom/patch/flat1.js`)

```js
// 虚拟 dom 比较算法

import VNode, { cloneVNode } from './vnode'
import config from '../config'
import { SSR_ATTR } from 'shared/constants'
import { registerRef } from './modules/ref'
import { traverse } from '../observer/traverse'
import { activeInstance } from '../instance/lifecycle'
import { isTextInputType } from 'web/util/element'

import {
  warn,
  isDef,
  isUndef,
  isTrue,
  makeMap,
  isRegExp,
  isPrimitive
} from '../util/index'

export const emptyNode = new VNode('', {}, [])

// 生命周期钩子
const hooks = ['create', 'activate', 'update', 'remove', 'destroy']

/* 比较节点 */
function sameVnode (a, b) {/* ... */}

/* 比较输入类型 */
function sameInputType (a, b) {/* ... */}

/* 建立 key: index 的映射 */
function createKeyToOldIdx (children, beginIdx, endIdx) {/* ... */}

/* 生成 patch 函数 */
export function createPatchFunction (backend) {/* ... */}
```

- `/src/core/vdom/patch.js`(阅读笔记文件路径：`/src/core/vdom/patch/createPatchFunction.flat2.js`)

```js
import VNode, { cloneVNode } from './vnode'
import config from '../config'
import { SSR_ATTR } from 'shared/constants'
import { registerRef } from './modules/ref'
import { traverse } from '../observer/traverse'
import { activeInstance } from '../instance/lifecycle'
import { isTextInputType } from 'web/util/element'

import {
  warn,
  isDef,
  isUndef,
  isTrue,
  makeMap,
  isRegExp,
  isPrimitive
} from '../util/index'

// vnode 生命周期钩子
const hooks = ['create', 'activate', 'update', 'remove', 'destroy']

/* 生成 patch 函数 */
export function createPatchFunction (backend) {

  // 收集 vnode 生命周期相关回调
  let i, j
  const cbs = {}

  const { modules, nodeOps } = backend

  for (i = 0; i < hooks.length; ++i) {
    cbs[hooks[i]] = []
    for (j = 0; j < modules.length; ++j) {
      if (isDef(modules[j][hooks[i]])) {
        cbs[hooks[i]].push(modules[j][hooks[i]])
      }
    }
  }

  /* 创建并返回空节点 */
  function emptyNodeAt (elm) {/* ... */}

  /* 创建移除监听器回调 */
  function createRmCb (childElm, listeners) {/* ... */}

  /* 移除旧节点 */
  function removeNode (el) {/* ... */}

  /* 判断是否为未知元素节点 */
  function isUnknownElement (vnode, inVPre) {/* ... */}

  let creatingElmInVPre = 0

  /* 创建元素 */
  function createElm (/* ... */) {/* ... */}

  /* 创建组件(子组件) */
  function createComponent (vnode, insertedVnodeQueue, parentElm, refElm) {/* ... */}

  /* 初始化组件(初始化 data, create 钩子, CSS scopeId, ref) */
  function initComponent (vnode, insertedVnodeQueue) {/* ... */}

  /* 激活已创建组件节点 */
  function reactivateComponent (vnode, insertedVnodeQueue, parentElm, refElm) {/* ... */}

  /* 将 elm 插入 parent(插到 ref 前 or 直接插入) */
  function insert (parent, elm, ref) {/* ... */}

  /* 递归创建子节点 */
  function createChildren (vnode, children, insertedVnodeQueue) {/* ... */}

  /* 检查是否有 tag(即 isRealElement) */
  function isPatchable (vnode) {/* ... */}

  /* 触发 create 钩子 */
  function invokeCreateHooks (vnode, insertedVnodeQueue) {/* ... */}

  /* 设置 CSS scoped Id */
  function setScope (vnode) {/* ... */}

  /* 直接添加剩余新节点 */
  function addVnodes (parentElm, refElm, vnodes, startIdx, endIdx, insertedVnodeQueue) {/* ... */}

  /* 触发 destroy 钩子 */
  function invokeDestroyHook (vnode) {/* ... */}

  /* 直接移除剩余旧节点 */
  function removeVnodes (vnodes, startIdx, endIdx) {/* ... */}

  /* 移除节点并触发 remove 钩子 */
  function removeAndInvokeRemoveHook (vnode, rm) {/* ... */}

  /* 递归更新子数组(patchVnode 内部调用) */
  function updateChildren (parentElm, oldCh, newCh, insertedVnodeQueue, removeOnly) {/* ... */}

  /* 检查子节点是否存在重复 key */
  function checkDuplicateKeys (children) {/* ... */}

  /* 从旧数组中查找节点 */
  function findIdxInOld (node, oldCh, start, end) {/* ... */}

  /* 比较新旧节点差异并更新 */
  function patchVnode (/* ... */) {/* ... */}

  /* 触发 insert 钩子 */
  function invokeInsertHook (vnode, queue, initial) {/* ... */}

  let hydrationBailed = false

  const isRenderedModule = makeMap('attrs,class,staticClass,staticStyle,key')

  function hydrate (elm, vnode, insertedVnodeQueue, inVPre) {/* ... */}

  function assertNodeMatch (node, vnode, inVPre) {/* ... */}

  return function patch (oldVnode, vnode, hydrating, removeOnly) {/* ... */}
}
```

我们可以看到 `createPatchFunction` 方法的最后其实就是返回一个 `patch` 函数，而 `patch` 函数则是透过闭包的方式调用一些内部方法，而这些内部方法则是与具体平台节点操作(nodeOps)和节点部件模块(modules)相关的

### 2.3 明确 patch 目标 & 区分操作类型

`createPatchFunction` 方法内部这一大堆内部方法着实让人头疼，还是比较难解析。所以我们先不硬爆，我们先回头看看明确一下我们的 `patch` 方法到底是为了完成什么事、可以拆分成哪几种方法。

还记得 `_update` 方法内我们是这么调用 `Vue.prototype.__patch__` 方法的

- `/src/core/instance/lifecycle.js`(阅读笔记文件路径：`/src/core/instance/lifecycle/lifecycleMixin.flat2._update.js`)

```js
/* __patch__ 方法比较并更新渲染树 */
if (!prevVnode) {
  // 首次渲染
  vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */)
} else {
  // 后续更新
  vm.$el = vm.__patch__(prevVnode, vnode)
}
```

第一个参数 `vm.$el / preVnode` 为旧的 DOM 节点，而 `vnode` 则是前面调用 `_render` 生成的新的虚拟 DOM 节点。

也就是说，`patch` 方法的核心目标就是 **比较新旧虚拟 DOM 并更新真实 DOM 节点**

这时候我们其实可以把所谓的 **比较 & 更新** 操作区分为三种行为：

- 创建节点：旧 VNode 不存在的节点需要被创建
- 删除节点：新 Vnode 不存在的节点需要被移除
- 比较更新节点：新旧 VNode 都存在，进行深度比较和更新

而第三种的 **比较更新节点** 内部又会递归调用三种操作进行创建/删除/更新节点的操作，下面我们就一个个来看三种操作的具体流程

#### 2.3.1 createElm 创建 DOM 节点

首先第一种 **创建节点** 对应了 `createPatchFunction` 方法闭包内的 `createElm` 方法：

- `/src/core/vdom/patch.js`(阅读笔记文件路径：`/src/core/vdom/patch/createPatchFunction.flat2.createElm.flat3.js`)

```js
export function createPatchFunction (backend) {

  // ...

  /* 创建元素 */
  function createElm (
    vnode,
    insertedVnodeQueue,
    parentElm,
    refElm,
    nested,
    ownerArray,
    index
  ) {
    if (isDef(vnode.elm) && isDef(ownerArray)) {
      // This vnode was used in a previous render!
      // now it's used as a new node, overwriting its elm would cause
      // potential patch errors down the road when it's used as an insertion
      // reference node. Instead, we clone the node on-demand before creating
      // associated DOM element for it.
      vnode = ownerArray[index] = cloneVNode(vnode)
    }

    vnode.isRootInsert = !nested // for transition enter check
    if (createComponent(vnode, insertedVnodeQueue, parentElm, refElm)) {
      return
    }

    const data = vnode.data
    const children = vnode.children
    const tag = vnode.tag
    if (isDef(tag)) {
      /* 一般元素节点(Element) */
    } else if (isTrue(vnode.isComment)) {
      /* 注释节点(Comment) */
    } else {
      /* 文本节点(Text) */
    }
  }

  // ...

}
```

首先对于创建 DOM 节点的操作我们又可以区分为三种节点：

- 一般元素节点
- 注释节点
- 文本节点

三种节点可以透过 VNode 上的 `tag`、`isComment` 属性来区分

下面我们再看到更详细的操作

- `/src/core/vdom/patch.js`(阅读笔记文件路径：`/src/core/vdom/patch/createPatchFunction.flat2.createElm.flatx.js`)

```js
import VNode, { cloneVNode } from './vnode'
import config from '../config'
import { SSR_ATTR } from 'shared/constants'
import { registerRef } from './modules/ref'
import { traverse } from '../observer/traverse'
import { activeInstance } from '../instance/lifecycle'
import { isTextInputType } from 'web/util/element'

import {
  warn,
  isDef,
  isUndef,
  isTrue,
  makeMap,
  isRegExp,
  isPrimitive
} from '../util/index'

export function createPatchFunction (backend) {

  // ...

  function createElm (
    vnode,
    insertedVnodeQueue,
    parentElm,
    refElm,
    nested,
    ownerArray,
    index
  ) {
    if (isDef(vnode.elm) && isDef(ownerArray)) {
      // 已经渲染过的需要创建克隆节点，否则比较时对树遍历会出问题
      vnode = ownerArray[index] = cloneVNode(vnode)
    }

    vnode.isRootInsert = !nested  // 检查是否为
    if (createComponent(vnode, insertedVnodeQueue, parentElm, refElm)) {
      // 作为子节点时直接返回
      return
    }

    const data = vnode.data
    const children = vnode.children
    const tag = vnode.tag
    if (isDef(tag)) {
      /* 一般元素节点(Element) */

      // unknown element warning ...

      vnode.elm = vnode.ns
        ? nodeOps.createElementNS(vnode.ns, tag)
        : nodeOps.createElement(tag, vnode)
      setScope(vnode)

      if (__WEEX__) {
        /* weex environment ... */
      } else {
        // 创建子节点
        createChildren(vnode, children, insertedVnodeQueue)
        if (isDef(data)) {
          // 触发 create 钩子
          invokeCreateHooks(vnode, insertedVnodeQueue)
        }
        // 插入父节点
        insert(parentElm, vnode.elm, refElm)
      }

      // creatingElmInVPre update(in production) ...
    } else if (isTrue(vnode.isComment)) {
      /* 注释节点(Comment) */
      vnode.elm = nodeOps.createComment(vnode.text)
      insert(parentElm, vnode.elm, refElm)
    } else {
      /* 文本节点(Text) */
      vnode.elm = nodeOps.createTextNode(vnode.text)
      insert(parentElm, vnode.elm, refElm)
    }
  }

  // ...

}
```

对于注释节点和文本节点，我们可以看到其实就是直接调用 nodeOps 创建元素之后插入父节点即可

```js
// ...

} else if (isTrue(vnode.isComment)) {
  /* 注释节点(Comment) */
  vnode.elm = nodeOps.createComment(vnode.text)
  insert(parentElm, vnode.elm, refElm)
} else {
  /* 文本节点(Text) */
  vnode.elm = nodeOps.createTextNode(vnode.text)
  insert(parentElm, vnode.elm, refElm)
}

// ...
```

而对于元素节点我们则需要调用 `nodeOps.createElementNS/createElement` 创建节点之后调用 `createChildren` 来创建子节点数组

```js
// ...

if (isDef(tag)) {
  /* 一般元素节点(Element) */

  // unknown element warning ...

  vnode.elm = vnode.ns
    ? nodeOps.createElementNS(vnode.ns, tag)
    : nodeOps.createElement(tag, vnode)
  setScope(vnode)

  if (__WEEX__) {
    /* weex environment ... */
  } else {
    // 创建子节点
    createChildren(vnode, children, insertedVnodeQueue)
    if (isDef(data)) {
      // 触发 create 钩子
      invokeCreateHooks(vnode, insertedVnodeQueue)
    }
    // 插入父节点
    insert(parentElm, vnode.elm, refElm)
  }

// ...
```

##### 2.3.1.1 createComponent 尝试创建组件节点

我们好像遗漏了一句话

```js
// ...

if (createComponent(vnode, insertedVnodeQueue, parentElm, refElm)) {
  // 作为组件节点时直接返回
  return
}

// ...
```

我们可以看到在真实创三种不同节点之前其实调用了一个 `createComponent` 来尝试创建组件节点

- `/src/core/vdom/patch.js`(阅读笔记文件路径：`/src/core/vdom/patch/createPatchFunction.flat2.createComponent.js`)

```js
export function createPatchFunction (backend) {

  // ...

  /* 创建组件节点 */
  function createComponent (vnode, insertedVnodeQueue, parentElm, refElm) {
    // 有数据项
    let i = vnode.data
    if (isDef(i)) {
      const isReactivated = isDef(vnode.componentInstance) && i.keepAlive
      if (isDef(i = i.hook) && isDef(i = i.init)) {
        // 调用 init 钩子
        i(vnode, false /* hydrating */)
      }

      // 如果 Vue 实例已经存在(表示当前为组件节点)
      if (isDef(vnode.componentInstance)) {
        // 初始化子节点组件
        initComponent(vnode, insertedVnodeQueue)
        // 将子组件插入父节点
        insert(parentElm, vnode.elm, refElm)
        if (isTrue(isReactivated)) {
          // 激活组件
          reactivateComponent(vnode, insertedVnodeQueue, parentElm, refElm)
        }
        return true
      }
    }
  }

  // ...

}
```

我们看到它会检查 VNode 上的 data 数据项

```js
let i = vnode.data
if (isDef(i)) {
    /* ... */
}
```

也就是说其实我们隐隐约约能猜测出对于 Vue 组件会经过编译之后插入到 vnode.data 上，所以我们可以透过该属性来判断是不是一个 Vue 组件

内部的定义则比较简单明了了

```js
const isReactivated = isDef(vnode.componentInstance) && i.keepAlive
if (isDef(i = i.hook) && isDef(i = i.init)) {
  // 调用 init 钩子
  i(vnode, false /* hydrating */)
}

// 如果 Vue 实例已经存在(表示当前为组件节点)
if (isDef(vnode.componentInstance)) {
  // 初始化组件节点
  initComponent(vnode, insertedVnodeQueue)
  // 将子组件插入父节点
  insert(parentElm, vnode.elm, refElm)
  if (isTrue(isReactivated)) {
    // 激活组件
    reactivateComponent(vnode, insertedVnodeQueue, parentElm, refElm)
  }
  return true
}
```

首先调用前面提过的 modules 的 insert 钩子，接下来调用 `initComponent` 初始化组件，并可选的调用 `reactivateComponent` 来激活已存在的组件

```js
return true
```

注意这边返回的一个 `true`，还记得上面有这么一句话

```js
if (createComponent(vnode, insertedVnodeQueue, parentElm, refElm)) {
  // 作为组件节点时直接返回
  return
}
```

也就是说 `createElm` 方法会先调用 `createComponent` 尝试创建一个组件对象，如果不是组件才会直接去做下面三种实际节点的判断

```js
if (isDef(tag)) {
  /* 一般元素节点(Element) */
} else if (isTrue(vnode.isComment)) {
  /* 注释节点(Comment) */
} else {
  /* 文本节点(Text) */
}
```

##### 2.3.1.2 initComponent 初始化组件节点

下面我们再向内探一层，看看 `initComponent` 如何初始化组件节点

- `/src/core/vdom/patch.js`(阅读笔记文件路径：`/src/core/vdom/patch/createPatchFunction.flat2.initComponent.js`)

```js
export function createPatchFunction (backend) {

  // ...

  /* 初始化组件(初始化 data, create 钩子, CSS scopeId, ref) */
  function initComponent (vnode, insertedVnodeQueue) {
    if (isDef(vnode.data.pendingInsert)) {
      // 缓冲 data 队列(确保多层组件的 data 初始化顺序)
      insertedVnodeQueue.push.apply(insertedVnodeQueue, vnode.data.pendingInsert)
      vnode.data.pendingInsert = null
    }
    vnode.elm = vnode.componentInstance.$el
    if (isPatchable(vnode)) {
      // 调用组件的 create 钩子，并设置 CSS scopeId
      invokeCreateHooks(vnode, insertedVnodeQueue)
      setScope(vnode)
    } else {
      // 重新注册 ref
      registerRef(vnode)
      insertedVnodeQueue.push(vnode)
    }
  }

  // ...

}
```

`initComponent` 方法会将组件本身加入 `insertedVnodeQueue` 队列并刷新 `pendingInsert` 状态属性；接下来是根据 `isPatchable` 方法判断是否为真实组件选择调用 `invokeCreateHooks` 触发模块的 create 钩子，或是简单调用 `registerRef` 只更新 ref 模块的注册

##### 2.3.1.3 invokeCreateHooks 调用节点模块 create 钩子

这边我们第一次遇到了组件生命周期钩子的调用，这边以 create 钩子举例来看看上面提到的节点部件模块的钩子是如何被调用的

- `/src/core/vdom/patch.js`(阅读笔记文件路径：`/src/core/vdom/patch/createPatchFunction.flat2.invokeCreateHooks.js`)

```js
export function createPatchFunction (backend) {

  // ...

  /* 触发 create 钩子 */
  function invokeCreateHooks (vnode, insertedVnodeQueue) {
    // 调用所有模块的 create 回调
    for (let i = 0; i < cbs.create.length; ++i) {
      cbs.create[i](emptyNode, vnode)
    }
    // 调用 vnode.data.hook 上的 create 钩子，并插入 insertedVnodeQueue 队列 
    i = vnode.data.hook
    if (isDef(i)) {
      if (isDef(i.create)) i.create(emptyNode, vnode)
      if (isDef(i.insert)) insertedVnodeQueue.push(vnode)
    }
  }

  // ...

}
```

其实就是循环前面已经收集到 `cbs` 的所有钩子

##### 2.3.1.4 createChildren 创建子组件列表

最后一个则是来说明一下针对非组件节点进行创建的时候调用的一个 `createChildren` 方法来创建子组件

- `/src/core/vdom/patch.js`(阅读笔记文件路径：`/src/core/vdom/patch/createElm.flatx.js`)

```js
// ...

vnode.elm = vnode.ns
  ? nodeOps.createElementNS(vnode.ns, tag)
  : nodeOps.createElement(tag, vnode)
setScope(vnode)

if (__WEEX__) {
  /* weex environment ... */
} else {
  // 创建子节点
  createChildren(vnode, children, insertedVnodeQueue)
  if (isDef(data)) {
    // 触发 create 钩子
    invokeCreateHooks(vnode, insertedVnodeQueue)
  }
  // 插入父节点
  insert(parentElm, vnode.elm, refElm)
}

// ...
```

- `/src/core/vdom/patch.js`(阅读笔记文件路径：`/src/core/vdom/patch/createPatchFunction.flat2.createChildren.js`)

```js
export function createPatchFunction (backend) {

  // ...

  /* 递归创建子节点 */
  function createChildren (vnode, children, insertedVnodeQueue) {
    if (Array.isArray(children)) {
      if (process.env.NODE_ENV !== 'production') {
        checkDuplicateKeys(children)
      }
      for (let i = 0; i < children.length; ++i) {
        // 递归创建子节点
        createElm(children[i], insertedVnodeQueue, vnode.elm, null, true, children, i)
      }
    } else if (isPrimitive(vnode.text)) {
      // 创建文本子节点
      nodeOps.appendChild(vnode.elm, nodeOps.createTextNode(String(vnode.text)))
    }
  }

  // ...

}
```

其实我们就可以看到 `createChildren` 是递归调用 `createElm` 来创建组件或是直接调用 `nodeOps.appendChild` 方法插入子节点

#### 2.3.2 removeNode 删除 DOM 节点

第二种行为 **删除节点** 就比较简单了(对应的是 `removeNode` 方法)

- `/src/core/vdom/patch.js`(阅读笔记文件路径：`/src/core/vdom/patch/createPatchFunction.flat2.removeNode.js`)

```js
export function createPatchFunction (backend) {

  // ...

  /* 移除旧节点 */
  function removeNode (el) {
    const parent = nodeOps.parentNode(el)
    // 使用 v-html / v-text 时不需要进行移除
    if (isDef(parent)) {
      nodeOps.removeChild(parent, el)
    }
  }

  // ...

}
```

既然我已经知道要删除节点了，检查一下 `parent` 指向正确就能够调用 `nodeOps.removeChild` 移除子节点了

#### 2.3.3 patch 比较并更新 DOM 树(\_\_patch\_\_ 方法的本体)

最后第三个操作 **比较更新节点** 则是作为 `__patch__` 方法的本体，也就是三种行为中最复杂的操作

- `/src/core/vdom/patch.js`(阅读笔记文件路径：`/src/core/vdom/patch/createPatchFunction.flat2.patch.js`)

```js
import VNode, { cloneVNode } from './vnode'
import config from '../config'
import { SSR_ATTR } from 'shared/constants'
import { registerRef } from './modules/ref'
import { traverse } from '../observer/traverse'
import { activeInstance } from '../instance/lifecycle'
import { isTextInputType } from 'web/util/element'

import {
  warn,
  isDef,
  isUndef,
  isTrue,
  makeMap,
  isRegExp,
  isPrimitive
} from '../util/index'

export function createPatchFunction (backend) {

  // ...

  /* 比较虚拟 dom 差异并返回合并后节点 */
  /**
   * case 1: 销毁节点
   * case 2: 新增节点
   * case 3: 深度 patch
   * case 4: 直接替换策略(服务端渲染 || hydrating === true)
   * case 5: 以新节点替换旧节点
   */
  return function patch (oldVnode, vnode, hydrating, removeOnly) {
    if (isUndef(vnode)) {
      /* case 1: oldVnode 存在、vnode 不存在 -> 销毁节点 */
      if (isDef(oldVnode)) invokeDestroyHook(oldVnode)
      return
    }

    let isInitialPatch = false
    const insertedVnodeQueue = []

    if (isUndef(oldVnode)) {
      /* case 2: oldVnode 不存在 -> 新增节点(首次渲染节点) */
      isInitialPatch = true
      createElm(vnode, insertedVnodeQueue)
    } else {
      const isRealElement = isDef(oldVnode.nodeType)
      if (!isRealElement && sameVnode(oldVnode, vnode)) {
        /* case 3: oldVnode.nodeType 存在 && 新旧节点一样 -> 深度比较根节点差异 */
        patchVnode(oldVnode, vnode, insertedVnodeQueue, null, null, removeOnly)
      } else {
        if (isRealElement) {
          if (oldVnode.nodeType === 1 && oldVnode.hasAttribute(SSR_ATTR)) {
            /* case 4: 服务端渲染直接强制替换旧节点 */
            oldVnode.removeAttribute(SSR_ATTR)
            hydrating = true
          }
          if (isTrue(hydrating)) {
            // 4.1 hydrating === true -> 直接替换旧节点策略
            if (hydrate(oldVnode, vnode, insertedVnodeQueue)) {
              // 调用 insert 钩子
              invokeInsertHook(vnode, insertedVnodeQueue, true)
              return oldVnode
            } else if (process.env.NODE_ENV !== 'production') {
              // 4.2 client-side render with hydrating === true warning ...
            }
          }
          // !4.1 hydrating 失败 -> 返回缺失节点
          oldVnode = emptyNodeAt(oldVnode)
        }
        /* case 5: 默认策略 -> 以新节点替换旧节点 */

        // 替换已存在 elm
        const oldElm = oldVnode.elm
        const parentElm = nodeOps.parentNode(oldElm)

        // 创建新节点
        createElm(
          vnode,
          insertedVnodeQueue,
          oldElm._leaveCb ? null : parentElm,
          nodeOps.nextSibling(oldElm)
        )

        // 如果父节点存在，递归替换位于父节点的位置
        if (isDef(vnode.parent)) {
          let ancestor = vnode.parent
          const patchable = isPatchable(vnode)
          while (ancestor) {
            // 首先调用父节点所有 destroy 钩子
            for (let i = 0; i < cbs.destroy.length; ++i) {
              cbs.destroy[i](ancestor)
            }
            // 放入新节点
            ancestor.elm = vnode.elm
            if (patchable) {
              /* 实节点 */
              // 调用所有 create 钩子
              for (let i = 0; i < cbs.create.length; ++i) {
                cbs.create[i](emptyNode, ancestor)
              }

              // 调用所有 data.hook.insert.fns 钩子              
              const insert = ancestor.data.hook.insert
              if (insert.merged) {
                for (let i = 1; i < insert.fns.length; i++) {
                  insert.fns[i]()
                }
              }
            } else {
              /* 虚节点 */
              // 仅记录 ref
              registerRef(ancestor)
            }
            ancestor = ancestor.parent
          }
        }

        // 销毁旧节点
        if (isDef(parentElm)) {
          // 父节点存在则移除旧节点即可
          removeVnodes([oldVnode], 0, 0)
        } else if (isDef(oldVnode.tag)) {
          // 触发旧节点的 destroy 生命周期钩子
          invokeDestroyHook(oldVnode)
        }
      }
    }

    // 触发新节点的 insert 生命周期钩子
    invokeInsertHook(vnode, insertedVnodeQueue, isInitialPatch)
    return vnode.elm
  }

  // ...

}
```

整个方法还是比较冗长的，我们将它拆成几种场景/阶段来看

1. case 1: 销毁节点

```js
if (isUndef(vnode)) {
   /* case 1: oldVnode 存在、vnode 不存在 -> 销毁节点 */
   if (isDef(oldVnode)) invokeDestroyHook(oldVnode)
   return
}
```

如果传入的 vnode 为空，则表示我们想要直接删除整个旧节点，也就是调用一下 `invokeDestroyHook` 就得了

2. case 2: 新增节点

```js
if (isUndef(oldVnode)) {
   /* case 2: oldVnode 不存在 -> 新增节点(首次渲染节点) */
   isInitialPatch = true
   createElm(vnode, insertedVnodeQueue)
   
   // ...
```

第二种情况则是 oldVnode 为空，则表示 vnode 是一个全新的节点，直接调用 `createElm` 进行创建

3. case 3: 深度 patch

```js
if (!isRealElement && sameVnode(oldVnode, vnode)) {
    /* case 3: oldVnode.nodeType 存在 && 新旧节点一样 -> 深度比较根节点差异 */
    patchVnode(oldVnode, vnode, insertedVnodeQueue, null, null, removeOnly)
```

第三种情况是新旧节点都存在，则调用 `patchVnode` 进行深度比较

4. case 4: 直接替换策略

```js
if (isRealElement) {
  if (oldVnode.nodeType === 1 && oldVnode.hasAttribute(SSR_ATTR)) {
    /* case 4: 服务端渲染直接强制替换旧节点 */
    oldVnode.removeAttribute(SSR_ATTR)
    hydrating = true
  }
  if (isTrue(hydrating)) {
    // 4.1 hydrating === true -> 直接替换旧节点策略
    if (hydrate(oldVnode, vnode, insertedVnodeQueue)) {
      // 调用 insert 钩子
      invokeInsertHook(vnode, insertedVnodeQueue, true)
      return oldVnode
    } else if (process.env.NODE_ENV !== 'production') {
      // 4.2 client-side render with hydrating === true warning ...
    }
  }
  // !4.1 hydrating 失败 -> 返回缺失节点
  oldVnode = emptyNodeAt(oldVnode)
}
```

第四种情况可能是针对服务端渲染的逃生通道 / 或是指定 `hydrating = true` 所触发的直接替换策略，直接调用 `hydrate` 尝试直接替换后，调用 `invokeInsertHook` 触发模块的 insert 钩子进行更新

5. case 5: 以新节点替换旧节点

```js
/* case 5: 默认策略 -> 以新节点替换旧节点 */

// 替换已存在 elm
const oldElm = oldVnode.elm
const parentElm = nodeOps.parentNode(oldElm)

// 创建新节点
createElm(
  vnode,
  insertedVnodeQueue,
  oldElm._leaveCb ? null : parentElm,
  nodeOps.nextSibling(oldElm)
)

// 如果父节点存在，递归替换位于父节点的位置
if (isDef(vnode.parent)) {
  let ancestor = vnode.parent
  const patchable = isPatchable(vnode)
  while (ancestor) {
    // 首先调用父节点所有 destroy 钩子
    for (let i = 0; i < cbs.destroy.length; ++i) {
      cbs.destroy[i](ancestor)
    }
    // 放入新节点
    ancestor.elm = vnode.elm
    if (patchable) {
      /* 实节点 */
      // 调用所有 create 钩子
      for (let i = 0; i < cbs.create.length; ++i) {
        cbs.create[i](emptyNode, ancestor)
      }

      // 调用所有 data.hook.insert.fns 钩子              
      const insert = ancestor.data.hook.insert
      if (insert.merged) {
        for (let i = 1; i < insert.fns.length; i++) {
          insert.fns[i]()
        }
      }
    } else {
      /* 虚节点 */
      // 仅记录 ref
      registerRef(ancestor)
    }
    ancestor = ancestor.parent
  }
}
```

第五种情况就是两个节点是不一样的节点没办法进行深度比较，也不是服务端渲染的直接替换策略，所以需要调用 `createElm` 创建新节点之后一步步将新节点替换到旧节点在父节点中的位置

##### 2.3.3.1 invokeDestroyHook 销毁节点并调用 destroy 钩子

`invokeDestroyHook` 方法对应的是 case 1 的场景：销毁节点

- `/src/core/vdom/patch.js`(阅读笔记文件路径：`/src/core/vdom/patch/createPatchFunction.flat2.invokeDestroyHook.js`)

```js
export function createPatchFunction (backend) {

  // ...

  /* 触发 destroy 钩子 */
  function invokeDestroyHook (vnode) {
    let i, j
    const data = vnode.data
    if (isDef(data)) {
      if (isDef(i = data.hook) && isDef(i = i.destroy)) i(vnode)
      // 调用所有 destroy 钩子并传入 vnode
      for (i = 0; i < cbs.destroy.length; ++i) cbs.destroy[i](vnode)
    }
    if (isDef(i = vnode.children)) {
      for (j = 0; j < vnode.children.length; ++j) {
        // 递归触发子节点的 destroy 钩子
        invokeDestroyHook(vnode.children[j])
      }
    }
  }

  // ...

}
```

其实 `invokeDestroyHook` 就如同其名，其实就是调用各个模块的 destroy 钩子，相当于是针对节点的各个部件进行卸载，并对子组件递归调用 `invokeDestroyHook`

##### 2.3.3.2 patchVnode 比较并更新新旧节点

第二个要提出来特别说明的方法是 `patchVnode`，对应于前面的 case 3: 深度 patch

- `/src/core/vdom/patch.js`(阅读笔记文件路径：`/src/core/vdom/patch/createPatchFunction.flat2.patchVNode.flatx.js`)

```js
export function createPatchFunction (backend) {

  // ...

  /* 比较新旧节点差异并更新 */
  /**
   * case 1: 新旧节点相等 -> 直接返回
   * case 2: vnode.asyncFactory.resolved === true -> hydrate 快速创建直接替换
   * case 3: 新旧都是静态节点 -> 直接复用旧节点
   * case 4~7 非文本节点
   *   case 4: 比较并更新子节点数组
   *   case 5: 旧节点无子节点 -> 直接插入子节点数组
   *   case 6: 新节点无子节点 -> 移除旧子节点数组
   *   case 7: 旧节点存在文本内容 -> 节点文本置为空
   * case 8: 文本节点 -> 直接替换内容
   */
  function patchVnode (
    oldVnode,
    vnode,
    insertedVnodeQueue,
    ownerArray,
    index,
    removeOnly
  ) {
    if (oldVnode === vnode) {
      /* case 1: 新旧节点相等 -> 直接返回 */
      return
    }

    if (isDef(vnode.elm) && isDef(ownerArray)) {
      // 克隆数组中节点
      vnode = ownerArray[index] = cloneVNode(vnode)
    }

    const elm = vnode.elm = oldVnode.elm

    if (isTrue(oldVnode.isAsyncPlaceholder)) {
      if (isDef(vnode.asyncFactory.resolved)) {
        /* case 2: vnode.asyncFactory.resolved === true -> hydrate 快速创建直接替换 */
        hydrate(oldVnode.elm, vnode, insertedVnodeQueue)
      } else {
        vnode.isAsyncPlaceholder = true
      }
      return
    }

    if (isTrue(vnode.isStatic) &&
      isTrue(oldVnode.isStatic) &&
      vnode.key === oldVnode.key &&
      (isTrue(vnode.isCloned) || isTrue(vnode.isOnce))
    ) {
      /* case 3: 新旧都是静态节点 -> 直接复用旧节点 */
      vnode.componentInstance = oldVnode.componentInstance
      return
    }

    let i
    const data = vnode.data
    if (isDef(data) && isDef(i = data.hook) && isDef(i = i.prepatch)) {
      // 根据 vnode.data.hook.prepatch 钩子
      i(oldVnode, vnode)
    }

    const oldCh = oldVnode.children
    const ch = vnode.children
    if (isDef(data) && isPatchable(vnode)) {
      // 调用所有 update 钩子
      for (i = 0; i < cbs.update.length; ++i) cbs.update[i](oldVnode, vnode)
      // 调用 data.hook.update 钩子
      if (isDef(i = data.hook) && isDef(i = i.update)) i(oldVnode, vnode)
    }
    if (isUndef(vnode.text)) {
      // 非文本节点
      if (isDef(oldCh) && isDef(ch)) {
        /* case 4: 比较并更新子节点数组 */
        if (oldCh !== ch) updateChildren(elm, oldCh, ch, insertedVnodeQueue, removeOnly)
      } else if (isDef(ch)) {
        /* case 5: 旧节点无子节点 -> 直接插入子节点数组 */
        if (process.env.NODE_ENV !== 'production') {
          checkDuplicateKeys(ch)
        }
        if (isDef(oldVnode.text)) nodeOps.setTextContent(elm, '')
        addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue)
      } else if (isDef(oldCh)) {
        /* case 6: 新节点无子节点 -> 移除旧子节点数组 */
        removeVnodes(oldCh, 0, oldCh.length - 1)
      } else if (isDef(oldVnode.text)) {
        /* case 7: 旧节点存在文本内容 -> 节点文本置为空 */
        nodeOps.setTextContent(elm, '')
      }
    } else if (oldVnode.text !== vnode.text) {
      /* case 8: 文本节点 -> 直接替换内容 */
      nodeOps.setTextContent(elm, vnode.text)
    }
    if (isDef(data)) {
      // 调用 vnode.data.hook.postpatch 钩子
      if (isDef(i = data.hook) && isDef(i = i.postpatch)) i(oldVnode, vnode)
    }
  }

  // ...

}
```

在深度比较新旧节点的过程中，又可以在细分为八种情况

1. case 1: 新旧节点相等 -> 直接返回

```js
if (oldVnode === vnode) {
  /* case 1: 新旧节点相等 -> 直接返回 */
  return
}
```

2. case 2: hydrate 快速创建直接替换

```js
if (isTrue(oldVnode.isAsyncPlaceholder)) {
  if (isDef(vnode.asyncFactory.resolved)) {
    /* case 2: vnode.asyncFactory.resolved === true -> hydrate 快速创建直接替换 */
    hydrate(oldVnode.elm, vnode, insertedVnodeQueue)
  } else {
    vnode.isAsyncPlaceholder = true
  }
  return
}
```

跟前面一样直接调用 `hydrate` 方法快速替换

3. case 3: 新旧都是静态节点 -> 直接复用旧节点

```js
if (isTrue(vnode.isStatic) &&
  isTrue(oldVnode.isStatic) &&
  vnode.key === oldVnode.key &&
  (isTrue(vnode.isCloned) || isTrue(vnode.isOnce))
) {
  /* case 3: 新旧都是静态节点 -> 直接复用旧节点 */
  vnode.componentInstance = oldVnode.componentInstance
  return
}
```

VNode 中有这么一个属性 `isStatic` 标志其是否为静态节点，也就是不与任何动态数据绑定的固定节点，那只需要进行渲染然后多次复用就行了

下面的 case 4~7 对应的是非文本节点

4. case 4: 比较并更新子节点数组

```js
if (isDef(oldCh) && isDef(ch)) {
  /* case 4: 比较并更新子节点数组 */
  if (oldCh !== ch) updateChildren(elm, oldCh, ch, insertedVnodeQueue, removeOnly)
```

对于非文本节点，我们比较新旧即节点不同之后，就调用 `updateChildren` 来更新子数组

5. case 5: 旧节点无子节点 -> 直接插入子节点数组

```js
} else if (isDef(ch)) {
  /* case 5: 旧节点无子节点 -> 直接插入子节点数组 */
  if (process.env.NODE_ENV !== 'production') {
    checkDuplicateKeys(ch)
  }
  if (isDef(oldVnode.text)) nodeOps.setTextContent(elm, '')
  addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue)
```

如果旧节点没有子节点，那就直接调用 `addVnodes` 将整个新的子节点数组插入

6. case 6: 新节点无子节点 -> 移除旧子节点数组

```js
} else if (isDef(oldCh)) {
  /* case 6: 新节点无子节点 -> 移除旧子节点数组 */
  removeVnodes(oldCh, 0, oldCh.length - 1)
```

如果是新节点没有子节点，则调用 `removeVnodes` 一次销毁子节点数组

7. case 7: 旧节点存在文本内容 -> 节点文本置为空

```js
} else if (isDef(oldVnode.text)) {
  /* case 7: 旧节点存在文本内容 -> 节点文本置为空 */
  nodeOps.setTextContent(elm, '')
}
```

如果子节点为文本节点，则将其内容置为空

8. case 8: 文本节点 -> 直接替换内容

```js
} else if (oldVnode.text !== vnode.text) {
  /* case 8: 文本节点 -> 直接替换内容 */
  nodeOps.setTextContent(elm, vnode.text)
}
```

如果旧节点本身只是简单的文本节点，那我们就直接替换其内容

##### 2.3.3.3 addVnodes 添加多个节点

在前面提到的 case 5 当中，我们使用 `addVnodes` 来一次插入多个节点如下

```js
} else if (isDef(ch)) {
  /* case 5: 旧节点无子节点 -> 直接插入子节点数组 */
  if (process.env.NODE_ENV !== 'production') {
    checkDuplicateKeys(ch)
  }
  if (isDef(oldVnode.text)) nodeOps.setTextContent(elm, '')
  addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue)
```

下面我们看看具体内部使用哪些操作

- `/src/core/vdom/patch.js`(阅读笔记文件路径：`/src/core/vdom/patch/createPatchFunction.flat2.addVnodes.js`)

```js
export function createPatchFunction (backend) {

  // ...

  /* 直接添加剩余新节点 */
  function addVnodes (parentElm, refElm, vnodes, startIdx, endIdx, insertedVnodeQueue) {
    for (; startIdx <= endIdx; ++startIdx) {
      createElm(vnodes[startIdx], insertedVnodeQueue, parentElm, refElm, false, vnodes, startIdx)
    }
  }

  // ...

}
```

其实就是遍历要插入的数组，一个个调用 `createElm` 插入节点就行了

##### 2.3.3.4 removeVnodes 移除多个节点

对应于 case 6 的批量删除节点也是类似

- `/src/core/vdom/patch.js`(阅读笔记文件路径：`/src/core/vdom/patch/createPatchFunction.flat2.removeVnodes.js`)

```js
export function createPatchFunction (backend) {

  // ...

  /* 直接移除剩余旧节点 */
  function removeVnodes (vnodes, startIdx, endIdx) {
    for (; startIdx <= endIdx; ++startIdx) {
      const ch = vnodes[startIdx]
      if (isDef(ch)) {
        if (isDef(ch.tag)) {
          // realElement: removeNode 并调用 remove & destroy 钩子
          removeAndInvokeRemoveHook(ch)
          invokeDestroyHook(ch)
        } else { // Text node
          // 非 realElement: 直接调用 removeNode
          removeNode(ch.elm)
        }
      }
    }
  }

  // ...

}
```

它会遍历节点数组并调用 `removeNode` 后触发 remove 钩子，最后再触发 destroy 钩子完成子节点的销毁

##### 2.3.3.5 updateChildren 更新子节点数组

最后一个更新节点的操作是 case 4 中用到的 `updateChildren`，比较子节点数组并进行更新，也是 diff 算法中相对最复杂的部分

- `/src/core/vdom/patch.js`(阅读笔记文件路径：`/src/core/vdom/patch/createPatchFunction.flat2.updateChildren.js`)

```js
export function createPatchFunction (backend) {

  // ...

  /* 递归更新子数组(patchVnode 内部调用) */
  /**
   * case 1: 旧头 & 新头相同 -> 递归 patch
   * case 2: 旧尾 & 新尾相同 -> 递归 patch
   * case 3: 旧头 & 新尾相同 -> 递归 patch
   * case 4: 旧尾 & 新头相同 -> 递归 patch
   * case 5: 按序查找并更新节点
   */

  function updateChildren (parentElm, oldCh, newCh, insertedVnodeQueue, removeOnly) {
    let oldStartIdx = 0
    let newStartIdx = 0
    let oldEndIdx = oldCh.length - 1
    let oldStartVnode = oldCh[0]
    let oldEndVnode = oldCh[oldEndIdx]
    let newEndIdx = newCh.length - 1
    let newStartVnode = newCh[0]
    let newEndVnode = newCh[newEndIdx]
    let oldKeyToIdx, idxInOld, vnodeToMove, refElm

    // removeOnly 作用于特殊标签 <transition-group>
    const canMove = !removeOnly

    if (process.env.NODE_ENV !== 'production') {
      checkDuplicateKeys(newCh)
    }

    while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
      if (isUndef(oldStartVnode)) {
        // 旧节点 moveTo 最左侧 -> 直接跳过找下一个
        oldStartVnode = oldCh[++oldStartIdx] // Vnode has been moved left

      } else if (isUndef(oldEndVnode)) {
        // 旧节点 moveTo 最右侧 -> 直接跳过找前一个
        oldEndVnode = oldCh[--oldEndIdx]

      } else if (sameVnode(oldStartVnode, newStartVnode)) {
        /* case 1: 旧头 & 新头相同 -> 递归 patch */
        patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
        oldStartVnode = oldCh[++oldStartIdx]
        newStartVnode = newCh[++newStartIdx]

      } else if (sameVnode(oldEndVnode, newEndVnode)) {
        /* case 2: 旧尾 & 新尾相同 -> 递归 patch */
        patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
        oldEndVnode = oldCh[--oldEndIdx]
        newEndVnode = newCh[--newEndIdx]

      } else if (sameVnode(oldStartVnode, newEndVnode)) { // Vnode moved right
        /* case 3: 旧头 & 新尾相同 -> 递归 patch */
        patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
        // 更新后旧节点 moveTo 未处理节点最右侧
        canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm))
        oldStartVnode = oldCh[++oldStartIdx]
        newEndVnode = newCh[--newEndIdx]

      } else if (sameVnode(oldEndVnode, newStartVnode)) { // Vnode moved left
        /* case 4: 旧尾 & 新头相同 -> 递归 patch */
        patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
        // 更新后旧节点 moveTo 未处理节点最左侧
        canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)
        oldEndVnode = oldCh[--oldEndIdx]
        newStartVnode = newCh[++newStartIdx]

      } else {
        /* case 5: 按序查找并更新节点 */

        // oldKeyToIdx 旧节点 key: index 的映射
        if (isUndef(oldKeyToIdx)) oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)
        // 查找新头在旧数组的 index
        idxInOld = isDef(newStartVnode.key)
          ? oldKeyToIdx[newStartVnode.key]
          : findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx)
        if (isUndef(idxInOld)) {
          // 旧数组中不存在 -> 新元素直接插入
          createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
        } else {
          // 旧数组中存在
          vnodeToMove = oldCh[idxInOld]
          if (sameVnode(vnodeToMove, newStartVnode)) {
            // 递归更新子节点
            patchVnode(vnodeToMove, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
            oldCh[idxInOld] = undefined
            canMove && nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm)
          } else {
            // 直接替换子节点
            createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
          }
        }
        // 检查下一个新节点
        newStartVnode = newCh[++newStartIdx]
      }
    }
    if (oldStartIdx > oldEndIdx) {
      // 新旧子节点数不对等 -> startIdx 越界 -> 插入将剩余新节点
      refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm
      addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue)
    } else if (newStartIdx > newEndIdx) {
      // 移除剩余旧节点
      removeVnodes(oldCh, oldStartIdx, oldEndIdx)
    }
  }

  // ...

}
```

当我们要对子节点数组进行比较和更新的时候，我们有几种方案可以选择

1. 二重循环

我们可以对新旧子节点数组进行二重循环的遍历来进行节点的比较和更新

```js
for (const oldCh : oldChildren) {
    for (const ch : children) {
        patchVnode(oldCh, ch)
    }
}
```

最后再以新的子节点数组替换旧的子节点数组但是这样并不高效

然而这种实现方案效率非常低，最坏可能产生 $O(n^2)$ 的时间复杂度，加上递归遍历可能变成 $O(n^3)$ 的结果。

2. 头尾比较

第二种方案是先对头尾进行比较。由于我们从经验实践中可以发现大多数时候 DOM 元素的顺序是比较少改变的，而且相对顺序大多数时候也是稳定的，所以我们其实可以透过优先检查并比较头尾元素来提高性能

3. 头尾交叉比较

Vue2 中就采用了第二种的想法，并在此基础上更进一步增加了新头旧尾、新尾旧头的比较，也就是说对于一个子数组的更新会分为下列五种情况

- 3.1 新头与旧头比较
- 3.2 新尾与旧尾比较
- 3.3 新头与旧尾比较
- 3.4 新尾与旧头比较
- 3.5 回到原始按序遍历比较

用图表示就是下列五种

![](https://picures.oss-cn-beijing.aliyuncs.com/img/vue2_virtual_dom_diff_patch_updateChildren_case1.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/vue2_virtual_dom_diff_patch_updateChildren_case2.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/vue2_virtual_dom_diff_patch_updateChildren_case3.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/vue2_virtual_dom_diff_patch_updateChildren_case4.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/vue2_virtual_dom_diff_patch_updateChildren_case5.png)

第五种比较特别的是，原本我们需要拿新头来遍历旧子节点来找到对应的节点，这边其实可以保存一个 `key -> index` 的映射关系(哈希映射表)，使用哈希的方式直接找到对应的节点进行比较。

下面我们配合代码一段段来看

##### 2.3.3.6 更新子节点策略代码 & 图解

1. case 1: 新头 & 旧头相同 -> 递归 patch

![](https://picures.oss-cn-beijing.aliyuncs.com/img/vue2_virtual_dom_diff_patch_updateChildren_case1.png)

```js
} else if (sameVnode(oldStartVnode, newStartVnode)) {
  /* case 1: 旧头 & 新头相同 -> 递归 patch */
  patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
  oldStartVnode = oldCh[++oldStartIdx]
  newStartVnode = newCh[++newStartIdx]
```

比较新头 & 旧头相同(`sameVnode(oldStartVnode, newStartVnode)`)则递归执行 `patchVnode`

2. case 2: 新尾 & 旧尾相同 -> 递归 patch

![](https://picures.oss-cn-beijing.aliyuncs.com/img/vue2_virtual_dom_diff_patch_updateChildren_case2.png)

```js
} else if (sameVnode(oldEndVnode, newEndVnode)) {
  /* case 2: 旧尾 & 新尾相同 -> 递归 patch */
  patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
  oldEndVnode = oldCh[--oldEndIdx]
  newEndVnode = newCh[--newEndIdx]
```

比较新尾 & 旧尾相同(`sameVnode(oldEndVnode, newEndVnode)`)则递归执行 `patchVnode`

3. case 3: 新头 & 旧尾相同 -> 递归 patch

![](https://picures.oss-cn-beijing.aliyuncs.com/img/vue2_virtual_dom_diff_patch_updateChildren_case3.png)

```js
} else if (sameVnode(oldEndVnode, newStartVnode)) {
  /* case 3: 旧尾 & 新头相同 -> 递归 patch */
  patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
  // 更新后旧节点 moveTo 未处理节点最左侧
  canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)
  oldEndVnode = oldCh[--oldEndIdx]
  newStartVnode = newCh[++newStartIdx]
```

比较新头 & 旧尾相同(`sameVnode(oldEndVnode, newStartVnode)`)则递归执行 `patchVnode`

4. case 4: 新尾 & 旧头相同 -> 递归 patch

![](https://picures.oss-cn-beijing.aliyuncs.com/img/vue2_virtual_dom_diff_patch_updateChildren_case4.png)

```js
} else if (sameVnode(oldStartVnode, newEndVnode)) {
  /* case 4: 旧头 & 新尾相同 -> 递归 patch */
  patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
  // 更新后旧节点 moveTo 未处理节点最右侧
  canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm))
  oldStartVnode = oldCh[++oldStartIdx]
  newEndVnode = newCh[--newEndIdx]
```

比较新尾 & 旧头相同(`sameVnode(oldStartVnode, newEndVnode)`)则递归执行 `patchVnode`

5. case 5: 按序查找并更新节点

![](https://picures.oss-cn-beijing.aliyuncs.com/img/vue2_virtual_dom_diff_patch_updateChildren_case5.png)

```js
} else {
  /* case 5: 按序查找并更新节点 */

  // oldKeyToIdx 旧节点 key: index 的映射
  if (isUndef(oldKeyToIdx)) oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)
  // 查找新头在旧数组的 index
  idxInOld = isDef(newStartVnode.key)
    ? oldKeyToIdx[newStartVnode.key]
    : findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx)
  if (isUndef(idxInOld)) {
    // 旧数组中不存在 -> 新元素直接插入
    createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
  } else {
    // 旧数组中存在
    vnodeToMove = oldCh[idxInOld]
    if (sameVnode(vnodeToMove, newStartVnode)) {
      // 递归更新子节点
      patchVnode(vnodeToMove, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
      oldCh[idxInOld] = undefined
      canMove && nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm)
    } else {
      // 直接替换子节点
      createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
    }
  }
  // 检查下一个新节点
  newStartVnode = newCh[++newStartIdx]
}
```

最后一种由于头尾或交叉都不匹配，所以原本需要进行顺序遍历，然而我们还可以预先建立一个 key $\to$ index 的映射来简化遍历操作的代价，使用 `createKeyToOldIdx` 方法生成映射表

- `/src/core/vdom/patch.js`(阅读笔记文件路径：`/src/core/vdom/patch/createKeyToOldIdx.js`)

```js
/* 建立 key: index 的映射 */

function createKeyToOldIdx (children, beginIdx, endIdx) {
  let i, key
  const map = {}
  for (i = beginIdx; i <= endIdx; ++i) {
    key = children[i].key
    if (isDef(key)) map[key] = i
  }
  // map 后续可用于快速查找相同 key 的节点进行比较
  return map
}
```

如此一来我们就能透过哈希映射将查找相同节点的操作简化为常数时间

```js
// 查找新头在旧数组的 index
idxInOld = isDef(newStartVnode.key)
  ? oldKeyToIdx[newStartVnode.key]
  : findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx)
```

最后一个策略是：**如果函数标签一样，则深度比较并更新节点；否则直接使用新节点进行替换**

```js
if (sameVnode(vnodeToMove, newStartVnode)) {
  // 递归更新子节点
  patchVnode(vnodeToMove, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
  oldCh[idxInOld] = undefined
  canMove && nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm)
} else {
  // 直接替换子节点
  createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
}
```

##### 2.3.3.7 invokeInsertHook 完成节点插入调用 insert 钩子

在整个 `patch` 方法的最后，再调用一下 `invokeInsertHook` 来调用 insert 钩子表示组件被插入的生命周期

```js
// 触发新节点的 insert 生命周期钩子
invokeInsertHook(vnode, insertedVnodeQueue, isInitialPatch)
return vnode.elm
```

- `/src/core/vdom/patch.js`(阅读笔记文件路径：`/src/core/vdom/patch/createPatchFunction.flat2.invokeInsertHook.js`)

```js
export function createPatchFunction (backend) {

  // ...

  /* 触发 insert 钩子 */
  function invokeInsertHook (vnode, queue, initial) {
    if (isTrue(initial) && isDef(vnode.parent)) {
      // 维护子节点 insert 钩子调用顺序
      vnode.parent.data.pendingInsert = queue
    } else {
      for (let i = 0; i < queue.length; ++i) {
        // 调用 insert 钩子
        queue[i].data.hook.insert(queue[i])
      }
    }
  }

  // ...

}
```

## 3. 虚拟 DOM 与 diff 算法总结

最后给全篇下一个总结：

### 3.1 实现选择和主体流程

- Vue 对于 MVVM 的数据绑定渲染节点的方式采用 **虚拟 DOM** 的实现方式
- 更新节点的时候会调用 `updateComponent` 方法，流程如下：

![](https://picures.oss-cn-beijing.aliyuncs.com/img/vue2_virtual_dom_diff_updateComponent.png)

  - `_render` 根据当前数据快照生成 VNode 虚拟 DOM 树
  - `_update` 比较新旧虚拟 DOM 树并将差异更新(patch 补丁)到真实 DOM 上(借由调用 `__patch__` 方法)

### 3.2 比较更新 VNode 节点树：__patch__ 方法

- `__patch__` 方法分为三种操作类型
  - 创建节点：对应 `createElm` 方法
  - 删除节点：对应 `removeVnode` 方法
  - 比较并更新节点：对应 `patch` 方法。

### 3.3 比较更新操作的五种情形：patch 方法

- `createPatchFunction` 方法参数选创建参数
  - nodeOps 平台相关元素节点操作方法
  - modules 节点部件管理模块
    - baseModules 核心模块(Vue 特性)
    - platformModules 平台相关模块
- `patch` 比较过程可以分为五种情况：
  1. case 1: 销毁节点
  2. case 2: 新增节点
  3. case 3: 深度 patch，对应 `patchVNode` 方法
  4. case 4: 直接替换策略(服务端渲染 || hydrating === true)
  5. case 5: 以新节点替换旧节点

### 3.4 深度比较更新：patchVNode 方法

- 比较新旧节点的 `patchVNode` 方法可以分为以下八种情况
  1. case 1: 新旧节点相等 -> 直接返回
  2. case 2: vnode.asyncFactory.resolved === true -> hydrate 快速创建直接替换
  3. case 3: 新旧都是静态节点 -> 直接复用旧节点
  4. case 4: 比较并更新子节点数组，对应 `updateChildren` 方法
  5. case 5: 旧节点无子节点 -> 直接插入子节点数组，对应 `addVnodes` 方法
  6. case 6: 新节点无子节点 -> 移除旧子节点数组，对应 `removeVnodes` 方法
  7. case 7: 旧节点存在文本内容 -> 节点文本置为空
  8. case 8: 文本节点 -> 直接替换内容

### 3.5 深度比较子节点：updateChildren 方法

- `updateChildren` 方法针对五种情况比较子节点
  1. case 1: 新头 & 旧头相同 -> 递归 patch
  2. case 2: 新尾 & 旧尾相同 -> 递归 patch
  3. case 3: 新头 & 旧尾相同 -> 递归 patch
  4. case 4: 新尾 & 旧头相同 -> 递归 patch
  5. case 5: 按序查找并更新节点(建立 key:index 映射)

# 结语

本篇到此完全把 Vue 的核心：虚拟 DOM 的原理、diff 算法的具体流程都从源码的视角过了一遍。接下来就只剩下 MVVM 的第三阶段：**模版编译** 原理了。

把 Vue 的源码全部看过一遍还是比较清楚的，对于 diff 算法的了解也更直接和清楚，不再是抽象的比较替换流程。对于技术的落地，以及编写开源框架主要注意的平台/环境检查和适配，以及学到一些 Vue 对于比较复杂架构的流程设计方式(如 hook 钩子、modules 模块的组合)，渐渐培养能自己构建基础建设(造轮子)的能力。

# 其他资源

## 参考连接

<table>
  <tr>
    <td>Vue源码系列-Vue中文社区</td>
    <td><a href="https://vue-js.com/learn-vue/virtualDOM/">https://vue-js.com/learn-vue/virtualDOM/</a></td>
  </tr>
  <tr>
    <td>vuejs/vue-2.6.12-Github</td>
    <td><a href="https://github.com/vuejs/vue/tree/v2.6.12">https://github.com/vuejs/vue/tree/v2.6.12</a></td>
  </tr>
</table>

## 阅读笔记参考

<a href="https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/vue-2.6.12">https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/vue-2.6.12</a>
