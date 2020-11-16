# Cpp 进阶：Smart Pointer 智能指针

@[TOC](文章目录)

<!-- TOC -->

- [Cpp 进阶：Smart Pointer 智能指针](#cpp-进阶smart-pointer-智能指针)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [核心目的](#核心目的)
  - [功能需求](#功能需求)
  - [C++ 实现](#c-实现)
    - [智能指针(包装类)](#智能指针包装类)
    - [全局工厂包装函数](#全局工厂包装函数)
    - [测试用对象类](#测试用对象类)
    - [测试代码](#测试代码)
- [结语](#结语)

<!-- /TOC -->

## 简介

我们都知道 Java 和 C++ 一样，运行时会同时维护一个`堆(Heap)空间`与`栈(Stack)空间`。通常局部变量、函数参数等都会存放在栈空间，跟着函数运行周期一起消亡；而 Java 和 C++ 对于堆空间的控制则不太相同：

- 对于 Java **“万物皆对象”**的设计理念，程序以`对象(Object)`为运行单元，所有对象都保存在堆空间，并透过 JVM 的`垃圾回收(GC = Garbage Collection)`机制自动管理所有对象的存亡

- 对于 C++ 来说，一切的内存管理都交由程序员管理，所以同样创建一个对象我们可以看到下面两种不同的写法：

```cpp
// 在栈上创建对象，并返回对象（触发拷贝构造函数）
Object createObject() {
    Object object;
    return object;
}

// 在堆上存放对象，并返回对象指针
Object *createObject() {
    Object *op = new Object;
    return op;
}
```

第一种写法会在退出函数时自动释放栈空间上对象占用的内存空间，然而第二种写法的对象生存在堆上，因此需要程序员主动释放空间。

为了避免程序员忘记释放指针的内存，我们可以使用一个`智能指针(Smart Pointer)`用于包装一般的指针，使得作为局部变量的智能指针在回收时自动释放它所指向的堆上的对象所占用的堆空间。

## 参考

<table>
  <tr>
    <td>指针运算符(*,->)重载</td>
    <td><a href="https://www.kancloud.cn/cpp-jdxia/cpp/1408812">https://www.kancloud.cn/cpp-jdxia/cpp/1408812</a></td>
  </tr>
  <tr>
    <td>c++中堆、栈内存分配</td>
    <td><a href="https://www.cnblogs.com/yyxt/p/4268304.html">https://www.cnblogs.com/yyxt/p/4268304.html</a></td>
  </tr>
</table>

# 正文

## 核心目的

我们在上面简介中已经简述了智能指针的使用场景，使用智能指针的核心价值在于`自动释放堆上对象`，使我们在 C++ 中能像 Java 一样自动回收堆上的对象一般，无需在每个函数的尾部显式释放对象，做到实现堆上对象内存空间的自动管理。

## 功能需求

为了实现自动释放堆上对象的功能，我们透过定义一个`包装类(wrapper)`来代理我们想要自动回收的指针，代理的原理在于创建一个`栈上的局部变量(智能指针)`来承接我们想要自动管理的指针，同时能够`代理(proxy)`访问对象的操作，最后在释放栈上的智能指针时同时`释放堆上的对象`

- 主要功能
  - 保存要自动管理的对象的指针
  - 代理访问对象的操作
  - 自动释放堆上对象

同时我们还实现两个辅助功能：

- 作为普通函数返回包装类的全局工厂函数
- 使智能指针具备传递指针的能力(拷贝构造函数)

## C++ 实现

所有实现都统一写在 `main.cpp` 中间，若想要单独分离到头文件内部，注意由于智能指针实现使用了`模版类(template)`需要在`头文件(header)`内声明并定义完全

### 智能指针(包装类)

```cpp
// 智能指针
template<class T>
class SmartPointer {
public:
    // 构造函数接受要管理的指针
    SmartPointer(T *pointer) : pointer(pointer) {}

    // 拷贝构造函数保存堆上对象的拷贝
    SmartPointer(const SmartPointer<T> &other) : pointer(new T(*other.pointer)) {}

    // 在析构函数中释放指针指向的堆上对象
    ~SmartPointer() {
        if (pointer != nullptr) {
            delete pointer;
        }
    }

    // 用于代理 -> 操作符
    T *operator->() {
        return pointer;
    }

    // 用于代理 * 操作符
    T &operator*() {
        return *pointer;
    }

    // 重载 << 操作符
    friend ostream &operator<<(ostream &os, SmartPointer<T> &smartPointer) {
        if (smartPointer.pointer != nullptr) {
            os << *smartPointer.pointer;
        } else {
            os << smartPointer.pointer;
        }
        return os;
    }

private:
    // 被自动管理的指针
    T *pointer;
};
```

### 全局工厂包装函数

```cpp
templac
template<class T>
SmartPointer<T> wrapper(T *op) {
    SmartPointer<T> p(op);
    return p;
}
```

### 测试用对象类

```cpp
int global_id = 0;

class Object {
public:
    Object() : id(global_id++) {
        cout << "Constructor(" << id << ")" << endl;
    }

    Object(const Object &object) : id(object.id + 10) {
        cout << "Copy Constructor(" << id << ")" << endl;
    }

    ~Object() {
        cout << "Destructor(" << id << ")" << endl;
    }

    friend ostream &operator<<(ostream &os, Object &object) {
        os << "Object(" << object.id << ")";
        return os;
    }

private:
    int id;
};

class A {
public:
    A() : a(global_id++) {}

    A(int a) : a(a) {}

    int getA() {
        return a;
    }

    friend ostream &operator<<(ostream &os, A &a) {
        cout << "A(" << a.a << ")";
        return os;
    }

private:
    int a;
};

class B {
public:
    B() : b(global_id++) {}

    B(int b) : b(b) {}

    int getB() {
        return b;
    }

    bool setB(int b) {
        this->b = b;
        return true;
    }

    friend ostream &operator<<(ostream &os, B &b) {
        cout << "B(" << b.b << ")";
        return os;
    }

private:
    int b;
};
```

### 测试代码

```cpp
int main() {
    SmartPointer<Object> p = new Object;            // Constructor(0)
    cout << "p: " << p << endl;                     // p: Object(0)
    cout << endl;
    /* 说明：
        new Object 使用默认构造函数，id 为 0
        智能指针 p 代理 Object(0)
    */

    SmartPointer<Object> p1(p);                     // Copy Constructor(10)
    cout << "p1: " << p1 << endl;                   // p1: Object(10)
    cout << endl;
    /* 说明：
        p1(p) 使用了传递指针的特性，触发 Object 的拷贝构造函数(id + 10)并由 p1 进行管理
        此时 p1 管理的对象为 Object(10)
    */

    SmartPointer<Object> p2 = wrapper(new Object);  // Constructor(1)
    cout << "p2: " << p2 << endl;                   // p2: Object(1)
    cout << endl;
    /* 说明：
        使用工厂函数 wrapper 创建智能指针，new Object 使用默认构造函数，id 为 1
        p2 管理的对象为 Object(1)
    */

    SmartPointer<A> pa(new A(100));
    cout << "pa: " << pa << endl;                   // pa: A(100)
    cout << "new A.getA(): " << pa->getA() << endl; // new A.getA(): 100
    cout << endl;
    /* 说明：
        pa 管理类 A 的实例，即 A(100)
        pa->getA() 模拟智能指针代理访问类的操作，可以是成员函数调用(getA)或是成员变量的访问
    */

    SmartPointer<B> pb = new B(200);
    cout << "pb: " << pb << endl;                   // pb: B(200)
    cout << "new B.getB(): " << pb->getB() << endl; // new B.getB(): 200
    cout << endl;
    /* 说明：
        pb 的使用与 pa 类似
        这边主要说明透过使用模版类型(template)，以及 `->`、`*` 操作符的重载，即可实现对象访问操作的代理(proxy)
    */

    SmartPointer<B> pc = pb;
    cout << "pb: " << pb << endl;                   // pb: B(200)
    cout << "pc: " << pc << endl;                   // pc: B(200)
    pc->setB(pc->getB() + 100);
    cout << "pb: " << pb << endl;                   // pb: B(200)
    cout << "pc: " << pc << endl;                   // pc: B(300)
    cout << endl;
    /* 说明：
        这边同样使用传递指针的效果，由于使用了被代理对象的构造函数作为新的智能指针所代理的对象
        这边透过 setB 可以看到 pb 和 pc 所代理的对象是不同的
    */

    // Destructor(1)
    // Destructor(10)
    // Destructor(0)
    /* 说明：
        自动释放三个创建过的 Object，id 分别是 1, 10, 0
    */
    return 0;
}
```

# 结语

作为 Java 入门的程序猿回头写 C++ 总是觉得绑手绑脚的，报过几次内存溢出才比较感受得到释放内存的重要性(是被 gc 宠坏的孩子呢)。不过使用 C++ 释放更多决定权给程序员，除了考验细心程度和对于对象管理的意识，也能更好的理解堆栈机制的内存管理，对于程序运行时的掌控也更上一层楼。
