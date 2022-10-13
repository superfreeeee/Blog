# TS 进阶: Utility Types 工具类型

@[TOC](文章目录)

<!-- TOC -->

- [TS 进阶: Utility Types 工具类型](#ts-进阶-utility-types-工具类型)
- [完整代码示例](#完整代码示例)
- [前言](#前言)
- [对象相关](#对象相关)
  - [Awaited\<T\>](#awaitedt)
  - [Partial\<T\>](#partialt)
  - [Required\<T\>](#requiredt)
  - [Readonly\<T\>](#readonlyt)
  - [Record\<Keys, Type\>](#recordkeys-type)
  - [Pick\<Type, Keys\>](#picktype-keys)
  - [Omit\<Type, Keys\>](#omittype-keys)
  - [Exclude\<UnionType, Members\>](#excludeuniontype-members)
  - [Extract\<UnionType, Union\>](#extractuniontype-union)
  - [NonNullable\<UnionType\>](#nonnullableuniontype)
- [函数相关](#函数相关)
  - [Parameters\<FunctionType\>](#parametersfunctiontype)
  - [ConstructorParameters\<Type\>](#constructorparameterstype)
  - [ReturnType\<FunctionType\>](#returntypefunctiontype)
  - [InstanceType\<FunctionType\>](#instancetypefunctiontype)
  - [ThisParameterType\<FunctionType\>](#thisparametertypefunctiontype)
  - [OmitThisParameter\<FunctionType\>](#omitthisparameterfunctiontype)
  - [ThisType\<FunctionType\>](#thistypefunctiontype)
- [字符串相关](#字符串相关)
- [结语](#结语)
- [参考连接](#参考连接)

<!-- /TOC -->

# 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/typescript/ts_utility_types](https://github.com/superfreeeee/Blog-code/tree/main/front_end/typescript/ts_utility_types)

# 前言

你现在在前端用上 Typescript 了吗？你以为用用 `number`, `string`, `() => void`, `interface` 就可以了吗。本篇要带各位读者来学习认识 TS 提供的众多内置工具类型，帮助我们进行类型的推导、分离、重组。当我们作为一个库的开发者时，这些类型的学习和运用都是不可或缺的。

# 对象相关

## Awaited\<T\>

简单来说就是，取出 Promise 返回的数据类型

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_utility_types_1_awaited.png)

## Partial\<T\>

将类型的所有字段改为可选

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_utility_types_2_partial.png)

## Required\<T\>

Partial 的反向，将所有字段改为必选

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_utility_types_3_required.png)

## Readonly\<T\>

为类型每个字段加上 readonly 标识符

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_utility_types_4_readonly.png)

## Record\<Keys, Type\>

对 Object 有更详细的类型描述和限制

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_utility_types_5_record.png)

## Pick\<Type, Keys\>

从类型中抽出指定字段，组合成新的类型(相当于过滤出部分字段)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_utility_types_6_pick.png)

## Omit\<Type, Keys\>

与 Pick 互补，Omit 是指定要删除的字段(可以想成一个是取交集，一个是取差集)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_utility_types_7_omit.png)

## Exclude\<UnionType, Members\>

Exclude 与 Omit 不同在于，抽取目标一个是对象类型中的字段集合，一个是简单的联合类型

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_utility_types_8_exclude.png)

## Extract\<UnionType, Union\>

Extract 则是与 Pick 类似，将操作目标改成联合类型即可

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_utility_types_9_extract.png)

## NonNullable\<UnionType\>

NonNullable 是删除联合类型中的空类型，也就是 `null` 跟 `undefined`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_utility_types_10_non_nullable.png)

# 函数相关

## Parameters\<FunctionType\>

Parameters 是从一个函数类型中抽取所需参数类型

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_utility_types_11_parameters.png)

## ConstructorParameters\<Type\>

ConstructorParameters 则是获取对象的构造函数类型，需要目标类型有 `new(params): T` 标签的字段

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_utility_types_12_contructor_parameters.png)

## ReturnType\<FunctionType\>

ReturnType 返回函数的返回类型

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_utility_types_13_return_type.png)

## InstanceType\<FunctionType\>

InstanceType 有点像是针对构造函数的 ReturnType，所谓的实例类型其实就是构造函数的返回类型

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_utility_types_14_instance_type.png)

## ThisParameterType\<FunctionType\>

ThisParameterType 能更精准的识别函数标签内的 this 参数类型，而这也是 ts 专有的特性

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_utility_types_15_this_parameter.png)

## OmitThisParameter\<FunctionType\>

OmitThisParameter 则是完全相反，返回去除 this 之后的完整函数类型标签

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_utility_types_16_omit_this_parameter.png)

## ThisType\<FunctionType\>

最后一个最特别的是 ThisType，他比较像是一种类型的补充

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_utility_types_17_this_type.png)

该特性需要开启 tsconfig 中的 `compilerOptions.noImplicitThis = true` 才能够使用

如上图我们为 methods 描述中加上 ThisType 的补充说明，就能够在后续使用的时候，在不明确写出 methods 函数内的 this 也能够正确识别 this 的类型

# 字符串相关

最后一个大类就是将字符串一些常见的方法加入到类型操作里面，针对需要精细控制字符串内容的部分比较有用(利用字符串操作构建导出名称作为字段名称等等)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ts_utility_types_18_string_manipulation.png)

# 结语

关于 TS 的一切都在编译后回归到原始的 JS，但是良好的类型检查能够确保我们在类型安全的掩护下进行编程。一方面帮助我们省略不必要的类型检查减少运行时开销，一方面也是保护我们不进行错误的类型引用而产生运行时异常。

上面提到的功能虽然基础，但是在实际生产环境中还是有非常多项目没有很好的实践 TS 推荐的类型规范，持续学习，目的为构建更安全也更舒适的类型编程体验。

# 参考连接

| Title                      | Link                                                                                                                               |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| Utility Types - TypeScript | [https://www.typescriptlang.org/docs/handbook/utility-types.html](https://www.typescriptlang.org/docs/handbook/utility-types.html) |
