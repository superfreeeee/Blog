# JS 基础: 你真的了解 console 吗？

@[TOC](文章目录)

<!-- TOC -->

- [JS 基础: 你真的了解 console 吗？](#js-基础-你真的了解-console-吗)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [`console` 的方法](#console-的方法)
  - [console.log](#consolelog)
  - [console.warn](#consolewarn)
  - [console.error](#consoleerror)
  - [console.clear](#consoleclear)
  - [console.time & console.timeEnd](#consoletime--consoletimeend)
  - [console.table](#consoletable)
  - [console.count](#consolecount)
  - [console.group & console.groupEnd](#consolegroup--consolegroupend)
  - [console.log（自定义样式）](#consolelog自定义样式)
- [结语](#结语)

<!-- /TOC -->

## 简介

相信大部分的人学习任何语言或是工具都是从 `Hello World` 开始的，它代表的不仅仅是输出字符串的 demo，更是一个以最基础的实现最快了解工具运行原理的重要思想。不过这不是本篇要讨论的（笑，本篇要讨论的是 js 开发者中用得最多没有之一的 `console`。

用过 js 都知道 console 就是在控制台输出一些信息，可以是任何类型。甚至到了现在我还是会用上 console 来输出一些信息用于 debug。但是 console 除了最常用的 `log` 方法还有许多在浏览器可视化的版本，接下来就让我们来看看到底有哪些变化吧。

## 参考

<table>
  <tr>
    <td>别只用 console.log() 调试 js 代码了</td>
    <td><a href="https://mp.weixin.qq.com/s/3RKcEsXI1jSXrh0V_NY2jA">https://mp.weixin.qq.com/s/3RKcEsXI1jSXrh0V_NY2jA</a></td>
  </tr>
</table>

# 正文

## `console` 的方法

首先我们先来看看 console 实际上有哪些可用的方法



哗的就是一大票hhh，不过我们也不是全试一遍，下面列出我们将要演示的方法和用途的简要说明

| Method           | Usage            |
| ---------------- | ---------------- |
| log              | 一般输出         |
| warn             | 输出警告         |
| error            | 输出异常         |
| clear            | 清除控制台       |
| time & timeEnd   | 计时             |
| table            | 可视化键值对     |
| count            | 变量使用次数计数 |
| group & groupEnd | 输出分块         |
| log(with style)  | 自定义样式输出   |

接下来我们就一个个看看效果吧，`注意！要在浏览器输出或是支持可视化的控制台输出才有用，你在命令行那就都是一般输出`

## console.log

一般输出，没啥好说的hhh

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_console_sample_log.png)

## console.warn

输出警告（黄底样式）

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_console_sample_warn.png)

## console.error

输出异常（红底样式）

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_console_sample_error.png)

## console.clear

清理输出，清完就没了hhh，所以我也只能放两张图你自己领悟了，看能不能悟出些什么hhh

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_console_sample_clear1.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_console_sample_clear2.png)

## console.time & console.timeEnd

相当于一个计时器，开始和结束需要传入计时器的标识

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_console_sample_time&timeEnd.png)

## console.table

这大概是众多方法最令人意外的一个，做成一个表格，左边为键右边为值（数组则是以下标为键）

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_console_sample_table.png)

偷偷告诉你，他只能展开一层hhh，如果值又是一个对象就会变成 `{...}`

## console.count

计数器，记录该变量被引用的次数，值改变之后会被重置（输出格式为 `变量值:使用次数`）

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_console_sample_count.png)

## console.group & console.groupEnd

group 方法也是比较特别的一个，可以对输出做分块

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_console_sample_group&groupEnd.png)

## console.log（自定义样式）

最后一个，使用 log 方法的时候在字符串开头传入 `%c` 并将样式作为第二个参数就可以产生自定义样式的输出

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_console_sample_log_with_style.png)

# 结语

到此为止啦，console 其实还是有很多花样的，供大家参考参考（回头还是 log 最香hhhh。
