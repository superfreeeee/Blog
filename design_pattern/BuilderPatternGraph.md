# 设计模式: Builder Pattern 生成器模式(创建者模式)

@[TOC](文章目录)

<!-- TOC -->

- [设计模式: Builder Pattern 生成器模式(创建者模式)](#设计模式-builder-pattern-生成器模式创建者模式)
  - [简介](#简介)
  - [参考](#参考)
  - [完整示例代码](#完整示例代码)
- [正文](#正文)
  - [场景](#场景)
  - [模式结构](#模式结构)
  - [示例：HTML 2D图形文本绘制](#示例html-2d图形文本绘制)
    - [目标](#目标)
    - [类型结构](#类型结构)
    - [具体实现](#具体实现)
      - [GraphBuilder(Builder)](#graphbuilderbuilder)
      - [GraphDrawer(Director)](#graphdrawerdirector)
      - [SVGBuilder(Concrete Builder)](#svgbuilderconcrete-builder)
    - [测试代码](#测试代码)
- [结语](#结语)

<!-- /TOC -->

## 简介

| 目的 | 创建型                                                                                | 结构型                                                                                                                         | 行为型                                                                                                                                                                                     |
| ---- | ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 类   | Factory Method 工厂方法                                                               | Adapter 适配器                                                                                                                 | Interpreter 解释器<br />Template Method 模版方法                                                                                                                                           |
| 对象 | Abstract Factory 抽象工厂<br />Builder 生成器<br />Prototype 原型<br />Singleton 单例 | Adapter 适配器<br />Bridge 桥接<br />Composite 组合<br />Decorator 装饰器<br />Facade 外观<br />Flyweight 享元<br />Proxy 代理 | Chain of Responsibility 职责链<br />Command 命令<br />Iterator 迭代器<br />Mediator 中介者<br />Memento 备忘录<br />Observer 观察者<br />State 状态<br />Strategy 策略<br />Visitor 访问者 |

上表为 GoF 四人帮的 23 种设计模式，本篇将要来介绍的是对象创建型的 **Builder 生成器模式**。

## 参考

<table>
  <tr>
    <td>Design Patterns-Elements of Reusable Object-Oriented Software</td>
    <td><a href=""></a></td>
  </tr>
</table>

## 完整示例代码

<a href="https://github.com/superfreeeee/Blog-code/tree/main/design_pattern/src/main/java/com/example/builder/graph">https://github.com/superfreeeee/Blog-code/tree/main/design_pattern/src/main/java/com/example/builder/graph</a>

# 正文

## 场景

设计模式的宗旨在于对复杂场景共有特性进行抽象，实现逻辑解耦和事务的高层抽象。

本篇要介绍的 Builder 模式是在这样的一个场景之下：程序员希望利用一个**与具体类型无关的方法**创建对象。在一些开发场景中，我们可能需要创建某一个类型，而这个类型可能会存在**不同的实现**，甚至**不同的组合方式和顺序**。

这时候就能够透过使用 Builder 模式，在一定的接口规范之下开发**与实际类型无关的对象组合方式**，同时也能够**屏蔽不同实际类型所需要的部件类型**，增强内部结构的内聚力并放松与使用者的耦合度。

## 模式结构

该模式中存在三个重要角色：

![](https://picures.oss-cn-beijing.aliyuncs.com/img/builder_pattern_structure.png)

- Director 导向器：描述与类型无关的对象创建方法和过程
- Builder 生成器接口：定义不同生成器的共同接口(Director 是面向 Builder 编写的)
- Concrete Builder 具体生成器：定义生成器的具体操作

在这样的结构之下，实际真正生成对象的职责位于 Concrete Builder 身上，而 Director 只透过使用 Builder 的公共接口来描述对象的生成。

如此一来不仅能够将具体的对象生成方式与使用者结藕，同时使用者无需知道任何具体的产品类型(Product)或至多需要知道最终结果返回的公共类型即可。

## 示例：HTML 2D图形文本绘制

接下来我们给出一个具体的示例来帮助大家更深刻的了解 Builder 模式。

### 目标

下面我们的目标是生成可以生成图形的 html 文本，具体的实现类型有 svg 和直接使用 `<div>` 元素组合出图形

### 类型结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/builder_pattern_graph.png)

在该示例中：

1. `GraphBuilder` 描述了两种方式共同的绘图方法接口
2. `GraphDrawer` 使用 GraphBuilder 的接口绘制出与类型无关的图形文本
3. `SVGBuilder`、`ElementBuilder` 各自继承并实现 GraphBuilder 的公共方法，各个生成器内部使用的类型对外是不可见的同时也是与其他实际的 Builder 类型无关的

### 具体实现

接下来我们给出几个类的部分实现代码作代表，完整实现请参考[完整示例代码](#完整示例代码)

#### GraphBuilder(Builder)

首先最重要的就是 Builder 接口，也是整个模式的灵魂，定义了不同实际生成器之间的公共接口

```java
package com.example.builder.graph;

/**
 * HTML 2D图像生成器
 */
public interface GraphBuilder {

    /* 初始化 */
    GraphBuilder init();

    /* 添加新的分组 */
    GraphBuilder group(String id);

    default GraphBuilder group() {
        group(null);
        return this;
    }

    /* 结束当前分组 */
    GraphBuilder groupEnd();

    /* 添加矩形 */
    GraphBuilder addRect(int x, int y, int rx, int ry, int width, int height, String[] style);

    /* 添加圆形 */
    GraphBuilder addCircle(int cx, int cy, int r, String[] style);

    /* 添加直线 */
    GraphBuilder addLine(int x1, int y1, int x2, int y2, String[] style);

    /* 生成最终结果 */
    String build();

    /* 可视化结果 */
    void check();
}
```

#### GraphDrawer(Director)

我们在 GraphBuilder 中定义好公共接口之后，其实就能够根据接口直接绘制高层抽象的图形了，下面我们定义两个方法：一个简单的用到全部的接口，第二个画一张脸出来

```java
package com.example.builder.graph;

public class GraphDrawer {

    public String test(GraphBuilder builder) {
        return builder.init()
                .addRect(0, 0, 0, 0, 100, 100, new String[]{"fill: black"})
                .group()
                .addRect(100, 0, 0, 0, 100, 100, new String[]{"fill: black"})
                .groupEnd()
                .addRect(200, 0, 0, 0, 100, 100, new String[]{"fill: black"})
                .addCircle(50, 50, 20, new String[]{"fill: black"})
                .addLine(0, 0, 100, 100, new String[]{"stroke: black", "strokeWidth: 3"})
                .build();
    }

    public String createFace(GraphBuilder builder) {
        return builder.init()
                .addCircle(50, 50, 50, new String[]{"stroke: black"})
                .addCircle(30, 50, 5, new String[]{"stroke: black"})
                .addCircle(70, 50, 5, new String[]{"stroke: black"})
                .addCircle(50, 50, 5, new String[]{"stroke: black"})
                .addLine(30, 70, 70, 70, new String[]{"stroke: black"})
                .build();
    }
}
```

我们可以看出两个方法都是使用参数的 GraphBuilder 对象，也就是根据共同接口的高层抽象来描述图形，所以接下来我们就需要创建实际的 GraphBuilder 类型来定义图形绘制的基本操作。

#### SVGBuilder(Concrete Builder)

由于篇幅和实现问题，我们仅仅给出 SVGBuilder 部分的实现

```java
package com.example.builder.graph.svg;

import com.example.builder.graph.GraphBuilder;

import java.util.ArrayList;
import java.util.List;

public class SVGBuilder implements GraphBuilder {
    private List<Item> items;
    private Group group;

    private void addItem(Item item) {
        List<Item> items = group == null ? this.items : group.items;
        items.add(item);
    }

    @Override
    public GraphBuilder init() {
        items = new ArrayList<>();
        return this;
    }

    @Override
    public GraphBuilder group(String id) {
        Group g = new Group(group, id);
        addItem(g);
        group = g;
        return this;
    }

    @Override
    public GraphBuilder groupEnd() {
        if (group != null) {
            group = group.outer;
        }
        return this;
    }

    @Override
    public GraphBuilder addCircle(int cx, int cy, int r, String[] style) {
        addItem(Circle.from(cx, cy, r, style));
        return this;
    }

    @Override
    public GraphBuilder addLine(int x1, int x2, int y1, int y2, String[] style) {
        addItem(Line.from(x1, x2, y1, y2, style));
        return this;
    }

    @Override
    public GraphBuilder addRect(int x, int y, int rx, int ry, int width, int height, String[] style) {
        addItem(Rect.from(x, y, rx, ry, width, height, style));
        return this;
    }

    @Override
    public String build() {
        StringBuilder res = new StringBuilder();
        res.append("<svg>");
        for (Item item : items) res.append(item.build());
        res.append("</svg>");
        return res.toString();
    }

    @Override
    public void check() {
        System.out.println("<svg>");
        showItems(items, "  ");
        System.out.println("</svg>");
    }

    private void showItems(List<Item> items, String prefix) {
        for (Item item : items) {
            if (item instanceof Group) {
                System.out.println(prefix + "<g>");
                showItems(((Group) item).items, prefix + "  ");
                System.out.println(prefix + "</g>");
            } else {
                System.out.println(prefix + item.build());
            }
        }
    }
}
```

我们可以看到在 SVGBuilder 内部，所有元素都继承 Item，而 Item 和所有具体的图形对象是对 GraphDrawer 屏蔽的，如此一来就实现了具体实现类型和高层创建方法结藕。

### 测试代码

最后给出测试代码

```java
package com.example.builder.graph;

import com.example.builder.graph.element.ElementBuilder;
import com.example.builder.graph.svg.SVGBuilder;
import org.junit.Test;

public class GraphDrawerTest {

    private GraphDrawer director = new GraphDrawer();

    private void testTemplate(GraphBuilder builder) {
        String res = director.test(builder);
        System.out.println("res: " + res);
        builder.check();

        res = director.createFace(builder);
        System.out.println("face: " + res);
        builder.check();
    }

    @Test
    public void createSVG() {
        testTemplate(new SVGBuilder());
    }

    @Test
    public void createElement() {
        testTemplate(new ElementBuilder());
    }
}
```

# 结语

本篇介绍的 Builder 模式，相当于是将所有内部类型和实现细节包装到各个 Builder 内部，而 Director 则是面对 Builder 接口的高层抽象方法来描述图形。实际上调用方法的时候则根据传入的 Builder 对象的不同绘制出不同的结果，供大家参考。
