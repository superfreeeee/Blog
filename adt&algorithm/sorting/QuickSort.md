# Sorting 排序算法: Quick Sort 快速排序

@[TOC](文章目錄)

<!-- TOC -->

- [Sorting 排序算法: Quick Sort 快速排序](#sorting-排序算法-quick-sort-快速排序)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [算法思想原理](#算法思想原理)
    - [输入](#输入)
    - [算法思想](#算法思想)
    - [算法流程](#算法流程)
    - [算法复杂度分析](#算法复杂度分析)
  - [Java 实现](#java-实现)
- [结语](#结语)

<!-- /TOC -->

## 简介

快速排序作为最常被应用的排序算法，因为其拥有最好的平均时间效率，同时只用了常数的额外空间，所以受到大多数应用的青睐。

## 参考

<table>
  <tr>
    <td></td>
    <td><a href=""></a></td>
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

快速排序也和归并排序一样采用了分治的思想，但是与归并不同的是`划分(partition)阶段`并不只是单纯的分成两半，而是选定一个`基准值(pivot)`，将整个数组划分成"比 pivot 小"和"比 pivot 大"的两半，然后再分别排序。分类时的交换使得`合并(merge)阶段`只需要花费常数的时间代价即可完成

- 动图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/quick_sort.gif)

### 算法流程

伪代码（参考：算法导论）

```
Quick-Sort(A)
    Quick-Sort(A, 0, A.length)

Quick-Sort(A, l, r)
    if l < r
        q = partition(A, l, r)
        Quick-Sort(A, l, q - 1)
        Quick-Sort(A, q + 1, r)

Partition(A, l, r)
    pivot = A[r]
    i = l - 1
    for j = l to r - 1
        if A[j] <= pivot
            i = i + 1
            exchange A[i] and A[j]
    exchange A[i + 1] and A[r]
    return i + 1
```

与归并排序相同，分治算法使用左右界作为参数，以 \[0 ... A.length - 1\] 区间为起始。当子数组长度大于 1 时(l < r)，就进行划分，然后再分别对两个子数组进行排序。
在划分阶段，直接选取最右边的数作为`基准(pivot)`，i 指向左子数组(比 pivot 小)的末尾，j 遍历一遍数组，遇到比 pivot 小的数就交换到 i 位置。循环结束时将第 i + 1 个数也就是第一个大于 pivot 的值与 pivot 交换，就相当于划分出以 `i + 1` 位置为界：i + 1 以左的都小于(等于) pivot，以右的都大于 pivot。如此递归知道数组长度为 1。

### 算法复杂度分析

先说结论：

1. 时间复杂度：最坏 O(n<sup>2</sup>) / 平均 **O(nlogn)**
2. 空间复杂度：O(1)
3. 原址排序
4. 不稳定的

- 时间复杂度：最坏 O(n<sup>2</sup>) / 平均 **O(nlogn)**

在最坏的情况下每次划分都将所有元素划分到同一边，这时时间复杂度就接近 O(n<sup>2</sup>)，然而这种情况是极少出现的，只要中间有几次有分出两个子数组，渐进时间上还是趋近于 O(nlogn) 的，所以平均效率上是极好的

- 空间复杂度：1

在 partition 阶段进行两两交换，没有使用其他额外的空间，复杂度为 O(1)

- 原址排序

在原数组内交换元素，使用区间作为排序界线

- 不稳定的

由于交换是小于等于 pivot 的一律向前交换，所以当 pivot 选取到存在重复的元素时会出现相对顺序改变的问题，所以算法是不稳定的

## Java 实现

- `QuickSort.java`

```java
public class QuickSort {
    public static void sort(int[] A) {
        sort(A, 0, A.length - 1);
    }

    private static void sort(int[] A, int l, int r) {
        if (l < r) {
            int m = partition(A, l, r);
            sort(A, l, m - 1);
            sort(A, m + 1, r);
        }
    }

    private static int partition(int[] A, int l, int r) {
        int pivot = A[r];
        int i = l - 1;
        for (int j = l; j < r; j++) {
            if(A[j] <= pivot) {
                int tmp = A[++i];
                A[i] = A[j];
                A[j] = tmp;
            }
        }
        A[r] = A[i + 1];
        A[i + 1] = pivot;
        return i + 1;
    }
}

```

- `QuickSortTest.java`

```java
public class QuickSortTest {
    @Test
    public void test() {
        int[] A = new int[]{1,3,5,7,9,2,4,6,8,0};
        int[] ans = new int[]{0,1,2,3,4,5,6,7,8,9};
        System.out.println("origin: " + Arrays.toString(A));
        QuickSort.sort(A);
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

# 结语

作为拥有最好的比较排序代价，同时使用最少的额外空间(O(1))的比较排序算法，快速排序是最常被调用的排序算法之一。然而在基本款的快排之上，我们还可以在 pivot 的选取上进行优化，使得 partition 过后的子数组更加平均以提升快速排序的平均效率。
