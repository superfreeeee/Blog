# ADT: Binary-Search-Tree 二叉搜索树

@[TOC](文章目录)

<!-- TOC -->

- [ADT: Binary-Search-Tree 二叉搜索树](#adt-binary-search-tree-二叉搜索树)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [二叉搜索树(BST)结构](#二叉搜索树bst结构)
  - [抽象接口](#抽象接口)
  - [实现要素](#实现要素)
  - [Java 实现](#java-实现)
- [结语](#结语)

<!-- /TOC -->

## 简介

`树(Tree)`数据结构也是非常常见的数据结构之一，以链表的形式为基础，以`节点`为保存数据的单位，每个节点都有一个指向子节点的指针，每个节点可以存在任意数量的子节点。

我们先来看看一般树的定义(参考：算法导论)：

```
有根树定义：
    根节点 root

树节点定义：
    关键词 key
    附带数据(卫星数据) data
    子节点列表 children
        # 为子节点数组，保存指向子节点的指针或句柄
        # 子节点的可以依左到右的顺序保存，也可以无视左右顺序随机保存在数组内              
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/tree.png)

由于一般有根树的结构太过松散，通常对于特定应用场景我们会对加上一些限制：
- 限制子节点数：限制每个节点最大/最小的子节点数量（例：n叉树-限定子节点数最大为 n 的树）
- 数据编排：将节点按键排列成指定形式（例：搜索树-左子树的键 < 根节点的键 < 右子树的键）
- 平衡条件：维护指定条件(深度、黑高\[红黑树\]、...)的平衡（例：AVL树(高度平衡)、红黑树(黑高平衡)）
- 操作副作用：对每次操作进行一定程度上的结构变换(向树根提升、旋转、摊平、...)（例：路径压缩）

而本篇要介绍的`二叉搜索树(BST = Binary Search Tree)`就是在一般树的基础上加上以下条件：

1. 每个节点至多存在两个子节点
2. 每个节点满足以下条件
   1. 左子节点的键 < 该节点的键 < 右子节点的键
   2. 左子节点和右子节点分别都是二叉搜索树

接下来我们就来看看二叉搜索树的实现细节吧

## 参考

<table>
  <tr>
    <td></td>
    <td><a href=""></a></td>
  </tr>
</table>

# 正文

## 二叉搜索树(BST)结构

结构限制：

1. 每个节点至多存在两个子节点
2. 每个节点满足以下条件
   1. 左子节点的键 < 该节点的键 < 右子节点的键
   2. 左子节点和右子节点分别都是二叉搜索树

由于上面简介已经解释各种限制条件的区别，这边就不再赘述

- 二叉搜索树实例

![](https://picures.oss-cn-beijing.aliyuncs.com/img/bst.jpg)

## 抽象接口

```
SEARCH(key)
根据键查找数据(卫星数据)

MINIMUM()
查找拥有最小键的数据

MAXIMUM()
查找拥有最大键的数据

PREDECESSOR(key)
查找指定键的前驱元素，也就是在所有元素按全序排序后指定键指向的节点的前一个节点

SUCCESSOR(key)
查找指定键的后继元素

INSERT(key, item)
插入元素，并指定该元素的键

DELETE(key)
删除指定键值的节点
```

由于二叉搜索树的性质(结构-第二条规则)，在键值不重复的情况下每个插入节点的位置是固定的并由数据结构内部自主维护

## 实现要素

首先给出二叉搜索树的节点定义

```
BSTNode
    key # 键值
    data # 卫星数据
    left # 指向左子节点
    right # 指向右子节点
```

而数据结构内部仅仅需要保存`根节点 root`即可。

```
BST
    BSTNode root # 根节点指针
```

## Java 实现

- `BinarySearchTree.java`

```java
package adt.bst;

public interface BinarySearchTree<T> {

    /**
     * 根据键查找元素
     * @param key
     * @return
     */
    T search(int key);

    /**
     * 查找键最小的元素
     * @return
     */
    T minimum();

    /**
     * 查找键最大的元素
     * @return
     */
    T maximum();

    /**
     * 查找给定键的前驱元素
     * @param key
     * @return
     */
    T predecessor(int key);

    /**
     * 查找给定键的后继元素
     * @param key
     * @return
     */
    T successor(int key);

    /**
     * 插入元素并指定键
     * @param key
     * @param t
     */
    void insert(int key, T t);

    /**
     * 删除元素
     */
    void delete(int key);
}
```

- `BSTSimple.java`

```java
package adt.bst;

public class BSTSimple<T> implements BinarySearchTree<T> {

    protected static class Node<T> {
        int key;
        T data;
        Node<T> parent;
        Node<T> left;
        Node<T> right;

        Node(int key, T data) {
            this.key = key;
            this.data = data;
        }

        Node(int key, T data, Node<T> parent, Node<T> left, Node<T> right) {
            this.key = key;
            this.data = data;
            this.parent = parent;
            this.left = left;
            this.right = right;
        }

        @Override
        public String toString() {
            return "{key=" + key + ", data=" + data + ", left=" + left + ", right=" + right + '}';
        }
    }

    protected Node<T> root;

    public BSTSimple() {}

    public BSTSimple(int[] keys, T[] values) {
        int n = keys.length;
        build(keys, values, 0, n - 1);
    }

    private void build(int[] keys, T[] values, int l, int r) {
        int m = (l + r) / 2;
        insert(keys[m], values[m]);
        if (l < r) {
            build(keys, values, l, m - 1);
            build(keys, values, m + 1, r);
        }
    }

    /**
     * 根据键查找元素
     *
     * @param key
     * @return
     */
    public T search(int key) {
        return _search(root, key).data;
    }

    protected Node<T> _search(Node<T> cur, int key) {
        while (cur != null) {
            if (key == cur.key) return cur;
            if (key < cur.key) cur = cur.left;
            else cur = cur.right;
        }
        return new Node<T>(0, null);
    }

    protected boolean find(Node<T> cur, int key) {
        return _search(cur, key) != null;
    }

    /**
     * 查找键最小的元素
     *
     * @return
     */
    public T minimum() {
        return root == null ? null : minimum(root).data;
    }

    private Node<T> minimum(Node<T> node) {
        while (node != null && node.left != null) node = node.left;
        return node;
    }

    /**
     * 查找键最大的元素
     *
     * @return
     */
    public T maximum() {
        return root == null ? null : maximum(root).data;
    }

    private Node<T> maximum(Node<T> node) {
        while (node != null && node.right != null) node = node.right;
        return node;
    }

    /**
     * 查找给定键的前驱元素
     *
     * @param key
     * @return
     */
    public T predecessor(int key) {
        Node<T> cur = _search(root, key);
        if (cur.left != null) return maximum(cur.left).data;
        while (cur.parent != null && cur == cur.parent.left) cur = cur.parent;
        return cur.parent == null ? null : cur.parent.data;
    }

    /**
     * 查找给定键的后继元素
     *
     * @param key
     * @return
     */
    public T successor(int key) {
        Node<T> cur = _search(root, key);
        if (cur.right != null) return minimum(cur.right).data;
        while (cur.parent != null && cur == cur.parent.right) cur = cur.parent;
        return cur.parent == null ? null : cur.parent.data;
    }

    /**
     * 插入元素并指定键
     *
     * @param key
     * @param t
     */
    public void insert(int key, T t) {
        Node y = null, x = root, node = new Node<T>(key, t);
        if (x == null) {
            root = node;
            return;
        }
        while (x != null) {
            y = x;
            if (key < x.key) x = x.left;
            else x = x.right;
        }
        if (key < y.key) y.left = node;
        else y.right = node;
        node.parent = y;
    }

    /**
     * 删除元素
     *
     * @param key
     */
    public void delete(int key) {
        Node<T> cur = _search(root, key);
        if (cur == null) return;
        if (cur.left == null) transplant(cur, cur.right);
        else if (cur.right == null) transplant(cur, cur.left);
        else {
            Node<T> post = minimum(cur.right);
            if (post.parent != cur) {
                transplant(post, post.right);
                post.right = cur.right;
                post.right.parent = post;
            }
            transplant(cur, post);
            post.left = cur.left;
            post.left.parent = post;
        }
    }

    private void transplant(Node<T> u, Node<T> v) {
        if (u.parent == null) root = v;
        else if (u == u.parent.left) u.parent.left = v;
        else u.parent.right = v;
        if (v != null) v.parent = u.parent;
    }
}
```

- `BinarySearchTreeTest.java`

```java
public class BinarySearchTreeTest {
    @Test
    public void test_bst_simple() {
        BinarySearchTree<Integer> t = new BSTSimple<Integer>(new int[]{0, 1, 2, 3, 4, 5, 6}, new Integer[]{10, 11, 12, 13, 14, 15, 16});
        assertEquals((Integer) 10, t.minimum());
        assertEquals((Integer) 16, t.maximum());
        assertEquals((Integer) 13, t.search(3));
        t.delete(1);
        t.delete(3);
        assertEquals((Integer) 12, t.search(2));
        assertEquals((Integer) 10, t.predecessor(2));
        assertEquals((Integer) 14, t.successor(2));
    }
}
```

由于树节点的管理被屏蔽在数据结构内部，我们可以自己定义一个 `info` 接口来可视化内部结构细节：

- `BinarySearchTree.java`

```java
public interface BinarySearchTree<T> {
    // ...
    
    /**
     * 展示结构内容
     */
    void info();
}
```

- `BSTSimple.java`

```java
public class BSTSimple<T> implements BinarySearchTree<T> {
    protected static class Node<T> {
        // ..

        @Override
        public String toString() {
            return "{key=" + key + ", data=" + data + ", left=" + left + ", right=" + right + '}';
        }
    }
    // ...

    /**
     * 展示结构内容
     */
    public void info() {
        System.out.println("BST: " + root);
    }
}
```

# 结语

`二叉搜索树(BST)`可说是学习任何树结构时最基础也最经典的树结构。二叉树的结构限定一个节点至多存在左、右两个子节点，而二叉搜索树更进一步限定键值的顺序。后面将会分别介绍 BST 的更进一步扩展：`AVL(高度平衡二叉搜索树)`、`红黑树`。

