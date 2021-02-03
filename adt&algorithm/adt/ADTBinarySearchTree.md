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
    - [`interface Tree<T>` 树接口](#interface-treet-树接口)
    - [`interface BinarySearchTree<T>` 二叉搜索树接口](#interface-binarysearchtreet-二叉搜索树接口)
    - [`class BinarySearchTreeImpl<T>` 二叉搜索树实现](#class-binarysearchtreeimplt-二叉搜索树实现)
    - [`class BinarySearchTreeTest` 单元测试](#class-binarysearchtreetest-单元测试)
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
Node
    key # 键值
    data # 卫星数据
    parent # 指向父节点
    left # 指向左子节点
    right # 指向右子节点
```

而数据结构内部仅仅需要保存`根节点 root`即可。

```
BST
    Node root # 根节点指针
```

## Java 实现

### `interface Tree<T>` 树接口

```java
package adt.tree;

/**
 * 树
 *
 * @param <T>
 */
public interface Tree<T> {

    /**
     * 插入节点
     *
     * @param key
     * @param data
     */
    void insert(int key, T data);

    /**
     * 删除节点
     *
     * @param key
     * @return
     */
    T delete(int key);

    /**
     * 返回树高
     *
     * @return
     */
    int height();

    /**
     * 检查树是否为空
     *
     * @return
     */
    boolean empty();

    /**
     * 返回节点数量
     *
     * @return
     */
    int nodes();

    /**
     * 先序遍历
     */
    void preorder();

    /**
     * 中序遍历
     */
    void inorder();

    /**
     * 后序遍历
     */
    void postorder();

    /**
     * 层序遍历
     */
    void layerOrder();
}
```

### `interface BinarySearchTree<T>` 二叉搜索树接口

```java
package adt.tree.bst;

import adt.tree.Tree;

/**
 * 二叉搜索树
 *
 * @param <T>
 */
public interface BinarySearchTree<T> extends Tree<T> {

    /**
     * 根据键查找元素
     *
     * @param key
     * @return
     */
    T search(int key);

    /**
     * 查找键最小的元素
     *
     * @return
     */
    T minimum();

    /**
     * 查找键最大的元素
     *
     * @return
     */
    T maximum();

    /**
     * 查找给定键的前驱元素
     *
     * @param key
     * @return
     */
    T predecessor(int key);

    /**
     * 查找给定键的后继元素
     *
     * @param key
     * @return
     */
    T successor(int key);

    /**
     * 展示树形结构
     */
    void tree();
}
```

### `class BinarySearchTreeImpl<T>` 二叉搜索树实现

```java
package adt.tree.bst;

import java.util.LinkedList;
import java.util.Queue;

public class BinarySearchTreeImpl<T> implements BinarySearchTree<T> {

    protected static class Node<T> {
        public int key;
        public T data;
        public Node<T> parent;
        public Node<T> left;
        public Node<T> right;

        public Node(int key, T data) {
            this.key = key;
            this.data = data;
        }

        @Override
        public String toString() {
            return "{key=" + key +
                    ", data=" + data +
                    ", left=" + left +
                    ", right=" + right +
                    '}';
        }

        String simple() {
            return "{key=" + key + ", data=" + data + "}";
        }

        void tree(String prefix) {
            System.out.println(prefix + simple());
            prefix = prefix + "  ";
            if (left != null) left.tree(prefix);
            else if (right != null) System.out.println(prefix + "null");
            if (right != null) right.tree(prefix);
        }
    }

    protected Node<T> root;

    public static <T> BinarySearchTreeImpl<T> from(int[] keys, T[] values) {
        int n = keys.length;
        BinarySearchTreeImpl<T> bst = new BinarySearchTreeImpl<>();
        bst.build(keys, values, 0, n - 1);
        return bst;
    }

    private void build(int[] keys, T[] values, int l, int r) {
        if (l + 1 == r) { // [l, r]
            insert(keys[l], values[l]);
            insert(keys[r], values[r]);
        } else if (l == r) { // [l]
            insert(keys[l], values[l]);
        } else if (l + 1 < r) { // [l .. m .. r]
            int m = (l + r) / 2;
            insert(keys[m], values[m]);
            build(keys, values, l, m - 1);
            build(keys, values, m + 1, r);
        }
    }

    @Override
    public T search(int key) {
        Node<T> node = search(root, key);
        return node == null ? null : node.data;
    }

    protected Node<T> search(Node<T> node, int key) {
        while (node != null && node.key != key) {
            node = (key < node.key ? node.left : node.right);
        }
        return node;
    }

    @Override
    public T minimum() {
        Node<T> min = minimum(root);
        return min == null ? null : min.data;
    }

    private Node<T> minimum(Node<T> node) {
        if (node == null) return null;
        while (node.left != null) node = node.left;
        return node;
    }

    @Override
    public T maximum() {
        Node<T> max = maximum(root);
        return max == null ? null : max.data;
    }

    private Node<T> maximum(Node<T> node) {
        if (node == null) return null;
        while (node.right != null) node = node.right;
        return node;
    }

    @Override
    public T predecessor(int key) {
        if (root == null) return null; // 树为空
        Node<T> node = searchClosest(root, key);
        if (node.key < key) return node.data;
        Node<T> pre = predecessor(node);
        return pre == null ? null : pre.data;
    }

    private Node<T> predecessor(Node<T> node) {
        if (node.left != null) {
            Node<T> pre = maximum(node.left);
            return pre;
        }
        while (node.parent != null && node == node.parent.left) {
            node = node.parent;
        }
        if (node.parent == null) return null;
        return node.parent;
    }

    private Node<T> searchClosest(Node<T> node, int key) {
        Node<T> pre = null;
        while (node != null && node.key != key) {
            pre = node;
            node = (key < node.key ? node.left : node.right);
        }
        return node == null ? pre : node;
    }

    @Override
    public T successor(int key) {
        if (root == null) return null;
        Node<T> node = searchClosest(root, key);
        if (node.key > key) return node.data;
        Node<T> next = successor(node);
        return next == null ? null : next.data;
    }

    private Node<T> successor(Node<T> node) {
        if (node.right != null) {
            Node<T> next = minimum(node.right);
            return next;
        }
        while (node.parent != null && node == node.parent.right) {
            node = node.parent;
        }
        if (node.parent == null) return null;
        return node.parent;
    }

    @Override
    public void insert(int key, T data) {
        insert(new Node<>(key, data));
    }

    protected void insert(Node<T> x) {
        Node<T> pre = null, cur = root;
        while (cur != null) {
            pre = cur;
            cur = (x.key <= pre.key ? pre.left : pre.right);
        }
        x.parent = pre;
        if (pre == null) {
            root = x;
        } else if (x.key <= pre.key) {
            pre.left = x;
        } else {
            pre.right = x;
        }
    }

    @Override
    public T delete(int key) {
        Node<T> z = search(root, key);
        if (z == null) return null;
        delete(z);
        return z.data;
    }

    protected Node<T> delete(Node<T> z) {
        Node<T> x;
        if (z.left == null) {
            x = z.right;
            transplant(z, z.right);
        } else if (z.right == null) {
            x = z.left;
            transplant(z, z.left);
        } else {
            // target 必有两子
            Node<T> y = x = successor(z); // cur 为 target 后继
            if (y.parent != z) {
                if (y.right != null) x = y.right;
                transplant(y, y.right); // 必无左子
                y.right = z.right;
                y.right.parent = y;
            }
            transplant(z, y);
            y.left = z.left;
            y.left.parent = y;
        }
        //  x        x
        //   \  or  /
        //    z    z
        return x;
    }

    /**
     * 处理 u.parent 与 v 的链接
     *
     * @param u
     * @param v
     */
    private void transplant(Node<T> u, Node<T> v) {
        if (u == root) root = v;
        else if (u == u.parent.left) u.parent.left = v;
        else u.parent.right = v;
        if (v != null) {
            v.parent = u.parent;
        }
    }

    @Override
    public int height() {
        return height(root);
    }

    protected int height(Node node) {
        if (node == null) return 0;
        return Math.max(height(node.left), height(node.right)) + 1;
    }

    @Override
    public boolean empty() {
        return root == null;
    }

    @Override
    public int nodes() {
        return nodes(root);
    }

    private int nodes(Node node) {
        if (node == null) return 0;
        return nodes(node.left) + nodes(node.right) + 1;
    }

    @Override
    public void preorder() {
        preorder(root);
    }

    private void preorder(Node node) {
        if (node != null) {
            System.out.println(node.simple());
            preorder(node.left);
            preorder(node.right);
        }
    }

    @Override
    public void inorder() {
        inorder(root);
    }

    private void inorder(Node node) {
        if (node != null) {
            inorder(node.left);
            System.out.println(node.simple());
            inorder(node.right);
        }
    }

    @Override
    public void postorder() {
        postorder(root);
    }

    private void postorder(Node node) {
        if (node != null) {
            postorder(node.left);
            postorder(node.right);
            System.out.println(node.simple());
        }
    }

    @Override
    public void layerOrder() {
        if (root != null) {
            Queue<Node> Q = new LinkedList<>();
            Q.offer(root);
            while (Q.size() > 0) {
                Node node = Q.poll();
                System.out.println(node.simple());
                if (node.left != null) Q.offer(node.left);
                if (node.right != null) Q.offer(node.right);
            }
        }
    }

    @Override
    public String toString() {
        return "BST: " + (root == null ? "empty" : root.toString());
    }

    @Override
    public void tree() {
        root.tree("");
    }
}
```

### `class BinarySearchTreeTest` 单元测试

```java
package adt.tree.bst;

import org.junit.Test;

import static org.junit.Assert.*;

public class BinarySearchTreeTest {

    @Test
    public void test_1() {
        BinarySearchTree<Integer> bst = BinarySearchTreeImpl.from(
                new int[]{1, 3, 5, 7, 9, 11, 13},
                new Integer[]{10, 30, 50, 70, 90, 110, 130}
        );
        bst.tree();

        // traversal
        System.out.println("--- traversal ---");
        System.out.println("preorder:");
        bst.preorder();
        System.out.println("\ninorder:");
        bst.inorder();
        System.out.println("\npostorder:");
        bst.postorder();
        System.out.println("\nlayerOrder:");
        bst.layerOrder();

        // others info
        System.out.println("--- others info ---");
        assertEquals(3, bst.height());
        assertEquals(false, bst.empty());
        assertEquals(7, bst.nodes());

        // search & predecessor & successor
        System.out.println("--- search & predecessor & successor ---");
        assertEquals((Integer) 10, bst.search(1));
        assertEquals(null, bst.predecessor(1));
        assertEquals((Integer) 30, bst.successor(1));
        assertEquals((Integer) 70, bst.search(7));
        assertEquals((Integer) 50, bst.predecessor(7));
        assertEquals((Integer) 90, bst.successor(7));
        assertEquals((Integer) 130, bst.search(13));
        assertEquals((Integer) 110, bst.predecessor(13));
        assertEquals(null, bst.successor(13));
        assertEquals(null, bst.search(8));
        assertEquals((Integer) 70, bst.predecessor(8));
        assertEquals((Integer) 90, bst.successor(8));
        assertEquals(null, bst.search(21));
        assertEquals((Integer) 130, bst.predecessor(21));
        assertEquals(null, bst.successor(21));

        // minimum & maximum
        System.out.println("--- minimum & maximum ---");
        assertEquals((Integer) 10, bst.minimum());
        assertEquals((Integer) 130, bst.maximum());

        // delete
        System.out.println("--- delete ---");
        Integer d = bst.delete(3);
        bst.tree();
        assertEquals((Integer) 30, d);
        d = bst.delete(9);
        bst.tree();
        assertEquals((Integer) 90, d);

        // search & predecessor & successor after delete
        System.out.println("--- search & predecessor & successor after delete ---");
        assertEquals((Integer) 10, bst.search(1));
        assertEquals(null, bst.predecessor(1));
        assertEquals((Integer) 50, bst.successor(1));
        assertEquals((Integer) 70, bst.search(7));
        assertEquals((Integer) 50, bst.predecessor(7));
        assertEquals((Integer) 110, bst.successor(7));
        assertEquals((Integer) 130, bst.search(13));
        assertEquals((Integer) 110, bst.predecessor(13));
        assertEquals(null, bst.successor(13));
        assertEquals(null, bst.search(8));
        assertEquals((Integer) 70, bst.predecessor(8));
        assertEquals((Integer) 110, bst.successor(8));
        assertEquals(null, bst.search(21));
        assertEquals((Integer) 130, bst.predecessor(21));
        assertEquals(null, bst.successor(21));

        // others info 2
        System.out.println("--- others info 2 ---");
        assertEquals(3, bst.height());
        assertEquals(false, bst.empty());
        assertEquals(5, bst.nodes());
    }

    @Test
    public void test_2() {
        BinarySearchTree<Integer> bst = BinarySearchTreeImpl.from(
                new int[]{1, 3, 5, 7, 9, 11, 13, 15, 17},
                new Integer[]{1, 3, 5, 7, 9, 11, 13, 15, 17}
        );
        bst.tree();

        // traversal
        System.out.println("--- traversal ---");
        System.out.println("preorder:");
        bst.preorder();
        System.out.println("\ninorder:");
        bst.inorder();
        System.out.println("\npostorder:");
        bst.postorder();
        System.out.println("\nlayerOrder:");
        bst.layerOrder();

        // others info
        System.out.println("--- others info ---");
        assertEquals(4, bst.height());
        assertEquals(false, bst.empty());
        assertEquals(9, bst.nodes());

        // search & predecessor & successor
        System.out.println("--- search & predecessor & successor ---");
        assertEquals((Integer) 1, bst.search(1));
        assertEquals(null, bst.predecessor(1));
        assertEquals((Integer) 3, bst.successor(1));
        assertEquals((Integer) 7, bst.search(7));
        assertEquals((Integer) 5, bst.predecessor(7));
        assertEquals((Integer) 9, bst.successor(7));
        assertEquals((Integer) 13, bst.search(13));
        assertEquals((Integer) 11, bst.predecessor(13));
        assertEquals((Integer) 15, bst.successor(13));
        assertEquals(null, bst.search(8));
        assertEquals((Integer) 7, bst.predecessor(8));
        assertEquals((Integer) 9, bst.successor(8));
        assertEquals(null, bst.search(21));
        assertEquals((Integer) 17, bst.predecessor(21));
        assertEquals(null, bst.successor(21));

        // minimum & maximum
        System.out.println("--- minimum & maximum ---");
        assertEquals((Integer) 1, bst.minimum());
        assertEquals((Integer) 17, bst.maximum());

        // delete
        System.out.println("--- delete ---");
        Integer d = bst.delete(3);
        bst.tree();
        assertEquals((Integer) 3, d);
        d = bst.delete(9);
        bst.tree();
        assertEquals((Integer) 9, d);
        d = bst.delete(11);
        bst.tree();
        assertEquals((Integer) 11, d);

        // search & predecessor & successor after delete
        System.out.println("--- search & predecessor & successor after delete ---");
        assertEquals((Integer) 1, bst.search(1));
        assertEquals(null, bst.predecessor(1));
        assertEquals((Integer) 5, bst.successor(1));
        assertEquals((Integer) 7, bst.search(7));
        assertEquals((Integer) 5, bst.predecessor(7));
        assertEquals((Integer) 13, bst.successor(7));
        assertEquals((Integer) 13, bst.search(13));
        assertEquals((Integer) 7, bst.predecessor(13));
        assertEquals((Integer) 15, bst.successor(13));
        assertEquals(null, bst.search(8));
        assertEquals((Integer) 7, bst.predecessor(8));
        assertEquals((Integer) 13, bst.successor(8));
        assertEquals(null, bst.search(21));
        assertEquals((Integer) 17, bst.predecessor(21));
        assertEquals(null, bst.successor(21));

        // others info 2
        System.out.println("--- others info 2 ---");
        assertEquals(3, bst.height());
        assertEquals(false, bst.empty());
        assertEquals(6, bst.nodes());
    }
}
```

# 结语

`二叉搜索树(BST)`可说是学习任何树结构时最基础也最经典的树结构。二叉树的结构限定一个节点至多存在左、右两个子节点，而二叉搜索树更进一步限定键值的顺序。后面将会分别介绍 BST 的更进一步扩展：`AVL(高度平衡二叉搜索树)`、`红黑树`。

