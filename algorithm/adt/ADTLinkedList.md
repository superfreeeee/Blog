# ADT: LinkedList 链表

@[TOC](文章目录)

<!-- TOC -->

- [ADT: LinkedList 链表](#adt-linkedlist-链表)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [链表结构](#链表结构)
  - [抽象接口](#抽象接口)
  - [实现要素](#实现要素)
    - [单向链表](#单向链表)
    - [双向链表](#双向链表)
  - [Java 实现](#java-实现)
    - [链表接口](#链表接口)
    - [单向链表](#单向链表-1)
    - [双向链表](#双向链表-1)
- [结语](#结语)

<!-- /TOC -->

## 简介

几乎所有高级语言都提供`数组(Array)`作为基础数据结构，从存储的角度来看数组就是一个连续的存储单元，当时当我们并不能在创建时就知道未来需要用到多少空间，同时我们也不想预先分配空间而造成浪费，这时候我们就可以使用`链表(LinkedList)`来实现线性表，每一个数据块都附带一个指向下一个数据的指针，这样使得各个数据放置的位置更加灵活且允许不连续的存储。

## 参考

<table>
  <tr>
    <td></td>
    <td><a href=""></a></td>
  </tr>
</table>

# 正文

## 链表结构

链表与一般数组不同的地方在于，各个元素的排列不是根据固定偏移量连续存储，而是在数据块中保存指向下一个节点的指针(或是句柄)，透过这种形式允许数据不连续存储且按需分配空间。

- 链表结构图示

![](https://picures.oss-cn-beijing.aliyuncs.com/img/linkedlist.png)

## 抽象接口

链表相当于一个动态数组，原来数组能够直接访问指定位置的数据块，而链表则需要抽象出访问的接口(get、add、remove)来取代，将查找数据块的操作封装，以下为链表的抽象接口，也是一般线性表的抽象操作接口：

```
GET(index)
查找指定位置(index)的数据块(元素)

ADD(item)
在链表尾部添加元素(item)

ADD(index, item)
在指定位置(index)添加元素(item)

REMOVE(index)
删除指定位置(index)的元素

SIZE()
返回当前链表的大小

EMPTY()
检查当前链表是否为空
```

## 实现要素

本篇介绍单向链表和双向链表的实现结构：

### 单向链表

单向链表的节点定义如下：

```
Single-Node
    T data
    Single-Node next # 指向下一个节点
```

而在单向链表中只需要保存链表的头节点，其余节点都能够透过 `next` 属性访问得到：
1. 链表头节点 `SingleNode head`

### 双向链表

双向链表的节点定义：

```
Double-Node
    T data
    Double-Node prev # 指向上一个节点
    Double-Node next # 指向下一个节点
```

双向链表的实现要素如下：
1. 链表头节点 `DoubleNode head`
2. 链表尾节点 `DoubleNode tail`

由于单向链表要访问中间元素需要依序访问 next 一步步找到节点，双向链表透过在每个节点记录前一个节点的指针，并且保存链表尾节点的指针，使得查找位置靠近链表后半部的时候能从尾部向前查找。

在稍稍增加每次 ADD、REMOVE 操作的代价之下，提高 GET 操作的性能。

## Java 实现

### 链表接口

- `LinkedList.java`

```java
package adt.linkedlist;

/**
 * 链表
 *
 * @param <T>
 */
public interface LinkedList<T> {

    /**
     * 添加元素到尾部
     * @param t
     */
    default void add(T t) {
        add(size(), t);
    }

    /**
     * 添加元素到指定下标
     * @param index
     * @param t
     */
    void add(int index, T t);

    /**
     * 获取指定下标元素
     * @param index
     * @return
     */
    T get(int index);

    /**
     * 获取第一个元素
     * @return
     */
    default T getFirst() {
        return get(0);
    }

    /**
     * 获取最后一个元素
     * @return
     */
    default T getLast() {
        return get(size() - 1);
    }

    /**
     * 移除指定下标元素
     * @param index
     * @return
     */
    T remove(int index);

    /**
     * 移除第一个元素
     * @return
     */
    default T removeFirst() {
        return remove(0);
    }

    /**
     * 移除最后一个元素
     * @return
     */
    default T removeLast() {
        return remove(size() - 1);
    }

    /**
     * 获取链表大小
     * @return
     */
    int size();

    /**
     * 判断链表是否为空
     * @return
     */
    boolean empty();
}
```

### 单向链表

- `LinkedListSingle.java`

```java
package adt.linkedlist;

public class LinkedListSingle<T> implements LinkedList<T> {

    private static class SingleNode<T> {
        T data;
        SingleNode<T> next;

        public SingleNode(T data) {
            this.data = data;
        }

        @Override
        public String toString() {
            return data + (next == null ? "" : " -> " + next);
        }
    }

    private SingleNode<T> head;

    /**
     * 添加元素到指定下标
     *
     * @param index
     * @param t
     */
    @Override
    public void add(int index, T t) {
        if (index < 0 || index > size()) return;
        SingleNode<T> cur = head, node = new SingleNode<>(t);
        if (index == 0) {
            node.next = cur;
            head = node;
            return;
        }
        while (--index > 0) cur = cur.next;
        node.next = cur.next;
        cur.next = node;
    }

    /**
     * 获取指定下标元素
     *
     * @param index
     * @return
     */
    @Override
    public T get(int index) {
        if (index < 0 || index >= size()) return null;
        SingleNode<T> cur = head;
        while (index-- > 0) cur = cur.next;
        return cur.data;
    }

    /**
     * 移除指定下标元素
     *
     * @param index
     * @return
     */
    @Override
    public T remove(int index) {
        if (index < 0 || index >= size()) return null;
        SingleNode<T> cur = head;
        if (index == 0) {
            head = head.next;
            return cur.data;
        }
        while (--index > 0) cur = cur.next;
        SingleNode<T> res = cur.next;
        cur.next = cur.next.next;
        return res.data;
    }

    /**
     * 获取链表大小
     *
     * @return
     */
    @Override
    public int size() {
        SingleNode<T> cur = head;
        int size = 0;
        while (cur != null) {
            cur = cur.next;
            size++;
        }
        return size;
    }

    /**
     * 判断链表是否为空
     *
     * @return
     */
    @Override
    public boolean empty() {
        return head == null;
    }
}
```

### 双向链表

- `LinkedListDouble.java`

```java
package adt.linkedlist;

public class LinkedListDouble<T> implements LinkedList<T> {

    private static class DoubleNode<T> {
        T data;
        DoubleNode<T> prev;
        DoubleNode<T> next;

        public DoubleNode(T data) {
            this.data = data;
        }

        String forward() {
            return data + (next == null ? "" : " => " + next.forward());
        }

        String backward() {
            return (prev == null ? "" : prev.backward() + " <= ") + data;
        }

    }

    private DoubleNode<T> head;
    private DoubleNode<T> tail;

    /**
     * 添加元素到指定下标
     *
     * @param index
     * @param t
     */
    @Override
    public void add(int index, T t) {
        int size = size();
        if (index < 0 || index > size) return;
        if (size > 1 && index >= size / 2) {
            addFromTail(size - index, t);
            return;
        }
        DoubleNode<T> cur = head, node = new DoubleNode<>(t);
        if (size == 0) {
            head = tail = node;
            return;
        }
        if (index == 0) {
            node.next = head;
            head.prev = node;
            head = node;
            return;
        }
        while (--index > 0) cur = cur.next;
        node.next = cur.next;
        node.prev = cur;
        if (cur.next != null) cur.next.prev = node;
        cur.next = node;
        if (size == 1) tail = node;
    }

    private void addFromTail(int index, T t) {
        DoubleNode<T> cur = tail, node = new DoubleNode<>(t);
        if (index == 0) {
            node.prev = tail;
            tail.next = node;
            tail = node;
            return;
        }
        while (--index > 0) cur = cur.prev;
        node.prev = cur.prev;
        node.next = cur;
        cur.prev.next = node;
        cur.prev = node;
    }

    /**
     * 获取指定下标元素
     *
     * @param index
     * @return
     */
    @Override
    public T get(int index) {
        int size = size();
        if (index < 0 || index >= size) return null;
        if (index >= size / 2) return getFromTail(size - 1 - index);
        DoubleNode<T> cur = head;
        while (index-- > 0) cur = cur.next;
        return cur.data;
    }

    private T getFromTail(int index) {
        DoubleNode<T> cur = tail;
        while (index-- > 0) cur = cur.prev;
        return cur.data;
    }

    /**
     * 移除指定下标元素
     *
     * @param index
     * @return
     */
    @Override
    public T remove(int index) {
        int size = size();
        if (index < 0 || index >= size) return null;
        if (size == 0) {
            head = tail = null;
            return null;
        }
        if (size == 1) {
            T res = head.data;
            head = tail = null;
            return res;
        }
        if (size > 1 && index >= size / 2) return removeFromTail(size - index - 1);
        if (index == 0) {
            T res = head.data;
            head = head.next;
            head.prev = null;
            return res;
        }
        DoubleNode<T> cur = head;
        while (--index > 0) cur = cur.next;
        T res = cur.next.data;
        if (cur.next.next != null) cur.next.next.prev = cur;
        cur.next = cur.next.next;
        return res;
    }

    private T removeFromTail(int index) {
        if (index == 0) {
            T res = tail.data;
            tail = tail.prev;
            tail.next = null;
            return res;
        }
        DoubleNode<T> cur = tail;
        while (--index > 0) cur = cur.prev;
        T res = cur.prev.data;
        cur.prev.prev.next = cur;
        cur.prev = cur.prev.prev;
        return res;
    }

    /**
     * 获取链表大小
     *
     * @return
     */
    @Override
    public int size() {
        DoubleNode<T> cur = head;
        int size = 0;
        while (cur != null) {
            cur = cur.next;
            size++;
        }
        return size;
    }

    /**
     * 判断链表是否为空
     *
     * @return
     */
    @Override
    public boolean empty() {
        return head == null && tail == null;
    }
}

```

# 结语

链表将连续的数据抽象成一个个节点透过指针(句柄)链接在一起，在空间分配上属于更灵活的线性表。双向链表由于需要维护 `head`、`tail` 以及每个节点的 `prev`、`next` 指针，在 `ADD`、`REMOVE` 操作上比单向链表复杂一些，但是却能使 `GET` 的平均查找节点数从 `size/2` 进步到 `size/4`。后面将会介绍的`树(Tree)`和`图(Graph)`等数据结构也与链表相似，都是透过指针保存数据块的信息。
