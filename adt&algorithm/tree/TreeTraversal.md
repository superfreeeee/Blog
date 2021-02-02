# Tree 树算法: BST-Traversal 二叉(搜索)树的遍历

@[TOC](文章目录)

<!-- TOC -->

- [Tree 树算法: BST-Traversal 二叉(搜索)树的遍历](#tree-树算法-bst-traversal-二叉搜索树的遍历)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [什么是遍历？](#什么是遍历)
  - [二叉树遍历的种类](#二叉树遍历的种类)
  - [伪代码](#伪代码)
    - [先序遍历 Preorder](#先序遍历-preorder)
    - [中序遍历 Inorder](#中序遍历-inorder)
    - [后序遍历 Postorder](#后序遍历-postorder)
    - [层序遍历 Layer](#层序遍历-layer)
  - [Java 实现](#java-实现)
    - [树接口和二叉搜索树](#树接口和二叉搜索树)
    - [四种遍历的实现](#四种遍历的实现)
    - [测试](#测试)
- [结语](#结语)

<!-- /TOC -->

## 简介

数据结构通常是为了某些特定场景所服务，透过付出建立数据结构的时间和空间代价来在搜索、查找、增删等操作上获得更好的性能。而树的相关算法在应用上主要增加搜索的性能，透过建立不同限制条件的树结构来达成目的，例如：`二叉搜索树(Binary-Search Tree)`、`AVL树(高度平衡BST)`、`线索树(Thread Tree)`、`红黑树(Red-Black Tree)`、`区间树`。

树相关的算法差异主要都体现在树的限制条件不同上，根据不同的建树规则，各种操作的实现等。本篇就来介绍对树数据结构最基本的操作之一：`遍历(Traversal)`。

## 参考

<table>
  <tr>
    <td></td>
    <td><a href=""></a></td>
  </tr>
</table>

# 正文

## 什么是遍历？

前一篇：<a href="https://blog.csdn.net/weixin_44691608/article/details/109479061">ADT: Binary-Search-Tree 二叉搜索树</a>提到了最基本的树的定义以及二叉搜索树的实现，本篇就更进一步来讨论二叉搜索树的遍历。

`遍历(Traversal)`这个动作的意思是以某种规则或是顺序访问树中的所有节点。本篇主要针对二叉树(每个节点至多有两个子节点；是否为搜索树与遍历无关)进行遍历。

## 二叉树遍历的种类

在二叉树的遍历中，我们将遍历一棵二叉树的动作拆分成三个子步骤：

- 访问根节点
- 遍历左子树
- 遍历右子树

由于我们已经将子节点区分成左节点和右节点，所以访问子树的顺序通常(可以根据实际算法实现调整)由左至右；而我们又能够根据访问根节点的前后关系将遍历方式分为(root 表示根节点；left 表示左子树；right 表示右子树)：

1. 先序遍历(preorder)：访问顺序 root -> left -> right
2. 中序遍历(inorder)：访问顺序 left -> root -> right
3. 后序遍历(postorder)：访问顺序 left -> right -> root

本篇最后会再介绍一个不局限于二叉树的(当然先序和后序也能够推广到 n 叉树的场景下)遍历方式：

4. 层序遍历(layer)：按层(深度)访问节点

四种遍历方式的差异可以以`深度优先搜索(DFS)`和`广度优先搜索(BFS)`的差异来解释：前三种遍历递归遍历子树类似于优先向下探查，属于`深度优先搜索`的变形；而层序遍历则是按层遍历所有节点，也就是依照与根节点的距离由小到大遍历，属于一种`广度优先搜索`的变形。

## 伪代码

由于树抽象结构以及遍历方式可以高度抽象与实现无关，这边给出四种遍历方式的伪代码。前三种遍历方式的共同点在于：对于一棵树的遍历拆分成根节点、左子树、右子树三个部分，而遍历子树时都是利用同样的遍历规则递归进行遍历。

### 先序遍历 Preorder

```
Preorder(root)
  visit(root)          # 先访问根节点(先序)
  Preorder(root.left)  # 再递归遍历左子树
  Preorder(root.right) # 最后递归遍历右子树
```

### 中序遍历 Inorder

```
Inorder(root)
  Inorder(root.left)  # 先递归遍历左子树
  visit(root)         # 再访问根节点(中序)
  Inorder(root.right) # 最后递归遍历右子树
```

### 后序遍历 Postorder

```
Postorder(root)
  Postorder(root.left)  # 先递归遍历左子树
  Postorder(root.right) # 再递归遍历右子树
  visit(root)           # 最后访问根节点(后序)
```

我们到此可以看出，递归遍历左右子树的相对顺序是不变的，三种遍历方式的差距在于遍历根节点的时机的不同。

### 层序遍历 Layer

层序遍历与前三种遍历不同的是，需要按层访问所有节点，属于一种`广度优先搜索(BFS)`的变形。而广度优先搜索的算法最常见的就是引入一个`队列(Queue)`来排队访问节点，在之后的图遍历也能看到类似的用法

```
Layer(root)
  let Q be a Queue
  Q.add(root)
  while Q is not empty:
    N = Q.poll()               # 拿到下一个要访问的节点
    if N is null then continue # 如果节点为空则跳过
    # 否则访问该节点后并将所有子节点都加入队列之中
    visit(N)
    Q.add(N.left)
    Q.add(N.right)
```

## Java 实现

这边的实现部分我们一样借用前一篇<a href="https://blog.csdn.net/weixin_44691608/article/details/109479061">ADT: Binary-Search-Tree 二叉搜索树</a>所给出的二叉搜索树结构。为了能够直接访问内部结构(节点)，我们使用继承的方式来实现遍历算法，然而在实际应用中通常我们会将遍历算法`直接写在自定义的树结构之中`，或是作为某个特定算法的子历程来实现，而不需要使用继承的手段。

### 树接口和二叉搜索树

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

    /**
     * 展示结构内容
     */
    void info();
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

        public T getData() {
            return data;
        }

        public Node<T> getLeft() {
            return left;
        }

        public Node<T> getRight() {
            return right;
        }

        @Override
        public String toString() {
            return "{key=" + key + ", data=" + data + ", left=" + left + ", right=" + right + '}';
        }
    }

    protected Node<T> root;

    public BSTSimple() {
    }

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

    public T minimum() {
        return root == null ? null : minimum(root).data;
    }

    private Node<T> minimum(Node<T> node) {
        while (node != null && node.left != null) node = node.left;
        return node;
    }

    public T maximum() {
        return root == null ? null : maximum(root).data;
    }

    private Node<T> maximum(Node<T> node) {
        while (node != null && node.right != null) node = node.right;
        return node;
    }

    public T predecessor(int key) {
        Node<T> cur = _search(root, key);
        if (cur.left != null) return maximum(cur.left).data;
        while (cur.parent != null && cur == cur.parent.left) cur = cur.parent;
        return cur.parent == null ? null : cur.parent.data;
    }

    public T successor(int key) {
        Node<T> cur = _search(root, key);
        if (cur.right != null) return minimum(cur.right).data;
        while (cur.parent != null && cur == cur.parent.right) cur = cur.parent;
        return cur.parent == null ? null : cur.parent.data;
    }

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

    public void info() {
        System.out.println("BST: " + root);
    }
}
```

### 四种遍历的实现

- `BSTTraversal.java`

```java
package tree;

import adt.bst.BSTSimple;

import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;
import java.util.Queue;

/**
 * 四种二叉树的遍历
 * 1. 先序遍历
 * 2. 中序遍历
 * 3. 后序遍历
 * 4. 层序遍历
 * @param <T>
 */
public class BSTTraversal<T> extends BSTSimple<T> {

    public BSTTraversal() {
    }

    public BSTTraversal(int[] keys, T[] values) {
        super(keys, values);
    }

    /**
     * 先序遍历
     * @return
     */
    public List<T> preorderTraversal() {
        List<T> res = new ArrayList<>();
        preorder(root, res);
        return res;
    }

    private void preorder(Node<T> node, List<T> res) {
        if (node != null) {
            res.add(node.getData());
            preorder(node.getLeft(), res);
            preorder(node.getRight(), res);
        }
    }

    /**
     * 中序遍历
     * @return
     */
    public List<T> inorderTraversal() {
        List<T> res = new ArrayList<>();
        inorder(root, res);
        return res;
    }

    private void inorder(Node<T> node, List<T> res) {
        if (node != null) {
            inorder(node.getLeft(), res);
            res.add(node.getData());
            inorder(node.getRight(), res);
        }
    }

    /**
     * 后序遍历
     * @return
     */
    public List<T> postorderTraversal() {
        List<T> res = new ArrayList<>();
        postorder(root, res);
        return res;
    }

    private void postorder(Node<T> node, List<T> res) {
        if (node != null) {
            postorder(node.getLeft(), res);
            postorder(node.getRight(), res);
            res.add(node.getData());
        }
    }

    /**
     * 层序遍历
     * @return
     */
    public List<T> layerTraversal() {
        List<T> res = new ArrayList<>();
        if (root == null) return res;
        Queue<Node<T>> Q = new LinkedList<>();
        Q.add(root);
        while (Q.size() > 0) {
            Node<T> cur = Q.poll();
            res.add(cur.getData());
            if (cur.getLeft() != null) Q.add(cur.getLeft());
            if (cur.getRight() != null) Q.add(cur.getRight());
        }
        return res;
    }
}
```

### 测试

- `BSTTraversalTest.java`

```java
package tree;

import org.junit.Before;
import org.junit.Test;

import java.util.Arrays;
import java.util.List;

import static org.junit.Assert.*;

public class BSTTraversalTest {

    private BSTTraversal<Integer> bst;

    @Before
    public void setUp() throws Exception {
        bst = new BSTTraversal<>(new int[]{0, 1, 2, 3, 4, 5, 6}, new Integer[]{10, 11, 12, 13, 14, 15, 16});
        bst.info();
    }

    @Test
    public void test_preorder_traversal() {
        List<Integer> res = bst.preorderTraversal();
        List<Integer> ans = Arrays.asList(new Integer[]{13, 11, 10, 12, 15, 14, 16});
        assertEquals(ans, res);
    }

    @Test
    public void test_inorder_traversal() {
        List<Integer> res = bst.inorderTraversal();
        List<Integer> ans = Arrays.asList(new Integer[]{10, 11, 12, 13, 14, 15, 16});
        assertEquals(ans, res);
    }

    @Test
    public void test_postorder_traversal() {
        List<Integer> res = bst.postorderTraversal();
        List<Integer> ans = Arrays.asList(new Integer[]{10, 12, 11, 14, 16, 15, 13});
        assertEquals(ans, res);
    }

    @Test
    public void test_layer_traversal() {
        List<Integer> res = bst.layerTraversal();
        List<Integer> ans = Arrays.asList(new Integer[]{13, 11, 15, 10, 12, 14, 16});
        assertEquals(ans, res);
    }
}
```

# 结语

本篇介绍了四种遍历树的方式，其实严格意义上这并不能单独算作一个树的算法，而仅仅只能说是树的基本操作之一(增删改查中的`查`动作)，通常不同的遍历方式会应用在不同的算法，并作为算法的子历程(或是说算法的结构)来实现，实际上遍历过程对于每个节点的`访问(visit)`操作则是因应不同算法的需求而变化。
