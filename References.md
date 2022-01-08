# 重要連結參考 / 三方库依赖

<!-- TOC -->

- [重要連結參考 / 三方库依赖](#重要連結參考--三方库依赖)
- [Front-end 前端相关](#front-end-前端相关)
  - [客户端响应式框架](#客户端响应式框架)
    - [React 体系](#react-体系)
    - [Vue 体系](#vue-体系)
  - [前端路由相关](#前端路由相关)
  - [通用状态管理](#通用状态管理)
  - [服务端框架](#服务端框架)
    - [Express 体系](#express-体系)
    - [Koa 体系](#koa-体系)
  - [CSS 预处理器](#css-预处理器)
  - [Compiler 编译工具](#compiler-编译工具)
    - [Babel 体系](#babel-体系)
  - [Bundler 打包工具](#bundler-打包工具)
    - [Webpack 体系](#webpack-体系)
  - [前端工程化](#前端工程化)
  - [可视化图形库](#可视化图形库)
  - [CDN 分发网络](#cdn-分发网络)
  - [UI 组件库](#ui-组件库)
  - [测试框架](#测试框架)
  - [时间处理](#时间处理)
  - [其他 & 三方工具库](#其他--三方工具库)
  - [游戏框架](#游戏框架)
    - [Docs 教学文档](#docs-教学文档)
- [Back-end 后端](#back-end-后端)
  - [后端框架](#后端框架)
    - [Spring 相关](#spring-相关)
  - [Deployment 部署](#deployment-部署)

<!-- /TOC -->

# Front-end 前端相关

## 客户端响应式框架

| Name      | Link                      |
| --------- | ------------------------- |
| React     |                           |
| Vue       |                           |
| Angular   | https://angular.cn/       |
| Svelte    | https://www.sveltejs.cn/  |
| Bootstrap | https://getbootstrap.com/ |
|           |                           |

### React 体系

| Name         | Link                         | Usage      |
| ------------ | ---------------------------- | ---------- |
| React        | https://zh-hans.reactjs.org/ | 响应式框架 |
| React Router | https://reactrouter.com/     | 前端路由   |
| Recoil       |                              | 状态管理   |
| Next         | https://www.nextjs.cn/       | 服务端渲染 |
|              |                              |            |


### Vue 体系

| Name       | Link                            | Usage      |
| ---------- | ------------------------------- | ---------- |
| Vue        | https://cn.vuejs.org/index.html | 响应式框架 |
| Vue Router | https://router.vuejs.org/zh/    | 前端路由   |
| Vuex       | https://vuex.vuejs.org/zh/      | 状态管理   |
| Pinia      |                                 | 状态管理   |
| Nuxt       | https://zh.nuxtjs.org/          | 服务端渲染 |
|            |                                 |            |

## 前端路由相关

| Name    | Link                                     | Usage                   |
| ------- | ---------------------------------------- | ----------------------- |
| history | https://github.com/ReactTraining/history | 浏览器 History API 封装 |
|         |                                          |                         |

## 通用状态管理

| Name  | Link                            | Usage |
| ----- | ------------------------------- | ----- |
| Redux | https://redux.js.org/           | 通用  |
| MobX  | https://mobx.js.org/README.html | 通用  |
|       |                                 |       |

## 服务端框架

| Name    | Link                         | Usage          |
| ------- | ---------------------------- | -------------- |
| Node    | https://nodejs.org/en/       | JS 运行时      |
| Express | https://expressjs.com/zh-tw/ | Web 服务端框架 |
| Koa     | https://koa.bootcss.com/     | Web 服务端框架 |
| Egg     | https://eggjs.org/en/        | Web 服务端框架 |
| Nest    | https://nestjs.com/          | Web 服务端框架 |
|         |                              |                |

### Express 体系

| Name    | Link                         | Usage          |
| ------- | ---------------------------- | -------------- |
| Express | https://expressjs.com/zh-tw/ | Web 服务端框架 |
|         |                              |                |

### Koa 体系

| Name        | Link                              | Usage                  |
| ----------- | --------------------------------- | ---------------------- |
| Koa         | https://koa.bootcss.com/          | Web 服务端框架         |
| @koa/cors   | https://github.com/koajs/cors     | 中间件 - 跨域请求      |
| @koa/multer | https://github.com/koajs/multer   | 中间件 - 文件下载      |
| @koa/router | https://github.com/koajs/router   | 中间件 - API 路由      |
| koa-body    | https://github.com/koajs/koa-body | 中间件 - FormData 解析 |
| koa-static  | https://github.com/koajs/static   | 中间件 - 静态资源分发  |
|             |                                   |                        |

## CSS 预处理器

| Name | Link                      |
| ---- | ------------------------- |
| Less | https://less.bootcss.com/ |
| Sass | https://sass-lang.com/    |
|      |                           |

## Compiler 编译工具

| Name       | Link                                            | Usage    |
| ---------- | ----------------------------------------------- | -------- |
| Babel      | https://babeljs.io/<br/>https://www.babeljs.cn/ | ES 通用  |
| TypeScript |                                                 | TS to ES |
| SWC        |                                                 |
|            |                                                 |

### Babel 体系

| Name                              | Link                                                                                 | Usage                   |
| --------------------------------- | ------------------------------------------------------------------------------------ | ----------------------- |
| @babel/cors                       | https://babel.dev/docs/en/babel-core<br/>https://github.com/babel/babel              | Babel 核心              |
| @babel/preset-react               | https://babel.dev/docs/en/babel-preset-react<br/>https://github.com/babel/babel      | Preset - for React      |
| @babel/preset-typescript          | https://babel.dev/docs/en/babel-preset-typescript<br/>https://github.com/babel/babel | Preset - for Typescript |
| @babel/plugin-proposal-decorators |                                                                                      |                         |
|                                   |                                                                                      |                         |

## Bundler 打包工具

| Name     | Link                                                       | Usage            |
| -------- | ---------------------------------------------------------- | ---------------- |
| Webpack  | https://webpack.js.org/<br/>https://webpack.docschina.org/ | 通用（应用程序） |
| Parcel   | https://zh.parceljs.org/                                   | 通用             |
| Rollup   | https://rollupjs.org/guide/zh/                             | 通用（三方库）   |
| Vite     | https://www.vitejs.net/                                    | 打包工具         |
| Snowpack |                                                            |
| Skypack  |                                                            |
| Umi      |                                                            |
| Ice      |                                                            |
| esbuild  |                                                            |
| Rome     |                                                            |
|          |                                                            |

### Webpack 体系

| Name                 | Link                                             | Usage                        |
| -------------------- | ------------------------------------------------ | ---------------------------- |
| webpack              |                                                  | webpack 核心                 |
| webpack-dev-server   |                                                  | webpack 开发用服务器         |
| webpack-cli          |                                                  | webpack 交互式命令           |
| clean-webpack-plugin | https://github.com/johnagan/clean-webpack-plugin | Plugin - 清理打包目录        |
| html-webpack-plugin  |                                                  | Plugin - html 模版生成插件   |
| style-loader         |                                                  | Loader - css 作为 style 引入 |
| css-loader           |                                                  | Loader - 引入 .css           |
| sass-loader          |                                                  | Loader - 引入 .sass / .scss  |
| less-loader          |                                                  | Loader - 引入 .less          |
| vue-loader           |                                                  | Loader - 引入 .vue           |
| babel-loader         |                                                  | Loader - 引入 babel          |
|                      |                                                  |                              |


## 前端工程化

| Name        | Link                                  | Usage                              |
| ----------- | ------------------------------------- | ---------------------------------- |
| Husky       | https://typicode.github.io/husky/#/   | Git Hook                           |
| commitlint  | https://commitlint.js.org/#/          | 静态检查 - Commit Message          |
| eslint      | https://eslint.bootcss.com/           | 静态检查 - 代码(js、ts)            |
| stylelint   | https://stylelint.io/                 | 静态检查 - 样式(css、less、scss)   |
| lint-staged | https://github.com/okonet/lint-staged | 静态检查 - Git Staged 阶段钩子扩展 |
| Prettier    | https://prettier.io/                  | 代码格式化                         |
|             |                                       |                                    |

## 可视化图形库

| Name       | Link                                    | Usage          |
| ---------- | --------------------------------------- | -------------- |
| BigChart   |                                         |                |
| ECharts    | http://echarts.apache.org/zh/index.html | 图形库(canvas) |
| HighCharts | https://www.highcharts.com.cn/          | 图形库(canvas) |
| d3         | https://d3js.org/                       | 图形库(svg)    |
| threejs    | https://threejs.org/                    | 3D 图形库      |
| G6         |                                         |                |
|            |                                         |                |

## CDN 分发网络

| Name      | Link                      |
| --------- | ------------------------- |
| unpkg     | https://unpkg.com/        |
| cdnjs     | https://cdnjs.com/        |
| jsDeliver | https://www.jsdelivr.com/ |
|           |                           |

## UI 组件库

| Name        | Link                                   | Usage                              |
| ----------- | -------------------------------------- | ---------------------------------- |
| ElementUI   | https://element.eleme.io/#/zh-CN       | 通用组件库 for React, Vue, Angular |
| Ant Design  | https://ant.design/index-cn            | 通用组件库 for React, Vue          |
| Vuetify     | https://vuetifyjs.com/en/              | 通用组件库 for Vue                 |
| iView       | http://v1.iviewui.com/                 | 通用组件库 for Vue                 |
| Vant        | https://youzan.github.io/vant/#/zh-CN/ | 移动端 UI for Vue                  |
| MintUI      | http://mint-ui.github.io/#!/zh-cn      | 移动端 UI for Vue                  |
| KUI         | https://react.k-ui.cn/#/               | 通用组件库 for React               |
| React Suite | https://rsuitejs.com/                  | 通用组件库 for React               |
| MaterailUI  | https://material-ui.com/zh/            | 通用组件库 for React               |
| LayUI       | https://www.layui.com/                 |
| chakra UI   | https://chakra-ui.com/                 | 通用组件库 for React               |
| Framer      | https://www.framer.com/                | 通用组件库 for React               |
|             |                                        |                                    |

## 测试框架

| Name | Link               |
| ---- | ------------------ |
| jest | https://jestjs.io/ |
|      |                    |

## 时间处理

| Name   | Link                         |
| ------ | ---------------------------- |
| moment | http://momentjs.cn/          |
| dayjs  | https://dayjs.fenxianglu.cn/ |
|        |                              |

## 其他 & 三方工具库

| Name              | Link                                               | Usage               |
| ----------------- | -------------------------------------------------- | ------------------- |
| core-js           | https://github.com/zloirock/core-js/               | 补丁 - JS 语言核心  |
| core-decorators   | https://github.com/jayphelps/core-decorators       | 补丁 - 核心注解实现 |
| traits-decorators | https://github.com/CocktailJS/traits-decorator     | 补丁 - 进阶注解实现 |
| lodash            | https://www.lodashjs.com/                          | 工具函数库          |
| axios             | https://github.com/axios/axios                     | Http 请求           |
| Storybook         | https://storybook.js.org/                          | 代码演示            |
| js-cookie         | https://github.com/js-cookie/js-cookie             | cookie 操作         |
| nprogress         | https://github.com/rstacruz/nprogress              | 进度条实现          |
| JQuery            | https://jquery.com/<br/>https://www.jquery123.com/ | DOM 操作库          |
| JSZip             | https://stuk.github.io/jszip/                      | 多文件压缩          |
| SparkMD5          | https://github.com/satazor/js-spark-md5            | MD5 算法特征值生成  |
| fs-extra          | https://github.com/jprichardson/node-fs-extra      | fs 扩展             |
| clipboard         | https://clipboardjs.com/                           | 剪贴板              |
| Gitgraph          | https://gitgraphjs.com/#0                          | Git 分支图          |
|                   |                                                    |                     |

## 游戏框架

| Name     | Link                |
| -------- | ------------------- |
| CreateJS | http://createjs.cc/ |
|          |                     |

### Docs 教学文档

| Name     | Link                       |
| -------- | -------------------------- |
| ES6 文檔 | http://caibaojian.com/es6/ |

# Back-end 后端

## 后端框架

| Name   | Link               | Usage                 |
| ------ | ------------------ | --------------------- |
| Spring | https://spring.io/ | 通用框架 - for Java   |
| Flask  |                    | Web 框架 - for Python |
| .NET   |                    | 通用框架 - for C#     |
|        |                    |                       |

### Spring 相关

| Name                 | Link                      |
| -------------------- | ------------------------- |
| Baeldung-Spring 社區 | https://www.baeldung.com/ |
|                      |                           |

## Deployment 部署

| Name       | Link                    |
| ---------- | ----------------------- |
| Docker     | https://www.docker.com/ |
| Docker Hub | https://hub.docker.com/ |
| Kubernetes |                         |
|            |                         |
