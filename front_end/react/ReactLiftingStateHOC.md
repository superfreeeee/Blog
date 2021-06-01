# React 高级指引: 从状态提升到高阶组件(HOC)

@[TOC](文章目录)

<!-- TOC -->

- [React 高级指引: 从状态提升到高阶组件(HOC)](#react-高级指引-从状态提升到高阶组件hoc)
- [前言](#前言)
- [正文](#正文)
  - [1. 状态提升](#1-状态提升)
    - [1.1 第一个组件](#11-第一个组件)
    - [1.2 两个实例](#12-两个实例)
    - [1.3 将实例放到一个组件中共享状态](#13-将实例放到一个组件中共享状态)
      - [1.3.1 摄氏 / 华氏温度互相转换](#131-摄氏--华氏温度互相转换)
      - [1.3.2 两个输入框的组件](#132-两个输入框的组件)
    - [1.4 状态提升](#14-状态提升)
      - [1.4.1 共享状态](#141-共享状态)
      - [1.4.2 叶子组件(状态消费者)](#142-叶子组件状态消费者)
      - [1.4.3 状态流图](#143-状态流图)
    - [1.5 状态提升小结](#15-状态提升小结)
  - [2. 高阶组件 HOC](#2-高阶组件-hoc)
    - [2.1 抽象数据源 DataSource](#21-抽象数据源-datasource)
      - [2.1.1 addChangeListener, removeChangeListener, notify 维护数据观察者队列](#211-addchangelistener-removechangelistener-notify-维护数据观察者队列)
      - [2.1.2 addComment, getComments 评论相关](#212-addcomment-getcomments-评论相关)
      - [2.1.3 setBlogPost, getBlogPost 博客帖子相关](#213-setblogpost-getblogpost-博客帖子相关)
      - [2.1.4 填充数据](#214-填充数据)
    - [2.2 直接依赖数据源](#22-直接依赖数据源)
      - [2.2.1 CommentList 评论列表](#221-commentlist-评论列表)
      - [2.2.2 BlogPost 博客帖子](#222-blogpost-博客帖子)
    - [2.3 使用高阶组件](#23-使用高阶组件)
      - [2.3.1 共同逻辑抽象](#231-共同逻辑抽象)
      - [2.3.2 withSubscription 高阶组件包装函数](#232-withsubscription-高阶组件包装函数)
      - [2.3.3 修改原组件 CommentList, BlogPost](#233-修改原组件-commentlist-blogpost)
    - [2.4 高阶组件小结](#24-高阶组件小结)
      - [2.4.1 高阶组件扩展](#241-高阶组件扩展)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

今天给大家带来的，既不是 React 的 API，也不是新的组件类型；相反的，更多的是一种实现思想，也可以说是组合逻辑，说的直白一点就是一些套路，也可以说是一种应用上的限定领域下的一种设计模式。

**状态提升** 要讨论的是关于多个组件之间如何共享状态；而 **高阶组件(HOC = Higher Order Components)** 则是对于特定实现组件的包装类型，相当为多个存在共同逻辑的组件建立共同的抽象逻辑，进而简化渲染层的展示逻辑。

下面我们就来看看这两个在 React 开发中占有重要地位的实战技巧是如何实现。

P.S. 对 React 基础不熟悉的可以先移步上一篇：[React 入门: 核心特性全面解析](https://blog.csdn.net/weixin_44691608/article/details/117343164)

# 正文

## 1. 状态提升

首先我们先来讨论第一个概念：**状态提升(Lifting State Up)**。

下面我们要演示的代码示例是一个温度转换器，我们的目标是建立一个华氏温度和摄氏温度的实时转换器，修改一边的值另一边也要跟着修改。

下面我们就来看看这个示例和状态提升之间的关联

### 1.1 第一个组件

我们的目标一个温度转换器，第一步我们先写一个输入温度的组件

- `src/lifting/Single.jsx`

```js
import React, { Component } from 'react'

class Single extends Component {
  constructor(props) {
    super(props)
    this.state = {
      temparature: '',
    }
    this.handleTemperatureChange = this.handleTemperatureChange.bind(this)
  }

  handleTemperatureChange(e) {
    this.setState({ temparature: e.target.value })
  }

  render() {
    return (
      <div>
        <label>
          摄氏
          <br />
          <input
            value={this.state.temparature}
            onChange={this.handleTemperatureChange}
          ></input>
        </label>
      </div>
    )
  }
}

export default Single
```

效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_lifting_state_hoc_lifting_sample1.png)

### 1.2 两个实例

下面我们的目标是摄氏和华氏温度的实时转换器，也就是说我们需要第二个华氏温度输入框。这时候我们可以复用一下前面的组件，将它的标签变成由 `props.label` 来指定，从而产生两个不同的输入框

- `src/lifting/Separate.jsx`

```js
import React, { Component } from 'react'

class Single extends Component {
  constructor(props) {
    super(props)
    this.state = {
      temparature: '',
    }
    this.handleTemperatureChange = this.handleTemperatureChange.bind(this)
  }

  handleTemperatureChange(e) {
    this.setState({ temparature: e.target.value })
  }

  render() {
    return (
      <div style={{ display: 'inline-block' }}>
        <label>
          {/* 由 props.label 指定输入框标签 */}
          {this.props.label}
          <br />
          <input
            value={this.state.temparature}
            onChange={this.handleTemperatureChange}
          ></input>
        </label>
      </div>
    )
  }
}

class Separate extends Component {
  render() {
    return (
      <div>
        {/* 创建两个实例来实现两个输入框 */}
        <Single label="摄氏" /> <Single label="华氏" />
      </div>
    )
  }
}

export default Separate
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_lifting_state_hoc_lifting_sample2.png)

看起来已经产生两个输入框了，但是我们注意到这时候两个输入框维持的温度数据是不同步的，这是因为我们透过组件建立的两个实例其实各自拥有自己的状态，如下图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_lifting_state_hoc_separate.png)

然而我们需要的则是一个实时的温度转换器，也就是说两个输入框应该要共享一个温度数据，并进行相应的转换后展示(摄氏/华氏之间的转换)

### 1.3 将实例放到一个组件中共享状态

那我们最直接最简单的想法就是，把两个输入框写到一个组件当中，并共享一个 `state` 就好啦

#### 1.3.1 摄氏 / 华氏温度互相转换

首先我们先利用温度转换公式，定义几个温度转换方法

```js
// 转摄氏温度
function toCelsius(fahrenheit) {
  return ((fahrenheit - 32) * 5) / 9
}

// 转华氏温度
function toFahrenheit(celsius) {
  return (celsius * 9) / 5 + 32
}

// 尝试转换温度
function tryConvert(temperature, convert) {
  const input = parseFloat(temperature)
  if (Number.isNaN(input)) {
    return ''
  }
  const output = convert(input)
  // 对齐到小数后三位
  const rounded = Math.round(output * 1000) / 1000 // fixed 3
  return rounded.toString()
}
```

#### 1.3.2 两个输入框的组件

接下来就是扩展一下我们刚刚定义的输入框组件，变成一个拥有两个依赖于相同状态(`state`)的组件

- `src/lifting/Together.jsx`

```js
const ScaleSymbols = {
  c: Symbol('celsius'),
  f: Symbol('fahrenheit'),
}

class Together extends Component {
  constructor(props) {
    super(props)
    this.state = {
      scale: ScaleSymbols.c,
      temperature: '0',
    }
  }

  handleTemperatureChange(scale, temperature) {
    this.setState({ scale, temperature })
  }

  render() {
    const { scale, temperature } = this.state
    const celsius =
      scale === ScaleSymbols.c
        ? temperature
        : tryConvert(temperature, toCelsius)
    const fahrenheit =
      scale === ScaleSymbols.f
        ? temperature
        : tryConvert(temperature, toFahrenheit)

    return (
      <div>
        <div style={{ display: 'inline-block' }}>
          <label>
            摄氏
            <br />
            <input
              value={celsius}
              onChange={(e) =>
                this.handleTemperatureChange(ScaleSymbols.c, e.target.value)
              }
            ></input>
          </label>
        </div>{' '}
        <FontAwesomeIcon icon={faExchangeAlt} />{' '}
        <div style={{ display: 'inline-block' }}>
          <label>
            华氏
            <br />
            <input
              value={fahrenheit}
              onChange={(e) =>
                this.handleTemperatureChange(ScaleSymbols.f, e.target.value)
              }
            ></input>
          </label>
        </div>
      </div>
    )
  }
}
```

我们将组件状态分为 `scale` 保存当前温度类型、`temparature` 保存当前温度值

后面我们在渲染的时候再使用 `tryConvert` 配合 `toCelsius、toFahrenheit` 计算出指定单位的温度值 `celsius、fahrenheit` 并传入组件的 `value`。

而当用户进行输入的时候则使用 `handleTemperatureChange` 更新组件状态，效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_lifting_state_hoc_lifting_sample3.png)

看起来输入框的温度保持同步了！

### 1.4 状态提升

但是上面的实现未免有些丑陋，前面我们透过将需要相同状态的组件融合成一个组件，所以本质上还是在同组件内的共享状态。

然而实际开发情况并不如下，两个需要共享状态的组件可能都非常复杂，而不得不再分离出各自的组件，这时候我们就不能再使用融合的方式，而是要采用本篇最初提到的 **状态提升** 的方式

#### 1.4.1 共享状态

首先我们先来明确一下前面的例子中我们想要共享的状态，也就是我们融合组件的时候使用的独立状态(`scale+ temparature`)

- `src/lifting/LiftingState.jsx`

```js
const ScaleSymbols = {
  c: Symbol('celsius'),
  f: Symbol('fahrenheit'),
}

class LiftingState extends Component {
  constructor(props) {
    super(props)
    this.state = {
      scale: ScaleSymbols.c,
      temperature: '0',
    }
    this.handleTemperatureChange = this.handleTemperatureChange.bind(this)
  }
```

同时我们也定义一个组件状态更新函数

```js
  handleTemperatureChange(scale, temperature) {
    this.setState({ scale, temperature })
  }
```

#### 1.4.2 叶子组件(状态消费者)

再来就是我们跟第一小节一样为输入框抽象出一个组件，不同的是这次的组件不再维护自己的温度状态，而是从 `props` 获取温度和更新温度的函数

```js
class Single extends Component {
  render() {
    const { label, scale, temperature, handleChange } = this.props
    return (
      <div style={{ display: 'inline-block' }}>
        <label>
          {label}
          <br />
          <input
            value={temperature}
            onChange={(e) => handleChange(scale, e.target.value)}
          ></input>
        </label>
      </div>
    )
  }
}
```

这时候我们就可以在刚才保存状态的类中创建两个输入组件

```js
class LiftingState extends Component {

  // ...

  render() {
    const { scale, temperature } = this.state
    const celsius =
      scale === ScaleSymbols.c
        ? temperature
        : tryConvert(temperature, toCelsius)
    const fahrenheit =
      scale === ScaleSymbols.f
        ? temperature
        : tryConvert(temperature, toFahrenheit)

    return (
      <div>
        <Single
          label="摄氏"
          scale={ScaleSymbols.c}
          temperature={celsius}
          handleChange={this.handleTemperatureChange}
        />{' '}
        <FontAwesomeIcon icon={faExchangeAlt} />{' '}
        <Single
          label="华氏"
          scale={ScaleSymbols.f}
          temperature={fahrenheit}
          handleChange={this.handleTemperatureChange}
        />
      </div>
    )
  }
}
```

效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_lifting_state_hoc_lifting_sample4.png)

#### 1.4.3 状态流图

这时候 `Single` 组件就不再用拥有自己的状态，仅仅只是将 `props` 接收到的温度(`temparature`)展示出来，并在修改温度的时候调用 `props` 传入的更新函数(`handleChange`)，所以实际上是调用了父组件的 `handleTemperatureChange` 更新组件的温度数据 `scale, temparature`，然后 React 会自动的将新的温度透过 `props` 传递并更新子组件的渲染，也就是说状态图如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_lifting_state_hoc_lifting.png)

### 1.5 状态提升小结

以上就是状态提升的示例，最后我们给出状态提升的使用场景和使用要点

- 使用场景

    两个(叶)子节点(`Single`)需要共享同一份状态/数据(`state = scale + temparature`)

- 使用要点

    将共享状态(`state`)提升到最近的共同祖先组件(`LiftingState`)当中，再将状态和状态更新函数(`scale, temperature, handleTemperatureChange`)透过 `props` 从上至下传入(叶)子组件当中

透过 `props` 传入的原理在于 React 会自动在 `props` 数据改变的时候重新渲染子组件，所以对于 `Single` 组件来说就好像状态(`temparature` 温度)改变了一样。

## 2. 高阶组件 HOC

第二种场景则与第一种有些类似又有些不相同的部分，下面我们展示的例子为：

一个渲染评论列表的组件(CommentList)，与一个渲染指定博客帖子的组件(BlogPost)，两个组件共同依赖于相同的数据源(DataSource)

接下来我们就透过这个例子来说明高阶组件的作用

### 2.1 抽象数据源 DataSource

首先我们先明确两个组件共同依赖的抽象数据源 `DataSource`，我们将整个数据源分成三个区块

#### 2.1.1 addChangeListener, removeChangeListener, notify 维护数据观察者队列

第一个部分是维护一个数据观察者队列，记录每个使用数据的对象(保存监听处理函数 `listener`)，并在数据更新的时候进行调用以通知更新

- `src/hoc/store/DataSource.js`

```js
class DataSource {
  listeners = []

  addChangeListener(listener) {
    this.listeners.push(listener)
    console.log(`[DataSource.addChangeListener]listeners`, this.listeners)
  }

  removeChangeListener(listener) {
    const listeners = this.listeners
    if (listeners.includes(listener)) {
      listeners.splice(listeners.indexOf(listener), 1)
      console.log(`[DataSource.removeChangeListener]listeners`, this.listeners)
    }
  }

  // 通知更新
  notify() {
    this.listeners.forEach((listener) => listener())
  }
```

#### 2.1.2 addComment, getComments 评论相关

第二个部分是评论列表组件(`CommentList`)需要使用到的数据(`comments` 评论数据)

```js
  commentId = 0
  comments = []

  clear() {
    this.commentId = 0
    this.comments = []
    this.blogPosts = {}
  }

  addComment(comment) {
    const newComment = {
      ...comment,
      id: this.commentId++,
    }
    this.comments.push(newComment)
    // 数据更新时通知更新
    this.notify()
  }

  getComments() {
    return this.comments
  }
```

#### 2.1.3 setBlogPost, getBlogPost 博客帖子相关

第三部分是博客帖子组件(`BlogPost`)需要用到的博客数据(`blogPost`)

```js
  blogPosts = {}

  setBlogPost(id, post) {
    this.blogPosts[id] = post
    this.notify()
  }

  getBlogPost(id) {
    return this.blogPosts[id]
  }
}
```

#### 2.1.4 填充数据

最后我们定义一个数据源实例，并添加一下实例数据，并设定计时器添加数据，模拟数据更新时组件是否如预期重新渲染

- `src/hoc/store/index.js`

```js
import DataSource from './DataSource'

const store = new DataSource()

init()

export function init() {
  store.clear()
  store.addComment({ title: 'Comment A', content: 'blablabla' })

  store.addComment({ title: 'Comment B', content: 'blablabla' })

  setTimeout(() => {
    store.addComment({ title: 'Comment C', content: 'blablabla' })
  }, 1000)

  store.setBlogPost(1, { title: 'A Blog Post', content: 'blablabla' })

  setTimeout(() => {
    store.setBlogPost(1, { title: 'A Blog Post', content: 'post changed' })
  }, 2000)
}

export default store
```

### 2.2 直接依赖数据源

接下来就是获取这些数据并借由组件渲染成页面，第一个版本我们先简单实现直接依赖数据源的渲染组件

#### 2.2.1 CommentList 评论列表

首先是渲染评论列表

- `src/hoc/version1/CommentList.js`

```js
import React, { Component } from 'react'
import store from '../store'

const Comment = (props) => {
  const {
    comment: { title, content },
  } = props

  const style = { display: 'inline-block', margin: '5px 0' }

  return (
    <div>
      <h3 style={style}>{title}:</h3> {content}
    </div>
  )
}

class CommentList extends Component {
  constructor(props) {
    super(props)
    this.state = {
      comments: store.getComments(),
    }
    this.handleChange = this.handleChange.bind(this)
  }

  componentDidMount() {
    store.addChangeListener(this.handleChange)
  }

  componentWillUnmount() {
    store.removeChangeListener(this.handleChange)
  }

  handleChange() {
    this.setState({
      comments: store.getComments(),
    })
  }

  render() {
    return (
      <div>
        <h2>CommentList 评论列表</h2>
        {this.state.comments.map((comment) => (
          <Comment comment={comment} key={comment.id}></Comment>
        ))}
      </div>
    )
  }
}

export default CommentList
```

我们先在构造函数初始化状态添加评论 `comments`

```js
  constructor(props) {
    super(props)
    this.state = {
      comments: store.getComments(),
    }
    this.handleChange = this.handleChange.bind(this)
  }
```

然后定义一个数据更新时从数据源重新拉取数据更新状态的方法

```js
  handleChange() {
    this.setState({
      comments: store.getComments(),
    })
  }
```

并且在生命周期钩子里面对数据源进行订阅

```js
  componentDidMount() {
    store.addChangeListener(this.handleChange)
  }

  componentWillUnmount() {
    store.removeChangeListener(this.handleChange)
  }
```

最后渲染的时候根据状态内的数据进行渲染

```js
  render() {
    return (
      <div>
        <h2>CommentList 评论列表</h2>
        {this.state.comments.map((comment) => (
          <Comment comment={comment} key={comment.id}></Comment>
        ))}
      </div>
    )
  }
```

最后呈现的结果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_lifting_state_hoc_hoc_sample1.png)

组件首次渲染之后一秒钟评论数据变成三条，而评论列表组件也正确的重新渲染了

#### 2.2.2 BlogPost 博客帖子

第二个博客帖子组件也是一样的套路：

- 定义状态内的数据
- 定义数据更新时更新状态的函数
- 在生命周期钩子订阅数据源
- 渲染部分根据状态内的数据进行渲染

- `src/hoc/version1/BlogPost.js`

```js
import React, { Component } from 'react'
import store from '../store'

const TextBlock = (props) => {
  const { title, content } = props
  return (
    <div>
      <h3>{title}</h3>
      <div>{content}</div>
    </div>
  )
}

class BlogPost extends Component {
  constructor(props) {
    super(props)
    this.state = {
      blogPost: store.getBlogPost(props.id),
    }
    this.handleChange = this.handleChange.bind(this)
  }

  componentDidMount() {
    store.addChangeListener(this.handleChange)
  }

  componentWillUnmount() {
    store.removeChangeListener(this.handleChange)
  }

  handleChange() {
    this.setState({
      blogPost: store.getBlogPost(this.props.id),
    })
  }

  render() {
    return (
      <div>
        <h2>博客帖子</h2>
        <TextBlock {...this.state.blogPost}></TextBlock>
      </div>
    )
  }
}

export default BlogPost
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_lifting_state_hoc_hoc_sample2.png)

同样的我们看到组件渲染的内容也成功替换成数据更新后的内容了

### 2.3 使用高阶组件

前一小结的例子看起来没啥问题了，确实将数据都渲染出来了，而且也成功订阅并响应数据更新后重新渲染出新的结果。但是其实上面的代码出了个问题，两个组件的内容高度相似，在组件内部逻辑出现了高度的重复性：

1. 状态数据

- `CommentList.jsx`

```js
this.state = {
    comments: store.getComments(),
}
```

- `BlogPost.jsx`

```js
this.state = {
    blogPost: store.getBlogPost(props.id),
}
```

2. 数据更新处理

- `CommentList.jsx`

```js
handleChange() {
  this.setState({
    comments: store.getComments(),
  })
}
```

- `BlogPost.jsx`

```js
handleChange() {
  this.setState({
    blogPost: store.getBlogPost(this.props.id),
  })
}
```

3. 订阅数据源

- `CommentList.jsx`

```js
componentDidMount() {
  store.addChangeListener(this.handleChange)
}

componentWillUnmount() {
  store.removeChangeListener(this.handleChange)
}
```

- `BlogPost.jsx`

```js
componentDidMount() {
  store.addChangeListener(this.handleChange)
}

componentWillUnmount() {
  store.removeChangeListener(this.handleChange)
}
```

最后根据数据渲染结果。这时候两个组件的状态关系如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_lifting_state_hoc_direct_listening.png)

#### 2.3.1 共同逻辑抽象

我们可以看到两个组件大部分的内容都在处理数据订阅和状态更新，存在高度的重复性，这时候我们就可以提取出共同的抽象逻辑。跟前面的状态提升的想法也有些类似：**将组件内部的状态 state 提升到某个父组件，并透过 props 接受数据直接渲染** 这样的思想

也就是说我们想要让最底层的组件专心负责结果的展示(直接根据 `props` 渲染就是最好的了)，而将数据/状态的管理交由上级组件来处理，这时候我们就可以将两个渲染组件改成如下：

- `src/hoc/version2/CommentList.js`

```js
const Comment = (props) => {
  const {
    comment: { title, content },
  } = props

  const style = { display: 'inline-block', margin: '5px 0' }

  return (
    <div>
      <h3 style={style}>{title}:</h3> {content}
    </div>
  )
}

class CommentList extends Component {
  constructor(props) {
    super(props)
  }

  render() {
    const { data: comments } = this.props
    return (
      <div>
        <h2>CommentList 评论列表</h2>
        {comments.map((comment) => (
          <Comment comment={comment} key={comment.id}></Comment>
        ))}
      </div>
    )
  }
}
```

- `src/hoc/version2/BlogPost.js`

```js
const TextBlock = (props) => {
  const { title, content } = props
  return (
    <div>
      <h3>{title}</h3>
      <div>{content}</div>
    </div>
  )
}

class BlogPost extends Component {
  constructor(props) {
    super(props)
  }

  render() {
    const blogPost = this.props.data
    return (
      <div>
        <h2>博客帖子</h2>
        <TextBlock {...blogPost}></TextBlock>
      </div>
    )
  }
}
```

我们可以看到这样一来组件的逻辑就很清晰了，根据 `props` 传递下来的 `data` 来渲染渲染视图

#### 2.3.2 withSubscription 高阶组件包装函数

下面一步我们就是要定义两个组件的上级组件，除了为两个组件提供数据之外，还需要负责向数据源订阅数据，也就是说我们希望创造出一种函数(`withSubscription`)能够处理如下的事情

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_lifting_state_hoc_hoc_listening.png)

`withSubscription` 方法将对目标组件进行封装，同时向数据源订阅数据保存在状态中，最后将数据传递给被封装的组件，最后我们来看看高阶组件 `withSubscription` 的最终定义

- `src/hoc/version2/Subscription.js`

```js
import React, { Component } from 'react'
import store from '../store'

export function withSubscription(WrappedComponent, selectData) {
  return class extends Component {
    constructor(props) {
      super(props)
      this.state = {
        data: selectData(store, props),
      }
      this.handleChange = this.handleChange.bind(this)
    }

    componentDidMount() {
      store.addChangeListener(this.handleChange)
    }

    componentWillUnmount() {
      store.removeChangeListener(this.handleChange)
    }

    handleChange() {
      this.setState({
        data: selectData(store, this.props),
      })
    }

    render() {
      return (
        <WrappedComponent
          {...this.props}
          data={this.state.data}
        ></WrappedComponent>
      )
    }
  }
}
```

`withSubscription` 提供了这样的一个函数标签

```ts
{
  withSubscription: (Component, selectData) => Component
  selectData: (datasource, props) => data
}
```

我们就可以看到高阶组件内部定义了另一个上级组件，负责刚才的重复逻辑部分

1. 保存数据(状态)

```js
this.state = {
  data: selectData(store, props),
}
```

2. 订阅数据

```js
componentDidMount() {
    store.addChangeListener(this.handleChange)
}

componentWillUnmount() {
    store.removeChangeListener(this.handleChange)
}

handleChange() {
    this.setState({
        data: selectData(store, this.props),
    })
}
```

3. 将数据透过 `data` 属性传递给内部组件

```js
render() {
  return (
    <WrappedComponent
      {...this.props}
      data={this.state.data}
    ></WrappedComponent>
  )
}
```

与刚才不同的是我们将两个组件从数据源获取数据的方式抽象成一个方法 `selectData` 并透过参数传入，这时候我们的高阶组件就能够忽略组件是如何从数据源获取数据，而是依赖于一个抽象的契约：**指定 selectData 获取数据逻辑** 的方式，来将订阅数据和更新的逻辑整个抽象成一个新的上级组件，也就是 `withSubscription` 返回的高级组件

#### 2.3.3 修改原组件 CommentList, BlogPost

最后我们再使用 `withSubscription` 方法包装一下前面修改过的"单纯渲染"

- `src/hoc/version2/CommentList.js`

```js
const SubscribedCommentList = withSubscription(CommentList, (store) =>
  store.getComments()
)

export default SubscribedCommentList
```

- `src/hoc/version2/BlogPost.js`

```js
const SubscribedBlogPost = withSubscription(BlogPost, (store, props) =>
  store.getBlogPost(props.id)
)

export default SubscribedBlogPost
```

我们可以看到透过 `withSubscription` 的封装，我们只需要提供一个根据数据源获取数据的逻辑 `selectData`，`withSubscription` 高阶组件就会自动为我们订阅数据，并将数据透过 `props.data` 传递进来

这时候两个组件的状态关系就变成如下图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_lifting_state_hoc_hoc_listening2.png)

最终得效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_lifting_state_hoc_hoc_sample3.png)

### 2.4 高阶组件小结

最后我们总结一下高阶组件的使用场景和使用要点，与状态提升的应用场景还是有些不同的

- 使用场景

    多个组件之间存在重复的抽象逻辑

- 使用要点

    透过定义某个函数标签为 `(Component, ...) => Component` 的高阶组件函数，他的职责是为被包装的组件进行一定程度上的逻辑封装，再返回包装过的高级组件；这时候被包装的内部组件则可以透过 `props` 来获取高阶组件向内部组件注入的数据、行为

#### 2.4.1 高阶组件扩展

最后再多说说几个，我们可以再对 `withSubscription` 进行一些修改，我们可以再多接受一个参数来指定传入内部组件的数据名称

```js
export function withSubscription(WrappedComponent, propName, selectData) {

    // ...

    render() {
      const data = { [propName]: this.state.data }
      return (
        <WrappedComponent
          {...this.props}
          {...data}
        ></WrappedComponent>
      )
    }
```

```js
const SubscribedCommentList = withSubscription(
    CommentList,
    'comments',
    (store) => store.getComments()
)
```

或是我们可以将数据源的绑定柯里化

```js
export const withSubscription = (store) => (WrappedComponent, selectData) => {/* ... */}
```

```js
const SubscribedCommentList = withSubscription(store)(
    CommentList,
    (store) => store.getComments()
)
```

其实我们前面在[另一篇](https://blog.csdn.net/weixin_44691608/article/details/116363154)提过的，在 react-redux 中提供的 `connect` 函数就是这样一个高阶组件的实现，其函数签名如下

```ts
{
    connect: (mapStateToProps, mapDispatchToProps)(Component)
}
```

先绑定两个函数：状态到 `props` 的映射、行为到 `props` 的映射，最后绑定要包装的组件。也就是说其实这时候我们是可以复用这个先绑定好状态映射行为的函数如下

```js
const connectedMapping = connect(mapStateToProps, mapDispatchToProps)

const wrappedComponentA = connectedMapping(ComponentA)
const wrappedComponentB = connectedMapping(ComponentB)
```

# 结语

本篇给大家介绍 React 中最常见也是非常重要的开发模式(其实就是套路)，其实 **状态提升、高阶组件** 两个方法的概念都不难，难点在于如何对现有组件进行提炼，识别共享状态和数据流、抽取重复逻辑的任务才是比较复杂的部分。同时也是帮助开发专注在处理业务逻辑以及对重复业务逻辑进行重新的思考和组合，供大家参考。

# 其他资源

## 参考连接

| Title                 | Link                                                                                                                           |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| React 官方 - 状态提升 | [https://react.docschina.org/docs/lifting-state-up.html](https://react.docschina.org/docs/lifting-state-up.html)               |
| React 官方 - 高阶组件 | [https://react.docschina.org/docs/higher-order-components.html](https://react.docschina.org/docs/higher-order-components.html) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_lifting_state_hoc](https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_lifting_state_hoc)
