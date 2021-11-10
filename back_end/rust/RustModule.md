# Rust 基础: Module 模块系统

@[TOC](文章目录)

<!-- TOC -->

- [Rust 基础: Module 模块系统](#rust-基础-module-模块系统)
- [正文](#正文)
  - [1. 简介](#1-简介)
  - [2. 文件内模块](#2-文件内模块)
  - [3. 单文件模块](#3-单文件模块)
  - [4. 目录为模块](#4-目录为模块)
  - [5. 目录下其他模块](#5-目录下其他模块)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

## 1. 简介

在 Rust 中的模块系统感觉与 ESM(ECMAScript Module)非常类似，只是更加严谨，同时也限制了作用域与可见性

模块系统实际上用关键字 `mod` 就能进行区分，同时也跟 ESM 一样，一个文件将被视为一个单独的模块；而目录则与 ESM 类似，由特殊名称的文件 `mod.rs` 为代表(就好比 ESM
 里面的 `index.js` 一样)

## 2. 文件内模块

Rust 默认的入口为 `main.rs`，我们可以直接在文件内创建新的模块，而模块内的内容默认为 private 的可见性，我们需要有意识的加上 pub 才能变成外部可见

- `/src/main.rs`

```rs
mod mod_a {
    pub fn f() {
        println!("invoke f in mod_a");
        g();
    }

    fn g() {
        println!("invoke g in mod_a");
    }
}

fn main() {
    mod_a::f();
}
```

- 输出

```
invoke f in mod_a
invoke g in mod_a
```

## 3. 单文件模块

第二种我们使用一个独立的文件作为一个模块

- `/src/mod_b.rs`

```rs
pub fn f() {
    println!("invoke f in mod_b");
}
```

下面我们只要使用 mod 关键字引入就可以了，默认会在当前目录下(同 crate 下)寻找同名模块

- `/src/main.rs`

```rs
mod mod_b;

fn main() {
    mod_b::f();
}
```

- 输出

```
invoke f in mod_b
```

## 4. 目录为模块

第三种使用一个独立的目录区分不同的模块，记得使用 `mod.rs` 来作为主入口(起到 `index.js` 的作用)

- `/src/mod_c/mod.rs`

```rs
pub fn f() {
    println!("invoke f in mod_c");
}
```

- `/src/main.rs`

```rs
mod mod_c;

fn main() {
    mod_c::f();
}
```

- 输出

```
invoke f in mod_c
```

## 5. 目录下其他模块

目录下的其他模块就跟在文件内定义的模块一样，需要使用 pub，并在 `mod.rs` 主入口文件导出才行

- `/src/mod_c/mod_d.rs`

```rs
pub fn f() {
    println!("invoke f in mod_d");
}
```

- `/src/mod_c/mod.rs`

```rs
pub mod mod_d;
```

使用 pub 关键字导出！

- `/src/main.rs`

```rs
mod mod_c;

fn main() {
    mod_c::mod_d::f();
}
```

或是可以使用 use 关键字

```rs
mod mod_c;
fn main() {
    use mod_c::mod_d;
    
    mod_d::f();
}
```

- 输出

```
invoke f in mod_d
```

# 其他资源

## 参考连接

| Title                            | Link                                                                                                                 |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| File hierarchy - Rust by Example | [https://doc.rust-lang.org/rust-by-example/mod/split.html](https://doc.rust-lang.org/rust-by-example/mod/split.html) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/back_end/rust/rust_module](https://github.com/superfreeeee/Blog-code/tree/main/back_end/rust/rust_module)
