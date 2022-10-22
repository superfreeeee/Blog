# i18next 国际化 & 与 React 联动

@[TOC](文章目录)

<!-- TOC -->

- [i18next 国际化 & 与 React 联动](#i18next-国际化--与-react-联动)
- [完整代码示例](#完整代码示例)
- [基础版](#基础版)
- [React 组件联动](#react-组件联动)
- [参考链接](#参考链接)

<!-- /TOC -->

# 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/others/i18next_sample](https://github.com/superfreeeee/Blog-code/tree/main/front_end/others/i18next_sample)

# 基础版

- 安装 & 初始化

```bash
pnpm i i18next
```

- `/src/locales/index.ts`

```ts
/**
 * Init i18next when loaded
 */
i18next.init({
  // lng: 'en',
  debug: true,
  resources: {
    zh: {
      translation: {
        "greeting": "早安藍天"
      },
    },
    en: {
      translation: {
        "greeting": "Hello World"
      },
    },
  },
});

/**
 * 导出方法，保证 i18next 正确初始化
 */
export const i18n = (key: string, options?: TOptions): string => {
  return i18next.t(key, options);
};
```

而使用的时候其实就是根据 key 去 i18n 维护的那个对象里面找文案罢了

```ts
const App: FC = () => {
  return (
    <div className={styles.container}>
      <h1>React App</h1>
      <div>Project build by @youxian/cli</div>
      <h2>{i18n('greeting')}</h2>
      <div>
        <button onClick={() => changeLanguage('zh')}>中文</button>
        <button onClick={() => changeLanguage('en')}>英文</button>
      </div>
    </div>
  );
};
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/i18next_sample_1.png)

而我们想要变换语言的时候就可以使用 `changeLanguage` 方法，也是稍微封装一下保证 i18n 初始化

```ts
export const changeLanguage = (lang: string) => {
  i18next.changeLanguage(lang);
};
```

# React 组件联动

然而你会发现使用上面的组件的时候点按钮是无效的，这是因为 `i18next.t` 本身其实就是一个静态方法，是与 React 组件的状态更新无关联的，因此我们如果想要动态的在运行时改变语言的话，我们就要再加上 `react-i18next` 的包装

- `/src/locales/index.ts`

首先是初始化的地方需要也把 react18next 也初始化

```ts
/**
 * Init i18next when loaded
 */
i18next.use(initReactI18next).init({
  // lng: 'en',
  debug: true,
  resources,
});
```

使用的时候我们就需要用 Hook 或是 HOC 将浏览器语言作为一种状态与 React 组件的渲染进行关联，这里以 Hooks 举例

```ts
export const useTrans = () => {
  const { t } = useTranslation();
  return (key: string, options?: TOptions) => t(key, options);
};
```

最后在组件内部改成使用 Hooks，就能够在语言更新的时候重渲染组件来看到最新的文案啦

```ts
const App: FC = () => {
  const t = useTrans();

  return (
    <div className={styles.container}>
      <h1>React App</h1>
      <div>Project build by @youxian/cli</div>
      <h2>{t('greeting')}</h2>
      <div>
        <button onClick={() => changeLanguage('zh')}>中文</button>
        <button onClick={() => changeLanguage('en')}>英文</button>
      </div>
    </div>
  );
};
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/i18next_sample_2.png)

# 参考链接

| Title         | Link                                                     |
| ------------- | -------------------------------------------------------- |
| i18next       | [https://www.i18next.com/](https://www.i18next.com/)     |
| react-i18next | [https://react.i18next.com/](https://react.i18next.com/) |
