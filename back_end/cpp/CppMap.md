# Cpp 進階：Map 映射表

@[TOC](文章目錄)

<!-- TOC -->

- [Cpp 進階：Map 映射表](#cpp-進階map-映射表)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Import 導入庫](#import-導入庫)
  - [重要類型](#重要類型)
  - [元素操作](#元素操作)
    - [`map.insert` 插入(增)](#mapinsert-插入增)
    - [`map.find`、`map.lower_bound`、`map.upper_bound` 查找(查)](#mapfindmaplower_boundmapupper_bound-查找查)
    - [`map.erase`、`map.clear` 刪除元素(刪)](#maperasemapclear-刪除元素刪)
  - [迭代器](#迭代器)
  - [其他方法](#其他方法)
- [結語](#結語)

<!-- /TOC -->

## 簡介

上一篇：<a href="https://blog.csdn.net/weixin_44691608/article/details/107045275">Cpp 進階：Vector 向量</a>介紹了 C++ 的 STL 中 Vector 容器的使用方式。最基礎的數據結構中最常用的除了`序列(sequence)`就當數`映射表(map)`了（如 JS 中的 object、python 中的 dictionary），映射表是一個`鍵值對(key-vaule)`的集合，通常以`二叉樹(binary tree)`的結構來存儲和查詢，所以會涉及`鍵(key)`的比較(compare)和排序(sorting)，本篇就來介紹 STL 中 Map 數據結構的用法。

## 參考

<table>
  <tr>
    <td>C/C++ - Map (STL) 用法與心得完全攻略</td>
    <td><a href="https://mropengate.blogspot.com/2015/12/cc-map-stl.html">https://mropengate.blogspot.com/2015/12/cc-map-stl.html</a></td>
  </tr>
  <tr>
    <td>C++中的STL中map用法详解</td>
    <td><a href="https://www.cnblogs.com/fnlingnzb-learner/p/5833051.html">https://www.cnblogs.com/fnlingnzb-learner/p/5833051.html</a></td>
  </tr>
</table>

# 正文

與 vector 不同的是，map 的方法比較少，大多是透過`方法重載(overload)`的方式來提供不同的方法調用，方法名比較單一。以下列出本篇將要介紹的 map 相關類型和方法：

- 重要類型
  - 映射表類：`std::map<key_type, value_type>`
  - 鍵值對：`std::pair<key_type, value_type>`、`std::map<key_type, value_type>::value_type`
  - 迭代器：`std::map<key_type, value_type>::iterator`
- 元素操作
  - 插入：`map.insert`、`map[key] = value`
  - 查找：`map.find`、`map.lower_bound`、`map.upper_bound`
  - 刪除：`map.erase`
  - 清空：`map.clear`
- 迭代器
  - 正向迭代器：`map.begin`、`map.end`
  - 反向迭代器：`map.rbegin`、`map.rend`
  - 常量迭代器(const iterator 類型)：`map.cbegin`、`map.cend`
- 其他方法
  - 比較：`map.key_comp`、`map.value_comp`
  - 元素個數：`map.size`
  - 判斷是否為空：`map.empty`
  - 交換容器：`map.swap`

## Import 導入庫

```cpp
#include<map>
```


## 重要類型

map 中有幾個重要的數據類型：

- `std::map<key_type, value_type>`：映射表類，也是所有操作的核心類型(class)
- `std::pair<key_type, value_type>`、`std::map<key_type, value_type>::value_type`：鍵值對，兩個類型都用於插入元素時使用
- `std::map<key_type, value_type>::iterator`：在遍歷 map 數據結構時返回的迭代器類型，透過 +- 來向前後移動
- 構造函數：map 的構造函數涉及`內存分配器(allocator_type)`，不過一般簡單使用較少用到，這邊就不詳細解說

首先我們先使用 `typedef` 操作符定義一些接下來要用到的類型：

```cpp
#include <map>
#include <string>

using namespace std;

typedef map<int, string> MapIS;
typedef pair<int, string> PairIS;
typedef MapIS::value_type MapIS_VT;
typedef MapIS::iterator MapIS_IT;
typedef MapIS::reverse_iterator MapIS_RIT;
typedef MapIS::const_iterator MapIS_CIT;
```

以及一個查看 map 內容的`輸出運算符重載(operator<<)`

```cpp
ostream& operator<<(ostream& out, MapIS mis) {
    out << "map {";
    for(MapIS_IT iter = mis.begin(), last = --mis.end() ; iter != mis.end() ; iter++) {
        cout << " " << iter->first << "=" << iter->second << (iter == last ? " " : ",");
    }
    out << "}" << endl;
    return out;
}
```

## 元素操作

任何數據結構最重要的操作就是四種：增、刪、改、查。map 對每種操作提供了一道多種的方法供選擇

### `map.insert` 插入(增)

插入元素使用 `map.insert` 方法，可以傳入兩種參數，以及另一種類似數組的訪問方式，語法如下：

```cpp
// 使用 pair<key_type, value_type>(key, value) 插入元素
map.insert(pair<key_type, value_type>(key, value))
// 使用 map<key_type, value_type>::value_type(key, value) 插入元素
map.insert(map<key_type, value_type>::value_type(key, value))
// 使用 map[key] = value 插入元素
map[key] = value
```

- Sample

```cpp
// 類型定義
typedef map<int, string> MapIS;
typedef pair<int, string> PairIS;
typedef MapIS::value_type MapIS_VT;

int main() {
    MapIS m1;
    // 使用 pair<key_type, value_type>(key, value) 插入元素
    m1.insert(PairIS(1, "a"));
    m1.insert(PairIS(2, "b"));
    // 使用 map<key_type, value_type>::value_type(key, value) 插入元素
    m1.insert(MapIS_VT(3, "c"));
    m1.insert(MapIS_VT(4, "d"));
    // 使用 map[key] = value 插入元素
    m1[5] = "e";
    m1[6] = "f";

    cout << m1;
    // map { 1=a, 2=b, 3=c, 4=d }
}
```

### `map.find`、`map.lower_bound`、`map.upper_bound` 查找(查)

最主要的查找操作是 `map.find` 方法，`lower_bound`、`upper_bound` 則提供查找目標較為模糊時候的操作

- 語法

```cpp
// find 查找，鍵不存在時返回 map.end()
map.find(key)
// lower_bound 方法返回`大於等於`給定鍵的最小鍵
map.lower_bound(key)
// upper_bound 方法返回`大於`給定鍵的最小鍵
map.lower_bound(key)
```

- Sample

```cpp
int main() {
    MapIS m1;
    m1.insert(PairIS(1, "a"));
    m1.insert(PairIS(3, "c"));
    m1.insert(PairIS(5, "e"));

    // 三種查找方法都返回 map<key_type, value_type>::iterator 類型
    MapIS_IT iter;

    // 正常 find 方法
    iter = m1.find(3);
    cout << (iter == m1.end()) << endl; // 0
    cout << "pair( " << iter->first << "=" << iter->second << " )" << endl; // pair( 3=c )

    // 查找鍵不存在時返回 map.end()
    iter = m1.find(7);
    cout << (iter == m1.end()) << endl; // 1
    cout << "pair( " << iter->first << "=" << iter->second << " )" << endl; // pair( 1916449749= )

    // 查找 >= 3 的第一個鍵
    iter = m1.lower_bound(3);
    cout << "pair( " << iter->first << "=" << iter->second << " )" << endl; // pair( 3=c )

    // 查找 > 3 的第一個鍵
    iter = m1.upper_bound(3);
    cout << "pair( " << iter->first << "=" << iter->second << " )" << endl; // pair( 5=e )
}
```

### `map.erase`、`map.clear` 刪除元素(刪)

刪除單個元素(erase)，或是清空所有元素(clear)

- 語法

```cpp
// 刪除單一元素，傳入元素迭代器
map.erase(iterator)
// 清空映射表
map.clear()
```

- Sample

```cpp
int main() {
    MapIS mis;
    mis.insert(PairIS(1, "a"));
    mis.insert(PairIS(2, "b"));
    mis.insert(PairIS(3, "c"));
    mis.insert(PairIS(4, "d"));
    mis.insert(PairIS(5, "e"));
    cout << mis; // map { 1=a, 2=b, 3=c, 4=d, 5=e }

    MapIS_IT iter; // 查找元素返回迭代器
    MapIS_IT res; // 刪除元素返回迭代器

    // 查找(find)然後刪除(erase)元素，如果刪除 map.end() 會拋出異常
    iter = mis.find(3);
    res = mis.erase(iter);
    cout << res->first << "=" << res->second << endl; // 4=d
    cout << (res == mis.find(4)) << endl; // 1
    // 刪除後返回下一個元素的 iterator

    cout << mis; // map { 1=a, 2=b, 4=d, 5=e }

    // 清空映射表
    mis.clear();
    cout << mis; // map {}
}
```

## 迭代器

數據結構一個非常重要的思想就是透過迭代器來訪問元素，可以將內部數據結構與接口分離解耦。可以看到上面 `find`、`erase` 等方法返回的都是元素的迭代器，除此之外我們還可以使用 `map.begin`、`map.end`、`map.rbegin`、`map.rend`、`map.cbegin`、`map.cend` 等方法返回不同種的迭代器（`begin` 表示起始位置、`end` 為結束位置、`r-` 為反向迭代器、`c-` 為常量迭代器）。

而迭代器類型的 `first`、`second` 分別表示`鍵(key)`和`值(value)`

- 遍歷迭代器

```cpp
typedef MapIS::iterator MapIS_IT; // 對應 begin、end
typedef MapIS::reverse_iterator MapIS_RIT; // 對應 rbegin、rend
typedef MapIS::const_iterator MapIS_CIT; // 對應 cbegin、cend

int main() {
    MapIS mis;
    mis.insert(PairIS(1, "a"));
    mis.insert(PairIS(2, "b"));
    mis.insert(PairIS(3, "c"));
    mis.insert(PairIS(4, "d"));
    mis.insert(PairIS(5, "e"));
    cout << mis; // map { 1=a, 2=b, 3=c, 4=d, 5=e }

    // 一般迭代器
    for(MapIS_IT it = mis.begin() ; it != mis.end() ; it++) {
        cout << it->first << "=" << it->second << endl;
    }
    // 1=a
    // 2=b
    // 3=c
    // 4=d
    // 5=e

    // 反向迭代器
    for(MapIS_RIT rit = mis.rbegin() ; rit != mis.rend() ; rit++) {
        cout << rit->first << "=" << rit->second << endl;
    }
    // 5=e
    // 4=d
    // 3=c
    // 2=b
    // 1=a

    // 常量迭代器，差別在於不可改變當前鍵值對
    for(MapIS_CIT cit = mis.cbegin() ; cit != mis.cend() ; cit++) {
        cout << cit->first << "=" << cit->second << endl;
    }
    // 1=a
    // 2=b
    // 3=c
    // 4=d
    // 5=e
}
```

## 其他方法

其他方法比較簡單就不用代碼演示了

- 語法

```cpp
// 比較鍵或值，返回一個比較器
map.key_comp()
map.value_comp()
// 返回元素個數
map.size()
// 判斷是否為空，與 map.size() == 0 等價但效率較好
map.empty()
// 交換兩個 map 的所有元素
map.swap(other_map)
```

# 結語

本篇介紹了與 vector 幾乎相等重要的容器類：`map 映射表`。與一般數組不同的是，它背後的抽象數據結構是一顆二叉樹，便能夠透過維持一定的排序來保證增刪改查的效率，供大家參考。
