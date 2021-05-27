# React 入门: 核心特性全面解析

@[TOC](文章目录)

<!-- TOC -->

- [React 入门: 核心特性全面解析](#react-入门-核心特性全面解析)
- [前言](#前言)
- [正文](#正文)
  - [0. 项目搭建](#0-项目搭建)
  - [1. JSX](#1-jsx)
    - [1.1 揭露 JSX 的神秘面纱](#11-揭露-jsx-的神秘面纱)
    - [1.2 扩展 JS(强化版的 html)](#12-扩展-js强化版的-html)
    - [1.3 ReactDOM 渲染模版](#13-reactdom-渲染模版)
  - [2. 组件基础](#2-组件基础)
    - [2.1 类组件(Class Component) vs 函数组件(Function Component)](#21-类组件class-component-vs-函数组件function-component)
  - [3. Props 属性/数据传递(父组件传递给子组件的属性/参数)](#3-props-属性数据传递父组件传递给子组件的属性参数)
    - [3.1 构造函数参数 props](#31-构造函数参数-props)
    - [3.2 渲染多个元素(React.Fragment 的使用)](#32-渲染多个元素reactfragment-的使用)
    - [3.3 属性保留字](#33-属性保留字)
  - [4. State 组件状态](#4-state-组件状态)
    - [4.1 初始化状态](#41-初始化状态)
    - [4.2 改变状态](#42-改变状态)
  - [5. 事件处理](#5-事件处理)
    - [5.1 绑定函数](#51-绑定函数)
    - [5.2 文本开关](#52-文本开关)
  - [6. 表单使用](#6-表单使用)
    - [6.1 通用用户输入处理函数](#61-通用用户输入处理函数)
    - [6.2 select 选择框](#62-select-选择框)
    - [6.3 file 文件上传 & ref 引用](#63-file-文件上传--ref-引用)
    - [6.4 ref 特性](#64-ref-特性)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

在此之前主要都是学习 Vue 同时也是使用 Vue 来进行项目开发，近期打算开始好好学一学社区更加丰富，也更多被大公司采用的 React。作为一个在前端开发中比 Vue 耕耘更久的 MVVM 框架，React 的各种特性、用法可以说是也比 Vue 要在完善许多。当然最近 Vue3 的崛起也使得 Vue 框架更加成熟可用。

之后应该会开启一系列的 React 特性解说和使用记录的博客，有机会的话也像 Vue 一样爬爬源码。下面我们开始本篇的正文：**React 的核心概念**。

# 正文

本篇说是对核心概念的讲解，不如说是跟着官方教程实践的一个记录(毕竟 React 的[官方文档](https://react.docschina.org/docs/hello-world.html)写的还是挺舒服的)

下面我们就跟着官方文档走一遍，再加入本人对于使用上的一些心得和见解。

## 0. 项目搭建

项目搭建的部分前面其实已经写过两篇关于 React 项目启动的方式

- [React項目啟動：create-react-app](https://blog.csdn.net/weixin_44691608/article/details/106516736)
- [React 项目启动2：使用 webpack 手动创建 React 项目(附加 React Router + Redux)](https://blog.csdn.net/weixin_44691608/article/details/116363154)

不管是直接使用官方提供的 CRA 脚手架或是自己使用 npm/yarn 搭建项目都行，本篇就不多做着墨，专注在 React 框架的特性使用上。下面我们开始

## 1. JSX

在 React 当中最显著的第一个特性就是 **JSX**，他的样子长这样：

- `src/views/JSX.jsx`

```jsx
import React from 'react'

class JSXSample extends React.Component {
  render() {
    return (
      <>
        <h2>1. JSX</h2>
        <h3>这是一个 JSX，被编译为 React.createElement</h3>
      </>
    )
  }
}

export default JSXSample
```

我们编写 React 组件的时候可以定义一个类(class，基本的类组件)并继承 `React.Component` ，JSX 透过将模版和逻辑放到一起的方式，使得我们在编写代码的时候能够将关注点集中在 UI 模版上，毕竟我们使用像 Vue、React 这类的框架最终的目的还是编写出一个页面，理所当然还是以模版为核心。

使用 JSX 使我们可以直接将一个看起来像是 html 元素的东西当成一个变量来操作！

```jsx
const template = <div>这是一个 JSX 对象</div>
```

### 1.1 揭露 JSX 的神秘面纱

实际上在运行时 JSX 会将我们写下来的模版对象转换成 `React.createElement` 的调用(好我知道大家都听腻了hhh)，如下

```jsx
const template = React.createElement('div', {}, '这是一个 JSX 对象')
```

我们可以将代码贴到 babel 官网测试 JSX 的编译结果

[babel-repl](https://www.babeljs.cn/repl#?browsers=&build=&builtIns=false&corejs=3.6&spec=false&loose=false&code_lz=Q&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=react&prettier=false&targets=Node-10.13&version=7.14.3&externalPlugins=)

### 1.2 扩展 JS(强化版的 html)

而当我们需要向 JSX 的模版插入对象时、或是文本内容的时候，我们要透过`{}`将我们可以像下面这样

```jsx
const el1 = <div>普通 html 文本</div>
const el2 = <div>title: {title}</div>  // 插入 title 变量
const el3 = <div onClick={this.clickHandler}></div>
const el4 = <div style={{ width: '300px' }}></div>  // {} 内为 JS 内容，使用一个 JS 对象表示 style 属性(第二重括号)
```

我们将最一开始的 `JSX` 组件(后面提到什么是组件)加入主要的页面 `App` 如下

- `src/views/App.jsx`

```jsx
import React, { Component } from 'react'

import JSXSample from './JSX'

class App extends Component {

  render() {
    return (
      <>
        <h1>React 基础特性演示</h1>
        <h4>Hello World</h4>
        <JSXSample></JSXSample>
      </>
    )
  }
}

export default App
```

这里我们就创建了一个主要页面的 `App` 组件，并在内部使用另一个内部组件 `JSXSample`。

### 1.3 ReactDOM 渲染模版

最后我们要将组件渲染到页面上的时候则要使用 ReactDOM 如下

- `src/index.js`

```js
import React from 'react'
import ReactDOM from 'react-dom'

import App from './views/App'

ReactDOM.render(<App></App>, document.querySelector('#app'))
```

这时候我们就可以在页面上看到如下结果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_basic_1_jsx.png)

## 2. 组件基础

有了 JSX 之后我们已经可以在 React 当中使用形似 html 的元素标签，然后使用 `{}` 将 JS 的数据和对象放到 JSX 当中。

第二个重要的概念则是 **组件(Component)**。组件的简单定义就是一个能独立运作的 html 片段组合。透过将原本要写在 html 的标签 **切割** 成一个个的组件的组合，并将与该片段相关的行为封装到组件内，就是在 React 框架当中的基本开发模式。

### 2.1 类组件(Class Component) vs 函数组件(Function Component)

在 React 当中有两种方式创建组件，一个是我们前面已经出现过的使用 ES6 Class 语法的 **类组件**

- `src/views/Component.js`

```jsx
import React from 'react'

class ComponentA extends React.Component {
  render() {
    return <h3>这是一个 class 类组件</h3>
  }
}
```

另一种则是直接使用写成一般的 function 函数的 **函数组件**

```jsx
function ComponentB() {
  return <h3>这是一个 function 函数组件</h3>
}

const ComponentC = () => {
  return <h3>箭头函数也可以哦</h3>
}
```

两种组件的使用方式一模一样，可以直接作为 JSX 的新标签来使用(也是参照了官方推荐的使用 **组合** 而非继承的方式来复用组件)

```js
class Component extends React.Component {
  render() {
    return (
      <>
        <h2>2. 组件</h2>
        <ComponentA></ComponentA>
        <ComponentB></ComponentB>
        <ComponentC></ComponentC>
      </>
    )
  }
}

export default Component
```

一样放入我们的根组件 `App` 当中

- `src/views/App.jsx`

```js
import React, { Component } from 'react'

import JSXSample from './JSX'
import ComponentSample from './Component'

class App extends Component {
  render() {
    return (
      <>
        <h1>React 基础特性演示</h1>
        <h4>Hello World</h4>
        <JSXSample></JSXSample>
        <ComponentSample></ComponentSample>
      </>
    )
  }
}

export default App
```

页面呈现的效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_basic_2_component.png)

## 3. Props 属性/数据传递(父组件传递给子组件的属性/参数)

有了 JSX 创建元素，Component 封装成组件之后，我们就会联想到在使用 html 的时候我们可以像下面这样设置元素属性

```html
<div data-src="xxx"></div>
```

而前面我们也提到可以在 JSX 元素上附加属性，甚至可以带上 JS 对象

```jsx
const obj = {
    style: { width: '300px' },
    title: '鼠标停止时显示细节', 
    onClick: () => console.log('鼠标点击')
}

// ...

const el = <div {...obj}></div>
```

同样的，我们可以向我们的组件传递参数(属性)，使我们可以提高组件的抽象化程度，根据不同的属性来生成不一样的表现行为，具体使用与上面没什么区别：

```jsx
const el = <SomeComponent data={dataObj}></SomeComponent>
```

下面我们先建立一个 `Props(PropsSample)` 组件，并向它传递一些属性：

- `src/views/App.jsx`

```jsx
import React, { Component } from 'react'

import JSXSample from './JSX'
import ComponentSample from './Component'
import PropsSample from './Props'

class App extends Component {

  render() {
    const obj = { a: 123 }
    const otherProps = {
      b: 456,
      c: 789,
    }
    return (
      <>
        <h1>React 基础特性演示</h1>
        <h4>Hello World</h4>
        <JSXSample></JSXSample>
        <ComponentSample></ComponentSample>
        <PropsSample
          title="Title from App"
          dynamic={obj}
          {...otherProps}
        ></PropsSample>
      </>
    )
  }
}

export default App
```

上面我们看到我们总共对 `PropsSample` 组件传递了 `title, dynamic, b, c` 四个属性(`b, c` 透过传递展开的对象的形式)

### 3.1 构造函数参数 props

当我们向组件传递参数的时候，对于组件来说，则作为构造函数的第一个参数(`props`)的形式来呈现，而在组件内部要使用的时候可以直接使用 `this.props.xxx` 的形式来引用

- `src/views/Props.jsx`

```jsx
import React from 'react'

function Prop(props) {
  const { prop, value } = props
  return (
    <h4>
      {'> '}
      {prop}: {value}
    </h4>
  )
}

class Props extends React.Component {
  constructor(props) {
    super(props)
  }

  buildProps() {
    const props = []
    for (const prop in this.props) {
      if (prop === 'title') continue
      props.push({ prop, value: JSON.stringify(this.props[prop]) })
    }

    return props.map((prop) => <Prop key={prop.prop} {...prop}></Prop>)
  }

  render() {
    return (
      <>
        <h2>3. Props - 数据传递(从上到下的数据流)</h2>
        <h3>title: {this.props.title}</h3>
        <h3>other props:</h3>
        {this.buildProps()}
      </>
    )
  }
}

export default Props
```

在这段代码中有几个点需要注意

1. 使用 `constructor` 构造函数的时候，由于继承 `React.Component` 类型，所以在做任何操作之前需要先调用 `super(props)` 构建出父类的实例才能获得正确的 `this` 指向的实例(具体详细参考 ES6 的 class 用法)

```jsx
class Props extends React.Component {
  constructor(props) {
    super(props)
  }
  // ...
```

2. 要使用 `props` 传递过来的参数我们可以直接在组件内部透过 `this.props.xxx` 来获得传递参数值，React 还会在 props 修改的时候自动帮我们重新渲染组件

```jsx
class Props extends React.Component {

  // ...

  render() {
    return (
      <>
        <h2>3. Props - 数据传递(从上到下的数据流)</h2>
        <h3>title: {this.props.title}</h3>
```

3. 对于函数组件来说， `props` 一样是透过函数的第一个参数传递进来，这里我们透过将除了 `title` 外的其他属性做成一个 `Prop` 组件实例的方式，并将结果列表直接写在 `{}` 内

```jsx
function Prop(props) {
  const { prop, value } = props
  return (
    <h4>
      {'> '}
      {prop}: {value}
    </h4>
  )
}

class Props extends React.Component {
  constructor(props) {/* ... */}

  buildProps() {
    const props = []
    for (const prop in this.props) {
      if (prop === 'title') continue
      props.push({ prop, value: JSON.stringify(this.props[prop]) })
    }

    return props.map((prop) => <Prop key={prop.prop} {...prop}></Prop>)
  }

  render() {
    return (
      <>
        <h2>3. Props - 数据传递(从上到下的数据流)</h2>
        <h3>title: {this.props.title}</h3>
        <h3>other props:</h3>
        {this.buildProps()}
      </>
    )
  }

  // ...
```

### 3.2 渲染多个元素(React.Fragment 的使用)

这里我们可以看到，实际上 `this.buildProps()` 返回的值将会是一个 `Prop[]` 数组，React 则会识别出来然后直接渲染成多个元素；同样的当我们的组件需要返回非单根的模版的时候，我们就可以返回一个数组

```jsx
render() {
    return [
        <div>element1</div>,
        <div>element2</div>,
        // ...
    ]
}
```

或是我们可以使用 `React.Fragment` 标签作为假的根元素，实际渲染的时候 React 不会将其生成成真正的元素标签

```jsx
render() {
    return (
        <React.Fragment>
            <div>element1</div>
            <div>element2</div>
        </React.Fragment>
    )
}
```

看起来就没有那么丑了，我们还可以使用缩写 `<></>`，看起来有更简洁有力而不影响可读性

```jsx
render() {
    return (
        <>
            <div>element1</div>
            <div>element2</div>
        </>
    )
}
```

最后页面渲染的效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_basic_3_props.png)

### 3.3 属性保留字

这里在唠叨一句，在我们向组件传递属性的时候，有一些属性名是属于保留字，也就是 React 会对其进行特殊处理而不会真的传递个内部组件的 props，主要有以下几个

- `key`：虚拟 DOM 比较更新的时候参考的唯一标识
- `ref`：用于引用 DOM 元素/组件的相当于一个锚点(除非使用 `React.forwardRef`)

这边不对其做讲解，我们目前只要知道 React 不会将上述两种属性传递到子组件的 props 上，对于这两个属性实际的用法会在其他篇章做讲解。

## 4. State 组件状态

下一个非常重要的概念就是 **state 状态**。我们前面提过对于 React 框架来说，每一个组件(Component)不仅是一个 html 元素片段的单位，更是一个能 **独立运行** 的基本单位；也就是说除了 html 模版之外，我们还要在组件之中处理与模版相关的行为、并保存组件自有的状态，同时当组件状态改变的时候 React 会为我们负责组件的重新渲染。

### 4.1 初始化状态

由于本篇主要讲解 React 的基础特性，所以先从最基本的形式，使用 ES6 的类组件作为示例。首先使用状态的第一步就是要先初始化这个状态

- `src/views/State.jsx`

```jsx
class State extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      date: new Date(),
    }
  }
```

这边需要注意一点：上一个小节提到的 props 属性只是由父组件，或是说外部组件所传递进来的参数/属性，他可能代表了某种状态，也可能只是一个渲染的选项参数；而这里使用的 state 状态则是表示一个可能被改变的状态。

上面的代码示例说明我们初始化状态的时候要在组件的构造函数(`constructor`)中，在 `super(props)`构造实例之后，直接对 `this.state` 进行赋值来完成初始化

当我们要引用状态的时候就可以在组件内部其他方法透过 `this.state.xxx` 进行引用。一样，状态更新的时候 React 也会很聪明的帮我们自动重新渲染组件

```jsx
class State extends React.Component {

  // ...

  render() {
    return (
      <>
        <h2>4. State 组件内部状态</h2>
        <h3>date: {this.state.date.toLocaleString()}</h3>
      </>
    )
  }
}
```

### 4.2 改变状态

既然一个组件有了状态，我们当然是预期他会随时间改变，否则我们直接写死一个值就得了，所以下面我们让 `date` 状态动起来。

```jsx
  tick() {
    this.setState({ date: new Date() })
  }
  
  componentDidMount() {
    console.log('[StateSample.LifeCycle] componentDidMount')
    this.timer = setInterval(() => this.tick(), 1000)
  }
```

我们透过在 `componentDidMount` 生命周期函数中定义一个计时器，让他随时间每一秒就更新一下 `date` 的状态值；而要更新组件的状态的时候我们不能直接对 `date` 直接重新赋值

```jsx
/* 错误示范 */ this.state.date = new Date()
```

这样 React 哪知道要更新组件，所以这时候我们继承的 `React.Component` 就起到了作用，其实我们在继承 `React.Component` 的时候也继承了一个 `setState` 原型方法，我们必须透过 `setState` 方法来更新我们的状态 React 才能够侦测到状态的变化并进行比较和重新渲染。(有关 `setState` 更多详细和进阶的用法我们会在其他篇章进行解说)

这边还有一个要点，注意到上面有这么一句话

```jsx
this.timer = setInterval(() => this.tick(), 1000)
```

当我们调用 `setInterval` 设定好计时器的时候，返回的计时器其实在整个组件的生命周期中是不会被改变的(又或是说与组件状态是无关的，所以我们可以直接放在 `this.timer` 而不需要添加到我们的 `this.state` 状态当中)

最后我们看一下完整版本的计时器状态

- `src/views/State.jsx`

```jsx
import React from 'react'

class State extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      date: new Date(),
    }
  }

  tick() {
    this.setState({ date: new Date() })
  }

  componentDidMount() {
    console.log('[StateSample.LifeCycle] componentDidMount')
    this.timer = setInterval(() => this.tick(), 1000)
  }

  componentWillUnmount() {
    console.log('[StateSample.LifeCycle] componentWillUnmount')
    clearInterval(this.timer)
  }

  stop() {
    console.log('timer stop')
    clearInterval(this.timer)
    this.stop = () => {}
  }

  render() {
    return (
      <>
        <h2>4. State 组件内部状态</h2>
        <h3>date: {this.state.date.toLocaleString()}</h3>
        <button onClick={() => this.stop()}>别跑了</button>
      </>
    )
  }
}

export default State
```

效果如下，我们可以看到页面上的时间每一秒都会更新一次到最新的时间

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_basic_4_state.gif)

## 5. 事件处理

下一个阶段，有了状态，我们最后一种基本能力就是要处理事件。

我们知道在浏览器页面中，采用的是事件触发/响应的架构，由用户的操作发起浏览器事件，而页面提供事件处理机制来对响应并处理用户的操作。

在原生的 html 总我们可能会这样写

```html
<button onclick="clickHandler($event)">Click!</div>

<script>
    function clickHandler(e) {
        console.log('click event', e)
    }
</script>
```

React 也沿用了这样的形式，不过由于 JSX 语法的关系同时需要与原生行为相互隔离，所以猜用小驼峰的命名规则来处理此类的事件响应属性

```jsx
<button onClick={(e) => clickHandler(e)}>Click!</div>
```

下面我们一样做一个组件当示例

- `src/views/Events.jsx`

```jsx
class Events extends React.Component {

  unbindFunction() {
    console.log(this)
  }

  render() {
    return (
      <>
        <h2>5. 事件处理</h2>
        <button onClick={this.unbindFunction}>未绑定函数</button>
      </>
    )
  }
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_basic_5_events_1.gif)

结果你会发现打印出来的结果其实是 `undefined`，这就不得不说到 ES6 的 class 特性了

### 5.1 绑定函数

由于 ES6 的类作为原本 `function` 类型的语法糖，也就是说写在类里面的方法与透过 `XxxClass.prototype.xxxMethod` 注入的方法一样

```js
class MyClass {
    f() { return this }
}

// 等价于

function MyClass() {}

MyClass.prototype.f = function() { return this }
```

在这样的用法场景之下如果我们新建一个实例并调用他的方法时，`this` 会正确指向调用方法的实例；然而当我们将方法单独提出的时候，`this` 则会指向调用时的全局对象，通常不是我们所希望的

```js
const myClass = new MyClass()
const f = myClass.f
console.log(f() === myClass.f())  // false
```

这在 React 中也是一样的，还记得前面我们是这么写的

```js
<button onClick={this.unbindFunction}>未绑定函数</button>
```

也就是说我们其实将处理方法单独提取出来传入 `onClick`，则 `this` 当然不会正确的指向组件实例，这时候我们有两种解决方案

1. 提前绑定

第一种我们可以在构造函数里面提前绑定好方法调用的组件实例如下

```jsx
class Events extends React.Component {
  constructor(props) {
    super(props)
    // ...
    this.bindFunction = this.bindFunction.bind(this)
  }

  bindFunction() {
    console.log(this)
  }

  render() {
    return (
      <>
        <h2>5. 事件处理</h2>
        <button onClick={this.bindFunction}>绑定函数</button>
      </>
    )
  }
}
```

2. 箭头函数

第二种我们可以使用 ES6 箭头函数绑定定义时作用域的特性来绑定调用上下文

```jsx
class Events extends React.Component {

  bindFunction() {
    console.log(this)
  }

  render() {
    return (
      <>
        <h2>5. 事件处理</h2>
        <button onClick={() => this.bindFunction()}>绑定函数</button>
      </>
    )
  }
}
```

效果如下，可以看到点击后的方法确实正确返回了该组件实例

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_basic_5_events_2.gif)

不过这边要非常注意一点，由于这里的 `() => this.bindFunction()` 其实在每次渲染的时候都是一个新定义的完全一个全新的函数，也就是说每次都会对该元素进行重新渲染，数量多之后可能就必须考虑性能问题了。

### 5.2 文本开关

最后我们透过提供一个显示/隐藏文本内容的按钮作为事件处理的最终示例

- `src/views/Events.jsx`

```jsx
import React from 'react'

class Events extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      visible: true,
    }
    this.toggleVisible = this.toggleVisible.bind(this)
  }

  toggleVisible() {
    this.setState({ visible: !this.state.visible })
  }

  render() {
    return (
      <>
        <h2>5. 事件处理</h2>
        <button onClick={this.toggleVisible}>
          {this.state.visible ? '隐藏文本' : '显示文本'}
        </button>
        <h3 style={{ opacity: this.state.visible ? '100%' : '0%' }}>
          内容文本...
        </h3>
      </>
    )
  }
}

export default Events
```

`toggleVisible` 会对 `visible` 变量取反，而按钮的文字和文字的 `opacity` 则会根据 `visible` 的状态而改变，效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_basic_5_events.gif)

## 6. 表单使用

最后的最后我们不得不提一下表单。作为前端与用户交互的第一线，搜集用户最直接也是最主观的方法便是提供一个给用户填写的表单，在 React 中使用表单又与原生 html 的表单有一点点不那么相似，我们先看看第一个版本的基础用法

- `src/views/Forms.jsx`

```jsx
import React from 'react'

class Form extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      name: '',
      password: '',
    }

    this.handleNameChange = this.handleNameChange.bind(this)
    this.handlePasswordChange = this.handlePasswordChange.bind(this)
    this.handleSubmit = this.handleSubmit.bind(this)
  }

  handleNameChange(e) {
    console.log('handleNameChange', e, e.target.value)
    this.setState({ name: e.target.value })
  }

  handlePasswordChange(e) {
    console.log('handlePasswordChange', e, e.target.value)
    this.setState({ password: e.target.value })
  }

  handleSubmit(e) {
    e.preventDefault()
    console.log('handleSubmit')
    console.log('form:', this.state.form)
  }

  render() {
    const {
      handleNameChange,
      handlePasswordChange,
      handleSubmit,
      state: { name, password },
    } = this
    return (
      <>
        <h2>6. 表单</h2>
        <div>
          <form onSubmit={handleSubmit}>
            <label>
              名称:{' '}
              <input value={name} onChange={handleNameChange} />
            </label>
            <br />
            <label>
              密码:{' '}
              <input
                value={password}
                onChange={handlePasswordChange}
              />
            </label>
            <br />

            <button type="submit">提交表单</button>
          </form>
        </div>
      </>
    )
  }
}

export default Form
```

我们先定义两个属性 `name、password` 然后为这两个属性编写一个状态更新方法 `handleNameChange、handlePasswordChange`。在 React 中对于文本输入，推荐使用将组件状态传入 `value`，并监听 `change` 事件并改变组件状态的形式，这样的输入处理我们称为 **受控组件**，也就是说组件输入框的内容其实是强依赖于组件状态(`value={this.state.xxx}`)，我们则是透过监听 `change` 事件的方法来更新组件状态，也就是说如果我们不调用任何 `setState` 方法，则下次组件渲染的时候还是根据 state 渲染，也就是用户的输入是无效的(对于 **不受控组件** 这里不进行讨论，不过下面将要提到的 **ref** 则是不受控组件使用的方法之一)

上述代码的效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_basic_6_form.png)

### 6.1 通用用户输入处理函数

然而我们要向上面那样为每个输入每一个变量编写一个处理函数实在太过麻烦，所以既然这几个函数的形式非常类似，我们就可以编写一个更通用用于处理文本输入并修改状态的函数

```jsx
render() {
    return (
        // ...
        <label>
          名称:{' '}
          <input
            name="name"
            value={name}
            onChange={handleTextChange}
          />
        </label>
        <br />
        <label>
          密码:{' '}
          <input
            name="password"
            value={password}
            onChange={handleTextChange}
          />
        </label>
    )
}
```

首先我们为输入框提供一个 `name` 属性，用于表示对应状态中的属性

```jsx
  handleTextChange(e) {
    console.log('handleTextChange', e, e.target.value)
    const { value, name } = e.target
    this.setState({ [name]: value })
  }
```

接下来我们就可以透过 `e.target.name` 来获得我们要更新的状态属性名，并使用 ES6 的特性 `{ [name]: value }` 来动态选择要更新的目标属性名称了。

### 6.2 select 选择框

html 的除了提供 `<input>` 接受用户输入文本，还提供更多形式的用户输入标签，下面我们演示使用 `<select>` 提供用户一个选择框输入

- `src/views/Forms.jsx`

```jsx
import React from 'react'

class Form extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      // ...
      lazy: '',
    }

    this.handleLazyChange = this.handleLazyChange.bind(this)
    this.handleSubmit = this.handleSubmit.bind(this)
  }

  handleLazyChange(e) {
    console.log('handleLazyChange', e, e.target.value)
    this.setState({ lazy: e.target.value })
  }

  handleSubmit(e) {
    e.preventDefault()
    console.log('handleSubmit')
    console.log('form:', this.state)
  }

  render() {
    const {
      handleLazyChange,
      handleSubmit,
      state: { name, password, lazy },
    } = this
    return (
      <>
        <h2>6. 表单</h2>
        <div>
          <form onSubmit={handleSubmit}>
            
            {/* ... */}

            <label>
              懒惰程度:{' '}
              <select value={lazy} onChange={handleLazyChange}>
                <option value="lazy">懒</option>
                <option value="aBitLazy">极懒</option>
                <option value="superLazy">超级懒</option>
              </select>
            </label>
            <br />

            <button type="submit">提交表单</button>
          </form>
        </div>
      </>
    )
  }
}

export default Form
```

其实与文本输入也非常类似，就是透过 `value` 来将状态映射到模版上，用 `change` 事件来更新状态，效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_basic_6_form_1.gif)

### 6.3 file 文件上传 & ref 引用

最后我们再提一种表单中可能会使用的输入类型：**文件上传**。原生 html 的部分可能要这么写

```html
<input type="file" />
```

然而在 React 中我们是没办法主动修改这个输入的 `value` 值的(因为上传文件需要透过浏览器的 API 进行调用后生成 File 对象)，也就是说这时候我们需要将保管输入值(文件)的能力交还给 dom 元素自己管理，这时候我们就必须用上 `ref` 的功能。

使用 `ref` 相当于是添加一个引用(也可以说打上一个标签)，前面我们提过 `ref` 属性被保留的用意就在这里，他是一个特殊属性，用于放置引用的标签纸，具体的使用方法如下

- `src/views/Forms.jsx`

```jsx
import React from 'react'

class Form extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      name: '',
      password: '',
      lazy: '',
    }

    this.file = React.createRef()
    this.handleFileChange = this.handleFileChange.bind(this)
    this.handleSubmit = this.handleSubmit.bind(this)
  }

  handleFileChange(e) {
    console.log('handleFileChange', e)
    const fileInput = this.file.current
    const file = fileInput.files[0]
    console.log(fileInput, file)
    console.log(`file: ${file.name}`)
  }

  handleSubmit(e) {
    e.preventDefault()
    console.log('handleSubmit')
    console.log('form:', this.state)
  }

  render() {
    const {
      handleSubmit,
      handleFileChange,
      file,
    } = this
    return (
      <>
        <h2>6. 表单</h2>
        <div>
          <form onSubmit={handleSubmit}>
            <label>
              上传文件:{' '}
              <input type="file" ref={file} onChange={handleFileChange} />
            </label>
            <br />

            <button type="submit">提交表单</button>
          </form>
        </div>
      </>
    )
  }
}

export default Form
```

我们先使用 `React.createRef` 方法创建一个引用对象(标签纸)，然后将其放入文件上传框的 `ref` 属性上如下

```jsx
<input type="file" ref={file} onChange={handleFileChange} />
```

如此一来就好像将标签纸贴到了这个元素上，如此一来我们就可以透过访问 `file` (React.createRef 返回的引用)的 `current` 属性来访问得到真实 dom 元素对象，然后再透过元素上的 `files[0]` 来获得选中的文件信息

```jsx
  handleFileChange(e) {
    console.log('handleFileChange', e)
    const fileInput = this.file.current
    const file = fileInput.files[0]
    console.log(fileInput, file)
    console.log(`file: ${file.name}`)
  }
```

最终效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_basic_6_form_2.png)

### 6.4 ref 特性

最后再啰嗦一下，有关 `ref` 的特殊用法。在前面的例子我们已经看到我们透过将 `React.createRef` 创建的引用放到 dom 元素的 `ref` 属性上，就可以透过引用的 `current` 属性来访问到真实的 dom 元素。

同时由于 ref 的特殊性，当我们将 ref 放到组件实例的属性上如下

```jsx
const ref = React.createRef()

const el = <SomeComponent ref={ref}></SomeComponent>
```

这时候 `ref` 属性并不会像前面提过的 `props` 一样传递到组件内部作为 `props` 中的其中一个属性；实际上这时候 `ref` 变量则会直接指向 `SomeConponent` 的组件实例。

如果我们想要使 `ref` 引用指向组件内的特定属性的时候，则需要使用 `React.forwardRef` 方法来传递引用，详细使用我会在另一篇详细说明。

# 结语

本篇根据官方的核心概念 API 走了一遍，带大家入门，认识 React 的基础能力。在这些核心概念的基础之上还有更多应对不同场景的用法和优化手段如 `React.memo`、高阶组件 HOC、refs 转发等，后续会再分别对不同的用法进行研究和分享，供大家参考。

# 其他资源

## 参考连接

| Title                                                                        | Link                                                                                                                                                                                                                                                                                                                                                                             |
| ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| React官方 - Hello World                                                      | [https://react.docschina.org/docs/hello-world.html](https://react.docschina.org/docs/hello-world.html)                                                                                                                                                                                                                                                                           |
| React項目啟動：create-react-app                                              | [https://blog.csdn.net/weixin_44691608/article/details/106516736](https://blog.csdn.net/weixin_44691608/article/details/106516736)                                                                                                                                                                                                                                               |
| React 项目启动2：使用 webpack 手动创建 React 项目(附加 React Router + Redux) | [https://blog.csdn.net/weixin_44691608/article/details/116363154](https://blog.csdn.net/weixin_44691608/article/details/116363154)                                                                                                                                                                                                                                               |
| Babel - repl                                                                 | [https://www.babeljs.cn/repl](https://www.babeljs.cn/repl#?browsers=&build=&builtIns=false&corejs=3.6&spec=false&loose=false&code_lz=Q&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=react&prettier=false&targets=Node-10.13&version=7.14.3&externalPlugins=) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_basic](https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_basic)
