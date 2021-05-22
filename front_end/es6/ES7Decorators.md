# ES7 提案: Decorators 装饰器

@[TOC](文章目录)

<!-- TOC -->

- [ES7 提案: Decorators 装饰器](#es7-提案-decorators-装饰器)
- [前言](#前言)
- [正文](#正文)
  - [1. Decorator 装饰器使用规范](#1-decorator-装饰器使用规范)
    - [1.1 装饰器的使用形式 & 具体行为](#11-装饰器的使用形式--具体行为)
    - [1.2 为何不能修饰函数？](#12-为何不能修饰函数)
  - [2. 详细说明 & 代码示例](#2-详细说明--代码示例)
    - [2.1 类装饰器 Class Decorators](#21-类装饰器-class-decorators)
      - [2.1.1 传入参数 & 基础用法](#211-传入参数--基础用法)
      - [2.1.2 装饰器返回值](#212-装饰器返回值)
      - [2.1.3 装饰器表达式](#213-装饰器表达式)
      - [2.1.4 装饰器注入原型方法](#214-装饰器注入原型方法)
      - [2.1.5 装饰器实现混入(Mixin)模式](#215-装饰器实现混入mixin模式)
    - [2.2 类方法装饰器 Class Method Decorators](#22-类方法装饰器-class-method-decorators)
      - [2.2.1 传入参数 & 基础用法](#221-传入参数--基础用法)
      - [2.2.2 readonly 只读属性](#222-readonly-只读属性)
      - [2.2.3 logger 日志装饰器](#223-logger-日志装饰器)
      - [2.2.4 autobind 绑定实例](#224-autobind-绑定实例)
    - [2.3 补充：类实例属性、访问器属性装饰器](#23-补充类实例属性访问器属性装饰器)
  - [3. 特性总结](#3-特性总结)
  - [补充：使用环境配置注意事项 & 三方库应用](#补充使用环境配置注意事项--三方库应用)
    - [MobX](#mobx)
    - [core-decorators](#core-decorators)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

今天我们来说说一个 ES7 提出的实验性特性，截止目前为止还处于 stage-2 的 **Decorators 装饰器**。他的使用形式就好像 Java 里面的注解(Annotation)一样，然而其实现机制和能力却又比 Java 的标记型注解要强大许多，下面我们就来看看 ES7 装饰器的具体用法和效果。

# 正文

## 1. Decorator 装饰器使用规范

首先我们先看看装饰器的使用形态和使用范围，装饰器的表达式如下

```js
@<decorator-expression>
```

使用 `@` 符号加上一个返回一个函数的表达式

装饰器通常是用于 **"修饰"** 某个目标，也就是为某个目标添加一些特性，同时装饰器能够装饰的目标也被局限为下列两种

1. 类(ES6 的 class)
2. 类属性
   1. 类实例属性(field)
   2. 类方法(method)
   3. 类访问器属性(accessor = getter/setter)

下面在进入实际的代码测试环节之前，我们先看看几个表现特性和前提

### 1.1 装饰器的使用形式 & 具体行为

前面提过装饰器实际上可以说是仅仅作为 ES6 的 class 的扩展属性，因为他不能用于装饰 ES5 以前的函数类，也不能用于装饰一般变量

所以说实际上装饰器的使用形式大致就是以下几种

```js
@classDecorator
class MyClass {

    @fieldDecorator
    myField = 0

    @methodDecorator
    f() {}

    @accessorDecorator
    get otherField() {}
}
```

而我们用来装饰目标的装饰器其本质上就是一个函数，同时根据装饰目标对象的不同接受不同的参数如下：

```js
// 类装饰器
function classDecorator(target) {/* ... */}
// 类属性装饰器
function methodDecorator(target, name, description) {/* ... */}
```

后面我们会再详细说明不同装饰器接受的参数和返回值对装饰目标的影响

### 1.2 为何不能修饰函数？

细心的人可能会注意到，装饰器这么好用，但是却不能修饰一般对象，这是为什么呢？

我们看看如下代码段，假设装饰器可以装饰普通函数的话会发生什么事：

```js
var counter = 0;

var add = function () {
  counter++;
};

@add
function foo() {}
```

按代码顺序我们可能预期的是：

```
1. 定义 counter 变量
2. 定义 add 装饰器
3. 定义 foo 函数并用 add 装饰，counter 记录方法数 +1
```

然而普通的函数其实存在所谓的 **函数提升**，也就是说实际作用的代码段应该如下

```js
var counter;
var add;

@add
function foo() {}

counter = 0;
add = function() {
    counter++
}
```

这时候实际上 `foo` 方法真正定义并存在的时候 `add` 装饰器还是空的，甚至 `counter` 变量都不一定被初始化好，所以实际上 `counter` 的结果为 0。

## 2. 详细说明 & 代码示例

好了基础认识都差不多了，下面来看看装饰器在不同目标上的具体行为与特性

### 2.1 类装饰器 Class Decorators

首先第一种我们先来看看目标为 ES6 的 class 时的 **类装饰器(Class Decorator)** 实现。

#### 2.1.1 传入参数 & 基础用法

首先我们可以先用一个 `test` 装饰器来测试一下接受哪些参数了

- `src/class/basic.js`

```js
function test(...args) {
  let i = 0
  args.forEach((arg) => log(i++, arg))
}

@test
class MyClass {}
```

```
0 [Function: MyClass]
```

我们看到作为类装饰器的时候只接受一个参数，也就是装饰的类定义本身，也就是说我们可以在这个阶段对类直接添加一些属性如下

```js
function testable(target) {
  target._isTestable = true
}

@testable
class TestableClass {}

class OtherClass {}

log('TestableClass:             ', TestableClass)
log('TestableClass._isTestable: ', TestableClass._isTestable)

log('OtherClass:             ', OtherClass)
log('OtherClass._isTestable: ', OtherClass._isTestable)
```

```
TestableClass:              [Function: TestableClass] { _isTestable: true }
TestableClass._isTestable:  true
OtherClass:              [Function: OtherClass]
OtherClass._isTestable:  undefined
```

我们定义了 `testable` 的装饰器，用于对对象添加 `_isTestable` 标记，这个用法就跟 Java 中的注解比较相似，仅仅是对目标类型打上一些标记。

还记得 ES6 的 class 仅仅是作为 ES5 以前的函数类的语法糖，也就是说 `TestableClass、OtherClass` 其实本质上就是作为类的构造方法如下

```js
function TestableClass() {}
function OtherClass() {}
```

而这时候我们添加的 `TestableClass_isTestable`、`OtherClass._isTestable` 是直接添加到构造方法的属性，对于类型来说就是只能透过类名访问的静态属性

#### 2.1.2 装饰器返回值

既然我们已经知道装饰器实际上就是定义一个函数来对类进行修饰，那我就有点好奇，作为装饰器的函数的返回值又会有什么影响呢？

- `src/class/return.js`

```js
import { log } from '../utils'

function replaceWithPrimitive(target) {
  return 123
}

function replaceWithObject(target) {
  return { target }
}

@replaceWithPrimitive
class A {}

@replaceWithObject
class B {}

log('class A: ', A)
log('class B: ', B)
```

```
class A:  123
class B:  { target: [Function: B] }
```

这时候我们定义两个装饰器，一个是 `replaceWithPrimitive`、一个是 `replaceWithObject`，分别返回基础类型和一个新的对象，我们发现输出也不管，返回的是啥就是啥，加上前一小节无返回值的装饰器，我们就可以说实际上一个被修饰的类对象其实上与下列表达式等价

```js
@func
class A {}

/* 等价于 */
const A = func(A) || A
```

这个特性实际上就为装饰器的应用敞开了大门，甚至可以以装饰器作为函数构造代理的应用，不过本篇就不再做探讨，知道就行。

#### 2.1.3 装饰器表达式

前面我们提过，`@` 符号后面接的是一个返回函数的表达式，也就是说我们不一定要直接使用装饰器函数的名字，只要传入一个 **能返回装饰器函数的表达式** 即可：

- `src/class/wrapper.js`

```js
import { log } from '../utils'

function bindColor(color) {
  return (target) => {
    target._color = color
  }
}

@bindColor('red')
class Red {}

@bindColor('green')
class Green {}

@bindColor('Blue')
class Blue {}

log('Red:   ', Red)
log('Green: ', Green)
log('Blue:  ', Blue)
```

我们定义一个 `bindColor` 方法，传入 `color` 后返回将 `color` 绑定到类定义上的装饰器函数，看看效果

```
Red:    [Function: Red] { _color: 'red' }
Green:  [Function: Green] { _color: 'green' }
Blue:   [Function: Blue] { _color: 'Blue' }
```

我们可以看到每个类都绑定了自己的属性，透过 `bindColor` 方法使得同一个装饰器函数能够被多次复用，也使装饰器函数的使用更加灵活

#### 2.1.4 装饰器注入原型方法

前面提过对于 **类装饰器(Class Decorator)** 函数接受的参数只有一个就是类定义本身，前面的示例我们为类定义添加静态属性，下面我再告诉你他还能访问类定义的 `prototype` 属性进而添加甚至修改原型方法

- `src/class/proto.js`

```js
import { log } from '../utils'

function info(target) {
  target.prototype.greeting = function () {
    console.log(`This is class ${this.name}`)
  }
}

@info
class A {}

const a = new A()
a.name = 'a instance of class A'

console.log('A: ', A)
console.log('a: ', a)
a.greeting()
```

这里我们透过 `target.prototype.greeting = function () {/* ... */}` 来对 `class A` 添加一个原型方法 `greeting`，这时候我们就可以在类实例上调用 `a.greeting` 原型方法，输出：

```
A:  [Function: A]
a:  A { name: 'a instance of class A' }
This is class a instance of class A
```

也就是说我们透过拿到的 `target` 类定义可以对类型进行非常灵活的方法扩展和静态标记，而这有没有让你想起前端领域非常常见的 `混入(Mixin)设计模式`，下面来看看。

#### 2.1.5 装饰器实现混入(Mixin)模式

装饰器实际上就是一个非常适合实现混入设计模式的地点。在混入设计模式中，我们通常是对于某个现存的类型进行原型方法的混入，在 Vue 源码实习中也被大量使用：

```js
function Vue() {}

Vue.prototype._init = function() {/* ... */}
Vue.prototype.$set = function() {/* ... */}
Vue.prototype.$delete = function() {/* ... */}
// ...
```

那么这个新的装饰器是不是根本就像是为了混入设计模式量身定做的特性呢：

```js
@mixin({ _init: init, $set: set, $delete, del })
class Vue {/* ... */}
```

下面我们看看自定义的代码示例：

- `src/class/mixin.js`

```js
import { log } from '../utils'

function mixin(...methods) {
  return (target) => {
    Object.assign(target.prototype, ...methods)
  }
}

const humanActions = {
  greeting() {
    log(`this is ${this.name}`)
  },
}

const birdActions = {
  fly() {
    log('I can fly')
  },
}

@mixin(humanActions, birdActions)
class A {}

log('A:           ', A)
log('A.prototype: ', A.prototype)
```

这里我们定义一个 `mixin` 方法，接受多个方法集合对象做参数，并使用 `Object.assign` 方法注入 `target.prototype` 原型对象中，借此就可以向现存类型注入新的可用方法

```
A:            [Function: A]
A.prototype:  A { greeting: [Function: greeting], fly: [Function: fly] }
```

### 2.2 类方法装饰器 Class Method Decorators

第二种应用场景是类属性装饰器，我们以 **类方法装饰器(Class Method Decorators)** 为代表

#### 2.2.1 传入参数 & 基础用法

我们知道其实在 `class` 关键字后定义的方法、属性、访问器属性其实都是作为某个对象的属性(原型方法、实例属性、实例访问器属性)，也就是说其实他们接受的参数类型是非常相似的：

- `src/method/mixin.js`

```js
import { log } from '../utils'

function test(...args) {
  args.forEach((arg, i) => log(i, arg))
}

class A {
  @test
  field = 0

  @test
  f() {}

  @test
  get name() {}
}
```

```
0 A {}
1 field
2 {
  configurable: true,
  enumerable: true,
  writable: true,
  initializer: [Function: initializer]
}
0 A {}
1 f
2 {
  value: [Function: f],
  writable: true,
  enumerable: false,
  configurable: true
}
0 A {}
1 name
2 {
  get: [Function: get],
  set: undefined,
  enumerable: false,
  configurable: true
}
```

我们可以看到针对三种目标都接受三个参数：

1. 类定义对象
2. 方法/实例属性/实例访问器属性名
3. 属性描述符(description)

这里的属性描述符其实就跟 `Object.defineProperty` 的第三个参数相似，就是一些描述对象属性的标志(`configurable` 可配置性、`enumerable` 可遍历性、`writable` 可写性)，而不同对象有不同的默认值可以看到上面的输出

也就是说精确的函数标签如下

```js
class B {
  @test2
  f() {}
}

function test2(target, name, desc) {
  log('target: ', target)
  log('name:   ', name)
  log('desc:   ', desc)
}
```

```
target:  B {}
name:    f
desc:    {
 value: [Function: f],
 writable: true,
 enumerable: false,
 configurable: true
}
```

这时候其实留下了一个扩展点：我们可不可以透过定义属性的 getter/setter 来对属性进行访问的扩展，而这也就为后续的装饰器应用留下极大的扩展空间(当然更完整的方法扩展还是推荐使用 Proxy 代理对象)

#### 2.2.2 readonly 只读属性

第一种应用我们可以定义用于类方法的只读装饰器实现

- `src/method/readonly.js`

```js
import { log } from '../utils'

function readonly(target, name, desc) {
  desc.writable = false
  desc.enumerable = true
}

class A {
  @readonly
  f() {}
}

log('A.prototype', A.prototype)

try {
  A.prototype.f = 'new one'
} catch (e) {
  log(e)
}
```

```
A.prototype A { f: [Function: f] }
TypeError: Cannot assign to read only property 'f' of object '#<A>'
```

我们可以看到当我们尝试重新对 `@readonly f() {}` 重新赋值的时候就会因为 `writable: false` 而报错

#### 2.2.3 logger 日志装饰器

第二种是定义一个 `logger` 作为日志装饰器，记录一个对象指定方法的各个调用记录

- `src/method/logger.js`

```js
import { log } from '../utils'

function logger(target, name, desc) {
  const fn = desc.value

  desc.value = function (...args) {
    log(`[logger] invoke ${target.constructor.name}#${name}`)
    return fn.apply(this, ...args)
  }
  return desc
}

class Counter {
  count = 0

  @logger
  increment() {
    this.count++
    this.show()
  }

  @logger
  reset() {
    this.count = 0
    this.show()
  }

  show() {
    log(`count = ${this.count}`)
  }
}

log('Counter: ', Counter)

const counter = new Counter()
log('counter: ', counter)
counter.increment()
counter.increment()
counter.increment()
counter.reset()
counter.increment()
```

我们透过重新定义一个 `desc.value` 方法，相当于是进行一层方法的代理，并在每次调用方法的时候记录(输出)操作

#### 2.2.4 autobind 绑定实例

前面我们提过，事实上实例方法就是作为一个属性的 `value` 存在，也就是说我们可以把这个方法的获取改为 `get` 访问器属性进而实现关于调用对象或是其他的提前绑定

本节来实现一个绑定实例的装饰器

- `src/method/autobind.js`

```js
import { log, group } from '../utils'

function bindSelf(target, name, { value: fn, configurable, enumerable }) {
  const { constructor } = target

  return {
    configurable,
    enumerable,
    get() {
      group('in getter', () => {
        log(`target === A.prototype: ${target === A.prototype}`)
        log(`this === a:             ${this === a}`)
      })
      const boundFn = fn.bind(this)
      return boundFn
    },
    set() {},
  }
}

let id = 0

class A {
  id = id++

  @bindSelf
  getInstance() {
    return this
  }
}

const a = new A()
const a2 = new A()
const getInstance = a.getInstance
log(getInstance())
const getInstance2 = a2.getInstance
log(getInstance2())
```

其中最核心的就是将装饰目标方法改为 getter 并绑定访问对象 `const boundFn = fn.bind(this)`，这样就使得我们在根据 `a.getInstance` 提取方法的时候返回的就是一个与实例绑定的方法 `getInstance`

```
in getter
  target === A.prototype: true
  this === a:             true
A { id: 0 }
in getter
  target === A.prototype: true
  this === a:             false
A { id: 1 }
```

如此一来我们就可以看到 getter 方法内部的 this 就会绑定最后访问该方法的那个实例对象

### 2.3 补充：类实例属性、访问器属性装饰器

最后我们补充一下对于了实例属性、访问器属性的修饰

- `src/field/basic.js`

```js
import { log, group } from '../utils'

const test =
  (tag) =>
  (...args) => {
    group(tag, () => {
      args.forEach((arg, i) => log(i, arg))
    })
  }

class A {
  @test('field')
  num = 0

  @test('accessor')
  get show() {
    log(`num = ${this.num}`)
  }
}
```

```
field
  0 A {}
  1 num
  2 {
    configurable: true,
    enumerable: true,
    writable: true,
    initializer: [Function: initializer]
  }
accessor
  0 A {}
  1 show
  2 {
    get: [Function: get],
    set: undefined,
    enumerable: false,
    configurable: true
  }
```

我们可以看到与类原型方法大同小异，主要就是属性描述符的差别

## 3. 特性总结

最后我们根据装饰对象的不同记录以下装饰器方法的参数和默认属性：

| 装饰目标             | 参数列表                      | 描述符属性                                                                               |
| -------------------- | ----------------------------- | ---------------------------------------------------------------------------------------- |
| 类定义(class)        | `(target)`                    | $\times$                                                                                 |
| 实例属性(field)      | `(target, name, description)` | configurable: true<br/>enumerable: true<br/>writable: true<br/>initializer: \[Function\] |
| 原型方法(method)     | `(target, name, description)` | configurable: true<br/>enumerable: false<br/>writable: true<br/>value: \[Function\]      |
| 访问器属性(accessor) | `(target, name, description)` | configurable: true<br/>enumerable: true<br/>get: \[Function\] <br/>set: \[Function\]     |

## 补充：使用环境配置注意事项 & 三方库应用

最后再提一点，由于前面提过的装饰器实际上还处于提案阶段(目前到 stage-2 了)，实际上并不属于任何版本的现行标准，通常要真正使用的话必须配合 babel 的插件 `@babel/plugin-proposal-decorators` 使用，如下(使用 `@babel/register` 运行时启用)：

- `register.js`

```js
require('@babel/register')({
  presets: ['@babel/env'],
  plugins: [
    [
      '@babel/plugin-proposal-decorators',
      {
        legacy: true
      }
    ]
  ]
})

module.exports = require('./index.js')
```

### MobX

其他还有像是 MobX 在版本 6 之前有大量对于装饰器语法的实现和应用

### core-decorators

与 `core-js` 相似，`core-decorators` 提供了许多基础常见的装饰器实现，开箱即用

# 结语

ES7 的 **装饰器(Decorator)** 特性是一个非常有趣的特性，算是一种属于语言语法层面的特性，但是却又是与其他语言(如 Java)的原生特性非常之相似，甚至具备更强大的能力，合理的使用和发挥想象力的应用模式可以创造出兼具创意和可读性、可用性高的代码应用，供大家参考。 

# 其他资源

## 参考连接

| Title                              | Link                                                                                         |
| ---------------------------------- | -------------------------------------------------------------------------------------------- |
| ES6 修饰器                         | [http://caibaojian.com/es6/decorator.html](http://caibaojian.com/es6/decorator.html)         |
| tc39/proposal-decorators - github  | [https://github.com/tc39/proposal-decorators](https://github.com/tc39/proposal-decorators)   |
| jayphelps/core-decorators - github | [https://github.com/jayphelps/core-decorators](https://github.com/jayphelps/core-decorators) |
| MobX-Enabling decorators           | [https://mobx.js.org/enabling-decorators.html](https://mobx.js.org/enabling-decorators.html) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/es6/es7_decorator](https://github.com/superfreeeee/Blog-code/tree/main/front_end/es6/es7_decorator)
