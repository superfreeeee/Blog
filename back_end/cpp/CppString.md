# Cpp 進階：String 字符串

@[TOC](文章目錄)

<!-- TOC -->

- [Cpp 進階：String 字符串](#cpp-進階string-字符串)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Overview 總覽](#overview-總覽)
  - [Import 引入](#import-引入)
  - [構造函數](#構造函數)
  - [擷取單個字符或子串(查)](#擷取單個字符或子串查)
  - [修改字符串(增、刪、改)](#修改字符串增刪改)
  - [其他屬性](#其他屬性)
- [結語](#結語)

<!-- /TOC -->

## 簡介

原來 C 中使用字符串時，是一個以 `\0` 結尾的字符序列(`char[]`)，而 `string` 庫提供了一個字符串類，提供一些基本的字符串操作手段，將程序員從重複的方法構造中解放出來，也是大部分程序最常用到的類沒有之一。接下來就來看看到底該如何使用這個類吧。

## 參考

<table>
  <tr>
    <td>C/C++ - String 用法與心得完全攻略</td>
    <td><a href="https://mropengate.blogspot.com/2015/07/cc-string-stl.html">https://mropengate.blogspot.com/2015/07/cc-string-stl.html</a></td>
  </tr>
</table>

# 正文

## Overview 總覽

`string` 類對許多操作符都進行重載，像是 `assign` 可以用 `=`、`append` 可以用 `+`、`compare` 可以用 `< <= > >=` 替代，在使用時可以避免需要去既很多的方法名直接使用更直白的操作符。以下列出本篇幾介紹的所有操作：

- 構造方法
- 擷取單個字符或子串
  - 偏移量：`s[]`、`s.at`
  - 擷取子串：`s.substr`
  - 查找子串：`s.find`
- 修改字符串
  - 賦值：`=`、`s.assign`
  - 連接（添加）：`+`、`s.append`
  - 插入：`s.insert`
  - 刪除：`s.erase`
- 其他屬性
  - 字符串長度：`s.size`、`s.length`
  - 判斷空串：`s.empty`
  - 比較：`< <= > >= ==`、`compare`

## Import 引入

```cpp
#include<string>
```

## 構造函數

首先先來介紹如何創建一個`字符串(string)`類，我們可以透過*直接量*或是*其他字符串*來創建一個字符串：

```cpp
int main() {
    // 棧上變量
   string s1;
   cout << "s1: " << s1 << endl; // s1:

   // 傳入字符串直接量
   string s2("string 2");
   cout << "s2: " << s2 << endl; // s2: string 2

   // 傳入另一個 string
   string s3(s2);
   cout << "s3: " << s3 << endl; // s3: string 2

   // 使用直接量初始化
   string s4 = "string 4";
   cout << "s4: " << s4 << endl; // s3: string 4
}
```

## 擷取單個字符或子串(查)

接下來介紹查找字符串的方法，我們可以查找字串位置或是查找指定位置的子串

- 語法

```cpp
// 偏移量，查找指定位置字符
s[index]
s.at(index)
// 擷取子串，start、n 為可選，表示`起始位置`和`長度`
s.substr(start, n)
// 查找子串，返回子串第一次出現的位置；start 可選，表示查找起始位置
s.find(str, start)
```

- Sample

```cpp
int main() {
    string s = "abcdefg";
    cout << s << endl; // abcdefg

    // 使用偏移量，UTF-8 編碼時需注意
    cout << s[1] << endl; // b

    // 使用 at 方法
    cout << s.at(2) << endl; // c

    // 擷取子串
    cout << s.substr(3, 3) << endl; // def

    // 查找子串，不存在時返回 -1
    cout << s.find("e") << endl; // 4
    cout << (int)s.find("e", 5)<< endl; // -1
}
```

## 修改字符串(增、刪、改)

修改字符串的方法可以分為四類：重新賦值、插入、連接、刪除

- 語法

```cpp
// 連接字符串，start、n 為可選，分別表示`起始位置`和`長度`
s1 + s2
s1.append(s2, start, n) // 等價於 s1 += s2.substr(start, end)
// 重新賦值，同樣 start、n 為可選
s1 = s2
s1.assign(s2, start, n) // 等價於 s1 = s2.substr(start, end)
// 插入字符串，pos 表示插入位置、n 為字符重複次數、str 為字符串、c 為單個字符
s.insert(pos, str)
s.insert(pos, n, c)
// 刪除子串，pos 表示起始位置、n 為刪除長度
s.erase(pos, n)
```

- Sample

```cpp
int main() {
    string s;

    // 直接量賦值
    s = "___";
    cout << s << endl; // ___

    // 連接字符串
    s += "__";
    cout << s << endl; // _____

    // 插入字符串
    s.insert(1, "2");
    cout << s << endl; // _2____

    // 插入重複字符，第二個參數為次數
    s.insert(3,3, '4');
    cout << s << endl; // _2_444___

    // 重新賦值，與 s = string("abcde").substr(2, 4) 等價
    s.assign("abcde", 2, 4);
    cout << s << endl; // cde

    // 於結尾連接字符串，與 s += string("abcde").substr(2, 4) 等價
    s.append("abcde", 2, 4);
    cout << s << endl; // cdecde

    // 刪除子串
    s.erase(1, 4);
    cout << s << endl; // ce
}
```

## 其他屬性

省下的諸如長度、比較、判斷空串等方法

- 語法

```cpp
// 長度
s.size()
s.length()
// 判斷空串
s.empty()
// 比較
s1 ( == | < | <= | > | >= ) s2
s1.compare(s2)
```

- Sample

```cpp
int main() {
    string s1 = "12345";
    string s2 = "67890";
    string s3 = "12345";

    // 字符串長度
    cout << s1.size() << endl; // 5
    cout << s1.length() << endl; // 5

    // 判斷字符串是否為空
    cout << s1.empty() << endl; // 0

    // 比較字符串內容
    cout << (s1 == s2) << endl; // 0
    cout << (s1 == s3) << endl; // 1

    cout << (s1 < s2) << endl; // 1
    cout << s1.compare(s2) << endl; // -5
}
```

# 結語

看起來好像很多複雜的操作，其實都離不開增刪改查，不過是提供了各種形式的方法選用而已，供大家參考。
