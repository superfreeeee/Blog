# 设计模式: Proxy 代理模式

@[TOC](文章目录)

<!-- TOC -->

- [设计模式: Proxy 代理模式](#设计模式-proxy-代理模式)
  - [简介](#简介)
  - [参考](#参考)
  - [完整示例代码](#完整示例代码)
- [正文](#正文)
  - [场景](#场景)
  - [模式结构](#模式结构)
  - [代码示例](#代码示例)
    - [Subject 操作接口](#subject-操作接口)
    - [Real Subject 操作实体](#real-subject-操作实体)
    - [Proxy Subject 操作代理](#proxy-subject-操作代理)
    - [测试代码](#测试代码)
- [结语](#结语)

<!-- /TOC -->

## 简介

| 目的 | 创建型                                                                                | 结构型                                                                                                                         | 行为型                                                                                                                                                                                     |
| ---- | ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 类   | Factory Method 工厂方法                                                               | Adapter 适配器                                                                                                                 | Interpreter 解释器<br />Template Method 模版方法                                                                                                                                           |
| 对象 | Abstract Factory 抽象工厂<br />Builder 生成器<br />Prototype 原型<br />Singleton 单例 | Adapter 适配器<br />Bridge 桥接<br />Composite 组合<br />Decorator 装饰器<br />Facade 外观<br />Flyweight 享元<br />Proxy 代理 | Chain of Responsibility 职责链<br />Command 命令<br />Iterator 迭代器<br />Mediator 中介者<br />Memento 备忘录<br />Observer 观察者<br />State 状态<br />Strategy 策略<br />Visitor 访问者 |


## 参考

<table>
  <tr>
    <td>Design Patterns-Elements of Reusable Object-Oriented Software</td>
    <td><a href=""></a></td>
  </tr>
</table>

## 完整示例代码

<a href="https://github.com/superfreeeee/Blog-code/tree/main/design_pattern/src/main/java/com/example/proxy/classic">https://github.com/superfreeeee/Blog-code/tree/main/design_pattern/src/main/java/com/example/proxy/classic</a>

# 正文

## 场景

先说说代理模式的应用场景。在 OOP 的编程思想当中，通常我们习惯将操作和关联数据封装成一个**对象(Object)**，而我们操作的目标则是以对象为单位。然而很多时候我们可能并不希望直接访问对象，而是根据需要访问不同的**代理对象(Proxy)**：

1. 通过不同途径访问对象(二次请求、远程代理、编码) $\to$ $Remote Proxy \space 远程对象$
2. 对象缓冲(附加信息、静态信息缓冲) $\to$ $Virtual Proxy \space 虚拟对象$
3. 检查、过滤使用者权限(访问控制) $\to$ $Protection Proxy \space 保护对象$

当然代理对象还能有其他用途，然而不论基于什么目的，**代理**的本质在于：建立一个新的对象，包裹并代理目标对象的必要操作，并在真正调用目标对象方法过程中(调用前 or 后)加入额外的**其他操作、检查**等，这就是一种代理。

## 模式结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/proxy_pattern_structure.png)

- Subject 操作对象接口：定义目标对象的公共操作接口
- RealSubject 操作对象实体：真正执行核心操作的对象
- ProxySubject 操作对象代理：保存操作对象实体，并对实体的操作进行代理

代理模式的类型结构非常简单，我们对原本**直接操作的对象(Real Subject)**抽象出一个**公共的操作接口(Subject)**，并定义一个新的**代理对象(Proxy)**用于代理一个对象实体的操作。

## 代码示例

下面给出一个极其简单的代码示例，本身并不代表任何功能，但是实现代理模式的精神

### Subject 操作接口

```java
package com.example.proxy.classic;

public interface Subject {
    void f(); // method1
    
    void g(); // method2
}
```

我们一共定义了两种操作接口

### Real Subject 操作实体

首先我们先给出操作实体实现的具体操作

```java
package com.example.proxy.classic;

public class SubjectImpl implements Subject {
    @Override
    public void f() {
        System.out.println("invoke function f");
    }

    @Override
    public void g() {
        System.out.println("invoke function g");
    }
}
```

### Proxy Subject 操作代理

接下来的代理对象接受另一个操作对象作构造函数参数，这表示代理对象还能够多层嵌套的，我们调用一个代理对象的时候，调用的顺序可能如下图所示。

![](https://picures.oss-cn-beijing.aliyuncs.com/img/proxy_pattern_nesting_proxy.png)

其他操作接口仅仅是对构造时保存的对象的操作进行代理，额外操作的体现是在执行前多输出加一个前缀。

```java
package com.example.proxy.classic;

public class SubjectProxy implements Subject {

    private Subject subject;

    public SubjectProxy(Subject subject) {
        this.subject = subject;
    }

    private void prefix() {
        System.out.print("[Subject Proxy]");
    }

    @Override
    public void f() {
        prefix();
        subject.f();
    }

    @Override
    public void g() {
        prefix();
        subject.g();
    }
}
```

### 测试代码

最后给出测试代码和输出

```java
package com.example.proxy.classic;
import org.junit.Test;
import static org.junit.Assert.*;

public class SubjectTest {

    @Test
    public void test() {
        Subject subject = new SubjectImpl();
        Subject proxy = new SubjectProxy(subject);
        subject.f();
        subject.g();
        proxy.f();
        proxy.g();
    }
}
```

```
invoke function f
invoke function g
[Subject Proxy]invoke function f
[Subject Proxy]invoke function g
```

我们建立了一个操作实体，并用它来创建了一个代理对象。前两个调用直接调用原对象的方法，可以看到直接输出两个字符串；后两次调用则是透过代理对象进行调用，与设想的一样，多了代理对象的前缀。

# 结语

代理模式本质上是对操作对象的一种保护，不论是**访问远程对象**或是需要进行**访问管理控制**，或是需要对对象的操作**附带额外信息、进行额外的操作(记录、编码)**等，简单来说就是对真正的对象操作进行一层包裹、一层封装。

同时代理模式也引出一个 OOP 编程设计中非常重要的一条思想：面对接口编程。我们必须接受面对接口编程的好处，仅仅只需要知道对象的模样(look like)，而不是直接与对象的实际类型打交道，如此一来我们也就不需要区分操作对象的类型究竟是对象实体(Real Subject)还是代理对象(Proxy)。
