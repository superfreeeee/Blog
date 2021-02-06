# 设计模式: Factory Method 工厂方法模式

@[TOC](文章目录)

<!-- TOC -->

- [设计模式: Factory Method 工厂方法模式](#设计模式-factory-method-工厂方法模式)
  - [简介](#简介)
  - [参考](#参考)
  - [完整示例代码](#完整示例代码)
- [正文](#正文)
  - [场景](#场景)
  - [模式结构](#模式结构)
  - [代码实现](#代码实现)
    - [Products 产品类](#products-产品类)
    - [Creator 创建者](#creator-创建者)
    - [测试类](#测试类)
  - [补充：静态工厂方法](#补充静态工厂方法)
    - [静态工具类](#静态工具类)
    - [私有构造函数](#私有构造函数)
- [结语](#结语)

<!-- /TOC -->

## 简介


| 目的 | 创建型                                                                                | 结构型                                                                                                                         | 行为型                                                                                                                                                                                     |
| ---- | ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 类   | Factory Method 工厂方法                                                               | Adapter 适配器                                                                                                                 | Interpreter 解释器<br />Template Method 模版方法                                                                                                                                           |
| 对象 | Abstract Factory 抽象工厂<br />Builder 生成器<br />Prototype 原型<br />Singleton 单例 | Adapter 适配器<br />Bridge 桥接<br />Composite 组合<br />Decorator 装饰器<br />Facade 外观<br />Flyweight 享元<br />Proxy 代理 | Chain of Responsibility 职责链<br />Command 命令<br />Iterator 迭代器<br />Mediator 中介者<br />Memento 备忘录<br />Observer 观察者<br />State 状态<br />Strategy 策略<br />Visitor 访问者 |

- 回顾

接着上一篇的<a href="https://blog.csdn.net/weixin_44691608/article/details/113731955">设计模式: Abstract Factory 抽象工厂模式</a>，我们在创建对象的时候发现一种模式，为产品类型创造出公共接口，并建立抽象工厂的共同模式来提取**创建一组产品**的抽象行为，实际创建对象的行为则交由不同的实际工厂类来执行，同时实际工厂类还使得同系列产品的创建聚合到统一的工厂类型之中。

- 改善

然而在实际开发的时候仍然可能存在一个问题，开发时各个'产品'并不一定存在公共表现或是没有到'一组'产品那么多的类型。建立抽象产品组和抽象工厂的开销远比实际上需要管理的产品对象要复杂许多，实际上我们可能只是想单独为某个类型的产品开发多个实现而已，这时候我们就可以使用**Factory Method 工厂方法模式**。

## 参考

<table>
  <tr>
    <td>Design Patterns-Elements of Reusable Object-Oriented Software</td>
    <td><a href=""></a></td>
  </tr>
</table>

## 完整示例代码

<a href="https://github.com/superfreeeee/Blog-code/tree/main/design_pattern/src/main/java/com/example/factory_method/classic">https://github.com/superfreeeee/Blog-code/tree/main/design_pattern/src/main/java/com/example/factory_method/classic</a>

# 正文

## 场景

与前面的抽象工厂相比，工厂方法更适合于**产品类型较少**，而对**实际产品类型的扩展性**要求更高的场景

在这样的场景之下，我们希望免去建立抽象工厂类型的开销，同时避免为了维护共同抽象产品组接口以及各个工厂的一致性(实现抽象方法)。这时候我们可以从抽象工厂类型中提取出用于**创建单一个产品类型的工厂方法**，并将其放入到实际使用或是对实际产品类型进行再加工的类当中。

如此一来相当于是将负责**生产多个产品的'工厂'**的职责缩小成负责**单一产品的'创建者'**，同时透过继承并实现创建者中声明用于创建产品的方法来实现产品的多态。；厂方法也是由抽象工厂中抽取出单一个用于生产产品的方法而命名的。

## 模式结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/factory_method_pattern_structure.png)

- Product 抽象产品类：需要创造的产品的公共接口
- Concrete Product 实际产品类：真正创建的产品类型，继承抽象产品类并实现产品的多态
- Creator 创建者(抽象类)：声明创建产品的抽象工厂方法，并定义其他对产品加工或需要使用产品的方法(这边的重点专注于对象的创建)
- Concrete Creator 实际创建者：继承并实现真正创建产品的工厂方法(其他加工方法已经在抽象类中定义，在不同产品实现下依旧相同的公共方法)

由图中我们可以看到：我们将原来的抽象工厂中的一个个工厂方法切分出来，并将该职责分派给实际创建并加工产品的创建者，相当于将创建的职责缩小为创建单一类型的产品，增加了对产品变化的灵活性

## 代码实现

下面我们直接根据上面的类型结构做一个简单的实现示意。在真正的应用当中应该要去思考系统中的'产品'类型的抽象以及建立抽象工厂的必要性，最后才决定使用工厂方法或是抽象工厂

### Products 产品类

首先先定义好抽象产品类和实际产品实现

```java
/* Product.java */
public interface Product {}

/* ProductA.java */
public class ProductA implements Product {}

/* ProductB.java */
public class ProductB implements Product {}
```

### Creator 创建者

接下来是负责创建产品的创建者类

```java
/* Creator.java */
public abstract class Creator {

    public abstract Product createProduct();

    public void operation() {
        Product product = createProduct();
        System.out.println("using Product " + product);
    }
}
```

我们先在抽象类中定义好用于加工产品的方法，并留下负责创建产品的**工厂方法(抽象方法)**；下面我们定义具体的创建者来实现不同实际产品的创建

```java
/* CreatorA.java */
public class CreatorA extends Creator {
    @Override
    public Product createProduct() {
        return new ProductA();
    }
}

/* CreatorB.java */
public class CreatorB extends Creator {
    @Override
    public Product createProduct() {
        return new ProductB();
    }
}
```

### 测试类

最后给出测试用代码和结果

```java
/* CreatorTest.java */
public class CreatorTest {

    private void testTemplate(Creator creator) {
        creator.operation();
    }

    @Test
    public void test_creatorA() {
        testTemplate(new CreatorA());
    }

    @Test
    public void test_creatorB() {
        testTemplate(new CreatorB());
    }
}
```

```
using Product com.example.factory_method.classic.ProductA@50134894
using Product com.example.factory_method.classic.ProductB@deb6432
```

## 补充：静态工厂方法

接下来我们补充一个在实践中也经常使用的工厂方法模式的变种：**静态工厂方法(Static Factory Method)**

在前面的工厂方法中我们透过实现抽象方法在**管理产品的创建**的同时完成**产品类型的多态**，然而工厂方法更多的是要表达对于对象创建的管理。然而我们可以在更进一步，透过静态方法的形式(或是语言的特性)来统一管理特定类型所有对象的创建。

下面我们以 Java 实现举例

### 静态工具类

第一种我们接续上面用到的产品类型，单独创建一个类，并为其添加静态的、用于创建不同对象的类。由于这个类本身并不需要是一个对象，我们可以声明为**静态**的方法，使得这些工厂方法是绑定在该工具类的命名空间之下，就好像一个放在全局的函数一样。

```java
package com.example.factory_method.classic;
/* StaticCreator.java */
public class StaticCreator {

    public static ProductA createProductA() {
        return new ProductA();
    }

    public static ProductB createProductB() {
        return new ProductB();
    }

    // ...
}
```

你说这样就没法多态了？确实，要为每一种产品实现维护一个工厂方法相对来说还是略显麻烦(不过这也是系统初期刚开始建立产品类型时抽象程度不高的基础办法，读者也要切记**不可为了抽象而抽象**，而是在产品类型到达一定规模之后存在必要性或其他优化、管理需求，才需要进行更进一步的抽象)，我们还可以借助 Java 的反射机制一劳永逸。

```java
/* StaticCreator.java */
public class StaticCreator {

    /* createProductA ... */
    /* createProductB ... */

    public static Product createProduct(Class<? extends Product> clazz) throws IllegalAccessException, InstantiationException {
        return clazz.newInstance();
    }

    public static Product createProduct(String type) throws ClassNotFoundException, IllegalAccessException, InstantiationException {
        return ((Class<? extends Product>) Class.forName(type)).newInstance();
    }
}
```

- 测试代码

```java
/* StaticCreatorTest.java */
public class StaticCreatorTest {

    @Test
    public void test() throws InstantiationException, IllegalAccessException, ClassNotFoundException {
        Product product = StaticCreator.createProductA();
        System.out.println(product);
        product = StaticCreator.createProductB();
        System.out.println(product);
        product = StaticCreator.createProduct(ProductA.class);
        System.out.println(product);
        product = StaticCreator.createProduct("com.example.factory_method.classic.ProductB");
        System.out.println(product);
    }
}
```

- 测试结果

```
com.example.factory_method.classic.ProductA@28ba21f3
com.example.factory_method.classic.ProductB@694f9431
com.example.factory_method.classic.ProductA@f2a0b8e
com.example.factory_method.classic.ProductB@593634ad
```

### 私有构造函数

第二种我们可以透过私有化(`private` 或 `protected`)构造函数来限制其他类来实例化(`new`)产品，同时再对外暴露一个静态的工厂方法用于创建对象如下

```java
/* ProductC */
public class ProductC {

    private ProductC() {}

    public static ProductC create() {
        return new ProductC();
    }
}
```

如此一来所有 ProductC 对象的创建都必须经过静态的 create 方法来创建，如果我们想要对创建进行**记录**、**加上其他限制**或**产生其他副作用**的话都能够加在工厂方法内部

```java
private static instances = new ProductC[100];
private static int count = 0; // 创建实例的计数
public static ProductC create() {
    ProductC productC = count >= 100 ? instances[count % 100] : (instances[count] = new ProductC()); // 限制实例个数
    record("create " + count); // 创建方法调用记录(副作用)
    count++;
    return new ProductC();
}
```

同时使用工厂方法也能够使得使用者更具体的了解并给出创建产品时需要的关键信息，如下工厂方法返回一个 Spring MVC 的 Http 响应对象

```java
/* ResponseUtils.java */

public class ResponseUtils {
    public static ResponseEntity<Object> create(Object data, HttpStatus status) {
        HttpHeaders headers = new HttpHeaders();
        headers.setContentTypt(MediaType.APPLICATION_JSON);
        return new ReponseEntity<>(data, headers, status);
    }
}
```

# 结语

本篇介绍了抽象工厂的派生模式 - Factory Method 工厂方法模式，相当于是对抽象工厂的切分，在避免了维护抽象工厂的额外开销之下同时保证的产品类型的稳定性和创建的管理，是应对**产品种类不多时**的更好的实现。
