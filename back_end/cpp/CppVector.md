# Cpp 進階：Vector 向量

@[TOC](文章目錄)

<!-- TOC -->

- [Cpp 進階：Vector 向量](#cpp-進階vector-向量)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Import 導入庫](#import-導入庫)
  - [Overview 總覽](#overview-總覽)
    - [Printer 輔助函數](#printer-輔助函數)
  - [Declaration 變量聲明](#declaration-變量聲明)
    - [構造函數語法](#構造函數語法)
  - [Access 訪問元素(查、改)](#access-訪問元素查改)
    - [訪問元素(查、改)相關語法](#訪問元素查改相關語法)
  - [Insert & Erase 增刪元素(增、刪)](#insert--erase-增刪元素增刪)
    - [增刪元素(增、刪)相關語法](#增刪元素增刪相關語法)
  - [Capacity 容量](#capacity-容量)
    - [容量相關語法](#容量相關語法)
  - [Iterator 迭代](#iterator-迭代)
    - [獲取迭代器語法](#獲取迭代器語法)
- [結語](#結語)

<!-- /TOC -->

## 簡介

C++ 中著名的`STL容器類(Standard Template Library 標準模板庫)`，是一些常用的`數據結構(data structure)`的實現，其中分成兩大類：

1. 序列式容器：以`序列(sequence)`的形式保存內容元素。主要實現有 `vector 向量`、`list 列表`、`deque 雙向隊列`、`stack 棧`、`queue 隊列`、`priority_queue 優先級隊列`

2. 關聯式容器：以`鍵值對(key-value)`的形式保存信息，內部數據結構可能使用各種的`樹(tree)`或`哈希表(hash-table)`來提升查找效率。主要實現有 `set 集合`、`map 映射表`、`multiset 多鍵集合`、`multimap 多鍵映射表`，以及上述容器的哈希(hash)版本，以 `hash_` 開頭

關於更多 STL 的資訊可以參考鏈接一，或是自行查詢相關資料。本篇作為 STL 容器系列的開篇，先來介紹最常用的 `Vector 向量`

## 參考

<table>
  <tr>
    <td>標準C++中的STL容器類簡介</td>
    <td><a href="https://www.itread01.com/content/1541343903.html">https://www.itread01.com/content/1541343903.html</a></td>
  </tr>
  <tr>
    <td>C/C++ - Vector (STL) 用法與心得完全攻略</td>
    <td><a href="https://mropengate.blogspot.com/2015/07/cc-vector-stl.html">https://mropengate.blogspot.com/2015/07/cc-vector-stl.html</a></td>
  </tr>
</table>

# 正文

## Import 導入庫

```cpp
#include<vector>
```

## Overview 總覽

`vector 向量`跟 Java 中的 ArrayList 很像，都是一般`數組(array)`的強化版，能夠自動擴展適當大小，以及自動管理元素的進出等，我們將 vector 所有方法分成五類

- Declaration 變量聲明
  - 棧(stack)上變量
  - 堆(heap)上變量
  - 三種構造函數
- Access 訪問元素
  - 偏移量訪問
  - `at`：訪問指定位置
  - `front`：訪問第一個元素
  - `back`：訪問最後一個元素
- Modify 插入和刪除
  - `push_back`：在最後插入元素
  - `pop_back`：彈出最後一個元素
  - `insert`：插入元素到指定位置
  - `erase`：刪除指定區間
  - `clear`：清空元素
- Capacity 容量
  - `empty`：檢查向量是否為空
  - `size`：獲取當前向量大小
  - `resize`：修改當前向量大小
  - `capacity`：取得當前數組大小（與內存分配相關）
  - `assign`：重置向量
  - `reserve`：配置更多內存
- Iterator 迭代
  - `begin`：返回一個 iterator，指向第一個元素
  - `end`：返回一個 iterator，指向最後一個元素
  - `rbegin`：返回一個反向 iterator，指向最後一個元素（方向向前）
  - `rend`：返回一個反向 iterator，指向第一個元素（方向向前）

### Printer 輔助函數

在開始介紹所有方法之前，我們先定義一個用於打印向量內容的輔助函數：

```cpp
// 打印向量
template <class T>
void print_vector(const vector<T>& v) {
    cout << "[";
    for(int i=0 ; i<v.size() ; i++) {
        cout << v[i] << (i < v.size() - 1 ? ", " : "");
    }
    cout << "]" << endl;
}
```

## Declaration 變量聲明

接下來就開始介紹該如何使用 vector，首先我們需要聲明變量。在 cpp 中 class 有兩種生存位置，一個是`棧(stack)`上的變量，一個是`堆(heap)`上的變量，聲明方式如下：

```cpp
int main() {
  // 聲明棧上的變量
  vector<int> v1;
  // 聲明堆上的變量
  vector<int>* v2 = new vector<int>;
}
```

接下來是各種構造函數的形式（vector 為`模板(template)`類，在 `<>` 內傳入泛型類型）：

### 構造函數語法

```cpp
// 默認構造函數
vector<T>()
// 指定初始大小，默認構造函數初始化大小為 0
vector<T>(size)
// 指定初始大小並指定初始值
vector<T>(size, init_value)
```

- Sample

```cpp
int main() {
    // 棧上變量
    vector<int> v1;
    print_vector(v1); // []

    // 堆上變量
    vector<int>* v2 = new vector<int>;
    print_vector(*v2); // []

    // 默認構造函數
    vector<int> v3;
    print_vector(v3); // []

    // 指定初始大小
    vector<int> v4(10);
    print_vector(v4); // [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]

    // 指定初始大小並指定初始值
    vector<int> v5(5, 2);
    print_vector(v5); // [2, 2, 2, 2, 2]
}
```

## Access 訪問元素(查、改)

聲明好向量之後，我們有四種方法可以訪問其中的元素：

### 訪問元素(查、改)相關語法

```cpp
// 指定偏移量，與數組相同
vec[index]
// 指定下標，與偏移量同義
vec.at(index)
// 查找第一個元素
vec.front()
// 查找最後一個元素
vec.back()
```

- Sample

```cpp
int main() {
    vector<int> v1(10);
    for(int i=0 ; i<10 ; i++) {
        v1[i] = i;
    }
    print_vector(v1); // [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

    cout << v1[2] << endl; // 2
    cout << v1.at(4) << endl; // 4
    cout << v1.front() << endl; // 0
    cout << v1.back() << endl; // 9
}
```

## Insert & Erase 增刪元素(增、刪)

我們可以透過 `[index]` 或 `at(index)` 訪問並修改元素之後，接下來介紹如何插入新的元素以及刪除現有元素：

### 增刪元素(增、刪)相關語法

```cpp
// 從後面插入元素
vec.push_back(value)
// 從後面彈出元素
vec.push_back(value)
// 插入元素到指定位置
vec.insert(position, value)
// 插入多個元素到指定位置
vec.insert(position, n, value)
// 刪除指定位置元素
vec.erase(position)
// 刪除指定範圍元素
vec.erase(position_start, position_end)
// 清空向量
vec.clear()

// 注意！position 必須是一個 iterator，如後面介紹的 begin、end 等
```

- Sample

```cpp
int main() {
    vector<int> v1;
    print_vector(v1); // []

    // 從後面插入元素
    for(int i=0 ; i<4 ; i++) {
        v1.push_back(i);
    }
    print_vector(v1); // [0, 1, 2, 3]

    // 從後面彈出元素
    v1.pop_back();
    print_vector(v1); // [0, 1, 2]

    // 插入元素到指定位置
    for(int i=0 ; i<3 ; i++) {
        v1.insert(v1.begin()+i, i+10);
    }
    print_vector(v1); // [10, 11, 12, 0, 1, 2]

    // 插入多個元素到指定位置
    v1.insert(v1.begin(), 3, 5);
    print_vector(v1); // [5, 5, 5, 10, 11, 12, 0, 1, 2]

    // 刪除指定位置元素
    v1.erase(v1.begin()+2);
    print_vector(v1); // [5, 5, 10, 11, 12, 0, 1, 2]

    // 刪除指定範圍元素
    v1.erase(v1.begin()+2, v1.begin()+5);
    print_vector(v1); // [5, 5, 0, 1, 2]

    // 清空向量
    v1.clear();
    print_vector(v1); // []
    cout << "size: " << v1.size() << ", capacity: " << v1.capacity() << endl; // size: 0, capacity: 16，可以看到原來擴展的內存依舊存在並沒有歸零
}
```

## Capacity 容量

以上基本上已經能夠完全操作向量內的元素了，接下來還缺少的就是訪問並且控制內存大小的函數，先看語法：

### 容量相關語法

```cpp
// 檢查向量是否為空，與 vec.size() == 0 等價但是效率更好
vec.empty()
// 查詢當前元素個數
vec.size()
// 查詢當前向量容量，即分配內存大小
vec.capacity()
// 重新分配 size
vec.resize(n)
// 重新分配 size 並且初始化
vec.assign(n, value)
// 確保分配足夠的內存，擴展 capacity
vec.reserve(n)
```

- Sample

```cpp
int main() {
    vector<int> v1(5, 3);
    print_vector(v1); // [3, 3, 3, 3, 3]
    // 查詢元素個數與分配內存大小
    cout << "size: " << v1.size() << ", capacity: " << v1.capacity() << endl; // size: 5, capacity: 5

    v1.push_back(-1);
    print_vector(v1); // [3, 3, 3, 3, 3, -1]
    cout << "size: " << v1.size() << ", capacity: " << v1.capacity() << endl; // size: 6, capacity: 10
    cout << "isEmpty: " << v1.empty() << endl; // 0
//    可以看到 size 指具體元素個數，而 capacity 則是當前數組容量，容量不足時通常擴展為兩倍大小

    // 重新分配 size
    v1.resize(2);
    print_vector(v1); // [3, 3]
    cout << "size: " << v1.size() << ", capacity: " << v1.capacity() << endl; // size: 2, capacity: 10
    cout << v1[5] << endl; // -1
//    resize 若小於當前 size 時，僅僅只會縮小 size，而 capacity 不變，並且也不會改變原來位置的值

    v1.resize(4);
    print_vector(v1); // [3, 3, 0, 0]
    cout << "size: " << v1.size() << ", capacity: " << v1.capacity() << endl; // size: 4, capacity: 10
    cout << v1[5] << endl; // -1
//    resize 大於當前 size 時，則會將多出來的空間初始化，其他位置一樣不變

    // 重新分配 size 並且初始化
    v1.assign(10, -2);
    print_vector(v1); // [-2, -2, -2, -2, -2, -2, -2, -2, -2, -2]
    cout << "size: " << v1.size() << ", capacity: " << v1.capacity() << endl; // size: 10, capacity: 10
//    與 resize 類似，第二個參數指定初始化值

    // 確保分配足夠的內存
    v1.reserve(15);
    print_vector(v1); // [-2, -2, -2, -2, -2, -2, -2, -2, -2, -2]
    cout << "size: " << v1.size() << ", capacity: " << v1.capacity() << endl; // size: 10, capacity: 15
    v1.reserve(5);
    cout << "size: " << v1.size() << ", capacity: " << v1.capacity() << endl; // size: 10, capacity: 15
    cout << v1[14] << endl; // 0
//    reserve 與 resize 不同，僅僅只是確保 capacity 的大小，比當前 capacity 大時會預先擴展
}
```

`reserve` 方法的好處在於，當你明確知道接下來要插入的元素個數時，就能夠預先分配內存，避免自動分配時分配多餘的內存：

```cpp
int main() {
    vector<int> v0(100);
    v0.push_back(0);
    cout << "size: " << v0.size() << ", capacity: " << v0.capacity() << endl; // size: 101, capacity: 200

    vector<int> v1(100);
    v1.reserve(101);
    v1.push_back(0);
    cout << "size: " << v1.size() << ", capacity: " << v1.capacity() << endl; // size: 101, capacity: 101
//    可以看到使用 reserve 後，size 沒有超出 capacity 因此不會自動擴展，大大的節省內存空間
}
```

## Iterator 迭代

最後的最後是有關迭代器，主要用途在於 `insert`、`erase` 指定位置(position)，以及 `algorithm` 庫中的函數都依賴迭代器來查找向量，以此便能夠忽略 STL 中不同數據結構的內存形式差異

### 獲取迭代器語法

```cpp
// 返回一個 iterator，指向第一個元素
vec.begin()
// 返回一個 iterator，指向最後一個元素
vec.end()
// 返回一個反向 iterator，指向最後一個元素（方向向前）
vec.rbegin()
// 返回一個反向 iterator，指向第一個元素（方向向前）
vec.rend()
```

- Sample

```cpp
int main() {
    vector<int> v1;
    for(int i=0 ; i<10 ; i++) {
        // 應用在 insert 中的 position
        v1.insert(v1.begin(), i);
    }
    print_vector(v1); // [9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
    // 應用在 algorithm 的 sort 函數
    sort(v1.begin(), v1.end());
    print_vector(v1); // [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
}
```

# 結語

本篇介紹了 C++ 的 STL 容器類中的 `vector 向量`，簡單來說就是一個強化版的數組，與 Java 中 ArrayList 的存在類似，作為實現數據結構的第一個例子非常合適，也是大部分應用最常用的數據結構之一。下一篇將要介紹與 vector 幾乎同等重要的數據結構：`map 鍵值對`。
