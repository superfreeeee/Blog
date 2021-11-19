# Rust 基础: 三方库依赖 & 自定义三方库

@[TOC](文章目录)

<!-- TOC -->

- [Rust 基础: 三方库依赖 & 自定义三方库](#rust-基础-三方库依赖--自定义三方库)
- [正文](#正文)
  - [1. 项目类型](#1-项目类型)
  - [2. 引入官方三方库](#2-引入官方三方库)
  - [3. 自定义三方库](#3-自定义三方库)
    - [3.1 创建库项目](#31-创建库项目)
    - [3.2 加点内容](#32-加点内容)
    - [3.3 引入自定义三方库](#33-引入自定义三方库)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

今天来介绍一下如何在 rust 项目中依赖三方库，以及如何自定义一个三方库

## 1. 项目类型

在 Rust 中存在两种基础的项目类型

- 一种属于应用(`application`)，主入口默认采用 `src/main.rs`
- 第二种为三方库(`library`)，主入口为 `src/lib.rs`

了解两者的区别之后我们就可以来看看如何在项目中引入三方库

## 2. 引入官方三方库

- [rust 三方库 crates.io](https://crates.io/)

我们知道 rust 项目主要使用 cargo 同时作为依赖管理和打包工具，因此我们只需要在 `Cargo.toml` 配置文件内标明依赖项目和版本号即可

- `/app/Cargo.toml`

```toml
[dependencies]
rand = "0.8.0"
```

接下来就可以在项目内来使用，使用 `extern crate` 关键字来引入三方库

- `/app/src/main.rs`

```rust
extern crate rand;

fn main() {
    println!("Hello, world!");
    let x: i32 = rand::random();
    println!("random x = {}", x);
}
```

- 输出

```
Hello, world!
random x = -700735415
```

## 3. 自定义三方库

而写一个自定义三方库也很简单，就是创建一个 library 项目就可以了！

### 3.1 创建库项目

```bash
$ cargo new lib --lib
```

### 3.2 加点内容

- `/lib/src/lib.rs`

注意三方库的主入口为 `lib.rs`

```rust
pub fn test_fn() {
    println!("invoke test_fn from lib");
}

pub fn add(x: i32, y: i32) -> i32 {
    x + y
}
```

### 3.3 引入自定义三方库

这时候我们就可以放着，回到 app 项目加入依赖

- `/app/Cargo.toml`

```toml
[dependencies]
lib = { path = "../lib" }
```

然后就可以在项目里面使用了

- `/app/src/main.rs`

```rust
extern crate lib;

fn main() {
    lib::test_fn();
    println!("1 + 2 = {}", lib::add(1, 2));
}
```

- 输出

```
invoke test_fn from lib
1 + 2 = 3
```

# 其他资源

## 参考连接

| Title                                   | Link                                                                                                                                   |
| --------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| Dependencies - Rust by Example          | [https://doc.rust-lang.org/rust-by-example/cargo/deps.html](https://doc.rust-lang.org/rust-by-example/cargo/deps.html)                 |
| Creating a New Package - The Cargo Book | [https://doc.rust-lang.org/cargo/guide/creating-a-new-project.html](https://doc.rust-lang.org/cargo/guide/creating-a-new-project.html) |
| Random values - The Rust Rand Book      | [https://rust-random.github.io/book/guide-values.html](https://rust-random.github.io/book/guide-values.html)                           |
| Rust使用国内镜像安装依赖                | [https://www.cnblogs.com/YMaster/p/13232965.html](https://www.cnblogs.com/YMaster/p/13232965.html)                                     |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/back_end/rust/rust_dependency_library](https://github.com/superfreeeee/Blog-code/tree/main/back_end/rust/rust_dependency_library)
