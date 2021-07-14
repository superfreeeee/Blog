# Git 实战: 利用 stash 保存当前未完成工作

@[TOC](文章目录)

<!-- TOC -->

- [Git 实战: 利用 stash 保存当前未完成工作](#git-实战-利用-stash-保存当前未完成工作)
- [前言](#前言)
  - [场景](#场景)
- [正文](#正文)
  - [1. 使用 stash 保存未完成工作](#1-使用-stash-保存未完成工作)
  - [2. 恢复修改](#2-恢复修改)
  - [3. 在不同 commit 恢复修改](#3-在不同-commit-恢复修改)
  - [4. 恢复记录产生冲突](#4-恢复记录产生冲突)
    - [4.1 冲突后记录未消失](#41-冲突后记录未消失)
  - [5. 总结](#5-总结)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

## 场景

在我们使用 git 的时候可能会有这样的一种场景：当前分支上的工作未完成，但是我们又需要立马切换到另一个分支上调整代码，这时候我们有几个选择：

- 先 commit 一次，将当前未完成的内容作为一次记录提交，之后在透过 reset 回退一次记录

然而这种用法是不太优雅的，因为使用 git 进行版本管理的初衷就是保持 commit 记录的不可变性，所以 reset 回退的操作是我们希望尽量避免的

这时候我们就可以借助 git stash 这个方法来为我们完成这个任务

# 正文

## 1. 使用 stash 保存未完成工作

stash 命令就好像将内容提交到一个特殊的队列当中作为一个特殊的 commit，但是不会添加任何记录到当前分支，我们可以透过 `git stash` 命令来完成这个操作

![](https://picures.oss-cn-beijing.aliyuncs.com/img/git_stash_sample1_untracked.png)

然而如上图所示，对于 Untracked 的纪录，stash 是看不见的(就像我们刚刚说的，stash 就好像一种特殊的 commit，一样需要先 add 才能commit)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/git_stash_sample2_list.png)

我们可以看到 add 过后就能成功 stash 保存修改了；另外所有 stash 记录就好像一个栈一样，可以保存多条记录，我们可以使用 `git stash list` 命令来查看 stash 栈

## 2. 恢复修改

`git stash` 保存记录之后，我们爱干嘛干嘛去，再回到这个分支的时候就可以使用 `git stash pop` 恢复记录(也可以使用 `git stash pop stash@{n}` 来恢复第 n 条记录)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/git_stash_sample3_pop.png)

## 3. 在不同 commit 恢复修改

当然由于 stash 本质上就是一个特殊的 commit，所以我们还可以在原分支已经存在其他提交之后再恢复记录

![](https://picures.oss-cn-beijing.aliyuncs.com/img/git_stash_sample4_pop_after_commit.png)

这个暂存的 stash 记录就好像一个浮动的 commit，恢复之后天生就自带类似 rebase 的效果

## 4. 恢复记录产生冲突

最后一种场景，就是后来 commit 的纪录与 stash 的纪录产生冲突了，这时候我们再 pop 恢复记录，git 就会自动帮我们变成类似 merge 冲突时的处理情况，改一下内容再 commit 就行了

![](https://picures.oss-cn-beijing.aliyuncs.com/img/git_stash_sample5_conflict1.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/git_stash_sample5_conflict2.png)

### 4.1 冲突后记录未消失

这里又一个小问题在于，当我们 pop 之后而产生冲突的时候，git 会认为我们还没 pop 所以没有删除记录，然而实际上未完成代码是已经恢复到当前工作区的，这时候我们就需要使用 `git stash drop`、`git stash drop stash@{n}` 来手动删除记录

## 5. 总结

最后汇总一下关于 stash 的各种指令

- `git stash` 保存未完成修改记录
- `git stash list` 查看保存记录
- `git stash pop` 恢复新的一条记录
- `git stash pop stash@{n}` 恢复指定记录
- `git stash drop` 删除最新纪录
- `git stash drop stash@{n}` 删除指定记录

# 结语

本篇介绍 git stash 的用法，是一种比起使用 commit + reset 来说更为优雅的处理方式，毕竟未完成修改本身本来就不应该归类为一个完整的 commit 提交来处理。

# 其他资源

## 参考连接

| Title                                                             | Link                                                                                                 |
| ----------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| 【狀況題】手邊的工作做到一半，臨時要切換到別的任務                | [https://gitbook.tw/chapters/faq/stash.html](https://gitbook.tw/chapters/faq/stash.html)             |
| git stash pop 冲突，git stash list 中的记录不会自动删除的解决方法 | [https://www.cnblogs.com/wshiqtb/p/7100248.html](https://www.cnblogs.com/wshiqtb/p/7100248.html)     |
| 记录一次git stash找回删除的存储                                   | [https://www.cnblogs.com/shuhaonb/p/12530107.html](https://www.cnblogs.com/shuhaonb/p/12530107.html) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/others/tools/git_stash](https://github.com/superfreeeee/Blog-code/tree/main/others/tools/git_stash)
