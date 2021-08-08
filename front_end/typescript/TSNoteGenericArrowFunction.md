# TS 踩坑笔记: 箭头函数添加泛型报错(Error: JSX element 'T' has no corresponding closing tag.ts(17008))

@[TOC](文章目录)

<!-- TOC -->

- [踩坑笔记: (Error: )](#踩坑笔记-error-)
- [前言](#前言)
- [正文](#正文)
  - [项目背景](#项目背景)
  - [问题描述](#问题描述)
  - [解决方案](#解决方案)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

今天给大家分享一个在 React 项目中使用 TypeScript 遇到的错误

# 正文

## 项目背景

React + TS 的项目配置，项目中关于 React 组件的使用 .tsx 后缀，其他单纯的文件使用 .ts 后缀

## 问题描述

在 React 组件附近定义泛型的箭头函数时产生 TS 报错警告，原本以为是语法写错了

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_note_generic_arrow_function_2_tsx.png)

但是实际上在 .ts 文件中是正常解析的，也就是说并不是语法问题

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_note_generic_arrow_function_1_ts.png)

## 解决方案：加逗号

最后发现其实是因为泛型的语法与 JSX 的语法冲突，导致 TS 解析成 JSX 而产生 unexpected token 的问题

其实解决方案很简单

- 一种是就不要写在 .tsx 文件里面就不会报错
- （推荐）第二种就是在后面加一个逗号就能正确解析了

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_note_generic_arrow_function_3_resolve.png)

# 结语

TS 某些语法和解析规则藏的比较深，直接去看最表面的文档有时候可能比较难发觉到问题，主要就是要多用多会

# 其他资源

## 参考连接

| Title                                        | Link                                                                                                                                                                                                               |
| -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 具有泛型的 Typescript 箭头函数的语法是什么？ | [https://qastack.cn/programming/32308370/what-is-the-syntax-for-typescript-arrow-functions-with-generics](https://qastack.cn/programming/32308370/what-is-the-syntax-for-typescript-arrow-functions-with-generics) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/typescript/ts_note_generic_arrow_function](https://github.com/superfreeeee/Blog-code/tree/main/front_end/typescript/ts_note_generic_arrow_function)
