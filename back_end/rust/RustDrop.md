# Rust 特性: Drop 特性(类 C++ 析构函数)

@[TOC](文章目录)

<!-- TOC -->

- [Rust 特性: Drop 特性(类 C++ 析构函数)](#rust-特性-drop-特性类-c-析构函数)
- [正文](#正文)
  - [0. 关于析构函数](#0-关于析构函数)
  - [1. 代码实现](#1-代码实现)
    - [1.0 Drop 实现](#10-drop-实现)
    - [1.1 自动回收](#11-自动回收)
    - [1.2 主动回收](#12-主动回收)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

## 0. 关于析构函数

写过 C++ 的应该都知道，资源被释放的时候会调用析构函数，方法签名如下

```cpp
class AClass {
    public:
        ~AClass();
}
```

Java 也有类似的函数

```java
class Object {
    finialize();
}
```

但是由于 Java 引入了 GC 的概念，因此真实的资源回收时机变得模糊，可以说是一个挺失败的设计hh

当然，作为与 C 语言同层级的现代语言，Rust 也提供了相应的析构函数写法，以一个特殊的 `Trait - Drop` 的形式提供，实现方式如下

```rs
impl Drop for AStruct {
    fn drop(&mut self) {
        // do something before drop
    }
}
```

## 1. 代码实现

下面我们来看看代码实现，先定义两个结构体，又是熟悉的 Point 和 Rectangle

```rs
#[derive(Debug)]
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

### 1.0 Drop 实现

然后为这两个结构体实现一个一样的 Drop 特性

```rs
use std::fmt::Debug;
use std::mem::size_of_val;

fn drop_resource<T: Debug>(item: &mut T) {
    println!("=== [release resources] {:?} ,free: {} bytes", item, size_of_val(item));
}

impl Drop for Point {
    fn drop(&mut self) {
        drop_resource(self);
    }
}

impl Drop for Rectangle {
    fn drop(&mut self) {
        drop_resource(self);
    }
}
```

### 1.1 自动回收

第一个例子我们先看自动回收的效果

- Sample1：两个 Point

```rs
fn main() {
    let point = Point { x: -1, y: -1 };
    println!("point = {:?}", point);

    let point = Point { x: 0, y: 0 };
    println!("point = {:?}", point);
}
```

```
point = Point { x: -1, y: -1 }
point = Point { x: 0, y: 0 }
=== [release resources] Point { x: 0, y: 0 } ,free: 8 bytes
=== [release resources] Point { x: -1, y: -1 } ,free: 8 bytes
```

我们可以看到即便修改 point 指向，实际上 -1 的那个 Point 依旧会存活整个作用域的生命周期；同时注意到释放看起来应该是照着方法栈的顺序一一回收，不过直觉告诉我们这个顺序不可依赖

- Sample2：Rectangle 与 Box

```rs
fn main() {
    let rect = Rectangle {
        top_left: Point { x: 1, y: 1 },
        bottom_right: Point { x: 2, y: 2 },
    };
    println!("rect = {:?}", rect);

    let box_rect = Box::new(Rectangle {
        top_left: Point { x: 3, y: 3 },
        bottom_right: Point { x: 4, y: 4 },
    });
    println!("box_rect = {:?}", box_rect);
}
```

```
rect = Rectangle { top_left: Point { x: 1, y: 1 }, bottom_right: Point { x: 2, y: 2 } }
box_rect = Rectangle { top_left: Point { x: 3, y: 3 }, bottom_right: Point { x: 4, y: 4 } }
=== [release resources] Rectangle { top_left: Point { x: 3, y: 3 }, bottom_right: Point { x: 4, y: 4 } } ,free: 16 bytes
=== [release resources] Point { x: 3, y: 3 } ,free: 8 bytes
=== [release resources] Point { x: 4, y: 4 } ,free: 8 bytes
=== [release resources] Rectangle { top_left: Point { x: 1, y: 1 }, bottom_right: Point { x: 2, y: 2 } } ,free: 16 bytes
=== [release resources] Point { x: 1, y: 1 } ,free: 8 bytes
=== [release resources] Point { x: 2, y: 2 } ,free: 8 bytes
```

我们发现作为 Rectangle 子元素的 Point 也会递归释放，一样顺序不可依赖(如果编译器对栈上变量做重排列会打乱顺序)

### 1.2 主动回收

实际上我们依赖自动回收已经很够了，同时 Rust 的资源利用跟踪机制实在是非常的好，所以我们会用到主动回收的时机是在少之又少，也是一个全局提供的 `drop` 函数

```rs
fn main() {
    let point = Point { x: -1, y: -1 };
    println!("point = {:?}", point);
    drop(point);

    let point = Point { x: 0, y: 0 };
    println!("point = {:?}", point);
}
```

```
point = Point { x: -1, y: -1 }
=== [release resources] Point { x: -1, y: -1 } ,free: 8 bytes
point = Point { x: 0, y: 0 }
=== [release resources] Point { x: 0, y: 0 } ,free: 8 bytes
```

都主动回收了，当然 drop 特性也就提前执行咯

# 其他资源

## 参考连接

| Title                  | Link                                                                                                                   |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Drop - Rust by Example | [https://doc.rust-lang.org/rust-by-example/trait/drop.html](https://doc.rust-lang.org/rust-by-example/trait/drop.html) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/back_end/rust/rust_drop](https://github.com/superfreeeee/Blog-code/tree/main/back_end/rust/rust_drop)
