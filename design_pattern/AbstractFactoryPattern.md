# 设计模式: Abstract Factory 抽象工厂模式

@[TOC](文章目录)

<!-- TOC -->

- [设计模式: Abstract Factory 抽象工厂模式](#设计模式-abstract-factory-抽象工厂模式)
  - [简介](#简介)
  - [参考](#参考)
  - [完整示例代码](#完整示例代码)
- [正文](#正文)
  - [场景](#场景)
  - [模式结构](#模式结构)
  - [代码实现](#代码实现)
    - [Product 产品类](#product-产品类)
      - [Abstract Product 抽象产品类型](#abstract-product-抽象产品类型)
      - [Concrete Product 具体产品类型](#concrete-product-具体产品类型)
    - [Factory 工厂类](#factory-工厂类)
      - [Abstract Factory 抽象工厂类](#abstract-factory-抽象工厂类)
      - [Concrete Factory 具体工厂类](#concrete-factory-具体工厂类)
    - [Client 客户端](#client-客户端)
  - [测试代码](#测试代码)
- [结语](#结语)
  - [优点](#优点)
  - [缺点](#缺点)

<!-- /TOC -->

## 简介

| 目的 | 创建型                                                                                | 结构型                                                                                                                         | 行为型                                                                                                                                                                                     |
| ---- | ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 类   | Factory Method 工厂方法                                                               | Adapter 适配器                                                                                                                 | Interpreter 解释器<br />Template Method 模版方法                                                                                                                                           |
| 对象 | Abstract Factory 抽象工厂<br />Builder 生成器<br />Prototype 原型<br />Singleton 单例 | Adapter 适配器<br />Bridge 桥接<br />Composite 组合<br />Decorator 装饰器<br />Facade 外观<br />Flyweight 享元<br />Proxy 代理 | Chain of Responsibility 职责链<br />Command 命令<br />Iterator 迭代器<br />Mediator 中介者<br />Memento 备忘录<br />Observer 观察者<br />State 状态<br />Strategy 策略<br />Visitor 访问者 |

作为创建型设计模式中最广为人知的就是**抽象工厂**和**工厂方法**两种模式，又因为常常被简称为工厂模式使得许多人搞不太清楚两个之间的区别。本篇要介绍的是 **Abstract Factory 抽象工厂**模式。

## 参考

<table>
  <tr>
    <td>Design Patterns-Elements of Reusable Object-Oriented Software</td>
    <td><a href=""></a></td>
  </tr>
</table>

## 完整示例代码

<a href="https://github.com/superfreeeee/Blog-code/tree/main/design_pattern/src/main/java/com/example/abstract_factory/classic">https://github.com/superfreeeee/Blog-code/tree/main/design_pattern/src/main/java/com/example/abstract_factory/classic</a>

# 正文

## 场景

现在我们假设场景是我们需要创建一组产品对象(Product)，然而这组产品对象可能存在不同的风格、系列、行为、外观，因此不同系列的产品都会存在自己的一组产品实现。

例如我们现在需要生产的目标是一组可视化组件，我们可以抽象出以下组件类型

- Window 视窗组件
- ScrollBar 滚动条组件
- Menu 导航栏
- ...

而不同风格或是特性都存在自己的一组实现组件，例如我们可以创建一组 **PM(Presentation Manager 组件控制器)风格**的组件，一组**Motif(主题式)风格**的组件，因此对应不同的组件类型会存在以下实现

- PM 系列
  - PMWindow
  - PMScrollBar
  - PMMenu
  - ...
- Motif 系列
  - MotifWindow
  - MotifScrollBar
  - MotifMenu
  - ...

然而同系列的组件之间应该是共同协作的，也就是选定一个系列之后所创建的所有组件都应该属于一个系列的组件。

这时候我们就可以把组件的创建委托给一个**工厂(factory)**进行创建，每个系列存在一个对应的工厂类，而每个工厂都为每一个组件类型提供一个创建方法。如此一来只要是出自于同一个工厂的组件我们就能够保证它们都属于同一个系列的组件了。

## 模式结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/abstract_factory_pattern_structure.png)

上图的 UML 图明确的演示出 Abstract Factory 模式中各个角色之间的互动行为

- Abstract Product 抽象产品类：特定产品类型的抽象类
- ProductXxx 具体产品类：不同风格的具体产品类型
- Abstract Factory 抽象工厂类：声明所有生产同样一组产品的工厂的公共创建行为，创建的类型为 Abstract Product
- FactoryXxx 具体工厂类：具体实现产品创建的工厂，每个系列或是风格对应一个具体工厂类型
- Client 客户端：具体使用产品的类，不直接创建产品而是委托工厂创建产品，而使用的具体产品类型取决于实际的工厂类型

## 代码实现

接下来我们直接使用 **'工厂'生产'产品'** 为示例，多少有些抽象不过还是比较浅显易懂的。应用到实际业务的时候，需要根据具体要创建的所谓的'产品'进行抽象，再建立用于创建的'抽象工厂'

### Product 产品类

首先第一步当然就是创建我们最终的目标类型，也就是我们的**产品**

#### Abstract Product 抽象产品类型

我们创建出两种抽象产品类型

```java
/* 抽象产品类型：ProductA.java */
public interface ProductA {}

/* 抽象产品类型：ProductB.java */
public interface ProductB {}
```

#### Concrete Product 具体产品类型

产品的具体实现类型有两种风格：1 和 2

- type 1

```java
/* ProductA1.java */
public class ProductA1 implements ProductA {}

/* ProductB1.java */
public class ProductB1 implements ProductB {}
```

- type 2

```java
/* ProductA2.java */
public class ProductA2 implements ProductA {}

/* ProductB2.java */
public class ProductB2 implements ProductB {}
```

### Factory 工厂类

有了产品之后，我们接下来就可以创建工厂，负责所有产品的创建

#### Abstract Factory 抽象工厂类

抽象工厂作为所有工厂的接口规范，声明创建的产品是面对**抽象产品类型**而创建的，而创建出来的产品的实际类型则是由实际工厂类型所决定的。

```java
/* Factory.java */
public interface Factory {

    ProductA createProductA();

    ProductB createProductB();
}
```

#### Concrete Factory 具体工厂类

由于一共有两种产品系列，因此我们也要为每个系列创建对应的工厂类型(Factory1 对应 type1 系列的产品；Factory2 对应 type2 系列的产品)

```java
/* Factory1.java */
/* 用于创建 type1 系列的产品 */
public class Factory1 implements Factory {
    @Override
    public ProductA createProductA() { return new ProductA1(); }

    @Override
    public ProductB createProductB() { return new ProductB1(); }
}
```

```java
/* Factory2.java */
/* 用于创建 type2 系列的产品 */
public class Factory2 implements Factory {
    @Override
    public ProductA createProductA() { return new ProductA2(); }

    @Override
    public ProductB createProductB() { return new ProductB2(); }
}
```

### Client 客户端

最后我们就可以从客户端来委托工厂创建对象并取得不同实际类型的产品了

```java
/* Client.java */
public class Client {
    void buildSomething(Factory factory) {
        ProductA productA = factory.createProductA();
        ProductB productB = factory.createProductB();
        System.out.println("Factory: " + factory);
        System.out.println("ProductA: " + productA);
        System.out.println("ProductB: " + productB);
    }
}
```

## 测试代码

最后附上测试代码和结果

```java
/* ClientTest.java */
public class ClientTest {

    private Client client = new Client();

    @Test
    public void test_factory1() {
        client.buildSomething(new Factory1());
    }

    @Test
    public void test_factory2() {
        client.buildSomething(new Factory2());
    }
}
```

```
Factory: com.example.abstract_factory.type1.Factory1@50134894
ProductA: com.example.abstract_factory.type1.ProductA1@2957fcb0
ProductB: com.example.abstract_factory.type1.ProductB1@1376c05c
Factory: com.example.abstract_factory.type2.Factory2@694f9431
ProductA: com.example.abstract_factory.type2.ProductA2@f2a0b8e
ProductB: com.example.abstract_factory.type2.ProductB2@593634ad
```

# 结语

## 优点

使用抽象工厂有以下优点

1. 分离具体的类：工厂类封装了创建产品对象的职责，使得客户与具体实现的类分离
2. 易于交换产品系列：改变具体应用的产品类型只需要更换工厂类即可
3. 有利于产品的一致性：所有产品实现遵守产品接口(或继承共同抽象产品类)，所有实际产品有共同表现

## 缺点

然而简单直接使用抽象工厂可能会差生下列问题

1. 难支持新的产品种类：需要在抽象工厂类增加新的接口，并为所有工厂类添加新的实现
2. 容易产生类型爆炸：需要为每一个系列的每一个产品创建独立的类，代码复用性低；即便系列间产品类的差异很小还是必须创建独立的工厂类型实现

抽象工厂相对其他创建型模式来说是相当重量级的设计模式，下一篇将要介绍相对来说较为轻量级的**工厂方法(Factory Method)模式**。
