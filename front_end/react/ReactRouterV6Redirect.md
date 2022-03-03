# React Router 升级 v6: Redirect 重定向替代方案 

@[TOC](文章目录)

<!-- TOC -->

- [React Router 升级 v6: Redirect 重定向替代方案](#react-router-升级-v6-redirect-重定向替代方案)
- [React Router v6 Redirect 更新](#react-router-v6-redirect-更新)
- [官方推荐方案 1: 使用 Navigate 组件替代](#官方推荐方案-1-使用-navigate-组件替代)
- [官方推荐方案 2: 自定义 Redirect 组件](#官方推荐方案-2-自定义-redirect-组件)
- [自定义 Redirect 示例: 防止异常跳转并上报](#自定义-redirect-示例-防止异常跳转并上报)
- [完整代码示例](#完整代码示例)
- [参考连接](#参考连接)

<!-- /TOC -->

# React Router v6 Redirect 更新

React Router 的第六版做了许多破坏性的更新，其中一个就是将 `<Redirect>` 组件移除了。本篇就来介绍一下几个 Redirect 的替代方案

# 官方推荐方案 1: 使用 Navigate 组件替代

简而言之：直接使用 Navigate 替换原来的 Redirect 组件

- in v5

```ts
<Route path="about" render={() => <Redirect to="about-us" />} />
```

- in v6

```ts
// in v6
<Route path="*" element={<Navigate to="/" replace />} />
```

# 官方推荐方案 2: 自定义 Redirect 组件

更进一步，我们来看一下源码

- [传送门：Navigate 源码](https://github.com/remix-run/react-router/blob/7dca9dc38c837ed94796325b1e0582aa72a9313f/packages/react-router/index.tsx#L165-L186)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/react_router_v6_redirect_1_source.png)

我们可以看到核心代码其实就是一个 `useNavigate` 加上一个 `useEffect`。官方也告诉我们可以仿照 `Navigate` 组件自己重新实现一个

```ts
const Redirect: FC<RedirectProps> = ({ to }) => {
  const navigate = useNavigate();

  useEffect(() => {
    navigate(to, { replace: true });
  });

  return null;
};

// ...

<Route path="*" element={<Redirect to="/" />} />
```

# 自定义 Redirect 示例: 防止异常跳转并上报

当然 react router 官方的初衷实际上是希望用户能够更优雅的处理重定向的问题，而不是各种异常境况重定向就完事。因此合并跳转和定向的方法，合成了新的 `useNavigate` 钩子，让我们能在如上自定义组件内对重定向行为有更详细的描述

下面我们提供一个扩张自定义重定向组件的示例，拒绝非法重定向并向控制台上报错误信息

```ts
const Redirect: FC<RedirectProps> = ({ to }) => {
  const navigate = useNavigate();

  useEffect(() => {
    navigate(-1);
    console.error('Bad route');
  });

  return null;
};
```

我们使用 `navigate(-1)` 的方式来回到跳转到异常路由前的页面并打印一个错误信息

---

# 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_router_v6_redirect](https://github.com/superfreeeee/Blog-code/tree/main/front_end/react/react_router_v6_redirect)

# 参考连接

| Title                                                              | Link                                                                                                                                                                                                                                                                   |
| ------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Remove \<Redirect\>s inside \<Switch\> - React Router Docs         | [https://reactrouter.com/docs/en/v6/upgrading/v5#remove-redirects-inside-switch](https://reactrouter.com/docs/en/v6/upgrading/v5#remove-redirects-inside-switch)                                                                                                       |
| What about clicking Links that aren't updated? - React Router Docs | [https://reactrouter.com/docs/en/v6/upgrading/reach#what-about-clicking-links-that-arent-updated](https://reactrouter.com/docs/en/v6/upgrading/reach#what-about-clicking-links-that-arent-updated)                                                                     |
| Navigate Component - remix-run/react-router - Github               | [https://github.com/remix-run/react-router/blob/7dca9dc38c837ed94796325b1e0582aa72a9313f/packages/react-router/index.tsx#L165-L186](https://github.com/remix-run/react-router/blob/7dca9dc38c837ed94796325b1e0582aa72a9313f/packages/react-router/index.tsx#L165-L186) |
