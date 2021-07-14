# Git 实战: 利用 rebase 让你的提交/合并记录更清晰

@[TOC](文章目录)

<!-- TOC -->

- [Git 实战: 利用 rebase 让你的提交/合并记录更清晰](#git-实战-利用-rebase-让你的提交合并记录更清晰)
- [前言](#前言)
- [正文](#正文)
  - [实验一：普通 merge](#实验一普通-merge)
    - [1.1 提交记录](#11-提交记录)
    - [1.2 图解说明](#12-图解说明)
  - [实验二：rebase 应用之一 - 合并记录](#实验二rebase-应用之一---合并记录)
    - [2.1 提交记录](#21-提交记录)
    - [2.2 图解说明](#22-图解说明)
  - [实验三：rebase 应用之二 - 合并分支](#实验三rebase-应用之二---合并分支)
    - [3.1 提交记录](#31-提交记录)
    - [3.2 图解说明](#32-图解说明)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

在使用 git 进行版本管理乃至多人合作的时候，我们总是会需要合并一些新的代码。基本款我们可能会直接使用 merge 进行分支合并，但是实际上每个 merge 动作都是一个新的提交，有时候是非常破坏提交记录的。

本篇将要介绍 git rebase 的指令来帮助大家整洁 git 提交历史记录

# 正文

本篇采用三次实验的方式来说明两种 rebase 的应用

## 实验一：普通 merge

第一个实验我们先展现一个普通的 merge 场景

### 1.1 提交记录

首先是 git 详细记录

1. 创建项目 & 初始化仓库

```bash
$ mkdir test1
$ cd test1
$ git init
```

2. 首次提交

```bash
$ touch a.txt
$ git add . && git commit -m 'init'
```

3. f1 分支提交两次

```bash
$ git checkout -b f1
$ echo "b.txt by f1" > b.txt
$ git add . && git commit -m 'add b.txt'
$ touch c.txt
$ git add . && git commit -m 'add c.txt'
```

4. master 分支再提交

```bash
$ git checkout master
$ echo "b.txt by master" > b.txt
$ git add . && git commit -m 'add b.txt'
```

5. 使用 merge 将 f1 合并到 master

```bash
$ git merge f1
```

log 记录如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/git_rebase_sample1_merge.png)

### 1.2 图解说明

这时候实际上我们经过了如下的版本过程

1. 两个分支分别迭代了几个版本

![](https://picures.oss-cn-beijing.aliyuncs.com/img/git_rebase_graph1_merge1.png)

2. 在 master 分支上执行 `git merge f1` 实际上是将 master 与 f1 分别指向的最新版本内容进行合并，然后生成一条新纪录如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/git_rebase_graph1_merge2.png)

但是这样当版本、分支一多之后，非常容易产生混乱，所以这时候我们就可以使用 git rebase 方法

## 实验二：rebase 应用之一 - 合并记录

接下来我们介绍 rebase 的第一个应用：合并多条记录

### 2.1 提交记录

1. 创建项目 & 初始化仓库

```bash
$ mkdir test2
$ cd test2
$ git init
```

2. 提交多条记录

```bash
$ touch a.txt
$ git add . && git commit -m 'init'
$ touch b.txt
$ git add . && git commit -m 'add b.txt'
$ touch c.txt
$ git add . && git commit -m 'add c.txt'
$ touch d.txt
$ git add . && git commit -m 'add d.txt'
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/git_rebase_sample2_rebase_shorten1.png)

3. rebase 合并记录

```bash
$ git rebase -i HEAD~3
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/git_rebase_sample2_rebase_shorten2.png)

Hint: 每条记录的前方修改为希望执行的操作是 rebase 的全部能力，s 表示挤压，即合并记录

最后合并后的记录如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/git_rebase_sample2_rebase_shorten3.png)

三条记录被合并成一条了

### 2.2 图解说明

第二个实验实际上向我们揭露 rebase 的基础用法

一开始的提交记录如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/git_rebase_graph2_rebase_shorten1.png)

接下来我们指定 rebase 最后三条(`git rebase -i HEAD~3`)，并将前两条标记为 s 表示合并提交内容，然后就变成下面这样

![](https://picures.oss-cn-beijing.aliyuncs.com/img/git_rebase_graph2_rebase_shorten2.png)

## 实验三：rebase 应用之二 - 合并分支

第三个实验则是演示如何使用 rebase 优雅的合并分支

### 3.1 提交记录

1. 创建项目 & 初始化仓库

```bash
$ mkdir test3
$ cd test3
$ git init
```

2. 首次提交

```bash
$ touch a.txt
$ git add . && git commit -m 'init'
```

3. f2 分支提交两次

```bash
$ git checkout -b f2
$ echo "b.txt by f2" > b.txt
$ git add . && git commit -m 'add b.txt'
$ touch c.txt
$ git add . && git commit -m 'add c.txt'
```

4. master 分支再提交

```bash
$ git checkout master
$ echo "b.txt by master" > b.txt
$ git add . && git commit -m 'add b.txt'
```

5. 使用 rebase 压缩 f2 上的提交记录

```bash
$ git checkout f2
$ git rebase -i HEAD~2
```

6. 使用 rebase 修改 f2 分支的版本基础

```bash
$ git rebase master
```

这一步才是 rebase 的精华所在，透过 rebase 我们可以使当前分支的版本基础向前移动，移动到目标分支的最新版本，是我们的整个提交记录就好像线性一样

![](https://picures.oss-cn-beijing.aliyuncs.com/img/git_rebase_sample3_rebase_merge.png)

### 3.2 图解说明

一开始跟实验一一样，两个分支分别提交一些内容

![](https://picures.oss-cn-beijing.aliyuncs.com/img/git_rebase_graph3_rebase_merge1.png)

接下来我们使用前面实验二提过的特性，使用 rebase 对 f2 的分支进行压缩

![](https://picures.oss-cn-beijing.aliyuncs.com/img/git_rebase_graph3_rebase_merge2.png)

接下来使用 rebase 修改 f2 的版本基础，使其基于 master 版本的最新提交作为新的基础

![](https://picures.oss-cn-beijing.aliyuncs.com/img/git_rebase_graph3_rebase_merge3.png)

最后我们可以多一句 `git checkout master && git merge f2`，由于 f2 已经是基于 master 分支上的最新版本，所以 master 会直接移动到 f2 的最新版本上，而不会产生多余的纪录

![](https://picures.oss-cn-beijing.aliyuncs.com/img/git_rebase_graph3_rebase_merge4.png)

如此一来就能使得整个提交记录就好像线性一样

# 结语

git rebase 是一个整洁 git 历史记录的好方法，但是实际上真正的 rebase 比较记录是一个挺费功夫的操作，所以在真实的开发场景还是可以斟酌使用

# 其他资源

## 参考连接

| Title                                                   | Link                                                                                                                                                                         |
| ------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| git rebase简介(基本篇)                                  | [https://blog.csdn.net/hudashi/article/details/7664631](https://blog.csdn.net/hudashi/article/details/7664631)                                                               |
| git在工作中正确的使用方式----git rebase篇               | [https://blog.csdn.net/nrsc272420199/article/details/85555911](https://blog.csdn.net/nrsc272420199/article/details/85555911)                                                 |
| git merge和git rebase的区别, 切记：永远用rebase         | [https://zhuanlan.zhihu.com/p/75499871](https://zhuanlan.zhihu.com/p/75499871)                                                                                               |
| git仓库包含子仓库时，add报错的解决办法                  | [https://www.w3h5.com/post/471.html](https://www.w3h5.com/post/471.html)                                                                                                     |
| File Staged Content Different from HEAD - stackoverflow | [https://stackoverflow.com/questions/37953319/file-staged-content-different-from-head](https://stackoverflow.com/questions/37953319/file-staged-content-different-from-head) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/others/tools/git_rebase](https://github.com/superfreeeee/Blog-code/tree/main/others/tools/git_rebase)
