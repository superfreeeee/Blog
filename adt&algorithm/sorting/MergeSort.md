# Sorting 排序算法: Merge Sort 归并排序

@[TOC](文章目錄)

<!-- TOC -->

- [Sorting 排序算法: Merge Sort 归并排序](#sorting-排序算法-merge-sort-归并排序)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [算法思想原理](#算法思想原理)
    - [输入](#输入)
    - [算法思想](#算法思想)
    - [算法流程](#算法流程)
    - [算法复杂度分析](#算法复杂度分析)
  - [Java 实现](#java-实现)
- [結語](#結語)

<!-- /TOC -->

## 簡介

归并排序是一个经典的分治算法，分治算法的核心思想便是将原问题`分解(partition)`成多个拥有相同结构的子问题，分别求解之后再`合并(merge)`所有子问题的解。接下来就来看看归并排序是如何运作的吧。

## 參考

<table>
  <tr>
    <td>动图来源</td>
    <td><a href="https://blog.csdn.net/qq_20011607/article/details/82351225">https://blog.csdn.net/qq_20011607/article/details/82351225</a></td>
  </tr>
</table>

# 正文

## 算法思想原理

### 输入

```java
/* 传入参数 */
int[] A // 原数组(可以是任何类型的数据，这边使用 int 类型为示例)

/* 限制 */
int n // 数组长度
```

### 算法思想

归并排序的核心思想就是将数组切分成左右两个部分，分别排序后再将两个排序好的数组合并。不断切割出子数组知道数组长度为 1 时直接返回。

- 动图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/merge_sort.gif)

### 算法流程

伪代码（参考：算法导论）

```
Merge-Sort(A)
    Merge-Sort(A, 0, A.length - 1)

Merge-Sort(A, l, r)
    if l = r
        return
    m = (l + r) / 2
    Merge-Sort(A, l, m)
    Merge-Sort(A, m + 1, r)
    
    n1 = m - l + 1
    n2 = r - m
    let L[0 ... n1] and R[0 ... n2]
    for i = 0 to n1 - 1
        L[i] = A[l + i]
    for j = 0 to n2 - 1
        R[j] = A[m + j + 1]
    L[n1] = infinite
    R[n2] = infinite
    
    i = 0
    j = 0
    for k=l to r
        if L[i] <= R[j]
            A[k] = L[i]
            i = i + 1
        else
            A[k] = R[j]
            j = j + 1
```

```
参数说明
l 为最左边元素的下标
r 为最右边元素的下标
m 为两个子数组的中间结构(=左子数组的最后一个位置=右子数组的前一个位置)
```

- 排序的区间从 \[0, A.length - 1\] 开始
- 如果 l = r 代表子数组长度为 1 直接返回
- 否则对左右子数组(L\[l, m\] 和 R\[m + 1, r\])分别递归进行排序，然后再**合并**两个子数组
- 合并时按顺序将两个数组由前到后回填到原数组中

### 算法复杂度分析

先说结论：

1. 时间复杂度：**O(nlogn)**
2. 空间复杂度：**O(n)**
3. 原址排序
4. 稳定的

- 时间复杂度：**O(nlogn)**

由于每次将数组切分成两半，合并的代价为 O(n)，则递归后的复杂度为 **O(nlogn)**

- 空间复杂度：**O(n)**

保存子数组时使用的 L 和 R 加起来最大的长度为 n + 2 的数组，复杂度为 **O(n)**

- 原址排序

在合并阶段将子数组回填到原数组空间中，属于原址排序

- 稳定的

回填过程由于判断 `L[i] <= R[j]`，所以两个值相等时左边的数先填入，相对位置没有改变，所以是排序是稳定的

## Java 实现

- `MergeSort.java`

```java
public class MergeSort {
    public static void sort(int[] A) {
        sort(A, 0, A.length - 1);
    }

    private static void sort(int[] A, int l, int r) {
        if (l == r) return;
        int m = (l + r) / 2;
        sort(A, l, m);
        sort(A, m + 1, r);
        merge(A, l, m, r);
    }

    private static void merge(int[] A, int l, int m, int r) {
        int lenL = m - l + 1, lenR = r - m, i, j;
        int[] L = new int[lenL + 1];
        int[] R = new int[lenR + 1];
        for(i=0 ; i<lenL ; i++) L[i] = A[l + i];
        for(j=0 ; j<lenR ; j++) R[j] = A[m + j + 1];
        L[lenL] = R[lenR] = Integer.MAX_VALUE;
        i = j = 0;
        for(int k=l ; k<=r ; k++) {
            A[k] = L[i] <= R[j] ? L[i++] : R[j++];
        }
    }
}
```

- `MergeSortTest.java`

```java
public class MergeSortTest {
    @Test
    public void test() {
        int[] A = new int[]{1,3,5,7,9,2,4,6,8,0};
        int[] ans = new int[]{0,1,2,3,4,5,6,7,8,9};
        System.out.println("origin: " + Arrays.toString(A));
        MergeSort.sort(A);
        System.out.println("sorted: " + Arrays.toString(A));
        assertArrayEquals(ans, A);
    }
}
```

- 输出

```
origin: [1, 3, 5, 7, 9, 2, 4, 6, 8, 0]
sorted: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

# 結語

归并排序的复杂度为 O(nlogn) 与快排相同，但是由于中间划分子数组并回填的操作代价使得他还是比快速排序要慢。不过归并排序是经典的分治算法解法，他的算法核心就是根据分治算法的两种核心操作：`划分(partition)`子结构以及`合并(merge)`子问题解。后续使用的快速排序相当于分别对这两种操作进行优化，动态规划则是将划分数组抽象成更高级的`划分最佳子结构`的问题，可见分治思想的灵活性。
