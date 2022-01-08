# Pnpm Workspace: 单仓库多项目(monorepo)

@[TOC](文章目录)

<!-- TOC -->

- [Pnpm Workspace: 单仓库多项目(monorepo)](#pnpm-workspace-单仓库多项目monorepo)
- [pnpm vs npm vs yarn](#pnpm-vs-npm-vs-yarn)
- [Workspace 实战（monorepo）](#workspace-实战monorepo)
  - [1. 构建项目](#1-构建项目)
  - [2. 填充项目内容](#2-填充项目内容)
  - [3. 项目配置 & 启动指令](#3-项目配置--启动指令)
  - [4. 观察 node_modules 目录结构](#4-观察-node_modules-目录结构)
- [小结](#小结)
- [参考连接](#参考连接)
- [完整代码示例](#完整代码示例)

<!-- /TOC -->

# pnpm vs npm vs yarn

- npm/yarn 采用了直接平铺的方式，而 pnpm 则是采用 `.pnpm` 隐藏目录隐藏真实的平铺结构，在使用链接(symbollink)的方式将真实安装的目录映射到 `node_modules` 下
  - 参考链接(非常推荐)：[平铺的结构不是 node_modules 的唯一实现方式](https://pnpm.io/zh/blog/2020/05/27/flat-node-modules-is-not-the-only-way)
- 天生支持 monorepo(workspace 特性，体验也比 lerna 或是 yarn workspace 好太多)

# Workspace 实战（monorepo）

## 1. 构建项目

首先我们先构建出如下的项目结构

```
/pnpm_workspace
├── package.json
├── packages
│   ├── add-one
│   │   ├── index.js
│   │   ├── package.json
│   │   └── test.test.js
│   ├── add-two
│   │   ├── index.js
│   │   ├── package.json
│   │   └── test.test.js
│   └── adder
│       ├── index.js
│       └── package.json
├── pnpm-lock.yaml
└── pnpm-workspace.yaml
```

- 构建过程

```bash
mkdir pnpm_workspace
cd pnpm_workspace
# 构建根目录项目 & 添加 workspace 配置
pnpm init -y
touch pnpm-workspace.yaml
# 构建子项目
mkdir packages
cd packages

mkdir add-one
cd adder
pnpm init -y
cd ..

mkdir add-two
cd adder
pnpm init -y
cd ..

mkdir adder
cd adder
pnpm init -y
pnpm add add-one --workspace
pnpm add add-two --workspace

cd ../..
```

## 2. 填充项目内容

本次实验主要在于体验 pnpm 的 workspace，所以项目内容尽量简单

- `/packages/add-one/index.js`

```js
const addOne = (x) => x + 1;

export default addOne;
```

- `/packages/add-two/index.js`

```js
const addTwo = (x) => x + 2;

export default addTwo;
```

- `/packages/adder/index.js`

```js
import addOne from 'add-one';
import addTwo from 'add-two';

const x = 10;
console.log(`${x} + 1 = ${addOne(x)}`);
console.log(`${x} + 2 = ${addTwo(x)}`);
```

## 3. 项目配置 & 启动指令

由于我们直接在项目中使用 ESM 的模块化方案，需要修改 `package.json` 来对 node 声明项目类型

- 在 `/packages/adder/package.json`、`/packages/add-one/package.json`、`/packages/add-two/package.json` 都加上 `type` 类型

```json
{
  // ...

  "type": "module",
  
  // ...
}
```

然后我们在 adder 项目添加启动指令

- `/packages/adder/package.json`

```json
{
  "scripts": {
    "start": "node index.js"
  },
}
```

并且在根目录下添加启动指令

- `/package.json`

```json
{
  "scripts": {
    "start": "pnpm run start --filter '*'",
  },
}
```

最后就可以在根目录下启动 adder 项目了

```bash
pnpm start
```

- 输出结果

```
> pnpm_workspace@1.0.0 start ~/pnpm_workspace
> pnpm run start --filter '*'

Scope: all 4 workspace projects
packages/adder start$ node index.js
│ 10 + 1 = 11
│ 10 + 2 = 12
└─ Done in 43ms
```

## 4. 观察 node_modules 目录结构

- 参考链接：[平铺的结构不是 node_modules 的唯一实现方式](https://pnpm.io/zh/blog/2020/05/27/flat-node-modules-is-not-the-only-way)

最后我们来观察一下 pnpm 官方介绍的新的依赖管理逻辑，首先先看到各个项目内的 `node_modules` 管理结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/pnpm_workspace_1_flat.png)

我们可以看到子项目内的依赖非常干净，装了啥就是啥，不会将嵌套的依赖库也都安装进来

接下来我们按 pnpm 官方说明的根目录下的 node_modules，以及 `.pnpm` 的隐藏目录

![](https://picures.oss-cn-beijing.aliyuncs.com/img/pnpm_workspace_2_root.png)

我们可以看到根目录下的 `node_modules/.pnpm` 保存了原始的项目平铺结构，并且其他子项目都是透过链接的方式来引用依赖包。这样做的好处这里就不一一赘述了

# 小结

pnpm 作为最新一代的 node package management 管理器，确实有其独到之处，打破以往 yarn 建立起来的平铺结构，避免了隐藏依赖的问题，非常值得大家参考与使用

---

# 参考连接

| Title                                             | Link                                                                                                                                                 |
| ------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| 工作空间 - pnpm                                   | [https://pnpm.io/zh/workspaces](https://pnpm.io/zh/workspaces)                                                                                       |
| 平铺的结构不是 node_modules 的唯一实现方式 - pnpm | [https://pnpm.io/zh/blog/2020/05/27/flat-node-modules-is-not-the-only-way](https://pnpm.io/zh/blog/2020/05/27/flat-node-modules-is-not-the-only-way) |
| package.json and file extensions - Node           | [https://nodejs.org/api/packages.html#packagejson-and-file-extensions](https://nodejs.org/api/packages.html#packagejson-and-file-extensions)         |

# 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/package/pnpm_workspace](https://github.com/superfreeeee/Blog-code/tree/main/front_end/package/pnpm_workspace)
