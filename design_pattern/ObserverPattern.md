# 设计模式: Observer 观察者模式

@[TOC](文章目录)

<!-- TOC -->

- [设计模式: Observer 观察者模式](#设计模式-observer-观察者模式)
  - [简介](#简介)
    - [从 MVC 到 MVVM](#从-mvc-到-mvvm)
  - [参考](#参考)
  - [完整示例代码](#完整示例代码)
- [正文](#正文)
  - [场景](#场景)
  - [模式结构](#模式结构)
  - [代码示例](#代码示例)
    - [Subject 可订阅对象/主题](#subject-可订阅对象主题)
    - [Observer 观察者](#observer-观察者)
    - [测试代码](#测试代码)
- [结语](#结语)

<!-- /TOC -->

## 简介

| 目的 | 创建型                                                                                | 结构型                                                                                                                         | 行为型                                                                                                                                                                                     |
| ---- | ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 类   | Factory Method 工厂方法                                                               | Adapter 适配器                                                                                                                 | Interpreter 解释器<br />Template Method 模版方法                                                                                                                                           |
| 对象 | Abstract Factory 抽象工厂<br />Builder 生成器<br />Prototype 原型<br />Singleton 单例 | Adapter 适配器<br />Bridge 桥接<br />Composite 组合<br />Decorator 装饰器<br />Facade 外观<br />Flyweight 享元<br />Proxy 代理 | Chain of Responsibility 职责链<br />Command 命令<br />Iterator 迭代器<br />Mediator 中介者<br />Memento 备忘录<br />Observer 观察者<br />State 状态<br />Strategy 策略<br />Visitor 访问者 |

**Observer 观察者模式**，又称为**发布-订阅(subscribe-publish)模式**。在前端最有名的应用就是 MVVM 的响应式数据的设计，就应用到了观察者模式。

### 从 MVC 到 MVVM

![](https://picures.oss-cn-beijing.aliyuncs.com/img/observer_application_mvc_mvvm.png)

一开始出现的是 **MVC(Model-View-Controller)** 架构，将系统的 **展示部分(View)** 和 **数据模型(Model)** 分离，并透过统一的 **控制器(Controller)** 负责数据的更新和视图的选择。

然而这样存在一些缺点：

- 视图(View)和控制器(Controller)的联系过于紧密：控制器需要对视图的部件行为进行管理，造成展示逻辑与控制逻辑的耦合
- 视图(View)访问数据模型(Model)的低效：由于用户在视图界面发起请求之后会交由控制器(Controller)处理，最后在到数据模型(Model)中改变状态，最后视图才能从数据模型'看'到状态的改变而改变展示数据

**MVVM 架构**则是对 MVC 的一种改进，将展示控制逻辑与展示页面放在一起，同时实现 View 与 View Model 的双向数据绑定，省去了对页面展示数据的控制(将视图的更新透过双向绑定自动更新)，使得 ViewModel 能更**专注在业务的逻辑以及与数据层的交互上**。

而实现 MVVM 最关键的 **双向绑定** 效果就是透过观察者模式来实现的。网上有许多关于 Vue 对 MVVM 架构的实现原理、解析啥的，有兴趣的可以去查查，本篇希望专注在更本质的观察者模式的抽象意义上。

## 参考

<table>
  <tr>
    <td>Design Patterns-Elements of Reusable Object-Oriented Software</td>
    <td><a href=""></a></td>
  </tr>
  <tr>
    <td>谈谈MVC、MVP和MVVM的优缺点</td>
    <td><a href="https://blog.csdn.net/github_34402358/article/details/88735473">https://blog.csdn.net/github_34402358/article/details/88735473</a></td>
  </tr>
</table>

## 完整示例代码

<a href="https://github.com/superfreeeee/Blog-code/tree/main/design_pattern/src/main/java/com/example/observer/classic">https://github.com/superfreeeee/Blog-code/tree/main/design_pattern/src/main/java/com/example/observer/classic</a>

# 正文

## 场景

在生产中我们常常会有这样一个场景：对象A的数据或是行为依赖于对象B的状态，同时我们希望每次对象B的状态发生改变的时候，对象A能自动的**观察**到对象B的变化，并针对变化作出更新。

在这样的场景下我们就可以说对象A是一个**观察者(Observer)**，而对象B则是一个**被观察的对象(Subject)**(从发布-订阅的角度来说，对象B是被订阅的主题；而对象A则是订阅者)。观察者模式就是在此基础之上，建立一个自动通知更新的对象交互模型。

## 模式结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/observer_pattern_structure.png)

- Subject 可订阅对象抽象类：定义观察-通知相关的操作接口，保存观察者列表
- ConcreteSubject 可订阅对象类：实际操作对象，在必要时刻(状态改变)时调用 notify 通知观察者更新
- Observer 观察者抽象类：声明更新接口，保存订阅对象
- ConcreteObserver 观察者类：定义实际更新操作

从上图的结构我们可以看到我们将大部分的订阅-发布相关的操作在抽象类就定义好了，只留下 Subject 的 getState 由具体类给定如何获取状态；以及 Observer 的 update 定义具体更新操作的接口。而只需要 ConcreteSubject 在其他操作中主动调用 notify 通知观察者进行更新即可实现观察者的被动通知并更新的特性。

## 代码示例

接下来我们给出根据上述类图给出实际的 Java 代码示例

### Subject 可订阅对象/主题

首先是与订阅-发布相关的操作定义，也就是类图中的抽象类 Subject。这边我们直接以一个 State 枚举类来代表可订阅状态的对象，实际应用当中可以抽象出其他状态对象。

备注：由于 notify 方法与 Java 在 Object 对象上的接口重名，所以本篇使用 `subscribe-publish` 的命名来实现

```java
/* Subject.java */
public abstract class Subject {

    // 观察者列表
    private List<Observer> observers;

    protected Subject() {
        this.observers = new ArrayList<>();
    }

    /*订阅 */
    public void subscribe(Observer observer) {
        if (!observers.contains(observer)) {
            observers.add(observer);
        }
    }

    /* 取消订阅 */
    public void unsubscribe(Observer observer) {
        if (observers.contains(observer)) {
            observers.remove(observer);
        }
    }

    /* 发布事件 */
    public void publish() {
        for (Observer observer : observers) {
            observer.update();
        }
    }

    /* 获取订阅对象状态 */
    abstract State getState();
}
```

接下来是实际使用的操作对象。由于我们使用一个 State 枚举类来代表订阅对象的状态，因此 nextState 方法就是模拟订阅对象的状态更新后调用发布方法通知观察者

```java
/* ConcreteSubject.java */
public class ConcreteSubject extends Subject {

    private State state;

    public ConcreteSubject() {
        this.state = State.Sleep;
    }

    public void nextState() {
        switch (state) {
            case Sleep:
                state = State.Ready;
                break;
            case Ready:
                state = State.Run;
                break;
            case Run:
                state = State.Sleep;
                break;
        }
        publish();
    }

    @Override
    public State getState() {
        return state;
    }
}
```

### Observer 观察者

第二个是我们的观察者对象，首先给出抽象类 Observer 负责管理订阅对象

```java
/* Observer.java */
public abstract class Observer {

    private Subject subject;

    protected Observer(Subject subject) {
        this.subject = subject;
    }

    /**
     * 更新
     */
    public abstract void update();

    public State getState() {
        return subject.getState();
    }
}
```

最后给出具体的观察者类 ConcreteObserver，定义具体的更新操作

```java
/* ConcreteObserver.java */
public class ConcreteObserver extends Observer {

    private ConcreteObserver(Subject subject) {
        super(subject);
    }

    public static ConcreteObserver init(Subject subject) {
        ConcreteObserver observer = new ConcreteObserver(subject);
        subject.subscribe(observer);
        return observer;
    }

    @Override
    public void update() {
        State state = getState();
        System.out.println(String.format("[Observer@%x update] getState: %s", this.hashCode(), state));
    }
}
```

### 测试代码

最后给出测试代码

```java
/* ObserverTest.java */
public class ObserverTest {

    @Test
    public void test() {
        System.out.println(1);

        ConcreteSubject subject = new ConcreteSubject();
        Observer observer1 = ConcreteObserver.init(subject);
        Observer observer2 = ConcreteObserver.init(subject);
        subject.nextState();

        System.out.println(2);

        Observer observer3 = ConcreteObserver.init(subject);
        subject.nextState();

        System.out.println(3);

        subject.unsubscribe(observer2);
        subject.unsubscribe(observer1);
        subject.nextState();

        System.out.println(4);

        subject.unsubscribe(observer3);
        subject.nextState();

        System.out.println(5);
    }
}
```

- 输出结果

```
1
[Observer@28ba21f3 update] getState: Ready
[Observer@694f9431 update] getState: Ready
2
[Observer@28ba21f3 update] getState: Run
[Observer@694f9431 update] getState: Run
[Observer@f2a0b8e update] getState: Run
3
[Observer@f2a0b8e update] getState: Sleep
4
5
```

- $1 \to 2$：创建了两个观察者，并改变订阅对象的状态后，可以看到观察者获得新的订阅对象状态了
- $2 \to 3$：再创建第三个观察者，这次改变状态后有三个观察者都正常输出
- $3 \to 4$：取消前两个观察者的订阅，这次改变状态后只剩最后一个观察者更新
- $4 \to 5$：取消最后一个观察者的订阅，这次状态改变没有引起任何观察者的响应，此时订阅对象的 observers 队列也为空

# 结语

观察者模式在前后端都有广泛的应用，在前端有 MVVM 双向数据绑定的应用；后端可能作为消息队列中间件的实现核心技术。观察者模式透过维护一个观察者队列并由订阅对象主动通知所有观察者的模式，避免观察者进行**忙等待(轮询)**或**周期性检查状态**的开销，作为一种经典的异步模型基础，供大家参考。
