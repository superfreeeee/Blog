# 设计模式: Creational 创建型的 5 种设计模式总汇(TS 实现版本)

@[TOC](文章目录)

<!-- TOC -->

- [设计模式: Creational 创建型的 5 种设计模式总汇(TS 实现版本)](#设计模式-creational-创建型的-5-种设计模式总汇ts-实现版本)
- [前言](#前言)
  - [GoF 四人帮：设计模式集大成者](#gof-四人帮设计模式集大成者)
  - [设计模式分类](#设计模式分类)
- [正文](#正文)
  - [0. Creational 创建型模式简述](#0-creational-创建型模式简述)
    - [0.1 创建型模式的关注点](#01-创建型模式的关注点)
  - [1. Abstract Factory 抽象工厂模式](#1-abstract-factory-抽象工厂模式)
    - [1.1 模式背景](#11-模式背景)
    - [1.2 模式结构](#12-模式结构)
    - [1.3 模式实现代码示例](#13-模式实现代码示例)
      - [1.3.1 产品定义](#131-产品定义)
      - [1.3.2 工厂定义](#132-工厂定义)
      - [1.3.3 客户端](#133-客户端)
    - [1.4 模式特性小结](#14-模式特性小结)
      - [1.4.1 对象的职责](#141-对象的职责)
      - [1.4.2 抽象工厂的优劣](#142-抽象工厂的优劣)
  - [2. Builder 创建者模式](#2-builder-创建者模式)
    - [2.1 模式背景](#21-模式背景)
    - [2.2 模式结构](#22-模式结构)
    - [2.3 模式实现代码示例](#23-模式实现代码示例)
      - [2.3.1 产品定义](#231-产品定义)
      - [2.3.2 创建者定义](#232-创建者定义)
      - [2.3.3 导向器定义 & 使用示例](#233-导向器定义--使用示例)
    - [2.4 模式特性小结](#24-模式特性小结)
      - [2.4.1 对象的职责](#241-对象的职责)
      - [2.4.2 创建者模式的优劣](#242-创建者模式的优劣)
  - [3. Factory Method 工厂方法模式](#3-factory-method-工厂方法模式)
    - [3.1 模式背景](#31-模式背景)
    - [3.2 模式结构](#32-模式结构)
    - [3.3 模式实现代码示例](#33-模式实现代码示例)
      - [3.3.1 产品定义](#331-产品定义)
      - [3.3.2 创建者对象](#332-创建者对象)
      - [3.3.3 静态工厂方法](#333-静态工厂方法)
      - [3.3.4 运行结果](#334-运行结果)
    - [3.4 模式特性小结](#34-模式特性小结)
      - [3.4.1 对象的职责](#341-对象的职责)
      - [3.4.2 工厂方法小结](#342-工厂方法小结)
  - [4. Prototype 原型模式](#4-prototype-原型模式)
    - [4.1 模式背景](#41-模式背景)
    - [4.2 模式结构](#42-模式结构)
    - [4.3 模式实现代码示例](#43-模式实现代码示例)
      - [4.3.1 产品定义](#431-产品定义)
      - [4.3.2 运行示例](#432-运行示例)
    - [4.4 模式特性小结](#44-模式特性小结)
      - [4.4.1 对象的职责](#441-对象的职责)
  - [5. Singleton 单例模式](#5-singleton-单例模式)
    - [5.1 模式背景](#51-模式背景)
    - [5.2 模式结构](#52-模式结构)
    - [5.3 模式实现代码示例](#53-模式实现代码示例)
      - [5.3.1 静态变量实例](#531-静态变量实例)
      - [5.3.2 ES6 Proxy 代理实现单例模式](#532-es6-proxy-代理实现单例模式)
      - [5.3.3 ES6 Module 唯一实例](#533-es6-module-唯一实例)
      - [5.3.4 输出示例](#534-输出示例)
    - [5.4 模式特性小结](#54-模式特性小结)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

在面对对象的编程思想下(OOP = Object-Oriented Programming)，我们通常在使用所谓的 `class` 来定义一个对象的 **类**，而以 `object` 来表示运行时创立的 **对象** (或是称为 **实例** )。

然而从结构化编程到面向对象编程的转变是痛苦的，常常很多系统会迷失在不知道如何界定对象和对象的职责使得系统开发一次又一次的碰壁，随之而来的是一次又一次的重构。

## GoF 四人帮：设计模式集大成者

所幸我们已经有很多前辈为我们试过很多方案，最终由 GoF 四人帮总结出前人经过无数次的试错之后，提炼出来的 **23 种设计模式**。

所谓的设计模式其实就是一些通用的对象交互场景，以及对应的对象职责划分与设计。在处理相似的问题的时候我们就可以直接以适合的设计模式为基础来进行系统的设计。

## 设计模式分类

本篇就不再对设计模式继续说了，可以说个几千字也写不完。

23 种设计模式又被分为三大类：**Creational 创建型**、**Structural 结构型**、**Behavioral 行为型**


| 大类 | 创建型                                                                                | 结构型                                                                                                                         | 行为型                                                                                                                                                                                     |
| ---- | ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 类   | Factory Method 工厂方法                                                               | Adapter 适配器                                                                                                                 | Interpreter 解释器<br />Template Method 模版方法                                                                                                                                           |
| 对象 | Abstract Factory 抽象工厂<br />Builder 生成器<br />Prototype 原型<br />Singleton 单例 | Adapter 适配器<br />Bridge 桥接<br />Composite 组合<br />Decorator 装饰器<br />Facade 外观<br />Flyweight 享元<br />Proxy 代理 | Chain of Responsibility 职责链<br />Command 命令<br />Iterator 迭代器<br />Mediator 中介者<br />Memento 备忘录<br />Observer 观察者<br />State 状态<br />Strategy 策略<br />Visitor 访问者 |

今天要给大家带来的是创建型的 5 种设计模式总汇，使用 TypeScript 语言实现(你会发现 TS 的实现与 Java 有着惊人的相似之处)

# 正文

## 0. Creational 创建型模式简述

在开始具体的一个个模式之前，我们先来解释一下什么叫 **创建型(Creational)** 的设计模式。

我们知道，在 OOP 的编程思想下，一段程序或是说一个系统总是由很多 **对象** 组成，而这些所谓的对象实际上是将数据与相关的操作逻辑进行 **封装**。

那么本篇要介绍的创建型设计模式就是关于如何 **创造** 对象的 5 种常见形式

### 0.1 创建型模式的关注点

那么创建反正就是用 `new` 或是其他分配内存的语法就是了呗？为啥还要搞什么模式？

事实上任何系统在进行对象的创建时都必须注意以下条件

- 对象类型的确定：如何定位对象的类型定义(`class`)，并且尽可能的模糊实际对象的构造方法，以此来提高类型的可复用性
  - 类型的构造方法：构造函数、构造参数、精确类型
  - 类型的位置：类型定义的位置、类型的加载
- 对象类型的创建：由于系统的资源是有限的，所以我们应该对 **"分配内存"** 的对象创建行为进行封装，或是使用其他手段来对系统资源的使用进行管理
  - 对象创建形式：直接分配内存并初始化、对象先有对象克隆生成
  - 对象数量限制：限定单一类型的实例数量、根据不同时机构造新的实例化
- 对象的构建：在系统中可能会存在一些特别复杂的对象，甚至需要经过多个步骤才能够完全创建完毕，所以针对复杂对象的构建我们可以透过一些模式来进行抽象

实际上直接看模式都有些抽象，具体多写一些 OOP 的代码再回来看设计模式会更了解它到底想干嘛。下面我们进入正题，介绍具体 5 种创建型模式的背景与细节。

## 1. Abstract Factory 抽象工厂模式

### 1.1 模式背景

第一种是 **抽象工厂模式(Abstract Factory)**，使用抽象工厂模式的主要背景在于：

> - 创建目标对象分为多个类型
> - 存在不同风格的产品针对不同产品的实现

有点抽象，再白话一点：

- 系统将要使用"一组"对象来实现一些功能
- 我们可能需要改变"风格"，也就是存在"很多"个对象组合，每个风格需要与相同风格的对象共同作用

在这个模式中我们通常将要创建的对象比喻为一种 **产品**，然后我们将创造产品的责任委托给所谓的 **工厂** 来执行，下面我们就来看看模式中的重要对象

### 1.2 模式结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/abstract_factory_pattern_structure.png)

- Abstract Product 抽象产品：外部可见的产品共同行为表现 **接口**
- Concrete Product 具体产品：实现产品的 **具体逻辑**
- Abstract Factory 抽象工厂：定义构建产品的 **接口**
- Concrete Factory 具体工厂：定义产品的 **具体创建行为**，即确定实际产品的类型
- Client 客户端：使用工厂来创建，并最终使用产品的对象

在抽象工厂模式中，客户端总是只能调用抽象工厂/产品的接口，因为对于客户端来说重要的是创建了什么产品(工厂的接口)，而具体的产品"风格"则是被隐藏在具体工厂类与具体产品类当中。

下面我们看看代码实践

### 1.3 模式实现代码示例

#### 1.3.1 产品定义

- `/src/creational/abstract_factory/products.ts`

- 抽象产品类型：A、B 两种

```ts
export interface ProductA {}
export interface ProductB {}
```

-  具体产品：风格分为 1、2 两种

```ts
// 具体产品类型 A
export class ProductA1 implements ProductA {}
export class ProductA2 implements ProductA {}

// 具体产品类型 B
export class ProductB1 implements ProductB {}
export class ProductB2 implements ProductB {}
```

#### 1.3.2 工厂定义

- `/src/creational/abstract_factory/factorys.ts`

- 抽象工厂的定义如下

```ts
export interface Factory {
  createProductA: () => ProductA
  createProductB: () => ProductB
}
```

一个工厂有两个方法，负责创建两种产品

- 生产产品风格 1 的工厂

```ts
export class Factory1 implements Factory {
  createProductA() {
    return new ProductA1()
  }

  createProductB() {
    return new ProductB1()
  }
}
```

- 生产产品风格 2 的工厂

```ts
export class Factory2 implements Factory {
  createProductA() {
    return new ProductA2()
  }

  createProductB() {
    return new ProductB2()
  }
}
```

我们可以看到实际上由具体工厂决定了产品风格，也就是一个工厂会负责同一种风格下的所有产品

#### 1.3.3 客户端

最后给出客户端的测试代码和输出结果

- `/src/creational/abstract_factory/index.ts`

```ts
group('factory1', () => {
  const factory1: Factory = new Factory1()

  const productA1 = factory1.createProductA()
  const productB1 = factory1.createProductB()

  log('productA1:', productA1)
  log('productB1:', productB1)
})

group('factory2', () => {
  const factory2: Factory = new Factory2()

  const productA2 = factory2.createProductA()
  const productB2 = factory2.createProductB()

  log('productA2:', productA2)
  log('productB2:', productB2)
})
```

- 输出结果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_creational_console_abstract_factory.png)

### 1.4 模式特性小结

最后我们针对抽象工厂模式进行一个小结

#### 1.4.1 对象的职责

- 对于用户来说：使用抽象工厂的接口，创建一个抽象产品来使用
- 对于工厂来说：实现接口定义，负责封装对于同样风格而不同种类产品的创建
- 对于产品来说：不同风格的产品实现不同风格的封装，对外暴露相同的抽象产品接口

#### 1.4.2 抽象工厂的优劣

- 特点/优点
  - 将产品的具体实现与接口分离
  - 将产品类型透过工厂实现分离
  - 产品风格改变、扩展容易(增加新的产品实现与工厂实现)
- 缺点/劣势
  - 难以扩展产品的种类(需要同时修改产品接口、所有风格的产品实现、所有工厂实现)
  - 容易产生类型爆炸(新的产品风格可能差异很小，但是还是要实现重新为该风格创建所有种类的产品，同时创建一个具体的工厂类来生产)

## 2. Builder 创建者模式

### 2.1 模式背景

第二个我们要介绍的是 **创建者模式(Builder)**，它适用的场景如下：

> 将复杂对象的组件实现与组件组合逻辑相互分离，允许用户创建不同组合逻辑，搭配不同产品风格

白话文：

产品的创建由一个对象负责，而产品的组合由另一个对象负责，最终生产出一个复杂的组合型产品

### 2.2 模式结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/builder_pattern_structure.png)

- Product 产品：客户端最终需要使用的 **具体产品**
- Builder 创建者接口：提供 **创建、组合** 产品的 **接口**
- Concrete Builder 具体创建者：实际 **实现** 产品的创建、组合
- Director 导向器：调用创建者接口对产品进行组装

### 2.3 模式实现代码示例

#### 2.3.1 产品定义

- `/src/creational/builder/products.ts`

首先我们定义最终产品的接口和部件类型

```ts
export interface Component {}

export interface Product {
  componentA?: Component
  componentB?: Component
  componentC?: Component
}
```

接下来我们创建三种实际部件类型，用于组装成实际产品

```ts
export class ComponentA implements Component {}
export class ComponentB implements Component {}
export class ComponentC implements Component {}
```

实际上这里使用的部件可以是任意形式的内部类型，甚至只是对于现有组合方式的排列也可以作为一种可选的"组装操作"

#### 2.3.2 创建者定义

- `/src/creational/builder/builder.ts`

接下来是创建者的公有接口

```ts
export interface Builder {
  setComponentA: (component: Component) => this
  setComponentB: (component: Component) => this
  setComponentC: (component: Component) => this
  build: () => Product
}
```

定义三个用于组装部件的方法接口(再次强调，这里的"组装操作"可以是任意类型)

下面则是具体的创建者方法实现

```ts
export class ConcreteBuilder implements Builder {
  product: Product = {}

  setComponentA(component: Component) {
    this.product.componentA = component
    return this
  }

  setComponentB(component: Component) {
    this.product.componentB = component
    return this
  }

  setComponentC(component: Component) {
    this.product.componentC = component
    return this
  }

  build() {
    return this.product
  }
}
```

#### 2.3.3 导向器定义 & 使用示例

- `/src/creational/builder/builder.ts`

最后就是我们的导向器，用于调用创建者接口来描述组装产品的逻辑

```ts
class Director {
  buildProductInOrder(builder: Builder): Product {
    return builder
      .setComponentA(new ComponentA())
      .setComponentB(new ComponentB())
      .setComponentC(new ComponentC())
      .build()
  }

  buildProductBackward(builder: Builder): Product {
    return builder
      .setComponentC(new ComponentA())
      .setComponentB(new ComponentB())
      .setComponentA(new ComponentC())
      .build()
  }
}
```

我们可以看到这里定义的导向器描述了两种产品组装逻辑(一种是顺序 A、B、C 组装，一种是逆序按 C、B、A 组装)

最后使用导向器传入具体的创建者对象，生产两种不同的组合产品

```ts
const director = new Director()
const productInOrder = director.buildProductInOrder(new ConcreteBuilder())
log('productInOrder:', productInOrder)
const productBackward = director.buildProductBackward(new ConcreteBuilder())
log('productBackward:', productBackward)

```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_creational_console_builder.png)

### 2.4 模式特性小结

#### 2.4.1 对象的职责

- 对于创建者来说：对外提供灵活而基础的产品组合接口，更复杂可实现成另一种的 DSL 语言来增强接口的表达能力
- 对于导向器来说：面对创建者提供的接口来描述具体产品的不同组合逻辑，相当于是对复杂产品的创建步骤进行封装

#### 2.4.2 创建者模式的优劣

- 特性
  - **容易修改产品的内部表示**：具体的产品构造被封装成创建者的接口实现，与导向器描述的组合逻辑完全分离
  - **构造代码与表示分离**：由于导向器是面对创建者的接口编程，所以实际上是面对产品的高层描述进行逻辑上的组合，所以可以很轻易的透过扩展导向器的方法来复用基础的创建者接口构造新的复杂产品对象
  - **对构造过程有更精细的控制**：与抽象工厂不同的是，创建者模式下的产品创建不再是对构造函数的简单调用，而是分为基础部件构造与部件组合两步骤，对产品对象实现更细粒度的控制

## 3. Factory Method 工厂方法模式

### 3.1 模式背景

第三种是所谓的 **工厂方法模式(Factory Method)**，一看就知道跟抽象工厂方法有点关联。

实际上工厂方法可以看成是抽象工厂的退化：

- 对于抽象工厂：一个工厂负责一系列产品的创建
- 对于工厂方法：一个方法表述对于一个产品的创建

实际上工厂方法的核心就是：

> 将实际的对象创建行定义为抽象方法，由具体的实现类实现具体产品的创建

白话文：

为基类留下一个创建对象的洞(工厂方法)，等待具体的子类来实现工厂方法创建真正的对象

### 3.2 模式结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/factory_method_pattern_structure.png)

- Product 抽象产品：客户端可能使用的产品类型
- Concrete Product 具体产品：与工厂方法一样，对于抽象产品的具体实现
- Creator 抽象创建基类：定义除了具体对象创建外的所有对象使用方法，并留下工厂方法接口
- Concrete Creator 具体创建类：实现工厂方法创建实际产品对象

简单来说工厂方法将创建对象的抽象接口由一整个工厂缩小为一个抽象方法(也可以说整个抽象工厂就是由多个工厂方法组成)；同时基类还能够实现多个使用对象的具体逻辑，并等待基类提供工厂方法的实现，来完成完整的对象逻辑

### 3.3 模式实现代码示例

#### 3.3.1 产品定义

- `/src/creational/factory_method/products.ts`

一样首先我们先定义要创建的目标产品

```ts
export interface Product {}

export class ProductA implements Product {}
export class ProductB implements Product {}
```

#### 3.3.2 创建者对象

- `/src/creational/factory_method/creators.ts`

接下来就是对于 Creator 基类和具体类的定义

```ts
export abstract class Creator {
  abstract createProduct(): Product

  operation() {
    const product = this.createProduct()
    log(`product:`, product)
  }
}
```

在基类中定义 `operation` 引用工厂方法 `createProduct` 创建对象并在此基础之上进行操作

下面则是不同的具体创建者实现工厂方法时，提供不同的具体产品类型

```ts
export class CreatorA extends Creator {
  createProduct() {
    return new ProductA()
  }
}

export class CreatorB extends Creator {
  createProduct() {
    return new ProductB()
  }
}
```

#### 3.3.3 静态工厂方法

除了前面提过的工厂方法之外，还有一个实际应用中也很常见的静态工厂方法，简单来说就是定义一个全局/静态的方法用于对象的创建(在 JS 中可以直接定义一个全局方法作为工厂方法)，具体定义如下

- `/src/creational/factory_method/static.ts`

```ts
export type ProductCreator = () => Product

export const createProductA: ProductCreator = () => {
  return new ProductA()
}

export const createProductB: ProductCreator = () => {
  return new ProductB()
}
```

#### 3.3.4 运行结果

最后的最后我们看到如何使用一般的工厂方法创建者基类，已经静态工厂方法的使用

- `/src/creational/factory_method/index.ts`

```ts
group('test Creator', () => {
  function testCreator(creator: Creator) {
    creator.operation()
  }

  testCreator(new CreatorA())
  testCreator(new CreatorB())
})

group('test static Creator', () => {
  function testStaticCreator(creator: ProductCreator) {
    const product = creator()
    log('product:', product)
  }

  testStaticCreator(createProductA)
  testStaticCreator(createProductB)
})
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_creational_console_factory_method.png)

### 3.4 模式特性小结

#### 3.4.1 对象的职责

- 对于创建者类型来说：其实就跟一般的业务逻辑对象没什么区别，问题在于操作对象的来源源自于一个抽象的工厂方法，而这个工厂方法要等待具体的实现子类，创建真正的具体产品并返回
- 对于创建者子类：实际上所有创建者的行为都写在基类里面，子类只需要提供具体对象创建结果，将具体对象的创建过程进行封装

#### 3.4.2 工厂方法小结

- 特性
  - **为子类提供钩子(hook)函数**：由于基类的业务逻辑其实依赖于抽象的工厂方法所创建的对象，相当于变相的提供一个 **钩子(hook)** 并暴露给子类，让子类有机会介入基类的业务流程当中
  - **平行的类层次**：与抽象抽象工厂将对象的创建与使用完全分离，工厂方法反倒是将创建对象的工厂方法与操作对象的其他方法一起放到了创建对象的创建者类型(Creator)当中，也就使得普通操作可以向看到内部方法一半重复或是观测构造方法的调用

## 4. Prototype 原型模式

### 4.1 模式背景

**原型模式(Prototype)** 与前面三种方法有些不同，它适用的场景不再是直接或间接的调用某个工厂方法/部件组合方法来创建对象；相反的，他则是使用产品对象 **自克隆(clone)** 的方法来完成对象的创建，简单来说就是一句话：

> 我不知道要用什么方法来创建对象，但是我有我要创建的对象的实例/模版，复制一份就得了

下面看看模式的结构

### 4.2 模式结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/prototype_pattern_structure.png)

在原型模式中，我们透过"复制"现有产品的形式来创建新的对象，所以我们也会将所有产品对象视为一个"可克隆"的对象

- Prototype 产品接口(可克隆对象接口)：为产品基类(声明克隆接口)
- Concrete Prototype 具体产品：具体产品类，实现克隆时的逻辑
- Client 客户端：客户端透过调用现有对象的克隆函数来创建新的对象

### 4.3 模式实现代码示例

#### 4.3.1 产品定义

- `/src/creational/prototype/products.ts`

首先是产品接口(可克隆对象接口)，最重要的就是要声明 **克隆(clone)** 方法的接口

```ts
export interface Product {
  clone: () => Product
}
```

接下来我们实现三个具体产品类型，模拟任意层次设计的产品对象，都要实现克隆接口

```ts
export class ProductA implements Product {
  static count: number = 0
  origin: number
  id: number

  constructor(proto: ProductA | void) {
    this.origin = proto ? proto.id : -1
    this.id = ProductA.count++
  }

  clone() {
    return new ProductA(this)
  }
}

export class ProductB extends ProductA {
  static count: number = 0
  origin: number
  id: number

  constructor(proto: ProductB | void) {
    super()
    this.origin = proto ? proto.id : -1
    this.id = ProductB.count++
  }

  clone() {
    return new ProductB(this)
  }
}

export class ProductC implements Product {
  static count: number = 0
  origin: number
  id: number

  constructor(proto: ProductC | void) {
    this.origin = proto ? proto.id : -1
    this.id = ProductC.count++
  }

  clone() {
    return new ProductC(this)
  }
}
```

#### 4.3.2 运行示例

- `/src/creational/prototype/index.ts`

运行示例比较简单，通常在原型模式下，会存在所谓的 **产品注册表(registry)** 一般的存在，他可能是一个全局的对象池(如 Java Bean)，也可能是隶属于一个范围之内的对象注册/缓存表

不论如何本篇采用一个简单的全局映射表作为对象注册表

```ts
const products = new Map<string, Product>()
products.set('a', new ProductA())
products.set('b', new ProductB())
products.set('c', new ProductC())

log('products:', products)

const productA = products.get('a')?.clone()
log('productA:', productA)

const productB = products.get('b')?.clone()
log('productB:', productB)

const productC = products.get('c')?.clone()
log('productC:', productC)
```

我们可以看到当我们需要一个新的对象的时候，就是先找到我们需要的类型实例，然后克隆一下就能创建出新的对象了

![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_creational_console_prototype.png)

### 4.4 模式特性小结

#### 4.4.1 对象的职责

- 对于产品来说：产品除了描述自身的业务逻辑和意义之外，就是要实现并对外暴露一个克隆的接口(`clone`)，以提供根据自身示例创建新对象的方法(当然这样其实与对当前对象生成一个快照的行为非常类似，差别在于克隆之后的对象将作为新的对象投入系统，而不是简单的备份或是复原用的原始数据)
- 对客户端来说：产品的创建就像细胞分裂一样，总是透过克隆某个现有对象的方式进行创建。

## 5. Singleton 单例模式

### 5.1 模式背景

最后一个模式与前面四个模式都不太相同，**单例模式(Singleton)** 关注的重点只有一个：

> 一个类型只能存在一个运行实例

当然也可以扩展成有限个运行实例，但是其本质上的最终目的就是要 **控制系统运行时的实例数量**

### 5.2 模式结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/singleton_pattern_structure.png)

- Singleton 单例类型：需要实现单例模式的产品类型

我们访问单例实例的时候通常是透过 `static getInstance` 的静态方法来获取唯一实例，然而在 JS 中我们有多种不同的做法

### 5.3 模式实现代码示例

#### 5.3.1 静态变量实例

第一种是将唯一的实例放在类型的静态属性 `_instance` 上，之后我们可以透过静态的 `static getInstance()` 方法或是直接访问 `_instance` 属性来获取唯一的实例

- `/src/creational/prototype/products.ts`

```ts
/********** singleton by static instance **********/
// access by ProductWithStaticInstance.getInstance()
export class ProductWithStaticInstance {
  static _instance: ProductWithStaticInstance | undefined

  constructor() {
    log('ProductWithStaticInstance created')
  }

  static getInstance(): ProductWithStaticInstance {
    if (!ProductWithStaticInstance._instance) {
      ProductWithStaticInstance._instance =
        new ProductWithStaticInstance()
    }
    return ProductWithStaticInstance._instance
  }
}
```

#### 5.3.2 ES6 Proxy 代理实现单例模式

第二种我们利用了 ES6 的 Proxy 特性，来拦截对于实例变量的访问，并确保实例被唯一创建(懒创建：第一次访问时才真正创建对象)

- `/src/creational/prototype/products.ts`

```ts
/********** singleton by Proxy using registry **********/
class ProductWithProxy {
  constructor() {
    log('ProductWithProxy created')
  }
}

const registry2: { instance: null | ProductWithProxy } = {
  instance: null,
}

// access by instanceWithProxy.instance
export const instanceWithProxy = new Proxy(registry2, {
  get(target, key, receiver) {
    if (key !== 'instance') return null
    let instance = Reflect.get(target, key, receiver)
    if (!instance) {
      instance = new ProductWithProxy()
      Reflect.set(target, key, instance, receiver)
    }
    return instance
  },
})
```

#### 5.3.3 ES6 Module 唯一实例

最后一种我们利用 ES Module 的特性，对于一个模块导出的变量的多次引用，实际上都是引用到原始定义模块中的唯一变量

- `/src/creational/prototype/products.ts`

```ts
/********** singleton by ES6 Module **********/
// directly access
export const instanceWithModule = new (class ProductWithModule {
  constructor() {
    log('ProductWithModule created')
  }
})()
```

这个方法与前一种的差异在于：

- Proxy 实现：懒创建，第一次访问时才真正创建对象
- ESM 实现：模块加载的同时创建实例

#### 5.3.4 输出示例

最后看一下对于单例对象的访问(包含三种实现模式)

- `/src/creational/prototype/index.ts`

```ts
group('ProductWithStaticInstance', () => {
  log('access 1:', ProductWithStaticInstance.getInstance())
  log('access 2:', ProductWithStaticInstance.getInstance())
  log('access 3:', ProductWithStaticInstance.getInstance())
})

group('ProductWithProxy', () => {
  log('access 1:', instanceWithProxy.instance)
  log('access 2:', instanceWithProxy.instance)
  log('access 3:', instanceWithProxy.instance)
})

group('ProductWithModule', () => {
  log('access 1:', instanceWithModule)
  log('access 2:', instanceWithModule)
  log('access 3:', instanceWithModule)
})
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_creational_console_singleton.png)

### 5.4 模式特性小结

- 特性
  - **对于全局唯一(有限个)实例的受控访问**：所有对于单例对象的访问必定经过一定程度的控制，才能保证对于全局唯一实例的维护完整性
  - **缩小命名空间**：透过将唯一实例挂载到类的静态属性上，来避免为单例定义一个全局变量而污染命名空间
  - **允许操作和表示的精化**：除了限制实例对象唯一存在之外，还能够很灵活的随时动态切换当前存在的实际实例类型

# 结语

本篇对于 5 种创建型模式有相对全面的讨论，供大家参考～

# 其他资源

## 参考连接

| Title                                                           | Link                                                                                                                           |
| --------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| Design Patterns - Elements of Reusable Object-Oriented Software | []()                                                                                                                           |
| ts中class类讲解                                                 | [https://blog.csdn.net/thickhair_cxy/article/details/108893456](https://blog.csdn.net/thickhair_cxy/article/details/108893456) |
| typescript(五)--ts中抽象类、继承、多态                          | [https://blog.csdn.net/jasnet_u/article/details/81144130](https://blog.csdn.net/jasnet_u/article/details/81144130)             |
|                                                                 | []()                                                                                                                           |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/design_pattern_js/src/creational](https://github.com/superfreeeee/Blog-code/tree/main/design_pattern_js/src/creational)
