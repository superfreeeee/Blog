# 前端工作流: 自动化 Code Lint 使你的项目代码更规范

@[TOC](文章目录)

<!-- TOC -->

- [前端工作流: 自动化 Code Lint 使你的项目代码更规范](#前端工作流-自动化-code-lint-使你的项目代码更规范)
- [前言](#前言)
- [正文](#正文)
  - [0. 使用工具项介绍](#0-使用工具项介绍)
  - [1. Prettier 代码格式化](#1-prettier-代码格式化)
  - [2. Commitlint 提交信息校验](#2-commitlint-提交信息校验)
  - [3. ESLint 代码校验(可以覆盖TS！)](#3-eslint-代码校验可以覆盖ts)
  - [4. Stylelint 样式表校验](#4-stylelint-样式表校验)
  - [5. Husky 接入 Git Hooks](#5-husky-接入-git-hooks)
  - [6. lint-staged 提交前校验](#6-lint-staged-提交前校验)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

Code Lint 翻译来说叫做静态代码分析校验blablabla，简单来说就是 **代码规范**！

这里说到的规范不只是 javascript 上的规范，同时我们也可以对 css、json 甚至对 commit 的 message 都进行校验，所以主题取名叫 **Code Lint** 而不只是单纯的 eslint

# 正文

## 0. 使用工具项介绍

首先我们先来大致浏览一下整个工作流的代码校验全部会用到哪些工具：

- `prettier` 代码格式化
- `commitlint` 校验 commit 信息
- `eslint` js、ts 代码校验
- `stylelint` css、less、scss 样式校验
- `husky` 接入 git hooks 使用
- `lint-staged` 对 staged 阶段修改进行校验

接下来我们从一个全新的项目开始一步步的配置各个依赖项

## 1. Prettier 代码格式化

[Prettier 官方文档](https://prettier.io/)

第一个是进行代码格式化用的 prettier

- 安装依赖

```bash
$ yarn add prettier -D
```

- 配置文件：根目录下新建 `.prettierrc.json`

```json
{
  "singleQuote": true,
  "semi": true
}
```

然后我们创建 `src/index.ts` 并输入一小段代码，并使用 `yarn prettier src/* --write` 命令看看格式化的效果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/code_lint_1_prettier1.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/code_lint_1_prettier2.png)

在 Vscode 环境下可以直接安装 Prettier 插件

![](https://picures.oss-cn-beijing.aliyuncs.com/img/code_lint_vscode_plugin_1_prettier.png)

这样就可以在不运行命令的情况下使用快捷键或是本地配置自动格式化

## 2. Commitlint 提交信息校验

[commitlint 官方文档](https://commitlint.js.org/)

commitlint 则是用于校验 git commit 的时候输入的提交信息

- 安装依赖

```bash
$ yarn add @commitlint/{cli,config-conventional} -D
```

- 配置文件：`commitlint.config.js`

```js
module.exports = {
  extends: ['@commitlint/config-conventional'],
};
```

- 运行示例

![](https://picures.oss-cn-beijing.aliyuncs.com/img/code_lint_2_commitlint1.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/code_lint_2_commitlint2.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/code_lint_2_commitlint3.png)

这里我们直接使用 `echo` 命令并通过管道传给 commitlint，后面我们会集成到 husky 定义的 git hooks 里面

## 3. ESLint 代码校验(可以覆盖TS！)

[ESLint 官方文档](https://eslint.org/)

大名鼎鼎的 eslint 相信大家都不陌生，就是校验 js 代码的；你可能会听说 tslint 对应于 ts 的代码校验，但是实际上 eslint 已经同时能够兼容 ts 代码的校验，而且 tslint 已经宣布停止更新，并向 eslint 合并为新的方向

- 安装依赖

```bash
$ yarn add eslint -D
```

- 初始化配置：生成 `.eslintrc.json`

```bash
$ yarn eslint --init
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/code_lint_3_eslint1.png)

由于本篇使用的是 `yarn` 作为包管理器，所以在自动安装依赖的部分选择了 No，因此要自己在下面手动安装相关依赖(记得加 typescript 提供 ts 默认编译器避免报错)

```bash
$ yarn add eslint-plugin-react@latest @typescript-eslint/eslint-plugin@latest @typescript-eslint/parser@latest typescript -D
```

最后我们在 `/src/index.ts` 里面加一句会报错的语句

```ts
const s = 'Hello World';
```

然后我们就可以运行命令看看效果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/code_lint_3_eslint2.png)

将错误或是警告改正之后重新校验就通过了

![](https://picures.oss-cn-beijing.aliyuncs.com/img/code_lint_3_eslint3.png)

在 Vscode 环境下可以直接安装 ESLint 插件

![](https://picures.oss-cn-beijing.aliyuncs.com/img/code_lint_vscode_plugin_2_eslint.png)

重新打开 vscode 之后就会直接对项目当前代码进行校验，而不用等到执行命令之后才校验

## 4. Stylelint 样式表校验

[stylelint 官方文档](https://stylelint.io/)

有了 eslint 我们自然也会联想到，那是不是 css 相关的样式代码也有 lint 校验，没错它就是 **stylelint**

- 安装依赖

```bash
$ yarn add stylelint stylelint-config-standard -D
```

- 配置文件：`.stylelintrc.json`

```json
{
  "extends": "stylelint-config-standard"
}
```

- 运行示例

![](https://picures.oss-cn-beijing.aliyuncs.com/img/code_lint_4_stylelint1.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/code_lint_4_stylelint2.png)

一样地 stylelint 在 vscode 环境下也有对应的插件

![](https://picures.oss-cn-beijing.aliyuncs.com/img/code_lint_vscode_plugin_3_stylelint.png)

## 5. Husky 接入 Git Hooks

[Husky 官方文档](https://typicode.github.io/husky/)

前面我们介绍了：

- ESLint：js, ts 代码的校验
- stylelint：css, less, scss 的校验
- commitlint：commit message 的校验

下面我们就要将整个工作流接在一起，核心的技术在于利用了 **Git Hook** 的特性，在指定的生命周期钩子插入相应的校验规则，并在失败的时候阻断提交或是其他下一步的动作

- 安装依赖

```bash
$ yarn add husky -D
```

- 初始化 & 创建 `commit-msg` 钩子加入 commitlint 校验

```bash
$ yarn husky install
$ yarn husky add .husky/commit-msg "yarn commitlint --edit \$1"
```

在这里我们先加入 commitlint 的校验，通过 `add .husky/commit-msg` 来向 commit-msg 钩子插入校验脚本 `yarn commitlint --edit $1`

看看效果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/code_lint_5_husky.png)

我们可以看到提交信息写 `init` 的时候会失败，而改成 `build: init project` 就能成功提交了

## 6. lint-staged 提交前校验

[lint-staged - Github](https://github.com/okonet/lint-staged)

你可能注意到我们没有将代码相关的校验直接使用 husky 加入 git hook 当中，这是因为每次提交都对整个项目进行校验是没有效率的，所以我们要再加入一个 lint-staged 工具，使我们可以只针对 staged 阶段(也就是 `git add .` 之后的部分)的修改进行校验

- 安装依赖

```bash
$ yarn add lint-staged -D
```

- 创建 `pre-commit` 钩子执行 lint-staged

```bash
$ yarn husky add .husky/pre-commit "yarn lint-staged"
```

- `package.json` 中配置校验规则

```json
{
  "lint-staged": {
    "*.json": [
      "prettier --write"
    ],
    "*.{js,ts}": [
      "prettier --write",
      "yarn eslint --fix"
    ],
    "*.{css,less,scss}": [
      "prettier --write",
      "yarn stylelint --fix"
    ]
  },
}
```

上述配置完成之后我们稍微修改 ts 跟 scss 文件来测试有没有正确进行格式化并对代码进行校验了

![](https://picures.oss-cn-beijing.aliyuncs.com/img/code_lint_6_lint_staged.png)

# 结语

本篇一次加入了几乎前端可能会用上的全面的代码校验，几乎能够覆盖所有类型的文件，供大家参考。有兴趣的可以到完整代码示例里面查看完整的项目配置

# 其他资源

## 参考连接

| Title                                                                             | Link                                                                                                                                               |
| --------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| 前端codeLint-- 为项目集成ESLint、StyleLint、commitLint实战和原理                  | [https://zhuanlan.zhihu.com/p/100427908](https://zhuanlan.zhihu.com/p/100427908)                                                                   |
| 使用husky, prettier, lint-stage在代码提交时自动格式化                             | [https://blog.csdn.net/qq_41725450/article/details/95319428](https://blog.csdn.net/qq_41725450/article/details/95319428)                           |
| Prettier 官方                                                                     | [https://prettier.io/](https://prettier.io/)                                                                                                       |
| commitlint 官方                                                                   | [https://commitlint.js.org/](https://commitlint.js.org/)                                                                                           |
| Commitlint 使用总结                                                               | [https://blog.csdn.net/qq_38290251/article/details/111646491](https://blog.csdn.net/qq_38290251/article/details/111646491)                         |
| ESLint 官方                                                                       | [https://eslint.org/](https://eslint.org/)                                                                                                         |
| TypeScript如何使用ESLint进行代码检测                                              | [https://www.jianshu.com/p/c4fda089a981](https://www.jianshu.com/p/c4fda089a981)                                                                   |
| Warning: React version not specified in eslint-plugin-react settings.             | [https://blog.csdn.net/qq_36658051/article/details/106856236](https://blog.csdn.net/qq_36658051/article/details/106856236)                         |
| stylelint 官方                                                                    | [https://stylelint.io/](https://stylelint.io/)                                                                                                     |
| Husky 官方                                                                        | [https://typicode.github.io/husky/](https://typicode.github.io/husky/)                                                                             |
| GitHook 工具 —— husky介绍及使用                                                   | [https://www.cnblogs.com/jiaoshou/p/12222665.html](https://www.cnblogs.com/jiaoshou/p/12222665.html)                                               |
| typicode/husky - Github                                                           | [https://github.com/typicode/husky/tree/main](https://github.com/typicode/husky/tree/main)                                                         |
| 升级husky5实践                                                                    | [https://zhuanlan.zhihu.com/p/356924268](https://zhuanlan.zhihu.com/p/356924268)                                                                   |
| husky6版本+commitlint使用与脚本全面分析（husky v4升级v6变化巨大）                 | [https://blog.csdn.net/qq_21567385/article/details/116429214](https://blog.csdn.net/qq_21567385/article/details/116429214)                         |
| sh: husky: command not found                                                      | [https://stackoverflow.com/questions/67063993/sh-husky-command-not-found](https://stackoverflow.com/questions/67063993/sh-husky-command-not-found) |
| husky配置 =＞ git 日志提交规范限制, eslint检查                                    | [https://blog.csdn.net/jiandan1127/article/details/108388328](https://blog.csdn.net/jiandan1127/article/details/108388328)                         |
| lint-staged - Github                                                              | [https://github.com/okonet/lint-staged](https://github.com/okonet/lint-staged)                                                                     |
| lint-staged 使用教程                                                              | [https://www.cnblogs.com/jiaoshou/p/12250278.html](https://www.cnblogs.com/jiaoshou/p/12250278.html)                                               |
| \[前端采坑\]lint-staged就是匹配不到文件                                           | [https://zhuanlan.zhihu.com/p/102104085](https://zhuanlan.zhihu.com/p/102104085)                                                                   |
| Problems loading reference : Unable to load schema from : Unable to connect...... | [https://blog.csdn.net/qq_38918953/article/details/106518204](https://blog.csdn.net/qq_38918953/article/details/106518204)                         |
| vscode显示.git                                                                    | [https://www.cnblogs.com/amiezhang/p/13446524.html](https://www.cnblogs.com/amiezhang/p/13446524.html)                                             |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/others/code_lint](https://github.com/superfreeeee/Blog-code/tree/main/front_end/others/code_lint)
