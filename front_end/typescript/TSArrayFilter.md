# TS 开发经验分享: 使用 Array.prototype.filter 遇到的问题与思考

@[TOC](文章目录)

<!-- TOC -->

- [TS 开发经验分享: 使用 Array.prototype.filter 遇到的问题与思考](#ts-开发经验分享-使用-arrayprototypefilter-遇到的问题与思考)
- [完整代码示例](#完整代码示例)
- [问题场景：Array.prototype.filter 类型不对](#问题场景arrayprototypefilter-类型不对)
- [一些尝试](#一些尝试)
  - [尝试 Array.prototype.reduce](#尝试-arrayprototypereduce)
  - [Array.prototype.filter 类型定义](#arrayprototypefilter-类型定义)
- [推荐写法](#推荐写法)
  - [使用 predicate 类型预测](#使用-predicate-类型预测)
  - [风险：predicate 属于类型断言的一种](#风险predicate-属于类型断言的一种)
  - [目前最推荐写法](#目前最推荐写法)
- [一些思考](#一些思考)
- [参考链接](#参考链接)

<!-- /TOC -->

# 完整代码示例

[https://github.com/superfreeeee/Blog-code/blob/main/front_end/typescript/ts_array_filter/array-filter.ts](https://github.com/superfreeeee/Blog-code/blob/main/front_end/typescript/ts_array_filter/array-filter.ts)

# 问题场景：Array.prototype.filter 类型不对

今天来分享一个使用 TS 遇到的小问题

最近的业务场景常常用到 `Array.prototype.filter` 的场景，因此遇到了以下代码

```ts
const arr = [123, '456', true, 789, new Date()];
// const arr: (string | number | boolean | Date)[]

// 0. 使用 Array.prototype.filter => 类型错误
const arr2 = arr.filter((item) => typeof item === 'number');
// const arr: (string | number | boolean | Date)[]
```

但是这时候我们会发现 arr2 的类型是不正确的，原因是与 filter 的类型定义有关的，这个我们后面会提到

# 一些尝试

这时候高度依赖 TS 的我就想把这个类型给他订好，我第一个想到的就是我们都知道函数式写法的老祖宗就是 `reduce` 函数，那我用 reduce 总行了吧

## 尝试 Array.prototype.reduce

```ts
// 1. 使用 Array.prototype.reduce + 类型断言保证类型
const arr3 = arr.reduce((res, item) => {
  if (typeof item === 'number') {
    res.push(item);
  } else if (typeof item === 'string') {
    // res.push(item);
    // Argument of type 'string' is not assignable
    //   to parameter of type 'number'.ts(2345)
  }
  return res;
}, [] as number[]);
// const arr3: number[]
```

我们提供 reduce 一个初始值并使用类型断言来确保传递对象的稳定，这样我们在函数内部就能够利用类型守卫正确的识别 item 的类型

但是这样写实在是太麻烦了，我们还是必须回头去看一看 filter 的类型定义

## Array.prototype.filter 类型定义

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_array_filter_1_filter_type_define.png)

TS 总共提供了两种重载，下面那个是大家最常用的，会直接返回原来的 T 数组类型。这下我们就只剩下上面那个重载能够使用，也就是需要将传入的过滤判断函数改成使用 predicate 类型预测的返回值才行

# 推荐写法

## 使用 predicate 类型预测

```ts
// 3. 使用 predicate 预测类型
const arr4 = arr.filter((item): item is number => typeof item === 'number');
// const arr4: number[]
```

其实我们只需要为我们的判断表达式加上 predicate 返回类型，我们就可以看到 arr4 的类型神奇的符合预期了

## 风险：predicate 属于类型断言的一种

然而这其中还是存在部分风险的：predicate 并无法识别内部函数逻辑是否正确

```ts
// 4. 风险：predicate 为断言表达式，无法识别内部逻辑
const arr5 = arr.filter((item): item is number => typeof item === 'string');
// const arr5: number[]
```

我们可以看到明明写了 `item is number` 但是里面实际上的逻辑却是 `typeof item === 'string'`

当然这种的 TS 也本来就没法识别，需要所有使用者自行去注意，并保证判断函数的类型正确，否则容易出现运行时 bug

## 目前最推荐写法

作者自己推荐将类型判断的函数独立出来，并在编写这类函数的时候小心注意每一个细节，才能保证运行时访问该类型的数据不会产生问题

```ts
// 5. 推荐写法：单独抽离类型判断
const isNumber = (item: any): item is number => typeof item === 'number';
const arr6 = arr.filter(isNumber);
// const arr6: number[]
```

# 一些思考

在上述思考过程当中其实我们就能够发现一个 TS 开发遇到的难题：当我们遇到类型错误的时候都是怎么解决的？

1. 有人会直接在表达式后面再加上类型断言

```ts
const arr7 = arr.filter((item) => typeof item === 'number') as number[];
// const arr7: number[]
```

这样也不是说错，但是当里面的过滤条件越来越复杂的时候就容易产生隐藏风险

2. 使用 reduce

有些人可能会采用上面提过的 reduce 方案，但是缺点就是写法上太麻烦了

3. 正解：查询所用方法的类型定义

我们使用 TS 的时候非常容易忘记去查阅使用数据的类型定义，我们或许很常依赖一些自动的类型推导、类型守卫，但是当我们遇到一些比较基础的 API 或是一些需要使用者主动来维护的类型判断(如 predicate 返回类型)，我们很容易就想去偷懒使用 any、? 的方式草草带过。

这样其实就是给自己在埋坑，迟早会在运行时产生问题的。

# 参考链接

| Title | Link |
| ----- | ---- |
|       | []() |
