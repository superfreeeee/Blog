# Sorting 排序算法: Insertion Sort 插入排序

@[TOC](文章目錄)

<!-- TOC -->

- [Sorting 排序算法: Insertion Sort 插入排序](#sorting-排序算法-insertion-sort-插入排序)
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

比较排序算法的时间复杂度理论极限是 O(n<sup>2</sup>)，但是作为排序中非常通用，或是作为分治算法中的子历程，在确保数组长度足够小的情况下平均性能是要优于快速排序的，同时算法的简单性和易读性也适合作为排序算法的入门介绍。

## 參考

<table>
  <tr>
    <td>动图来源</td>
    <td><a href="https://blog.csdn.net/mengmengdajuanjuan/article/details/83932933">https://blog.csdn.net/mengmengdajuanjuan/article/details/83932933</a></td>
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

如果在每轮迭代中把数组中的前 i(i = 0...n-1) 位视为已经完成排序的序列，则下一个数(`num`)只要又后往前找到第一个比 `num` 小的数，将 `num` 插入到该数之后便完成第 i + 1 个数的插入

- 动图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/insertion_sort.gif)

### 算法流程

伪代码（参考：算法导论）

```
Insertion-Sort(A)
for j = 1 to A.length - 1
    key = A[j]
    i = j - 1
    while i>= 0 and A[i] > key
        A[i + 1] = A[i]
        i = i - 1
    A[i + 1] = key
```

算法的开始将第一个数(下标从零开始)视为已排序好的数组，每轮迭代中保存 A\[j\] 作为 key，i 指向前一位；如果前一位大于当前的数则向后移一位，最后将 key 插入到第一个小于等于它的数后面也就是在 A\[i+1\]的位置

### 算法复杂度分析

先说结论：

1. 时间复杂度：**O(n<sup>2</sup>)**
2. 空间复杂度：1 (额外空间复杂度，不考虑原数组的空间)
3. 原址排序
4. 稳定的

- 时间复杂度：O(n<sup>2</sup>)

外层循环 j 遍历整个数组，复杂度为 n；内层循环不一定，平均需要移动 j / 2 的距离复杂度也是 n；所以整个算法的复杂度为 **O(n<sup>2</sup>)**

- 空间复杂度：1 
  
只使用了常数个额外空间

- 原址排序

在原数组内部进行排序，属于原址排序

- 稳定的

由于交换的条件是前一个数要**大于**当前的数，所以对于一样大的两个值的相对位置不会改变，所以整个交换是稳定的。

## Java 实现

- `InsertionSort.java`

```java
public class InsertionSort {
    public static void sort(int[] A) {
        for(int i=1 ; i < A.length ; i++) {
            int num = A[i];
            int p = i - 1;
            while(p >= 0 && A[p] > num) {
                A[p + 1] = A[p];
                p--;
            }
            A[p + 1] = num;
        }
    }
}

```

- `InsertionSortTest.java`

```java
public class InsertionSortTest {
    @Test
    public void test_1() {
        int[] A = new int[]{1,3,5,7,9,2,4,6,8,0};
        int[] ans = new int[]{0,1,2,3,4,5,6,7,8,9};
        System.out.println("origin: " + Arrays.toString(A));
        InsertionSort.sort(A);
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

在大多数场景之下快速排序是通用场景下的排序算法首选，也是平均性能最好的。而排序算法作为平方时间复杂度的算法通常需要在特殊场景下、或是确保数组大小足够小时采用，才能保证算法的效率。
