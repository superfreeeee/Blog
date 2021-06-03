# React 升级: Hook API 基础

@[TOC](文章目录)

<!-- TOC -->

- [React 升级: Hook API 基础](#react-升级-hook-api-基础)
- [系列文章](#系列文章)
- [前言](#前言)
  - [Hook API](#hook-api)
- [正文](#正文)
  - [0. 类组件 vs 函数组件](#0-类组件-vs-函数组件)
  - [1. State Hook: useState](#1-state-hook-usestate)
    - [1.1 实现差异](#11-实现差异)
    - [1.2 useState API](#12-usestate-api)
  - [2. Effect Hook: useEffect](#2-effect-hook-useeffect)
    - [2.1 实现差异](#21-实现差异)
    - [2.2 useEffect API](#22-useeffect-api)
    - [2.3 useRef API](#23-useref-api)
  - [3. Context Hook: useContext](#3-context-hook-usecontext)
    - [3.1 Context & Provider HOC](#31-context--provider-hoc)
    - [3.2 Consumer & 实现差异](#32-consumer--实现差异)
    - [3.3 useContext API](#33-usecontext-api)
  - [4. Customer Hook 自定义Hook](#4-customer-hook-自定义hook)
    - [4.1 Hook 的限制/规范](#41-hook-的限制规范)
    - [4.2 自定义 Hook 将逻辑封装](#42-自定义-hook-将逻辑封装)
    - [4.3 自定义 Hook 之前](#43-自定义-hook-之前)
      - [4.3.1 自定义 Hook: useName](#431-自定义-hook-usename)
      - [4.3.2 自定义 Hook: useTimer](#432-自定义-hook-usetimer)
    - [4.4 自定义 Hook 之后完整版本(由上到下的视角)](#44-自定义-hook-之后完整版本由上到下的视角)
      - [4.4.1 由上到下看 useName](#441-由上到下看-usename)
      - [4.4.2 由上到下看 useTimer](#442-由上到下看-usetimer)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 系列文章

- [React 入门: 核心特性全面解析](https://blog.csdn.net/weixin_44691608/article/details/117343164)
- [React 高级指引: 从状态提升到高阶组件(HOC)](https://blog.csdn.net/weixin_44691608/article/details/117440794)
- [React 高阶指引: Context 上下文 & 组件组合 & Render Props](https://blog.csdn.net/weixin_44691608/article/details/117458645)
- [React 核心特性: 3 种创建 Refs 引用 & 2 种 Refs 转发的方法](https://blog.csdn.net/weixin_44691608/article/details/117478115)

# 前言

React 16.8 版本提出了一个重大的更新：**Hook API** 的问世。

在[前面的章节](https://blog.csdn.net/weixin_44691608/article/details/117343164)我们介绍了一些 React 基础能力，也提到在 React 当中存在类组件(使用 ES6 class 关键字定义)和函数组件(使用 function 关键字定义)两种组件的定义方式。

然而在旧版本中函数组件只能作为纯渲染组件来使用，纯渲染组件的意思是只能完全透过 props 来直接渲染内容，既不能保有自己的状态，也缺乏很多几乎所有类组件能够使用的能力和特性。

## Hook API

今天我们就来介绍，React 团队推出的 Hook API。使用 Hook API 使得我们能将类组件的能力也能在函数组件里面实现，很神奇吧！

由于 Hook API 算是一种渐进式非破坏性的改动，我们可以同时使用类组件和 Hook，当发现存在 Hook 没办法实现的能力时依旧能够转回到使用类组件来使用。

下面我们主要介绍 4 个基础的 Hook 加上自定义 Hook 的使用，带大家进入 React Hook 的怀抱，一起体验 React Hook 的强大之处！

备注：[React Conf - 2018 关于 Hook 的介绍](https://www.youtube.com/watch?v=dpw9EHDh2bM&feature=youtu.be)

# 正文

## 0. 类组件 vs 函数组件

首先我们先复习以下两种组件的创建方法

1. 使用 ES6 的 class 创建 **类组件**

```js
class ClassComponent extends React.Component {
    render() {
        return <div>Hello World</div>
    }
}
```

类组件中只有 `render` 方法是必须实现的，由 `render` 方法返回组件的视图结果，并会在组件更新的时候重复调用，这是类组件的限制

2. 使用 function 关键字创建 **函数组件**

```js
function FunctionComponent(props) {
    return <div>Hello World</div>
}
```

对于函数组件来说整个函数其实就好像一个 `render` 方法一样，而这也是为什么原来的函数组件不能暴露状态，因为他不存在自己的组件实例，他自己本身就是一个重复调用的 `render` 方法

下面我们就开始看看如何透过 React Hook API 将类组件改造成等价的函数组件

## 1. State Hook: useState

首先第一个特性是 **state 状态**，我们使用类组件很大一个原因就是它能够保有自己的状态，同时状态更新的时候 React 会自动帮我们调用 render 进行重新渲染，实例如下

- `src/useState/ClassVersion.jsx`

```js
import React, { Component } from 'react'

class ClassVersion extends Component {
  constructor(props) {
    super(props)
    this.state = {
      name: 'Alice',
    }
    this.handleNameChange = this.handleNameChange.bind(this)
  }

  handleNameChange(e) {
    this.setState({ name: e.target.value })
  }

  render() {
    return (
      <div>
        <h3>In Class Component</h3>
        <div>name: {this.state.name}</div>
        <input
          value={this.state.name}
          onChange={this.handleNameChange}
        />
      </div>
    )
  }
}

export default ClassVersion
```

在这个例子中我们进一步将这个状态转变为动态的，并定义一个 `handleNameChange` 来更新状态

下面我们就使用 `useState` 这个钩子来在函数组件中实现同样的效果

- `src/useState/FunctionVersion.jsx`

```js
import React, { useState } from 'react'

function FunctionVersion() {
  const [name, setName] = useState('Alice')

  function handleNameChange(e) {
    setName(e.target.value)
  }

  return (
    <div>
      <h3>In Function Component</h3>
      <div>name: {name}</div>
      <input value={name} onChange={handleNameChange} />
    </div>
  )
}

export default FunctionVersion
```

`useState` 会返回两个值(透过 ES6 的数组解构赋值)，第一个是状态的值(`name`)，第二个则是更新状态的方法(`setName`)，这时候我们就可以定义一个输入处理函数 `handleNameChange` 绑定到 `onChange` 事件上，调用 `setName` 进行更新，React 知道我们更新了组件状态之后就会自动重新渲染函数组件，而且这时候 `useState` 返回的是新的值，最终效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_hook_basic_useState_sample.gif)

### 1.1 实现差异

下面我们比较以下两种实现完成同样功能的差异

- 对于类组件
  - 使用 `this.state` 初始化状态
  - 定义 `handleNameChange` 输入处理函数，调用 `setState` 更新状态
  - `this.handleNameChange = this.handleNameChange.bind(this)` 绑定函数
- 对于函数组件
  - 使用 `useState(initialValue)` 初始化状态
  - 定义 `handleNameChange` 输入处理函数，调用 `setName` 更新状态

我们发现要做的工作其实都差不多，但是代码量就少了许多，帮助我们从原有的 React 语法中解放出来，是我们的代码更精炼也对组件能力有更直接的表达

### 1.2 useState API

最后我们重新明确一下这个 `useState` 要怎么用，函数标签如下

```js
const [value, setValue] = useState(initialValue)
```

参数为初始状态，返回最新的状态值和状态更新函数

## 2. Effect Hook: useEffect

第二项能力要说明在组件中的副作用。前面我们提到的 `state` 通常是组件内的状态变化，属于 React 组件内的受控范围，然而在开发的过程中我们可能还需要对一些外部状态随着组件的更新同步的更新，相当于是一种组件之外的副作用

下面的例子中，我们将组件内的状态同步到 `document.title`，实时的修改页面的 title

- `src/useEffect/ClassVersion.jsx`

```js
import React, { Component } from 'react'

class ClassVersion extends Component {
  constructor(props) {
    super(props)
    this.state = {
      name: 'Alice',
    }
    this.handleNameChange = this.handleNameChange.bind(this)
    this.ref = React.createRef()
  }

  componentDidMount() {
    document.title = `name: ${this.state.name}`
    this.ref.current.focus()
  }

  componentDidUpdate() {
    document.title = `name: ${this.state.name}`
  }

  handleNameChange(e) {
    this.setState({ name: e.target.value })
  }

  render() {
    return (
      <div>
        <h3>In Class Component</h3>
        <div>name: {this.state.name}</div>
        <input
          ref={this.ref}
          value={this.state.name}
          onChange={this.handleNameChange}
        />
      </div>
    )
  }
}

export default ClassVersion
```

我们需要在 `componentDidMount、componentDidUpdate` 生命周期钩子中，在组件初始化后、更新时等时间点更新 `document.title` 的值

下面我们看 Hook 的版本

- `src/useEffect/FunctionVersion.jsx`

```js
import React, { useEffect, useRef, useState } from 'react'

function FunctionVersion() {
  const [name, setName] = useState('Alice')

  function handleNameChange(e) {
    setName(e.target.value)
  }

  useEffect(() => {
    document.title = `name: ${name}`
  })

  const ref = useRef(null)

  useEffect(() => {
    ref.current.focus()
  }, [])

  return (
    <div>
      <h3>In Function Component</h3>
      <div>name: {name}</div>
      <input ref={ref} value={name} onChange={handleNameChange} />
    </div>
  )
}

export default FunctionVersion
```

而在函数组件中我们只需要调用一个 `useEffect` 并传入一个回调函数，这个回调函数将在每次组件更新的时候都被调用，相当于是 `componentDidMount + componentDidUpdate` 组合使用的效果

效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_hook_basic_useEffect_sample.gif)

### 2.1 实现差异

我们看到在类组件中，我们需要分别在 `componentDidMount、componentDidUpdate` 两个钩子中定义组件初始化、更新时的逻辑

而在函数组件中我们只需要调用 `useEffect` 并给出回调函数就完成了一样的事情

### 2.2 useEffect API

函数标签如下

```js
useEffect(callback, [...observeValues])
```

第一个参数就是要执行的回调函数，就好像是副作用 effect 一样；而第二个则是关注列表

前面我们提过 `callback` 会在每次组件更新的时候都调用，然而有时候我们并不是每次都要调用，所以我们就可以将关注的值写到第二个参数的数组当中，只有当关注列表中的值改变的时候回调函数才会被调用

### 2.3 useRef API

前面的代码中我们还用到了 `useRef` API，这个方法与类组件中的 `React.createRef` 等价的

```js
const ref = useRef(null)
```

同时我们透过定义一个不观察任何值的副作用

```js
useEffect(() => {
  ref.current.focus()
}, [])
```

当我们第二个观察列表为空的时候，就只会在第一次调用的时候执行，也就是与 `componentDidMount` 等价

上面的代码的意义就是第一次渲染的时候自动 focus 输入框

## 3. Context Hook: useContext

第三个我们要介绍 Context 的 Hook 使用

### 3.1 Context & Provider HOC

首先我们先创建我们的 Context 对象和全局数据值

- `src/useContext/context.js`

```js
import React from 'react'

export const themes = {
  light: {
    foreground: '#000000',
    background: '#eeeeee',
  },
  dark: {
    foreground: '#ffffff',
    background: '#222222',
  },
}

export const ThemeContext = React.createContext({
  theme: themes.dark,
  toggleTheme: () => {},
})

export const languages = {
  chinese: '你好',
  english: 'hello',
}

export const LanguageContext = React.createContext({
  language: languages.chinese,
  changeLanguage: () => {},
})
```

然后我们定义一个高阶组件来提供两个 Context 的 Provider

- `src/useContext/withContexts.js`

```js
import React, { Component, useState } from 'react'
import {
  LanguageContext,
  languages,
  ThemeContext,
  themes,
} from './context'

const withContexts = (WrappedComponent, title) => {
  return () => {
    const [theme, setTheme] = useState(themes.light)
    const [language, setLanguage] = useState(languages.chinese)

    const toggleTheme = () => {
      setTheme(theme === themes.light ? themes.dark : themes.light)
    }

    const toggleLanguage = () => {
      setLanguage(
        language === languages.chinese
          ? languages.english
          : languages.chinese
      )
    }

    return (
      <div>
        <h3>{title}</h3>
        <ThemeContext.Provider value={{ theme, toggleTheme }}>
          <LanguageContext.Provider
            value={{ language, toggleLanguage }}
          >
            <WrappedComponent />
          </LanguageContext.Provider>
        </ThemeContext.Provider>
      </div>
    )
  }
}

export default withContexts
```

### 3.2 Consumer & 实现差异

下面我们就来看最核心的 Consumer 使用，一样分为类组件和函数组件

- `src/useContext/ClassVersion.js`

```js
import React, { Component } from 'react'
import { LanguageContext, ThemeContext } from './context'
import withContexts from './withContexts'

class ClassVersion extends Component {
  render() {
    return (
      <ThemeContext.Consumer>
        {({ theme, toggleTheme }) => (
          <LanguageContext.Consumer>
            {({ language, toggleLanguage }) => (
              <div>
                <div
                  style={{
                    width: '200px',
                    height: '40px',
                    backgroundColor: theme.background,
                    color: theme.foreground,
                  }}
                >
                  {language}
                </div>
                <button onClick={toggleTheme}>改变主题</button>
                <button onClick={toggleLanguage}>改变语言</button>
              </div>
            )}
          </LanguageContext.Consumer>
        )}
      </ThemeContext.Consumer>
    )
  }
}

export default withContexts(ClassVersion, 'In Class Component')
```

我们看到在多 Context 的场景之下，我们使用 `Context.Consumer` 来获取上下文的全局数据来进行渲染(单 Context 的时候我们可以使用 `contextType` 初步简化)，同时经过多层的嵌套，其实对于单纯的展示逻辑来说还是有点累赘

下面我们看看使用 Context Hook 的函数组件版本

- `src/useContext/FunctionVersion.js`

```js
import React, { useContext } from 'react'
import { LanguageContext, ThemeContext } from './context'
import withContexts from './withContexts'

function FunctionVersion() {
  const { theme, toggleTheme } = useContext(ThemeContext)
  const { language, toggleLanguage } = useContext(LanguageContext)

  return (
    <div>
      <div
        style={{
          width: '200px',
          height: '40px',
          backgroundColor: theme.background,
          color: theme.foreground,
        }}
      >
        {language}
      </div>
      <button onClick={toggleTheme}>改变主题</button>
      <button onClick={toggleLanguage}>改变语言</button>
    </div>
  )
}

export default withContexts(FunctionVersion, 'In Function Component')
```

我们可以看到函数组件的版本就非常直观了，`useContext` 说明我要使用 Context 全局输入，并传入指定 Context 作为参数(这时候单个还是多个 Context 就没关系了)，然后在组件内就可以直接使用 `theme, toggleTheme, language, toggleLanguage` 等全局数据，不需要再套上一层 `Context.Consumer` 的外壳，可以说是 `contextType` 和 `Consumer` 的集大成版本

最终效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_hook_basic_useContext_sample.gif)

### 3.3 useContext API

我们从上面的例子可以看到，`useContext` 相当于是函数组件的 `contextTYpe` 参数传入的是我们要消费的 Context，具体使用的函数标签如下

```js
const value = useContext(ContextType)
```

## 4. Customer Hook 自定义Hook

前面我们介绍了 `useState、useEffect、useRef、useContext` 等钩子，其实已经能够覆盖非常大一部分的日常开发场景了，其他还有更多的 Hook 能满足更多的能力，本篇就不再介绍了。

在最后一个小节之中我要告诉你 Hook 最强大的部分：**自定义 Hook**

### 4.1 Hook 的限制/规范

在讨论自定义 Hook 之前我们要先讨论一下 Hook 的使用规则，我们拿 `useState` 举例

- `src/useEffect/FunctionVersion.jsx`

```js
import React, { useEffect, useRef, useState } from 'react'

function FunctionVersion() {
  const [name, setName] = useState('Alice')

  function handleNameChange(e) {
    setName(e.target.value)
  }

  useEffect(() => {
    document.title = `name: ${name}`
  })

  const ref = useRef(null)

  useEffect(() => {
    ref.current.focus()
  }, [])

  return (
    <div>
      <h3>In Function Component</h3>
      <div>name: {name}</div>
      <input ref={ref} value={name} onChange={handleNameChange} />
    </div>
  )
}

export default FunctionVersion
```

从代码上看起来总是调用同一个 `useState`，那 React 是怎么确定组件更新之后会返回正确的 name 呢？其实 React 是根据 Hook `useState` 的调用顺序来保证我们的组件运行正确，也就是说如果我们将 Hook 放到一个条件语句或是循环语句中，多次渲染下钩子的调用顺序就不再一致，也就造成组件崩溃

```js
if (condition) {
    useState()
}
```

所以使用 Hook 有一个不寻常但是非常重要的限制：

> 只能在组件内顶层 or 其他 Hook 中调用 Hook API

这是 React 为我们完成组件状态完整性的保证

如果我们真的需要条件执行逻辑的话应该写在 Hook 里面

```js
useEffect(() => {
    if (condition) {
        // do something
    }
})
```

### 4.2 自定义 Hook 将逻辑封装

从前面的规则我们知道，也就是说其实 Hook 的调用是可以与相关逻辑被封装到另一个函数当中，只要保证调用 Hook 的调用顺序即可

新封装的方法就是我们的 **自定义 Hook**，命名规范我们遵照 React 官方推荐的以 `use` 做前缀

### 4.3 自定义 Hook 之前

我们先来看看使用自定义 Hook 之前的一个示例

- `src/customer/BeforeCustomer.jsx`

```js
import React, { useEffect, useState } from 'react'

function BeforeCustomer() {
  const [name, setName] = useState('Alice')

  function handleNameChange(e) {
    setName(e.target.value)
  }

  useEffect(() => {
    document.title = `name: ${name}`
  }, [name])

  const [time, setTime] = useState(new Date())

  useEffect(() => {
    const timer = setTimeout(() => {
      setTime(new Date())
    }, 1000)
    return () => clearTimeout(timer)
  }, [time])

  return (
    <div>
      <h2>封装之前</h2>
      <div>name: {name}</div>
      <input value={name} onChange={handleNameChange} />
      <div>当前时间: {time.toLocaleString()}</div>
    </div>
  )
}

export default BeforeCustomer
```

随着组件的逻辑定义得越来愈丰富，代码量也会越来愈庞大，这并不是一件坏事，但是阅读起来还是会有点辛苦，所以我们可以将相关的逻辑再透过自定义 Hook 封装起来，甚至可以封装为可重用的逻辑

#### 4.3.1 自定义 Hook: useName

如下面跟表单输入相关的逻辑有这些

```js
const [name, setName] = useState('Alice')

function handleNameChange(e) {
  setName(e.target.value)
}
```

另外还有根据 `name` 同步到 `document.title` 的逻辑

```js
useEffect(() => {
  document.title = `name: ${name}`
}, [name])
```

这时候我们就可以封装出一个自定义 Hook

```js
function useName(initName) {
  const [name, setName] = useState(initName)

  function handleNameChange(e) {
    setName(e.target.value)
  }

  useEffect(() => {
    document.title = `name: ${name}`
  }, [name])

  return [name, handleNameChange]
}
```

最后只要返回我们关心和需要的值或方法即可

```js
function AfterCustomer() {
  const [name, onNameChange] = useName('Alice')

  return (
    <div>
      <h2>封装之后</h2>
      <div>name: {name}</div>
      <input value={name} onChange={onNameChange} />
    </div>
  )
}
```

#### 4.3.2 自定义 Hook: useTimer

另外我们还有一个展示当前时间的逻辑

```js
const [time, setTime] = useState(new Date())

useEffect(() => {
  const timer = setTimeout(() => {
    setTime(new Date())
  }, 1000)
  return () => clearTimeout(timer)
}, [time])
```

我们也可以封装成一个自定义 Hook

```js
function useTimer() {
  const [time, setTime] = useState(new Date())

  useEffect(() => {
    const timer = setTimeout(() => {
      setTime(new Date())
    }, 1000)
    return () => clearTimeout(timer)
  }, [time])

  return time
}
```

最后组件内我们只需要当前最新的时间

```js
function AfterCustomer() {
  const time = useTimer()

  return (
    <div>
      <h2>封装之后</h2>
      <div>当前时间: {time.toLocaleString()}</div>
    </div>
  )
}
```

### 4.4 自定义 Hook 之后完整版本(由上到下的视角)

最后我们看到完整版本中，组件的逻辑变得清楚了

- `src/customer/AfterCustomer.jsx`

```js
import React, { useEffect, useState } from 'react'

function AfterCustomer() {
  const [name, onNameChange] = useName('Alice')
  const time = useTimer()

  return (
    <div>
      <h2>封装之后</h2>
      <div>name: {name}</div>
      <input value={name} onChange={onNameChange} />
      <div>当前时间: {time.toLocaleString()}</div>
    </div>
  )
}

export default AfterCustomer
```

整个组件只有：

1. 我要"用"一个名字

```js
const [name, onNameChange] = useName('Alice')
```

2. 我要"用"一个时间

```js
const time = useTimer()
```

3. 最后根据拿到的值进行渲染

```js
return (
 <div>
   <h2>封装之后</h2>
   <div>name: {name}</div>
   <input value={name} onChange={onNameChange} />
   <div>当前时间: {time.toLocaleString()}</div>
 </div>
)
```

逻辑非常清晰简单

#### 4.4.1 由上到下看 useName

当我们想理解 `name` 是怎么来的是怎么更新的我们就可以看到 `useName` 钩子里面

```js
function useName(initName) {
  const [name, setName] = useState(initName)

  function handleNameChange(e) {
    setName(e.target.value)
  }

  useEffect(() => {
    document.title = `name: ${name}`
  }, [name])

  return [name, handleNameChange]
}
```

然后得知他是来源于一个状态

```js
const [name, setName] = useState(initName)
```

跟一个状态更新函数

```js
function handleNameChange(e) {
 setName(e.target.value)
}
```

还有一个名称改变的时候同步到 `document.title` 的副作用

```js
useEffect(() => {
 document.title = `name: ${name}`
}, [name])
```

#### 4.4.2 由上到下看 useTimer

另一个 `time` 也是一样，我们想知道实时的时间怎么来的，就看到 `useTimer` Hook

```js
function useTimer() {
  const [time, setTime] = useState(new Date())

  useEffect(() => {
    const timer = setTimeout(() => {
      setTime(new Date())
    }, 1000)
    return () => clearTimeout(timer)
  }, [time])

  return time
}
```

具体的时间也是一个状态

```js
const [time, setTime] = useState(new Date())
```

然后每次时间更新之后设置一个定时器在下一秒更新时间

```js
useEffect(() => {
 const timer = setTimeout(() => {
   setTime(new Date())
 }, 1000)
 return () => clearTimeout(timer)
}, [time])
```

到此完结。

# 结语

本篇介绍了 React Hook API 的几个基本示例，最后从上到下的视角我们发现我们不仅仅是能从模版的角度切分组件，组件内部也可以透过自定义 Hook 的方式切分我们的代码逻辑块，使得同一个组件中的状态和相关的逻辑内聚到共同的 Hook 当中，甚至可以在不同的组件中复用自定义 Hook，如下定义一个处理表单输入的自定义 Hook

```js
function useFormInput(initialValue) {
    const [value, setValue] = useState(initialValue)

    function handleValueChange(e) {
        setValue(e.target.value)
    }

    return [value, handleValueChange]
}
```

我们就可以在各种不同的组件中使用 `value={value}、onchange={handleValueChange}`

可以说 React 似乎找到了一个前端 MVVM 模版语法的正确打开方式，真正的将 MVVM 的模版语法最小化，同时强化组件内部状态和逻辑的内聚性，供大家参考。

# 其他资源

## 参考连接

| Title                      | Link                                                                                                           |
| -------------------------- | -------------------------------------------------------------------------------------------------------------- |
| React 官方 - Hook 概览     | [https://react.docschina.org/docs/hooks-overview.html](https://react.docschina.org/docs/hooks-overview.html)   |
| React 官方 - Hook API 索引 | [https://react.docschina.org/docs/hooks-reference.html](https://react.docschina.org/docs/hooks-reference.html) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_hook_basic](https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_hook_basic)
