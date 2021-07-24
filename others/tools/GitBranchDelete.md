# Git 实战: 删除本地 & 远程分支

@[TOC](文章目录)

<!-- TOC -->

- [Git 实战: 删除本地 & 远程分支](#git-实战-删除本地--远程分支)
- [前言](#前言)
- [正文](#正文)
  - [0. 环境准备](#0-环境准备)
  - [1. 删除本地分支](#1-删除本地分支)
  - [2. 删除远程分支](#2-删除远程分支)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)

<!-- /TOC -->

# 前言

本篇的目标很简单，就是一个 git 操作的纪录，涉及的范围比较小

# 正文

我们在团队开发中总是需要建立许多的分支来满足并行开发的需求，当分支合并的时候我们可以通过 git merge 或是 git rebase + merge 来将新的功能、新的特性合并到代码当中

合并完之后如果没有再新的需求或是后续的开发，新建立的这条分支就需要废弃，这时候我们就需要用到删除分支的操作

## 0. 环境准备

首先我们先准备一个长得这样的 git 仓库，同时同步一个远程仓库如下图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/git_branch_delete_0_prepare.png)

要查看分支情况我们还能使用 `git branch`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/git_branch_delete_0_prepare2.png)

`-r` 表示远程分支、`-a` 表示全部分支

## 1. 删除本地分支

删除本地分支比较简单，使用 `git branch -d f1`(`-d` 为 `--delete` 的缩写)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/git_branch_delete_1_local.png)

## 2. 删除远程分支

接下来是删除远程分支，使用 `git push origin -d f1` 就好啦！

![](https://picures.oss-cn-beijing.aliyuncs.com/img/git_branch_delete_2_remote.png)

# 结语

git 删除分支操作，还挺简单的

# 其他资源

## 参考连接

| Title                     | Link                                                                                                     |
| ------------------------- | -------------------------------------------------------------------------------------------------------- |
| git删除远程分支和本地分支 | [https://www.cnblogs.com/luosongchao/p/3408365.html](https://www.cnblogs.com/luosongchao/p/3408365.html) |
