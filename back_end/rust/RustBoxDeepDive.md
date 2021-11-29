# Rust: Box 智能指针进阶 - Rc、RefCell、Weak

@[TOC](文章目录)

<!-- TOC -->

- [Rust: Box 智能指针进阶 - Rc、RefCell、Weak](#rust-box-智能指针进阶---rcrefcellweak)
- [正文](#正文)
  - [1. Box 类型复习](#1-box-类型复习)
  - [2. 自定义模拟 Box 类型](#2-自定义模拟-box-类型)
  - [3. 递归类型定义](#3-递归类型定义)
  - [4. Box 进阶：Rc](#4-box-进阶rc)
  - [5. Box 进阶：RefCell](#5-box-进阶refcell)
  - [6. 循环依赖 & Weak](#6-循环依赖--weak)
  - [7. 结论](#7-结论)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

## 1. Box 类型复习

[Rust 内置类型: Box、Option、Result](https://blog.csdn.net/weixin_44691608/article/details/121316161)

- Box 为一种特别的引用类型，实际数据存储在栈上
- Box 作为引用类型默认实现了 `Deref` 特性来实现数据访问（获取真实数据引用）

## 2. 自定义模拟 Box 类型

我们自定义实现 Box 类型有两个目标

- 作为智能指针管理真实数据的生命周期
- 实现 `Deref` 特性代理真实数据的引用

下面直接看代码

- `/src/own_smart_pointer.rs`

首先定义一个 MyBox 类型，并定义 `new` 方法用于构造

```rust
use std::ops::Deref;

struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}
```

接下来实现 `Deref` 特性，会在引用 Box 数据的时候代理成内部数据对象的引用

```rust
impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &Self::Target {
        &self.0
    }
}
```

测试的时候我们可以看到，我们调用 `hello` 并传入 `&name`，Rust 会自动调用 `deref` 方法将 `&name` 转换成 `&String` 类型，然后再透过 String 的 `Deref` 实现转换成 `&str` 类型，最终才能够成功被调用

```rust
fn hello(name: &str) {
    println!("Hello {}", name);
}

pub fn test() {
    println!(">>>>> test own_smart_pointer");

    let name = MyBox::new(String::from("superfree"));
    hello(&name);

    println!();
}
```

- 输出

```
>>>>> test own_smart_pointer
Hello superfree
```

## 3. 递归类型定义

然而上面的自定义 Box 类型有一个致命的缺陷：数据依旧保存在栈上

标准库自带的 Box 类型会将真实数据存入堆，不过由于 Rust 的引用跟踪非常好几乎不会产生内存泄漏，所以大多数时候我们会基于以下几个条件来判断是否需要用到 Box 类型来保存数据

- 占用大空间的数据对象：避免方法栈替换时的数据传输开销
- 递归类型定义：编译时无法确定数据大小，则以固定大小的栈指针代替

接下来我们来看看如何定义一个递归类型

首先我们先来看错误的例子

```rust
enum List {
    Cons(i32, List),
    Nil,
}
```

这样定义的一个问题在于，enum 会参照最大实例的对象划分空间，但是 `Cons` 将能够无限堆叠，变成一个占用无限空间的数据对象，因此我们需要将 `Cons` 的第二个数据改成一个固定大小的指针

```rust
#[derive(Debug)]
enum List {
    Cons(i32, Box<List>),
    Nil,
}
```

这样一来 Rust 就能确定 `Cons` 是一个 i32 + 一个 Box 引用，而第二个数据指向的对象是 Cons 还是 Nil，则是保存在堆上与栈空间的分配无关

- 完整例子：`/src/recursive_type.rs`

下面我们定义两种类型：`List、Node` 都是属于包含自身类型的递归定义类型，因此需要使用 Box 包装来保证编译时的大小确定

```rust
use crate::recursive_type::List::{Cons, Nil};

#[derive(Debug)]
enum List {
    Cons(i32, Box<List>),
    Nil,
}

#[derive(Debug)]
struct Node {
    val: i32,
    next: Box<Option<Node>>,
}
```

```rust
pub fn test() {
    println!(">>>>> test recursive_type");

    let list = Cons(3, Box::new(Cons(2, Box::new(Nil))));
    println!("list = {:?}", list);

    let mut node = Node {
        val: 1,
        next: Box::new(Some(Node {
            val: 2,
            next: Box::new(None),
        })),
    };
    println!("node = {:?}", node);
    *node.next = None;
    println!("node = {:?}", node);

    println!();
}
```

倒数第二句复习了一下 Box 的数据修改方法，使用 `*` 标识符并赋值新的数据对象

- 输出

```
>>>>> test recursive_type
list = Cons(3, Cons(2, Nil))
node = Node { val: 1, next: Some(Node { val: 2, next: None }) }
node = Node { val: 1, next: None }
```

接下来我们要来介绍几个更进阶的 Box 类型，他们都跟 Box 一样将数据保存在堆上，不过提供了更高级的特性

## 4. Box 进阶：Rc

第一种我们介绍的是 `Rc<T>` 类型。

我们知道 Rust 定义了一种特别的 Ownership 数据所有权的概念，也就是一个数据只能有一个人掌握，当我们想借用数据的时候就要用 `&、&mut` 等方法创建引用

然而大多时候 `&` 更多的是用在方法调用等暂时的借用上，当我们想让多个数据共同拥有一个数据项，也就是创建多个 Ownership 的时候就要用上 `Rc<T>` 类型

基于上面的 List 类型，想象我们有下面这样一个链表场景

![](https://picures.oss-cn-beijing.aliyuncs.com/img/rust_box_deep_dive_1_rc_sample.png)

这时候节点的存在并不依赖于一个 a，而是只要 a、b、c 三个有任何一个存活，节点 3 都应该继续存在，也就是三个指针同时共享了这个节点

`Rc<T>` 类型做的就是自动记录这个引用的数量，知道引用归 0 的时候才将数据对象删除，创建引用的时候推荐使用 `Rc::clone(&a)` 的写法

- `/src/reference_count.rs`

下面我们把前面的 List 数据类型从 Box 改成使用 Rc 来记录引用数量

```rust
use std::rc::Rc;
use crate::reference_count::List::{Cons, Nil};

#[derive(Debug)]
enum List {
    Cons(i32, Rc<List>),
    Nil,
}
```

接下来模拟上面那张图的引用方式，然后使用 `Rc::strong_count` 来检查当前引用数量

```rust
pub fn test() {
    println!(">>>>> test reference_count");

    let a = Rc::new(Cons(3, Rc::new(Cons(5, Rc::new(Nil)))));
    println!("a = {:?}", a);
    println!("a strong_count = {}", Rc::strong_count(&a));

    {
        let b = Rc::new(Cons(1, Rc::clone(&a)));
        println!("b = {:?}", b);
        println!("a strong_count = {}", Rc::strong_count(&a));
        {
            let c = Rc::new(Cons(2, Rc::clone(&a)));
            println!("c = {:?}", c);
            println!("a strong_count = {}", Rc::strong_count(&a));
        }
        println!("! drop c");
        println!("a strong_count = {}", Rc::strong_count(&a));
    }

    println!();
}
```

- 输出

```
>>>>> test reference_count
a = Cons(3, Cons(5, Nil))
a strong_count = 1
b = Cons(1, Cons(3, Cons(5, Nil)))
a strong_count = 2
c = Cons(2, Cons(3, Cons(5, Nil)))
a strong_count = 3
! drop c
a strong_count = 2
```

我们可以看到当 c 消失之后，a 的引用数量剩 2 了

## 5. Box 进阶：RefCell

第二种则是 RefCell，这个 RefCell 想要解决的问题是数据可变性的问题，由于 Rust 严格的所有权系统，使得我们想要修改数据的时候变得不那么方便，全部写成 `mut` 又显得格外丑陋，不符合 Rust 对于数据修改性最小化的理念

因此诞生出了一个 `RefCell` 类型，这样我们使用它的时候就好像在用普通的引用类型一样，并在必要的时候可以使用 `borrow_mut` 来创建一个可变引用。

这样一来并不是说 Rust 放弃了对于所有权的管理，而是将可变/不可变引用的检查从编译时改编为运行时检查

- `/src/reference_cell.rs`

官方手册给出的例子是在 Mock 测试的时候，我们需要在预备阶段手动进行数据赋值，但是测试过程又希望将数据作为不可变引用来使用

首先我们先定义一个接口，描述一个 Messenger 的行为

```rust
trait Messenger {
    fn send(&self, message: &str);
}
```

接下来我们定义一个 Mock 类型作为桩

```rust
#[derive(Debug)]
struct MockMessenger {
    send_messages: RefCell<Vec<String>>,
}

impl MockMessenger {
    fn new() -> MockMessenger {
        MockMessenger {
            send_messages: RefCell::new(vec![])
        }
    }
}

impl Messenger for MockMessenger {
    fn send(&self, message: &str) {
        let mut mut_sm = self.send_messages.borrow_mut();
        mut_sm.push(String::from(message));
        drop(mut_sm);
        println!("receive message = {:?}", self.send_messages.borrow());
    }
}
```

这里我们看到 `send_messages` 属性是一个 `RefCell` 类型，就允许我们在运行时动态的去借用可变引用，来修改内部数据

```rust
pub fn test() {
    println!(">>>>> test reference_cell");

    let mock_messenger = MockMessenger::new();
    mock_messenger.send("Hello");
    mock_messenger.send("World");
    println!("mock_messenger = {:?}", mock_messenger);
    println!("mock_messenger len = {}", mock_messenger.send_messages.borrow().len());

    println!();
}
```

- 输出

```
>>>>> test reference_cell
receive message = ["Hello"]
receive message = ["Hello", "World"]
mock_messenger = MockMessenger { send_messages: RefCell { value: ["Hello", "World"] } }
mock_messenger len = 2
```

最后我们可以看到数据成功的写入，但是又保持了 `send_messages` 属性的不可变性

不过从另一个角度来说，`RefCell` 会造成我们需要主动的去维护 `borrow_mut` 产生的可变引用，Rust 不再会在运行时为我们检查（可以将上面的 `drop(mut_sm);` 取消注释就能够看到产生了运行时错误）

## 6. 循环依赖 & Weak

有了 `RefCell` 之后我们是有可能动态的去改变不可变引用的数据并同时能避过编译器的检查，直到运行时抛出异常

这就产生了循环依赖的可能

- `/src/reference_cycles.rs`

首先我们先定义一个 List 类型，并赋予 tail 方法返回 `Cons` 类型的第二个数据引用

```rust
use std::cell::RefCell;
use std::rc::Rc;
use crate::reference_cycles::List::{Cons, Nil};

#[derive(Debug)]
enum List {
    Cons(i32, RefCell<Rc<List>>),
    Nil,
}

impl List {
    fn tail(&self) -> Option<&RefCell<Rc<List>>> {
        match self {
            Cons(_, item) => Some(item),
            Nil => None
        }
    }
}

impl Drop for List {
    fn drop(&mut self) {
        println!("drop {:?}", self);
    }
}
```

然而第二个数据类型是 `RefCell`，允许我们动态修改内部数据

```rust
pub fn test() {
    println!(">>>>> test reference_cycles");

    let a = Rc::new(Cons(5, RefCell::new(Rc::new(Nil))));
    println!("reference a = {:?}", a);
    println!("reference a count = {}", Rc::strong_count(&a));
    println!("reference a next = {:?}", a.tail());

    let b = Rc::new(Cons(10, RefCell::new(Rc::clone(&a))));
    println!("reference b = {:?}", b);
    println!("reference b count = {}", Rc::strong_count(&b));
    println!("reference b next = {:?}", b.tail());

    if let Some(link) = a.tail() {
        *link.borrow_mut() = Rc::clone(&b);
    }

    println!("reference a count = {}", Rc::strong_count(&a));
    println!("reference b count = {}", Rc::strong_count(&b));

    println!();
}
```

如此一来，在程序结束的时候 `a、b` 引用都消失了，但是两个对象互相持有对方的引用，因此内存并不会被回收

- 输出

```
>>>>> test reference_cycles
reference a = Cons(5, RefCell { value: Nil })
reference a count = 1
reference a next = Some(RefCell { value: Nil })
reference b = Cons(10, RefCell { value: Cons(5, RefCell { value: Nil }) })
reference b count = 1
reference b next = Some(RefCell { value: Cons(5, RefCell { value: Nil }) })
drop Nil
reference a count = 2
reference b count = 2
```

要想解决这个问题我们可以使用另一种引用类型：`Weak<T>` 弱引用类型。

Rust 提供了另一种与 `Rc` 对应的弱引用类型 `Weak`，记录引用数量为 `weak_count`，不同的是真实的数据对象只关心 `strong_count` 的数量并在 `strong_count` 归零的时候直接回收。

也就是每次调用 Weak 类型之前，需要调用 `upgrade` 来获取最新的引用对象

- `/src/reference_cycles_prevent.rs`

```rust
use std::cell::RefCell;
use std::rc::{Rc, Weak};

#[derive(Debug)]
struct Node {
    val: i32,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}

pub fn test() {
    println!(">>>>> test reference_cycles_prevent");

    let leaf = Rc::new(Node {
        val: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    println!("! create leaf");
    println!("leaf = {:?}", leaf);
    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
    println!("leaf strong = {}, weak = {}", Rc::strong_count(&leaf), Rc::weak_count(&leaf));
    {
        let branch = Rc::new(Node {
            val: 5,
            parent: RefCell::new(Weak::new()),
            children: RefCell::new(vec![Rc::clone(&leaf)]),
        });

        println!("! create branch");
        println!("branch = {:?}", branch);
        println!("branch parent = {:?}", branch.parent.borrow().upgrade());
        println!("leaf strong = {}, weak = {}", Rc::strong_count(&leaf), Rc::weak_count(&leaf));
        println!("branch strong = {}, weak = {}", Rc::strong_count(&branch), Rc::weak_count(&branch));

        *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

        println!("! ref branch");
        println!("leaf = {:?}", leaf);
        println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
        println!("leaf strong = {}, weak = {}", Rc::strong_count(&leaf), Rc::weak_count(&leaf));
        println!("branch strong = {}, weak = {}", Rc::strong_count(&branch), Rc::weak_count(&branch));
    }
    println!("! drop branch");
    println!("leaf = {:?}", leaf);
    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
    println!("leaf strong = {}, weak = {}", Rc::strong_count(&leaf), Rc::weak_count(&leaf));

    println!();
}
```

- 输出

```
>>>>> test reference_cycles_prevent
! create leaf
leaf = Node { val: 3, parent: RefCell { value: (Weak) }, children: RefCell { value: [] } }
leaf parent = None
leaf strong = 1, weak = 0
! create branch
branch = Node { val: 5, parent: RefCell { value: (Weak) }, children: RefCell { value: [Node { val: 3, parent: RefCell { value: (Weak) }, children: RefCell { value: [] } }] } }
branch parent = None
leaf strong = 2, weak = 0
branch strong = 1, weak = 0
! ref branch
leaf = Node { val: 3, parent: RefCell { value: (Weak) }, children: RefCell { value: [] } }
leaf parent = Some(Node { val: 5, parent: RefCell { value: (Weak) }, children: RefCell { value: [Node { val: 3, parent: RefCell { value: (Weak) }, children: RefCell { value: [] } }] } })
leaf strong = 2, weak = 0
branch strong = 1, weak = 1
! drop branch
leaf = Node { val: 3, parent: RefCell { value: (Weak) }, children: RefCell { value: [] } }
leaf parent = None
leaf strong = 1, weak = 0
```

上面的程序建立了一个父子节点关系

![](https://picures.oss-cn-beijing.aliyuncs.com/img/rust_box_deep_dive_2_weak_sample.png)

理论上按照前面的经验两个节点互相持有对方的引用应该也是一个内存泄漏，但是由于 parent 指针持有的是弱引用，因此在 branch 变量回收的时候 `strong_count` 变成 0 就会被回收了，同时 leaf 也会一并被回收了。

## 7. 结论

在 Rust 的理念里面，我们需要全面按照 Rust 的所有权规则来进行数据的操作、访问、共享。

对于堆上数据我们则需要使用 Box、Rc、Weak、RefCell 等类型，并使用如 `borrow`、`borrow_mut` 方法来获取引用指针等

使用上还是比较难懂，需要多练习hh。

# 其他资源

## 参考连接

| Title                                          | Link                                                                                                                     |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Smart Pointers - The Rust Programming Language | [https://doc.rust-lang.org/book/ch15-00-smart-pointers.html](https://doc.rust-lang.org/book/ch15-00-smart-pointers.html) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/back_end/rust/rust_box_deep_dive](https://github.com/superfreeeee/Blog-code/tree/main/back_end/rust/rust_box_deep_dive)
