# Webpack: CSS 到 Sass/Scss 与 CSS Module

@[TOC](文章目录)

<!-- TOC -->

- [Webpack: CSS 到 Sass/Scss 与 CSS Module](#webpack-css-到-sassscss-与-css-module)
- [前言](#前言)
- [正文](#正文)
  - [1. 在 JS 中引入 CSS：style-loader、css-lodaer](#1-在-js-中引入-cssstyle-loadercss-lodaer)
  - [2. 从 CSS 进阶到 SCSS：sass-loader + node-sass](#2-从-css-进阶到-scsssass-loader--node-sass)
  - [3. CSS Module 实现命名空间隔离：css-loader](#3-css-module-实现命名空间隔离css-loader)
    - [3.1 Typescript 场景下使用 CSS Module](#31-typescript-场景下使用-css-module)
  - [Bonus: 踩坑记录](#bonus-踩坑记录)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

今天给大家分享的是关于在 webpack 场景下使用 SCSS 并加上 CSS Module 的配置方法

# 正文

之前有过几篇关于 webpack 的介绍，这里就默认大家都对 webpack 有初步的了解了（本篇主要会用到 loader 的部分）

## 1. 在 JS 中引入 CSS：style-loader、css-lodaer

第一部分先从基本的来，在不直接将 css 写入 html 的场景下，我们就需要将 css 文件作为一个独立的 module 由 webpack 解析并插入，这时候就需要 `style-loader` 与 `css-loader`

- 安装依赖

```bash
$ yarn add style-loader css-lodaer -D
```

- `webpack.config.js`

```js
const config = {
  // ...
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
    ]
  },
  // ...
}
```

接下来我们就可以直接在 JS 文件内引入（以 React 为例）

- `/src/layouts/Test1/index.css`

```css
.test1 h2::before {
  content: '<t1>';
}
```

- `/src/layouts/Test1/index.tsx`

```ts
import React from 'react';

import './index.css';

const Test1 = () => {
  return (
    <div className="test1">
      <h2>Test1</h2>
    </div>
  );
};

export default Test1;
```

效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_scss_css_module_1_css.png)

## 2. 从 CSS 进阶到 SCSS：sass-loader + node-sass

接下来 css 越写越多，可能就有点不太好用了，所以接下来我们选择上 Sass/Scss（当然你比较喜欢 Less 也是差不多的）

- 安装依赖

```bash
$ yarn add sass-loader node-sass -D
```

- `webpack.config.js`

```js
const config = {
  // ...
  module: {
    rules: [
      {
        test: /\.scss$/,
        use: ['style-loader', 'css-loader', 'sass-loader'],
      },
    ]
  },
  // ...
}
```

- `/src/layouts/Test2/index.scss`

```css
.test2 h2::before {
  content: '<t2>';
}
```

- `/src/layouts/Test2/index.tsx`

```ts
import React from 'react';

import './index.scss';

const Test2 = () => {
  return (
    <div className="test2">
      <h2>Test2</h2>
    </div>
  );
};

export default Test2;
```

效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_scss_css_module_2_scss.png)

## 3. CSS Module 实现命名空间隔离：css-loader

然而 scss 实际上只是一种 css 语法上的增强，但是依旧会产生命名冲突的问题，这时候我们就可以选择使用 CSS Module 的特性（实际上 css-loader 已经给出相关的配置属性，所以并不需要额外的 loader）

- `webpack.config.js`

```js
const config = {
  // ...
  module: {
    rules: [
      {
        test: /(\.module)?.(sass|scss)$/,
        use: [
          'style-loader',
          {
            loader: 'css-loader',
            options: {
              modules: {
                localIdentName: '[path][name]__[local]--[hash:base64:5]',
              },
              sourceMap: true,
            },
          },
          'sass-loader',
        ],
      },
    ]
  },
  // ...
}
```

- `/src/layouts/Test3/index.scss`

```css
.test3 h2::before {
  content: '<t3>';
}
```

- `/src/layouts/Test3/index.tsx`

```ts
import React from 'react';

import styles from './index.module.scss';

console.log('styles', styles);

const Test3 = () => {
  return (
    <div className={styles.test3}>
      <h2>Test3</h2>
    </div>
  );
};

export default Test3;
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_scss_css_module_3_scss_module.png)

这时候我们可以看到，css 的名字被加上了目录信息甚至还有 hash 值，使得多个文件之间的 css 几乎不可能重名

### 3.1 Typescript 场景下使用 CSS Module

由于使用 CSS Module 之后，我们需要使用

```ts
import styles from './index.module.scss';
```

才能获取正确的样式名称，但是 ts 是看不懂 .scss 后缀的文件的，这时候我们就可以在 src 目录下建立一个 `styles.d.ts` 的类型声明文件

- `styles.d.ts`

```ts
declare module '*.css' {
  const styles: { readonly [key: string]: string };
  export default styles;
}

declare module '*.scss' {
  const styles: { readonly [key: string]: string };
  export default styles;
}
```

如此一来 TS 就不会再报错了

## Bonus: 踩坑记录

如果你在配置的过程中遇到一堆奇奇怪怪的报错，以下是我踩过的几个坑可以检查一下是不是有相同的问题：

- 没有安装 `node-sass`：要解析 sass/scss 除了 sass-loader 之外，还需要 node-sass 解析器才能正确解析内容
- loader 顺序：老生常谈了，由后向前
- 多个相似配置：有一个问题是 scss 不能在同一个项目内重复定义规则，也就是 **不可以决定一些用 CSS loader，一些不用**，不然可能会产生一堆看不懂的报错
- node-sass 下载有够慢：参考链接里面有关于配置 yarn 源仓库的方法

# 结语

本篇就到此结束，其实就是一些关于 webpack 的配置手段，供大家参考。

# 其他资源

## 参考连接

| Title                             | Link                                                                                                                     |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| css-loader - Webpack 官方         | [https://webpack.js.org/loaders/css-loader/](https://webpack.js.org/loaders/css-loader/)                                 |
| devtool - Webpack 官方            | [https://webpack.js.org/configuration/devtool/](https://webpack.js.org/configuration/devtool/)                           |
| CSS Modules 用法教程 - 阮一峰     | [http://www.ruanyifeng.com/blog/2016/06/css_modules.html](http://www.ruanyifeng.com/blog/2016/06/css_modules.html)       |
| Webpack基于baseUrl和paths引入模块 | [https://juejin.cn/post/6901081123078537224](https://juejin.cn/post/6901081123078537224)                                 |
| typescript项目css modules         | [https://segmentfault.com/a/1190000021684206](https://segmentfault.com/a/1190000021684206)                               |
| yarn配置阿里源                    | [https://blog.csdn.net/qq_19107011/article/details/97992167](https://blog.csdn.net/qq_19107011/article/details/97992167) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/webpack/webpack_scss_css_module](https://github.com/superfreeeee/Blog-code/tree/main/front_end/webpack/webpack_scss_css_module)
