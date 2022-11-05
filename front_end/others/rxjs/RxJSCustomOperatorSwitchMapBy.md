# RxJS 实做: 自定义 operator - switchMapBy

@[TOC](文章目录)

<!-- TOC -->

- [RxJS 实做: 自定义 operator - switchMapBy](#rxjs-实做-自定义-operator---switchmapby)
- [完整代码示例](#完整代码示例)
- [Operator 运算子](#operator-运算子)
- [Custom Operator 自定义运算子](#custom-operator-自定义运算子)
- [switchMapBy](#switchmapby)
- [参考链接](#参考链接)

<!-- /TOC -->

# 完整代码示例

[https://github.com/superfreeeee/Blog-code/blob/main/front_end/others/rxjs/rxjs_custom_operator/src/operators/switchMapBy.ts](https://github.com/superfreeeee/Blog-code/blob/main/front_end/others/rxjs/rxjs_custom_operator/src/operators/switchMapBy.ts)

# Operator 运算子

RxJS 中关于 operator 有分好几类

> Creation Operators

创建类的主要是方便我们更快捷的创建 Observable 对象

```ts
of(1, 2, 3).subscribe(/* ... */)
from(new Promise(/* ... */)).subscribe(/* ... */)
```

> Join Creation Operators

这种通常是以多个 Observable 为参数合成一个新的 Observable 返回

```ts
const firstTimer = timer(0, 1000); // emit 0, 1, 2... after every second, starting from now
const secondTimer = timer(500, 1000); // emit 0, 1, 2... after every second, starting 0,5s from now
combineLatest([firstTimer, secondTimer]).subscribe(([first, second]) => {/* ... */});
```

> Transformation, Filtering, Join Operators

这一类的主要做一些数据的转换、过滤、合并等操作，从一个 Observable 流生成新的变换后的 Observable 流，通常我们会使用 `pipe` 方法来实现函数式编程中 compose function 的效果

```ts
of(1, 2, 3, 4, 5, 6).pipe(
  map(x => 10 * x),
  take(4)
).subscribe(console.log); // 10, 20, 30, 40
```

> Error Handling, Utility, Conditional Operators

其他还有如错误处理、工具类、条件分支等多种运算子，这里就不展开了

# Custom Operator 自定义运算子

本篇我们就来自定义一个 Operator，通常我们定义的 Operator 比较算是过程式的 Operator，因此采用如下 Operator 的类型定义

```ts
export interface OperatorFunction<T, R> extends UnaryFunction<Observable<T>, Observable<R>> {}
```

接受一个 Observable 作为参数，最终返回一个新的 Observable 流

第一种最简单的我们可以直接使用 `pipe` 方法包装一些现有的 Operator，就能够简单构造出一个新的 Operator 了

```ts
function discardOddDoubleEven() {
  return pipe(
    filter((v) => !(v % 2)),
    map((v) => v + v)
  );
}
```

另一种我们也可以自己编写详细的函数过程

```ts
function customOperator<T, R>(/* params */) {
  return (source: Observable<T>) => new Observable<R>((subscriber) => {
    const subscription = source.subscribe({
      // next
      // error
      // complete
    });

    return () => {
      // finalization logic
      subscription.unsubscribe();
      // other ...
    }
  })
}
```

当我们编写一个自定义的 Operator 时，所有的 next、error、complete 以及最终的 unsubcribe 流程都需要定义清楚，才能满足所有通用场景下的流程都能够导出预期中的结果

# switchMapBy

今天我们要来做的是一个名为 `switchMapBy` 的自定义 Operator。我们都知道 `switchMap` 能够先进行 `map` 操作，然后当下一个 Observable 进来之后就会终止上一个流的进行；然而有时候我们希望每一个流都能够有一个类似 id 的标记，让我们只有在相同 id 的 Observable 才需要进行替换，否则就保留知道 complete 或是下一个同 id 的 Observable 进来为止。

这个 switchMapBy 我们将会组合使用 `mergeMap` 与 `takeUntil` 来实现

- `/src/operators/switchMapBy.ts`

首先我们先来看重载类型定义

```ts
type Key<T> = keyof T;
type KeyMap<T, R> = (input: T) => R;
type Equal<T> = (input: T, other: T) => boolean;
type KeyOrKeyMap<T, R = any> = Key<T> | KeyMap<T, R> | Equal<T>;

export function switchMapBy<T, R>(
  key: Key<T>,
  mapFn: (val: T) => Observable<R>,
): OperatorFunction<T, R>;

export function switchMapBy<T, R, K>(
  keyMap: KeyMap<T, K>,
  mapFn: (val: T) => Observable<R>,
): OperatorFunction<T, R>;

export function switchMapBy<T, R, K>(
  equal: Equal<T>,
  mapFn: (val: T) => Observable<R>,
): OperatorFunction<T, R>;
```

我们希望该 Operator 能够处理的 id 有三种：
1. Key：是直接指定 `keyof T` 的字段，通常是一个字符串
2. KeyMap：则是从输入值 input 映射到 id 的单参数函数，里面会自动将两个 Observable 值映射为 id 后进行比较
3. Equal：最后就是更底层的接受两个值然后直接返回是否相同的 boolean 判断结果

接下来我们需要对三种参数类型进行归一化，根据不同参数签名最终返回第三种最基础的比较函数形式

```ts
const createPredicate = <T, R>(
  keyOrKeyMap: KeyOrKeyMap<T, R>,
): ((inputVal: T, otherVal: T) => boolean) => {
  if (typeof keyOrKeyMap !== 'function') {
    return (x: T, y: T) => x[keyOrKeyMap] === y[keyOrKeyMap];
  }

  if (keyOrKeyMap.length === 1) {
    const keyMap = keyOrKeyMap as (inputVal: T) => R;
    return (x: T, y: T) => keyMap(x) === keyMap(y);
  }

  return keyOrKeyMap as (inputVal: T, otherVal: T) => boolean;
};
```

最终就能看到 `switchMapBy` 的具体实现

```ts
// T = input, K = keyMapRes, R = output
export function switchMapBy<T, R, K = any>(
  keyOrKeyMap: KeyOrKeyMap<T, K>,
  mapFn: (val: T) => Observable<R>,
): OperatorFunction<T, R> {
  const predicate = createPredicate(keyOrKeyMap);

  return (input$) => {
    return input$.pipe(
      mergeMap((inputVal) => {
        return mapFn(inputVal).pipe(
          takeUntil(
            input$.pipe(filter((otherVal) => predicate(inputVal, otherVal))),
          ),
        );
      }),
    );
  };
}
```

mergeMap 的使用有点类似于先基于一个 Observable 内的多个 value 使用 mapFn 映射为一个单独的 Observable 之后，再将这多个 Observable 进行 merge，在不限定顺序的情况下重新合成一个 Observable

在此基础上，我们对每个 mapFn 返回的 Observable 进行限制，使用 `takeUntil` + `filter` 的组合，指定如果出现 `predicate()` 结果为 true 的就结束这个 input$ 所对应的 Observable 流

使用 Marble Diagrams 表示的话就如下图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/rxjs_custom_operator_switchMapBy_marble.png)

具体使用范例如下

```ts
const subject = new Subject<IData>();

subject
  .pipe(
    switchMapBy('id', (obj) => of(obj).pipe(delay(obj.delay || 0))),
  )
  .subscribe((obj) => {
    const now = Math.floor(performance.now());
    console.log(`[time: ${now}]`, obj);
  });

subject.next({ id: 1, seq: 1 });
subject.next({ id: 2, seq: 2 });
subject.next({ id: 3, seq: 3 });
subject.next({ id: 1, seq: 4 });
subject.next({ id: 2, seq: 5, delay: 3000 });
subject.next({ id: 2, seq: 6, delay: 2000 });
```

输出

```
[time: 301] { id: 3, seq: 3 }
[time: 303] { id: 1, seq: 4 }
[time: 2302] { id: 2, seq: 6, delay: 2000 }
```

# 参考链接

| Title                                    | Link                                                                                                                                                                           |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| switchMap distincted by property in RxJs | [https://stackoverflow.com/questions/56917296/switchmap-distincted-by-property-in-rxjs](https://stackoverflow.com/questions/56917296/switchmap-distincted-by-property-in-rxjs) |
| switchMapBy - StackBlitz                 | [https://stackblitz.com/edit/rxjs-x1g4vc?file=index.ts](https://stackblitz.com/edit/rxjs-x1g4vc?file=index.ts)                                                                 |
| Operators - RxJS                         | [https://rxjs.dev/guide/operators](https://rxjs.dev/guide/operators)                                                                                                           |
