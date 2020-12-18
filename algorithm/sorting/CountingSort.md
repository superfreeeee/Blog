# Sorting 排序算法: Counting Sort 计数排序

@[TOC](文章目录)

<!-- TOC -->

- [Sorting 排序算法: Counting Sort 计数排序](#sorting-排序算法-counting-sort-计数排序)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [算法介绍](#算法介绍)
    - [输入](#输入)
    - [核心思想](#核心思想)
    - [算法流程](#算法流程)
    - [算法复杂度分析](#算法复杂度分析)
- [结语](#结语)

<!-- /TOC -->

## 简介

今天来介绍第一个线性时间排序算法。由于`决策树模型`(详见参考链接)，比较排序算法的下界定在 **Ω(nlogn)**。但是如果我们知道或是对输入加上某些限制，便可以不使用`比较`的方式来进行排序，从而获得线性时间的排序算法。本篇介绍线性时间排序算法中的第一个：计数排序。

## 参考

<table>
  <tr>
    <td>基于比较排序的算法复杂度的下界</td>
    <td><a href="https://www.cnblogs.com/hyserendipity/p/10786592.html">https://www.cnblogs.com/hyserendipity/p/10786592.html</a></td>
  </tr>
  <tr>
    <td>决策树与排序算法的一般下界</td>
    <td><a href="https://blog.csdn.net/kdb_viewer/article/details/83010155">https://blog.csdn.net/kdb_viewer/article/details/83010155</a></td>
  </tr>
</table>

# 正文

## 算法介绍

### 输入

```java
/* 传入参数 */
int[] A // 原数组(可以是任何类型的数据，这边使用 int 类型为示例)
int k // 表示数组内容的区间为 [0, k]

/* 限制 */
int n // 数组长度
```

### 核心思想

计数排序顾名思义就是透过`计数(counting)`来排序，透过计算`小于等于当前数字的数的数量`来直接确定该数的最终位置，如下图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/counting_sort.png)

### 算法流程

接下来介绍算法的具体流程(如何计数并放到适当的位置)。

由于计数排序存在一个限制：数组中所有数都位于闭区间 [0, k] 之内，所以我们的计数方式就是透过建立一个大小为 k + 1 的数组记录各个数出现的次数，一次遍历之后便能知道小于等于当前数字的数量

- 伪代码（参考：算法导论）

```
Counting-Sort(A, k)
    n = A.length
    # 1. 建立大小为 k + 1 的数组记录不同数出现的次数
    let C be an array with size (k + 1), and set 0 
    for i = 0 to n - 1
        C[A[i]] = C[A[i]] + 1
    
    # 2. 每个数再加上前面所有数的数量，相当于每个位置记录了小于等于当前数字的数的数量
    for i = 1 to k
        C[i] = C[i - 1]
    
    # 3. 由后往前将数字填入正确的位置
    let B be an array with size n
    for i = n - 1 to 0
        # 若 C[x] = y 表示有 y 个小于等于 x 的数，于是下一个 x 要放到下标为 y - 1 的位置
        C[A[i]] = C[A[i]] - 1
        B[C[A[i]]] = A[i]
```

### 算法复杂度分析

先说结论：

1. 时间复杂度：**O(n + k)**
2. 空间复杂度：**O(k)** (额外空间复杂度，不考虑原数组的空间和保存结果的空间)
3. 非原址排序
4. 稳定的

- 时间复杂度：**O(n + k)**

在伪代码中第一步的循环复杂度为 O(n)、第二步为 O(k)、第三步为 O(n)，所以最终复杂度为 O(n) + O(k) + O(n) = O(n + k)；当数组值的区间 k = O(n) 时便有 O(n + k) = **O(n)** 为线性时间的复杂度

- 空间复杂度：**O(k)** (额外空间复杂度，不考虑原数组的空间和保存结果的空间)

算法过程仅仅是用了一个 k + 1 大小的数组来保存各个值出现的次数，空间代价为 **O(k)**

- 非原址排序

由于对于数组的计数和最后放入正确的位置都不需要也没有改变原数组的内容，属于非原址排序

- 稳定的

对于同样值的出现，由于最后填入结果时是由后往前，所以并不会改变同值的数的相对位置，结果是稳定的。

# 结语

由于在数据规模超过一定程度的时候，Ω(nlogn)的比较排序下界有时候是不被接受的，所以透过对输入数据作出限制(区间 [0, k])可以在不比较的情况下完成排序，同时也能够作为后续桶排序的基础排序。