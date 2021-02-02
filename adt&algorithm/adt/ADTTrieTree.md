# ADT: Trie Tree 字典树(附 Java 实现)

@[TOC](文章目录)

<!-- TOC -->

- [ADT: Trie Tree 字典树(附 Java 实现)](#adt-trie-tree-字典树附-java-实现)
  - [简介](#简介)
  - [参考](#参考)
  - [完整示例代码](#完整示例代码)
- [正文](#正文)
  - [数据结构](#数据结构)
  - [操作接口](#操作接口)
    - [`interface TrieTree` 接口声明](#interface-trietree-接口声明)
  - [具体实现](#具体实现)
    - [`class TrieTreeImpl` 具体实现](#class-trietreeimpl-具体实现)
    - [`class TrieTreeTest` 测试代码](#class-trietreetest-测试代码)
- [结语](#结语)

<!-- /TOC -->

## 简介

前段时间看到一个算法题-最长公共前缀的一种实现：使用 `Trie Tree 字典树`来实现。后来查阅 Trie Tree 相关的信息发现，字典树主要用于字符串排序和词频统计，公共前缀恰好是字典树能实现的功能之一。接下来我们就来看看字典树是如何实现的。

## 参考

<table>
  <tr>
    <td>TrieTree字典树数据结构的原理、实现及应用</td>
    <td><a href="https://blog.csdn.net/leasonw/article/details/78009402">https://blog.csdn.net/leasonw/article/details/78009402</a></td>
  </tr>
  <tr>
    <td>Leetcode题库-最长公共前缀（java语言版）</td>
    <td><a href="https://blog.csdn.net/weixin_37850160/article/details/86556879">https://blog.csdn.net/weixin_37850160/article/details/86556879</a></td>
  </tr>
</table>

## 完整示例代码

<a href="https://github.com/superfreeeee/Blog-code/tree/main/adt_algorithm/src/main/java/adt/tree/trie">https://github.com/superfreeeee/Blog-code/tree/main/adt_algorithm/src/main/java/adt/tree/trie</a>

# 正文

## 数据结构

在[简介](#简介)中我们简单提到 Trie Tree 字典树的常见应用，所以我们首先先来明确字典树的数据结构。

![](https://picures.oss-cn-beijing.aliyuncs.com/img/trie_tree_sample.png)

上图是一个字典树的实例之一，每个节点保存`一个字符`，由根结点到目标节点的路径上所有字符组成一个字符串(也就是单词)，所以我们还需要在每个节点保存一个计数，表示`单词的数量`。

在该例子中我们可以很直观的感受到这是一种以时间换取空间的做法，透过保存每个出现过的字符组合成单词，高效地利用单词前缀节省空间，同时能保留各个字符串的共同特性。

经过上述描述我们已经可以大略的定义出内部节点类的抽象结构：

```bash
Node:
    char c    # 节点字符
    int count # 单词出现次数
    Node[] children # 所有子节点
```

## 操作接口

接下来我们定义要实现的操作接口：

| Function                   | Usage                |
| -------------------------- | -------------------- |
| insert(String word)        | 加入单词             |
| count(String word)         | 计算给定单词出现次数 |
| countPrefix(String prefix) | 计算给定前缀出现次数 |
| words()                    | 返回全部单词数量     |
| commonPrefix()             | 返回所有单词公共前缀 |
| wordsFrequency()           | 单词词频统计         |

### `interface TrieTree` 接口声明

```java
package adt.tree.trie;

import java.util.Map;

public interface TrieTree {

    /**
     * 加入字符串
     *
     * @param word
     */
    void insert(String word);

    /**
     * 给定字符串出现次数
     *
     * @param word
     * @return
     */
    int count(String word);

    /**
     * 给定前缀出现次数
     *
     * @param prefix
     * @return
     */
    int countPrefix(String prefix);

    /**
     * 返回现有字符串个数
     */
    int words();

    /**
     * 最长公共前缀
     *
     * @return
     */
    String commonPrefix();

    /**
     * 给出所有字符串出现次数
     *
     * @return
     */
    Map<String, Integer> wordsFrequency();

}
```

## 具体实现

下面我们直接给出 Trie Tree 字典树的具体实现，相关实现细节可以直接看到代码内的注释

### `class TrieTreeImpl` 具体实现

```java
package adt.tree.trie;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.Map;

/**
 * 字典树基本实现
 */
public class TrieTreeImpl implements TrieTree {

    /**
     * 内部节点类
     */
    private static class Node {
        int count;
        char c;
        Map<Character, Node> children; // 透过 Map 的哈希映射来避免顺序查找所有字串

        Node(char c) {
            this.c = c;
            this.children = new HashMap<>();
        }

        void tree(String prefix) {
            System.out.println(prefix + c + (count > 0 ? ":" + count : ""));
            prefix = "  " + prefix;
            for (Node child : children.values()) {
                child.tree(prefix);
            }
        }
    }

    private Node root;
    private int words;

    public TrieTreeImpl() {
        this.root = new Node('/');
    }

    public static TrieTreeImpl from(String[] words) {
        TrieTreeImpl tree = new TrieTreeImpl();
        for (String word : words) {
            tree.insert(word);
        }
        return tree;
    }

    @Override
    public void insert(String word) {
        Node cur = root;
        for (char c : word.toCharArray()) {
            if (!cur.children.containsKey(c)) {
                // 路径上节点不存在则建立新的节点
                cur.children.put(c, new Node(c));
            }
            cur = cur.children.get(c);
        }
        // 字符串结尾处 count 递增，表示该单词数量 +1
        cur.count += 1;
        words += 1;
    }

    @Override
    public int words() {
        return words;
//        return countWords(root);
    }

    @Override
    public int count(String word) {
        Node node = getNode(word);
        return node == null ? 0 : node.count;
    }

    /**
     * 查找目标单词结尾节点
     *
     * @param word
     * @return
     */
    private Node getNode(String word) {
        Node cur = root;
        for (char c : word.toCharArray()) {
            if (!cur.children.containsKey(c)) return null;
            cur = cur.children.get(c);
        }
        return cur;
    }

    @Override
    public int countPrefix(String prefix) {
        Node node = getNode(prefix);
        return node == null ? 0 : countWords(node);
    }

    /**
     * 计算给定节点之下所有单词数量
     *
     * @param node
     * @return
     */
    private int countWords(Node node) {
        if (node == null) return 0;
        int res = node.count;
        for (Node child : node.children.values()) {
            res += countWords(child);
        }
        return res;
    }

    @Override
    public String commonPrefix() {
        StringBuilder prefix = new StringBuilder();
        Node cur = root;
        // 只存在单一子节点则为所有单词公共前缀
        // count > 0 表示单词结尾
        while (cur.count == 0 && cur.children.keySet().size() == 1) {
            cur = new ArrayList<>(cur.children.values()).get(0);
            prefix.append(cur.c);
        }
        return prefix.toString();
    }

    @Override
    public Map<String, Integer> wordsFrequency() {
        Map<String, Integer> freq = new HashMap<>();
        dfs(root, "", freq);
        return freq;
    }

    /**
     * 深度优先遍历计算词频
     * @param node
     * @param word
     * @param freq
     */
    private void dfs(Node node, String word, Map<String, Integer> freq) {
        if (node.count > 0) freq.put(word, node.count); // count > 0 表示有单词
        for (Node child : node.children.values()) {
            dfs(child, word + child.c, freq);
        }
    }

    @Override
    public String toString() {
        System.out.println("TrieTreeImpl:");
        root.tree("");
        return "";
    }
}
```

### `class TrieTreeTest` 测试代码

最后给出测试用的代码

```java
package adt.tree.trie;

import org.junit.Test;

import java.util.Arrays;
import java.util.Map;

import static org.junit.Assert.*;

public class TrieTreeTest {

    @Test
    public void test_1() {
        TrieTree trieTree = TrieTreeImpl.from(new String[]{"flower", "flow", "flight", "flow"});
        System.out.println(trieTree);
        assertEquals(4, trieTree.words());
        assertEquals(2, trieTree.count("flow"));
        assertEquals(1, trieTree.count("flower"));
        assertEquals(0, trieTree.count("flowerer"));
        assertEquals(4, trieTree.countPrefix("fl"));
        assertEquals(3, trieTree.countPrefix("flo"));
        assertEquals(1, trieTree.countPrefix("fligh"));
        assertEquals(0, trieTree.countPrefix("flighe"));
        assertEquals("fl", trieTree.commonPrefix());

        System.out.println("--- words frequency ---");
        for (Map.Entry<String, Integer> entry : trieTree.wordsFrequency().entrySet()) {
            System.out.println(entry);
        }
    }

    @Test
    public void test_from() {
        String[][] wordsList = new String[][]{
                new String[]{},
                new String[]{"flower", "flow", "flight", "flow"},
                new String[]{"dog", "racecar", "car"},
                new String[]{"banana", "band", "apple", "apt", "bbc", "app", "ba"},
        };
        for (String[] words : wordsList) {
            System.out.println("words: " + Arrays.toString(words));
            System.out.println(TrieTreeImpl.from(words));
        }
    }

    @Test
    public void test_insert() {
        String[] words = new String[]{"banana", "band", "apple", "apt", "bbc", "app", "ba", "bat"};
        TrieTree trieTree = new TrieTreeImpl();
        System.out.println(trieTree);
        for (String word : words) {
            System.out.println("append: \'" + word + '\'');
            trieTree.insert(word);
            System.out.println(trieTree);
        }
        System.out.println(trieTree.wordsFrequency());
    }
}
```

# 结语

树形的抽象数据结构有非常多的变种，对应不同的应用场景和目标有各种变形。本篇介绍的 Trie Tree 字典树主要用于单词组的统计相关应用，供大家参考。
