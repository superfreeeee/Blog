# NumPy：索引 + 切片

@[TOC](文章目錄)

<!-- TOC -->

- [NumPy：索引 + 切片](#numpy索引--切片)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [一維數組](#一維數組)
  - [多維數組](#多維數組)
  - [其他索引](#其他索引)
    - [布爾索引](#布爾索引)
  - [order 排列方式](#order-排列方式)
- [結語](#結語)

<!-- /TOC -->

## 簡介

前一篇：<a href="https://blog.csdn.net/weixin_44691608/article/details/106873111">NumPy：起步 + 創建數組</a>。我們已經能夠建立多維數組了，接下來本篇將要來講解如何`索引(Index)`和`切片(Slice)`，簡單來說就是選定`特定元素`或是`指定區間的元素`。

## 參考

<table>
  <tr>
    <td>NumPy 切片和索引</td>
    <td><a href="https://www.runoob.com/numpy/numpy-ndexing-and-slicing.html">https://www.runoob.com/numpy/numpy-ndexing-and-slicing.html</a></td>
  </tr>
</table>

# 正文

## 一維數組

在講解 numpy 強大的索引和切片能力之前，我們先來回顧一下最基本的一維數組的索引方式：

```py
import numpy as np

a = np.arange(10)

# 選擇特定元素
print(a[3])  # 3

# 選擇特定範圍
print(a[1:7]) # [1 2 3 4 5 6]

# 使用內置函數 slice
print(a[slice(1,7,2)]) # [1 3 5]

# 傳入索引數組，numpy 特有
print(a[[2,4,6]]) # [2 4 6]
```

切片索引的參數與內置函數 `slice` 是等價的，一個是 `[start:end:step]`、一個是 `slice(start,end,step)`；而最後一種透過傳入`索引數組`屬於 numpy 的特有寫法

## 多維數組

接下來我們就可以擴展到多維數組的索引和切片了，每個維度的索引都可以使用一維數組的四種索引方式，多個維度之間使用 `,` 分隔，使用 `...` 省略符號作為缺省（即選取所有，與 `[:]` 等價）：

```py
import numpy as np

# 原數組
a = np.arange(25).reshape(5,5)
# [[ 0  1  2  3  4]
#  [ 5  6  7  8  9]
#  [10 11 12 13 14]
#  [15 16 17 18 19]
#  [20 21 22 23 24]]

# 選擇特定元素
print(a[2][3])
# 13
print(a[3][2])
# 17

# 選擇限定範圍
print(a[1: , :2])
# [[ 5  6]
#  [10 11]
#  [15 16]
#  [20 21]]
print(a[..., :3])
# [[ 0  1  2]
#  [ 5  6  7]
#  [10 11 12]
#  [15 16 17]
#  [20 21 22]]

# 傳入索引數組
print(a[[2,4], 0:5:2])
# [[10 12 14]
#  [20 22 24]]
print(a[...,3])
# [ 3  8 13 18 23]
```

我們使用二維數組來畫圖說明

- 原數組

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/array.png)

- 索引

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/array_index.png)

- 切片

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/array_slice_1.png)

- 索引數組

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/array_slice_2.png)

是不是很簡單呢～

## 其他索引

除了最基本的四種索引以及運用 `,` 組合出多維索引以外，這邊再介紹兩種其他類型的索引

### 布爾索引

透過一個`布爾表達式`來創建的索引方式，將會遍歷整個數組，就好像使用 `map` 函數最後匯集成一個一維數組一樣

```py
import numpy as np

# 原數組
a = np.arange(25).reshape(5,5)
# [[ 0  1  2  3  4]
#  [ 5  6  7  8  9]
#  [10 11 12 13 14]
#  [15 16 17 18 19]
#  [20 21 22 23 24]]

# 使用布爾表達式（比較邏輯運算符）
print(a[7 <= a])
# [7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24]

# 使用 ~ 非運算符
print(a[~(a < 7)])
# [7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24]
```

## order 排列方式

最後的最後我們來解釋一下 `np.array` 中 `order` 參數的含義，相信看完上一篇的人依舊是一頭霧水，我也是 hhh，這邊將會詳細解說一次。

原來 C 語言的數組保存方式是行優先(row)的，內存模型如下：

```
a = np.arange(9).reshape(3,3)

+---------+
| a[0][0] |
+---------+
| a[0][1] |
+---------+
| a[0][2] |
+---------+
| a[1][0] |
+---------+
| a[1][1] |
+---------+
| a[1][2] |
+---------+
| a[2][0] |
+---------+
| a[2][1] |
+---------+
| a[2][2] |
+---------+
```

而 Fortran 語言的數組則是列優先，內存如下

```
b = a.copy(order='F')

+---------+
| b[0][0] |
+---------+
| b[1][0] |
+---------+
| b[2][0] |
+---------+
| b[0][1] |
+---------+
| b[1][1] |
+---------+
| b[2][1] |
+---------+
| b[0][2] |
+---------+
| b[1][2] |
+---------+
| b[2][2] |
+---------+
```

然而這對於數組索引上並不會有任何影響，只有在使用`迭代器(np.nditer)`遍歷的時候會影響順序：

```py
import numpy as np

a = np.arange(9).reshape(3,3)
b = a.copy(order='F')

# 原數組，可以看到 order 並不影響整個數組的元素索引
print(a)
# [[0 1 2]
#  [3 4 5]
#  [6 7 8]]
print(b)
# [[0 1 2]
#  [3 4 5]
#  [6 7 8]]

# 以行(row)列(col)遍歷數組，可以看到結果相同
print('\na:')
for row in a:
  for col in row:
    print(col, end=",")
# a:
# 0,1,2,3,4,5,6,7,8,

print('\nb:')
for row in b:
  for col in row:
    print(col, end=",")
# b:
# 0,1,2,3,4,5,6,7,8,

# 然而使用迭代器遍歷的時候，則會依照內存模型的順序遍歷，也就是 order='F'會以行優先的形式
print('\na by nditer:')
for x in np.nditer(a):
  print(x, end=",")
# a by nditer:
# 0,1,2,3,4,5,6,7,8,

print('\nb by nditer:')
for x in np.nditer(b):
  print(x, end=",")
# b by nditer:
# 0,3,6,1,4,7,2,5,8,
```

# 結語

本篇介紹了 numpy 中多維數組的索引方式，切片和布爾索引提供了強大的查找元素的能力，使開發者能方便的指定提取數組的目標範圍。下一篇我們將開始介紹 numpy 最強大的能力之一：多維數組運算函數。
