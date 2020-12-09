# C 进阶: Var Args 可变参数

@[TOC](文章目录)

<!-- TOC -->

- [C 进阶: Var Args 可变参数](#c-进阶-var-args-可变参数)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [固定参数函数](#固定参数函数)
  - [标准库 <stdarg.h>](#标准库-stdargh)
  - [可变参数函数实现](#可变参数函数实现)
- [结语](#结语)

<!-- /TOC -->

## 简介

学习过 C 语言肯定写过 `printf("Hello Wolrd!");` 这句，其中使用的 `printf` 便是一种接受可变长参数的函数。我们都知道只要在第一个参数的字符串里面加入 `%d`、`%s`、... 或其他`标签(%开头)`，然后后面接着传入对应的参数就行了。

现在我也想自己写一个可传入任意长度参数的函数要怎么做呢？接下来我们就来介绍 `stdarg.h` 为我们带来的方法。

## 参考

<table>
  <tr>
    <td>C 可变参数</td>
    <td><a href="https://www.runoob.com/cprogramming/c-variable-arguments.html">https://www.runoob.com/cprogramming/c-variable-arguments.html</a></td>
  </tr>
</table>

# 正文

## 固定参数函数

一般定义函数的时候我们都要显示的写出接受哪些参数：

```c
int max(int a, int b) {
    return a > b ? a : b;
}

int min(int a, int b) {
    return a < b ? a : b;
}
```

这两个函数都分别接受两个参数，如果我们想接受多个参数难道要这样声明吗？

```c
int max(int a, int b);
int max(int a, int b, int c);
int max(int a, int b, int c, int d);
int max(int a, int b, int c, int d, int e);
int max(int a, int b, int c, int d, int e, int f);
// ...
```

嗯...好像不太可能，所以接下来我们就要请出 `<stdarg.h>` 这个标准库来满足我们的需求。

## 标准库 <stdarg.h>

`<stdarg.h>` 这个库提供了我们实现可变长参数的方法，接下来我们会用到的重要类型和方法如下：

- `va_list`：保存变长参数的类型
- `va_start(va_list, int)`：初始化变长参数（获取传入的参数）
- `va_arg(va_list, type)`：指定下一个参数类型并取出
- `...`：参数声明为 `...` 表示后面传入数量不定的参数

> 示例

```c
#include <stdio.h>
#include <stdarg.h>

void print_ints(int argc, ...) {
    va_list ints; // 声明保存可变参数的变量
    va_start(ints, argc); // 初始化参数（获取参数）
    // 根据 argc 打印对应数量的参数
    for (int i=0 ; i<argc ; i++) {
        // 每次使用 va_arg(ints, int) 从 ints 取出一个 int 类型的参数
        printf("ints[%d]: %d\n", i, va_arg(ints, int));
    }
}

int main() {
    // 第一个参数为 argc: 表示后面接着三个参数
    // 1, 2, 3 在函数体内被放入 ints
    print_ints(3, 1, 2, 3);
}
```

```
ints[0]: 1
ints[1]: 2
ints[2]: 3
```

## 可变参数函数实现

接下来我们实现四个可变参数的函数：`sum`、`max`、`min`、`print`

```c
#include <stdio.h>
#include <stdarg.h>
#include <string.h>

// 总和
int sum(int argc, ...) {
    va_list nums;
    va_start(nums, argc);
    int res = va_arg(nums, int);
    for (int i = 1; i < argc; i++) {
        res += va_arg(nums, int);
    }
    return res;
}

// 最大值
int max(int argc, ...) {
    va_list nums;
    va_start(nums, argc);
    int res = va_arg(nums, int);
    for (int i = 1; i < argc; i++) {
        int num = va_arg(nums, int);
        if (num > res) res = num;
    }
    return res;
}

// 最小值
int min(int argc, ...) {
    va_list nums;
    va_start(nums, argc);
    int res = va_arg(nums, int);
    for (int i = 1; i < argc; i++) {
        int num = va_arg(nums, int);
        if (num < res) res = num;
    }
    return res;
}

void print(char *format, ...) {
    va_list vars;
    int len = strlen(format);
    // 以模版字符串的长度初始化参数列表
    va_start(vars, len);
    for (int i=0; i<len; i++) {
        char c = *(format + i);
        // 根据模版字符串决定下一个取出变量的类型
        switch (c) {
            case 'i':
                printf("param %d is int: %d\n", i, va_arg(vars, int));
                break;
            case 'f':
                printf("param %d is double: %f\n", i, va_arg(vars, double));
                break;
        }
    }
}

int main() {
    printf("sum(1, 2, 3, 4, 5) = %d\n", sum(5, 1, 2, 3, 4, 5));
    printf("max(1, 2, 3, 4, 5) = %d\n", max(5, 1, 2, 3, 4, 5));
    printf("min(1, 2, 3, 4, 5) = %d\n", min(5, 1, 2, 3, 4, 5));
    print("ifif", 2, 3.5, 5, 6.5);
}
```

```
sum(1, 2, 3, 4, 5) = 15
max(1, 2, 3, 4, 5) = 5
min(1, 2, 3, 4, 5) = 1
param 0 is int: 2
param 1 is double: 3.500000
param 2 is int: 5
param 3 is double: 6.500000
```

# 结语

到此完结啦，就这么简单。整个可变参数使用的流程就是：一个 `va_list` 保存参数，透过 `va_start` 初始化参数列表，再使用 `va_arg` 依序取出参数，搞定！
