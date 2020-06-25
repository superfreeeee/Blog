# Cpp 基礎：Constructor 各類構造函數

@[TOC](文章目錄)

<!-- TOC -->

- [Cpp 基礎：Constructor 各類構造函數](#cpp-基礎constructor-各類構造函數)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [1. 無參數構造函數](#1-無參數構造函數)
  - [2. 有參數構造函數（配合初始化列表）](#2-有參數構造函數配合初始化列表)
  - [3. 拷貝構造函數](#3-拷貝構造函數)
  - [4. 賦值運算符重載（對象賦值）](#4-賦值運算符重載對象賦值)
  - [5. 析構函數](#5-析構函數)
  - [6. 輸出運算符重載（輸出對象）](#6-輸出運算符重載輸出對象)
- [結語](#結語)

<!-- /TOC -->

## 簡介

C++ 相較於 Java 來說，對`對象(object)`的控管能力有更上一層樓了，有基本的`構造函數(Constructor)`、`析構函數(Destructor)`，並且提供 `new`、`delete` 關鍵字來管理堆上對象的存活，這比起 Java 的`垃圾回收機制(GC)`更為可靠，內存回收的時間點更為可控。接下來我們就來講解各式各樣的構造函數吧。

## 參考

<table>
  <tr>
    <td>c++:(各种)构造函数的调用方式</td>
    <td><a href="https://blog.csdn.net/VitaLemon__/article/details/60869721">https://blog.csdn.net/VitaLemon__/article/details/60869721</a></td>
  </tr>
  <tr>
    <td>C++ 类 & 对象-菜鸟教程</td>
    <td><a href="https://www.runoob.com/cplusplus/cpp-classes-objects.html">https://www.runoob.com/cplusplus/cpp-classes-objects.html</a></td>
  </tr>
</table>

# 正文

我們先列出本篇將要介紹的所有將會用到的構造函數和運算符重載：

```cpp
class Demo {
public:
    // 無參數構造函數
    Demo();
    // 有參數構造函數
    Demo(int);
    // 拷貝構造函數
    Demo(const Demo&);
    // 賦值運算符重載
    void operator=(const Demo&);
    // 析構函數
    ~Demo();
    // 輸出運算符重載
    friend ostream& operator<<(ostream&, const Demo&);

private:
    int _id;
};
```

接下來我們將一個一個展示使用的時機

## 1. 無參數構造函數

首先是無參數構造函數，分別是存活在`棧(Stack)`上和存活在`堆(Heap)`上的的對象

```cpp
Demo::Demo() {
  cout << "no-param constructor(" << _id << ")" << endl;
}
```

```cpp
int main() {
// 存活在棧上的對象，main 函數結束後調用析構函數並釋放內存
  Demo demo;
  // Demo demo()
  // 棧上調用無參數構造函數不可寫上圓括號

// 存活在堆上的對象，需要主動調用 delete 釋放內存
  Demo* demo2 = new Demo;
  Demo* demo3 = new Demo();
}
```

```
output:
no-param constructor(0)
no-param constructor(0)
no-param constructor(0)
destructor(0)
```

- 說明：我們創建了三個 Demo 對象，只有第一個棧上存活的對象在 main 函數出棧後會被回收

需要如下主動調用 `delete` 關鍵字

```cpp
delete demo2
delete demo3
```

## 2. 有參數構造函數（配合初始化列表）

第二種是調用有參數構造函數，這時候圓括號是必須的

```cpp
Demo::Demo(int id): _id(id) {
  cout << "param constructor(" << _id << ")" << endl;
}
```

```cpp
int main() {
// 棧上的對象
  Demo demo(1);

// 堆上的對象
  Demo* demo2 = new Demo(2);
  // 主動釋放對象內存
  delete demo2;
}
```

```cpp
output:
param constructor(1)
param constructor(2)
destructor(2)
destructor(1)
```

## 3. 拷貝構造函數

第三個是拷貝構造函數，在初始化對象的時候傳如一個對象作為範本

```cpp
Demo::Demo(const Demo& other) {
  this->_id = other._id;
  cout << "copy-constructor(" << _id << ")" << endl;
}
```

```cpp
int main() {
  // 原對象
  Demo demo(1);
  Demo demo2(2);

// 調用拷貝構造函數
  Demo demo3(demo);
// 初始化表達式，與上面等價
  Demo demo4 = demo2;
}
```

```
output:
param constructor(1)
param constructor(2)
copy-constructor(1)
copy-constructor(2)
destructor(2)
destructor(1)
destructor(2)
destructor(1)
```

- 說明：這邊需要釐清一件事，`Demo demo4 = demo2;` 是初始化表達式而不是一般的賦值表達式，所以調用的是`拷貝構造函數`，下面才是使用賦值表達式

## 4. 賦值運算符重載（對象賦值）

首先我們要先釐清`初始化表達式`和`賦值表達式`的差異

```cpp
// 初始化表達式
int i = 0;

// 賦值表達式
int i;
i = 1;
```

這樣非常容易搞混的，因此 cpp 提供了一種類似構造函數的表達式，能夠避免誤會

```cpp
int i(100);
// 等價於
int i = 100;
```

這在類對象賦值也是一樣的，先上方法定義

```cpp
void Demo::operator=(const Demo& other) {
  this->_id = other._id;
  cout << "Operator overload =: Demo(" << _id << ")" << endl;
}
```

```cpp
int main() {
// 對象初始化
  Demo demo(1);
  Demo demo2(2);
// 賦值表達式
  demo2 = demo;
}
```

```cpp
output:
param constructor(1)
param constructor(2)
Operator overload =: Demo(1)
destructor(1)
destructor(1)
```

## 5. 析構函數

`析構函數(destructor)`是在回收內存之前會調用的函數，前面已經在輸出的部份看到析構函數的輸出範例了，先看方法定義

```cpp
Demo::~Demo() {
  cout << "destructor(" << _id << ")" << endl;
}
```

```cpp
int main() {
// 棧上對象
  Demo demo(1);
// 堆上對象需要主動調用 delete 釋放內存
  Demo* demo2 = new Demo(2);
  delete demo2;
}
```

```
output:
param constructor(1)
param constructor(2)
destructor(2)
destructor(1)
```

## 6. 輸出運算符重載（輸出對象）

最後稍微展示一下自定義類的輸出運算符重載

```cpp
ostream& operator<<(ostream& out, const Demo& other) {
  out << "Operator overload <<: Demo(" << other._id << ")" << endl;
  return out;
}
```

```cpp
int main() {
  Demo demo(0);
  cout << demo;
}
```

```
output:
param constructor(0)
Operator overload <<: Demo(0)
destructor(0)
```

# 結語

本篇總共講解了四種構造函數：`無參數構造函數`、`有參數構造函數`、`拷貝構造函數`、`析構函數`，並且演示了`拷貝構造函數(copy-constructor)`和`賦值表達式(assign expression)`的差異，希望大家對於 C++ 中對象的掌控更上一層樓。
