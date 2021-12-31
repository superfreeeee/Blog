# Rust: Cargo Workspaces 多项目（即 Monorepo）

@[TOC](文章目录)

<!-- TOC -->

- [Rust: Cargo Workspaces 多项目（即 Monorepo）](#rust-cargo-workspaces-多项目即-monorepo)
- [Workspaces？Monorepo？](#workspacesmonorepo)
- [Cargo Workspace 特征](#cargo-workspace-特征)
- [项目实践](#项目实践)
  - [项目结构](#项目结构)
  - [项目构建](#项目构建)
    - [构建项目](#构建项目)
    - [根目录配置文件 Cargo.toml](#根目录配置文件-cargotoml)
    - [声明依赖](#声明依赖)
  - [项目功能填充](#项目功能填充)
  - [打包运行](#打包运行)
- [小结](#小结)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# Workspaces？Monorepo？

作为一个前端人，相信大家对于近期越来越火的 monorepo 都略有耳闻，从 lerna、yarn workspaces、pnpm 等，实际上这样的概念在后端也是一样的。

今天要介绍的 Cargo Workspaces 就是对于 Rust 开发下提供官方级别的 monorepo 实践方式，我们使用的官方工具 cargo 自带的功能，其称为 **Workspaces**

# Cargo Workspace 特征

workspaces 与 monorepo 其实几乎可以说是一样，不外乎就是把多个关联的项目统统放到同一个项目下；然而这与子项目的概念是不同的，大多数的实现主要用于一次构建多个平行、顶级的包作为独立模块，当然你用 Workspace 的时候定义一个对外暴露的主包，并用它来关联其他包也是可以的。

不过最重要的一个特征就是，在 workspaces 下每个包都应该是能够独立运行的个体，能够除了本项目以外的项目引入并运行

除了这些之外，cargo worksapces 还提出了其它约束如共同依赖包版本应该一致能，算是比较琐碎的规范，有兴趣可以看看[官方介绍](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html)

# 项目实践

接下来带大家用一用这个 Cargo Workspaces（官方的文档真的写得很好，跟着走一遍就懂了）

## 项目结构

在开始之前我们先来看看最终产物的项目结构

```
/rust_workspaces
├── Cargo.lock
├── Cargo.toml
└── crates
    ├── add-one
    │   ├── Cargo.toml
    │   └── src
    │       └── lib.rs
    ├── add-two
    │   ├── Cargo.toml
    │   └── src
    │       └── lib.rs
    └── adder
        ├── Cargo.toml
        └── src
            └── main.rs
```

本篇的最终产物会有三个包，分别是 `adder`、`add-one`、`add-two`，其中 `adder` 又依赖了其余两个包；全部三个包都放到 `crates` 目录下（当然你想放哪都行的），整个项目作为一个整体放到一个仓库进行管理

## 项目构建

接下来我们来看看如何构建一个如上的 cargo workspaces 多模块（或说多包）项目，实际上跟你原本用 cargo 的方式一模一样！

### 构建项目

首先先把三个小项目都建起来

```bash
# 目录结构
mkdir rust_workspaces
cd rust_workspaces
mkdir crates
cd crates
# 构建三个包
cargo new adder --lib
cargo new add-one --lib
cargo new add-two --lib
```

没错就是这么粗暴

### 根目录配置文件 Cargo.toml

我们知道通常一个由 Cargo 管理的 rust 项目，最主要的配置文件就是 `Cargo.toml`；一个 workspaces 项目也是一样的

- 创建配置文件

```bash
touch Cargo.toml
```

- 编辑配置文件

这里与我们通常熟悉的部分就不同了，我们不需要填哪些正常项目或是包应该要填的东西，只需要一个 `workspace` 属性，声明当前 workspaces 都有哪些项目就可以了！

- `/Cargo.toml`

```toml
[workspace]
members = [
    "crates/adder",
    "crates/add-one",
    "crates/add-two",
]
```

可以看到实际上就是指定每一个项目根目录就可以了，所以其实你也可以直接平铺在项目根目录下，而不用放到什么 `crates` 目录下

### 声明依赖

前面我们提过 `adder` 包需要依赖另外两个包，但是根目录下的 `Cargo.toml` 只会识别当前 workspaces 下的项目，而无法知道你到底想要怎样的依赖关系

这其实也好解决，分别在每个项目下自己的 `Cargo.toml` 定义好自己的依赖关系就可以了

- `/crates/adder/Cargo.toml`

```toml
[package]
name = "adder"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
add-one = { path = "../add-one" }
add-two = { path = "../add-two" }
```

## 项目功能填充

接下来我们稍微填充一下各个项目的内容代码

- `/crates/add-one/src/lib.rs`

```rust
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

- `/crates/add-two/src/lib.rs`

```rust
pub fn add_two(x: i32) -> i32 {
    x + 2
}
```

- `/crates/adder/src/main.rs`

```rust
use {add_one, add_two};

fn main() {
    let num = 10;
    println!("Hello, world! {} + 1 = {}", num, add_one::add_one(num));
    println!("Hello, world! {} + 2 = {}", num, add_two::add_two(num));
}
```

## 打包运行

最后你可能会疑惑，那我们该怎样运行呢？我们只需要在根目录下进行打包，Cargo 就会自动识别，将最终产物统一构建在根目录的 `/target` 目录下，甚至我们进入到单个项目的根目录下运行相同指令也是一样的效果，不会产生多余的 target 目录

```bash
# 当前在根目录，也就是 rust_workspaces 目录下
pwd  # ~/rust_workspaces

cargo build  # 打包项目（可以省略）
cargo run --bin adder  # 指定运行 adder 项目（adder 是一个 bin 项目）
```

- 最终结果

```
   Compiling adder v0.1.0 (~/rust_workspaces/crates/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.53s
     Running `target/debug/adder`
Hello, world! 10 + 1 = 11
Hello, world! 10 + 2 = 12
```

# 小结

如果读者有兴趣可以去找找一些开源库来看看，现在 workspaces 或是说 monorepo 的概念其实已经在 rust 的社区广泛被采纳，写一些自己的小项目的时候也可以上手用看看，非常不错。（重点是集成的很好，体验极佳，跟某 lernx、什么 yar? wor!space 不一样）

# 其他资源

## 参考连接

| Title                        | Link                                                                                                                         |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Cargo Workspaces - The Rust Programming Language | [https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/back_end/rust/rust_workspaces](https://github.com/superfreeeee/Blog-code/tree/main/back_end/rust/rust_workspaces)
