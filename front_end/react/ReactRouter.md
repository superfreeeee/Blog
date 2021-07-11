# React 路由: react-router-dom 前端路由 + connected-react-router 与 redux 连用

@[TOC](文章目录)

<!-- TOC -->

- [React 路由: react-router-dom 前端路由 + connected-react-router 与 redux 连用](#react-路由-react-router-dom-前端路由--connected-react-router-与-redux-连用)
- [前言](#前言)
- [正文](#正文)
  - [1. 基础使用](#1-基础使用)
    - [1.0 安装依赖](#10-安装依赖)
    - [1.1 路由定义](#11-路由定义)
    - [1.2 匹配类组件](#12-匹配类组件)
    - [1.3 匹配函数组件](#13-匹配函数组件)
      - [1.3.1 非直接相关函数组件](#131-非直接相关函数组件)
      - [1.3.2 非直接相关类组件](#132-非直接相关类组件)
  - [2. 与 redux 连用](#2-与-redux-连用)
    - [2.0 安装依赖](#20-安装依赖)
    - [2.1 configureStore 配置全局状态](#21-configurestore-配置全局状态)
    - [2.2 ConnectedRouter 绑定 redux 状态](#22-connectedrouter-绑定-redux-状态)
      - [2.2.1 类组件提取状态](#221-类组件提取状态)
      - [2.2.2 函数组件提取状态](#222-函数组件提取状态)
    - [2.3 提交 action 进行路由跳转](#23-提交-action-进行路由跳转)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

本篇将要带来 react-router-dom 的 React 路由库的使用

可能有些人会问 react-router 跟 react-router-dom 到底哪里不一样，实际上我们可以说 react-router-dom 是更为基础的库，而 react-router 则是在此之上再进行一层封装。

本篇则是全程使用 react-router-dom 来实现基于 React 的路由方案，同时后面介绍 connected-react-router 实现路由绑定到 redux 全局状态当中

# 正文

## 1. 基础使用

首先第一部分我们先来看看 React 中使用路由的方式

### 1.0 安装依赖

本篇使用 react-router-dom 为基础实现前端路由，安装一下依赖

```bash
$ yarn add react-router-dom
```

### 1.1 路由定义

前端路由实际上就是一些组件的抽换，所以路由的定义其实就是直接写在 jsx 中的组件

- `/src/basic/App.tsx`

```ts
export default function App() {
  return (
    <div>
      <h1>React Router</h1>
      <Router>
        <ul>
          <li><Link to="/">Default</Link></li>
          <li><Link to="/123">Default 123</Link></li>
          <li><Link to="/home">Home</Link></li>
          <li><Link to="/home/456">Home 456</Link></li>
        </ul>
        <Switch>
          <Route path="/"            exact={true} component={Default}></Route>
          <Route path="/home"        exact={true} component={Home}></Route>
          <Route path="/:userId"      exact={true} component={Default}></Route>
          <Route path="/home/:userId" exact={true} component={Home}></Route>
        </Switch>
      </Router>
    </div>
  )
}
```

- `Router` 组件表示需要路由部分的根组件
- `Switch` 组件表示只会渲染第一个匹配的子路由
- `Route` 组件表示可能被抽换的路径
  - `path` 属性表示匹配的规则
  - `exact` 表示必须精确匹配路径
  - `component` 表示该路径代表的组件

所以我们可以看到，对于路由来说是不管路由中的组件是怎么定义的，所以在传递路由参数的时候可能根据不同的组件定义形式有不同的用法

### 1.2 匹配类组件

对于作为路径之一的类组件，我们可以直接透过 `props` 属性来获得路径匹配的相关信息

- `/src/basic/Default.tsx`

```ts
import React, { Component } from 'react'
import { RouteComponentProps } from 'react-router-dom'
import { RouteUserParam } from './App'
import { group } from '../utils/msg'

class Default extends Component<RouteComponentProps<RouteUserParam>> {
  constructor(props) {
    super(props)
  }

  componentDidMount() {
    const match = this.props.match

    group('[Component] Default componentDidMount', () => {
      console.log('props', this.props)
      console.log('match', match)
    })
  }

  componentDidUpdate() {
    const match = this.props.match

    group('[Component] Default componentDidUpdate', () => {
      console.log('props', this.props)
      console.log('match', match)
    })
  }

  render() {
    const userId = this.props.match.params.userId
    return (
      <div>
        <h2>Default Page</h2>
        {userId && <h3>userId: {userId}</h3>}
      </div>
    )
  }
}

export default Default
```

默认路径 `/` 下的输出

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_router_basic_default1.png)

带参数 `/123` 的输出

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_router_basic_default2.png)

我们可以看到从 props 传递下来的属性中主要有以下几个

- `history` 历史记录 & 路由操作相关
- `location` 当前路径相关
- `match` 路径匹配相关信息

### 1.3 匹配函数组件

而在函数组件内也可以透过 props 来传递内容，也可以透过各种 Hook API 来获取相关指定信息

- `/src/basic/Home.tsx`

```tsx
import React from 'react'
import {
  useHistory,
  useLocation,
  useParams,
  useRouteMatch,
} from 'react-router-dom'
import { RouteUserParam } from './App'
import { group } from '../utils/msg'
import InnerHome from './InnerHome'

const Home = (props) => {
  const history = useHistory()
  const location = useLocation()
  const params = useParams<RouteUserParam>()
  const match = useRouteMatch()

  const userId = params.userId

  group('[Component] Home', () => {
    console.log('props', props)
    console.log('history', history)
    console.log('location', location)
    console.log('params', params)
    console.log('match', match)
  })

  return (
    <div>
      <h2>Home Page</h2>
      {userId && <h3>userId: {userId}</h3>}
      <InnerHome></InnerHome>
    </div>
  )
}

export default Home
```

`/home` 路径下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_router_basic_home1.png)

`/home/456` 路径下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_router_basic_home2.png)

你可能会有疑问是既然能从 props 那干嘛还要用 hook，这是因为透过 props 传递的信息仅限于作为 `<Route component={xxx} />` 标记的组件，而在其之下的子组件就无法直接透过 props 获取信息，这时候就可以用 hook 直接获取如下

#### 1.3.1 非直接相关函数组件

- `/src/basic/InnerHome.tsx`

使用 Hook 就可以直接在组件内引用到 history、location、match 等对象而不一定要是直接写成 `<Route component={xxx}/>`

```ts
import React from 'react'
import {
  useHistory,
  useLocation,
  useParams,
  useRouteMatch,
} from 'react-router-dom'
import { group } from '../utils/msg'
import { RouteUserParam } from './App'

const InnerHome = (props) => {
  const history = useHistory()
  const location = useLocation()
  const params = useParams<RouteUserParam>()
  const match = useRouteMatch()

  group('[Component] InnerHome', () => {
    console.log('props', props)
    console.log('history', history)
    console.log('location', location)
    console.log('params', params)
    console.log('match', match)
  })
  return <div></div>
}

export default InnerHome
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_router_basic_home3.png)

#### 1.3.2 非直接相关类组件

诶这时候又回头发现，类组件是不是也会出现类似的非直接作为路径的组件而需要用到 history、location、match 等对象，这时候就可以用 `withRouter` 高阶组件来实现

- `/src/basic/InnerDefault.tsx`

```ts
import React, { Component } from 'react'
import { RouteComponentProps, withRouter } from 'react-router-dom'
import { RouteUserParam } from './App'
import { group } from '../utils/msg'

class InnerDefault extends Component<
  RouteComponentProps<RouteUserParam>
> {
  constructor(props) {
    super(props)
  }

  componentDidMount() {
    const match = this.props.match

    group('[Component] InnerDefault componentDidMount', () => {
      console.log('props', this.props)
      console.log('match', match)
    })
  }

  componentDidUpdate() {
    const match = this.props.match

    group('[Component] InnerDefault componentDidUpdate', () => {
      console.log('props', this.props)
      console.log('match', match)
    })
  }

  render() {
    return <div></div>
  }
}

export default withRouter(InnerDefault)
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_router_basic_default3.png)

## 2. 与 redux 连用

第二部分是与 redux 连用的场景

很多时候对于单页面来说前端路由的部分几乎覆盖了整个页面，同时如果当我们还有用到 redux 的话，就会变成套了两层壳，一下要 `connect` 进行高阶组件化、一下要 `withRouter` 进行路由化，然而本质上他们的作用空间是一样的，所以我们就有这么一个设想

> 将路由的部分作为 redux 的其中一个全局状态，并将路由跳转等操作变成提交 dispatch 的方式展现

如此一来不仅实现了全局路由状态的共享，同时也将路由相关操作与 redux 的 dispatch(action) 靠拢

### 2.0 安装依赖

接下来我们将使用 `connected-react-router` 这个库来实现 router 与 redux 连用

```bash
$ yarn add connected-react-router
$ yarn add history@^4  # history 5 不兼容
```

### 2.1 configureStore 配置全局状态

要与 redux 连用首先要有 redux 的部分，也就是创建 store

- `/src/store/configureStore.tsx`

```ts
import {
  connectRouter,
  routerMiddleware,
} from 'connected-react-router'
import { History } from 'history'
import {
  applyMiddleware,
  combineReducers,
  compose,
  createStore,
} from 'redux'

export default function configureStore(history: History) {
  return createStore(
    combineReducers({
      router: connectRouter(history),
    }),
    compose(applyMiddleware(routerMiddleware(history)))
  )
}
```

- `connectRouter` 创建 router 专用的 reducer，名字必须取名为 router
- `routerMiddleware(history)` 创建路由专用中间件，是的我们可以使用 dispatch(action) 的形式来进行路由跳转

### 2.2 ConnectedRouter 绑定 redux 状态

而在组件使用的时候则是以 `ConnectedRouter` 组件来替换原本的 `Router` 组件

- `/src/connected/App.tsx`

```ts
import { ConnectedRouter } from 'connected-react-router'
import React from 'react'
import { Provider } from 'react-redux'
import { Link, Route, Switch } from 'react-router-dom'
import Default from './Default'
import Home from './Home'
import configureStore from '../store/configureStore'
import Info from './Info'
import { createBrowserHistory } from 'history'

const history = createBrowserHistory()

const store = configureStore(history)

const App = () => {
  return (
    <Provider store={store}>
      <ConnectedRouter history={history}>
        <>
          <h1>React Router with redux</h1>
          <ul>
            <li><Link to="/">Default</Link></li>
            <li><Link to="/123">Default 123</Link></li>
            <li><Link to="/home">Home</Link></li>
            <li><Link to="/home/456">Home 456</Link></li>
          </ul>
          <Info></Info>
          <Switch>
            <Route path="/"            exact={true} component={Default}></Route>
            <Route path="/home"        exact={true} component={Home}></Route>
            <Route path="/:userId"      exact={true} component={Default}></Route>
            <Route path="/home/:userId" exact={true} component={Home}></Route>
          </Switch>
        </>
      </ConnectedRouter>
    </Provider>
  )
}

export default App
```

这里我们使用 history 库来创建作为底层依赖的 history 对象

#### 2.2.1 类组件提取状态

对于类组件我们就可以直接跟使用 redux 一样，直接将路由相关的数据从 `state.router` 这一个 reducer 中提取出来

- `/src/connected/Default.tsx`

```ts
import { RouterState } from 'connected-react-router'
import React, { Component } from 'react'
import { connect, ReactReduxContext } from 'react-redux'
import { RouteComponentProps } from 'react-router-dom'
import { RouteUserParam } from '../basic/App'
import { group } from '../utils/msg'

type DefaultProps = RouteComponentProps<RouteUserParam> & {
  router: RouterState
}

class Default extends Component<DefaultProps> {
  constructor(props) {
    super(props)
  }

  componentDidMount() {
    group('[Component] Default componentDidMount', () => {
      console.log('props', this.props)
      console.log('router', this.props.router)
    })
  }

  componentDidUpdate() {
    group('[Component] Default componentDidUpdate', () => {
      console.log('props', this.props)
      console.log('router', this.props.router)
    })
  }

  render() {
    const userId = this.props.match.params.userId
    return (
      <div>
        <h2>Default Page</h2>
        {userId && <h3>userId: {userId}</h3>}
      </div>
    )
  }
}

const mapStateToProps = (state) => {
  return { ...state }
}

export default connect(mapStateToProps)(Default)
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_router_connected_default.png)

#### 2.2.2 函数组件提取状态

而对于函数组件，其实 `useXxx` 已经挺方便的了，不过我们也还是可以使用 `useSelector` 从 redux 中来提取 router

- `/src/connected/Home.tsx`

```ts
const Home = () => {
  const { history, location, params, match } = useRoute()

  const router = useSelector((state: RouterRootState) => state.router)

  const userId = params.userId

  group('[Component] Home', () => {
    console.log('history', history)
    console.log('location', location)
    console.log('params', params)
    console.log('match', match)
    console.log('router', router)
  })

  return (
    <div>
      <h2>Home Page</h2>
      {userId && <h3>userId: {userId}</h3>}
    </div>
  )
}

export default Home
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_router_connected_home.png)

### 2.3 提交 action 进行路由跳转

最后一个与 redux 连用的好处就是可以将路由跳转变成 action 的提交如下

- `/src/connected/Info.tsx`

```ts
import { push, RouterRootState } from 'connected-react-router'
import React, { useEffect, useState } from 'react'
import { useDispatch, useSelector } from 'react-redux'
import { useHistory } from 'react-router-dom'
import { group } from '../utils/msg'

const Info = () => {
  const router = useSelector((state: RouterRootState) => state.router)
  const dispatch = useDispatch()
  const [input, handleChange] = useInput()

  const goto = () =>
    dispatch(push(`/${input}`, { msg: 'message from Info' }))

  const history = useHistory()

  useEffect(() => {
    group('[Info] updated', () => {
      console.log('router', router)
      console.log('history', history)
    })
  })

  return (
    <div>
      <h3>router path: {router.location.pathname}</h3>
      <label>
        path:{' '}
        <input type="text" value={input} onChange={handleChange} />
      </label>
      <button onClick={goto}>back</button>
    </div>
  )
}

export default Info
```

核心在于

```ts
  const goto = () =>
    dispatch(push(`/${input}`, { msg: 'message from Info' }))
```

由于我们前面已经添加过路由相关的中间件了，所以我们这边就可以直接使用 `push` 方法，传入必要的参数它就会自动生成对应的 action 了

# 结语

本篇介绍在 React 中使用 router 的方式，基础版本的以及与 redux 连用的版本，算是平常开发中比较基础的技术，供大家参考

# 其他资源

## 参考连接

| Title                                                           | Link                                                                                                                                                                       |
| --------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ReactTraining/history - Github                                  | [https://github.com/ReactTraining/history](https://github.com/ReactTraining/history)                                                                                       |
| React Router - Quick Start                                      | [https://reactrouter.com/web/guides/quick-start](https://reactrouter.com/web/guides/quick-start)                                                                           |
| React Router 中文文档                                           | [https://react-guide.github.io/react-router-cn/](https://react-guide.github.io/react-router-cn/)                                                                           |
| 利用 typescript 写 react-router 5                               | [https://segmentfault.com/a/1190000021001976](https://segmentfault.com/a/1190000021001976)                                                                                 |
| \[筆記\]\[React\]React網頁好朋友Router(2)-掌握match是成功的一半 | [https://ithelp.ithome.com.tw/articles/10204451](https://ithelp.ithome.com.tw/articles/10204451)                                                                           |
| react-router二级页面刷新后显示404问题                           | [https://blog.csdn.net/HZZOU/article/details/102886863](https://blog.csdn.net/HZZOU/article/details/102886863)                                                             |
| webpack - devServer.historyApiFallback                          | [https://webpack.docschina.org/configuration/dev-server/#devserverhistoryapifallback](https://webpack.docschina.org/configuration/dev-server/#devserverhistoryapifallback) |
| supasate/connected-react-router - Github                        | [https://github.com/supasate/connected-react-router](https://github.com/supasate/connected-react-router)                                                                   |
| 使用connected-react-router使router与store同步                   | [https://www.cnblogs.com/tianyamoon/p/12298201.html](https://www.cnblogs.com/tianyamoon/p/12298201.html)                                                                   |
| 【React】Could not find router reducer in state tree解决方法    | [https://blog.csdn.net/yehuozhili/article/details/107193780](https://blog.csdn.net/yehuozhili/article/details/107193780)                                                   |
| 11、react withRouter的原理与使用                                | [https://www.cnblogs.com/gopark/p/12328001.html](https://www.cnblogs.com/gopark/p/12328001.html)                                                                           |
| Connected-React-Router库源码阅读笔记                            | [https://www.yuque.com/dengnan/qttex2/kmf4pr](https://www.yuque.com/dengnan/qttex2/kmf4pr)                                                                                 |
| connected-react-router                                          | [https://www.jianshu.com/p/8ee6ef097aac](https://www.jianshu.com/p/8ee6ef097aac)                                                                                           |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_router](https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_router)
