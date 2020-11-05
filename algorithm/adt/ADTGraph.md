# ADT: Graph 图

@[TOC](文章目录)

<!-- TOC -->

- [ADT: Graph 图](#adt-graph-图)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [名词解释](#名词解释)
  - [图的定义和表示](#图的定义和表示)
  - [抽象接口](#抽象接口)
  - [两种存储实现](#两种存储实现)
    - [邻接矩阵(Adjacent-Matrix)](#邻接矩阵adjacent-matrix)
    - [邻接链表(Adjacent-List)](#邻接链表adjacent-list)
  - [Java 实现](#java-实现)
    - [接口](#接口)
    - [邻接矩阵实现](#邻接矩阵实现)
    - [邻接表实现](#邻接表实现)
    - [测试](#测试)
- [结语](#结语)

<!-- /TOC -->

## 简介

`图(Graph)`数据结构是一个较为复杂，但是却能很好描述现实世界的各种信息，利用`节点(Vertex)`表示`实体(entity)`，以`边(Edge)`描述实体间的联系。例如：以人为节点，人的交友关系为边；以网页为节点，以链接为边等。透过建立图数据结构，我们可以进行一系列的图算法来对图进行搜索、迭代、排序等。

与图相关的基础算法可以分成几类（参考：算法导论）
- 搜索算法：深度优先搜索 DFS、广度优先搜索 BFS
- 生成树算法：最小生成树算法
- 路径搜索算法：单源最短路径算法、所有节点对间最短路径
- 流网络：计算最大流

本篇主要介绍图数据结构的定义和两种存储实现：`邻接矩阵(Adjacent-Matrix)` & `邻接表(Adjacent-List)`。

## 参考

<table>
  <tr>
    <td></td>
    <td><a href=""></a></td>
  </tr>
</table>

# 正文

## 名词解释

- 顶点 Vertex：图中顶点，通常代表某个实体
  - 键 Key：能唯一表示顶点的标识
  - 卫星数据 Data：顶点上附带的额外信息
- 边 Edge：图中的边，表示实体间的联系
- 有向边：具有方向的边，v1 指向 v2 的边可表示为 v1 -> v2
- 无向边：边不具有方向性，也可以看作两个方向同时存在的边
- 出边：从 v1 指向 v2 的边定义为 v1 的一个出边
- 入边：从 v1 指向 v2 的边定义为 v2 的一个入边

## 图的定义和表示

图数据结构的定义如下：

```
图 G = (V, E)
V 为顶点(Vertex)的集合
E 为边(Edge)的集合
```

每个节点需要有一个`键(key)`来唯一表示，同时可选附带卫星数据；每个边则是描述一个顶点到另一个顶点的有向或无向边，同时也能附带卫星数据(常见附带一个`权值 weight`)

- 无权值无向图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/undirected_graph.png)

- 带权值有向图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/directed_graph.png)

我们可以从示例中看出，图其实就是一种更自由的树结构，图结构不必须存在一个根节点，子节点也能够自由指向父节点。

## 抽象接口

由于后续章节会介绍一些图算法，同时又不想因为实现方式的差异而污染算法本身的思想，故这边先抽象出图数据结构几个对外开放的接口，以下为接口伪代码：

```
Vertex 顶点(v)
    KEY()
    获取顶点的关键字

    DATA()
    获取顶点的卫星数据

Edge 边(e)
    FROM()
    获取边来自的顶点

    TO()
    获取边去往的顶点

    WEIGHT()
    获取边上的权值(或是卫星数据)

Graph 图(G = (V, E))
    VERTICES()
    获取图中所有的顶点

    ADJ-VERTICES(v)
    根据给定节点，返回所有相邻的顶点(有出边指向的顶点)

    EDGES()
    获取图中所有的边

    ADJ-EDGES(v)
    根据给定节点，返回顶点所有的出边

    ADD-VERTEX(key)
    给定键值添加顶点

    ADD-EDGE(v1, v2)
    增加一条由 v1 指向 v2 的边
```

## 两种存储实现

定义好了公共接口之后，我们来看看两种存储实现方式的结构差异以及实现需要的要素

我们以下图的图结构实例来讲解两种实现方式的差异：

![](https://picures.oss-cn-beijing.aliyuncs.com/img/directed_graph.png)

### 邻接矩阵(Adjacent-Matrix)

使用邻接矩阵存储我们首先需要用一个`数组保存所有顶点(包括键值和卫星数据)`，然后我们再使用一个`矩阵保存所有边信息(权重或是卫星数据)`(这里先不考虑多重边)，保存结构如下图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/graph_adj_matrix.png)

由于使用矩阵保存边信息，我们很容易发现使用邻接矩阵表示`稀疏图(|E| << |V|<sup>2</sup>)`的时候，会造成大量的空间浪费。

### 邻接链表(Adjacent-List)

与邻接矩阵相似的是我们依旧需要先用一个数组保存所有顶点信息，不过边信息我们为了节省空间(稀疏图时效果更为明显)，我们使用一个`链表的数组`来记录边的信息，第 i 串链表对应了第 i 个节点的所有出边，如下图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/graph_adj_list.png)

使用链表的优点在于消去稀疏图使用矩阵保存时的空间浪费，但是在查找各个顶点的的出边时的代价也相应的提升了(数组随机访问 -> 链表按序查找)

## Java 实现

最后附上我自己捣鼓出来的图数据结构存储方式的实现，这边给出的是有向图的两种实现，无向图可以在增加边的时候同时添加双向的边即可（考虑的不周到的部分欢迎在下面留言指出

### 接口

- `Vertex.java`

```java
package adt.graph;

/**
 * 图节点
 * @param <T>
 */
public interface Vertex<K, T> {

    /**
     * 取得节点 id
     * @return
     */
    K getKey();

    /**
     * 取得节点卫星数据
     * @return
     */
    T getData();

    /**
     * 重新赋值卫星数据
     * @param t
     * @return
     */
    boolean setData(T t);
}
```

- `Edge.java`

```java
package adt.graph;

/**
 * 图边
 */
public interface Edge<K, T, W> {

    Object nil = new Object();

    /**
     * 边离开的节点
     * @return
     */
    Vertex<K, T> from();

    /**
     * 边进入的节点
     * @return
     */
    Vertex<K, T> to();

    /**
     * 获取边权重/卫星数据
     * @return
     */
    W getWeight();
}
```

- `Graph.java`

```java
package adt.graph;

import java.util.List;

/**
 * 图
 *
 * @param <T>
 */
public interface Graph<K, T, W> {

    class DefaultVertex<K, T> implements Vertex<K, T> {
        private K key;
        private T data;

        public DefaultVertex(K key, T data) {
            this.key = key;
            this.data = data;
        }

        @Override
        public K getKey() {
            return key;
        }

        @Override
        public T getData() {
            return data;
        }

        @Override
        public boolean setData(T t) {
            if (t == null) return false;
            data = t;
            return true;
        }

        @Override
        public String toString() {
            if (data == null) {
                return String.valueOf(key);
            }
            return "(key=" + key + ", data=" + data + ')';
        }
    }

    class DefaultEdge<K, T, W> implements Edge<K, T, W> {
        private Vertex<K, T> from;
        private Vertex<K, T> to;
        private W weight;

        public DefaultEdge(Vertex<K, T> from, Vertex<K, T> to) {
            this.from = from;
            this.to = to;
            this.weight = (W)Edge.nil;
        }

        public DefaultEdge(Vertex<K, T> from, Vertex<K, T> to, W weight) {
            this.from = from;
            this.to = to;
            this.weight = weight;
        }

        @Override
        public Vertex<K, T> from() {
            return from;
        }

        @Override
        public Vertex<K, T> to() {
            return to;
        }

        @Override
        public W getWeight() {
            return weight;
        }

        @Override
        public String toString() {
            if (weight == Edge.nil) {
                return "" + '{' + from + " -> " + to + '}';
            }
            return "{" + from + " -> " + to + ", weight=" + weight + '}';
        }
    }

    /**
     * 获取所有节点
     *
     * @return
     */
    List<Vertex<K, T>> getVertices();

    /**
     * 获取所有邻接节点
     *
     * @param key
     * @return
     */
    List<Vertex<K, T>> getAdjVertices(K key);

    /**
     * 添加节点
     *
     * @param key
     * @return
     */
    default boolean addVertex(K key) {
        return addVertex(key, null);
    }

    /**
     * 添加节点和卫星数据
     *
     * @param key
     * @param data
     * @return
     */
    boolean addVertex(K key, T data);

    default boolean[] addVertices(K[] keys) {
        int n = keys.length;
        boolean[] res = new boolean[n];
        for (int i = 0; i < n; i++) res[i] = addVertex(keys[i], null);
        return res;
    }

    default boolean[] addVertices(K[] keys, T[] dataList) {
        int n = keys.length;
        boolean[] res = new boolean[n];
        for (int i = 0; i < n; i++) res[i] = addVertex(keys[i], dataList[i]);
        return res;
    }

    /**
     * 获取所有边
     *
     * @return
     */
    List<Edge<K, T, W>> getEdges();

    /**
     * 获取所有出边
     *
     * @param key
     * @return
     */
    List<Edge<K, T, W>> getAdjEdges(K key);

    /**
     * 添加边
     * @param fromKey
     * @param toKey
     * @return
     */
    default boolean addEdge(K fromKey, K toKey) {
        return addEdge(fromKey, toKey, (W)Edge.nil);
    }

    /**
     * 添加边
     *
     * @param fromKey
     * @param toKey
     * @param weight
     * @return
     */
    boolean addEdge(K fromKey, K toKey, W weight);

    default boolean[] addEdges(K[] fromKeys, K[] toKeys) {
        int n = fromKeys.length;
        boolean[] res = new boolean[n];
        for (int i = 0; i < n; i++) res[i] = addEdge(fromKeys[i], toKeys[i]);
        return res;
    }

    default boolean[] addEdges(K[] fromKeys, K[] toKeys, W[] weights) {
        int n = fromKeys.length;
        boolean[] res = new boolean[n];
        for (int i = 0; i < n; i++) res[i] = addEdge(fromKeys[i], toKeys[i], weights[i]);
        return res;
    }
}
```

### 邻接矩阵实现

- `DirectedGraphWithAdjMatrix.java`

```java
package adt.graph;

import java.util.ArrayList;
import java.util.List;

/**
 * 有向图的邻接矩阵实现
 *
 * @param <K>
 * @param <T>
 * @param <W>
 */
public class DirectedGraphWithAdjMatrix<K, T, W> implements Graph<K, T, W> {

    private K[] vertexKeyList; // 保存顶点的键
    private T[] vertexDataList; // 保存顶点的卫星数据
    private W[][] edgeMatrix; //记录边信息的矩阵
    private int capacity; // 当前数组容量
    private int size; // 已加入的节点数

    public DirectedGraphWithAdjMatrix(int capacity) {
        this.capacity = capacity;
        vertexKeyList = (K[]) new Object[capacity];
        vertexDataList = (T[]) new Object[capacity];
        edgeMatrix = (W[][]) new Object[capacity][capacity];
    }

    /**
     * 由 key 查找 id
     *
     * @param key
     * @return
     */
    private int getId(K key) {
        for (int i = 0; i < size; i++) {
            if (vertexKeyList[i].equals(key)) return i;
        }
        return -1;
    }

    /**
     * 根据偏移量生成节点对象
     *
     * @param id
     * @return
     */
    private Vertex<K, T> getVertex(int id) {
        return new DefaultVertex<>(vertexKeyList[id], vertexDataList[id]);
    }

    @Override
    public List<Vertex<K, T>> getVertices() {
        List<Vertex<K, T>> vertices = new ArrayList<>();
        for (int i = 0; i < size; i++) vertices.add(getVertex(i));
        return vertices;
    }

    @Override
    public List<Vertex<K, T>> getAdjVertices(K key) {
        List<Vertex<K, T>> vertices = new ArrayList<>();
        int i = getId(key);
        if (i < 0) return vertices;
        for (int j = 0; j < size; j++) {
            if (edgeMatrix[i][j] != null) {
                Vertex<K, T> v = getVertex(j);
                vertices.add(v);
            }
        }
        return vertices;
    }

    @Override
    public boolean addVertex(K key, T data) {
        for (int i = 0; i < size; i++) {
            if (vertexKeyList[i].equals(key)) return false;
        }
        vertexKeyList[size] = key;
        vertexDataList[size] = data;
        size++;
        return true;
    }

    private void extend() {
        int c = capacity * 2;

        K[] newVertexKeyList = (K[]) new Object[c];
        T[] newVertexDataList = (T[]) new Object[c];
        W[][] newEdgeMatrix = (W[][]) new Object[c][c];
        for (int i = 0; i < size; i++) {
            newVertexKeyList[i] = vertexKeyList[i];
            newVertexDataList[i] = vertexDataList[i];
            for (int j = 0; j < size; j++) {
                newEdgeMatrix[i][j] = edgeMatrix[i][j];
            }
        }
        vertexKeyList = newVertexKeyList;
        vertexDataList = newVertexDataList;
        edgeMatrix = newEdgeMatrix;
        capacity = c;
    }

    @Override
    public List<Edge<K, T, W>> getEdges() {
        List<Edge<K, T, W>> edges = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            for (int j = 0; j < size; j++) {
                if (edgeMatrix[i][j] != null) {
                    Vertex<K, T> from = getVertex(i), to = getVertex(j);
                    Edge<K, T, W> edge = new DefaultEdge<>(from, to, edgeMatrix[i][j]);
                    edges.add(edge);
                }
            }
        }
        return edges;
    }

    @Override
    public List<Edge<K, T, W>> getAdjEdges(K key) {
        List<Edge<K, T, W>> edges = new ArrayList<>();
        int i = getId(key);
        if (i < 0) return edges;
        Vertex<K, T> from = getVertex(i);
        for (int j = 0; j < size; j++) {
            if (edgeMatrix[i][j] != null) {
                Vertex<K, T> to = getVertex(j);
                Edge<K, T, W> edge = new DefaultEdge<>(from, to, edgeMatrix[i][j]);
                edges.add(edge);
            }
        }
        return edges;
    }

    @Override
    public boolean addEdge(K fromKey, K toKey, W weight) {
        int i = getId(fromKey), j = getId(toKey);
        if (i < 0 || j < 0) return false;
        edgeMatrix[i][j] = weight;
        return true;
    }
}
```

### 邻接表实现

- `DirectedGraphWithAdjList.java`

```java
package adt.graph;

import java.util.ArrayList;
import java.util.List;

/**
 * 有向图的邻接表实现
 *
 * @param <K>
 * @param <T>
 * @param <W>
 */
public class DirectedGraphWithAdjList<K, T, W> implements Graph<K, T, W> {

    /**
     * 出边信息链表节点类
     * @param <W>
     */
    private static class EdgeListNode<W> {
        int adjId;
        W weight;
        EdgeListNode<W> next;

        public EdgeListNode(int adjId) {
            this.adjId = adjId;
            this.weight = (W) Edge.nil;
        }

        public EdgeListNode(int adjId, W weight) {
            this.adjId = adjId;
            this.weight = weight;
        }
    }

    private K[] vertexKeyList; // 保存顶点键
    private T[] vertexDataList; // 保存顶点卫星数据
    private EdgeListNode<W>[] edgeList; // 顶点链表数组
    private int capacity; // 当前数组容量
    private int size; // 已加入顶点数

    public DirectedGraphWithAdjList(int capacity) {
        this.capacity = capacity;
        vertexKeyList = (K[]) new Object[capacity];
        vertexDataList = (T[]) new Object[capacity];
        edgeList = new EdgeListNode[capacity];
    }

    /**
     * 由 key 查找 id
     *
     * @param key
     * @return
     */
    private int getId(K key) {
        for (int i = 0; i < size; i++) {
            if (vertexKeyList[i].equals(key)) return i;
        }
        return -1;
    }

    /**
     * 根据偏移量生成节点对象
     *
     * @param id
     * @return
     */
    private Vertex<K, T> getVertex(int id) {
        return new DefaultVertex<>(vertexKeyList[id], vertexDataList[id]);
    }

    @Override
    public List<Vertex<K, T>> getVertices() {
        List<Vertex<K, T>> res = new ArrayList<>();
        for (int i = 0; i < size; i++) res.add(getVertex(i));
        return res;
    }

    @Override
    public List<Vertex<K, T>> getAdjVertices(K key) {
        List<Vertex<K, T>> res = new ArrayList<>();
        int i = getId(key);
        if (i < 0) return res;
        EdgeListNode node = edgeList[i];
        while (node != null) {
            res.add(getVertex(node.adjId));
            node = node.next;
        }
        return res;
    }

    @Override
    public boolean addVertex(K key, T data) {
        for (int i = 0; i < size; i++) {
            if (vertexKeyList[i].equals(key)) return false;
        }
        vertexKeyList[size] = key;
        vertexDataList[size] = data;
        size++;
        return true;
    }

    @Override
    public List<Edge<K, T, W>> getEdges() {
        List<Edge<K, T, W>> res = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            Vertex<K, T> from = getVertex(i);
            EdgeListNode<W> node = edgeList[i];
            while (node != null) {
                Vertex<K, T> to = getVertex(node.adjId);
                res.add(new DefaultEdge<>(from, to, (W) node.weight));
                node = node.next;
            }
        }
        return res;
    }

    @Override
    public List<Edge<K, T, W>> getAdjEdges(K key) {
        List<Edge<K, T, W>> res = new ArrayList<>();
        int i = getId(key);
        if (i < 0) return res;
        EdgeListNode<W> node = edgeList[i];
        Vertex<K, T> from = getVertex(i);
        while (node != null) {
            Vertex<K, T> to = getVertex(node.adjId);
            res.add(new DefaultEdge<>(from, to, (W) node.weight));
            node = node.next;
        }
        return res;
    }

    @Override
    public boolean addEdge(K fromKey, K toKey, W weight) {
        int i = getId(fromKey), j = getId(toKey);
        if (i < 0 || j < 0) return false;
        EdgeListNode<W> node = new EdgeListNode<>(j, weight);
        if (edgeList[i] == null) {
            edgeList[i] = node;
        } else {
            EdgeListNode<W> cur = edgeList[i];
            while (cur.next != null) cur = cur.next;
            cur.next = node;
        }
        return true;
    }
}
```

### 测试

- 测试图1

![](https://picures.oss-cn-beijing.aliyuncs.com/img/test_1.png)

- 测试图2

![](https://picures.oss-cn-beijing.aliyuncs.com/img/test_2.png)

- `GraphTest.java`

```java
package adt.graph;

import org.junit.Test;

import java.util.Arrays;

import static org.junit.Assert.*;

public class GraphTest {

    private Graph<Integer, Integer, Integer> graph;

    private boolean[] buildSuccessSeq(int size) {
        boolean[] res = new boolean[size];
        Arrays.fill(res, true);
        return res;
    }

    @Test
    public void test_directed_graph_with_adj_matrix() {
        test_1(new DirectedGraphWithAdjMatrix<>(4));
        test_2(new DirectedGraphWithAdjMatrix<>(6));
    }

    @Test
    public void test_directed_graph_with_adj_list() {
        test_1(new DirectedGraphWithAdjList<>(4));
        test_2(new DirectedGraphWithAdjList<>(6));
    }

    /**
     * 第一张图测试
     *
     * @param graph
     */
    private void test_1(Graph<Integer, Integer, Integer> graph) {
        System.out.println("test_1: type<Integer, Integer, Integer>");
        Integer[] keys = new Integer[]{0, 2, 4, 6};
        boolean[] addVerticesRes = buildSuccessSeq(keys.length);
        assertArrayEquals(addVerticesRes, graph.addVertices(keys));

        Integer[] fromKeys = new Integer[]{0, 0, 0, 2, 2, 4};
        Integer[] toKeys = new Integer[]{2, 4, 6, 4, 6, 6};
        boolean[] addEdgesRes = buildSuccessSeq(fromKeys.length);
        assertArrayEquals(addEdgesRes, graph.addEdges(fromKeys, toKeys));

        System.out.println("all vertices: " + graph.getVertices());
        System.out.println("0 to : " + graph.getAdjVertices(0));
        System.out.println("2 to : " + graph.getAdjVertices(2));
        System.out.println("4 to : " + graph.getAdjVertices(4));
        System.out.println("6 to : " + graph.getAdjVertices(6));

        System.out.println("all edges" + graph.getEdges());
        System.out.println("0 edges: " + graph.getAdjEdges(0));
        System.out.println("2 edges: " + graph.getAdjEdges(2));
        System.out.println("4 edges: " + graph.getAdjEdges(4));
        System.out.println("6 edges: " + graph.getAdjEdges(6));
        System.out.println();
    }

    /**
     * 第二张图测试
     *
     * @param graph
     */
    private void test_2(Graph<String, Integer, Integer> graph) {
        System.out.println("test_2: type<String, Integer, Integer>");
        graph.addVertex("s", null);
        graph.addVertex("v1", null);
        graph.addVertex("v2", null);
        graph.addVertex("v3", null);
        graph.addVertex("v4", null);
        graph.addVertex("t", null);
        graph.addEdge("s", "v1", 16);
        graph.addEdge("s", "v2", 13);
        graph.addEdge("v1", "v3", 12);
        graph.addEdge("v2", "v1", 4);
        graph.addEdge("v2", "v4", 14);
        graph.addEdge("v3", "v2", 9);
        graph.addEdge("v3", "t", 20);
        graph.addEdge("v4", "v3", 7);
        graph.addEdge("v4", "t", 4);

        System.out.println("all vertices: " + graph.getVertices());
        System.out.println("s to: " + graph.getAdjVertices("s"));
        System.out.println("v1 to: " + graph.getAdjVertices("v1"));
        System.out.println("v2 to: " + graph.getAdjVertices("v2"));
        System.out.println("v3 to: " + graph.getAdjVertices("v3"));
        System.out.println("v4 to: " + graph.getAdjVertices("v4"));
        System.out.println("t to: " + graph.getAdjVertices("t"));

        System.out.println(graph.getEdges());
        System.out.println("s edges: " + graph.getAdjEdges("s"));
        System.out.println("v1 edges: " + graph.getAdjEdges("v1"));
        System.out.println("v2 edges: " + graph.getAdjEdges("v2"));
        System.out.println("v3 edges: " + graph.getAdjEdges("v3"));
        System.out.println("v4 edges: " + graph.getAdjEdges("v4"));
        System.out.println("t edges: " + graph.getAdjEdges("t"));
        System.out.println();
    }
}
```

# 结语

图数据结构相较于之前介绍过的线性表、树等更为复杂，不过应用面也更广泛。应用图数据结构的算法难点在于：

1. 将数据抽象成图结构
2. 将问题转化为图结构算法
3. 降低图结构算法使之在有效时间内完成计算

由于当前存在许多图算法都属于 NP 完全问题，也就是并不能在多项式的时间内完成，因此当面对大规模数据的计算的时候就可能需要采用求近似结果或是适当的贪心算法来加快算法的执行。本篇提供的图存储实现将作为后续图算法实现的基础，虽然在效率和空间复杂度上有待加强，但是便于理解和后续图算法的朴素实现。
