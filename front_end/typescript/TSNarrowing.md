# TS 进阶: Narrowing 类型缩紧 / Guards 类型守卫

@[TOC](文章目录)

<!-- TOC -->

- [TS 进阶: Narrowing 类型缩紧 / Guards 类型守卫](#ts-进阶-narrowing-类型缩紧--guards-类型守卫)
- [完整代码示例](#完整代码示例)
- [为何需要 Narrowing/Guards？](#为何需要-narrowingguards)
- [常见手段](#常见手段)
  - [typeof](#typeof)
  - [Truthiness 真/假值判断(短路判断) `&&`、`||`](#truthiness-真假值判断短路判断-)
  - [Equality 相等判断 `==`、`===`](#equality-相等判断-)
  - [in](#in)
  - [instanceof](#instanceof)
  - [Assignments 赋值表达式 `variable = value`](#assignments-赋值表达式-variable--value)
  - [Control flow 控制流 `if`、`else`、`while`](#control-flow-控制流-ifelsewhile)
  - [Type Predicates 类型预测 `param is Type`](#type-predicates-类型预测-param-is-type)
- [参考连接](#参考连接)

<!-- /TOC -->

# 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/typescript/ts_narrowing](https://github.com/superfreeeee/Blog-code/tree/main/front_end/typescript/ts_narrowing)

# 为何需要 Narrowing/Guards？

使用 TS 进行带类型的编程中，我们会很常遇到联合类型的场景，可能是作为处理函数的参数需要兼容多个类型，又或是外部数据源针对同一个字段给出不同类型如下

```ts
function processRes(res: number | string) {}
```

```ts
interface IXxxApiRes {
  // ...
  id: number | string,
}
```

那这时候在处理函数内部可能会遇到没办法正常访问类型方法的问题

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_narrowing_1_error.png)

这时候我们就需要使用所谓的 narrowing，将变量的类型缩紧到适合的范围，才能在类型安全的状况下确保能正确访问字段或是方法

有些人可能对 Type Guards 类型守卫的名词更熟悉，本质上都是指同样的作用的

# 常见手段

下面将列出在 TS 编程中会遇到的常见的 narrowing 方法和场景

## typeof

第一个是使用 JS 的 typeof 方法

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_narrowing_2_typeof.png)

如上图所示，TS 能够正确识别 typeof 表达式与比较的字符串，然后在条件代码块内部给出正确的类型。

## Truthiness 真/假值判断(短路判断) `&&`、`||`

第二个场景是真假值判断，TS 会帮我们过滤掉如 null 与 undefined 的类型

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_narrowing_3_truthiness.png)

接上一个例子，使用 `typeof === 'object'` 会过滤出数组和 null 两个类型，在加上一个 `strs &&` 就可以确定 strs 是 `string[]` 类型，能正确使用数组方法了

## Equality 相等判断 `==`、`===`

第三种是使用相等判断。在 JS 内分成强相等(`===`)于弱相等(`==`)，这两种相关的类型转换 TS 都能够正确识别

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_narrowing_4_equality_1.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_narrowing_5_equality_2.png)

## in

下面一种是使用 `in` 关键字。我们可以利用类型间的差异来作为类型守卫

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_narrowing_6_in.png)

如上图，我们利用 swim 字段是否存在的检查，来判断是否为 Fish 类型。如此一来就能够在不需要完全知道 Fish 类型细节的情况下与 Bird 作区分

## instanceof

instanceof 的判别方式是使用了原型链，相较于 in 或是 typeof 就更加严格了

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_narrowing_7_instanceof.png)

## Assignments 赋值表达式 `variable = value`

接下来这个比较有趣的是，可能很多已经在使用 TS 的人也没有注意到，即便变量类型被显式的定义了，但是赋值表达式的后面 TS 也是能够确定指定类型的

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_narrowing_8_assignment.png)

如上图，虽然 s 类型定义有两个可选类型，但是两个赋值表达式之后的语句其实 TS 是能够确定该变量类型的

## Control flow 控制流 `if`、`else`、`while`

承接赋值表达式的思想，其实在一些 if/else、while 等的控制流代码块之后，如果存在修改变量的表达式，该变量的类型也是会改变的。

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_narrowing_9_control_flow.png)

我们可以看到一开始 `n=123` 会将 n 转为 number 类型，然后又因为 if/else 语句内部将 n 赋与了 string/boolean 两种类型，所以函数最后返回的类型实际上是 `string | false` 类型而不是 `string | number | boolean`。

## Type Predicates 类型预测 `param is Type`

最后一种则是我们希望自定义类似 typeof 或是 instanceof 关键字，并且使用更语义化的形式来表达这种类型守卫，这时候我们就要用到 Type Predicates 类型预测表达式

相信写过 TS 的人可能都写过这种代码

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_narrowing_10_predicates_wrong.png)

我们写了一个关于类型判断的函数，返回了 boolean 类型，但是 TS 却没办法将简单的 boolean 返回值作为类型守卫，因此变量在内部还是没办法确定类型

类型守卫的写法如下：在返回类型的地方使用 `prop is Type` 的语句来表达

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_narrowing_11_predicates_right.png)

如上图，这样一来，我们在判断表达式的条件块内部就能够确定变量的类型了

# 参考连接

| Title                  | Link                                                                                                                           |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| Narrowing - TypeScript | [https://www.typescriptlang.org/docs/handbook/2/narrowing.html](https://www.typescriptlang.org/docs/handbook/2/narrowing.html) |
