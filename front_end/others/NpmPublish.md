# Npm 发布: 发布一个自己的 npm 三方包

@[TOC](文章目录)

<!-- TOC -->

- [Npm 发布: 发布一个自己的 npm 三方包](#npm-发布-发布一个自己的-npm-三方包)
- [前言](#前言)
- [正文](#正文)
  - [1. 准备一个 npm 项目](#1-准备一个-npm-项目)
    - [1.1 目录结构 & 项目源代码](#11-目录结构--项目源代码)
    - [1.2 package.json 配置](#12-packagejson-配置)
  - [2. npm 发布流程](#2-npm-发布流程)
    - [2.1 注册 npm 账号](#21-注册-npm-账号)
    - [2.2 添加组织](#22-添加组织)
    - [2.3 登录](#23-登录)
      - [2.3.1 切换 yarn 源](#231-切换-yarn-源)
    - [2.4 发布包](#24-发布包)
    - [2.5 发布结果](#25-发布结果)
  - [3. 打包后发布](#3-打包后发布)
    - [3.1 加入 babel 依赖](#31-加入-babel-依赖)
    - [3.2 源代码修改](#32-源代码修改)
    - [3.3 babel.config.json、package.json 配置文件](#33-babelconfigjsonpackagejson-配置文件)
    - [3.4 yarn add 引用依赖](#34-yarn-add-引用依赖)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

今天带大家一起走一次发布 npm 包的流程，同时也是接上上一篇 SDK 开发的流程，并作为后续开发自己的工具包做准备。下面马上开始

# 正文

## 1. 准备一个 npm 项目

首先第一步我们想要发包，那得先有一个包，先来搞个项目

### 1.1 目录结构 & 项目源代码

- `/package`

```tree
/package
├── package.json
├── src
│   └── index.js
└── yarn.lock
```

index.js 的内容很简单

- `/package/src/index.js`

```js
const f1 = () => {
  console.log('invoke f1');
};

module.exports = { f1 };
```

### 1.2 package.json 配置

接下来是 npm 发包的一个重点，package.json 要写对

- `/package/package.json`

```json
{
  "name": "@youxian/test-pkg",
  "version": "1.0.0-alpha.0",
  "main": "index.js",
  "license": "MIT"
}
```

首先是包名，我们利用 npm 项目名称的组织作用域(`@[orgName]/[pkgName]`)的方法来命名包名；然后版本号则使用测试版的 alpha；主入口则是设置为 index.js 即可

## 2. npm 发布流程

接下来我们就可以进入正式的发布流程了，本篇使用 yarn 作为项目的依赖于版本管理器

### 2.1 注册 npm 账号

首先第一步是去 npm 注册一个账号，如下

npm 地址：[https://www.npmjs.com/](https://www.npmjs.com/)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/npm_publish_1_register.png)

### 2.2 添加组织

由于 npm 的包还是非常多的，很容易出现重名的问题，所以比较好的方式是建立一个组织，透过组织名来进行作用域的区隔，像是 `@babel/core`、`@ali/xxx` 等方式

这边我也搞了一个自己的组织，用的花名哈哈

![](https://picures.oss-cn-beijing.aliyuncs.com/img/npm_publish_2_organization.png)

### 2.3 登录

接下来就是核心的发布流程了，首先现在命令行上进入项目的根目录并进行登录

- 登录：

```bash
$ yarn login
```

输入账号密码就能登录啦

![](https://picures.oss-cn-beijing.aliyuncs.com/img/npm_publish_3_login.png)

#### 2.3.1 切换 yarn 源

如果登录过程失败的话可以检查一下 yarn 的源是否正确，不要登错仓库啦hh

```bash
$ yarn config set registry https://registry.npmjs.org/
```

当然这个路径每次都要记着也是麻烦，可以下一个 yrm 工具方便切换

```bash
$ yarn global add yrm
$ yrm ls

* npm ---- https://registry.npmjs.org/
  cnpm --- http://r.cnpmjs.org/
  taobao - https://registry.npm.taobao.org/
  nj ----- https://registry.nodejitsu.com/
  rednpm - http://registry.mirror.cqupt.edu.cn/
  npmMirror  https://skimdb.npmjs.com/registry/
  edunpm - http://registry.enpmjs.org/
  yarn --- https://registry.yarnpkg.com

```

### 2.4 发布包

接下来最后一条指令就是调用 `yarn publish` 进行发布啦

```bash
$ yarn publish
```

如果这是这个包第一次发布的话，你还需要加上额外的参数 `--access=public` 来指定包的可访问性，也就是别人能不能下载啦

```bash
$ yarn publish --access=public
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/npm_publish_4_publish.png)

yarn 会告诉你上一次发布的版本（如图为 1.0.0-alpha.4），前一次发布成功的版本是不允许覆盖的，你只能继续往上加，所以如图我改成 1.0.0-alpha.5 了，这时候不管是 yarn 还是 npm 都会自动的为我们更新 package.json 中的版本号

### 2.5 发布结果

最后就可以到 npm 网页上看到发布结果啦

![](https://picures.oss-cn-beijing.aliyuncs.com/img/npm_publish_5_result.png)

## 3. 打包后发布

我们刚刚看到的是一个再简单不过的项目示例，实际上我们通常比较少直接将源代码进行发布，而是需要进行额外的打包再进行发布，本篇以 babel 为例

### 3.1 加入 babel 依赖

接下来我们打算把项目改造为 typescript 语言编写的项目，因此需要加入 babel 来进行编译，同时能够支持 ES5 甚至更低的兼容性

所以第一步是添加依赖

```bash
$ yarn add @babel/core @babel/cli @babel/preset-env @babel/preset-typescript -D
```

注意我们这里用的是 -D，也就是将依赖写入到 devDependencies，也就是说明在打包的时候不会将这些包打包到最终结果中，同时引用了这个包的人也会自动的纳入 devDependencies 的管理当中

### 3.2 源代码修改

接下来我们修改一下源代码，导出两个方法

- `/package/src/index.ts`

```ts
const tryParse = (text: string): any => {
  try {
    const data = JSON.parse(text);
    return data;
  } catch (e) {
    console.log('[tryParse] fail', e);
    return null;
  }
};

const tryStringify = (value: any): string => {
  try {
    const text = JSON.stringify(value);
    return text;
  } catch (e) {
    console.log('[tryStringify] fail', e);
    return null;
  }
};

export { tryParse, tryStringify };
```

### 3.3 babel.config.json、package.json 配置文件

最后加上 babel.config.json 配置文件

- `/package/babel.config.json`

```json
{
  "presets": ["@babel/preset-env", "@babel/preset-typescript"]
}
```

然后在 package.json 中添加打包指令

- `/package/package.json`

```diff
{
  "name": "@youxian/test-pkg",
  "version": "1.0.0-alpha.5",
- "main": "index.js",
+ "main": "lib/index.js",
  "license": "MIT",
+ "scripts": {
+   "build": "rm -rf lib/* && babel src/ -d lib/ -x .ts"
+ },
}
```

我们透过将打包的成果放到 lib 目录下，然后修改项目的主入口 main，指向 lib 目录下的 index 文件

### 3.4 yarn add 引用依赖

最后一样透过 2. 的发布流程再发个版，就能够在 node_modules 里面看到自己写的三方包啦

```bash
$ yarn add @youxian/test-pkg
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/npm_publish_6_import.png)

# 结语

本篇到这里就结束啦，带大家过一遍发布 npm 包的流程，用的是 yarn 包管理器。网上的资料还是非常丰富的，不过我认为如果读者自己是真的有兴趣并且有这个需求，最好还是自己实践一下，动手做对自己的帮助还是比较大的。

下一篇作者希望继续朝 monorep 的项目构建方式继续进行研究，有兴趣的欢迎提出来讨论，供大家参考哈。

# 其他资源

## 参考连接

| Title                                                                                                | Link                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ---------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Publishing a Package - yarn                                                                          | [https://classic.yarnpkg.com/en/docs/publishing-a-package/](https://classic.yarnpkg.com/en/docs/publishing-a-package/)                                                                                                                                                                                                                                                                                                                                                                                                                             |
| Yarn, npm and Git                                                                                    | [https://warlord0blog.wordpress.com/2018/07/31/yarn-npm-and-git/](https://warlord0blog.wordpress.com/2018/07/31/yarn-npm-and-git/)                                                                                                                                                                                                                                                                                                                                                                                                                 |
| 一分钟教你发布npm包                                                                                  | [https://segmentfault.com/a/1190000023075167](https://segmentfault.com/a/1190000023075167)                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| Code: 402 You must sign up for private packages : @org1/pkgname #1821                                | [https://github.com/lerna/lerna/issues/1821](https://github.com/lerna/lerna/issues/1821)                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| Is anonymous publishing allowed? #212                                                                | [https://github.com/verdaccio/verdaccio/issues/212](https://github.com/verdaccio/verdaccio/issues/212)                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| yarn install error - /usr/local/Cellar/yarn/1.22.11/libexec/lib/cli.js:46104                         | [https://blog.csdn.net/HobbitHero/article/details/119925473](https://blog.csdn.net/HobbitHero/article/details/119925473)                                                                                                                                                                                                                                                                                                                                                                                                                           |
| yarn publish错误                                                                                     | [https://blog.csdn.net/zimeng303/article/details/109770042](https://blog.csdn.net/zimeng303/article/details/109770042)                                                                                                                                                                                                                                                                                                                                                                                                                             |
| yarn报错：error An unexpected error occurred: “https://registry.yarnpkg.com/-/user/org.couchdb。。。 | [https://blog.csdn.net/zimeng303/article/details/109767492](https://blog.csdn.net/zimeng303/article/details/109767492)                                                                                                                                                                                                                                                                                                                                                                                                                             |
| Request failed: internal server error (500)错误解决                                                  | [https://blog.csdn.net/lovewcjl/article/details/44242325](https://blog.csdn.net/lovewcjl/article/details/44242325)                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| npm淘宝镜像和默认镜像切换                                                                            | [https://blog.csdn.net/qq_44917563/article/details/101434732?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.no_search_link&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.no_search_link](https://blog.csdn.net/qq_44917563/article/details/101434732?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.no_search_link&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.no_search_link) |
| .yarnrc - yarn                                                                                       | [https://classic.yarnpkg.com/en/docs/yarnrc/](https://classic.yarnpkg.com/en/docs/yarnrc/)                                                                                                                                                                                                                                                                                                                                                                                                                                                         |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/others/npm_publish](https://github.com/superfreeeee/Blog-code/tree/main/front_end/others/npm_publish)
