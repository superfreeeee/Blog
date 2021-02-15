# 设计模式: Mediator 中介者模式

@[TOC](文章目录)

<!-- TOC -->

- [设计模式: Mediator 中介者模式](#设计模式-mediator-中介者模式)
  - [简介](#简介)
  - [参考](#参考)
  - [完整示例代码](#完整示例代码)
- [正文](#正文)
  - [场景](#场景)
  - [模式结构](#模式结构)
  - [代码实现：文字对话框](#代码实现文字对话框)
    - [Widgets 窗口组件](#widgets-窗口组件)
    - [DialogDirector 对话框导向器](#dialogdirector-对话框导向器)
    - [测试](#测试)
- [结语](#结语)
  - [模式特点](#模式特点)

<!-- /TOC -->

## 简介

| 目的 | 创建型                                                                                | 结构型                                                                                                                         | 行为型                                                                                                                                                                                     |
| ---- | ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 类   | Factory Method 工厂方法                                                               | Adapter 适配器                                                                                                                 | Interpreter 解释器<br />Template Method 模版方法                                                                                                                                           |
| 对象 | Abstract Factory 抽象工厂<br />Builder 生成器<br />Prototype 原型<br />Singleton 单例 | Adapter 适配器<br />Bridge 桥接<br />Composite 组合<br />Decorator 装饰器<br />Facade 外观<br />Flyweight 享元<br />Proxy 代理 | Chain of Responsibility 职责链<br />Command 命令<br />Iterator 迭代器<br />Mediator 中介者<br />Memento 备忘录<br />Observer 观察者<br />State 状态<br />Strategy 策略<br />Visitor 访问者 |

本篇要介绍的是对象的行为型设计模式 **Mediator 中介者模式**，用于结藕多个对象之间的复杂交互关系，并将交互逻辑集中到中介者对象之中统一管理。

## 参考

<table>
  <tr>
    <td>Design Patterns-Elements of Reusable Object-Oriented Software</td>
    <td><a href=""></a></td>
  </tr>
  <tr>
    <td>通俗易懂设计模式解析——中介者模式</td>
    <td><a href="https://www.cnblogs.com/hulizhong/p/11655641.html">https://www.cnblogs.com/hulizhong/p/11655641.html</a></td>
  </tr>
</table>

## 完整示例代码

<a href="https://github.com/superfreeeee/Blog-code/tree/main/design_pattern/src/main/java/com/example/mediator/classic">https://github.com/superfreeeee/Blog-code/tree/main/design_pattern/src/main/java/com/example/mediator/classic</a>

# 正文

## 场景

在 OOP 的编程风格之下，我们通常会将互相关联的数据和操作封装到一个对象之中，然后再通过多个对象的合作来完成服务的实现。然而在某些场景之下可能出现多个复杂对象互相合作使用的场景，如果一个对象需要使用另一个对象的服务，我们可以看成两个对象之间存在一条边，则多个对象之间最复杂的交互关系就相当于一个**完全图**。

然而在多个对象交互关系过于复杂的情况下，就几乎失去了封装带来的便利，同时由于一个对象一次使用了多个其他对象，也是的**对象之间的耦合度太高**，要再维护甚至修改都是异常的困难。

在这样的场景之下我们就可以使用 **Mediator 中介者模式** 来解决。

## 模式结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/mediator_pattern_structure.png)

- Mediator 中介者接口：声明中介者应具备的操作接口
- Colleague 合作对象抽象类：定义合作对象与中介者的交互逻辑，并保存对中介者的引用
- ConcreteMediator 具体中介者：实现中介者的操作接口，并定义各个合作对象的交互逻辑
- ConcreteColleague 具体合作对象：定义对象内部逻辑，并在特定时机调用中介者来与其他对象交互

透过将合作对象定义为一个相对较**纯粹**的对象，并将对象之间的复杂调用逻辑转变为调用中介者的形式(相当于调用其他对象的行为转变为统一委托同一个中介者)。也就是使对象之间互相调用的逻辑从下面的左图转变成右图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/mediator_pattern_cooperation1.png) ![](https://picures.oss-cn-beijing.aliyuncs.com/img/mediator_pattern_cooperation2.png)

从对象交互的角度来看：合作对象都声明为一个纯粹的独立操作的个体，每个对象至多与中介者对象进行交互。

从对象职责的角度来看：我们将与原本分散在各个对象的复杂交互逻辑简化为单个对象能独立存在并作用的最基本的逻辑，并由中介者来负担起对象间的交互应用逻辑。

## 代码实现：文字对话框

接下来我们一样给出代码实现的核心部分，完整代码内容请参见[完整示例代码](#完整示例代码)

本次代码示例模拟的是一个**文字对话框**的展示，具体的类图如下所示

![](https://picures.oss-cn-beijing.aliyuncs.com/img/mediator_pattern_sample.png)

文字对话框窗口(`FontDialogDirector`)是对话框(`DialogDirector`)的一种实现，它使用了四个窗口组件(`Widget`)，分别是一个字体选择列表(`ListBox`)、一个输入框(`EntryField`)和两个按钮(`Button`)。

在实际业务场景中如果作为中介者的类只有唯一实现的时候并不需要特别独立出一个 Mediator 的接口，可以直接使用唯一实现的 ConcreteMediator 类型。本篇的代码实现虽然只有一个文字对话框的实现，但是依旧给出对话框的抽象操作接口，一来是为了符合上面的模式结构类图风格，二来是提供对话框组件的可扩展性，如果后续需要添加像是贴图对话框、图片/视频对话框等新的复杂对话框组件，就可以直接增加对 DialogDirector 的实现而不需要修改原来的文字对话框类(`FontDialogDirector`)。

### Widgets 窗口组件

首先我们定义一个抽象的窗口类，它封装了**与中介者的交互行为(changed 接口)**

```java
/* Widget.java */
public abstract class Widget {

    private DialogDirector director;

    public Widget(DialogDirector director) {
        this.director = director;
    }

    public void changed() {
        director.widgetChanged(this);
    }
}
```

作为与中介者交互的逻辑，我们通常也不需要为组件定义太复杂的通知调用逻辑，因为从调用的职责来看，组件内部并不知道组件作为复杂系统中的什么角色，因此与中介者交互的行为就能够简化为**通知中介者组件发生改变**的操作，具体哪个组件应该作出何种反应进行什么调用则是由掌控全局的中介者来决定。

下面我们定义三种组件的实现

```java
/* ListBox.java */
public class ListBox extends Widget {

    private List<String> options;
    private int select = -1;

    public ListBox(DialogDirector director) {
        super(director);
    }

    public String getSelection() {
        return select >= 0 ? options.get(select) : "";
    }

    public void setList(List<String> options) {
        this.options = options;
    }

    public void handleMouse(int select) {
        System.out.println("[Widget Event] select ListBox");
        this.select = select;
        changed();
    }
}
```

可选列表负责维护一个可选的选项列表(`options`)，以及选中的索引(`select`)，而点击可选列表的行为则简化为修改选中索引后就将控制权转交给中介者进行处理。

```java
/* EntryField.java */
public class EntryField extends Widget {

    private String text;

    public EntryField(DialogDirector director) {
        super(director);
    }

    public String getText() {
        return text;
    }

    public void setText(String text) {
        this.text = text;
    }

    public void handleInput(String input) {
        System.out.println("[Widget Event] input EntryField");
        setText(input);
        changed();
    }

}
```

在输入栏位组件则是维护一个输入字符串，主要的处理事件为用户输入，修改输入框(`text`)内容后通知中介者进行处理

```java
/* Button.java */
public class Button extends Widget {

    private String text = "button";

    public Button(DialogDirector director) {
        super(director);
    }

    public void setText(String text) {
        this.text = text;
    }

    public void handleMouse() {
        System.out.println("[Widget Event] click Button");
        changed();
    }
}
```

按钮组件就更简单了，仅仅只需要维护按钮的文字，以及简单的点击事件即可

### DialogDirector 对话框导向器

有了现成的'纯'组件类之后，我们就可以透过定义一个中介者来持有并操作多个组件来实现一个复杂组件。首先我们先定义对话框组件的抽象接口

```java
/* DialogDirector.java */
public interface DialogDirector {

    void init();

    void showDialog();

    void widgetChanged(Widget widget);

}
```

这边我们必须谨记对话框组件的抽象接口应该要是对'对话框'本身行为单纯的描述，而不能针对文字对话框进行抽象，因为文字对话框本身只是针对对话框的一种实现而已。下面我们给出文字对话框的具体实现

```java
/* FonDialogDirector.java */
public class FonDialogDirector implements DialogDirector {

    // 这边本应设置为 private，由于测试需要主动触发窗口组件事件所以设为 public
    public ListBox fontList;
    public EntryField field;
    public Button ok, cancel;

    @Override
    public void init() {
        fontList = new ListBox(this);
        field = new EntryField(this);
        ok = new Button(this);
        cancel = new Button(this);

        List<String> options = new ArrayList<>();
        options.add("Arial");
        options.add("Helvetica");
        options.add("sans-serif");
        fontList.setList(options);

        field.setText("");
    }

    @Override
    public void showDialog() {
        System.out.println(String.format("show text in font type: %s", field.getText()));
    }

    @Override
    public void widgetChanged(Widget widget) {
        if (widget == fontList) {
            String font = fontList.getSelection();
            System.out.println(String.format("[widgetChanged] select font: %s", font));
        } else if (widget == field) {
            System.out.println("[widgetChanged] handle fontName changed: " + field.getText());
        } else if (widget == ok) {
            System.out.println("[widgetChanged] press ok");
            showDialog();
        } else if (widget == cancel) {
            System.out.println("[widgetChanged] press cancel");
            field.setText("");
        }
        System.out.println();
    }

}
```

在初始化方法(`init`)中我们构造一个文字对话框需要的组件：可选语言列表、输入框、确认/取消按钮，接下来实现一个展示对话框(`showDialog`)的方法。最后一个 `widgetChanged` 方法是我们的重头戏

`widgetChanged` 方法本质上是一个对内的方法，它的作用在于提供组件更新后通知中介者的接口。这边的实现方法是组件更新后，透过引用判断是哪个组件之后进行对应的处理

### 测试

最后给出测试用例，由于本篇并没有实现可视化的部分，因此点击/输入事件就透过将成员组件设为 `public` 后主动触发事件的方式来进行测试

```java
/* FonDialogDirectorTest.java */
public class FonDialogDirectorTest {

    @Test
    public void test() {
        FonDialogDirector director = new FonDialogDirector();
        director.init();
        director.fontList.handleMouse(1); // 选择字体
        director.field.handleInput("input a message"); // 输入信息
        director.cancel.handleMouse(); // 点击取消
        director.field.handleInput("re-input a message"); // 再次输入信息
        director.ok.handleMouse(); // 点击确定
    }
}
```

- 输出结果

```
[Widget Event] select ListBox
[widgetChanged] select font: Helvetica

[Widget Event] input EntryField
[widgetChanged] handle fontName changed: input a message

[Widget Event] click Button
[widgetChanged] press cancel

[Widget Event] input EntryField
[widgetChanged] handle fontName changed: re-input a message

[Widget Event] click Button
[widgetChanged] press ok
show text in font type: re-input a message
```

我们可以看到每个事件都会有一个组件内的操作更新，以及中介者对组件更新后的操作输出，这便是将纯组件功能以及复杂的组件间交互逻辑分离的体现。

# 结语

## 模式特点

- 合作对象间的结藕：各个对象只需要与中介者进行交互，同时也使得合作对象的职责更为独立并集中
- 减少子类的生成：使得合作对象的职责更为纯粹而容易重用，复杂的交互逻辑只需要再实现新的 Director 即可实现
- 简化对象协议：由原来多个对象间的复杂调用逻辑和混乱的接口，变为与中介者的一对一交互
- 控制集中化：对象间的交互逻辑集中到中介者，使得相关的调用代码集中到临近的区域；然而控制集中可能带来中介者过于庞大、复杂，需要加入其他模式来调解。

实现中介者模式带来的好处简而言之就是**提高组件的可复用性**，透过将复杂的对象交互逻辑提取出来，留下较为纯粹单一的职责，使得单个组件更'纯粹'，也就是更容易复用，供大家参考。
