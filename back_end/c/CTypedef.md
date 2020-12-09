# C 基础：typedef 类型定义

@[TOC](文章目录)

<!-- TOC -->

- [C 基础：typedef 类型定义](#c-基础typedef-类型定义)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [复杂变量声明](#复杂变量声明)
    - [基本类型声明](#基本类型声明)
    - [数组类型声明](#数组类型声明)
    - [函数/函数指针声明](#函数函数指针声明)
    - [`先右再左`法则](#先右再左法则)
    - [复合类型](#复合类型)
  - [`typedef` 的使用](#typedef-的使用)
    - [基本类型](#基本类型)
    - [结构体](#结构体)
    - [函数/函数指针](#函数函数指针)
    - [复杂类型](#复杂类型)
- [结语](#结语)

<!-- /TOC -->

## 简介

今天来介绍 C 语言中 `typedef` 关键字的作用。相信用过 C/C++ 的人都知道 `#define` 也能起到类型定义的作用，但是实际上 `#define` 关键字只能起到静态的文本无脑替换的功能；相较之下 `typedef` 更接近声明表达式的形式，接下来我们就来看看 `typedef` 是怎么工作的吧。

## 参考

<table>
  <tr>
    <td>C++typedef的详细用法</td>
    <td><a href="https://www.cnblogs.com/phpandmysql/p/10816949.html">https://www.cnblogs.com/phpandmysql/p/10816949.html</a></td>
  </tr>
</table>

# 正文

## 复杂变量声明

在使用 typedef 之前，我们先来复习一下 C/C++ 中复杂变量的声明和相应的指针类型（已经非常熟悉 C/C++ 的类型和函数指针声明的可以跳过：[`typedef` 的使用](#typedef-的使用)）：

### 基本类型声明

```cpp
int i; // 整数类型，变量名 i
int *ip; // 整数类型指针，变量名 ip
int **ipp; // 指向整数类型指针的指针，变量名 ipp
```

### 数组类型声明

然而变量名不一定都出现在声明表达式的最后

```cpp
int ia[10]; // 整数数组，长度为 10，变量名 ia(同时也是数组首地址)
int (*iap)[10]; // 指向长度为 10 的整数数组的指针，变量名为 iap
// 注意：这里不能写成 int *iap[10]，因为 [] 的优先级比 * 高
```

### 函数/函数指针声明

函数声明的形式又更难理解了

```cpp
int f(); // 无参数、返回 int 类型，函数名 f
int (*fp)(); // 指向无参数、返回 int 类型的函数指针，指针名 fp
void (*fp2)(int, int, float); // 指向接受三个参数(int, int, float)、返回 void 类型的函数指针，指针名 fp2
double (*(*fp3)())(int) // 指向无参数，返回一个函数指针的函数指针，返回的函数指针接受一个 int 参数、返回 double 类型
```

### `先右再左`法则

到此我们可以看出一些端倪，标识符的声明都有 `先右再左` 的共同规则，从`标识符`开始，先向右探，看到 `)` 或 `(` 后调转方向，让我们一步步从简到繁来验证这个规则：

- `int i`
    1. 一般的 int 类型变量声明
   
- `int ia[10]`
    1. 先向右，`[10]` 表示该变量为一个大小为 10 的数组
    2. 再向左，`int` 表示数组元素为 int 类型

- `int (*iap)[10]`
    1. 先向右，`)` 转向
    2. 再向左，`*` 表示是一个指针
    3. 再向右，`[10]` 表示指针指向的是一个大小为 10 的数组
    4. 再向左，`int` 表示指针指向的数组的元素为 int 类型

函数声明和函数指针也是同理：

- `int f()`
    1. 向右，`(` 表示为一个函数，`()` 表示函数无参数
    2. 向左，`int` 表示函数返回类型为 int

- `int (*fp)()`
    1. 向右，`)` 转向
    2. 向左，`*` 表示 fp 是一个指针
    3. 向右，`()` 表示指针指向一个函数，函数无参数
    4. 向左，`int` 表示指针指向的函数返回 int 类型

- `void (*fp2)(int, int, float)`
    1. 向右，`)` 转向
    2. 向左，`*` 表示 fp2 是一个指针
    3. 向右，`(int, int, float)` 表示 fp2 指向一个函数，接受三个参数并确定参数类型
    4. 向右，`void` fp2 指向的函数返回 void 类型

- `double (*(*fp3)())(int)`
    1. 向右，`)` 转向
    2. 向左，`*` 表示 fp3 是一个指针
    3. 向右，`()` 表示 fp3 指向一个函数，不接受任何参数
    4. 向左，`*` 表示 fp3 指向的函数返回的是一个指针
    5. 向右，`(int)` 表示返回的指针是一个函数指针，接受一个 int 类型作为参数
    6. 向左，`double` 表示返回的函数指针的返回类型为 double

### 复合类型

最后我们解析两个 Linux 内核中出现的类型声明：

```cpp
double (*(*(*fp3)())[10])();
int (*(*f4())[10])();
```

我们一样根据`先右再左`的法则来解析这两个类型：

- `double (*(*(*fp3)())[10])()`
    1. `fp3` 向右，`)` 转向
    2. `fp3` 向左，`*` 表示 fp3 是一个指针
    3. `(*fp3)` 向右，`()` 表示 fp3 是一个函数指针，不接受参数
    4. `(*fp3)()` 向左，`*` 表示 fp3 指向的函数的返回类型是一个指针
    5. `(*(*fp3)())` 向右，`[10]` 表示返回的指针指向的是一个大小为 10 的数组
    6. `(*(*fp3)())[10]` 向左，`*` 表示返回的指针指向的数组的元素是指针类型
    7. `(*(*(*fp3)())[10])` 向右，`()` 表示数组元素的指针为函数指针，函数不接受参数
    8. `(*(*(*fp3)())[10])()` 向左，`double` 数组内的函数(指针)返回的是 double 类型

- `int (*(*f4())[10])()`
    1. `f4` 向右，`()` f4 是一个函数，不接受任何参数
    2. `f4()` 向左，`*` f4 返回的是一个指针
    3. `(*f4())` 向右，`[10]` 返回的指针指向的是一个大小为 10 的数组
    4. `(*f4())[10]` 向左，`*` 返回的指针指向的数组的元素是指针
    5. `(*(*f4())[10])` 向右，`()` 数组元素的指针是函数指针，函数不接受参数
    6. `(*(*f4())[10])()` 向左，`int` 数组元素的函数返回的是 int 类型

到此是不是对 C/C++ 的复杂类型和函数指针跟了解了，之后看到再复杂的类型只要把握着`先右再左`的原则就不怕看不懂了。

## `typedef` 的使用

然而当我们需要重复使用这种复杂类型的时候，每次都展开来写的话，代码的可读性将非常低，阅读代码的人需要花费大量时间来重复解析标识符的类型，这时候就是 `typedef` 派上用场的时候了。

`typedef` 关键字的规则很简单。当我们将 `typedef` 加到标识符声明的表达式前面时，标识符就成为一个新的类型：

### 基本类型

```cpp
// 一般的标识符(变量)
int i = 3;

// 标识符表示新的类型
typedef int I;
I i = 3; // 以下三种声明方式一样
I i(3);
I(i) = 3;
```

### 结构体

```cpp
// 一般变量 person
struct Person {
    int id;
    string name;
} p;
struct Person person;
person = {1, "A"};
p = {2, "B"};

// 定义类型
typedef struct { // 不指定名称为匿名结构体
    int id;
    string name;
} Person; // 直接使用类型来声明结构体变量
Person person({1, "A"});
Person(person2) = {2, "B"};
```

### 函数/函数指针

```cpp
// 一般函数声明
int add(int, int);
int sub(int, int);

// 将共同函数类型定义成新类型
typedef int binary_func(int, int);
typedef struct {
    inline binary_func add;
    inline binary_func sub;
} Calculator;
int Calculator::add(int x, int y) { return x + y; }
int Calculator::sub(int x, int y) { return x - y; }
```

```cpp
// 共同形式的函数
void f() { cout << "function f" << endl; }
void g() { cout << "function g" << endl; }
int add(int x, int y) { return x + y; }
int sub(int x, int y) { return x - y; }

// 定义一般函数指针
void (*fp)();
fp = f;
fp(); // output: function f
fp = g;
fp(); // output: function g

int (*fp2)(int, int);
fp2 = add;
cout << fp2(1, 2); // output: 3
fp2 = sub;
cout << fp2(1, 2); // output: -1

// 定义一个函数指针的新类型
typedef void (*FP)();
typedef int (*BFP)(int, int);

FP fp_f = f, fp_g = g;
fp_f(); // output: function f
fp_g(); // output: function g

BFP bfp_add = add, bfp_sub = sub;
cout << bfp_add(1, 2); // output: 3
cout << bfp_sub(1, 2); // output: -1
```

### 复杂类型

同理前面提到的复杂类型也能够构建成一个新的类型，甚至我们能够一步步拆解使得类型声明更平易近人（以 `double (*(*(*_fp)())[10])()` 为例）：

```cpp
typedef double (*DFPV)(); // return Double, Function Pointer, Void parameter
typedef DFPV (*DFPV_ten)[10]; // 指向一个大小为 10 数组，元素为上述类型的指针
typedef DFPV_ten (*FP)();

double (*(*(*_fp)())[10])();
FP fp = _fp; // 成立，说明 _fp 就是 FP 类型的函数指针
```

# 结语

简而言之，`typedef` 关键字的作用就是帮助我们复用复杂类型的声明。与一般的常量一样，我们都知道在程序中不应该出现未经定义的`魔数(magic number)`；同理，程序中也不应该到处冒出未经统一定义的野类型，这时我们就可以使用 `typedef` 关键字统一声明接下来要用到的复杂类型，不仅避免野类型的出现，也大大提高程序的可读性。
