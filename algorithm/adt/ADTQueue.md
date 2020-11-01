# ADT: Queue 队列

@[TOC](文章目录)

<!-- TOC -->

- [ADT: Queue 队列](#adt-queue-队列)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [队列结构](#队列结构)
  - [抽象接口](#抽象接口)
  - [实现要素](#实现要素)
  - [Java 实现(使用范型)](#java-实现使用范型)
  - [C++ 实现(使用模版类)](#c-实现使用模版类)
- [结语](#结语)

<!-- /TOC -->

## 简介

上一篇<a href="https://blog.csdn.net/weixin_44691608/article/details/109400531">ADT: Stack 栈</a>介绍了`Stack(栈)`的数据结构，这一篇来介绍与栈相似的`Queue(队列)`的数据结构，与栈相同的是都能够向其`添加(push, enqueue)`和`取出(pop, dequeue)`元素；不同的是栈的出入都从容器的同一边(LIFO)，而队列则是两边开口，一边进一边出，接下来就让我们来看看队列是怎么实现的吧。

## 参考

<table>
  <tr>
    <td></td>
    <td><a href=""></a></td>
  </tr>
</table>

# 正文

## 队列结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/queue.jpg)

队列是一个拥有两个开口的容器，一边作为输入(enqueue)端，一边为输出(dequeue)端，属于具有特殊结构的线性表，限定只能从队尾加入元素，从队首取出元素。

## 抽象接口

队列的抽象数据结构对外开放的接口有五个：

```
ENQUEUE(x)
入队，将元素 x 从队尾加入到队列中

DEQUEUE()
出队，将队首元素弹出并返回

FRONT() / getFirst()
取队首，获取队首元素，也就是当前队列下一个可以出队的元素

BACK() / getLast()
取队尾，获取队尾元素，也就是最近一次入队的元素

EMPTY()
检查队列是否为空，即队列中是否存在任意元素
```

## 实现要素

队列也能够分别使用数组和链表来实现，而通常我们偏好使用环形队列(环状数组or环状链表)，指定特定长度的容器后，使指针在环内循环，本篇介绍使用环形数组实现的队列

数组实现的队列需要一下要素：
1. 保存元素的数组 `T[] Q`
2. 记录当前数组的大小 `int size`
3. 记录队首位置 `int front`
4. 记录队尾位置 `int rear`
5. 标记队列是否为满 `boolean full` (不使用则大小为 n 的 Q 只能保存 n - 1 个元素)

## Java 实现(使用范型)

- `Queue.java`

```java
/**
 * 队列
 * @param <T>
 */
public interface Queue<T> {

    /**
     * 入队
     * @param t
     */
    void enqueue(T t);

    /**
     * 出队
     * @return
     */
    T dequeue();

    /**
     * 获取队首元素
     * @return
     */
    T getFirst();

    /**
     * 获取队尾元素
     * @return
     */
    T getLast();

    /**
     * 判断队列是否为空
     * @return
     */
    boolean empty();
}

```

- `QueueCircular.java`

```java
public class QueueCircular<T> implements Queue<T> {

    public QueueCircular() {
        this(10);
    }

    public QueueCircular(int size) {
        this.Q = (T[]) new Object[size];
        this.front = this.rear = 0;
        this.full = false;
    }

    private T[] Q;
    private int front;
    private int rear;
    private boolean full;

    /**
     * 入队
     *
     * @param t
     */
    public void enqueue(T t) {
        if (full) return;
        Q[rear] = t;
        rear = (rear + 1) % Q.length;
        if (rear == front) full = true;
    }

    /**
     * 出队
     *
     * @return
     */
    public T dequeue() {
        if (front == rear && !full) return null;
        if (full) full = false;
        T res = Q[front];
        front = (front + 1) % Q.length;
        return res;
    }

    /**
     * 获取队首元素
     *
     * @return
     */
    public T getFirst() {
        return empty() ? null : Q[front];
    }

    /**
     * 获取队尾元素
     *
     * @return
     */
    public T getLast() {
        return empty() ? null : Q[(rear - 1 + Q.length) % Q.length];
    }

    /**
     * 判断队列是否为空
     *
     * @return
     */
    public boolean empty() {
        return front == rear && !full;
    }
}
```

- `QueueTest.java`

```java

public class QueueTest {
    @Test
    public void test_queue_circular() {
        Queue<Integer> queue = new QueueCircular<Integer>();
        // 入队 11 个数
        for (int i = 1; i < 22; i += 2) {
            queue.enqueue(i);
        }
        // 出队 11 个数，检查是否符合 FIFO 顺序
        for (int i = 1; i < 20; i += 2) {
            assertEquals((Integer) i, queue.dequeue());
        }
        // 查看指针问题
        for (int i = 0; i < 10; i++) {
            queue.enqueue(i);
            assertEquals((Integer) i, queue.dequeue());
        }
        for (int i = 1; i < 20; i += 2) queue.enqueue(i);
        // 查看临界条件下的出入栈
        for (int i = 1; i < 20; i += 2) {
            assertEquals((Integer) i, queue.dequeue());
            queue.enqueue(i);
        }
    }
}
```

## C++ 实现(使用模版类)

- `queue.h`

```cpp
template<class T>
class Queue {
public:
    Queue();
    Queue(int);
    ~Queue();
    void enqueue(T);
    T dequeue();
    T getFirst();
    T getLast();
    bool empty();

private:
    T *Q;
    int size;
    int front;
    int rear;
    bool full;
};

template<class T>
Queue<T>::Queue(): Q(new T[10]), size(10), front(0), rear(0), full(false) {}

template<class T>
Queue<T>::Queue(int size) : Q(new T[size]), size(size), front(0), rear(0), full(false) {}

template<class T>
Queue<T>::~Queue() {
    delete[] Q;
}

template<class T>
void Queue<T>::enqueue(T t) {
    if (full) return;
    Q[rear] = t;
    rear = (rear + 1) % size;
    if (rear == front) full = true;
}

template<class T>
T Queue<T>::dequeue() {
    if (front == rear && !full) return 0;
    if (full) full = false;
    T res = Q[front];
    front = (front + 1) % size;
    return res;
}

template<class T>
T Queue<T>::getFirst() {
    return empty() ? 0 : Q[front];
}

template<class T>
T Queue<T>::getLast() {
    return empty() ? 0 : Q[(rear - 1 + size) % size];
}

template<class T>
bool Queue<T>::empty() {
    return front == rear && !full;
}
```

- `myassert.h`

```cpp
#include <iostream>

template<class T>
void assertEquals(T expected, T actual) {
    if (expected != actual) {
        cout << "AssertEquals Error:" << endl;
        cout << "\texpected: " << expected << endl;
        cout << "\tactual: " << actual << endl;
        throw "";
    }
}
```

- `queue_test.cpp`

```cpp
#include "queue_test.h"
#include "queue.h"
#include "myassert.h"

void test_queue_circular() {

    Queue<int> Q;
    for (int i = 1; i < 22; i += 2) {
        Q.enqueue(i);
    }

    // 出队 11 个数，检查是否符合 FIFO 顺序
    for (int i = 1; i < 20; i += 2) {
        assertEquals(i, Q.dequeue());
    }

    // 查看指针问题
    for (int i = 0; i < 10; i++) {
        Q.enqueue(i);
        assertEquals(i, Q.dequeue());
    }

    for (int i = 1; i < 20; i += 2) Q.enqueue(i);
    // 查看临界条件下的出入栈
    for (int i = 1; i < 20; i += 2) {
        assertEquals(i, Q.dequeue());
        Q.enqueue(i);
    }
    cout << "test queue circular success" << endl;
}
```

# 结语

队列跟栈非常类似，不过栈是`后进先出(LIFO)`的结构，而队列则是`先进先出(FIFO)`。借由这个 FIFO 的特性，我们能够在一些场景维护一个队列来维持按序处理任务的算法模型，如`树的按层遍历`、`进程调度就绪队列`等应用场景。