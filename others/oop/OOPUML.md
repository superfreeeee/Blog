# OOP 面对对象: 一次搞懂 UML 类图到底怎么画

@[TOC](文章目录)

<!-- TOC -->

- [OOP 面对对象: 一次搞懂 UML 类图到底怎么画](#oop-面对对象-一次搞懂-uml-类图到底怎么画)
- [前言](#前言)
- [正文](#正文)
  - [1. 类图包含元素](#1-类图包含元素)
    - [1.1 实体：类、抽象类、接口](#11-实体类抽象类接口)
    - [1.2 类属性/方法访问描述符](#12-类属性方法访问描述符)
    - [1.3 类与类之间的关系](#13-类与类之间的关系)
  - [2. 类与类之间的关系](#2-类与类之间的关系)
    - [2.1 关联 Association](#21-关联-association)
    - [2.2 聚合 Aggregation](#22-聚合-aggregation)
    - [2.3 组合 Composition](#23-组合-composition)
    - [2.4 依赖 Dependency](#24-依赖-dependency)
    - [2.5 泛化(继承) Generalization](#25-泛化继承-generalization)
    - [2.6 实现 Realization](#26-实现-realization)
    - [2.7 类与类之间的关系小结](#27-类与类之间的关系小结)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

今天来介绍在 OOP 面对对象编程中常用到的，用于描述对象之间的关系的 **类图(Class Diagram)**。

# 正文

## 1. 类图包含元素

首先我们先看看在类图中存在哪些核心要素

### 1.1 实体：类、抽象类、接口

类图中的实体不外乎就是 **类(Class)**，同时针对其抽象程度，又能再划分出 **抽象类(Abstract Class)** 与 **接口(Interface)**

![](https://picures.oss-cn-beijing.aliyuncs.com/img/oop_uml_class_diagram_classes.png)

### 1.2 类属性/方法访问描述符

而在类中的属性(成员变量)、方法，又存在所谓的 **访问描述符(Access Descriptor)**，也就是指该变量or方法对于其他类的可见性，常用的分为以下三种

- `-` 表示 private 私有，其他类不可访问
- `#` 表示 protected 保护，只有子类可访问
- `+` 表示 public 公有，所有类都可访问
- `~`(很少用) 表示 package 包，表示属于同一个包的类可用

![](https://picures.oss-cn-beijing.aliyuncs.com/img/oop_uml_class_diagram_field_modifiers.png)

### 1.3 类与类之间的关系

有了实体和实体的属性(包括成员变量和方法)之后，最后一个重要的元素就是描述 **类与类之间的关系**，在类图中我们通常使用不同样式的线段来表示

![](https://picures.oss-cn-beijing.aliyuncs.com/img/oop_uml_class_diagram_entity_relationships.png)

下面一个小结我们将一一介绍不同关系的含义

## 2. 类与类之间的关系

如上图我们看到，类与类之间的关系分成 6 种：

- 关联 Association
- 聚合 Aggregation
- 组合 Composition
- 依赖 Dependency
- 泛化 Generalization
- 实现 Realization

下面我们一个个来看各自的含义

### 2.1 关联 Association

第一种是 **关联 Association**，简单来说所谓的关联就是：

> 一个类持有对另一个类的引用称为 **关联**

依据不同的持有场景，又可以分为

1. 单向关联

![](https://picures.oss-cn-beijing.aliyuncs.com/img/oop_uml_class_diagram_entity_relationship_association.png)

一个用户(Account)拥有一个地址(Address)，也就是用户类单方面"持有"地址类的引用，这时候我们就从 Account 画一条实心箭头指向 Address

2. 双向关联

![](https://picures.oss-cn-beijing.aliyuncs.com/img/oop_uml_class_diagram_entity_relationship_association2.png)

有时候也会产生双向关联的场景如上图，这时候我们可以选择两端都加上箭头或是两端都不加上箭头

上面的例子就是一个顾客(Customer)持有购买的产品列表(products)，而每个产品对象(Product)持有一个购买者(buyer)的引用

3. 自关联

![](https://picures.oss-cn-beijing.aliyuncs.com/img/oop_uml_class_diagram_entity_relationship_association3.png)

当然，引用与自己同类型的不同对象也是非常容易想象的。如上例就是指一个节点(Node)可能持有其子节点数组(children)的引用

### 2.2 聚合 Aggregation

第二种关系是 **聚合 Aggregation**。相较于关联(Association)的持有引用的概念，聚合(Aggregation)则存在更严格的限制：

> 引用与被引用的对象必须存在 **包含** 关系

也就是说引用者应该属于一个范围比较大的类型，而被引用者应该属于引用者的一部分或是组成

![](https://picures.oss-cn-beijing.aliyuncs.com/img/oop_uml_class_diagram_entity_relationship_aggregation.png)

如例子中我们看到一个学校可能拥有很多个学生，我们也可以说一间学校是由很多个学生组成。如此一来学校与学生就产生"包含"关系

### 2.3 组合 Composition

第三种的 **组合 Composition** 又与第二种的聚合(Aggregation)，非常相似。不同的地方在于：

- 对于 **聚合** 来说：被包含的类是可以独立存在的
- 对于 **组合** 来说：被包含的类是 **不可以** 独立存在的

也就是说在组合当中，被包含的对象一离开宿主类型就失去意义

![](https://picures.oss-cn-beijing.aliyuncs.com/img/oop_uml_class_diagram_entity_relationship_composition.png)

如上图，一颗头一定属于一个人，这个头只有作为人的头才有意义。不同于前一个例子中，一个学生离开了学校，还是可以作为一名学生存在。

### 2.4 依赖 Dependency

前面提到的关系主要都是持有引用(关联)、包含关系(聚合、组合)，接下来的第四个关系 **依赖 Dependency** 属于一个比较弱的关系。

当我们在一个对象的类当中使用到了另一个对象的引用(参数、成员变量、静态方法调用)，这时候我们就说该类 **依赖** 与另一个对象，因为 **该对象的实现是依赖于被使用的类的实现的**

![](https://picures.oss-cn-beijing.aliyuncs.com/img/oop_uml_class_diagram_entity_relationship_dependency.png)

如上图我们看到，驾驶(Driver)与车(Car)并没有直接持有或是任何包含关系，但是一个驾驶要开车的时候就一定会用到一台车，这时候我们就可以说一个驾驶员的行为 **依赖** 与车，或是说依赖于车的实现

### 2.5 泛化(继承) Generalization

后面两个关系反而是大家比较熟悉的 OOP 编程非常常见的继承。

然而在 UML 中我们通常采用 **泛化 Generalization** 这种说法，因为不同语言对于类型系统的实现不尽然相同，有的可以多继承(C++)有的不行(Java)，有的采用了原型链(JavaScript)来实现类型系统。

所以我们可以统一采用 **泛化(Generalization)** 的说法，所谓的"子类"仅仅只是对于父类的 **扩展**，也就是对父类的能力进行泛化，使其**具备更多特征和能力**

![](https://picures.oss-cn-beijing.aliyuncs.com/img/oop_uml_class_diagram_entity_relationship_generalization.png)

我们可以看到人类(Human)作为基类，持有并描述作为一个人所具有的属性(姓名、年龄)和能力(走路、说话)；而我们再定义两个子类用来更具体的描述特定族群的人类：

- 学生拥有自己专属的学号(studentId)以及行为(学习，当然任何年龄都能学，可能改成考试会更好hh)
- 老师也拥有自己专属的教职工号(teacherId)以及行为(教学)

### 2.6 实现 Realization

最后一种则是对于接口的 **实现 Realization**。前面提到的泛化是一种对于现有类型的扩展、泛化，而有的时候我们会定义没有实现方法的 **抽象接口(interface)**，而不同的对象实现该接口并提供接口方法的具体行为后，我们就可以把这几个类视为具有相同行为的类型(接口类型)，这就是所谓的实现

![](https://picures.oss-cn-beijing.aliyuncs.com/img/oop_uml_class_diagram_entity_relationship_realization.png)

我们可以看到奔跑接口(Runnable)仅仅是描述一种能够奔跑的类型，而没有具体的实现；人类(Human)和狗(Dog)都分别提供对于奔跑接口的实现(实现 run 方法)，这时候我们就可以说人类与狗都是 **可奔跑的对象(实现 Runnable 类型的类)**

### 2.7 类与类之间的关系小结

看完六种对象之间的关系，我们最后做一个总整理和比较

| 关系类型            | 描述                               | 比较                                                                                     |
| ------------------- | ---------------------------------- | ---------------------------------------------------------------------------------------- |
| 关联 Association    | 一个类 **持有对另一个类的引用**    | 两者之间并无明显包含关系、仅仅就是持有引用                                               |
| 聚合 Aggregation    | 一个类 **包含** 另一个类           | 后者必须作为前者的部件或是组成元素                                                       |
| 组合 Composition    | 一个类 **包含** 另一个类           | 后者除了作为前者的部件或是组成元素之外，还必须是专属于前者的部件(离开前者则无含义的部件) |
| 依赖 Dependency     | 一个类 **使用(依赖)** 另一个类     | 前者在方法中直接、简介使用到了后者类型(参数、局部变量、静态方法调用)                     |
| 泛化 Generalization | 一个类 **继承/扩展/泛化** 另一个类 | 前者基于一个具体存在的类型(后者)进行扩展                                                 |
| 实现 Realization    | 一个类 **实现** 一个接口的方法     | 前者实现接口的方法后具备接口的特征，也就可以作为接口的类型(如 Runnable)被引用            |

# 结语

本篇介绍 UML 中关于 **类图(Class Diagram)** 的重要图例进行解说，供大家参考。

# 其他资源

## 参考连接

| Title                              | Link                                                                                                                   |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| 五分钟读懂UML类图                  | [https://www.cnblogs.com/shindo/p/5579191.html](https://www.cnblogs.com/shindo/p/5579191.html)                         |
| UML类图中的符号解释                | [https://blog.csdn.net/cabinriver/article/details/8894327](https://blog.csdn.net/cabinriver/article/details/8894327)   |
| uml 类图中+ - 含义                 | [https://blog.csdn.net/u014624241/article/details/74925442](https://blog.csdn.net/u014624241/article/details/74925442) |
| JAVA UML图，类图，接口图，抽象类图 | [https://my.oschina.net/u/4415254/blog/4556668](https://my.oschina.net/u/4415254/blog/4556668)                         |
| UML(一)：类、接口、抽象类          | [https://www.cnblogs.com/iamkk/p/5861025.html](https://www.cnblogs.com/iamkk/p/5861025.html)                           |

## 完整代码示例

[]()
