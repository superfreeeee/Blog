# 技术方案分享: gulp + webpack 联合编译三方库发布版本

@[TOC](文章目录)

<!-- TOC -->

- [技术方案分享: gulp + webpack 联合编译三方库发布版本](#技术方案分享-gulp--webpack-联合编译三方库发布版本)
- [正文](#正文)
  - [0. 打包的目标到底是什么？](#0-打包的目标到底是什么)
  - [1. 如何打包？](#1-如何打包)
  - [2. 代码示例](#2-代码示例)
    - [2.1 创建项目](#21-创建项目)
    - [2.2 安装依赖](#22-安装依赖)
    - [2.3 定义 gulp 工作流](#23-定义-gulp-工作流)
    - [2.4 ts、babel 编译配置项](#24-tsbabel-编译配置项)
    - [2.5 打包成果展示](#25-打包成果展示)
    - [2.6 webpack 打包单文件成果](#26-webpack-打包单文件成果)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

使用过 npm 写项目的同学或多或少都经历过使用某个打包工具进行打包然后发布项目的时候（可能用的是 webpack 又或是 rollup 等成为 bundler 的工程化工具）

本篇要来介绍的是关于如何编译并发布三方库，同时允许使用者基于不同的模块规范进行引用

## 0. 打包的目标到底是什么？

到底为啥要打包，我直接发原代码给别人用行不行。我跟你说，是可以的。但是问题是如果别人不用 ts 你用了 ts 怎么办；你用了 scss 而别人用的是 styled-components；别人用了 amd 的模块规范，而你用的是 ESM。种种规范使得前端的三方库越来越复杂，我们也不再能简单的发个源代码就完事了。

现在我们来看看前端各种千变万化的技术要如何进行适当的打包才能满足不同使用者的需求

1. TS / JS：第一个就是到底用不用 Typescript，简而言之就是，把东西都编译好，提供人家最原始、开箱即用的最直白的 JS 就好。最多你再提供一个 `.d.ts` 的说明文件就够了
2. Advanced CSS：现在各种预编译 CSS 框架盛行，甚至有 styled-components 这种 CSS-in-JS 开始横空出世，这时候我们一样需要把它变回原始 CSS，才让人家好用嘛
3. 模块化规范：前端模块化到了 ES6 才算是有个比较统一的标准，遗憾的是浏览器版本并不是全面支持，加上浏览器之外的 Node 采用的还是比较旧的 CommonJS 规范，这时候我们就需要透过一些编译手段，进行模块化规范的转换。也就是完成一套代码，映射到多种模块化方案的方式，供不同用户使用

## 1. 如何打包？

前面提到的几个问题本篇使用以下技术解决

1. 对于模块化方案，我们建立多条并行的 gulp 工作流，同时生成应对不同模块化规范的产物
2. 对于 TS 的编译，采用 Babel
3. 编译工作流的搭建采用 gulp，由于 webpack 属于 bundler，实际上会聚合多个模块，并不适合打包产物进行适当的 tree-shaking

## 2. 代码示例

本篇实际上是参考了某阿里开源项目的打包方式，这里提供 [传送门：a hooks library](https://ahooks.js.org/)

### 2.1 创建项目

首先一样推荐使用博主自己搭建的脚手架呵呵

```bash
$ yarn global add @youxian/cli
$ yx-cli 
```

选第三个的三方库

![](https://picures.oss-cn-beijing.aliyuncs.com/img/scheme_fe_lib_build_1_cli.png)

接下来就会看到创建的项目里面默认配置了一点 webpack 的内容

### 2.2 安装依赖

脚手架默认添加了很多依赖，不嫌麻烦我们在这里还是再列一次将要用到的几个核心依赖

- babel 相关 / TS 编译器

```bash
$ yarn add @babel/core @babel/cli @babel/preset-env @babel/preset-typescript typescript -D
```

- webpack 相关

```bash
$ yarn add webpack webpack-cli webpackbar cross-env babel-loader clean-webpack-plugin -D
```

- gulp 相关

```bash
$ yarn add gulp gulp-babel gulp-typescript -D
```

### 2.3 定义 gulp 工作流

由于我们现在要编译的是三方库，而不是把整个项目打包成单一或是简单几个文件，所以我们的核心打包工作都围绕在 gulp 上

- `gulpfile.js`

gulp 的用法就不说明了，自己查。首先先看看打包主链路

```js
exports.default = gulp.series('clean', 'cjs', 'es', 'declaration');
```

我们先后会经历：

1. clean：清理上次打包成果
2. cjs：打包 CommonJS 模块化规范结果
3. es：打包 ES Module 模块化规范结果
4. declaration：生成 `.d.ts` 声明文件

第一部分是 clean，使用了 `del` 包，简单来说就是把几个目录都删了就对了

```js
gulp.task('clean', async () => {
  await del('lib'); // cjs module
  await del('es'); // es module
  await del('dist'); // dist module
});
```

第二部分是两种模块化规范的打包，由于非常类似，我们稍微封装成配置形式，并生成任务

```js
const BASE_TS_CONFIG_PATH = './tsconfig.json';
const BASE_BABELRC_PATH = './.babelrc';
const CJS_TARGET_PATH = 'lib/';
const ES_TARGET_PATH = 'es/';

const BUILD_CONFIG = {
  CJS: {
    module: 'CommonJS',
    dest: CJS_TARGET_PATH,
  },
  ES: {
    module: 'ESNext',
    dest: ES_TARGET_PATH,
  },
};

const createTsProjectTask = (config) => {
  return () => {
    const tsProject = ts.createProject(BASE_TS_CONFIG_PATH, {
      module: config.module,
    });
    return tsProject
      .src()
      .pipe(tsProject())
      .pipe(babel({ configFile: BASE_BABELRC_PATH }))
      .pipe(gulp.dest(config.dest));
  };
};

/**
 * 打包 CommonJS 规范结果
 */
gulp.task('cjs', createTsProjectTask(BUILD_CONFIG.CJS));

/**
 * 打包 ES Module 规范
 */
gulp.task('es', createTsProjectTask(BUILD_CONFIG.ES));
```

使用 `gulp-typescript` 包，先设定配置文件名，然后加上 ts、babel 编译过程，最后生成到目标地址

第三部分则是声明文件的生成

```js
/**
 * 添加类型说明文件
 */
gulp.task('declaration', () => {
  const tsProject = ts.createProject(BASE_TS_CONFIG_PATH, {
    declaration: true,
    emitDeclarationOnly: true,
  });
  return tsProject
    .src()
    .pipe(tsProject())
    .pipe(gulp.dest(CJS_TARGET_PATH))
    .pipe(gulp.dest(ES_TARGET_PATH));
});
```

### 2.4 ts、babel 编译配置项

再来提供前面用到的两个编译配置项

- `.babelrc`

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "loose": true,
        "modules": false
      }
    ],
    "@babel/preset-typescript"
  ],
  "include": "**/*.js",
  "exclude": "**/*.ts"
}
```

使用两个 preset 进行编译，并取消了 env 的 module 编译，因为我们需要保留原始的 ES Module

- `tsconfig.json`

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@*": ["./src/*"]
    },
    "target": "ES5",
    "moduleResolution": "node",
    "allowJs": true,
    "allowSyntheticDefaultImports": true,
    "downlevelIteration": true,
    "esModuleInterop": true,
    "sourceMap": true,
    "experimentalDecorators": true,
    "lib": ["ES2015"]
  },
  "include": ["src"],
  "exclude": [
    // dependencies
    "node_modules",

    // test files
    "**/__tests__",

    // sample files
    "**/demos",

    // bundle result
    "lib",
    "es",
    "dist"
  ]
}
```

这里唯一需要注意的是不要把 `emitDeclarationOnly` 选项写进来，否则生成代码的时候就都没了

### 2.5 打包成果展示

最后我们看看打包成果

- es（ES Module 打包成果）

![](https://picures.oss-cn-beijing.aliyuncs.com/img/scheme_fe_lib_build_2_es.png)

- lib（CommonJS 打包成果）

![](https://picures.oss-cn-beijing.aliyuncs.com/img/scheme_fe_lib_build_3_lib.png)

### 2.6 webpack 打包单文件成果

最后我们还可以打包一个单文件，共 CDN 资源分发用

- `webpack.config.js` 配置文件

```js
module.exports = {
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'library.min.js',
    libraryTarget: 'umd',
    globalObject: 'this',
  },
}
```

entry 啥的其他的就自己搞吧，这里我们看看最重要的就是这个 `libraryTarget, globalObject` 这两项配好

- 打包成果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/scheme_fe_lib_build_4_dist.png)

# 其他资源

## 参考连接

| Title                                           | Link                                                                                                             |
| ----------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| Quick Start - gulp                              | [https://gulpjs.com/docs/en/getting-started/quick-start](https://gulpjs.com/docs/en/getting-started/quick-start) |
| gulp-typescript - npm                           | [https://www.npmjs.com/package/gulp-typescript](https://www.npmjs.com/package/gulp-typescript)                   |
| @babel/preset-env - babel                       | [https://babeljs.io/docs/en/babel-preset-env#loose](https://babeljs.io/docs/en/babel-preset-env#loose)           |
| Babel 6: loose mode                             | [https://2ality.com/2015/12/babel6-loose-mode.html](https://2ality.com/2015/12/babel6-loose-mode.html)           |
| Compiler Options - typescript                   | [https://www.typescriptlang.org/tsconfig#module](https://www.typescriptlang.org/tsconfig#module)                 |
| yarn cache clean - yarn                         | [https://yarnpkg.com/cli/cache/clean](https://yarnpkg.com/cli/cache/clean)                                       |
| yarn install missing files #8065 - Github issue | [https://github.com/yarnpkg/yarn/issues/8065](https://github.com/yarnpkg/yarn/issues/8065)                       |
| a hooks library                                 | [https://ahooks.js.org/](https://ahooks.js.org/)                                                                 |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/scheme/scheme_fe_lib_build](https://github.com/superfreeeee/Blog-code/tree/main/front_end/scheme/scheme_fe_lib_build)
