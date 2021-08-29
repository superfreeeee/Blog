# React Redux 进阶: Hooks 版本用法 & Custom Context 局部 Store 实践

@[TOC](文章目录)

<!-- TOC -->

- [React Redux 进阶: Hooks 版本用法 & Custom Context 局部 Store 实践](#react-redux-进阶-hooks-版本用法--custom-context-局部-store-实践)
- [前言](#前言)
- [正文](#正文)
  - [1. 在函数组件内消费 Redux 数据](#1-在函数组件内消费-redux-数据)
    - [1.1 Redux 全局状态定义](#11-redux-全局状态定义)
    - [1.2 使用 Hooks 消费 Redux 数据](#12-使用-hooks-消费-redux-数据)
  - [2. Custom Context 实现局部 store](#2-custom-context-实现局部-store)
    - [2.1 HeaderStore 局部状态定义](#21-headerstore-局部状态定义)
    - [2.2 消费局部 store](#22-消费局部-store)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

我们在 React 框架下使用 Redux 的时候通常就是简单的使用 react-redux 提供的全局 api，并且在整个项目的最高层组件外包裹一层 Provider 来维护一个全局 Store。

本篇将要带大家来学习如何在函数组件内使用 redux 数据，第二部分则是带大家认识其实 redux 是提供了我们局部 Store 的结局方案的！

# 正文

## 1. 在函数组件内消费 Redux 数据

第一部分我们照旧先使用全局的 store 对象，不过我们不再使用 connect 函数来将 Redux 数据映射到 props 了，而是用更适合于函数组件的 hooks api

### 1.1 Redux 全局状态定义

- `/src/store.ts`

当然第一步我们先定义好 store 相关的东西

首先是全局 state

```ts
// State
export interface IAppState {
  title: string;
}

const initAppState: IAppState = {
  title: '',
};
```

然后定义以下 actions

```ts
// Actions
enum EAppActionType {
  UpdateTitle,
}

type AppAction = { type: EAppActionType.UpdateTitle; payload: string };

export const updateTitleCreator: ActionCreator<AppAction> = (
  title: string
) => ({
  type: EAppActionType.UpdateTitle,
  payload: title,
});
```

接下来给出 reducer 来处理更新

```ts
// Reducer
const globalReducer = (
  prevState: IAppState = initAppState,
  action: AppAction
) => {
  switch (action.type) {
    case EAppActionType.UpdateTitle:
      return { ...prevState, title: action.payload };
    default:
      return prevState;
  }
};
```

最后使用 globalReducer 创建 store 对象

```ts
// Store
export const store = createStore(globalReducer);
```

另外我们还封装一个钩子使 ActionCreator 更方便使用

```ts
// hooks
export const useActions = (actions: ActionCreator<AppAction>) => {
  const dispatch = useDispatch();

  const boundActions = bindActionCreators(actions, dispatch);

  return boundActions;
};
```

### 1.2 使用 Hooks 消费 Redux 数据

定义好 Redux 数据之后，我们照旧在全局提供一个 Provider

- `/src/index.tsx`

```ts
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

import { Provider } from 'react-redux';
import { store } from './store';

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.querySelector('#app')
);
```

接下来我们就可以在 App 组件里面使用 useStore、useSelector、useDispatch 等方法消费 Redux 数据了

- `/src/App.tsx`

```ts
import styles from './index.module.scss';

import React, { useEffect } from 'react';
import classNames from 'classnames';

import useDocumentTitle from '@hooks/useDocumentTitle';
import Header from '@layouts/Header';
import { useSelector } from 'react-redux';
import { IAppState, updateTitleCreator, useActions } from './store';

const useTitle = () => {
  const title = useSelector((state: IAppState) => state.title);

  const updateTitle = useActions(updateTitleCreator);

  useEffect(() => {
    updateTitle('React Redux - Hooks');
  }, []);

  useDocumentTitle(title);
};

const App: React.FC<{}> = () => {
  useTitle();

  return (
    <div className={classNames(styles.app)}>
      <Header />
    </div>
  );
};

export default App;
```

useSelector 就是 useStore().getState() 的组合技，而 useDispatch 则是 hooks 版用于获取 store.dispatch。

如下我们就能看到 document.title 从默认的空串变成我们自定义的字符串了

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_redux_hooks_1_hooks.png)

## 2. Custom Context 实现局部 store

第二步实际上是使用 react-redux 非常重要的一个概念，我们不能总是将状态放到全局对象上，相反的我们应该总是将所谓的 store 压制在最小许可范围内，也才能更好的控制 store 中数据的生命周期

### 2.1 HeaderStore 局部状态定义

一样第一步我们要先定义 redux 中 store 相关配置

- `/src/layouts/Header/store.ts`

首先有 state 状态

```ts
// State 类型
interface IUserInfo {
  name: string;
}

export interface IHeaderState {
  title: string;
  userInfo: IUserInfo;
}

const initHeaderState: IHeaderState = {
  title: '',
  userInfo: {
    name: '',
  },
};
```

Acitons 类型

```ts
// Action 类型
enum EHeaderActionType {
  ResetAll,
  UpdateTitle,
  UpdateUserInfo,
}

type HeaderAction =
  | { type: EHeaderActionType.ResetAll }
  | { type: EHeaderActionType.UpdateTitle; payload: string }
  | { type: EHeaderActionType.UpdateUserInfo; payload: IUserInfo };
```

还有与 Actions 匹配的 ActionCreator

```ts
// ActionCreators
export const resetCreator: ActionCreator<HeaderAction> = () => ({
  type: EHeaderActionType.ResetAll,
});

export const updateTitleCreator: ActionCreator<HeaderAction> = (
  title: string
) => ({
  type: EHeaderActionType.UpdateTitle,
  payload: title,
});

export const updateUserInfoCreator: ActionCreator<HeaderAction> = (
  userInfo: IUserInfo
) => ({
  type: EHeaderActionType.UpdateUserInfo,
  payload: { ...userInfo },
});
```

以及 reducer

```ts
// Reducer
const headerReducer = (
  prevState: IHeaderState = initHeaderState,
  action: HeaderAction
) => {
  switch (action.type) {
    case EHeaderActionType.ResetAll:
      return initHeaderState;
    case EHeaderActionType.UpdateTitle:
      const title = action.payload;
      return { ...prevState, title };
    case EHeaderActionType.UpdateUserInfo:
      const userInfo = action.payload;
      return { ...prevState, userInfo };
    default:
      return prevState;
  }
};
```

最后是 store 与 hooks，与前面不同的是这时候我们需要提供一个自己的 Context 而不是默认的全局 Context，也就是说 useSelector、useDispatch 也是需要绑定到我们自己的 Context 上才能实现与全局 Context 区隔

```ts
// store
export const store = createStore(headerReducer);

// context
export const headerContext = React.createContext(null);

// hooks
export const useHeaderStore = createStoreHook(headerContext);
export const useHeaderSelector = createSelectorHook(headerContext);
export const useHeaderDispatch = createDispatchHook(headerContext);

export const useHeaderActions = (
  actions: ActionCreator<HeaderAction>,
  deps: DependencyList = []
) => {
  const dispatch = useHeaderDispatch();

  const boundActions = useMemo(() => {
    return bindActionCreators(actions, dispatch);
  }, [dispatch, ...deps]);

  return boundActions;
};
```

createStoreHook、createSelectorHook、createDispatchHook 都是用来绑定 context 后返回对应的 useStore、useSelector、useDispatch 钩子

### 2.2 消费局部 store

定义好了之后我们就可以到组件内消费了，首先是局部的根组件

- `/src/layouts/Header/index.tsx`

```ts
import styles from './index.module.scss';

import React from 'react';
import { Provider } from 'react-redux';

import { headerContext, store } from './store';
import Title from './Title';
import Avator from './Avator';
import MoreActions from './MoreActions';

const Header = () => {
  return (
    <Provider context={headerContext} store={store}>
      <div className={styles.header}>
        <Title />
        <div className={styles.info}>
          <Avator />
          <MoreActions />
        </div>
      </div>
    </Provider>
  );
};

export default Header;
```

在 Header 上注入自定义 context 的 Provider 之后，就可以在 Title、Avator、MoreActinos 中使用特制的钩子，与全局提供的 useSelector、useDispatch 没什么区别

- `/src/layouts/Header/Title/index.tsx`

```ts
import styles from './index.module.scss';

import React, { useEffect } from 'react';
import {
  IHeaderState,
  updateTitleCreator,
  useHeaderActions,
  useHeaderSelector,
} from '../store';

const Title = () => {
  const title = useHeaderSelector((state: IHeaderState) => state.title);

  const updateTitle = useHeaderActions(updateTitleCreator);

  useEffect(() => {
    const title = 'React Redux - Hooks';
    let n = 0;
    for (let i = 0; i < title.length; i++) {
      const nextChar = title.charAt(i);
      if (nextChar === ' ') {
        continue;
      }
      setTimeout(() => {
        updateTitle(title.substring(0, i + 1));
      }, n * 500);
      n++;
    }
  }, []);

  return <h1 className={styles.title}>{title}</h1>;
};

export default Title;
```

- `/src/layouts/Header/Avator/index.tsx`

```ts
import styles from './index.module.scss';

import React, { useEffect } from 'react';
import {
  IHeaderState,
  updateUserInfoCreator,
  useHeaderActions,
  useHeaderSelector,
} from '../store';

const Avator = () => {
  const userInfo = useHeaderSelector((state: IHeaderState) => state.userInfo);

  const updateUserInfo = useHeaderActions(updateUserInfoCreator);

  useEffect(() => {
    setTimeout(() => {
      updateUserInfo({ name: '超悠閒' });
    }, 2500);
  }, []);

  return (
    <div className={styles.avator}>
      <h3>{userInfo.name}</h3>
    </div>
  );
};

export default Avator;
```

- `/src/layouts/Header/MoreActions/index.tsx`

```ts
import styles from './index.module.scss';

import React, { useCallback } from 'react';
import {
  resetCreator,
  updateTitleCreator,
  updateUserInfoCreator,
  useHeaderActions,
} from '../store';

const MoreActions = () => {
  const updateTitle = useHeaderActions(updateTitleCreator);
  const updateUserInfo = useHeaderActions(updateUserInfoCreator);
  const resetHeader = useHeaderActions(resetCreator);

  const retry = useCallback(() => {
    resetHeader();

    const title = 'React Redux - Hooks';
    for (let i = 1; i < title.length; i++) {
      setTimeout(() => {
        updateTitle(title.substring(0, i));
      }, i * 500);
    }

    setTimeout(() => {
      updateUserInfo({ name: '超悠閒' });
    }, 2500);
  }, []);

  return (
    <div className={styles.more}>
      <button onClick={retry}>重试</button>
    </div>
  );
};

export default MoreActions;
```

最终结果如下，实际上自己把代码拉下来可以看到还做了一个小小的打字机动画

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_redux_hooks_2_custom_context.png)

# 结语

本篇其实就是介绍几个 react-redux 的进阶钩子，本质上用法都不是太难，非常重要的一点就是可以多使用 custom context 的特性区分多个 store 实例，避免将状态放置在全局

# 其他资源

## 参考连接

| Title                                                                        | Link                                                                                                                               |
| ---------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| React Redux - Hooks                                                          | [https://react-redux.js.org/api/hooks](https://react-redux.js.org/api/hooks)                                                       |
| React 项目启动2：使用 webpack 手动创建 React 项目(附加 React Router + Redux) | [https://blog.csdn.net/weixin_44691608/article/details/116363154](https://blog.csdn.net/weixin_44691608/article/details/116363154) |
| React 高阶指引: Context 上下文 & 组件组合 & Render Props                     | [https://blog.csdn.net/weixin_44691608/article/details/117458645](https://blog.csdn.net/weixin_44691608/article/details/117458645) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_redux_hooks](https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_redux_hooks)
