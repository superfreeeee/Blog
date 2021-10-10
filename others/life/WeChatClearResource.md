# 手把手带你清理电脑版微信冗余资源

@[TOC](文章目录)

<!-- TOC -->

- [手把手带你清理电脑版微信冗余资源](#手把手带你清理电脑版微信冗余资源)
- [正文](#正文)
  - [0. 痛点](#0-痛点)
  - [1. 检查 MacOS 系统资源占用情况](#1-检查-macos-系统资源占用情况)
  - [2. 删除电脑版微信冗余空间](#2-删除电脑版微信冗余空间)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)

<!-- /TOC -->

# 正文

真的是受够了电脑版微信

## 0. 痛点

作者使用的 mac 空间本来就比较小（128G），平常又会各方尝试各种技术，常常遇到电脑磁盘空间不足的情况。

今天带大家从查询磁盘空间占用情况的场景切入，到释放电脑版微信的资源，我们马上开始

## 1. 检查 MacOS 系统资源占用情况

相信苹果用户多多少少都有透过：`苹果图标` > `关于这台 Mac` > `存储空间` 看过自己的磁盘占用情形

![](https://picures.oss-cn-beijing.aliyuncs.com/img/wechat_clear_resource_1_mac_disk.png)

但是相信大多数人都有过疑惑是，这里的`其他`到底是个什么鬼

点击管理就会来到一个管理界面，有点像 finder，只是多了各个目录的占用空间

![](https://picures.oss-cn-beijing.aliyuncs.com/img/wechat_clear_resource_2_disk_profile.png)

这时候你会发现 `其他` 没法点，实际上我们还是可以透过 `文件` 来找到 `其他` 所占用的资源

点击 `文件` > `档案浏览器`，同时使用 `shift + command + .` 来显示隐藏目录

这时候我们来到 `~/Library/` 目录下，就能够看到占用最多空间的是 Containers（也可能是第二、第三多的）

![](https://picures.oss-cn-beijing.aliyuncs.com/img/wechat_clear_resource_3_containers.png)

这里的 Containers 其实就是 mac 电脑上关于不同应用程序所使用的其他资源，点进去就能看到各个目录分别对应的不同程序资源

有兴趣的可以自己研究研究，下面继续本篇的主题

## 2. 删除电脑版微信冗余空间

现在我们都知道电脑上的其他其实就是 `~/Library/` 目录下的一些应用程序产生的 "副作用" 的产物，当然并不是所有东西都是废物，但是确实是有些东西可以删掉也没关系的

macOS 上关于电脑版微信的资源放置在 `~/Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat` 目录下

进去之后顺着占用空间最大的目录一直想下找，就会找到叫做 `MessageTemp/` 的目录如下图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/wechat_clear_resource_4_messageTemp.png)

这时候你可能会联想到他跟聊天记录有关，没错，但是实际上看到下面的一排 `.db` 文件实际上也起了相同的作用，所以你可以选择 `MessageTemp/` 目录下几个指定聊天记录删除，也可以直接把整个 `MessageTemp/` 下所有内容删除。

即使你全删了，**实际上并不会使全部聊天记录消失！**，最多是有部分资源不可用，但是实际上大部分时候你也不会回去翻了

# 其他资源

## 参考连接

| Title                                                      | Link                                                         |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| 动手清理 Mac 微信缓存，至少帮你腾出 1GB 硬盘空间丨一日一技 | [https://sspai.com/post/42552](https://sspai.com/post/42552) |
