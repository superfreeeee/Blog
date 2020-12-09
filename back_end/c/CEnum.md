# C 基础: Enum 枚举类型

@[TOC](文章目录)

<!-- TOC -->

- [C 基础: Enum 枚举类型](#c-基础-enum-枚举类型)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [枚举类型语法](#枚举类型语法)
  - [非连续值](#非连续值)
- [结语](#结语)

<!-- /TOC -->

## 简介

今天来说说 C 语言里面的枚举类型。在程序中常常会需要对一些现实生活的属性进行枚举，如性别通常不是男就是女、一周七天、进程状态定义等。

第一种做法我们可以透过 `#define` 进行宏定义：

```c
#define Gender int
#define MALE 0
#define FEMALE 1

#define Day int
#define MON 1
#define TUE 2
#define WED 3
#define THU 4
#define FRI 5
#define SAT 6
#define SUN 7

#define ProcessState int
#define RUNNING 1
#define READY   2
#define BLOCK   3
```

但是这样写既冗长又容易出现手误，有时候我们也不关心这个状态实际代表的值是多少，仅仅是为了区分不同状态即可，接下来我们就使用 C 语言原本就提供的`枚举类型 enum`，来更优雅的完成枚举常量的定义

## 参考

<table>
  <tr>
    <td>C enum(枚举)-菜鸟教程</td>
    <td><a href="https://www.runoob.com/cprogramming/c-enum.html">https://www.runoob.com/cprogramming/c-enum.html</a></td>
  </tr>
</table>

# 正文

## 枚举类型语法

`枚举类型(enum)`的语法与`结构体(struct)`的语法相似：

```c
enum <enum_name> {
    val1, val2, ...
} [<variable> [, ...]];
```

当然我们同样可以使用 `typedef` 关键字来省略 `enum xxx` 的麻烦写法hh

> 示例

```c
#include <stdio.h>

typedef enum e_color {
    RED, GREEN, BLUE
} Color;

int main() {
    Color red = RED, green = GREEN, blue = BLUE;
    printf("----- Color -----\n");
    printf("red: %d\n", red);
    printf("green: %d\n", green);
    printf("blue: %d\n", blue);
    printf("sizeof(Color): %lu\n", sizeof(Color));
    printf("sizeof(int): %lu\n", sizeof(int));
}
```

输出

```
----- Color -----
red: 0
green: 1
blue: 2
sizeof(Color): 4
sizeof(int): 4
```

我们可以看到其实枚举量实际上保存的就是整型变量(占用内存为 4B)，而变量值就是枚举变量声明的顺序下标

## 非连续值

我们也可以在枚举类型中显示的指定各个枚举量的值，而没有定义值的变量会是前一个变量的递增：

```c
#include <stdio.h>

typedef enum e_day {
    MON = 1, TUE, WED, THU, FRI, SAT, SUN
    // 周二到周日以周一的 1 为基础依序递增
} Day;

int main() {
    printf("----- days using enum -----\n");
    for(Day day = MON;day<=SUN;day++) {
        switch (day) {
            case MON: printf("Monday is the %d day of the week\n", day); break;
            case TUE: printf("Tuesday is the %d day of the week\n", day); break;
            case WED: printf("Wednesday is the %d day of the week\n", day); break;
            case THU: printf("Thursday is the %d day of the week\n", day); break;
            case FRI: printf("Friday is the %d day of the week\n", day); break;
            case SAT: printf("Saturday is the %d day of the week\n", day); break;
            case SUN: printf("Sunday is the %d day of the week\n", day); break;
        }
    }
}
```

```
----- days using enum -----
Monday is the 1 day of the week
Tuesday is the 2 day of the week
Wednesday is the 3 day of the week
Thursday is the 4 day of the week
Friday is the 5 day of the week
Saturday is the 6 day of the week
Sunday is the 7 day of the week
```

我们可以看到周二以后的值自动根据前一个枚举值递增，如此一来就能避免手抖定义了重复的枚举值，也剩下客观的编码量，提高代码可读性和可维护性。

# 结语

就这，对就这，枚举就这么简单，枚举类型背后的值就是一个递增的计数，避免程序员手动维护每个状态的值，也提高代码的可读性和可维护性，还不赶紧用起来。
