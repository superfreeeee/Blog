# Rust 内置类型: Box、Option、Result

@[TOC](文章目录)

<!-- TOC -->

- [Rust 内置类型: Box、Option、Result](#rust-内置类型-boxoptionresult)
- [正文](#正文)
  - [1. Box](#1-box)
    - [1.1 创建 Box](#11-创建-box)
    - [1.2 修改 Box](#12-修改-box)
      - [1.2.1 修改 Box 数据](#121-修改-box-数据)
      - [1.2.2 修改 Box 指针](#122-修改-box-指针)
      - [1.2.3 引用其他变量](#123-引用其他变量)
  - [2. Option](#2-option)
    - [2.1 Option 作为返回值](#21-option-作为返回值)
    - [2.2 Option 解包(解构)：使用 match](#22-option-解包解构使用-match)
    - [2.3 Option 解包：使用 unwrap](#23-option-解包使用-unwrap)
    - [2.4 Option 解包：使用 ?](#24-option-解包使用-)
  - [3. Result](#3-result)
    - [3.1 Result 解包：使用 match](#31-result-解包使用-match)
    - [3.2 Result 解包：使用 ?](#32-result-解包使用-)
  - [4. 小结](#4-小结)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

今天来给大家说说 Rust 当中的三个内置类型：Box、Option、Result

## 1. Box

首先我们先来谈谈 Box 类型，由于 Rust 存在所谓的`所有权转移(move)`的功能，所以一个变量究竟是存在于栈上(stack)还是堆上(heap)似乎就不是那么重要的，但是实际上运行时栈上变量返回并退出是不可避免的会有类似 C++ 里面的拷贝赋值的效果，因此由程序员决定什么时候放置于堆上来降低拷贝赋值的代价就显得非常重要。

我们也可以理解成，在 C++ 当中指针的存在暧昧不清，程序员需要去记忆当前操作的变量到底存在于哪里，是不是应该要被释放；而在 Rust 当中使用明确的 Box 类型来作为堆上变量的引用，如此一来便可以交给 Rust 来管理对上引用计数，并自动清理堆上内存

### 1.1 创建 Box

第一部分我们先来看 Box 的创建

- `/src/box_test.rs`

首先是基础类型的创建

```rs
#[derive(Debug, Copy, Clone)]
struct Point {
    x: i32,
    y: i32,
}

#[derive(Debug)]
struct Rectangle {
    top_left: Point,
    bottom_right: Point,
}
```

接下来我们使用 `std::mem::size_of_val` 来区别 Box 指针与一般栈上变量的占用空间区别

```rs
fn test_alloc() {
    println!(">>>>> test allocation");

    // Point on stack
    let point = Point { x: 0, y: 0 };
    println!("point = {:?}", point);
    println!("sizeof(point) = {}", size_of_val(&point));

    // Rectangle on stack
    let point_b = Point { x: 10, y: 10 };
    let rect = Rectangle { top_left: point, bottom_right: point_b };
    println!("rect = {:?}", rect);
    println!("sizeof(rect) = {}", size_of_val(&rect));
```

```
>>>>> test allocation
point = Point { x: 0, y: 0 }
sizeof(point) = 8
rect = Rectangle { top_left: Point { x: 0, y: 0 }, bottom_right: Point { x: 10, y: 10 } }
sizeof(rect) = 16
```

普通的栈上变量理所当然就是整个数据块的大小，分别是两个 `i32` 和两个 `Point` 的大小，占用 8B 和 16B

```rs
    // Point on heap
    let box_point = Box::from(point);
    println!("box_point = {:?}", box_point);
    println!("sizeof(box_point) = {}", size_of_val(&box_point));

    // Rectangle on heap
    let box_rect = Box::from(rect);
    println!("box_rect = {:?}", box_rect);
    println!("sizeof(box_rect) = {}", size_of_val(&box_rect));

    println!();
}
```

```
box_point = Point { x: 0, y: 0 }
sizeof(box_point) = 8
box_rect = Rectangle { top_left: Point { x: 0, y: 0 }, bottom_right: Point { x: 10, y: 10 } }
sizeof(box_rect) = 8
```

而使用 Box 类型之后，数据都被转移到堆上，因此剩余的指针大小统一都是 8B，也就是 64 位

### 1.2 修改 Box

- `/src/box_test.rs`

#### 1.2.1 修改 Box 数据

修改 Box 的数据实际上就跟修改普通数据一样

- `/src/box_test.rs`

```rs
fn test_mut() {
    println!(">>>>> test mutability");

    // init
    let mut point = Point { x: 0, y: 0 };
    let mut box_point = Box::from(point);
    println!("point = {:?}", point);
    println!("box_point = {:?}", box_point);

    // assign 'mut Box<Point>'
    box_point.x = 1;
    box_point.y = 1;
    println!("point = {:?}", point);
    println!("box_point = {:?}", box_point);
```

```
>>>>> test mutability
point = Point { x: 0, y: 0 }
box_point = Point { x: 0, y: 0 }
point = Point { x: 0, y: 0 }
box_point = Point { x: 1, y: 1 }
```

我们可以直接对 Box 的指向的变量重新赋值（注意这里由于 point 实现了 Clone 特性，所以实际上 box_point 指向了另一个 Point 对象，这点可以从 `Box<Point>` 类型看出）

#### 1.2.2 修改 Box 指针

第二种我们还可以修改 Box 的指针，让他指向一个新的 Point 对象

```rs
    // rebind 'Box<Point>'
    *box_point = Point { x: 2, y: 2 };
    println!("point = {:?}", point);
    println!("box_point = {:?}", box_point);
```

```
point = Point { x: 0, y: 0 }
box_point = Point { x: 2, y: 2 }
```

注意这里还是 `Box<Point>` 类型，所以依旧不是原来的那个 point

#### 1.2.3 引用其他变量

由于 Rust 的所有权定义，我们可以明确辨别原始数据与引用数据的问题，也就是说 `Box<Point>`，永远都是持有一份自己的 Point 对象；因此我们想要引用别的数据时，其实也就是将引用的变量赋值给 Box 就能形成类似的效果

```rs
    // assign 'Box<&mut Point>'
    let box_point_ref = Box::from(&mut point);
    box_point_ref.x = 3;
    box_point_ref.y = 3;
    println!("box_point_ref = {:?}", box_point_ref);
    println!("point = {:?}", point);

    println!();
}
```

```
box_point_ref = Point { x: 3, y: 3 }
point = Point { x: 3, y: 3 }
```

这里就要注意所谓的引用所有权问题，Box 变量持有了原 point 变量的可变引用(`&mut`)，因此在这个 box_point_ref 使用完毕之前都不能再使用任何其他引用(`&point`)，这也是为什么需要把 print 的顺序反过来

## 2. Option

下面要介绍的两个类型就好像孪生兄弟，用法非常的相似。

Option 类型相当于 Java 中的对象的感觉，是一个可为'空'的数据类型。Java 中大名鼎鼎的 NullException 困扰了无数人，也写出了无数个 `if (value != null)` 的类型检查，不胜其扰。

在 Rust 中定义了所谓的 `变体(Variant)` 的概念，当一个数据并不确定其存在的时候，就使用变体来解决，可以看做就是 Null 的翻版

Option 存在两种变体：`Some<T>(value)` 和 `None`，下面我们来看看到底怎么用

### 2.1 Option 作为返回值

第一个我们先来看当一个函数返回 Option 的写法

- `/src/option_test.rs`

```rs
fn div(a: i32, b: i32) -> Option<f64> {
    if b == 0 {
        None
    } else {
        Some(a as f64 / b as f64)
    }
}

// Option 作为返回值
fn test_return() {
    println!(">>>>> test Option as return");

    println!("1 / 1 = {:?}", div(1, 1));
    println!("1 / 0 = {:?}", div(1, 0));
    println!("0 / 1 = {:?}", div(0, 1));
    println!("1 / 2 = {:?}", div(1, 2));

    println!();
}
```

```
>>>>> test Option as return
1 / 1 = Some(1.0)
1 / 0 = None
0 / 1 = Some(0.0)
1 / 2 = Some(0.5)
```

`Some` 跟 `None` 都作为内置类型，可以直接使用而不用一直 `Option::`，同时我们在 div 函数使用了表达式直接作为返回值的写法(不加 `;`)，这里我们就可以看到除数为 0 的时候除法无意义，因此返回 None

### 2.2 Option 解包(解构)：使用 match

接下来我们演示如何使用 match 来对 Option 变量解包

- `/src/option_test.rs`

```rs
fn match_option(option: Option<i32>) {
    match option {
        Some(val) => println!("match val = {}", val),
        None => println!("match Option::None")
    }
}

// Option 透过 match 解构
fn test_match() {
    println!(">>>>> test match Option");

    match_option(None);
    match_option(Some(4));
}
```

```
>>>>> test match Option
match Option::None
match val = 4
```

`match` 的语法就不多说了，自己看。本质上我们直接使用

```rs
match option {
    Some(val) => {/* ... */}
    None => {/*  ... */}
}
```

的定型就能够分别处理 Some 的返回类型与 None 的返回类型

```rs

fn match_assign_option(option: Option<i32>) -> i32 {
    if let Some(val) = option {
        val
    } else {
        0
    }
}

fn test_match() {
    println!("val = {}", match_assign_option(None));
    println!("val = {}", match_assign_option(Some(5)));
}
```

```
val = 0
val = 5
```

另外 match 还有一个更灵活的语法，可以使用 `if let Some(val) = option` 的表达式，来接住一个 Option 类型。前面我们说过 Option 属于一种变体(Variant)，因此当 `if let Some(val) = option` 尝试解包失败的时候，就相当于是 None，走下面的 else 路线

这在不需要处理 None 的时候将非常好用，如

```rs
if let Some(val) = option {
    // do something with val
}
```

### 2.3 Option 解包：使用 unwrap

有时候每次都写这么个 Some、None 还是挺烦的，因此当我们非常确定有数据的时候(或是没数据应该作为异常处理)，我们可以使用 `unwrap` 强行解包

- `/src/option_test.rs`

```rs
fn unwrap_option(option: Option<i32>) -> i32 {
    option.unwrap()
}
```

首先我们定义一个函数直接返回解包的结果

```rs
fn unwrap_print_option(f: fn() -> i32) {
    if let Ok(val) = panic::catch_unwind(f) {
        println!("val = {}", val);
    }
}
```

接下来我们使用 `panic::catch_unwind` 来接住 panic，并打印结果，如果有的话

```rs
// Option 透过 unwrap 解构
fn test_unwrap() {
    println!(">>>>> test Option.unwrap");

    unwrap_print_option(|| unwrap_option(None));
    unwrap_print_option(|| unwrap_option(Some(6)));

    println!();
}
```

最后我们使用 closure（匿名函数）来指定任务

```
>>>>> test Option.unwrap
thread 'main' panicked at 'called `Option::unwrap()` on a `None` value'
val = 6
```

这里有一个问题在于，当 `unwrap()` 尝试着解一个 `None` 的值的时候会产生一个 `panic`，我们需要使用 `panic::catch_unwind` 来接住，但是这样并不好，写起来也麻烦，最多只能作为全局异常处理来打日志。正常的处理流程我们都应该透过类型判断对 None 进行适当的处理

### 2.4 Option 解包：使用 ?

最后我们再提供一种对 Option 解包的方式，使用 `?` 关键字

- `/src/option_test.rs`

直接用 `unwrap` 很烦，不小心就抛出了个 panic，而使用 `?` 不同，如果是 none 的话他也会返回，这时候我们就可以使用更简洁的语法

```rs
fn create_option(x: i32) -> Option<i32> {
    if x >= 0 {
        Some(x)
    } else {
        None
    }
}

fn check_positive_integer(x: i32) {
    if let None = (|| {
        let val = create_option(x)?;

        println!("get positive integer: {}", val);

        Some(0)
    })() {
        println!("{} is not positive integer", x);
    }
}

// 使用 ? 语法解构
fn test_question_syntax() {
    println!(">>>>> test Option?");

    check_positive_integer(100);
    check_positive_integer(0);
    check_positive_integer(-100);

    println!();
}
```

```
>>>>> test Option?
get positive integer: 100
get positive integer: 0
-100 is not positive integer
```

这里一样用了一个匿名表达式，如果是整数就打印 `get`，如果返回了 None 就打印 `not positive`

## 3. Result

最后一个今天要介绍的类型是 Result，与 Option 类似，它也有两种变体：`Ok<T>(value)`、`Err<E>(why)`

实际上你可以把 Result 看成一个更具体的 Option，在 Option 之上赋予了更多语义：

1. `Ok(value)` 表示处理成功，并返回结果
2. `Err(why)` 表示处理失败，并附带原因（通常也就是异常信息）

### 3.1 Result 解包：使用 match

一样我们先用 match 解包试一把

- `/src/result_test.rs`

首先先定义好异常类型

```rs
#[derive(Debug)]
enum MathError {
    DivisionByZero,
    NonPositiveLogarithm,
    NegativeSquareRoot,
}

type MathResult = Result<f64, MathError>;
```

定义三个运算函数，并使用别名定义 `MathResult` 类型，作为一个更具体的 Result 类型（这也是 Rsut 里面常用的手段）

```rs
// 除法
fn div(x: f64, y: f64) -> MathResult {
    if y == 0.0 {
        Err(MathError::DivisionByZero)
    } else {
        Ok(x / y)
    }
}

// 平方根
fn sqrt(x: f64) -> MathResult {
    if x < 0.0 {
        Err(MathError::NegativeSquareRoot)
    } else {
        Ok(x.sqrt())
    }
}

// 取对数
fn ln(x: f64) -> MathResult {
    if x <= 0.0 {
        Err(MathError::NonPositiveLogarithm)
    } else {
        Ok(x.ln())
    }
}
```

三个函数都会返回一个 `MathResult`，也就是可能产生异常的方法，最后我们一样用多层的 match 一一解析

```rs
// sqrt(ln(x / y)) 测试
fn test_ops(x: f64, y: f64) -> MathResult {
    match div(x, y) {
        Err(e) => Err(e),
        Ok(ratio) => {
            match ln(ratio) {
                Err(e) => Err(e),
                Ok(ln) => {
                    match sqrt(ln) {
                        Err(e) => Err(e),
                        Ok(sqrt) => Ok(sqrt)
                    }
                }
            }
        }
    }
}

fn test_ops_result(x: f64, y: f64) {
    match test_ops(x, y) {
        Err(e) => {
            println!("test({}, {}) fail: {:?}", x, y, e);
        }
        Ok(val) => {
            println!("sqrt(ln({} / {})) = {}", x, y, val);
        }
    }
}
```

并额外定义一个打印结果的函数，下面就可以测试一下

```rs
pub fn test() {
    println!(">>>>> test 1");
    test_ops_result(1.0, 0.0);
    test_ops_result(-1.0, 2.0);
    test_ops_result(1.0, 2.0);
    test_ops_result(5.0, 2.0);
}
```

```
>>>>> test 1
test(1, 0) fail: DivisionByZero
test(-1, 2) fail: NonPositiveLogarithm
test(1, 2) fail: NegativeSquareRoot
sqrt(ln(5 / 2)) = 0.9572307620809911
```

我们看到虽然有点繁杂，但是非常的严谨，所有异常路径都会被考虑到

### 3.2 Result 解包：使用 ?

但是我们还是很懒，我们回头看一下 `test_ops` 函数

```rs
// sqrt(ln(x / y)) 测试
fn test_ops(x: f64, y: f64) -> MathResult {
    match div(x, y) {
        Err(e) => Err(e),
        Ok(ratio) => {
            match ln(ratio) {
                Err(e) => Err(e),
                Ok(ln) => {
                    match sqrt(ln) {
                        Err(e) => Err(e),
                        Ok(sqrt) => Ok(sqrt)
                    }
                }
            }
        }
    }
}
```

这个写法实在是太啰嗦而多余了，`Err(e) => Err(e)` 是什么鬼

这里与 Options 雷同，我们一样可以使用 `unwrap`、`?` 来解构；`unwrap` 就不说了（返回 `Ok(val)` 的 val，否则抛出 panic），而 `?` 则可以将 `Err` 路径的返回值直接作为函数返回值返回

- `/src/result_test.rs`

```rs
fn test_ops_simple(x: f64, y: f64) -> MathResult {
    let ratio = div(x, y)?;
    let ln = ln(ratio)?;
    let sqrt = sqrt(ln)?;

    Ok(sqrt)
}
```

使用 `?` 解包，简直清爽的不要不要的，甚至你想搞一点写成

```rs
fn test_ops_simple(x: f64, y: f64) -> MathResult {
    Ok(sqrt(ln(div(x, y)?)?)?)
}
```

也可以，下面一样测试一下输出就行啦

```rs
fn test_ops_result_simple(x: f64, y: f64) {
    match test_ops_simple(x, y) {
        Err(e) => {
            println!("test({}, {}) fail: {:?}", x, y, e);
        }
        Ok(val) => {
            println!("sqrt(ln({} / {})) = {}", x, y, val);
        }
    }
}
```

```rs
pub fn test() {
    println!("\n>>>>> test 2");
    test_ops_result_simple(1.0, 0.0);
    test_ops_result_simple(-1.0, 2.0);
    test_ops_result_simple(1.0, 2.0);
    test_ops_result_simple(5.0, 2.0);

    println!();
}
```

```
>>>>> test 2
test(1, 0) fail: DivisionByZero
test(-1, 2) fail: NonPositiveLogarithm
test(1, 2) fail: NegativeSquareRoot
sqrt(ln(5 / 2)) = 0.9572307620809911
```

## 4. 小结

这三个内置类型是 Rust 编程里面非常重要的三个类型，可以说是到处都会出现，学好基础操作才能 handle 更复杂的应用

# 其他资源

## 参考连接

| Title                                 | Link                                                                                                                     |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Box, stack and heap - Rust By Example | [https://doc.rust-lang.org/rust-by-example/std/box.html](https://doc.rust-lang.org/rust-by-example/std/box.html)         |
| 理解Rust中的Result/Option/unwrap/?    | [https://blog.csdn.net/shebao3333/article/details/106780900](https://blog.csdn.net/shebao3333/article/details/106780900) |
| Rust 抓取 panic 并恢复运行            | [https://blog.biofan.org/2019/04/rust-unwind/](https://blog.biofan.org/2019/04/rust-unwind/)                             |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/back_end/rust/rust_build_in_types](https://github.com/superfreeeee/Blog-code/tree/main/back_end/rust/rust_build_in_types)
