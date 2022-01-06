# React 代码拆分: 从 react-loadable 到 Suspense + lazy 组合

@[TOC](文章目录)

<!-- TOC -->

- [React 代码拆分: 从 react-loadable 到 Suspense + lazy 组合](#react-代码拆分-从-react-loadable-到-suspense--lazy-组合)
- [代码拆分（懒加载）](#代码拆分懒加载)
- [1. react-loadable 实现](#1-react-loadable-实现)
- [2. 简单实现](#2-简单实现)
- [3. 使用 Suspense + lazy 实现](#3-使用-suspense--lazy-实现)
- [4. 闪烁问题](#4-闪烁问题)
- [参考连接](#参考连接)
- [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 代码拆分（懒加载）

对于一些人可能会把代码拆分与模块化搞混（对说的就是作者自己），但是我们必须明白一件事情：通常我们使用 React + Webpack 的组合技的时候，实际上我们生产出来的最终代码是没有进行拆分的，充其量透过部分配置分成了几个文件，但是针对部分路由的懒加载行为实际上是需要单独配置的。

本篇介绍 react-loadable 库的使用，并以其为样本尝试实现一个类似效果的高阶组件来更深刻的理解它背后的运行原理

# 1. react-loadable 实现

下面我们将统一使用类似形式的 Sample 组件

- `/src/components/SampleX.tsx`

```ts
import React from 'react';

console.log(`first load Sample1.tsx`);

const Sample1 = () => {
  return (
    <div>
      <h1>Sample1</h1>
    </div>
  );
};

export default Sample1;
```

我们在组件之外加一句输出，表明组件脚本的加载时机（而非组件渲染的时机）

- `/src/components/index.tsx`

第一种我们直接先来看使用 react-loadable 库的效果

```ts
const Loading = (props) => {
  const { pastDelay } = props;
  const shouldRender = typeof pastDelay !== 'boolean' || pastDelay;
  console.log(`render Loading(shouldRender=${shouldRender})`, props);
  return shouldRender && <div>Loading...</div>;
};
```

上面我们先写一个加载中的 Loading 组件，下面才是 react-loadable 的使用方法核心

```ts
import Loadable from 'react-loadable';

export const Sample2 = Loadable({
  loader: () => import('./Sample2'),
  loading: Loading,
  delay: 30000,
});
```

`loader` 传入一个加载组件的简单函数，`loading` 则是传入加载过程要渲染的过渡组件（如骨架屏）

最终的实现效果如下：

首先一开始我们的首页是一个 Sample1 组件

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_loadable_suspense_lazy_1_base.png)

点击 Sample2 后才加载 Sample2 组件的脚本，同时下面的红色框部分是点击后才输出，说明 Sample2 组件确实是在切换之后才被实实在在的下载下来

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_loadable_suspense_lazy_2_import.png)

# 2. 简单实现

看过 react-loadable 之后好像也不是太难，下面我们动手实现一个

- `/src/components/index.tsx`

```ts
interface ComponentModule {
  default: React.ComponentType;
}

interface LazyOptions {
  loader: () => Promise<ComponentModule>;
  loading: React.ComponentType;
}
```

首先是参数类型，这里注意的是由于我们的懒加载实际上依赖了 webpack 提供的动态 `import()` 特性来实现，而该函数返回的数据则是类似 `Promise<{ default: Component }>` 的形式，因此我们定义了 `ComponentModule` 类型（同时可以参考 react-loadable 源码的类型定义）

接下来则是定义一个 `lazy` 高阶函数，返回一个内部组件，使用闭包来绑定要包装的目标组件

```ts
const lazy = (options: LazyOptions) => {
  const { loader, loading: Loading } = options;

  let Component = null;

  const InnerComponent: FC = (props) => {
    const [loading, setLoading] = useState(Component === null);

    useEffect(() => {
      if (!loading) {
        return;
      }
      loader().then((comp) => {
        Component = comp.default;
        setLoading(false);
      });
    }, []);

    return loading ? <Loading></Loading> : <Component {...props} />;
  };

  return React.memo(InnerComponent);
};
```

由于实际上我们可能会渲染组件多次，因此我们将组件加载后记录在闭包里面，然后使用 `useMount = useEffect(cb, [])` 来进行动态加载组件，并在加载完成之后撤销 `loading` 状态

最后我们就可以像前面使用 Loadable 方法一样来将一个组件懒加载化

```ts
export const Sample3 = lazy({
  loader: () => import('./Sample3'),
  loading: Loading,
});
```

- 实现效果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_loadable_suspense_lazy_3_implement.png)

一样下面红色部分是切换后才进行加载

# 3. 使用 Suspense + lazy 实现

上面的实现还是比较简陋，然而这样的问题实际上在真实场景之中需求非常大，React 也因此提供了官方版的标准措施：使用 `Suspense` 组件 + `React.lazy` API 的组合技来实现懒加载的效果

- `/src/components/index.tsx`

```ts
const lazy2 = (options: LazyOptions) => {
  const { loader, loading: Loading } = options;

  const CustomComponent = React.lazy(loader);

  const InnerComponent = () => {
    return (
      <Suspense fallback={<Loading />}>
        <CustomComponent />
      </Suspense>
    );
  };

  return React.memo(InnerComponent);
};
```

我们可以看到代码量一次缩减了非常多

```ts
export const Sample4 = lazy2({
  loader: () => import('./Sample4'),
  loading: Loading,
});
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_loadable_suspense_lazy_4_suspense_lazy.png)

# 4. 闪烁问题

上述几种实现看似都非常好，自己实现的可以理解一下写法，生产环境还是可以用一用封装好的哈

最后我们来讨论一个问题，关于懒加载产生的骨架屏闪烁问题。由于我们的懒加载都是透过实时的动态加载，有时候用户的网速其实非常良好，加上加载资源非常少，因此可以在很短的时间内就加载完成（ex：300ms内），这时候就会产生骨架屏闪了一下的问题，这就是所谓的闪烁问题。

这时候实际上我们可以有几种方案：

1. 加载时间小于某阈值的时候直接放弃骨架屏（用户体感顺畅）
2. 超过一定时间才显示骨架屏（当然也不能太长，否则像是网页很卡一样）

下面我们拿 `Suspense` 为例子，我们可以对 loader 函数进行加工

- `/src/components/index.tsx`

```ts
export const Sample4 = lazy2({
  loader: () =>
    Promise.all([
      import('./Sample4'),
      new Promise((resolve) => setTimeout(resolve, 1000)),
    ]).then(([moduleExports]) => moduleExports),
  loading: Loading,
});
```

这里的实现方案实际上是强制实现第二种情况，将真实的加载 `import(Component)` 与一个定时器混合，保证最少要 1000ms 后才会完成加载操作。如果读者对第一种实现有兴趣可以自己动手试试，最终效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_loadable_suspense_lazy_5_flash.png)

---

# 参考连接

| Title                                             | Link                                                                                                                                             |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| jamiebuilds/react-loadable - Github               | [https://github.com/jamiebuilds/react-loadable](https://github.com/jamiebuilds/react-loadable)                                                   |
| Code-Splitting - React                            | [https://zh-hant.reactjs.org/docs/code-splitting.html#reactlazy](https://zh-hant.reactjs.org/docs/code-splitting.html#reactlazy)                 |
| Suspense for Data Fetching (Experimental) - React | [https://zh-hant.reactjs.org/docs/concurrent-mode-suspense.html](https://zh-hant.reactjs.org/docs/concurrent-mode-suspense.html)                 |
| React suspense/lazy delay? - stackoverflow        | [https://stackoverflow.com/questions/54158994/react-suspense-lazy-delay](https://stackoverflow.com/questions/54158994/react-suspense-lazy-delay) |
| 模块方法 - webpack                                | [https://webpack.docschina.org/api/module-methods/](https://webpack.docschina.org/api/module-methods/)                                           |

# 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_loadable_suspense_lazy](https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_loadable_suspense_lazy)
