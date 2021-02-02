# Sorting 排序算法: Heap Sort 堆排序

@[TOC](文章目錄)

<!-- TOC -->

- [Sorting 排序算法: Heap Sort 堆排序](#sorting-排序算法-heap-sort-堆排序)
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

今天来介绍一个相对于其他比较排序算法比较特别的一个排序算法-堆排序(HeapSort)，堆排序顾名思义借助了`堆(Heap)`的数据结构性质，并借由`建堆(build-heap)`的复杂度 `O(n)`和中间抽取最大值需要`恢复堆的性质(maxHeapify)`的复杂度 `O(logn)`，组合操作使得堆排序的时间复杂度能够渐进趋近于比较排序的极限 **O(nlogn)**。

## 參考

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

堆排序的核心思想是使用最大堆(升序排序时)的性质，每次将最大值也就是根节点取出，以最后一个元素替代并恢复最大堆的性质，而所有的数抽出的顺序正好是由大到小，以此来完成排序。

- 图解：（绿色为已排序好的部分

![](https://picures.oss-cn-beijing.aliyuncs.com/img/heap_sort.jpeg)

### 算法流程

伪代码

```
Heap-Sort(A)
    Build-Max-Heap(A)
    for i = A.length - 1 to 0
        Swap A[0] and A[i]
        MaxHeapify(A) while heap size = i
```

### 算法复杂度分析

先说结论：

1. 时间复杂度：**O(nlogn)**
2. 空间复杂度：1 (额外空间复杂度，不考虑原数组的空间)
3. 原址排序
4. 不稳定的

- 时间复杂度：**O(nlogn)**

一开始进行建堆的复杂度为 O(n)，后面需要取 n 次最大值，每次恢复堆性质的复杂度为 O(logn)，故算法时间复杂度为 O(n) + n * O(logn) = **O(nlogn)**

- 空间复杂度：1、原址排序

由于建堆、抽取最大值并恢复堆性质等操作都是在原数组进行，没有使用额外空间，故算法是**空间复杂度为 1** 且为**原址排序**

- 不稳定的

由于恢复堆性质的过程中出现相同值依旧会向下交换，会打乱相同值的相对顺序，所以堆排序是**不稳定的**

## Java 实现

- `HeapSort.java`

```java
package sorting;

/**
 * 堆排序
 */
public class HeapSort {

    /**
     * 主方法
     *
     * @param A
     */
    public static void sort(int[] A) {
        // 建最大堆
        for (int i = parent(A.length - 1); i >= 0; i--) {
            maxHeapify(A, A.length, i);
        }
        for (int i = 0; i < A.length; i++) {
            // 抽取最大值并恢复堆
            swap(A, 0, A.length - i - 1);
            maxHeapify(A, A.length - i - 1, 0);
        }
    }

    /**
     * 将指定下标的数下降到适当的位置
     *
     * @param A
     * @param len
     * @param pos
     */
    private static void maxHeapify(int[] A, int len, int pos) {
        if (left(pos) >= len) return;
        int max = pos;
        if (A[max] <= A[left(pos)]) max = left(pos);
        if (right(pos) < len && A[max] <= A[right(pos)]) max = right(pos);
        if (max != pos) {
            swap(A, pos, max);
            maxHeapify(A, len, max);
        }
    }

    /**
     * 交换
     *
     * @param A
     * @param i
     * @param j
     */
    private static void swap(int[] A, int i, int j) {
        int tmp = A[i];
        A[i] = A[j];
        A[j] = tmp;
    }

    /**
     * 父节点下标
     *
     * @param pos
     * @return
     */
    private static int parent(int pos) {
        return (pos - 1) / 2;
    }

    /**
     * 左子节点下标
     *
     * @param pos
     * @return
     */
    private static int left(int pos) {
        return pos * 2 + 1;
    }

    /**
     * 右子节点下标
     *
     * @param pos
     * @return
     */
    private static int right(int pos) {
        return pos * 2 + 2;
    }
}
```

- `HeapSortTest.java`

```java
public class HeapSortTest {
    @Test
    public void test_1() {
        int[] A = new int[]{1,3,5,7,9,2,4,6,8,0};
        int[] ans = new int[]{0,1,2,3,4,5,6,7,8,9};
        System.out.println("origin: " + Arrays.toString(A));
        HeapSort.sort(A);
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

虽然堆排序的渐进时间复杂度与快速排序相同，不过在实际应用上却较少使用，因为建堆和恢复堆性质算法所隐藏的系数较大。`最大/最小堆(max/min-heap)`数据结构的另一个较常用的用途是`最大/最小优先队列(Priority Queue)`，透过维持最大/最小堆的性质可以在 `O(1)` 的时间复杂度找到队列中最大/最小的值，并仅仅使用 `O(logn)` 的代价恢复堆的性质，有关优先队列的内容会放在`堆数据结构`的时候再详细介绍。