# 踩坑笔记: 基于 rust-analyzer 在 vscode 中进行 rust 开发配置问题

@[TOC](文章目录)

<!-- TOC -->

- [踩坑笔记: 基于 rust-analyzer 在 vscode 中进行 rust 开发配置问题](#踩坑笔记-基于-rust-analyzer-在-vscode-中进行-rust-开发配置问题)
- [0. 项目背景](#0-项目背景)
- [1. 问题描述](#1-问题描述)
- [2. 解决](#2-解决)
- [参考连接](#参考连接)

<!-- /TOC -->

# 0. 项目背景

作为前端开发者，多多少少开始接触到使用 Rust 进行开发的模式。而选择使用什么 IDE 就非常重要了，你可以使用 CLion 进行 Rust 开发，配置上走的是 IDEA 那套。

不过我想大部分前端同学用的更多的是 vscode，今天就分享一个使用 vscode 配置 Rust 开发环境的踩坑笔记。

# 1. 问题描述

我们都知道 vscode 是基于插件的文本编辑器，而要进行 Rust 开发对应的插件就是 rust-analyzer

刚装好的时候看起来很爽，除了类型推导实在是太黑了根本看不清楚

![](https://picures.oss-cn-beijing.aliyuncs.com/img/rust_note_analyzer_settings_1_problem.png)

# 2. 解决

上网查了很久，还查到错误的 issue 被坑了一把。最后在 analyzer 的官方网站里面找到了配置说明

![](https://picures.oss-cn-beijing.aliyuncs.com/img/rust_note_analyzer_settings_2_setting.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/rust_note_analyzer_settings_3_result.png)

不得不说 Rust 社区让人最爽的地方就是在于开发工具链的丰富与极其优秀的文档支持，让人就算学习曲线陡峭还是有学下去的动力

最后附上 rust-analyzer 操作手册的传送门：[User Manual - rust-analyzer](https://rust-analyzer.github.io/manual.html)

遇到不会的就 `Cmd + F` 搜关键字就对啦

# 参考连接

| Title                                    | Link                                                                                                                                 |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| Editor Features - VScode - rust-analyzer | [https://rust-analyzer.github.io/manual.html#color-configurations](https://rust-analyzer.github.io/manual.html#color-configurations) |
