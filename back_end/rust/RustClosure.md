# Rust Closure 闭包解析（匿名函数）

@[TOC](文章目录)

<!-- TOC -->

- [Rust Closure 闭包解析（匿名函数）](#rust-closure-闭包解析匿名函数)
- [正文](#正文)
  - [1. 简单闭包 - 纯粹的匿名函数](#1-简单闭包---纯粹的匿名函数)
  - [2. 捕获上下文 & FnOnce、FnMut、Fn 类型验证](#2-捕获上下文--fnoncefnmutfn-类型验证)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

闭包与匿名函数是现代语言对与函数式编程范式常见的实现，与 JavaScript 惊人的相似的是对于外部上下文的变量捕捉等特性，下面我们就来介绍如何在 Rust 中使用闭包，同时它也将作为 Rust 多线程编程中的重要基础知识之一

## 1. 简单闭包 - 纯粹的匿名函数

首先我们先介绍第一种，最简单的闭包用法，在代码行中创建一个行内匿名函数

- `/src/anonymous_function.rs`

基本的函数的类型是 `Fn`，我们创建一个结构体用于保存一个函数与其运行

```rust
extern crate rand;

use std::thread;
use std::time::Duration;
use std::ops::Fn;
use rand::random;

struct FnCache<T> where T: Fn(u32) -> u32 {
    calculation: T,
    value: Option<u32>,
}
```

然后定义创建构造函数与数据访问函数

```rust
impl<T> FnCache<T> where T: Fn(u32) -> u32 {
    fn new(calculation: T) -> FnCache<T> {
        FnCache {
            calculation,
            value: None,
        }
    }

    fn value(&mut self, arg: u32) -> u32 {
        match self.value {
            Some(value) => value,
            None => {
                let value = (self.calculation)(arg);
                self.value = Some(value);
                value
            }
        }
    }
}
```

接下来我们就能够在行内调用构造函数并传入匿名函数作为可配置的算法过程

```rust
fn generate_task(intensity: u32) -> bool {
    let mut expensive_closure = FnCache::new(|num| {
        println!("calculating slowly...");
        thread::sleep(Duration::from_secs(2));
        println!("done!");
        num
    });

    if intensity < 20 {
        println!("intensity = {}, keep working", expensive_closure.value(intensity));
        false
    } else {
        println!("intensity = {}, enough", intensity);
        true
    }
}
```

我们可以看到 Rust 使用 `|params| { statements }` 的语法结构来表示匿名函数

最后我们做一个小循环进行测试

```rust
pub fn test() {
    println!(">>>>> anonymous_function");

    let mut count = 1;
    loop {
        let i: u32 = random();
        let random_intensity = i % 30;
        let done = generate_task(random_intensity);
        if done || count >= 3 {
            break;
        }
        count += 1;
    }
```

- 输出

```
>>>>> anonymous_function
calculating slowly...
done!
intensity = 7, keep working
intensity = 20, enough
```

当然实际上我们使用 `fn` 关键字定义的一般函数实际上也拥有一样的类型签名

```rust
fn simple_fn(x: u32) -> u32 {
    println!("simple_fn({})", x);
    x
}

pub fn test() {
    println!("\n! test normal function as Fn");
    let mut cache = FnCache::new(simple_fn);
    println!("cache.value(1) = {}", cache.value(1));
    println!("cache.value(2) = {}", cache.value(2));

    println!();
}
```

- 输出

```
! test normal function as Fn
simple_fn(1)
cache.value(1) = 1
cache.value(2) = 1
```

## 2. 捕获上下文 & FnOnce、FnMut、Fn 类型验证

然而闭包最强大的部分除了可以自由在运行时创建函数之外，就是能够捕获上下文环境的变量这个特性，就能够将一些已经消失的作用域中的变量透过闭包捕获并私有化

下面我们使用三个例子来演示使用闭包捕获上下文变量，同时顺便验证 `FnOnce、FnMut、Fn` 这三个类型的不同之处

- `/src/capture_environment.rs`

第一个例子创建一个比较函数，并能够提前绑定参数

```rust
fn capture_var(x: u32) -> Box<dyn Fn(u32) -> bool> {
    Box::new(move |y: u32| y == x)
}

pub fn test() {
    println!(">>>>> capture_environment");

    println!("! Fn test");
    let equal_one = capture_var(1);
    println!("equal_one(3) = {}", (*equal_one)(3));
    println!("equal_one(2) = {}", (*equal_one)(2));
    println!("equal_one(1) = {}", (*equal_one)(1));
}
```

- 输出

```
>>>>> capture_environment
! Fn test
equal_one(3) = false
equal_one(2) = false
equal_one(1) = true
```

第二个例子演示了一个计数器的实现，修改了上下文的变量因此用到了 `FnMut` 类型

```rust
fn counter() -> Box<dyn FnMut(bool) -> u32> {
    let mut val = 0;
    Box::new(move |incr| {
        if incr {
            val += 1;
        }
        val
    })
}

pub fn test() {
    println!("! FnMut test");
    let mut increment = counter();
    println!("increment() = {}", (*increment)(true));
    println!("increment() = {}", (*increment)(true));
    println!("increment() = {}", (*increment)(false));
}
```

- 输出

```
! FnMut test
increment() = 1
increment() = 2
increment() = 2
```

第三个例子则是测试只能调用一次的函数签名 `FnOnce`

```rust
fn once<T>(f: T) -> Box<dyn Fn() -> u32> where T: FnOnce() -> u32 {
    let val = f();
    Box::new(move || val)
}

pub fn test() {
    println!("! FnOnce test");
    let gettter = once(|| {
        3
    });
    println!("get = {}", (*gettter)());
}
```

- 输出

```
! FnOnce test
get = 3
```

# 其他资源

## 参考连接

| Title                                                                                            | Link                                                                                                         |
| ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------ |
| Closures: Anonymous Functions that Can Capture Their Environment - The Rust Programming Language | [https://doc.rust-lang.org/book/ch13-01-closures.html](https://doc.rust-lang.org/book/ch13-01-closures.html) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/back_end/rust/rust_closure](https://github.com/superfreeeee/Blog-code/tree/main/back_end/rust/rust_closure)
