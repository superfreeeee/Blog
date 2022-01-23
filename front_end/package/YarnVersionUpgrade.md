# Yarn 升级: v3 都出了不要再用 yarn1 了！

@[TOC](文章目录)

<!-- TOC -->

- [Yarn 升级: v3 都出了不要再用 yarn1 了！](#yarn-升级-v3-都出了不要再用-yarn1-了)
- [Yarn 默认版本](#yarn-默认版本)
- [Yarn 升级公告](#yarn-升级公告)
- [开始升级！](#开始升级)
  - [第一步：初始化项目/现有项目升级](#第一步初始化项目现有项目升级)
  - [第二步：安装依赖](#第二步安装依赖)
  - [查看变化](#查看变化)
- [小结](#小结)
- [参考连接](#参考连接)
- [完整代码示例](#完整代码示例)

<!-- /TOC -->

# Yarn 默认版本

2202 年了 yarn 也该升级了，什么 pnpm 都跑出来要喧宾夺主了，是时候来看看我们曾经的好帮手 yarn 的改变

默认版本下安装的 yarn 都是 v1 版本的

![](https://picures.oss-cn-beijing.aliyuncs.com/img/yarn_version_upgrade_1_old.png)

# Yarn 升级公告

实际上作者本身也好久没有去看 yarn 的官网（可能根本没看过hh），yarn 的官方网站一直都有告诉我们可以升级啦！[升级传送门：Installation](https://next.yarnpkg.com/getting-started/install)

下面我们马上来体验

# 开始升级！

## 第一步：初始化项目/现有项目升级

升级成 yarn v3 有两种方式，第一种是直接在初始化的时候启用

```bash
yarn init -2
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/yarn_version_upgrade_2_init.png)

或是如果是对现有项目我们则可以使用下面命令（有些人用的是 berry 也无伤大雅

```bash
yarn set version stable
```

上面两种方式都是官方推荐的正确做法，还不赶紧回去看文档！

## 第二步：安装依赖

创建好项目之后先不急着看在干嘛，先写点代码装装看依赖先

先建立一个 ts 文件

- `/src/index.ts`

```ts
const msg: string = "Hello World";
console.log(msg);
```

然后安装一些执行 typescript 需要的依赖

```bash
yarn add ts-node typscript -D
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/yarn_version_upgrade_4_add.png)

可以看到针对指令的输出也得到升级

运行就算了，反正能跑，想试试自己把项目下下来看看

## 查看变化

最后我们来看一下 yarn 的升级带来的变化

首先看到目录结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/yarn_version_upgrade_3_structure.png)

最大的不同在于 yarn v3 不再采用原始的 node_modules 模式了，理由blablabla，给大家整理一下重点

- 查找 node_modules 运行成本高，CI/CD 部署时下载时间太久，依赖版本不稳定（依赖包更新将改变原有版本
- 多个成员间共享依赖仅靠 `yarn.lock` 是不够的

因此 yarn 提出的解决方案是根据依赖包的特征实现特定的压缩方案，将依赖变成空间极小的压缩包形式放在 `.yarn` 目录之下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/yarn_version_upgrade_5_dep.png)

本地运行 yarn 的时候再根据压缩包信息重新展开成类似 node_modules 的形式（不会真的创建该目录），在运行时提供真正的模块依赖

以后在小组多个成员之间写作的时候就不再需要重新安装依赖，甚至是直接连带依赖的压缩包统统都 push 上去！大家一起用的感觉

# 小结

yarn 的升级相对于 pnpm 是更具破坏性的改变，比起 pnpm 使用链接的方式曲线救国，虽然符合原来的 node_modules 的模式，同时支持更多新特性；yarn 直接把 node_modules 干掉了，改成能让你 push 上去的压缩包形式，非常有意思。

不过对于 yarn v3 的应用我认为多数团队可能会保持观望态度，毕竟如此破坏性的改革对于 npm 上千变万化的包带来的风险是未知的，让我们继续看下去 yarn 的成长。

---

# 参考连接

| Title                                              | Link                                                                                                                                                             |
| -------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Installation - yarn                                | [https://next.yarnpkg.com/getting-started/install](https://next.yarnpkg.com/getting-started/install)                                                             |
| Migration - yarn                                   | [https://next.yarnpkg.com/getting-started/migration](https://next.yarnpkg.com/getting-started/migration)                                                         |
| 升级Yarn 2，摆脱node_modules                       | [https://segmentfault.com/a/1190000040520326](https://segmentfault.com/a/1190000040520326)                                                                       |
| Yarn 的 Plug'n'Play 特性                           | [https://loveky.github.io/2019/02/11/yarn-pnp/](https://loveky.github.io/2019/02/11/yarn-pnp/)                                                                   |
| Cannot find module using Yarn v. 3 - stackoverflow | [https://stackoverflow.com/questions/70446257/cannot-find-module-using-yarn-v-3](https://stackoverflow.com/questions/70446257/cannot-find-module-using-yarn-v-3) |
| Yarn v2 介绍                                       | [https://zhuanlan.zhihu.com/p/107343333](https://zhuanlan.zhihu.com/p/107343333)                                                                                 |

# 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/package/yarn_version_upgrade](https://github.com/superfreeeee/Blog-code/tree/main/front_end/package/yarn_version_upgrade)
