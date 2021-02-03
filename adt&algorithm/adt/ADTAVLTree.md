# ADT: AVL Tree 平衡二叉搜索树(附Java实现)

@[TOC](文章目录)

<!-- TOC -->

- [ADT: AVL Tree 平衡二叉搜索树(附Java实现)](#adt-avl-tree-平衡二叉搜索树附java实现)
  - [简介](#简介)
  - [参考](#参考)
  - [完整示例代码](#完整示例代码)
- [正文](#正文)
  - [平衡条件](#平衡条件)
    - [二叉搜索树有什么问题？](#二叉搜索树有什么问题)
    - [如何平衡？](#如何平衡)
  - [旋转](#旋转)
    - [左、右旋转](#左右旋转)
    - [单旋转(LL、RR)](#单旋转llrr)
    - [双旋转(LR、RL)](#双旋转lrrl)
  - [插入 & 删除操作 (insert & delete)](#插入--删除操作-insert--delete)
- [结语](#结语)

<!-- /TOC -->

## 简介

前一篇：<a href="https://blog.csdn.net/weixin_44691608/article/details/109479061">ADT: Binary-Search-Tree 二叉搜索树</a>我们介绍了二叉搜索树 Binary Search Tree 的基本实现，本篇将要介绍的 AVL Tree 是基于 BST 的基础之上，附带额外平衡条件的二叉搜索树的变种。


## 参考

<table>
  <tr>
    <td>算法导论-原书第三版</td>
    <td><a href=""></a></td>
  </tr>
</table>

## 完整示例代码

<a href="https://github.com/superfreeeee/Blog-code/tree/main/adt_algorithm/src/main/java/adt/tree/avl">https://github.com/superfreeeee/Blog-code/tree/main/adt_algorithm/src/main/java/adt/tree/avl</a>

# 正文

## 平衡条件

### 二叉搜索树有什么问题？

首先要了解 AVL 树我们要先了解到底 AVL 出了什么问题需要进行所谓的`平衡(balance)`操作。

由于二叉搜索树的性质，同样的数据序列根据不同的插入顺序会产生不同的二叉树，下面我们举一个例子：

- 我们要插入七个数据：`[1, 2, 3, 4, 5, 6, 7]`

    1. 第一种情况我们按大小顺序插入 `1, 2, 3, 4, 5, 6, 7`，会产生如下的二叉树

        ![](https://picures.oss-cn-beijing.aliyuncs.com/img/avl_tree_sample1.png)

    2. 但是如果我们以 `4, 3, 5, 1, 2, 6, 7` 的顺序插入，则会形成第二种的二叉搜索树

        ![](https://picures.oss-cn-beijing.aliyuncs.com/img/avl_tree_sample2.png)

我们知道`树`的数据结构相关操作很大的程度受到树的`高度(height)`影响，因此我们改善 BST 的目的便是尽可能的`最小化树的高度`

### 如何平衡？

我们知道一棵树最有效率的利用方法便是建立一棵`完全二叉树(complete tree)`，让所有节点尽可能的靠近根结点，这样我们就可以说这棵树的高度达到最小。但是由于二叉搜索树的性质，要位置完全二叉树的性质的代价太高了，因此将采用别的方法来限制树的高度的成长。

为了最小化树的高度，我们设定一个观察统计量叫`平衡因子(balance factor)`，它的计算方法是`二叉树左右子树的高度差`。因此我们能够设定一个临界值，只要平衡因子超过临界值就进行`旋转(rotate)`操作来维持平衡因子。

## 旋转

要进行旋转则代表 AVL 树的平衡特性遭到破坏($高度差 \ge 2$)，平衡因子破坏的情形可能有以下情形(节点内数字表示高度)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/avl_tree_broken.png)

上面两种情形我们需要进行`单旋转`，而下面两种则需要进行`双旋转`，而所谓的双旋转则是进行两次单旋转即可完成。下面我们就一一讲解对于不同情况我们要如何来旋转节点以保持平衡特性。

### 左、右旋转

首先我们先介绍两种旋转方式，单旋转和双旋转都是透过该操作完成平衡

![](https://picures.oss-cn-beijing.aliyuncs.com/img/avl_tree_broken_rotate.png)

两种旋转又是互相的逆操作，两种旋转都是将靠下面的节点提升为上层的节点，并将内侧的节点放到另一个节点之下来完成操作。接下来我们就来看看我们是如何透过两种旋转来完成高度的平衡

### 单旋转(LL、RR)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/avl_tree_broken_single.png)

如图，对于单旋转我们只需要进行一次的旋转操作即可恢复 AVL 的平衡性质

### 双旋转(LR、RL)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/avl_tree_broken_double.png)

对于双旋转，我们则需要先将破坏平衡条件的节点旋转到外侧，再对根进行单旋转来保持平衡性质

## 插入 & 删除操作 (insert & delete)

最后我们需要在节点的插入、删除操作后进行平衡性质的恢复，AVLTreeImpl 的实现代码。

代码的核心部分在于 `balance` 操作，从插入/删除位置一路向上检查，如果发现不平衡则判断需要进行何种旋转并恢复平衡性质

- `AVLTreeImpl.java`

```java
package adt.tree.avl;

import adt.tree.bst.BinarySearchTreeImpl;

public class AVLTreeImpl<T> extends BinarySearchTreeImpl<T> implements AVLTree<T> {

    @Override
    public void insert(int key, T data) {
        Node<T> x = new Node<>(key, data);
        insert(x);
        balance(x);
    }

    @Override
    public T delete(int key) {
        Node<T> z = search(root, key);
        if (z == null) return null;
        Node<T> x = delete(z);
        balance(x);
        return z.data;
    }

    // 平衡因子（临界值）
    private int factor = 1;

    /**
     * 计算平衡因子
     *
     * @param x
     * @return
     */
    private int balanceFactor(Node<T> x) {
        return height(x.left) - height(x.right);
    }

    /**
     * 插入/删除后平衡
     *
     * @param x
     */
    private void balance(Node<T> x) {
        while (x != null) {
            int f;
            if (Math.abs(f = balanceFactor(x)) > factor) {
                // 不平衡
                if (f > 0) {
                    if (balanceFactor(x.left) < 0) {
                        // LR
                        leftRotate(x.left);
                    }
                    // LL
                    rightRotate(x);
                } else {
                    if (balanceFactor(x.right) > 0) {
                        // RL
                        rightRotate(x);
                    }
                    // RR
                    leftRotate(x);
                }
                break;
            }
            x = x.parent;
        }
    }

    /**
     * 左旋转
     *
     * @param x
     */
//   x          y
//  / \        / \
// a   y  ->  x   c
//    / \    / \
//   b   c  a   b
    private void leftRotate(Node<T> x) {
        Node<T> y = x.right;
        // x & b
        x.right = y.left;
        if (x.right != null) x.right.parent = x;
        // y & x.parent
        y.parent = x.parent;
        if (x.parent == null) root = y;
        else if (x == x.parent.left) x.parent.left = y;
        else x.parent.right = y;
        // x & y
        y.left = x;
        x.parent = y;
    }

    /**
     * 右旋转
     *
     * @param y
     */
//   x          y
//  / \        / \
// a   y  <-  x   c
//    / \    / \
//   b   c  a   b
    private void rightRotate(Node<T> y) {
        Node<T> x = y.left;
        // y & b
        y.left = x.right;
        if (y.left != null) y.left.parent = y;
        // y & x.parent
        x.parent = y.parent;
        if (y.parent == null) root = x;
        else if (y == y.parent.left) y.parent.left = x;
        else y.parent.right = x;
        // x & y
        x.right = y;
        y.parent = x;
    }
}
```

# 结语

本篇介绍的 AVL 树是对 BST 的一种优化，然而在部分场景中这样的实现效率其实过于低下。这是由于 AVL 的平衡条件过于严苛，大量的 insert/delete 操作会引起效率问题。下一篇我们将带来二叉搜索树的另一种实现：`红黑树(Red-Black Tree)`，透过对节点进行着色来放宽平衡的限制(对`black-height 黑高`进行平衡)，同时又能一定程度上确保二叉搜索树的高度增长不那么快。
