# 设计模式: 11 种 Behaviroal 行为型设计模式总汇

@[TOC](文章目录)

<!-- TOC -->

- [设计模式: 11 种 Behaviroal 行为型设计模式总汇](#设计模式-11-种-behaviroal-行为型设计模式总汇)
- [相关系列文章](#相关系列文章)
- [前言](#前言)
- [正文](#正文)
  - [0. 行为型设计模式概述](#0-行为型设计模式概述)
  - [1. Chain of Responsibility 职责链模式](#1-chain-of-responsibility-职责链模式)
    - [1.1 适用场景](#11-适用场景)
    - [1.2 模式结构](#12-模式结构)
    - [1.3 代码示例](#13-代码示例)
      - [1.3.1 Handlers 处理者定义](#131-handlers-处理者定义)
      - [1.3.2 测试 & 输出](#132-测试--输出)
    - [1.4 效果、特点小结](#14-效果特点小结)
  - [2. Command 命令模式](#2-command-命令模式)
    - [2.1 适用场景](#21-适用场景)
    - [2.2 模式结构](#22-模式结构)
    - [2.3 代码示例](#23-代码示例)
      - [2.3.1 Commands 命令定义](#231-commands-命令定义)
      - [2.3.2 Receiver 命令接收者](#232-receiver-命令接收者)
      - [2.3.2 Invoker 命令触发者 & 测试 & 输出](#232-invoker-命令触发者--测试--输出)
    - [2.4 效果、特点小结](#24-效果特点小结)
  - [3. Interpreter 解释器模式](#3-interpreter-解释器模式)
    - [3.1 适用场景](#31-适用场景)
    - [3.2 模式结构](#32-模式结构)
    - [3.3 代码示例](#33-代码示例)
      - [3.3.1 Expressions 表达式定义](#331-expressions-表达式定义)
      - [3.3.2 Context 上下文对象](#332-context-上下文对象)
      - [3.3.3 测试 & 输出](#333-测试--输出)
    - [3.4 效果、特点小结](#34-效果特点小结)
  - [4. Iterator 迭代器模式](#4-iterator-迭代器模式)
    - [4.1 适用场景](#41-适用场景)
    - [4.2 模式结构](#42-模式结构)
    - [4.3 代码示例](#43-代码示例)
      - [4.3.1 Iterator 抽象迭代器接口](#431-iterator-抽象迭代器接口)
      - [4.3.2 Aggregate 聚合对象接口](#432-aggregate-聚合对象接口)
      - [4.3.3 ArrayList 数组实现列表](#433-arraylist-数组实现列表)
      - [4.3.4 LinkedList 链表实现列表](#434-linkedlist-链表实现列表)
      - [4.3.5 测试 & 输出](#435-测试--输出)
    - [4.4 效果、特点小结](#44-效果特点小结)
  - [5. Mediator 中介者模式](#5-mediator-中介者模式)
    - [5.1 适用场景](#51-适用场景)
    - [5.2 模式结构](#52-模式结构)
    - [5.3 代码示例](#53-代码示例)
      - [5.3.1 Colleagues 合作对象定义](#531-colleagues-合作对象定义)
      - [5.3.2 Mediator 中介者定义](#532-mediator-中介者定义)
      - [5.3.3 测试 & 输出](#533-测试--输出)
    - [5.4 效果、特点小结](#54-效果特点小结)
  - [6. Memento 备忘录模式](#6-memento-备忘录模式)
    - [6.1 适用场景](#61-适用场景)
    - [6.2 模式结构](#62-模式结构)
    - [6.3 代码示例](#63-代码示例)
      - [6.3.1 Originator 原始对象定义](#631-originator-原始对象定义)
      - [6.3.2 Memento 备忘录 & CareTaker 备忘录管理者定义](#632-memento-备忘录--caretaker-备忘录管理者定义)
      - [6.3.3 测试 & 输出](#633-测试--输出)
    - [6.4 效果、特点小结](#64-效果特点小结)
  - [7. Observer 观察者模式](#7-observer-观察者模式)
    - [7.1 适用场景](#71-适用场景)
    - [7.2 模式结构](#72-模式结构)
    - [7.3 代码示例](#73-代码示例)
      - [7.3.1 Observers 观察者定义](#731-observers-观察者定义)
      - [7.3.2 Subjects 观察目标定义](#732-subjects-观察目标定义)
      - [7.3.3 测试 & 输出](#733-测试--输出)
    - [7.4 效果、特点小结](#74-效果特点小结)
  - [8. State 状态模式](#8-state-状态模式)
    - [8.1 适用场景](#81-适用场景)
    - [8.2 模式结构](#82-模式结构)
    - [8.3 代码示例](#83-代码示例)
      - [8.3.1 Context 上下文对象定义](#831-context-上下文对象定义)
      - [8.3.2 States 状态对象定义](#832-states-状态对象定义)
      - [8.3.3 测试 & 输出](#833-测试--输出)
    - [8.4 效果、特点小结](#84-效果特点小结)
  - [9. Strategy 策略模式](#9-strategy-策略模式)
    - [9.1 适用场景](#91-适用场景)
    - [9.2 模式结构](#92-模式结构)
    - [9.3 代码示例](#93-代码示例)
      - [9.3.1 上下文对象定义](#931-上下文对象定义)
      - [9.3.2 策略定义](#932-策略定义)
      - [9.3.3 测试 & 输出](#933-测试--输出)
    - [9.4 效果、特点小结](#94-效果特点小结)
  - [10. Template Method 模版方法模式](#10-template-method-模版方法模式)
    - [10.1 适用场景](#101-适用场景)
    - [10.2 模式结构](#102-模式结构)
    - [10.3 代码示例](#103-代码示例)
      - [10.3.1 Templates 模版定义](#1031-templates-模版定义)
      - [10.3.2 测试 & 输出](#1032-测试--输出)
    - [10.4 效果、特点小结](#104-效果特点小结)
  - [11. Visitor 访问者模式](#11-visitor-访问者模式)
    - [11.1 适用场景](#111-适用场景)
    - [11.2 模式结构](#112-模式结构)
    - [11.3 代码示例](#113-代码示例)
      - [11.3.1 Visitors 访问者定义](#1131-visitors-访问者定义)
      - [11.3.2 对象结构定义](#1132-对象结构定义)
      - [11.3.3 Elements 对象定义](#1133-elements-对象定义)
      - [11.3.4 测试 & 输出](#1134-测试--输出)
    - [11.4 效果、特点小结](#114-效果特点小结)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 相关系列文章

- [设计模式: Creational 创建型的 5 种设计模式总汇(TS 实现版本)](https://blog.csdn.net/weixin_44691608/article/details/118078424)
- [设计模式: Structural 结构型共 7 种模式总汇(TS实现)](https://blog.csdn.net/weixin_44691608/article/details/118109582)

# 前言


| 大类 | 创建型                                                                                | 结构型                                                                                                                         | 行为型                                                                                                                                                                                     |
| ---- | ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 类   | Factory Method 工厂方法                                                               | Adapter 适配器                                                                                                                 | Interpreter 解释器<br />Template Method 模版方法                                                                                                                                           |
| 对象 | Abstract Factory 抽象工厂<br />Builder 生成器<br />Prototype 原型<br />Singleton 单例 | Adapter 适配器<br />Bridge 桥接<br />Composite 组合<br />Decorator 装饰器<br />Facade 外观<br />Flyweight 享元<br />Proxy 代理 | Chain of Responsibility 职责链<br />Command 命令<br />Iterator 迭代器<br />Mediator 中介者<br />Memento 备忘录<br />Observer 观察者<br />State 状态<br />Strategy 策略<br />Visitor 访问者 |

今天来讲解最后一个部分：**11 种行为型设计模式**

# 正文

## 0. 行为型设计模式概述

**创建型模式** 负责表达对象的创建

**结构型模式** 表达对象的组织结构、关联、组合的方式

而今天要介绍的 **行为型模式** 则是一种对象间交互、调用、委派的方法描述。下面我们来分别讨论 11 种模式的使用场景和模式比较。

## 1. Chain of Responsibility 职责链模式

### 1.1 适用场景

> - 多个对象都具备处理请求的能力
> - 不明确接收者的情况下，处理对象自己决定谁来处理
> - 请求处理对象的集合可以被动态指定

### 1.2 模式结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_behavioral_structure_chain_of_responsibility.png)

- Handler 抽象请求处理者：定义请求处理接口
- ConcreteHandler 具体处理者：实现具体请求逻辑或进行请求转发

### 1.3 代码示例

#### 1.3.1 Handlers 处理者定义

- `/src/behavioral/chain_of_responsibility/handlers.ts`

首先定义处理者接口

```ts
export abstract class Handler {
  successor?: Handler

  constructor(successor?: Handler) {
    this.successor = successor
  }

  abstract handleRequest(target: RequestTargetType): void
}
```

接下来是两个具体处理者

```ts
export type RequestTargetType = 'A' | 'B'

export class ConcreteHandlerA extends Handler {
  constructor(successor?: Handler) {
    super(successor)
  }

  handleRequest(target: RequestTargetType) {
    if (target === 'A') {
      log('ConcreteHandlerA handle request')
    } else {
      log('ConcreteHandlerA pass')
      this.successor?.handleRequest(target)
    }
  }
}

export class ConcreteHandlerB extends Handler {
  constructor(successor?: Handler) {
    super(successor)
  }

  handleRequest(target: RequestTargetType) {
    if (target === 'B') {
      log('ConcreteHandlerB handle request')
    } else {
      log('ConcreteHandlerA pass')
      this.successor?.handleRequest(target)
    }
  }
}
```

两个处理者的具体处理逻辑可以选择自己处理或是转发给后继处理者

#### 1.3.2 测试 & 输出

- `/src/behavioral/chain_of_responsibility/index.ts`

接下来我们创建三种处理者对象

```ts
const handlerA = new ConcreteHandlerA()
const handlerB = new ConcreteHandlerB()
const handlerC = new ConcreteHandlerA(new ConcreteHandlerB())
```

接下来我们看看对于 A、B 两种请求三个对象的处理方式

```ts
group("handlerA.handleRequest('A')", () => {
  handlerA.handleRequest('A')
})

group("handlerB.handleRequest('A')", () => {
  handlerB.handleRequest('A')
})

group("handlerC.handleRequest('A')", () => {
  handlerC.handleRequest('A')
})

group("handlerA.handleRequest('B')", () => {
  handlerA.handleRequest('B')
})

group("handlerB.handleRequest('B')", () => {
  handlerB.handleRequest('B')
})

group("handlerC.handleRequest('B')", () => {
  handlerC.handleRequest('B')
})
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_behavioral_console_chain_of_responsibility.png)

### 1.4 效果、特点小结

- **请求者与处理者解耦**：请求者并不知道具体处理者是谁也不知道处理流程，很好的将两种对象进行解耦
- **增强对象指派灵活性**：处理对象可以以任意次序组合，增加处理流程编程灵活度
- **请求处理不确定**：由于请求可能会在职责链上无限传递下去，所以没办法确保请求一定被处理

## 2. Command 命令模式

### 2.1 适用场景

> - 适合回调机制实现：先定义，后指定函数调用位置
> - 支持指定、排列、执行请求，也就是对请求行为本身进行管理
> - 支持撤销操作，作为命令/事务回滚的基础
> - 支持修改日志，对行为本身抽象化帮助记忆
> - 创建高层原语

### 2.2 模式结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_behavioral_structure_command.png)

- Command 抽象命令接口：定义命令执行接口
- ConcreteCommand 具体命令：实现具体命令操作
- Receiver 命令接收者：帮助完成命令，或是提供命令副作用所需接口，也就是可操作外部对象
- Invoker 命令触发者：根据某种机制触发产生命令对象后执行命令

### 2.3 代码示例

#### 2.3.1 Commands 命令定义

- `/src/behavioral/command/commands.ts`

首先是命令接口

```ts
export interface Command {
  execute(): void
}
```

而第一种定义的是简单命令类型

```ts
export class SimpleCommand implements Command {
  receiver: Application
  func: keyof ApplicationReceiver

  constructor(
    receiver: Application,
    func: keyof ApplicationReceiver
  ) {
    this.receiver = receiver
    this.func = func
  }

  execute() {
    log('SimpleCommand.execute')
    this.receiver[this.func]()
  }
}
```

第二种是使用了其他上下文对象的命令

```ts
export class CommandUsingContext implements Command {
  receiver: Application

  constructor(receiver: Application) {
    this.receiver = receiver
    this.receiver.context = new Context()
  }

  execute() {
    log('CommandUsingContext.execute')
    this.receiver.context?.commandRealize()
  }
}
```

最后一种则是定义组合命令的类型

```ts
export class MacroCommand implements Command {
  receiver: Application
  commands: Command[]

  constructor(receiver: Application, ...commands: Command[]) {
    this.receiver = receiver
    this.commands = commands
  }

  execute() {
    log('MacroCommand.execute')
    this.commands.forEach((command) => command.execute())
  }
}
```

#### 2.3.2 Receiver 命令接收者

- `/src/behavioral/command/receivers.ts`

这里我们使用一个 Application 类型作为命令接收者，同时抽象出 Application 的方法接口 `ApplicationReceiver` 和内部上下文对象 `Context`

```ts
export class Context {
  commandRealize() {
    log('invoke Context.commandRealize')
  }
}

export interface ApplicationReceiver {
  commandRealize(): void
}

export class Application implements ApplicationReceiver {
  context?: Context

  commandRealize() {
    log('invoke Application.commandRealize')
  }
}
```

#### 2.3.2 Invoker 命令触发者 & 测试 & 输出

- `/src/behavioral/command/index.ts`

最后是我们的 Invoker 负责发出命令的对象

```ts
class Invoker {
  receiver: Application
  commands: Command[] = []

  constructor(receiver: Application) {
    this.receiver = receiver
  }

  simple() {
    const command = new SimpleCommand(this.receiver, 'commandRealize')
    this.commands.push(command)
    command.execute()
  }

  usingContext() {
    const command = new CommandUsingContext(this.receiver)
    this.commands.push(command)
    command.execute()
  }
}
```

还有最终的测试代码

```ts
const app = new Application()
const invoker = new Invoker(app)

invoker.simple()
invoker.usingContext()
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_behavioral_console_command.png)

### 2.4 效果、特点小结

- **调用操作对象与调用实现对象解耦**：Command 封装命令行为的描述，而 Receiver 则是封装具体数据改变接口，就好像行为模式版本的创建者模式一样
- **容易扩展、组合、复合命令**：由于发出的命令本身也是一个可装配的类，提升了命令对象的可复用性

## 3. Interpreter 解释器模式

### 3.1 适用场景

> - 对于简单文法的解析
> - 封装模式转换以提升解释效率

### 3.2 模式结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_behavioral_structure_interpreter.png)

- AbstractExpression 抽象表达式：定义表达式的共同接口
- NonterminalExpression 非终结表达式：表示可能存在子表达式的高层表达式
- TerminalExpression 终结表达式：作为 AST 叶节点的最基础表达式
- Context 上下文对象：可以记录 AST 的遍历、解析操作，将树的遍历转换成结构化顺序型的操作

### 3.3 代码示例

#### 3.3.1 Expressions 表达式定义

- `/src/behavioral/interpreter/expressions.ts`

首先需要对表达式进行定义，第一个是抽象表达式，暴露一个可解释接口

```ts
export interface Expression {
  interpret(context: Context): void
}
```

接下来我们模拟一个二元运算表达式，分别有终结符号(数字)

```ts
function record(msg: string, context: Context) {
  context.log.push(msg)
  log(`meet ${msg}`)
}

export class TerminalToken implements Expression {
  num: number

  constructor(num: number) {
    this.num = num
  }

  interpret(context: Context): void {
    record(`token(${this.num})`, context)
  }
}
```

和非终结表达式(二元表达式)

```ts
export class BinaryExpression implements Expression {
  x: TerminalToken
  y: TerminalToken
  sign: string

  constructor(x: TerminalToken, y: TerminalToken, sign: string) {
    this.x = x
    this.y = y
    this.sign = sign
  }

  interpret(context: Context) {
    const exp = `${this.x.num} ${this.sign} ${this.y.num}`
    const msg = `binaryExpression(${exp})`
    record(msg, context)

    this.x.interpret(context)
    log(`sign ${this.sign}`)
    this.y.interpret(context)
  }
}
```

#### 3.3.2 Context 上下文对象

- `/src/behavioral/interpreter/Context.ts`

接下来是上下文对象，可以用来保留遍历信息或是记录等作用

```ts
export default class Context {
  log: string[] = []
}
```

#### 3.3.3 测试 & 输出

- `/src/behavioral/interpreter/index.ts`

最后我们先创建一个用于构造二元表达式的方法(这部分正常应该由语法解析来完成)

```ts
function createBinaryExpression(s: string) {
  const [num1, sign, num2] = s.split(' ')
  const x = new TerminalToken(Number(num1))
  const y = new TerminalToken(Number(num2))
  const be = new BinaryExpression(x, y, sign)
  return be
}
```

接下来我们分别测试两个表达式并查看输出结果

```ts
group('interpret expression1', () => {
  const context = new Context()
  const expression1 = createBinaryExpression('1 + 1')
  expression1.interpret(context)
  log('context:', context)
})

group('interpret expression2', () => {
  const context = new Context()
  const expression2 = createBinaryExpression('123 + 456')
  expression2.interpret(context)
  log('context:', context)
})
```


![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_behavioral_console_interpreter.png)

### 3.4 效果、特点小结

- **对语法改变和扩展开放**：由于客户端都是透过抽象表达式接口编程，因此具体语法和顺序对客户端屏蔽
- **易于实现文法**：表达式对象直接对应语法对象，易于实现语法解析
- **难以维护复杂文法**：对于复杂文法难以维护

## 4. Iterator 迭代器模式

### 4.1 适用场景

> - 访问聚合对象内容而不暴露内部表示
> - 支持多种遍历方式
> - 为不同聚合结构提供统一的遍历接口

### 4.2 模式结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_behavioral_structure_iterator.png)

- Iterator 迭代器接口：声明迭代器的共同接口
- ConcreteIterator 具体迭代器：真正遍历聚合对象并实现迭代器接口，通常每个聚合对象都有一个专属对应的迭代器
- Aggregate 抽象聚合对象：声明创建迭代器的接口
- ConcreteAggregate 具体聚合对象：实现并返回根据自身创建的迭代器对象

### 4.3 代码示例

#### 4.3.1 Iterator 抽象迭代器接口

- `/src/behavioral/iterator/Iterator.ts`

首先我们先定义好用于遍历对象所需要的共同迭代接口，这里用上了 TS 的范型特性

```ts
export default interface Iterator<Data> {
  first(): Data | null
  current(): Data | null
  next(): Data | null
  isDone(): boolean
}
```

#### 4.3.2 Aggregate 聚合对象接口

- `/src/behavioral/iterator/Aggregate.ts`

接下来定义聚合对象的接口，由于是测试用所以就只实现 add 添加方法，实际上我们可以将聚合对象的接口与迭代器接口分离，透过多实现的方式来组合不同接口

```ts
export default interface Aggregate<Data> {
  add(data: Data): void
  iterator(): Iterator<Data>
}
```

#### 4.3.3 ArrayList 数组实现列表

- `/src/behavioral/iterator/ArrayList.ts`

接下来我们实现一个数组实现的列表，就好像 Java 里面的 `ArrayList` 一般(当然其实在 JS 中数组本身已经是动态扩展大小了，所以真的没什么必要hh)

```ts
export default class ArrayList<Data> implements Aggregate<Data> {
  dataList: Data[] = []

  add(data: Data) {
    this.dataList.push(data)
  }

  iterator() {
    return new ArrayListIterator<Data>(this.dataList)
  }
}
```

基本的数组列表完成之后，还需要为该类特别实现一个专属的迭代器，它能够理解聚合对象的内部表示(即封装对于内部表示的遍历)

```ts
export class ArrayListIterator<Data> implements Iterator<Data> {
  dataList: Data[]
  currentIdx: number

  constructor(dataList: Data[]) {
    this.dataList = dataList
    this.currentIdx = 0
  }

  first() {
    return this.dataList.length ? this.dataList[0] : null
  }

  current() {
    return this.isDone() ? null : this.dataList[this.currentIdx]
  }

  next() {
    if (!this.isDone()) {
      this.currentIdx++
      return this.dataList[this.currentIdx]
    }
    return null
  }

  isDone() {
    return this.currentIdx >= this.dataList.length
  }
}
```

#### 4.3.4 LinkedList 链表实现列表

- `/src/behavioral/iterator/LinkedList.ts`

第二种聚合对象为链表，这里就不加赘述直接上代码

```ts
class Node<Data> {
  data: Data
  next: Node<Data> | null

  constructor(data: Data) {
    this.data = data
    this.next = null
  }
}

export default class LinkedList<Data> implements Aggregate<Data> {
  head: Node<Data> | null = null
  tail: Node<Data> | null = null

  add(data: Data) {
    if (!this.head) {
      this.tail = this.head = new Node(data)
    } else {
      this.tail!.next = new Node(data)
      this.tail = this.tail!.next
    }
  }

  iterator() {
    return new LinkedListIterator(this.head)
  }
}
```

以及链表专属的迭代器类

```ts
class LinkedListIterator<Data> implements Iterator<Data> {
  head: Node<Data> | null
  currentNode: Node<Data> | null

  constructor(head: Node<Data> | null) {
    this.head = head
    this.currentNode = head
  }

  first() {
    return this.head ? this.head.data : null
  }

  current() {
    return this.currentNode ? this.currentNode.data : null
  }

  next() {
    if (!this.isDone()) {
      this.currentNode = this.currentNode!.next
    }
    return this.current()
  }

  isDone() {
    return !this.currentNode
  }
}
```

#### 4.3.5 测试 & 输出

- `/src/behavioral/iterator/index.ts`

最后我们定义一个透过接口遍历聚合对象的方法

```ts
function traversal<Data>(iterator: Iterator<Data>) {
  let i = 0
  while (!iterator.isDone()) {
    log(`${i++}:`, iterator.current())
    iterator.next()
  }
}
```

然后我们创建两个对象并调用方法遍历

```ts
const arrayList: Aggregate<number> = new ArrayList<number>()
arrayList.add(1)
arrayList.add(3)
arrayList.add(5)
arrayList.add(7)
arrayList.add(9)

const linkedList: Aggregate<number> = new LinkedList<number>()
linkedList.add(2)
linkedList.add(4)
linkedList.add(6)
linkedList.add(8)
linkedList.add(10)

group('traversal arrayList', () => {
  traversal(arrayList.iterator())
})
group('traversal linkedList', () => {
  traversal(linkedList.iterator())
})
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_behavioral_console_iterator.png)

### 4.4 效果、特点小结

- **支持多种遍历方法**：具体遍历操作封装在迭代器内部，对用户是屏蔽的，可以透过暴露不同遍历接口的方法提供用户不同的遍历操作实现
- **简化聚合对象接口**：独立出遍历用的迭代器接口之后，可以简化聚合对象本身用于访问内容的接口
- **同时实现多个遍历**：由于返回的是具体的遍历"对象"，每个对象各自维护遍历状态，相互独立

## 5. Mediator 中介者模式

### 5.1 适用场景

> - 多个组件存在复杂的交互关系，耦合紧密
> - 多个对象之间直接通信，导致组件复用性不高
> - 定制访问组件的行为

### 5.2 模式结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_behavioral_structure_mediator.png)

- Colleague 合作对象接口：在中介者模式中我们称呼组件为合作对象，原因是实际上我们需要多个合作对象共同完成任务，又不希望对象直接过度耦合
- ConcreteColleague 具体合作对象：具体合作对象之间的通信都必须透过中介者对象，也就使得每个具体合作对象的接口和接口实现操作提升可复用性，同时降低对于通信目标的耦合
- Mediator 中介者接口：定义中介者接口，也是合作对象的消费目标
- ConcreteMediator 具体中介者对象：由于实际上中介者对象往往为合作对象提供许多接口，也与多个合作对象之间紧密关联，所以应用的时候抽象中介者的易用性不一定很高，也就不需要独立出一个抽象中介者接口

### 5.3 代码示例

#### 5.3.1 Colleagues 合作对象定义

- `/src/behavioral/mediator/colleagues.ts`

首先中介者模式中最重要的就是位于底层的具体合作对象与其接口，首先是合作对象共同接口

```ts
export abstract class Colleague {
  mediator?: Mediator

  changed(): void {
    this.mediator?.colleagueChanged(this)
  }
}
```

透过抽象合作对象维护对于中介者的引用，对于具体合作对象只需要在特定时机调用该接口通知中介者对象即可

下面是三个具体的合作对象

```ts
export class Colleague1 extends Colleague {
  receive(invoker: string) {
    log(`Colleague1.receive from ${invoker}`)
  }

  broadCast() {
    this.changed()
  }
}

export class Colleague2 extends Colleague {
  receive(invoker: string) {
    log(`Colleague2.receive from ${invoker}`)
  }

  broadCast() {
    this.changed()
  }
}

export class Colleague3 extends Colleague {
  receive(invoker: string) {
    log(`Colleague3.receive from ${invoker}`)
  }

  broadCast() {
    this.changed()
  }
}
```

#### 5.3.2 Mediator 中介者定义

- `/src/behavioral/mediator/Mediator.ts`

接下来就是我们的主角，负责维护多个合作对象的合作，同时也表示中介者需要知道更多关于实际合作对象的信息；所以实际上对于中介者对象来说，为合作对象抽象共同接口也没有什么意义

```ts
export default class Mediator {
  colleague1: Colleague1
  colleague2: Colleague2
  colleague3: Colleague3

  constructor(c1: Colleague1, c2: Colleague2, c3: Colleague3) {
    this.colleague1 = c1
    this.colleague2 = c2
    this.colleague3 = c3
  }

  colleagueChanged(colleague: Colleague): void {
    log('passing message by mediator')
    if (colleague instanceof Colleague1) {
      const invoker = 'colleague1'
      this.colleague2.receive(invoker)
      this.colleague3.receive(invoker)
    } else if (colleague instanceof Colleague2) {
      const invoker = 'colleague2'
      this.colleague1.receive(invoker)
      this.colleague3.receive(invoker)
    } else if (colleague instanceof Colleague3) {
      const invoker = 'colleague3'
      this.colleague1.receive(invoker)
      this.colleague2.receive(invoker)
    } else {
      throw new Error('unknown Colleague type')
    }
  }
}
```

#### 5.3.3 测试 & 输出

- `/src/behavioral/mediator/index.ts`

最后我们首先组合出一个包含三个合作对象的中介者对象

```ts
const c1 = new Colleague1()
const c2 = new Colleague2()
const c3 = new Colleague3()
const mediator = new Mediator(c1, c2, c3)
c1.mediator = mediator
c2.mediator = mediator
c3.mediator = mediator
```

然后分别触发三个对象的额外接口，观察对象间的信息传递

```ts
group('colleague1.broacast', () => {
  c1.broadCast()
})

group('colleague2.broacast', () => {
  c2.broadCast()
})

group('colleague3.broacast', () => {
  c3.broadCast()
})
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_behavioral_console_mediator.png)

### 5.4 效果、特点小结

- **减少子类生成**：组件对象的通信和高层逻辑都被封装到中介者对象了，所以合作对象的定义可以专注于可复用性高、高内聚的组件设计
- **合作对象解耦**：屏蔽了合作对象之间的具体联系，所有合作对象共同依赖中介者而不相互依赖
- **简化对象协议**：取代合作对象间耦合的是中介者暴露给具体合作对象的接口，一定程度上大大简化了对象间的通信宽度
- **控制集中化**：中介者对象掌管所有合作对象以及对象之间的交互，也就是将对象的控制逻辑集中到单一个中介者对象之中

## 6. Memento 备忘录模式

### 6.1 适用场景

> - 记录对象状态并需要复原(生产快照、备份)
> - 封装对象内部状态，保证数据完整性和安全性

### 6.2 模式结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_behavioral_structure_memento.png)

- Originator 原始对象：拥有自身内部状态的对象
- Memento 备忘录：记录对象内部状态快照的对象
- CareTaker 对象管理者：负责保留存储对象的管理者

### 6.3 代码示例

#### 6.3.1 Originator 原始对象定义

- `/src/behavioral/memento/originators.ts`

首先当然是我们的内部对象，这里使用一个 `version` 属性来象征对象的内部状态

```ts
export interface State {
  version: string
}

export class Originator {
  state: State = { version: 'v1.0.0' }

  upgrade(level: 'S' | 'M' | 'L' = 'S') {
    const [a, b, c] = this.state.version.split('.')
    let newVersion = this.state.version
    switch (level) {
      case 'S':
        newVersion = `${a}.${b}.${Number(c) + 1}`
        break
      case 'M':
        newVersion = `${a}.${Number(b) + 1}.${c}`
        break
      case 'L':
        newVersion = `${a[0]}${Number(a.substring(1)) + 1}.${b}.${c}`
        break
    }
    this.state.version = newVersion
  }

  createMemento(): Memento {
    log('memorized:', this.state)
    return new Memento(this.state)
  }

  restoreMemento(memento: Memento) {
    this.state = memento.state
  }
}
```

#### 6.3.2 Memento 备忘录 & CareTaker 备忘录管理者定义

- `/src/behavioral/memento/mementos.ts`

接下来是备忘录对象，保有原始对象的内部状态记录/快照

```ts
export class Memento {
  state: State

  constructor(state: State) {
    this.state = { ...state }
  }

  setState(state: State) {
    this.state = { ...state }
  }
}
```

以及一个用于保存状态的管理者对象

```ts
export class CareTaker {
  memento: Memento | null = null

  getMemento() {
    return this.memento
  }

  setMemento(memento: Memento) {
    this.memento = memento
  }
}
```

实际上 JS 中的属性都是公开的，所以实际应用场景可以尝试使用闭包的方式来实现

#### 6.3.3 测试 & 输出

- `/src/behavioral/memento/index.ts`

接下来我们就模拟一个版本不断迭代的应用，并在某个时间回退到上一次快照的版本

```ts
const careTaker = new CareTaker()
const originator = new Originator()

log(originator)

originator.upgrade()
log(originator)
careTaker.setMemento(originator.createMemento())

originator.upgrade()
log(originator)

originator.upgrade()
log(originator)

const memo = careTaker.getMemento()
memo && originator.restoreMemento(memo)
log(originator)
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_behavioral_console_memento.png)

### 6.4 效果、特点小结

- **保持封装边界**：备忘录模式的宗旨在于使内部状态对外部对象屏蔽，保证内部状态的完整性和私密性
- **可能存在巨大代价**：备忘录并没有限制状态大小和数量，需要非常小心保存过多的状态/快照可能存在存储开销问题(可以使用享元模式改善)

## 7. Observer 观察者模式

### 7.1 适用场景

> - 观察者对象依赖于目标对象，又希望目标对象改变的同时自动通知所有观察者
> - 目标对象不知道到底有多少对象依赖于自己

### 7.2 模式结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_behavioral_structure_observer.png)

- Observer 观察者对象：负责维护对于目标对象的指针，并定义观察者更新的共同接口
- ConcreteObserver 具体观察者：实现对于观察目标状态改变时的应对逻辑
- Subject 可观察对象接口：维护一个观察者对象队列，并定义状态改变时需要作用的通知方法
- ConcreteSubject 具体可观察对象：具体对象逻辑，与观察者解耦，只需要在特定时机调用通知方法即可

### 7.3 代码示例

#### 7.3.1 Observers 观察者定义

- `/src/behavioral/observer/observers.ts`

首先观察者模式的主角当然是我们的观察者咯，定义了响应目标对象变化的接口与具体实现

```ts
export abstract class Observer {
  subject: Subject | null = null

  observe(subject: Subject) {
    if (this.subject) {
      this.subject.detach(this)
    }
    this.subject = subject
    this.subject.attach(this)
  }

  abstract update(): void
}

export class ConcreteObserver extends Observer {
  static count = 0
  id = ++ConcreteObserver.count

  update() {
    log(`ConcreteObserver(${this.id}) update`)
  }
}
```

#### 7.3.2 Subjects 观察目标定义

- `/src/behavioral/observer/subjects.ts`

接下来是可观察目标的定义，我们需要很小心的设计可观察目标的接口，以确保观察者对象的可复用性

首先使用抽象类自动维护观察者队列和通知模型

```ts
export class Subject {
  observers: Observer[] = []

  attach(observer: Observer) {
    if (!this.observers.includes(observer)) {
      this.observers.push(observer)
    }
  }

  detach(observer: Observer) {
    if (this.observers.includes(observer)) {
      this.observers.splice(this.observers.indexOf(observer), 1)
    }
  }

  notify() {
    this.observers.forEach((observer) => observer.update())
  }
}
```

接下来则是具体可观察对象的实现

```ts
export class ConcreteSubject extends Subject {
  static count = 0
  id = ++ConcreteSubject.count

  change() {
    log(`ConcreteSubject(${this.id}) changed`)
    this.notify()
  }
}
```

这里使用继承的方式实现观察者模式，实际上使用组合能提供更好的灵活性和可扩展性

#### 7.3.3 测试 & 输出

- `/src/behavioral/observer/index.ts`

最后我们分别定义两个观察目标和三个观察者对象

```ts
const subject1 = new ConcreteSubject()
const subject2 = new ConcreteSubject()

const observer1 = new ConcreteObserver()
observer1.observe(subject1)
const observer2 = new ConcreteObserver()
observer2.observe(subject2)
const observer3 = new ConcreteObserver()
observer3.observe(subject2)
```

然后看看观察目标改变的时候会发生什么事(观察者收到通知并进行相应操作)

```ts
subject1.change()
subject2.change()
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_behavioral_console_observer.png)

### 7.4 效果、特点小结

- **目标和观察者抽象耦合**：目标通知观察者的方法仅根据接口，而观察者则是统一暴露更新接口，也与具体实现无关
- **支持广播**：由于观察者并不知道对于同一目标的其他观察者，目标对象可以很轻易的实现对于所有观察者的广播操作
- **多余的更新**：由于观察者状态对于观察目标是屏蔽的，可能造成多余不必要的观察者更新，这时候就要再对观察目标的状态有更细致的区分

## 8. State 状态模式

### 8.1 适用场景

> - 对象的行为取决于他的状态，并且需要在运行时动态的改变状态
> - 对象多个行为存在相似的关于状态的条件分支结构

### 8.2 模式结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_behavioral_structure_state.png)

- State 状态接口：声明状态相关操作的接口
- ConcreteState 具体状态对象：定义不同状态下的具体操作逻辑
- Context 上下文对象：需要根据不同状态展现不同行为的对象，也就是方法逻辑抽象依赖于状态对象

### 8.3 代码示例

#### 8.3.1 Context 上下文对象定义

- `/src/behavioral/state/index.ts`

首先我们先明确一下上下文对象的逻辑：根据连接到某个远程对象，需要根据连接状态表示请求成功与否

```ts
export default class Context {
  state: State = new Disconnected()

  connect() {
    this.state = new Connected()
    log('Context is Connected')
  }

  request() {
    log('invoke Context.requrest')
    this.state.handle()
  }
}
```

#### 8.3.2 States 状态对象定义

- `/src/behavioral/state/states.ts`

为了避免出现多余的条件分支语句，我们将连线状态提取成一个独立的对象，展现不同的行为

```ts
export interface State {
  handle(): void
}

export class Connected implements State {
  handle() {
    log('[Connected] handle success')
  }
}

export class Disconnected implements State {
  handle() {
    log('[Disconnected] handle fail')
  }
}
```

#### 8.3.3 测试 & 输出

- `/src/behavioral/state/index.ts`

```ts
const context = new Context()

context.request()
context.connect()
context.request()
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_behavioral_console_state.png)

### 8.4 效果、特点小结

- **状态与相关行为的局部封装**：将状态和相关行为封装成一个独立的对象
- **显式状态转换**：状态抽象成对象之后，状态的转换逻辑更加清晰(抽换状态对象)
- **状态可共享**：对于无实例变量的状态，可以被多个对象共享、复用

## 9. Strategy 策略模式

### 9.1 适用场景

> - 多个相关对象仅仅存在"行为差异"，即决策不同
> - 对同一算法实现不同变体、算法实现
> - 算法使用数据和逻辑对客户屏蔽

### 9.2 模式结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_behavioral_structure_strategy.png)

- Strategy 策略类：定义用于处理、调用某种算法的接口
- ConcreteStrategy 具体策略类：实现具体的算法逻辑
- Context 上下文对象：透过利用算法策略对象来完成业务需求

### 9.3 代码示例

#### 9.3.1 上下文对象定义

- `/src/behavioral/strategy/Context.ts`

一样我们先明确上下文对象到底要完成什么业务

```ts
export default class Context {
  name = 'sUpErFrEe'
  strategy: Strategy = new Default()

  setStrategy(strategy: Strategy) {
    this.strategy = strategy
  }

  toString() {
    return `{ name: ${this.strategy.operation(this.name)} }`
  }
}
```

在本示例中是要完成一个对于内部字符串的格式化输出操作

#### 9.3.2 策略定义

- `/src/behavioral/strategy/strategys.ts`

接下来我们为不同格式化策略定义不同的策略类实现

首先是抽象接口定义

```ts
export interface Strategy {
  operation(s: string): void
}
```

第一种策略是默认直接返回原字符串

```ts
export class Default implements Strategy {
  operation(s: string) {
    return s
  }
}
```

第二种是转大写

```ts
export class UpperCase implements Strategy {
  operation(s: string) {
    return s.toUpperCase()
  }
}
```

第三种是转小写

```ts
export class LowerCase implements Strategy {
  operation(s: string) {
    return s.toLowerCase()
  }
}
```

最后一种是转为首字母大写操作

```ts
export class Capitalize implements Strategy {
  operation(s: string) {
    const lower = s.toLowerCase()
    return `${lower[0].toUpperCase()}${lower.substring(1)}`
  }
}
```

#### 9.3.3 测试 & 输出

- `/src/behavioral/strategy/index.ts`

最后我们可以看看在不同的策略之下的输出

```ts
const upperCase = new UpperCase()
const lowerCase = new LowerCase()
const capitalize = new Capitalize()

const context = new Context()
log(`context: ${context}`)

context.setStrategy(upperCase)
log(`context: ${context}`)

context.setStrategy(lowerCase)
log(`context: ${context}`)

context.setStrategy(capitalize)
log(`context: ${context}`)
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_behavioral_console_strategy.png)

### 9.4 效果、特点小结

- **算法系列**：上下文仅根据策略接口调用，所以可以实现更多不同的算法实现策略系列
- **替代继承的方法**：透过继承也可以满足对于算法的不同实现，但是使用策略模式就是一种利用组合替代继承的实现方式
- **消除条件分支**：原本需要根据条件分支选择的策略，现在直接利用了多态来完成
- **实现的选择**：由于将策略独立出来变成具体对象，则需要透过其他模式来创建、管理现有的策略类和策略的调度、选择
- **通信开销**：从原本的单方法变成对于具体策略对象的委派制度，增加了对象的通信开销
- **对象膨胀**：策略逻辑独立成一个个新的对象，可能产生多余的策略对象，则可以使用如单例模式、享元模式来优化策略对象的额外开销

## 10. Template Method 模版方法模式

### 10.1 适用场景

> - 提炼出算法中不变的部分，将可变行为开放给子类实现，同时避免共同逻辑的重复定义
> - 也称作 **钩子模式(hook)** 相当于暴露出一个可选的钩子函数，供子类覆盖

### 10.2 模式结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_behavioral_structure_template_method.png)

- Template 抽象模版类：提炼并集中共同、不变的算法部分
- ConcreteTemplate 具体模版类：可选实现模版类暴露的抽象接口/钩子

### 10.3 代码示例

#### 10.3.1 Templates 模版定义

- `/src/behavioral/template_method/templates.ts`

首先是保有共同逻辑的抽象模版类

```ts
export abstract class ProcessTemplate {
  process() {
    this.step1()
    this.step2()
    this.step3()
  }

  step1() {
    log('default step2')
  }

  abstract step2(): void

  step3() {
    log('default step3')
  }
}
```

接下来是两种不同的模版实现，需要覆盖必须的 step2，而可选的覆盖 step1 和 step3

```ts
export class ProcessA extends ProcessTemplate {
  step2() {
    log('Custom step2 from ProcessA')
  }
}

export class ProcessB extends ProcessTemplate {
  step1() {
    log('Custom step1 from ProcessB')
  }

  step2() {
    log('Custom step2 from ProcessB')
  }
}
```

#### 10.3.2 测试 & 输出

- `/src/behavioral/template_method/index.ts`

最后我们可以看到不同的模版实现，透过重写钩子函数来修改算法的内部流程

```ts
const templateA: ProcessTemplate = new ProcessA()
const templateB: ProcessTemplate = new ProcessB()

group('templateA process', () => {
  templateA.process()
})

group('templateB process', () => {
  templateB.process()
})
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_behavioral_console_template_method.png)

### 10.4 效果、特点小结

- **反向控制结构**：父类直接调用内部方法的实现，而将内部方法的具体实现交由子类实现(也可以提供默认实现作为钩子存在)

## 11. Visitor 访问者模式

### 11.1 适用场景

> - 需要对操作目标实现更具体的访问(依赖于具体目标对象类型)
> - 对统一对象结构针对不同对象类型进行访问
> - **双重委派模型**：实现暴露具体类型的访问操作

### 11.2 模式结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_behavioral_structure_visitor.png)

- ObjectStructure 对象结构：可能是一个聚合对象，也可能是一个组合对象，提供用于访问内部对象的接口
- Element 内部对象抽象：声明接受一个访问者的接口(accept)
- ConcreteElement 具体内部对象：透过调用访问者的特定接口来实现暴露自身具体类型
- Visitor 抽象访问者：声明访问不同具体类型的接口
- ConcreteVisitor 具体访问者：可选的对不同的访问操作进行实现

### 11.3 代码示例

#### 11.3.1 Visitors 访问者定义

- `/src/behavioral/visitor/visitors.ts`

首先最重要的就是我们的访问者，先来看看访问者的接口

```ts
export abstract class Visitor {
  visitElementA(_: ElementA) {}
  visitElementB(_: ElementB) {}
}
```

两个方法分别接受不同的访问目标类型

然后是两个实际的访问者对象，分别重写一个方法

```ts
export class ConcreteVisitorA extends Visitor {
  visitElementA(elementA: ElementA) {
    log('ConcreteVisitorA visit', elementA)
  }
}

export class ConcreteVisitorB extends Visitor {
  visitElementB(elementB: ElementB) {
    log('ConcreteVisitorB visit', elementB)
  }
}
```


#### 11.3.2 对象结构定义

- `/src/behavioral/visitor/ObjectStructure.ts`

接下来定义一个对象组合结构，本示例就是一个简单的数组列表

```ts
export default class ObjectStructure {
  elements: Element[] = []

  accept(visitor: Visitor) {
    this.elements.forEach((element) => element.accept(visitor))
  }
}
```

#### 11.3.3 Elements 对象定义

- `/src/behavioral/visitor/elements.ts`

最后定义两个具体对象，并且 `accept` 方法分别调用不同的访问者方法

```ts
export interface Element {
  accept(visitor: Visitor): void
}

export class ElementA implements Element {
  static count = 1
  id = ElementA.count++

  accept(visitor: Visitor) {
    visitor.visitElementA(this)
  }
}

export class ElementB implements Element {
  static count = 1
  id = ElementB.count++

  accept(visitor: Visitor) {
    visitor.visitElementB(this)
  }
}
```

#### 11.3.4 测试 & 输出

- `/src/behavioral/visitor/index.ts`

输出中我们就能看到，实际上对于每个对象都会调用某个访问者的方法来表示确实走过，但是我们的访问者可以提供缺省方法和方法重写来允许指定访问者对于指定对象的遍历行为

```ts
const os = new ObjectStructure()
os.elements.push(new ElementA())
os.elements.push(new ElementA())
os.elements.push(new ElementB())
os.elements.push(new ElementA())
os.elements.push(new ElementB())
os.elements.push(new ElementB())
os.elements.push(new ElementB())

const visitorA = new ConcreteVisitorA()
const visitorB = new ConcreteVisitorB()

group('visitorA', () => {
  os.accept(visitorA)
})

group('visitorB', () => {
  os.accept(visitorB)
})
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/design_pattern_behavioral_console_template_visitor.png)

### 11.4 效果、特点小结

- **易于增加新的操作**：要想增加新的访问操作，只需要增加一个新的方法，或是添加一个新的继承类即可
- **将访问方法集中**：将访问对象的相关逻辑集中、内聚到一个或是多个相关的对象之内
- **通过类层次进行访问**：实际上访问者模式就是将整个遍历操作留下许多 **钩子** 方法，从而使得我们可以在遍历的行为上进行扩展；也就是说我们可以在对象结构(ObjectStructure)遍历的部分实现迭代器模式就可以对遍历方法的扩展开放
- **累积状态**：透过对遍历访问扩展开放，我们可以在对象之外累计遍历过程和结果
- **破坏封装**：具体访问者实际上会穿透对象接口直接看到实际的对象类型，实际上属于一种对于封装私密性的破坏，但同时这正是我们期望的能针对具体类型进行访问的模式

# 结语

本篇介绍完设计模式的最后一个部分：**行为型设计模式**，然而设计模式并不止于此，我们也无需为了模式而模式，而是根据需求选择相应需要的模式来进行组合、扩展，帮助我们更好的理清对象之间的交互模式才是设计模式的本质

# 其他资源

## 参考连接

| Title                                                           | Link |
| --------------------------------------------------------------- | ---- |
| Design Patterns - Elements of Reusable Object-Oriented Software | []() |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/design_pattern_js/src/behavioral](https://github.com/superfreeeee/Blog-code/tree/main/design_pattern_js/src/behavioral)
