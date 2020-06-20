# NumPy：創建數組

@[TOC](文章目錄)

<!-- TOC -->

- [NumPy：創建數組](#numpy創建數組)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Ndarray 對象](#ndarray-對象)
    - [Memory Layout 內存佈局](#memory-layout-內存佈局)
    - [Properties 屬性](#properties-屬性)
  - [Data Type Objects 數據類型對象(dtype)](#data-type-objects-數據類型對象dtype)
    - [Types  數據類型](#types--數據類型)
    - [`np.dtypes` 創建數據類型對象](#npdtypes-創建數據類型對象)
    - [內置類型符號表](#內置類型符號表)
  - [Create 創建數組](#create-創建數組)
    - [初始化數組](#初始化數組)
      - [語法](#語法)
      - [Sample](#sample)
    - [從已有資源創建 Ndarray](#從已有資源創建-ndarray)
      - [語法](#語法-1)
      - [Sample](#sample-1)
    - [指定數值範圍](#指定數值範圍)
      - [語法](#語法-2)
      - [Sample](#sample-2)
- [結語](#結語)

<!-- /TOC -->

## 簡介

numpy 作為一個強化版的數組庫，提供了更強大的數組創建和操作能力，也提供了許多默認的矩陣運算，是許多其他基於 python 的包的基礎庫之一，基本上沒有什麼操作能離得開數組和矩陣。

本篇作為 numpy 開篇，將首先介紹 numpy 包裡面最重要的數組對象(ndarray)相關的屬性以及創建方法。

## 參考

<table>
  <tr>
    <td>NumPy 创建数组-菜鸟教程</td>
    <td><a href="https://www.runoob.com/numpy/numpy-array-creation.html">https://www.runoob.com/numpy/numpy-array-creation.html</a></td>
  </tr>
  <tr>
    <td>Array objects-NumPy 官方文档</td>
    <td><a href="https://numpy.org/doc/stable/reference/arrays.html">https://numpy.org/doc/stable/reference/arrays.html</a></td>
  </tr>
</table>

# 正文

## Ndarray 對象

首先先要使用 NumPy 當然要先理解最重要的對象，也就是數組。在 NumPy 裡面新建了一個 Ndarray 作為 List 的強化版，接下來就先來看看 Ndarray 對象的內存結構以及各樣屬性吧。

### Memory Layout 內存佈局

Ndarray(N-dimensional array) 的內存模型主要有三個部分：

1. array 數組對象本身：一個 Ndarray 是一個由`同樣大小(dtype)的連續內存塊(block)`所組成的結構，`多維度(dimensional)`主要透過`索引(index)`的手段來體現
2. data-type 數據類型：另外保存一個`數據類型對象(dtype)`，同一個數組中所有內存塊都具有相同的大小和類型
3. scalar 標量類型：由於 Python 內置數據類型只有整數(int)和浮點數(float)兩種，而 NumPy 提供了 24 種類似 C 的數據類型，便是透過`標量(scalar)`的數據類型層次了來實現。詳細原理可以查詢相關內容。

### Properties 屬性

| Property | Description                                     |
| -------- | ----------------------------------------------- |
| flags    | 關於內存佈局的相關信息                          |
| shape    | 各維度大小                                      |
| strides  | 各維度進到下一個元素的偏移量(bytes)             |
| ndim     | 標示著此數組的維度(dimensional)，也可以說是深度 |
| data     | 保存著數據體的頭指針                            |
| size     | 數組的元素總個數                                |
| itemsize | 數組中每個元素的大小(bytes)                     |
| nbytes   | 數組的總大小(bytes)                             |
| base     | 創建數組的原對象指針                            |

- Sample

```py
import numpy as np

array = np.array([[1,2,3],[4,5,6],[7,8,9]])
print(array)
# [[1 2 3]
#  [4 5 6]
#  [7 8 9]]

print(array.flags)
#   C_CONTIGUOUS : True
#   F_CONTIGUOUS : False
#   OWNDATA : True
#   WRITEABLE : True
#   ALIGNED : True
#   WRITEBACKIFCOPY : False
#   UPDATEIFCOPY : False

print(array.shape)  # (3, 3)
print(array.strides)  # (24, 8)
print(array.ndim)  # 2
print(array.data)  # <memory at 0x11ee1d990>
print(array.size)  # 9
print(array.itemsize)  # 8
print(array.nbytes)  # 72
print(array.base)  # None
```

## Data Type Objects 數據類型對象(dtype)

接下來是 NumPy 提供的類似 C 的數據類型，用於擴展 Python 的數字數據類型

### Types  數據類型

- Boolean 布爾值類型

| Type   | Description        | Character Code |
| ------ | ------------------ | -------------- |
| bool\_ | Python 內置的 bool | `?`            |
| bool8  | 8 bits 的 bool     |

- Integer 整數類型

| Type      | Description              | Character Code |
| --------- | ------------------------ | -------------- |
| byte      | 對應 C 的 char 類型      | `b`            |
| short     | 對應 C 的 short 類型     | `h`            |
| intc      | 對應 C 的 int 類型       | `i`            |
| longlong  | 對應 C 的 long long 類型 | `q`            |
| int\_     | Python 內置的 int 類型   | `l`            |
| intp      | 指針類型                 | `p`            |
| int8      | 8-bits 的 int            |                |
| int16     | 16-bits 的 int           |                |
| int32     | 32-bits 的 int           |                |
| int64     | 64-bits 的 int           |                |
| ...       | ...                      | ...            |
| ubyte     | 無符號的 C byte          | `B`            |
| ushort    | 無符號的 C short         | `H`            |
| uintc     | 無符號的 C int           | `I`            |
| ulonglong | 無符號的 C long long     | `Q`            |
| uint      | 無符號的 Python int      | `L`            |
| uintp     | 無符號的 intp            | `P`            |
| uint8     | 無符號的 int8            |                |
| uint16    | 無符號的 int16           |                |
| uint32    | 無符號的 int32           |                |
| uint64    | 無符號的 int64           |                |

- Floating-point 浮點數類型

| Type      | Description          | Character Code |
| --------- | -------------------- | -------------- |
| half      | 一半大小的 float ?   | `e`            |
| single    | 對應 C 的 float      | `f`            |
| double    | 對應 C 的 double     |                |
| longfloat | 對應 C 的 long float | `g`            |
| float\_   | Python 內置的 float  | `d`            |
| float16   | 16-bits 的 float     |                |
| float32   | 32-bits 的 float     |                |
| float64   | 64-bits 的 float     |                |
| float96   | 96-bits 的 float     |                |
| float128  | 128-bits 的 float    |                |

- Complex 複數類型

| Type       | Description                 | Character Code |
| ---------- | --------------------------- | -------------- |
| csingle    |                             | `F`            |
| complex\_  | Python 內置的 complex       | `D`            |
| clongfloat | 對應 C 的 long float        | `G`            |
| complex64  | 兩個 32-bits 的 float 組成  |                |
| complex128 | 兩個 64-bits 的 float 組成  |                |
| complex192 | 兩個 96-bits 的 float 組成  |                |
| complex256 | 兩個 128-bits 的 float 組成 |                |

- 其他類型

| Type      | Description               | Character Code |
| --------- | ------------------------- | -------------- |
| bytes\_   | Python 內置的 bytes       | `S#`           |
| unicode\_ | Python 內置的 unicode/str | `U#`           |
| void      | 空類型                    | `V#`           |

### `np.dtypes` 創建數據類型對象

接下來我們就能夠創建數據類型對象(dtype)了，其實上面列了很多，常用的也那就那幾個：

- 使用`標量(scalar)`類型

```py
dt = np.dtype(np.int32)  # int32 類型
dt = np.dtype(np.complex128)  # complex128 類型
# 就是直接 np.xxx 就對了
# 默認為 float_
```

- 使用內置類型

```py
dt = np.dtype(int)  # 內置 int 類型
dt = np.dtype(float)  # 內置 float 類型
dt = np.dtype(object)  # 內置對象類型
# 默認為 float 類型
```

- 使用字符標示（上面提到的 Character Code）

```py
dt = np.dtype('?')  # 內置 bool 類型
dt = np.dtype('i')  # 內置 int 類型
dt = np.dtype('f')  # 內置 float 類型

dt = np.dtype('i4')  # 32-bits int 類型
dt = np.dtype('f8')  # 64-bits float 類型
dt = np.dtype('c16')  # 128-bits complex 類型
dt = np.dtype('a25')  # 25 個字符的 C 字符串（'\0' 結尾）
dt = np.dtype('U25')  # 25 個字符的 str
```

- 聯合類型

```py
dt = np.dtype("i4, (2,3)f8, f4")
dt = np.dtype("a3, 3u8, (3,4)a10")
```

### 內置類型符號表

這邊提供一個內置類型的符號表快速索引，供參考：

| Charater | Type                                |
| -------- | ----------------------------------- |
| b        | 布爾類型                            |
| i        | 有符號整數                          |
| u        | 無符號整數                          |
| f        | 浮點數                              |
| c        | 複數(由兩個浮點數組成)              |
| O        | 對象類型                            |
| m        | timedelta 時間間隔                  |
| M        | datetime 日期時間                   |
| S, a     | 字符串類型(byte- 字節流，`\0` 結尾) |
| U        | 字符串類型(Unicode)                 |
| V        | 空類型                              |

## Create 創建數組

介紹了這麼多前置條件，現在就要進入重頭戲：創建數組了。首先我們先羅列接下來要介紹的創建數組的方法（這不是全部，僅挑常用的，剩下的可以自己再查相關文檔）：

| Type           | Function      | Description                                       |
| -------------- | ------------- | ------------------------------------------------- |
| 初始化數組     | np.empty      | 創建新數組，內容為隨機值（未初始化）              |
|                | np.zeros      | 創建新數組，以 0 填充                             |
|                | np.ones       | 創建新數組，以 1 填充                             |
| 從已有資源創建 | np.array      | 以傳入數組創建新數組                              |
|                | np.asarray    | 同上，可選參數較少                                |
|                | np.frombuffer | 從`數據流(buffer)`創建數組                        |
|                | np.fromiter   | 從`可迭代對象(iterable)`創建數組                  |
| 指定數值範圍   | np.arange     | 創建指定數值範圍的數組，參數與 `range()` 方法類似 |
|                | np.linspace   | 創建等差數列                                      |
|                | np.logspace   | 創建等比數列                                      |

### 初始化數組

這邊的創建對象方法就是指定（或不指定）單一值用來開闢一個空間而已。

#### 語法

語法中傳入的參數表示默認值

```py
# 空數組（未初始化）
numpy.empty(shape, dtype = float, order = 'C')
# 0 數組
numpy.zeros(shape, dtype = float, order = 'C')
# 1 數組
numpy.ones(shape, dtype = float, order = 'C')
```

- 參數說明

| Param | Description                  |
| ----- | ---------------------------- |
| shape | 表示各維度大小的元組(tuple)  |
| dtype | 表示數組的數據類型           |
| order | 表示數據在內存模型的排列方式 |

#### Sample

```py
a = np.empty((2,4))
print(a)
# [[1.68653148e-051 7.16572198e-038 2.46580783e+184 1.25758944e-047]
#  [4.42905954e+175 4.65851170e-033 1.17633826e-047 0.00000000e+000]]

a = np.zeros((3,2), dtype=np.int8)
print(a)
# [[0 0]
#  [0 0]
#  [0 0]]

a = np.ones((3,3), dtype=np.complex)
print(a)
# [[1.+0.j 1.+0.j 1.+0.j]
#  [1.+0.j 1.+0.j 1.+0.j]
#  [1.+0.j 1.+0.j 1.+0.j]]
```

### 從已有資源創建 Ndarray

所謂的已有資源通常有三種：

1. `List 列表`類型，也就是內置數組，如 `[1,2,3]`
2. `流(buffer)`類型，如 `b'http://google.com'`
3. `可迭代對象(iterable)`類型，如 `iter()`

#### 語法

```py
# 數組方法一
numpy.array(object, dtype = float, copy = True, order = 'C', subok = True, ndmin = 0)
# 數組方法二
numpy.asarray(a, dtype = None, order = None)
# 從`流(buffer)`建立數組
numpy.frombuffer(buffer, dtype = None, count = -1, offset = 0)
# 從`可迭代對象`建立數組
numpy.fromiter(iterable, dtype, count=-1)
```

- 參數說明

| Param     | Description                                           |
| --------- | ----------------------------------------------------- |
| object, a | 由`列表(List)`或`元組(Tuple)`任意組合嵌套的數列       |
| dtype     | 數據類型對象                                          |
| copy      | 是否需要複製對象                                      |
| order     | 內存模型中排列方式                                    |
| subok     | False 時所有元素都必須是嚴格的 array 而不能是聯合類型 |
| ndim      | 生成數組的最小維度數                                  |
| buffer    | 以流的形式讀取對象                                    |
| count     | 讀取的數據量，默認 -1 為全部                          |
| offset    | 讀取起始位置，默認 0 為開頭                           |
| iterable  | 可迭代對象                                            |

#### Sample

```py
array = np.array([1,2,3,4,5], dtype = np.complex)
print(array)
# [1.+0.j 2.+0.j 3.+0.j 4.+0.j 5.+0.j]

array = np.array([1,2,3], ndmin = 2)
print(array)
# [[1 2 3]]

array = np.asarray([(1,2),(3,4,5)])
print(array)
# [(1, 2) (3, 4, 5)]

array = np.frombuffer(b'Hello World', dtype='S1', count=5)
print(array)
# [b'H' b'e' b'l' b'l' b'o']

array = np.fromiter(iter(range(5)), dtype=float)
print(array)
# [0. 1. 2. 3. 4.]
```

### 指定數值範圍

最後一種是指定數值範圍，就好像在 for 循環使用 `range` 建立的迭代序列一樣

#### 語法

```py
# 指定範圍，stop 不包含
numpy.arange(start, stop, step=1, dtype)
# 等差數列
numpy.linspace(start, stop, num=50, endpoint=True, retstep=False, dtype=None)
# 等比數列
numpy.logspace(start, stop, num=50, endpoint=True, base=10.0, dtype=None)
```

- 參數說明

| Param    | Description                                                |
| -------- | ---------------------------------------------------------- |
| start    | 起始值                                                     |
| stop     | 終止值（arange 默認不包含，linspace 和 logspace 默認包含） |
| step     | 步長(默認 1) or 等差值() or 等比值()                       |
| num      | 生成樣本數量，默認 50                                      |
| endpoint | 默認 True，表示是否包含 stop 值                            |
| retstep  | 默認 False，打印時顯示步長                                 |

#### Sample

```py
a = np.arange(1, 10)
print(a)
# [1 2 3 4 5 6 7 8 9]

a = np.linspace(0, 10, endpoint=False, retstep = True)
print(a)
# (array([0. , 0.2, 0.4, 0.6, 0.8,
#         1. , 1.2, 1.4, 1.6, 1.8,
#         2. , 2.2, 2.4, 2.6, 2.8,
#         3. , 3.2, 3.4, 3.6, 3.8,
#         4. , 4.2, 4.4, 4.6, 4.8,
#         5. , 5.2, 5.4, 5.6, 5.8,
#         6. , 6.2, 6.4, 6.6, 6.8,
#         7. , 7.2, 7.4, 7.6, 7.8,
#         8. , 8.2, 8.4, 8.6, 8.8,
#         9. , 9.2, 9.4, 9.6, 9.8]), 0.2)

a = np.logspace(0, 10, num=11, base=2, dtype=np.int32)
print(a)
# [   1    2    4    8   16   32   64  128  256  512 1024]
```

# 結語

本篇簡單介紹了 numpy 提供的數據類型，以及 ndarray 多維數組對象的三種共十個創建對象的方法。後續我們將會來介紹 numpy 中提供的數組訪問、矩陣操作、高級索引、切片等操作，敬請期待。
