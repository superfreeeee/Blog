# Lerna 项目启动（Monorepo 实践）

@[TOC](文章目录)

<!-- TOC -->

- [Lerna 项目启动（Monorepo 实践）](#lerna-项目启动monorepo-实践)
- [前言](#前言)
- [正文](#正文)
  - [1. 项目构建](#1-项目构建)
    - [1.1 初始化根目录](#11-初始化根目录)
    - [1.2 目录结构](#12-目录结构)
    - [1.3 使用 yarn 作为依赖管理器](#13-使用-yarn-作为依赖管理器)
  - [2. 子项目构建](#2-子项目构建)
    - [2.1 创建子项目](#21-创建子项目)
    - [2.2 安装依赖](#22-安装依赖)
    - [2.3 项目内容填充](#23-项目内容填充)
    - [2.4 建立软连接](#24-建立软连接)
    - [2.5 打包配置（babel、webpack 配置）](#25-打包配置babelwebpack-配置)
    - [2.6 typescript 开发配置](#26-typescript-开发配置)
  - [3. 运行脚本配置](#3-运行脚本配置)
    - [3.1 根目录命令配置](#31-根目录命令配置)
    - [3.2 各项目命令配置](#32-各项目命令配置)
  - [4. 打包 & 提交 & 发布](#4-打包--提交--发布)
    - [4.1 打包](#41-打包)
    - [4.2 提交到远程仓库](#42-提交到远程仓库)
    - [4.3 发布](#43-发布)
      - [4.3.1 踩坑记录1：访问性设置](#431-踩坑记录1访问性设置)
      - [4.3.2 踩坑记录2：已发布过标签](#432-踩坑记录2已发布过标签)
    - [4.4 npm 查看发布结果](#44-npm-查看发布结果)
  - [5. 安装 & 引用](#5-安装--引用)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

今天带大家认识一个平常比较少接触到的概念：monorepo

虽然也是一个类似多项目管理的方式，但是又跟微服务不太相同，每个项目实际上是可以独立运行、发布的，只不过是存在相互依赖的关系，并且作为一个项目的独立子模块，共同开发。

本篇要介绍的是一个叫做 lerna 的 monorepo 管理库，类似的还有 nx、rushstack 等，不过这些都是后话，有兴趣的都可以去尝试看看。

本篇主要还是使用比较常见的 lerna + yarn workspaces 的构建方式，当然并不是说所有项目都必须要使用 monorepo 的管理方式，有人说这是一种趋势，但不应该是一锅端。在前途并不明朗，你都不知道为什么要 monorepo 的情况下，还是继续使用 multirepo 也并无不可，真正的等到需要的时候再进行迁移也还来得及。

下面我们马上就带大家走一遍使用 lerna 构建一个多项目仓库的管理流程

# 正文

## 1. 项目构建

### 1.1 初始化根目录

首先当然是要初始化一个项目

```bash
$ mkdir lerna_launch
$ cd lerna_launch
```

（真实的测试代码我又把它迁移到 libs 目录下，跟另一个 usage 项目区隔开来了，不过这不是重点哈）

建好目录先别着急，全局装一下 lerna 命令

```bash
$ yarn global add lerna
```

接下来就在项目的根目录下使用 lerna 进行初始化

```bash
$ lerna init
lerna notice cli v4.0.0
lerna info Creating package.json
lerna info Creating lerna.json
lerna info Creating packages directory
lerna success Initialized Lerna files
```

### 1.2 目录结构

网上也有很多资料，`lerna init` 初始化过后的目录结构如下

```tree
/lerna_launch
├── lerna.json
├── package.json
└── packages/
```

简单来说就是两个配置文件：

- 一个是作为多个项目的根项目的 npm 配置文件 `package.json`
- 一个是 lerna 管理的配置文件 `lerna.json`
- 最后一个是放置所有项目的目录 `packages`（这也就是为什么 react、babel 等开源项目都没有一个 src，只看到了一个 packages 目录），而这也不仅仅只是 lerna 才是如此，使用 packages 目录放置项目可以说是 monorepo 的管理器通用的规范，也是业内的推荐实践标准

### 1.3 使用 yarn 作为依赖管理器

lerna 默认是直接使用 npm 作为依赖管理器的，但是说实话还是 yarn 比较出色一些，所以我们需要手动添加一些配置来启用 yarn 作为默认的依赖管理器

- `/libs/package.json`

首先是修改一下 package.json 的配置项

```json
{
  "name": "root",
  "private": true,
  "workspaces": [
    "packages/*"
  ]
}
```

我们在 package.json 中间加上 `"private": true` 表示作为私有包发布，因为根目录其实并不应该被发布；另一个是添加 `"workspaces": ["packages/*"]` 表示启用了 yarn workspaces 的特性，这也是 yarn 支持 monorepo 的正确配置方式

- `/libs/lerna.json`

第二部分则是修改 lerna 的配置项

```json
{
  "packages": [
    "packages/*"
  ],
  "version": "0.0.0",
  "npmClient": "yarn",
  "useWorkspaces": true
}
```

packages 属性和 version 属性在初始化的时候都有了，加上 `"npmClient": "yarn"` 和 `"useWorkspaces": true` 就能够与 yarn workspaces 连用了。

## 2. 子项目构建

有了 lerna 的壳之后，我们就可以先来填充一些内容啦

### 2.1 创建子项目

网上有的教程推荐自己 mkdir 然后 yarn init，也是可以，不过实际上 lerna 早就提供了相关的命令，用用看还是可以的，一些 npm 发布需要的属性都会给你配置好

```bash
$ lerna create pkg-1
$ lerna create pkg-2
```

使用 `lerna create [packageName]` 的方式，它就会自动给你在 packages 下创建你指定的项目名称作为目录名

![](https://picures.oss-cn-beijing.aliyuncs.com/img/lerna_launch_1_repo_create.png)

下面我们稍微修改一下 package.json 的包名，分别是 superfree-testpkg-1、superfree-testpkg-2

- `/libs/packages/pkg-1/package.json`

```json
{
  "name": "superfree-testpkg-1"
}
```

- `/libs/packages/pkg-2/package.json`

```json
{
  "name": "superfree-testpkg-2"
}
```

### 2.2 安装依赖

接下来我们两个项目都想要使用 babel 编译 + webpack 打包的构建模式，所以先来添加依赖。这里不同的是我们可以直接在根目录下（libs 目录下）为不同项目添加依赖

如果使用 lerna 的指令可以这样写

```bash
$ lerna add @babel/core
$ lerna add webpack
# ...
```

由于 lerna 的 add 命令并不支持一次安装多个依赖，但是实际上我们已经启用了 yarn workspace 了，所以如果我们直接选择用 yarn 命令安装也是不冲突的

```bash
$ yarn workspaces add @babel/core @babel/preset-env @babel/preset-typescript webpack webpack-cli babel-loader -D
```

这边注意到我们使用的是 `yarn workspaces` 指令，也就是会在前面配置过的每个 workspaces 匹配的目录下都执行相同的操作，如果我们只想给其中一个项目添加依赖，我们可以使用下面命令

```bash
$ yarn workspace pkg-1 add jest -D
```

这里就指定了 pkg-1 这个项目了

这里使用 yarn 的重点在于它会将所有依赖全部安装到根目录下的 node_modules 当中，就不用去管什么 --hoist 的共同依赖版本提升什么鬼的，反正就是在根目录就对了！

### 2.3 项目内容填充

搞了半天终于可以开始写代码了，我们先把 lib 下 lerna 默认创建的文件删掉，我们再另外创建一个 src 目录来放置源代码，lib 则是后面我们使用 webpack 打包后的目标目录

- `/libs/packages/pkg-1/src/index.ts`

```ts
export const greetingPkg1 = (from: string = '@youxiantest/pkg-1') => {
  console.log(`[greetingPkg1] invoke greetingPkg1 from ${from}`);
};
```

- `/libs/packages/pkg-2/src/index.ts`

```ts
import { greetingPkg1 } from 'superfree-testpkg-1';

export const greetingPkg2 = () => {
  greetingPkg1('@youxiantest/pkg-2');
  console.log('[greetingPkg2] invoke greetingPkg2 from @youxiantest/pkg-2');
};
```

两个子项目的内容还是比较简单，本篇的目标在于打包嘛，比较特别的点在于 pkg-2 依赖于 pkg-1。

### 2.4 建立软连接

在原来的 multirepo 的模式下，我们就需要使用：pkg-1 下 `yarn link` + pkg-2 下 `yarn link superfree-testpkg-1` 的组合技来体现本地不发包的方式，这里就体现出 lerna 强大了

```bash
$ lerna link
```

如此一来 lerna 会自动为 packages 下的所有项目自动进行软连接，一次搞定

### 2.5 打包配置（babel、webpack 配置）

下一步我们为两个项目配置类似的打包环境配置

- `/libs/packages/pkg-1/babel.config.json`
- `/libs/packages/pkg-2/babel.config.json` 相同

```json
{
  "presets": ["@babel/preset-env", "@babel/preset-typescript"]
}
```

- `/libs/packages/pkg-1/webpack.config.js`
- `/libs/packages/pkg-2/webpack.config.js` 相同，就是中间的 library 名字不同罢了

```js
const path = require('path');

module.exports = {
  mode: 'production',
  entry: path.join(__dirname, 'src/index'),
  output: {
    filename: 'index.js',
    path: path.resolve(__dirname, 'lib'),
    library: {
      name: '__youxiantest_pkg_1',
      type: 'umd',
    },
    globalObject: 'this',
  },
  resolve: {
    extensions: ['.js', '.jsx', '.ts', '.tsx', '.json'],
  },
  module: {
    rules: [
      {
        test: /\.(js|jsx|ts|tsx)$/,
        exclude: /node_modules/,
        use: 'babel-loader',
      },
    ],
  },
};
```

本质上就是配了下 babel 的编译（module.rules），ts 的扩展名（resolve.extensions），然后打包成 library（output.library）

不同的是对于 pkg-2 我们可以再多配置一个 externals，来避免将 pkg-1 的源代码也打包到 pkg-2 里面去

```js
module.exports = {
  // ...
  externals: {
    'superfree-testpkg-1': 'superfree-testpkg-1',
  },
}
```

同时记得确认一下 webpack 配置的打包结果目录和文件名要与 package.json 的 main 属性对上

### 2.6 typescript 开发配置

由于我们在开发中使用 typescirpt 了，所以我们需要稍微配置一下包映射，来避免项目间的相互引用出错，毕竟实际上是要发包之后才能看到最终的打包结果嘛

- `/libs/tsconfig.build.json`

首先我们可以先在 tsconfig.build.json 写好一些真正并且可以共用的 ts 配置

```json
{
  "compilerOptions": {
    "esModuleInterop": true,
    "jsx": "react",
    "module": "ES6",
    "sourceMap": true
  },
  "exclude": ["node_modules", "dist"]
}
```

- `/libs/tsconfig.json`

接下来则是写上真正的 ts 配置，附上多个项目的目录映射

```json
{
  "extends": "./tsconfig.build.json",
  "compilerOptions": {
    "baseUrl": "./packages",
    "paths": {
      "superfree-testpkg-1": ["pkg-1/src"],
      "superfree-testpkg-2": ["pkg-2/src"]
    },
    "moduleResolution": "Node"
  }
}
```

如此一来我们就能够在开发的时候直接体验到 ts 的类型提示，同时也是与真正打包后的类型一致的

## 3. 运行脚本配置

把两个项目的内容以及打包配置都填充好之后，下面我们加上一些运行脚本就准备打包然后发布啦

### 3.1 根目录命令配置

- `/libs/package.json`

首先我们在根目录下面的 package.json 加上一些命令

```json
{  
    "scripts": {
        "bootstrap": "lerna bootstrap",
        "clean": "rm -rf node_modules/ && lerna clean -y",
        "dev": "lerna run --stream --sort dev",
        "build": "lerna run --stream --sort build"
    }
}
```

- bootstrap 指令：安装依赖的时候我们可以使用 `lerna bootstrap` 也可以使用 `yarn install` 其实都是等价的没什么区别
- clean 指令：lerna 提供了 `lerna clean` 指令来清除 packages 下每个项目的 node_modules，但是反而没清根目录下面的hh，所以自己搞一个统一一下
- dev 指令：开发的时候我们希望每个项目都能够实时的构建并打包，不管是用 webpack-dev-server 还是 webpack --watch，然而这些对于 lerna 来说都是屏蔽的，我们使用 `lerna run dev` 来一次运行所有项目的 dev 指令，`--stream` 表示串形输出结果，`--sort` 表示由 lerna 自动识别多个项目之间的依赖关系并进行拓扑排序，最后按序执行指令
- build 指令：最后要发包之前就要进行打包，跟 dev 一样，lerna 并不关心具体的打包手法，反正就是每个项目执行一样的指令就对啦！

### 3.2 各项目命令配置

前面提到 dev 跟 build 都是依赖每个项目自己的开发/打包方式，所以我们需要进到每个项目再填充一下命令

- `/libs/packages/pkg-1/package.json`

```json
{
    "scripts": {
        "dev": "webpack --watch",
        "build": "webpack"
    },
}
```

- `/libs/packages/pkg-2/package.json`

```json
{
    "scripts": {
        "dev": "webpack --watch",
        "build": "webpack"
    },
}
```

## 4. 打包 & 提交 & 发布

最后终于进入到打包发布环节啦

### 4.1 打包

先运行一下打包指令

```bash
$ yarn build
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/lerna_launch_2_bundle.png)

确定一下打包成果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/lerna_launch_3_bundle_result.png)

### 4.2 提交到远程仓库

由于 lerna 的发布实际上是会依赖远程 git 仓库的 tag 的（当然你也可以配置成不要，但是跟着默认推荐的走嘛哈哈）

所以我们先去 github 上创建一个新的仓库，然后将当前仓库提交到远程仓库上（这里就不教学啦自己搞）

![](https://picures.oss-cn-beijing.aliyuncs.com/img/lerna_launch_4_add_remote.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/lerna_launch_5_remote_repo.png)

### 4.3 发布

最后的最后终于可以发布了！

不要忘了确认一下 yarn 源地址，还有登录信息

```bash
$ yrm ls

  npm ---- https://registry.npmjs.org/
  cnpm --- http://r.cnpmjs.org/
* taobao - https://registry.npm.taobao.org/
  nj ----- https://registry.nodejitsu.com/
  rednpm - http://registry.mirror.cqupt.edu.cn/
  npmMirror  https://skimdb.npmjs.com/registry/
  edunpm - http://registry.enpmjs.org/
  yarn --- https://registry.yarnpkg.com

$ yrm use npm   # 切换成官方源才能发布到官方仓库啦
$ yarn login
```

最后就是使用 `lerna publish` 命令发布啦

```bash
$ lerna publish
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/lerna_launch_6_publish.png)

#### 4.3.1 踩坑记录1：访问性设置

这里有个坑就是如果你的包名使用了组织作用域（如 `@youxian/xxx`），那在第一次发布的时候默认是需要加上一个参数的

```bash
$ yarn publish --access=public
```

否则会导致发布失败

所以在使用 lerna 发布的时候如果是上述情况推荐先在单个包下面单独发布一次设置可访问性 `--access=public` 后续再使用 lerna 统一管理

#### 4.3.2 踩坑记录2：已发布过标签

第二个坑是 lerna 发布的时候会添加一个 tag，并往远程仓库推送，而如果发布失败的话但是 tag 已经推送上去了，这时候我们就可以使用

```bash
$ lerna publish from-git
```

来直接使用 git 上的版本，而不需要再次更新版本号然后再提交

### 4.4 npm 查看发布结果

最后我们就可以到 npm 上看看有没有发布成功啦

![](https://picures.oss-cn-beijing.aliyuncs.com/img/lerna_launch_7_publish_result.png)

## 5. 安装 & 引用

最后我们可以再起一个小项目，然后安装自己刚发上去的包来体验一下啦！

```bash
$ mkdir usage
$ cd usage
$ yarn init -y
$ yarn add superfree-testpkg-2
$ yarn start
```

最终结果如下（addition 的输出是我修改源代码后 -> `yarn build` 重新打包 -> `lerna publish` 再发布，第一次成功后后面继续发布就又更顺手更快啦）

![](https://picures.oss-cn-beijing.aliyuncs.com/img/lerna_launch_8_usage.png)

# 结语

本篇就到此为止啦，相信跟着作者一路走到这里的小伙伴会发现，过程还是比较麻烦复杂的，一个环节出错都会导致卡很久，所以决定使用 monorepo 一定要慎重，不要为了用而用！供大家参考哈，有关于 lerna 的更好的实践方式欢迎提出，大家一起讨论学习～

# 其他资源

## 参考连接

| Title                                                                        | Link                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ---------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Yarn Workspace 使用指南                                                      | [https://blog.csdn.net/tianxintiandisheng/article/details/115329134](https://blog.csdn.net/tianxintiandisheng/article/details/115329134)                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| lerna+yarn workspace+monorepo项目的最佳实践                                  | [https://xiaoguoping.blog.csdn.net/article/details/99702447?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-6.no_search_link&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-6.no_search_link](https://xiaoguoping.blog.csdn.net/article/details/99702447?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-6.no_search_link&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-6.no_search_link) |
| Lerna                                                                        | [https://www.lernajs.cn/](https://www.lernajs.cn/)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| lerna - Github                                                               | [https://github.com/lerna/lerna](https://github.com/lerna/lerna)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| 包管理器 - peer dependency 的安装                                            | [https://blog.csdn.net/anleng6817/article/details/101126789?utm_term=yarn%E5%AE%89%E8%A3%85peerdependencies&utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~all~sobaiduweb~default-1-101126789&spm=3001.4430](https://blog.csdn.net/anleng6817/article/details/101126789?utm_term=yarn%E5%AE%89%E8%A3%85peerdependencies&utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~all~sobaiduweb~default-1-101126789&spm=3001.4430)                                                                                                                                           |
| npm 安装 yarn_npm与yarn对peerDependencies处理的差异                          | [https://blog.csdn.net/weixin_39797324/article/details/110892595?utm_term=yarn%E5%AE%89%E8%A3%85peerdependencies&utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~all~sobaiduweb~default-0-110892595&spm=3001.4430](https://blog.csdn.net/weixin_39797324/article/details/110892595?utm_term=yarn%E5%AE%89%E8%A3%85peerdependencies&utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~all~sobaiduweb~default-0-110892595&spm=3001.4430)                                                                                                                                 |
| Why babel remove lerna？ #12622                                              | [https://github.com/babel/babel/discussions/12622](https://github.com/babel/babel/discussions/12622)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| 使用Lerna、Yarn管理Monorepo项目                                              | [https://segmentfault.com/a/1190000039795228](https://segmentfault.com/a/1190000039795228)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| Lerna --多包存储管理工具（一）                                               | [https://segmentfault.com/a/1190000023954051](https://segmentfault.com/a/1190000023954051)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| lerna + workspaces使用手册                                                   | [https://segmentfault.com/a/1190000039077541](https://segmentfault.com/a/1190000039077541)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| Monorepo——大型前端项目的代码管理方式                                         | [https://segmentfault.com/a/1190000019309820?utm_source=tag-newest](https://segmentfault.com/a/1190000019309820?utm_source=tag-newest)                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| Lerna的依赖管理及hoisting浅析                                                | [https://yrq110.me/post/tool/how-lerna-manage-package-dependencies/](https://yrq110.me/post/tool/how-lerna-manage-package-dependencies/)                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| Using NPM Modules in Namespaced Typescript project                           | [https://stackoverflow.com/questions/64242513/using-npm-modules-in-namespaced-typescript-project](https://stackoverflow.com/questions/64242513/using-npm-modules-in-namespaced-typescript-project)                                                                                                                                                                                                                                                                                                                                                                                                   |
| Typescript + Webpack library generates "ReferenceError: self is not defined" | [https://stackoverflow.com/questions/64639839/typescript-webpack-library-generates-referenceerror-self-is-not-defined](https://stackoverflow.com/questions/64639839/typescript-webpack-library-generates-referenceerror-self-is-not-defined)                                                                                                                                                                                                                                                                                                                                                         |
| @lerna/filter-options - npm                                                  | [https://www.npmjs.com/package/@lerna/filter-options](https://www.npmjs.com/package/@lerna/filter-options)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| Yarn Workspaces: Organize Your Project’s Codebase Like A Pro                 | [https://www.smashingmagazine.com/2019/07/yarn-workspaces-organize-project-codebase-pro/#project-root-workspace](https://www.smashingmagazine.com/2019/07/yarn-workspaces-organize-project-codebase-pro/#project-root-workspace)                                                                                                                                                                                                                                                                                                                                                                     |
| Workspaces - yarn                                                            | [https://classic.yarnpkg.com/en/docs/workspaces/](https://classic.yarnpkg.com/en/docs/workspaces/)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| How to set up a TypeScript monorepo and make Go to definition work           | [https://medium.com/@NiGhTTraX/how-to-set-up-a-typescript-monorepo-with-lerna-c6acda7d4559](https://medium.com/@NiGhTTraX/how-to-set-up-a-typescript-monorepo-with-lerna-c6acda7d4559)                                                                                                                                                                                                                                                                                                                                                                                                               |
| lerna — JS package 管理工具                                                  | [https://medium.com/lion-f2e/lerna-js-package-%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7-e9ed360d1143](https://medium.com/lion-f2e/lerna-js-package-%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7-e9ed360d1143)                                                                                                                                                                                                                                                                                                                                                                                                     |
| lerna 使用指南                                                               | [https://www.jianshu.com/p/db3ee301af47](https://www.jianshu.com/p/db3ee301af47)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| 发布npm包时遇到的一些坑                                                      | [https://www.jianshu.com/p/40f732d91a8c](https://www.jianshu.com/p/40f732d91a8c)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| 多包依赖管理--Lerna                                                          | [https://cloud.tencent.com/developer/article/1731742](https://cloud.tencent.com/developer/article/1731742)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| externals - webpack                                                          | [https://webpack.js.org/configuration/externals/](https://webpack.js.org/configuration/externals/)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| lerna publish 失败                                                           | [https://class.imooc.com/course/qadetail/291954](https://class.imooc.com/course/qadetail/291954)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| git远程版本回退                                                              | [https://www.cnblogs.com/zjdxr-up/p/10941898.html](https://www.cnblogs.com/zjdxr-up/p/10941898.html)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| 为什么现代前端工程越来越离不开 Monorepo?                                     | [https://zhuanlan.zhihu.com/p/362228487](https://zhuanlan.zhihu.com/p/362228487)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| Rush Stack                                                                   | [https://rushstack.io/](https://rushstack.io/)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| Intro to Nx                                                                  | [https://nx.dev/latest/react/getting-started/intro](https://nx.dev/latest/react/getting-started/intro)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/others/lerna_launch](https://github.com/superfreeeee/Blog-code/tree/main/front_end/others/lerna_launch)
