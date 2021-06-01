# React 高阶指引: Context 上下文 & 组件组合 & Render Props

@[TOC](文章目录)

<!-- TOC -->

- [React 高阶指引: Context 上下文 & 组件组合 & Render Props](#react-高阶指引-context-上下文--组件组合--render-props)
- [前言](#前言)
- [正文](#正文)
  - [1. Context 上下文](#1-context-上下文)
    - [1.1 使用动机 & 场景](#11-使用动机--场景)
    - [1.2 基本用法：Provider + Consumer](#12-基本用法provider--consumer)
      - [1.2.1 定义全局数据对象](#121-定义全局数据对象)
      - [1.2.2 React.createContext 创建上下文对象 ThemeContext](#122-reactcreatecontext-创建上下文对象-themecontext)
      - [1.2.3 使用 ThemeContext.Provider 定义上下文](#123-使用-themecontextprovider-定义上下文)
      - [1.2.4 使用 ThemeContext.Consumer 获取全局数据](#124-使用-themecontextconsumer-获取全局数据)
      - [1.2.5 任意组件都能作为消费者](#125-任意组件都能作为消费者)
    - [1.3 使用 contextType 简化类组件](#13-使用-contexttype-简化类组件)
    - [1.4 多个 Context 上下文](#14-多个-context-上下文)
      - [1.4.1 新的上下文对象 UserContext](#141-新的上下文对象-usercontext)
      - [1.4.2 嵌套使用 Provider](#142-嵌套使用-provider)
      - [1.4.3 使用不同的 Consumer 接受数据](#143-使用不同的-consumer-接受数据)
    - [1.5 将状态转换函数也透过 Context 传递](#15-将状态转换函数也透过-context-传递)
    - [1.6 Context 小结：为什么少用 Context？](#16-context-小结为什么少用-context)
      - [1.6.1 使用规范(推荐)](#161-使用规范推荐)
  - [2. 组件组合(Component Composition)](#2-组件组合component-composition)
    - [2.1 什么叫组件组合？](#21-什么叫组件组合)
    - [2.2 组件组合第一式：渲染子节点数组](#22-组件组合第一式渲染子节点数组)
    - [2.3 组件组合第二式：插槽(slot)](#23-组件组合第二式插槽slot)
    - [2.4 组件组合第三式：特殊实例](#24-组件组合第三式特殊实例)
    - [2.5 组件组合第四式：Render Props 传递渲染函数](#25-组件组合第四式render-props-传递渲染函数)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

今天一样也是来解说 React 的高级应用技巧，内容可能涉及一些特别的 API 使用，又或是针对 React 组件和 props 的特殊用法，可以算作一种设计模式，React 内专属的设计模式！(初入 React 领域的同学请移步[React 入门: 核心特性全面解析](https://blog.csdn.net/weixin_44691608/article/details/117343164))

本篇要说明的主题主要有

- Context 上下文的使用
- Component Composition 组件组合
- Render props 函数渲染组件

同时会配合一些使用场景和示例代码，下面我们马上开始

# 正文

## 1. Context 上下文

### 1.1 使用动机 & 场景

在开始用之前我们先来谈谈为什么要有 Context 上下文这种东西。

在上一篇：[React 高级指引: 从状态提升到高阶组件(HOC)](https://blog.csdn.net/weixin_44691608/article/details/117440794)，我们提过当多个叶节点需要共享状态的时候可以透过将共享的状态提升到最近的共同父组件当中如下图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_context_component_composition_render_props_context_explain1.png)

当时当我们的组件嵌套逻辑非常复杂，组件渲染树变得越来越高的时候，要找到最近的共同父组件越来愈遥远

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_context_component_composition_render_props_context_explain2.png)

同时从父组件传递下来的状态需要根据组件嵌套关系一层层传递下来，对于中间的状态无关组件来说，不仅仅是多了好多与自己不相关的 props 需要处理，同时这也是对中间组件的逻辑的一种破坏。

这时候我们就设想能不能有一种方法能够穿透中间组件，直接将状态传递到目标组件当中

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_context_component_composition_render_props_context_basic.png)

而这就是 Context 上下文对象的原始动机。

备注：然而仅仅为了避免简单的 props 传递而滥用 Context 是不可取的，不过我们暂且先当作目标就是这么个回事，后面会在重新说明为什么仅仅用于简化 props 传递是不可取的

下面我们就一个个来看 Context 的不同使用方式和技巧

### 1.2 基本用法：Provider + Consumer

首先第一种我们先介绍最基础的 Provider + Consumer 的基本使用方式

#### 1.2.1 定义全局数据对象

首先我们先定义一个需要被共享的数据，而在选择使用 Context 的情况下，其实它可能是某个全局的共享数据对象

- `src/context/themes.js`

```js
const themes = {
  light: {
    foreground: '#000000',
    background: '#eeeeee',
  },
  dark: {
    foreground: '#ffffff',
    background: '#222222',
  },
}

export default themes
```

我们定义一个全局布局主题，分为一般模式和暗黑模式的前景和后景颜色

#### 1.2.2 React.createContext 创建上下文对象 ThemeContext

接下来是使用 `React.createContext` API 创建我们的上下文对象

- `src/context/ThemeContext.js`

```js
import React from 'react'
import themes from './themes'

export const ThemeContext = React.createContext(themes.dark)
```

`React.createContext(defaultValue)` 的参数传入的是默认值，当我们的 `Provider` 组件没有定义数据时则会使用一开始创建上下文对象时传入的默认值 `defaultValue`

下面我们来看看所谓的 `Provider` 是什么

#### 1.2.3 使用 ThemeContext.Provider 定义上下文

前面我们已经定义好了上下文对象和全局数据，那么我们要如何将这个全局输入加入我们的组件树当中呢？答案就是透过 `Context.Provider` 这个特殊组件，以它为根会创建一个存在全局数据的局部组件树，也就是从 `Context.Provider` 都将能够透过某种方式直接获取这个全局数据对象

- `src/context/Version1.jsx`

```js
class Version1 extends Component {
  constructor(props) {
    super(props)
    this.state = {
      theme: themes.light,
    }
    this.toggleTheme = this.toggleTheme.bind(this)
  }

  toggleTheme() {
    this.setState({
      theme:
        this.state.theme === themes.light
          ? themes.dark
          : themes.light,
    })
  }

  render() {
    return (
      <div>
        <ThemeContext.Provider value={this.state.theme}>
          <ToolBar changeTheme={this.toggleTheme} />
        </ThemeContext.Provider>
      </div>
    )
  }
}
```

我们看到 Version1 组件首先先将主题(theme)数据放到组件状态当中，然后渲染的时候透过 `ThemeContext.Provider` 特殊组件来创建含有上下文的局部组件树，并将全局数据透过 `value` 属性传入。

接下来只要是在 `ThemeContext.Provider` 之下任意层级的组件都能透过某种方式直接获取从 `value` 传入的全局数据对象

#### 1.2.4 使用 ThemeContext.Consumer 获取全局数据

在第一个例子中我们先展示最基础款的：使用 `Context.Consumer` 来获取局部的全局数据对象，我们使用 `ToolBar` 组件充当中间组件，说明全局数据(`theme`)直接跳过 `ToolBar` 从 `Version1` 组件直接传到 `ThemedButton` 组件中使用

- `src/context/Version1.jsx`

```js
function ToolBar(props) {
  return (
    <ThemedButton onClick={props.changeTheme}>
      Change theme
    </ThemedButton>
  )
}
```

```js
// 使用 Consumer
class ThemedButton extends Component {
  render() {
    const { children, onClick } = this.props
    return (
      <>
        {/* directly Usage Component */}
        <ThemeContext.Consumer>
          {(theme) => (
            <button
              onClick={onClick}
              style={{
                backgroundColor: theme.background,
                color: theme.foreground,
              }}
            >
              {children}
            </button>
          )}
        </ThemeContext.Consumer>
      </>
    )
  }
}
```

我们可以看到，在 `ThemedButton` 组件内我们透过使用 `ThemeContext.Consumer` 这个特殊组件就能获得由 `ThemeContext.Provider` 传递下来的全局对象。

具体接受数据的方式是透过定义一个 Render Props 的子组件(后面会再解释什么是 Render Props)，也就是定义一个标签为 `value => Component` 的函数作为子组件，这时候的 `value` 就是的当时传入 `ThemeContext.Provider` 组件的 `value` 属性的全局数据对象，而我们就可以根据这个全局数据对象 `value` 来渲染内部组件 Component

最终的效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_context_component_composition_render_props_context_sample1.gif)

我们可以看到透过点击按钮调用刚刚透过 props 流传递下来的 `toggleTheme` 就能够改变全局的 `theme` 数据，进而造成按钮的样式改变。

#### 1.2.5 任意组件都能作为消费者

最基础的 `Context.Consumer` 的用法虽然琐碎，确实功能上比较全面的，根据全局数据 `value` 渲染的子组件由于就是一个 JSX，所以不论是类组件或是函数组件都可以

- 消费者为 **类组件**

```js
// 使用 Consumer
class ThemedButton extends Component {
  render() {
    const { children, onClick } = this.props
    return (
      <>
        {/* Class Component */}
        <ThemeContext.Consumer>
          {(theme) => {
            const props = {
              children,
              onClick,
              style: {
                backgroundColor: theme.background,
                color: theme.foreground,
              },
            }
            return <StyledButton {...props} />
          }}
        </ThemeContext.Consumer>
      </>
    )
  }
}
```

```js
class StyledButton extends Component {
  render() {
    console.log('styled button 1')
    const { children, ...props } = this.props
    return <button {...props}>{children}</button>
  }
}
```

- 消费者为 **函数组件**

```js
// 使用 Consumer
class ThemedButton extends Component {
  render() {
    const { children, onClick } = this.props
    return (
      <>
        {/* Function Component */}
        <ThemeContext.Consumer>
          {(theme) => {
            const props = {
              children,
              onClick,
              style: {
                backgroundColor: theme.background,
                color: theme.foreground,
              },
            }
            return <StyledButton2 {...props} />
          }}
        </ThemeContext.Consumer>
      </>
    )
  }
}
```

```js
function StyledButton2(props) {
  console.log('styled button 2')
  const { children, ...rest } = props
  return <button {...rest}>{children}</button>
}
```

### 1.3 使用 contextType 简化类组件

然而使用 `Context.Consumer` 的方式其实还是有点麻烦，而且写起来还是有些庞大，所以对于类组件还提供了 `static.contextType` 的方式

- `src/context/Version2.jsx`

```js
import React, { Component } from 'react'
import { ThemeContext } from './ThemeContext'
import themes from './themes'

class ThemedButton extends Component {
  // 使用 contextType
  static contextType = ThemeContext

  render() {
    const theme = this.context
    const { children, onClick } = this.props
    return (
      <button
        onClick={onClick}
        style={{
          backgroundColor: theme.background,
          color: theme.foreground,
        }}
      >
        {children}
      </button>
    )
  }
}

function ToolBar(props) {
  return (
    <ThemedButton onClick={props.changeTheme}>
      Change theme
    </ThemedButton>
  )
}

class Version2 extends Component {
  constructor(props) {
    super(props)
    this.state = {
      theme: themes.light,
    }
    this.toggleTheme = this.toggleTheme.bind(this)
  }

  toggleTheme() {
    this.setState({
      theme:
        this.state.theme === themes.light
          ? themes.dark
          : themes.light,
    })
  }

  render() {
    return (
      <div>
        <ThemeContext.Provider value={this.state.theme}>
          <ToolBar changeTheme={this.toggleTheme} />
        </ThemeContext.Provider>
      </div>
    )
  }
}

export default Version2
```

第二个版本与第一个版本雷同，其核心在于

```js
class ThemedButton extends Component {
  // 使用 contextType
  static contextType = ThemeContext
```

当我们为类组件定义静态的上下文类型(`static contextType`)的时候，我们就可以直接透过 `this.context` 获取上下文中的全局数据对象如下

```js
class ThemedButton extends Component {
  // 使用 contextType
  static contextType = ThemeContext

  render() {
    const theme = this.context
    const { children, onClick } = this.props
    return (
      <button
        onClick={onClick}
        style={{
          backgroundColor: theme.background,
          color: theme.foreground,
        }}
      >
        {children}
      </button>
    )
  }
```

### 1.4 多个 Context 上下文

然而我们看到前面两个例子中，都只存在一个全局数据对象，那可怎么办。实际上我们可以直接简单嵌套 `Context.Provider` 就好了：

#### 1.4.1 新的上下文对象 UserContext

首先我们先创建一个新的全局数据

- `src/context/users.js`

```js
const users = {
  donovan: {
    name: 'Donovan',
    age: 22,
  },
  alice: {
    name: 'Alice',
    age: 18,
  },
}

export default users
```

接下来创建一个新的上下文对象

- `src/context/UserContext.js`

```js
import React from 'react'
import users from './users'

export const UserContext = React.createContext(users.donovan)
```

#### 1.4.2 嵌套使用 Provider

接下来我们直接将两个 `Provider` 组件叠加在一起就好了

- `src/context/Version3.jsx`

```js
class Version3 extends Component {
  constructor(props) {
    super(props)
    this.state = {
      theme: themes.light,
      user: users.donovan,
    }
    this.toggleTheme = this.toggleTheme.bind(this)
    this.signIn = this.signIn.bind(this)
  }

  toggleTheme() {
    this.setState({
      theme:
        this.state.theme === themes.light
          ? themes.dark
          : themes.light,
    })
  }

  signIn(user) {
    this.setState({ user })
  }

  render() {
    return (
      <div>
        <ThemeContext.Provider value={this.state.theme}>
          <UserContext.Provider value={this.state.user}>
            <ToolBar
              toggleTheme={this.toggleTheme}
              signIn={this.signIn}
            />
          </UserContext.Provider>
        </ThemeContext.Provider>
      </div>
    )
  }
}
```

#### 1.4.3 使用不同的 Consumer 接受数据

有了多个 Context 的存在的时候，我们如果使用 `contextType` 的用法那就只能使用一种 Context 数据类型，因为 `contextType` 只能有一个类型值咯。

要想一次使用多个全局数据对象的话，就需要回到 `Context.Consumer` 的用法，不同的 Context 对象提供的 Consumer 就会传入对应的全局数据值，如下：

- `src/context/Version3.jsx`

```js
// multiple context
function ToolBar(props) {
  const { toggleTheme, signIn } = props
  return (
    <>
      <ThemeContext.Consumer>
        {(theme) => (
          <button
            onClick={toggleTheme}
            style={{
              backgroundColor: theme.background,
              color: theme.foreground,
            }}
          >
            Change theme
          </button>
        )}
      </ThemeContext.Consumer>
      <br />
      <button onClick={() => signIn(users.donovan)}>
        Sign in as Donovan
      </button>
      <button onClick={() => signIn(users.alice)}>
        Sign in as Alice
      </button>
      <button onClick={() => signIn(null)}>Sign out</button>
      <UserContext.Consumer>
        {(user) => {
          return (
            <div>
              <h3 style={{ margin: '5px 0' }}>
                User: {user ? `${user.name}, ${user.age}` : ''}
              </h3>
            </div>
          )
        }}
      </UserContext.Consumer>
    </>
  )
}
```

我们可以看到 `ThemeContext.Consumer` 组件的 Render Props 传入的就是 `theme` 全局数据；而 `UserContext.Consumer` 传入的则是 `user` 全局数据，最终效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_context_component_composition_render_props_context_sample2.gif)

### 1.5 将状态转换函数也透过 Context 传递

前面我们注意到改变全局数据的方法如 `toggleTheme`、`signIn` 都是透过普通数据流 props 一层层传递下来的，其实我们也可以将相关的全局数据更新函数也放入上下文对象当中

- `src/context/Version4.jsx`

```js
class Version4 extends Component {
  constructor(props) {
    super(props)

    this.toggleTheme = this.toggleTheme.bind(this)

    this.state = {
      theme: themes.light,
      // 将状态改变也通过 context 上下文传递
      toggleTheme: this.toggleTheme,
    }
  }

  toggleTheme() {
    this.setState({
      theme:
        this.state.theme === themes.light
          ? themes.dark
          : themes.light,
    })
  }

  render() {
    return (
      <div>
        <ThemeContext.Provider value={this.state}>
          <ToolBar />
        </ThemeContext.Provider>
      </div>
    )
  }
}

export default Version4
```

这时候中间组件就完全不会出现任何与全局数据相关的部分

```js
function ToolBar() {
  return <ThemedButton>Change theme</ThemedButton>
}
```

最后直接透过设置 `contextType` 获取全局数据和更新函数

```js
class ThemedButton extends Component {
  static contextType = ThemeContext

  render() {
    const { theme, toggleTheme } = this.context
    return (
      <button
        onClick={toggleTheme}
        style={{
          backgroundColor: theme.background,
          color: theme.foreground,
        }}
      >
        {this.props.children}
      </button>
    )
  }
}
```

### 1.6 Context 小结：为什么少用 Context？

到此我们已经看过 Context 的各种使用场景和使用方式了，现在我们再回头来谈谈前面说过的：**不要为了仅仅只是简化 props 而使用 Context**。

Context 确实能够省略透过 props 传递数据的麻烦事，但是使用 Context 实现存在一个致命的缺陷，所有 Consumer / contextType 相关的组件其实是与全局数据强关联的，所以一旦数据改变的话所有依赖于此的组件都会强制更新。

也就是说如果我们把原本需要用 props 传递的共享数据一股脑塞入 Context 当中，会变成如下：

```js
<ThemeContext.Provider value={{
    props1: value1,
    props2: value2,
    props3: value2
}}>
```

看起来好像没问题，但是其实实际上子组件当中不同组件可能仅仅只是依赖于其中一个数据，然而 Context 更新数据是全局的，也就是说当我们更新 `props1` 的时候，可能造成依赖于 `props2、props3` 的子组件也都一起重新渲染，造成额外而不必要的渲染浪费性能。

#### 1.6.1 使用规范(推荐)

至此我们已经知晓 Context 的使用模式和缺陷，总归就是一句话：

> Context 用于传递真正需要被多个组件共同需要的 **全局数据**

而当我们仅仅只是需要简化数据在组件树透过 props 一层层传递的麻烦事的话，则应该使用下面一个段落要说明的 **组件组合(Component Composition)** 的方式。

## 2. 组件组合(Component Composition)

前面我们提到了，如果我们仅仅只是为了规避层层传递 props 的风险，而不是要使用真正的全局数据的时候，就应该避免使用 Context，而是使用 **组件组合** 的概念。

### 2.1 什么叫组件组合？

组件组合的核心思想在于，既然我们不希望共享数据透过 props 一层层传递下去，那么我们就先在顶层将绑定好数据的组件传入 props，而子组件则只需要指定传入的组件真实放置的位置就行

也就是从下面这种形式

```js
function ComponentA() {
    const data = {/* ... */}
    return <ComponentB data={data} />
}

function ComponentB(props) {
    return <ComponentC data={props.data} />
}

function ComponentC(props) {
    return <div>{props.data.toString()}</div>
}
```

变成这种形式

```js
function ComponentA() {
    const data = {/* ... */}
    return <ComponentB componentC={<ComponentC data={data} />} />
}

function ComponentB(props) {
    return props.componentC
}

function ComponentC(props) {
    return <div>{props.data.toString()}</div>
}
```

甚至进一步的

```js
function ComponentA() {
    const data = {/* ... */}
    const componentC = <div>{props.data.toString()}</div>
    return <ComponentB componentC={componentC} />
}

function ComponentB(props) {
    return props.componentC
}
```

如此一来我们就不需要透过 props 传递数据，而是直接传递绑定好数据的组件到指定位置，这就是组件的核心概念。

这种做法相当于是一种 **控制反转(Inversion of Controll)** 的体现，将子组件的渲染逻辑提升到更高级的组件，反过来由高级组件来提供子组件的实现，而原本的中间组件变成只需要接受父组件传递过来的部件绑定到正确的位置就行

下面我们就来看看组件组合的不同实现方式

### 2.2 组件组合第一式：渲染子节点数组

首先第一种就是最常见的 `children` 属性。在 React 中 children 是一个非常特别的属性，当我们使用组件并在组件标签之间放入数据的时候，它其实就会作为 chilren 属性的一员出现，也就是说下面两种实现是等价的

```js
const component = <div>A Component</div>
const Wrapper = <div children={component} />

// 等价于

const component = <div>A Component</div>
const Wrapper = <div>{component}</div>
```

而当 children 种存在多个元素的时候，他就自然而然变成一个数组，也就是列表渲染的形式，下面就是我们的演示代码

- `src/composition/index.jsx`

```js
import React, { Component } from 'react'

import SideBar from './SideBar'
import './index.css'
import Header from './Header'
import Main from './Main'
import Footer from './Footer'

function MenuItem(props) {
  const { label, title, onClick } = props
  return (
    <div className="item" title={title} onClick={onClick}>
      {label}
    </div>
  )
}

class Composition extends Component {
  constructor(props) {
    super(props)
    this.state = {
      menuItems: [
        { label: 'Menu Item 1', title: 'go menu item 1' },
        { label: 'Menu Item 2', title: 'go menu item 2' },
        { label: 'Menu Item 3', title: 'go menu item 3' },
        { label: 'Menu Item 4', title: 'go menu item 4' },
        { label: 'Menu Item 5', title: 'go menu item 5' },
      ],
    }
  }

  handlerMenuItemSelect(item) {
    console.log('item', item)
  }

  render() {
    const items = this.state.menuItems.map((item) => (
      <MenuItem
        {...item}
        key={item.label}
        onClick={() => this.handlerMenuItemSelect(item)}
      />
    ))

    return (
      <div className="composition">
        <div className="container">
          <SideBar>{items}</SideBar>
          {/* ... */}
        </div>
      </div>
    )
  }
}

export default Composition
```

我们可以看到，示例中我们直接在顶层组件根据数据组装好侧边栏(`SideBar`)需要使用的导航列表

```js
function MenuItem(props) {
  const { label, title, onClick } = props
  return (
    <div className="item" title={title} onClick={onClick}>
      {label}
    </div>
  )
}

// ...

    this.state = {
      menuItems: [
        { label: 'Menu Item 1', title: 'go menu item 1' },
        { label: 'Menu Item 2', title: 'go menu item 2' },
        { label: 'Menu Item 3', title: 'go menu item 3' },
        { label: 'Menu Item 4', title: 'go menu item 4' },
        { label: 'Menu Item 5', title: 'go menu item 5' },
      ],
    }

    // ...

    const items = this.state.menuItems.map((item) => (
      <MenuItem
        {...item}
        key={item.label}
        onClick={() => this.handlerMenuItemSelect(item)}
      />
    ))
```

然后将列表放入到 `SideBar` 组件的中间，也就是作为 `children` 属性传入

```js
render() {
  return (
    <SideBar>{items}</SideBar>
    // ...
```

最后在 `SideBar` 组件内部则是直接放置到目标位置即可

- `src/composition/SideBar.jsx`

```js
import React, { Component } from 'react'

class SideBar extends Component {
  render() {
    return <div className="sidebar">{this.props.children}</div>
  }
}

export default SideBar
```

效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_context_component_composition_render_props_composition_sample1.png)

### 2.3 组件组合第二式：插槽(slot)

前面我们使用 `children` 属性来传递子组件，但是它就只是单一的一个属性，不能非常精确的描述子组件的位置。

第二种实现方式则是透过指定名称的 props 传入子组件，进而指定特定子组件对应的位置，而这种实现被称为 **插槽(slot)**：

- `src/composition/Header.jsx`

```js
import React, { Component } from 'react'

class Header extends Component {
  render() {
    const { left, right } = this.props
    return (
      <div className="header">
        <div className="left">{left}</div>
        <div className="center">
          <h2>Header</h2>
        </div>
        <div className="right">{right}</div>
      </div>
    )
  }
}

export default Header
```

首先我们先定义一个 `Header` 组件，并留下 `left、right` 两个插槽，分别放置于 "Header" 文字的两侧

- `src/composition/index.jsx`

```js
function Title(props) {
  return <h3>{props.title}</h3>
}

function UserInfo(props) {
  return <h4>{props.username}</h4>
}

class Composition extends Component {
  render() {
    return (
      <div className="composition">
        <div className="container">
          <SideBar>{items}</SideBar>
          <div className="container vertical">
            <Header
              left={<Title title="This is a title for Header" />}
              right={<UserInfo username="Alice" />}
            />
          </div>
        </div>
      </div>
    )
  }
}
```

接下来我们将 `Title` 组件传入 `left` 属性；而将 `UserInfo` 组件传入 `right` 属性，这样对于外部组件来说我只要指定传入的属性便等同于传入指定位置，而不用关心具体渲染的位置，也不需要额外的 props 传递，效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_context_component_composition_render_props_composition_sample2.png)

### 2.4 组件组合第三式：特殊实例

第三种实现则是针对不同实现效果预先提取一些特殊绑定值的实例。

我们在开发组件的时候通常会遵从一个原则：尽量使得最底层的组件越简单愈好，最好只简单依赖于 props 来实现结果渲染，也就是说我们会先定义一个如下的抽象组件：

- `src/composition/Footer.jsx`

```js
function ColoredBlock(props) {
  return (
    <div
      style={{
        width: '50px',
        height: '50px',
        backgroundColor: props.color,
      }}
    ></div>
  )
}
```

但是每次要使用的时候都要再自己传入属性或是根据更高级的组件传递下来的某个值来渲染组件。事实上，我们还可以预先定义几个传入特定值的组件实例，同时将这些实例也做成另一个组件如下

```js
function SkyBlueBlock() {
  return <ColoredBlock color="skyblue" />
}

function CoralBlock() {
  return <ColoredBlock color="coral" />
}

function LimeBlock() {
  return <ColoredBlock color="limegreen" />
}

function CrimsonBlock() {
  return <ColoredBlock color="crimson" />
}
```

如此一来我们使用的时候就不在需要传入 `color` 属性进行绑定，而是好像使用静态组件一样，拿来直接用就行了

```js
class Footer extends Component {
  render() {
    return (
      <div className="footer">
        <LimeBlock />
        <SkyBlueBlock />
        footer
        <CoralBlock />
        <CrimsonBlock />
      </div>
    )
  }
}
```

效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_context_component_composition_render_props_composition_sample3.png)

### 2.5 组件组合第四式：Render Props 传递渲染函数

最后一种实现比较特别，记得我们前面使用 props 传入绑定好属性的的组件时，都是直接传入一个组件实例，然后在子组件中直接将部件放置到固定的位置，这时候我们是不是还可以传入一个函数，使得部件延迟到子组件中再进行绑定呢？下面我们就来试试看

1. 首先我们先定义一个跟踪鼠标位置的组件

- `src/composition/Main.jsx`

```js
class Mouse extends Component {
  constructor(props) {
    super(props)
    this.state = {
      x: 0,
      y: 0,
    }
    this.handleMouseMove = this.handleMouseMove.bind(this)
  }

  handleMouseMove(e) {
    this.setState({
      x: e.clientX,
      y: e.clientY,
    })
  }

  render() {
    const { x, y } = this.state
    return (
      <div className="backbone" onMouseMove={this.handleMouseMove}>
        <span
          style={{ position: 'relative', top: '10px', left: '10px' }}
        >
          mouse position: ({x}, {y})
        </span>
      </div>
    )
  }
}
```

2. 然后直接在父组件中使用

```js
class Main extends Component {
  render() {
    return (
      <div className="main">
        <Mouse />
      </div>
    )
  }
}
```

效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_context_component_composition_render_props_composition_sample4.png)

接下来我们想要渲染一个方块，跟着鼠标一起移动。

首先是方块类

```js
function Square(props) {
  const { x, y } = props.position
  const width = 100
  return (
    <div
      style={{
        backgroundColor: 'skyblue',
        position: 'fixed',
        left: x - width / 2,
        top: y - width / 2,
        width: width,
        height: width,
      }}
    />
  )
}
```

接下来我们可能想要直接放到 `Mouse` 组件里面如下

```js
class Mouse extends Component {
  // ...

  render() {
    const { x, y } = this.state
    return (
      <div className="backbone" onMouseMove={this.handleMouseMove}>
        <span
          style={{ position: 'relative', top: '10px', left: '10px' }}
        >
          mouse position: ({x}, {y})
        </span>
        <Square position={this.state} />
      </div>
    )
  }
}
```

但是这样有一个问题在于 `Mouse` 组件就与 `Square` 组件强耦合了，这是我们不想看到的，当我们需要将 `Mouse` 组件中的 `Square` 组件改成其他组件的时候就会遇到大麻烦。

这时候我们就可以用上 Render Props 的概念，透过父组件传入需要的部件(`Square`)；更进一步，这个部件依旧能够与子组件(`Mouse`)相关联，这时候我们就不能直接传入组件实例，而是一个 Render Props，也就是一个延迟绑定组件实例的方法如下

```js
class Main extends Component {
  render() {
    return (
      <div className="main">
        <Mouse
          render={(position) => <Square position={position} />}
        />
      </div>
    )
  }
}
```

我们想要在 `Mouse` 里面渲染 `Square` 没错，但是又必须等到拿到 `Mouse` 里面的位置信息才能够进行渲染，所以我们传入一个接受 `position` 为参数才生成真正的 `Square` 实例的函数，而在 `Mouse` 内部我们就可以这样写

```js
class Mouse extends Component {
  // ...

  render() {
    const { x, y } = this.state
    return (
      <div className="backbone" onMouseMove={this.handleMouseMove}>
        <span
          style={{ position: 'relative', top: '10px', left: '10px' }}
        >
          mouse position: ({x}, {y})
        </span>
        {this.props.render(this.state)}
      </div>
    )
  }
}
```

我们可以看到，虽然 `Mouse` 并不知道最终被渲染到该位置的组件是谁，但是我知道只要调用 `props.render` 方法并传入自己的位置信息，就能够返回一个绑定了自己的鼠标信息的某个组件，这就是 Render Props

效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_context_component_composition_render_props_composition_sample5.gif)

注意：这里说 Render Props 的意义只是 "传入某个能生成真正组件实例的函数"，也就是说我们并不是一定要用 `render` 属性，随便什么属性都好，核心思想就是传入一个根据子组件信息生成真正的部件一个设计模式。

是不是很像前面 `Context.Consumer` 的用法呢？

```js
render() {
    return (
        <Context.Consumer>
            {value => <Component />}
        </Context.Consumer>
    )
}
```

没错其实 `Context.Consumer` 就是利用 Render Props 的做法，将全局数据传入用户自定义的 `value => Component` 函数中，完整对全局数据的绑定的。

# 结语

本篇介绍了 Context 上下文的用法，还有一些组件组合的使用示例，最后引出 Render Props 的特性和用法，以及在 `Context.Consumer` 中的具体实现。这些都是实际开发的时候非常实用的特性，供大家参考。

# 其他资源

## 参考连接

| Title                                 | Link                                                                                                                                 |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| React 官方 - Context                  | [https://react.docschina.org/docs/context.html](https://react.docschina.org/docs/context.html)                                       |
| React 官方 - 组合 vs 继承             | [https://react.docschina.org/docs/composition-vs-inheritance.html](https://react.docschina.org/docs/composition-vs-inheritance.html) |
| React 官方 - Render Props             | [https://react.docschina.org/docs/render-props.html](https://react.docschina.org/docs/render-props.html)                             |
| React 中 Context 和 contextType的使用 | [https://blog.csdn.net/landl_ww/article/details/93514944](https://blog.csdn.net/landl_ww/article/details/93514944)                   |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_context_component_composition_render_props](https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_context_component_composition_render_props)
