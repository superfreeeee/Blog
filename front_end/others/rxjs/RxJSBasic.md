# RxJS: lodash for async、流式任务模型、函数式与响应式的结合

@[TOC](文章目录)

<!-- TOC -->

- [RxJS: lodash for async、流式任务模型、函数式与响应式的结合](#rxjs-lodash-for-async流式任务模型函数式与响应式的结合)
- [完整代码示例](#完整代码示例)
- [什么是 RxJS？](#什么是-rxjs)
- [核心概念 1: Observable](#核心概念-1-observable)
  - [创建 Observable](#创建-observable)
- [核心概念 2: Observer](#核心概念-2-observer)
  - [Subscription](#subscription)
- [核心概念 3: Operator](#核心概念-3-operator)
  - [High Order Observable](#high-order-observable)
  - [Marble Diagrams](#marble-diagrams)
- [核心概念 4: Subject](#核心概念-4-subject)
- [下集待续](#下集待续)
- [参考链接](#参考链接)

<!-- /TOC -->

# 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/others/rxjs/rxjs_basic](https://github.com/superfreeeee/Blog-code/tree/main/front_end/others/rxjs/rxjs_basic)

# 什么是 RxJS？

老实说这个命题太大了，RxJS 真的包含了太多的思想了，因此我更愿意从实践的角度来看他到底是什么，他到底能够帮我们完成什么。在我们真正能够理解他是如何运行的时候我们再回过来看他的背后到底是承载着怎么样的设计框架。

1. 流式任务模型

第一个我会说 RxJS 是一个流式的任务模型，类似于我们常说的函数式编程，但是 RxJS 所构造的流不仅仅是数据在一个纯函数的流之中进行计算并传递，而是以一个任务的结构进行传递。后面我们会看到 RxJS 中最重要、最核心的 Observable 对象与 function 之间的差异

2. Pull Model

第二重要的我觉得是所谓的拉模型，由于 RxJS 的书写形式参照了所谓的 ReactiveX 的 Observable 模型，因此看起来会非常类似我们常用的 callback、listener 这种的，但是多数情况这两个其实属于所谓的 Push Model，往往很容易产生异步调用的时序问题。

RxJS 就能够很好的帮助我们将所有的不管是命令式的函数调用、异步的 callback、listener 接收到的 Event，都化作任务流之中的 Observable，我们便能够很好的管理任务与任务之间的时序和取舍，无关乎他是异步还是同步，这也是这套模型强大的地方。

3. 响应式数据结构

在这样的流任务模型(暂时我们先这么称呼)之上，如果我们把起始的 Observable 用于保存状态，并且在此之上构建出多个基于状态的流任务，如此一来其实就能够构成所谓的响应式对象。

大家可能看过很多关于 Vue 的响应式实现，但是其实真正的响应式并不局限于 Proxy、对象上的观察者模式。响应式的本质应该是：我基于一个源头(不论是数据还是事件)，数据的变化或是事件的传递过程中，我能够捕捉并对该变化进行响应，这才是所谓的响应式。

好了经过这么多讨论，不亲自使用是没办法搞清楚到底在搞什么东西的哈哈

下面我们针对 RxJS 中几个比较重要的概念进行说明以及相关的代码示例

# 核心概念 1: Observable

- 代码：`/src/1.basic_observable.ts`

第一个是 Observable。在谈 Observable 之前，我们先来看看平常我们是怎么用函数的

> 普通函数

```ts
function f1() {
  return 1;
}

console.log(f1());
```

我们会调用它，然后拿到结果

那如果我们想要返回多个值呢？

> 函数返回多个值

```ts
function f2() {
  return [1, 2, 3];
}
f2().forEach(val => {
  console.log(val);
})
```

注意，我们这里实际上还是只返回了一个数据，只是我们赋予了这个数据多个值的意义

我们想要讨论的是，如果我们想要让一个函数按时间照顺序返回多个值呢，有点像是 ES6 的 Generator 函数一样

> Generator 函数

```ts
function* helloWorldGenerator() {
  yield 'hello';
  yield 'world';
  return 'ending';
}
const hw = helloWorldGenerator();
hw.next(); // { value: 'hello', done: false }
hw.next(); // { value: 'world', done: false }
hw.next(); // { value: 'ending', done: true }
```

而 RxJS 更进一步，定义了一个所谓的 Observable 对象，你可以想象成一个能够自动执行的 Generator 函数

> 一个普通的 Observable 对象的使用

```ts
const observable = new Observable((subscriber) => {
  console.log('observable start');
  subscriber.next(4);
  subscriber.next(5);
  subscriber.next(6);

  console.log('observable complete');
  subscriber.complete();

  console.log('observable after complete');
  subscriber.next(7);
});

observable.subscribe((val) => {
  console.log(val);
});
```

我们使用一个 Observable 对象的时候，调用他的 `subscribe` 方法有种调用函数的感觉，也可以形容成触发一次任务，他将会调用 Observable 构造时传入的函数，`next` 方法则是在任务的流程中逐次返回值，返回的形式也不是简单的 return，而是作为参数去调用我们 `subscribe` 的时候传入的函数。

`complete` 方法象征着这个 Observable 对象任务的终结，虽然后续的代码依旧会被执行，但是不会再去调用 subcribe 的回调函数，也就是对于调用者来说任务已经完成

上面的代码输出为

```
observable start
4
5
6
observable complete
observable after complete
```

Observable 为我们构建了一个特殊的任务模型，透过使用函数承接返回值的方式，我们就能够在一个任务的过程中多次对任务的返回值进行响应

下面我们来看看，任务的过程存在异步的情况

> 异步 Observable

```ts
const observable2 = new Observable((subscriber) => {
  setTimeout(() => {
    subscriber.next(8);
  });
});

observable2.subscribe((val) => {
  console.log(val);
});

const observable3 = new Observable((subscriber) => {
  Promise.resolve(9).then((res) => {
    subscriber.next(res);
  });
});

observable3.subscribe((val) => {
  console.log(val);
});
```

输出

```
9
8
```

我们可以看到，他其实就有点像是 Promise 的感觉，但是他比 Promise 强大的地方在于他可以返回多个值，而 Promise 只能返回一个 then

## 创建 Observable

为了便于下面例子的方便，我们先来看看 Observable 更快捷的创建方式，每次都 new 一下真的还蛮臃肿的是吧

- 代码：`/src/2.creation_observer.ts`

如果我们想要直接基于 next 多个值构建一个 Observable，我们可以这样写

```ts
of(0, 1, 2).subscribe(console.log);
```

针对 Promise 对象也提供了类似的形式

```ts
from(Promise.resolve).subscribe(console.log)
```

RxJS 会自动注册到 then 函数，并将结果使用 `next` 方法返回

如此一来就能够达成多种 input 形式，全部转化成 Observable 对象，也就是统一的任务模型之上，方便之后的统一操作

这一类的函数称为 `Creation Operator`，专门用于快捷创建 Observable 对象的

# 核心概念 2: Observer

- 代码：`/src/2.creation_observer.ts`

有了 Observable 的任务概念之后，我们接下来先来看看，到底是什么在承接 Observable 所返回的值呢？

> 简化的 Observer

前面我们都是这么写的

```ts
of(1, 2, 3).subscribe((val) => {
  console.log(val);
});
```

我们可以看到 `subscribe` 的方法的参数正是我们处理返回值的地方。还记得我们前面提过 Observable 对象内部会不断的 `next` 来返回值，使用 `complete` 来终结任务，所以其实在 `subscribe` 的时候传入的第一个函数就是一个 `next` 方法，实际上 RxJS 会将其转换成一个 Observer 对象

> 完整的 Observer

```ts
of(1, 2, 3).subscribe({
  next: (val) => {
    console.log(val);
  },
});
```

如此一来，`complete` 跟 `error` 也就很好理解了

```ts
of(4, 5, 6).subscribe({
  next: (val) => {
    console.log(val);
  },
  complete: () => {
    console.log('complete');
  },
  error: (err) => {
    // error handler
  },
});
```

## Subscription

- 代码：`/src/2.creation_observer.ts`

对于调用方来说，每次传入一个 Observer 对象(包含 `next`、`complete`、`error` 三个处理函数)。

注意！我们前面提过这里的 subscribe 方法不同于普通的 listener 函数，不是被动的等数据过来，而是主动的在 subscribe 的时候启动任务流，这才是所谓的 Pull Model。

同时在此基础之上，我们又可以像普通的观察/响应模型一样，去控制我们是否关心任务流的结果，这时候就会提到每次调用 `subscribe` 方法就会启动一个任务流，并返回一个 Subscription 对象，用于控制该任务是否应该完结

```ts
const subscription = of(7, 8, 9).subscribe(console.log);
subscription.unsubscribe();
```

输出

```ts
7
8
9
```

诶我们不是调用 `unsubscribe` 方法结束任务了吗，为什么还会输出？注意我们前面提过 Observable 就好像调用函数一样，调用 `subscribe` 方法的同时其实就是执行一个函数，所有同步能够返回的值都会立刻执行并返回，这才符合主动调用拿值的 Pull 模型

因此我们再举一个例子，让任务稍等我们一下

> 异步 Observable

```ts
const subscription2 = from(
  new Promise((resolve) => {
    setTimeout(() => {
      resolve(10);
    }, 1000);
  })
).subscribe(console.log);
subscription2.unsubscribe();
```

第二个例子就不会输出任何东西了，因为里面的人物被我们放到所谓的 timeout 队列上，在任务完成并响应之前我们就已经取消关注了，因此这次就不会输出任何东西。

# 核心概念 3: Operator

- 代码：`/src/3.operator.ts`

有了 Observable 的任务模型之后，接下来才是我们的重头戏：Operator，也就是所谓的流模型真正的应用。

我们可以利用 `pipe` 方法，透过定义 Observable 的转换方式，将一个 Observable 返回的数据放入一个处理的流当中，这个流的具体形式就是将一个个 Observable 进行某种转换之后继续传递下去，最后在我们 `subscribe` 的时候重新将数据吐出来

> 第一个 operator：映射

```ts
of(1, 2, 3)
  .pipe(map((x) => x * 10))
  .subscribe(console.log);
```

```
10
20
30
```

我们可以看到，原来 `of(1, 2, 3)` 会依次返回 `1, 2, 3`，我们透过 pipe + map 方法的组合，将内部值映射为十倍大小，最后打印的就是 `10, 20, 30`

> filter & take

接下来我们介绍过滤与终结的 operator

```ts
interval(100)
  .pipe(
    filter((x) => x > 3),
    take(5)
  )
  .subscribe(console.log);
```

interval 会构造一个固定时间不断 `next` 下一个值的计时器，`filter` 会将不符合条件的值拦截起来不继续向下 `next`，而 `take` 则可以指定我们接收到第几个 `next` 之后就将任务 `complete`，因此上面代码最后的输出应该是

```
4
5
6
7
8
```

## High Order Observable

接下来我们来介绍所谓的高阶 Observable。

我们都知道函数式编程中我们很常使用所谓的高阶函数，例如

```ts
const createFunc = () => () => {};
```

使用科里化的形式，返回一个新的函数的函数就是所谓的高阶函数

同样的 Observable 里面通常是 `next` 一个普通的值，我们也可以向下 `next` 一个新的 Observable 对象

> 高阶 Observable

```ts
of(1, 2, 3)
  .pipe(map((v) => interval(100)))
  .subscribe(console.log);
```

```
Observable { _subscribe: [Function (anonymous)] }
Observable { _subscribe: [Function (anonymous)] }
Observable { _subscribe: [Function (anonymous)] }
```

我们可以看到，如此一来我们的 `subscribe` 接收到的返回值就是一个 Observable 对象，第一种情况我们可以直接消费他

> 直接消费高阶 Observable 返回的 Observable

```ts
of(1, 2, 3)
  .pipe(
    map((v) =>
      interval(100).pipe(
        map((s) => `${v}-${s}`),
        take(3)
      )
    )
  )
  .subscribe((o) => {
    o.subscribe(console.log);
  });
```

```
1-0
2-0
3-0
1-1
2-1
3-1
1-2
2-2
3-2
```

这样其实蛮蠢的哈哈，当然 RxJS 提供了非常多的 operator，帮助我们对高阶 Observable 进行处理

> Join Operator 合并操作

```ts
of(1, 2, 3)
  .pipe(
    map((v) =>
      interval(100).pipe(
        map((s) => `${v}-${s}`),
        take(3)
      )
    ),
    concatAll()
  )
  .subscribe(console.log);
```

第一种是 `concatAll`，他会将接收到的多个 Observable 串联并形成一个新的 Observable，这样我们在外部就可以像调用一个简单的 Observable 直接 `subscribe` 拿到最终值就可以了

输出

```
1-0
1-1
1-2
2-0
2-1
2-2
3-0
3-1
3-2
```

如果我们将 `concatAll` 改成 `mergeAll`，那么他就不会串联，而是无脑的合并成一个 Observable 而不管顺序

> mergeAll

```
1-0
2-0
3-0
1-1
2-1
3-1
1-2
2-2
3-2
```

最终输出就是按时间顺序谁先 `next` 谁就先输出

> switchAll

第三种则是 `switchAll`，与 `concatAll`、`mergeAll` 会保留所有 Observable 并进行排列不同，`switchAll` 在看到下一个 Observable 的时候会立刻 copmlete 前一个 Observable 只留下后面的任务

因此输出就是

```
3-0
3-1
3-2
```

## Marble Diagrams

RxJS 为了帮助开发者们更好的理解 Observable 的流模型，特地设计出了一套所谓的 Marble Diagrams，用于描述 Observable 的状态变化。这里就不再贴出来了，官方的解释我觉得已经非常足够，同时搭配不同的 operator 不断的去实验并理解 RxJS 的模型之下数据的流动。

# 核心概念 4: Subject

我们介绍完流任务模型的核心 Observable 对象，用于处理任务返回值的 Observer，以及构建流管道的重要操作函数 operator 函数。实际上一个完整的流式任务模型就已经构建完整了

而在我的理解当中，其实 RxJS 的最终目的并不是用来做一个简单的响应式对象而已，他比这个强大太多了，它构建了一套自己的任务模型。

最后我们就来介绍一个能作为任务起点，也可以说是一个自由的任务触发器的 Subject 对象

- 代码：`/src/4.subject.ts`

```ts
const subject = new Subject();

subject.subscribe(console.log);

subject.next(1);
subject.next(2);
subject.next(3);
```

简单的例子如上，一个 Subject 对象其实也是一个 Observable 对象，因此我们能够基于 subject 构建一条自己的链。他与普通的 Observable 最大的不同在于，我还可以直接调用 `next` 方法，从外部来触发一个任务的返回值，也就是说我们能够将 Observable 定义任务流程的能力放到外面来

本来是

```ts
new Observable((subscriber) => {
  subscriber.next(1)
  subscriber.next(2)
  subscriber.next(3)
}).subscribe(console.log)
```

现在我们可以

```ts
const subject = new Subject();

subject.subscribe(console.log);

subject.next(1);
subject.next(2);
subject.next(3);
```

我们可以看到 subject 相当于是将原来的 `subscriber` 提出来到外面。

RxJS 提供了多种 Subject 的变形，其中一种是 `BehaviorSubject`，他其实就是会在初始化的时候基于默认值调用第一次的 `next`，因此就变成好像是一个响应式对象一样，保留了一个状态在哪里。如下

```ts
const subject = new BehaviorSubject(0);

subject.subscribe(console.log);

subject.next(1);
subject.next(2);
subject.next(3);

subject.subscribe(console.log);
```

```
0
1
2
3
3
```

# 下集待续

我认为 RxJS 提出了一个非常棒的模型，同时他也是集函数式编程与响应式编程的特色于一身，基于一个类似流任务模型的 Observable 来代替简单的函数或是 Promise 只能返回一个值的窘境。

我认为 RxJS 还是非常复杂的，要一篇说完有点难，建议大家看完之后还是要自己写些代码去试验，能够更好的理解每个 operator 的作用，并且去思考如何组织自己的任务流模型。尤其是牵扯到异步模型的时候，就非常容易在时序上搞混。

之后作者会陆续分享一些 RxJS 的实践结果，以及关于源码的阅读分析。

# 参考链接

| Title                             | Link                                                                                                         |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| RxJS                              | [https://rxjs.dev/](https://rxjs.dev/)                                                                       |
| redux-observable/redux-observable | [https://github.com/redux-observable/redux-observable](https://github.com/redux-observable/redux-observable) |
| Generator - ES6                   | [http://caibaojian.com/es6/generator.html](http://caibaojian.com/es6/generator.html)                         |
