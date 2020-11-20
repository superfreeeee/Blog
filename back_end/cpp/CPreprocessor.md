# C: Preprocessor 预处理器

@[TOC](文章目录)

<!-- TOC -->

- [C: Preprocessor 预处理器](#c-preprocessor-预处理器)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [什么是预处理？](#什么是预处理)
  - [主要功能](#主要功能)
    - [文件引入](#文件引入)
    - [宏定义](#宏定义)
    - [条件编译](#条件编译)
  - [应用](#应用)
    - [确保唯一引入头文件](#确保唯一引入头文件)
    - [特定运行模式](#特定运行模式)
    - [全局变量声明](#全局变量声明)
- [结语](#结语)

<!-- /TOC -->

## 简介

C 语言是一个古老却又屹立不摇的高级语言，它在计算机世界拥有不可动摇的地位。C/C++ 语言源文件生成可执行文件的大致过程如下：

![](https://picures.oss-cn-beijing.aliyuncs.com/img/c_compile_progress.png)

本篇将要来介绍在`预处理(C-Preprocessor)`阶段会使用到的代码以及相关写法和用法。

## 参考

<table>
  <tr>
    <td>C 预处理器</td>
    <td><a href="https://www.runoob.com/cprogramming/c-preprocessors.html">https://www.runoob.com/cprogramming/c-preprocessors.html</a></td>
  </tr>
  <tr>
    <td>C语言真正的编译过程</td>
    <td><a href="https://www.cnblogs.com/wuyouxiaocai/p/5701088.html">https://www.cnblogs.com/wuyouxiaocai/p/5701088.html</a></td>
  </tr>
  <tr>
    <td>C代码变成可执行文件的过程</td>
    <td><a href="https://blog.csdn.net/u012184539/article/details/81348529">https://blog.csdn.net/u012184539/article/details/81348529</a></td>
  </tr>
</table>

# 正文

## 什么是预处理？

`CPP = C Preprocessor(C 预处理器)`，嗯看名字看起来好棒棒，其实说白了就是一个文本替换工具(宏定义`#define`的部分)，当然其实他还具备了一些其他功能像是负责声明文件引入(`include`)、条件编译(`#if`、`#ifdef`)等。后面两种功能虽然不是简单直接的文本替换，但是引入文件或条件编译等在预编译阶段只是替换成另一种字符串并引入必要的变量等，后序我们可以看到详细内容

## 主要功能

透过上面的介绍，我们可以总结出预处理阶段大概能够分为三大类方法：

- 文件引入
- 宏定义
- 条件编译
- \*编译选项

第四项的编译选项主要指针对编译期间对编译器的特殊命令，相关的预处理指令为 `#pragma` 本篇不做讨论。

接下来我们可以根据三种分类罗列出接下来要介绍的所有预处理指令：

| 分类     | 指令                                                   |
| -------- | ------------------------------------------------------ |
| 文件引入 | `#include`                                             |
| 宏定义   | `#define`、`#undef`                                    |
| 条件编译 | `#if`、`#ifdef`、`#ifndef`、`#elif`、`#else`、`#endif` |

### 文件引入

有点类似其他语言的`模块化(module)系统`，在 C 语言当中并没有真正的模块化(在 C++ 中启用了 `namespace`)，虽然我们可以透过 `#include` 指令引入外部的文件，不过编译后将会合并成一个唯一的二进制文件(静态编译的情况下)。

不过这不妨碍我们对项目结构的划分，我们可以根据项目的特性将不同的变量、函数定义在不同的文件中，使用 `#include` 指令来引入即可

> 语法规则

```c
#include <头文件名/文件名>
// 或
#include "头文件名/文件名"
```

> 示例

- `print.c`

```c
#include <stdio.h>

void println(char* s) {
    printf("%s\n", s);
}
```

- `main.c`

```c
#include "print.c"

int main() {
    println("Hello World!");
}
```

- 运行结果

```bash
$ gcc main.c -o main
$ ./main
Hello World!

```

> 使用惯例

由于我们可以使用尖括号(`<>`)也可以选择使用双引号(`""`)来引入外部文件，照惯例我们通常在引入内置库(built-in)时使用尖括号，而在引入自定义的文件时使用双引号，例如：

```c
// 引入内置库 stdio.h
#include<stdio.h>
// 引入自定义头文件 print.h
#include "print.h"
```

### 宏定义

宏定义的指令有两种 `#define`、`#undef`，顾名思义一个就是定义，一个是取消定义。值得一提的是这边的`定义`其实是一种文本替换，相当于对源文件内容的直接替换，相关的错误可能在编译甚至运行时才会发现

> 语法规则

`#undef` 相对简单，它的作用是取消某个宏的定义；而 `#define` 则有简单的文本替换，也有参数化的宏定义：

```c
// 取消宏定义
#undef 宏

// 一般宏定义
#define 宏 替换文本

// 参数化定义
#define 宏函数(参数列表) 表达式
```

虽然宏定义也可以带参数，但是实际的运行效果就只是相对应的文本替换而已，上代码！

> 示例

- `main.c`

```c
#include <stdio.h>

#define N 10 // 定义常量
#ifdef N
    #undef N // 取消宏 N
    #define N 20 // 重新定义宏
#endif

#define A 3
#define B 4
#define max(a, b) (a >= b ? a : b)

int main() {
    printf("N: %d\n", N);
    printf("max(%d, %d): %d\n", A, B, max(A, B));
}
```

这边我们使用了常量 `N`、`max(a, b)` 函数等，但实际执行时并不存在名叫 max 的函数也没有名为 N 的变量，而是会先被预编译(直接文本替换)

```c
// 预编译前
printf("N: %d\n", N);
printf("max(%d, %d): %d\n", A, B, max(A, B));

// 预编译后
printf("N: %d\n", 20);
printf("max(%d, %d): %d\n", 3, 4, (3 >= 4 ? 3 : 4));
```

也就是说使用 `#define` 定义的常量会直接被替换成立即数或是其他符号(透过文本替换)，宏函数也是直接使用表达式替换

> 使用惯例

从上述范例我们看出使用 `#define` 的都是直接的文本替换，所以如果想用来定义变量或是函数需要格外小心：

- `#define var` 只用来定义常量，不能定义变量 -> 将直接替换成字面量
- `#define func(params)` 只用来定义简单逻辑或是非常简单的表达式
  - 定义布尔表达式时要用圆括号包起来，避免出现替换后逻辑错误，如下范例 2.1
  - 定义函数应该短而简单，否则定义成一般函数运行时效率更好

- 范例 2.1

```c
// 没有将布尔表达式包起来，造成逻辑错误
#include <stdio.h>
#define aboveOrEqual(a, b) a == b || a > b

int main() {
    int a = 1, b = 2, c = !aboveOrEqual(a, b);
    printf("a = %d, b = %d, c = %d\n", a, b, c);
}

// output:
// a = 1, b = 2, c = 0
```

说明：文本替换后表达式为 `c = !a == b || a > b` 而预期逻辑应为 `c = !(a == b || a > b)`

### 条件编译

条件编译都必须成对出现，由 `#if`、`#ifdef`、`#ifndef` 开始，由 `#endif` 结束。被这两句夹在中间的表达式就会被选择性的编译

> 语法

```c
// 直接条件
#if condition
#elif condition2 // 可选
#else            // 可选
#endif

// 检查宏是否被定义
#ifdef 宏 // 如果宏被定义就编译
#endif

#ifndef 宏 // 如果宏没有被定义就编译
#endif
```

> 示例

- `main1.c`

```c
#include <stdio.h>

#define DEBUG
#ifdef DEBUG
    #define N 10
#else
    #define N 20
#endif

int main() {
    printf("N = %d\n", N);
}
```

执行结果：

```bash
$ gcc main1.c -o main1
$ ./main1
N = 10
```

- `main2.c`

```c
#include <stdio.h>

// 注释掉 DEBUG 的宏定义
// #define DEBUG
#ifdef DEBUG
    #define N 10
#else
    #define N 20
#endif

int main() {
    printf("N = %d\n", N);
}
```

执行结果：

```bash
$ gcc main2.c -o main2
$ ./main2
N = 20
```

## 应用

以上为预编译的三种基本方法的用法，接下来介绍一些宏定义的常见用法。

### 确保唯一引入头文件

当我们的项目结构愈来愈复杂的时候，有时候我们可能在不同地方重复引入同一个文件，或是间接的接收到已经引入的文件而产生重复引入的问题。当文件引入过一次之后所有变量，宏定义等其实都已经存在了，所以为了`确保一个文件的内容只被加载一次`，我们可以透过宏定义来解决这个问题：

> 模版

```c
// 某头文件
#ifndef <文件名>_H
#define <文件名>_H

// ... 文件内容

#endif /* <文件名>_H */
```

> 示例

- `test.h`

```
#ifndef TEST_H
#define TEST_H

// ... 文件内容

#endif
```

说明：透过使用 `#ifndef TEST_H`，当我们重复引入相同头文件时就会跳过整个文件的内容，而直接使用第一次引用时引入的内容。

### 特定运行模式

有时候我们可能需要根据不同的运行模式进行相应的配置，这时我们就可以使用条件编译的效果来区分不同的运行模式：

> 模版

```c
// 两种模式：A、B
#ifdef <模式A>
    // 模式 A 的配置
#else
    // 模式 B 的配置
#endif
```

```c
// 多种模式
#if defined(<模式 A>)
    // 模式 A 的配置
#elif defined(<模式 B>)
    // 模式 B 的配置
#elif defined(<模式 C>)
    // 模式 C 的配置
#else
    // 默认模式配置
#endif
```

> 示例

- `hello.h`

```c
#include <stdio.h>

#ifdef DEBUG

// Debug 模式下的函数
void hello() {
    printf("Hello World! (Debug mode)\n");
}

#else

// 一般模式下的函数
void hello() {
    printf("Hello World!\n");
}

#endif
```

- `normal.c`

```c
#include "hello.h"

int main() {
    hello();
}
```

- `debug.c`

```c
#define DEBUG

#include "hello.h"

int main() {
    hello();
}
```

执行结果：

```bash
# 一般模式下的输出
$ gcc normal.c -o normal
$ ./normal
Hello World!

# Debug 模式下的输出
$ gcc debug.c -o debug
$ ./debug
Hello World! (Debug mode)
```

### 全局变量声明

使用多文件甚至汇编文件联合编译时，我们可能需要在头文件中唯一声明全局变量，我们可以使用宏定义重载 `EXTERN` 关键字，并额外定义一个声明全局变量的宏，使得全局变量在 `.c` 文件中唯一声明之后，其他文件看到的 `.h` 中都引用 `extern` 关键字

> 模版

- `const.h`

```c
#define EXTERN extern
```

- `global.h`

```c
#ifdef GLOBAL_VARIABLES_HERE
    #undef EXTERN
    #define EXTERN
#endif

// 要声明的全局变量
EXTERN int N;
```

- `global.c`

```c
// 在 .c 文件加上该宏定义进行唯一的声明
#define GLOBAL_VARIABLES_HERE

#include "const.h"
#include "global.h"
```

- `main.c`

```c
#include <stdio.h>
#include "const.h" // 引入全局 extern
#include "global.h" // 引入全局变量

int main() {
    printf("N = %d\n", N); // 访问
    N = 100;               // 修改值
    printf("N = %d\n", N);
}
```

# 结语

本篇介绍了`CPP(C Preprocessor)预处理器`的基础用法和几个简单的应用，在一般开发中三个基本用法都蛮长用到的：

- `#include` 用于插入外部文件
- `#define` 定义标识(模式、文件模块)、常量以及布尔表达式(简单函数)
- 条件编译可根据条件静态调整编译内容

以上用法供大家参考，有错误欢迎指正。
