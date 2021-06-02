# React 核心特性: 3 种创建 Refs 引用 & 2 种 Refs 转发的方法

@[TOC](文章目录)

<!-- TOC -->

- [React 核心特性: 3 种创建 Refs 引用 & 2 种 Refs 转发的方法](#react-核心特性-3-种创建-refs-引用--2-种-refs-转发的方法)
- [前言](#前言)
- [正文](#正文)
  - [0. 概念](#0-概念)
  - [1. 字符串 Refs(Depracated 废弃的方案)](#1-字符串-refsdepracated-废弃的方案)
  - [2. 回调 Refs](#2-回调-refs)
  - [3. React.createRef 创建 Refs 引用(推荐)](#3-reactcreateref-创建-refs-引用推荐)
  - [4. React.forwardRef 传递 Refs 引用](#4-reactforwardref-传递-refs-引用)
  - [5. Ref Props 透过别名传递](#5-ref-props-透过别名传递)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

在 React、Vue 等响应式框架兴起之前，最有名的前端框架非 jQuery 莫属。其最有名的功能就是使用 `$(selector)` 配合任意选择器便能够直接选取到目标元素，同时提供对 DOM 元素操作的代理，进而扩展成各式各样非常方便的操作。

当前端的时代进入 React、Vue 等 MVVM 框架时，我们不再直接或间接的操作，取而代之我们使用 **声明式的模版语法** 配合 **模版编译** 的能力来完成数据到视图的绑定。

然而 React 也不打算禁止开发者直接操作 DOM 元素，毕竟有些事情还是要直接操作元素才能够完成(如调用 DOM 元素原生的方法、对最底层的 DOM 元素进行度量等)。

为此 React 提供了一个 Refs 的特性，提供开发者一个安全直接访问 DOM 元素的桥梁，并由 React 来保证在组件的更新、重新渲染的过程中，Refs 都能正确的指向最新而正确的 DOM 元素。

下面我们就来看看 Refs 的各种使用方式。

# 正文

鉴于 React 在 Refs 特性上存在旧版本的遗留问题，从最早的版本到现在也出现了各种用法和一些特殊的解决方案。当然使用官方推荐的最新也相对比较稳定的 API 是最好的，不过我们还是按先后顺序一一介绍不同的 Refs 的用法

## 0. 概念

在看 Refs 各种具体的用法之前，我们先来感受一下 Refs 大致的形式：

```js
class Component extends React.Component {
    constructor(props) {
        super(props)
        this.ref = React.createRef()
    }

    render() {
        return (
            <div ref={this.ref}>A Elemnt</div>
        )
    }
}
```

首先我们会先创建一个 ref(透过 `React.createRef` API)，然后这个 ref 就好像一张标签纸一样"贴到"某个元素上，后面我们再查看这个标签纸(`this.ref`)的时候，就可以拿到目标元素节点了

下面我们来看看不同的 ref 的用法

## 1. 字符串 Refs(Depracated 废弃的方案)

第一种是使用字符串来定义 ref 引用(该用法已经不再推荐，是极早期的用法，在当前比较新的版本里面会发出警告)

- `src/refs/StringRef.jsx`

```js
import React, { Component } from 'react'
import { group, log } from '../utils/console'
import Other from './Other'

class StringRef extends Component {
  componentDidMount() {
    group('StringRef', () => {
      log('this.refs.el1', this.refs.el1)
      log('this.refs.el2', this.refs.el2)
    })
  }

  render() {
    return (
      <div className="block">
        <h2>String Ref - 字符串 Refs</h2>
        <div ref="el1">this is a simple dom element</div>
        <Other ref="el2" />
      </div>
    )
  }
}

export default StringRef
```

字符串引用的定义方式就是直接传入一个字符串作为标签

```html
<div ref="el1">this is a simple dom element</div>
```

然后我们就可以透过 `this.refs.el1` 访问到 dom 元素或是组件

```js
group('StringRef', () => {
  log('this.refs.el1', this.refs.el1)
  log('this.refs.el2', this.refs.el2)
})
```

效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_refs_string_ref_sample.png)

我们可以注意到以下几点：

- `ref` 属性放在类组件上时会获取 **组件实例的引用** (注意这边我们不可以为函数组件添加 `ref` 属性，因为函数组件不存在组件实例)；同时我们发现组件自有的属性只有绑定过的函数

    ```js
    this.boundFunction = this.boundFunction.bind(this)
    ```

    而没有绑定的函数则只会出现在原型对象上

- 第二点要注意的是当我们使用 `ref` 来获取一个类组件，并调用其中的方法的时候，其实这是一个 **产生高耦合** 的动作，会使得上级组件高度依赖于内部组件的实现，产生破坏性的影响。

    所以当我们仅仅只是为了调用子组件的行为来改变子组件的状态时(如调用 `open、close` 方法)，我们应该考虑是不是直接传递一个状态参数即可(如 `isOpen={true}`，也就是 `props.isOpen`)，剩下的都交由子组件根据状态自己负责渲染的部分

## 2. 回调 Refs

第一种的字符串 ref 其实是一个非常不好的实现，React 为了处理静态定义的 ref 引用在背后需要做很多额外的工作。

取而代之的是，我们可以传递一个回调函数来代替静态的字符串类型

- `src/refs/StringRef.jsx`

```js
import React, { Component } from 'react'
import { group, log } from '../utils/console'
import Other from './Other'

class CallbackRef extends Component {
  constructor(props) {
    super(props)

    this.setRefEl1 = (element) => {
      this.refEl1 = element
    }

    this.setRefEl2 = (element) => {
      this.refEl2 = element
    }
  }

  componentDidMount() {
    group('CallbackRef', () => {
      log('this.refEl1', this.refEl1)
      log('this.refEl2', this.refEl2)
    })
  }

  render() {
    return (
      <div className="block">
        <h2>Callback Ref - 回调 Refs</h2>
        <div ref={this.setRefEl1}>this is a simple dom element</div>
        <Other ref={this.setRefEl2} />
      </div>
    )
  }
}

export default CallbackRef
```

我们看到第二个回调 Ref 版本我们传入一个回调函数作为 `ref` 属性的值

```html
<div ref={this.setRefEl1}>this is a simple dom element</div>
```

```js
this.setRefEl1 = (element) => {
  this.refEl1 = element
}
```

而 React 就会在背后为我们确保 `refEl1` 引用的有效性，每当 dom 元素改变的时候，就会重新调用一次这个回调函数，并传入最新的 DOM 元素作第一个参数(`element`)

效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_refs_callback_ref_sample.png)

## 3. React.createRef 创建 Refs 引用(推荐)

然而上面两种用法都有点过时，或是写起来还是挺麻烦的，下面我们介绍一种当前最常用也是后面比较推荐的用法，透过调用 `React.createRef` API 来创建引用

- `src/refs/ReactCreateRef.jsx`

```js
import React, { Component } from 'react'
import { group, log } from '../utils/console'
import Other from './Other'

class ReactCreateRef extends Component {
  constructor(props) {
    super(props)

    this.refEl1 = React.createRef()
    this.refEl2 = React.createRef()
  }

  componentDidMount() {
    group('ReactCreateRef', () => {
      log('this.refEl1.current', this.refEl1.current)
      log('this.refEl2.current', this.refEl2.current)
    })
  }

  render() {
    return (
      <div className="block">
        <h2>React.createRef - Refs API</h2>
        <div ref={this.refEl1}>this is a simple dom element</div>
        <Other ref={this.refEl2} />
      </div>
    )
  }
}

export default ReactCreateRef
```

与回调版本类似，然而不同的是我们传入的是一个由 `React.createRef()` 的实例

```js
this.refEl1 = React.createRef()
```

```html
<div ref={this.refEl1}>this is a simple dom element</div>
```

而 React 则会在运行时确保 `refEl1` 引用到正确的 DOM 元素。

比较不同的是，使用 `React.createRef()` API 之后，我们就不再是直接拿到 DOM 元素，而是绑定在 ref 对象的 `current` 属性上

```js
group('ReactCreateRef', () => {
  log('this.refEl1.current', this.refEl1.current)
  log('this.refEl2.current', this.refEl2.current)
})
```

效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_refs_react_create_ref_sample.png)

## 4. React.forwardRef 传递 Refs 引用

前面我们已经学会了三种创建 Ref 的方法(当然最终还是推荐都使用第三种 `React.createRef`)，但是我们发现到当我们将引用放在类组件的 `ref` 属性上时会引用到组件实例本身，而且函数组件甚至都不能用！

由于函数组件实际上不存在组件实例所以没办法被 `ref` 引用，为了解决这个问题其实我们可以将附着在函数组件上的 `ref` 指向组件内部的某个元素即可，也就是说我们要将 **Ref 转发给组件内的 dom 元素** (也可以称作 ref 的传递)

这时候我们就要使用 `React.forwardRef` 这个 API，而它只能对函数组件使用而不能对类组件或是一般函数使用

- `src/refs/ReactCreateRef.jsx`

```js
import React, { Component } from 'react'
import { group, log } from '../utils/console'

function OriginButton(props, ref) {
  log(`${props.children} ref: ${ref}`)
  return <button ref={ref}>{props.children}</button>
}

const RefedButton = React.forwardRef(OriginButton)

class ForwardRef extends Component {
  constructor(props) {
    super(props)

    this.refButton = React.createRef()
  }

  componentDidMount() {
    group('ForwardRef', () => {
      log('this.refButton.current', this.refButton.current)
    })
  }

  render() {
    return (
      <div className="block">
        <h2>React.forwardRef - 将 ref 传递给函数组件</h2>
        <RefedButton>不带 ref 的按钮</RefedButton>
        <RefedButton ref={this.refButton}>带 ref 的按钮</RefedButton>
      </div>
    )
  }
}

export default ForwardRef
```

我们可以看到 `React.forwardRef` 接受一个函数组件为参数，返回一个包装过的函数组件，函数标签如下

```ts
{
    'React.forwardRef': (FunctionalComponent) => WrappedComponent
    FunctionalComponent: (props, ref) => ReactElemnt
}
```

调用方法

```js
const RefedButton = React.forwardRef(OriginButton)
```

这时候我们的函数组件除了原本的 props 之外，就可以再接受第二个参数 `ref`，也就是上面传递下来的引用，然后我们把它贴到 `button` 元素上

```js
function OriginButton(props, ref) {
  log(`${props.children} ref: ${ref}`)
  return <button ref={ref}>{props.children}</button>
}
```

最后我们就可以看到包装过的函数组件就跟原本的类组件一般，都可以传递 `ref` 引用属性了

```js
render() {
  return (
    <div className="block">
      <h2>React.forwardRef - 将 ref 传递给函数组件</h2>
      <RefedButton>不带 ref 的按钮</RefedButton>
      <RefedButton ref={this.refButton}>带 ref 的按钮</RefedButton>
    </div>
  )
}
```

但是这边要注意的是：

- 对于传递给 **类组件** 的 ref，指向 **组件实例本身**
- 对于传递给 **函数组件** 的 ref，则是指向 **组件内的某个元素**

也就是这时候对于函数组件的 `ref` 直接穿透了组件的外壳指向组件内部的元素，效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_refs_react_forward_ref_sample.png)

## 5. Ref Props 透过别名传递

既然我们可以穿透函数组件将 ref 传递进去，我们也可以使 ref 穿透 **类组件**。

由于 `ref` 在 React 中属于特殊的属性名称，才会被标记并保留引用绑定到 `ref` 对象上，那其实我们只要透过一个不叫 `ref` 的属性来传递引用就行啦！

- `src/refs/AliasRef.jsx`

```js
import React, { Component } from 'react'
import { group, log } from '../utils/console'

class ClassButton extends Component {
  render() {
    const { buttonRef: ref, children } = this.props
    log(`${children} ref: ${ref}`)
    return <button ref={ref}>{children}</button>
  }
}

function FunctionButton(props) {
  const { buttonRef: ref, children } = props
  log(`${children} ref: ${ref}`)
  return <button ref={ref}>{children}</button>
}

class AliasRef extends Component {
  constructor(props) {
    super(props)

    this.refButton1 = React.createRef()
    this.refButton2 = React.createRef()
  }

  componentDidMount() {
    group('AliasRef', () => {
      log('this.refButton1.current', this.refButton1.current)
      log('this.refButton2.current', this.refButton2.current)
    })
  }

  render() {
    return (
      <div className="block">
        <h2>Ref Props - 将 ref 透过其他 prosp 传递给子组件</h2>
        <ClassButton buttonRef={this.refButton1}>
          类组件按钮
        </ClassButton>
        <FunctionButton buttonRef={this.refButton2}>
          函数组件按钮
        </FunctionButton>
      </div>
    )
  }
}

export default AliasRef
```

这就好像取一个别名一样，上面我们透过将引用传入 `buttonRef` 属性

```html
<ClassButton buttonRef={this.refButton1}>
  类组件按钮
</ClassButton>
<FunctionButton buttonRef={this.refButton2}>
  函数组件按钮
</FunctionButton>
```

然后在组件内部再将引用贴到正确的元素的 `ref` 属性上即可

```js
class ClassButton extends Component {
  render() {
    const { buttonRef: ref, children } = this.props
    log(`${children} ref: ${ref}`)
    return <button ref={ref}>{children}</button>
  }
}

function FunctionButton(props) {
  const { buttonRef: ref, children } = props
  log(`${children} ref: ${ref}`)
  return <button ref={ref}>{children}</button>
}
```

效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_refs_alias_ref_sample.png)

我们可以看到最后一种方法的转发方式非常直接而且清晰，同时不论是函数组件还是类组件都能用，因为都是透过基本的 `props` 来传递。

在此之上我们就可以透过 **高阶组件** 配合 `React.forwardRef` 再加上别名的用法来完成对于类组件的 `ref` 直接转发，还是比较麻烦，直接用别名最省事。

# 结语

本篇解说了在 React 当中关于 Refs 的创建和传递的用法，对于函数组件其实还有一个 `useRef` 的方法来创建 Ref 引用，不过那个我们就放到 Hook 篇在讨论。

经过几篇 React 特性的介绍和使用，我也觉得差不多了，下一篇我们就进入 React 16.8 的重头戏 - **React Hook API 的用法！**

# 其他资源

## 参考连接

| Title                                                         | Link                                                                                                                                 |
| ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| React Doc - Refs and the DOM                                  | [https://react.docschina.org/docs/refs-and-the-dom.html](https://react.docschina.org/docs/refs-and-the-dom.html)                     |
| React Doc - Refs 转发                                         | [https://react.docschina.org/docs/forwarding-refs.html](https://react.docschina.org/docs/forwarding-refs.html)                       |
| github issue - dom_ref_forwarding_alternatives_before_16.3.md | [https://gist.github.com/gaearon/1a018a023347fe1c2476073330cc5509](https://gist.github.com/gaearon/1a018a023347fe1c2476073330cc5509) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_refs](https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_refs)


