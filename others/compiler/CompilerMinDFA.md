# 编译原理: 最小化 DFA(划分) & 验证 DFA(Kleene 闭包)

@[TOC](文章目录)

<!-- TOC -->

- [编译原理: 最小化 DFA(划分) & 验证 DFA(Kleene 闭包)](#编译原理-最小化-dfa划分--验证-dfakleene-闭包)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [示例回顾](#示例回顾)
  - [最小化 DFA(划分)](#最小化-dfa划分)
    - [怎么划分？](#怎么划分)
    - [最小化示例](#最小化示例)
  - [验证 DFA](#验证-dfa)
    - [Kleene 闭包定义 $R_{ij}^{k}$](#kleene-闭包定义-r_ijk)
    - [重建 RE 并验证等价](#重建-re-并验证等价)
  - [最终版本的 DFA](#最终版本的-dfa)
- [结语](#结语)

<!-- /TOC -->

## 简介

- 上一篇（第二部）：<a href="https://blog.csdn.net/weixin_44691608/article/details/110213913">编译原理: Subset Construction 子集构造法(幂集构造)(NFA转DFA)</a>
- 上上一篇（第一部）：<a href="https://blog.csdn.net/weixin_44691608/article/details/110195743">编译原理: Thompson 构造法（正则表达式 转 NFA）</a>

作为正则表达式转 DFA 三部曲的最后一部，本篇将解说四个步骤：

1. Thompson 构造法：正则表达式 RegExp -> 不确定有限状态机 NFA
2. 子集构造法：不确定有限状态机 NFA -> 确定有限状态机 DFA
3. 最小化：最小化确定有限状态机 DFA
4. 验证：DFA 转换为的等价正则表达式并验证等价

中的最后两步：`最小化`和`验证`

## 参考

<table>
  <tr>
    <td>编译原理-DFA与正规式的转化</td>
    <td><a href="https://www.cnblogs.com/mznsndy/p/10742827.html">https://www.cnblogs.com/mznsndy/p/10742827.html</a></td>
  </tr>
</table>

# 正文

## 示例回顾

我们在前两部使用了两个例子作为范例，完成了从正则表达式 -> NFA -> DFA 的转换，最终结果是一张 DFA 图：

> 示例1: $a(b|c)^{*}$

![](https://picures.oss-cn-beijing.aliyuncs.com/img/subset_construction_Dtran_a5.png)
![](https://picures.oss-cn-beijing.aliyuncs.com/img/subset_construction_dfa_1.png)

> 示例2: $(a|b)^{*}abb$

![](https://picures.oss-cn-beijing.aliyuncs.com/img/subset_construction_Dtran_b6.png)
![](https://picures.oss-cn-beijing.aliyuncs.com/img/subset_construction_dfa_2.png)

然而这样构建出来的 DFA 其实还存在相当大的冗余部分，接下来我们就要透过`Kleene 闭包`来`最小化 DFA`，同时验证简化后的 DFA 和原来的正则表达式是等价的。

## 最小化 DFA(划分)

### 怎么划分？

直接上伪代码

```
Min-DFA(Dtran)
    let U = {S\F, F} # 初始的等价类有两个组，分别是非终止状态和终止状态
    for (G in U) { # 对等价类中每个组做划分
        while (G can be divided) {
            g1, g2, g3, ... = divide(G)
            remove G from U
            add g1, g2, g3, ... into U
        }
    }
```

1. 首先将上面的 DFA 分为非终止状态和终止状态两组，然后对每一组进行检查是否能够再做划分，若可则对划分后的组再划分，直到不可再划分
2. 最后根据划分后的组重建 Dtran 表并重绘 DFA 图

- 这边需要说明一下`不可划分`的概念：

    若对于组内所有状态，对于所有输入都有相同的输出状态，则称该组`不可再被划分`。

其实简单的看就是当 Dtran 右侧的转移函数的`行`相同时就可以被分为一组

### 最小化示例

> 示例1: $a(b|c)^{*}$

![](https://picures.oss-cn-beijing.aliyuncs.com/img/subset_construction_Dtran_a5.png)

首先根据是否为终止状态(是否包含 a10)划分为两组：$U = \{\{A\},\{B,C,D\}\}$

接下来我们发现对于组 ${B,C,D}$ 中的每一个状态，对于同一个输入都有同样的输出状态（$Dtran[X,b]=C, Dtran[X,c]=D$），则组 ${B,C,D}$ 不可再被划分了，因此最终划分便是 $U = \{\{A\},\{B,C,D\}\}$

我们使用 $B^{'}$ 来取代原来 $B,C,D$ 的位置并重建 $Dtran$ 跟 $DFA$

![](https://picures.oss-cn-beijing.aliyuncs.com/img/min_dtran_sample_1.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/min_dfa_sample_1.png)

> 示例2: $(a|b)^{*}abb$

![](https://picures.oss-cn-beijing.aliyuncs.com/img/subset_construction_Dtran_b6.png)

初始划分为 $U = \{\{A,B,C,D\},\{E\}\}$，然后我们发现只有 $A,C$ 的转换函数结果完全相同（$Dtran[A|C,a]=B, Dtran[A|C,b]=C$），则我们使用 $A^{'}$ 来取代 $A,C$，最终划分为 $U = \{A^{'}: \{A,C\},\{B\},\{D\},\{E\}\}$，新的 $Dtran$ 跟 $DFA$ 如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/min_dtran_sample_2.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/min_dfa_sample_2.png)

## 验证 DFA

接下来我们要验证我们最小化后的 DFA 与原来的正则表达式是等价的，我们将使用 `Kleene 闭包`来完成这个动作

### Kleene 闭包定义 $R_{ij}^{k}$

- 符号定义：$R_{ij}^{k}$、$\oslash$、$\epsilon$

    1. $R_{ij}^{k}$ 表示 Kleene 闭包

        我们将 DFA 中所有节点依序编号为为 $0 \sim n - 1$

        $R_{ij}^{k}$ 则表示从节点 $i$ 到节点 $j$ 途中经过小于等于编号为 $k$ 的中间节点可达的所有路径

    2. $\oslash$ 表示路径不可达
        对于路径不可达的并（$|$和）连接（$\cap$）操作有以下结果：
        - 并：$\oslash | a = a$
        - 连接：$\oslash a = \oslash$

        注意：不可达 $\oslash$ 与空串 $\epsilon$ 不同
        - $\epsilon | a = a$
        - $\epsilon a = a$
    
    3. $\epsilon$ 表示空路径，即状态不发生变化的边（回到同一状态）

- Kleene 初始化 $k = -1$

    而接下来我们介绍 Kleene 闭包的`初始化状态`

    $R_{ij}^{-1}$ 表示从节点 $i$ 到节点 $j$ 不经过`任意节点`的所有路径

    以下为初始化状态的示例

    > $R_{ij}^{-1} = a|b|c$

    ![](https://picures.oss-cn-beijing.aliyuncs.com/img/kleene_initial1.png)

    > $R_{ij}^{-1} = \oslash$

    ![](https://picures.oss-cn-beijing.aliyuncs.com/img/kleene_initial2.png)

    > $R_{ii}^{-1} = a|\epsilon$

    ![](https://picures.oss-cn-beijing.aliyuncs.com/img/kleene_initial3.png)

    > $R_{ii}^{-1} = \epsilon$
    
    ![](https://picures.oss-cn-beijing.aliyuncs.com/img/kleene_initial4.png)

    我们可以看到当出节点和入节点相同时（$i = j$），路径中必然存在一个 $\epsilon$

- Kleene 递归步

    $k = -1$时表示不经过任何中间节点，接下来我们就慢慢的让中间节点的编号限制（$k$）慢慢向上累加，核心公式如下：

    $R_{ij}^{k} = R_{ik}^{k-1}(R_{kk}^{k-1})^{\ast}R_{kj}^{k-1}|R_{ij}^{k-1}$
    
    这个转换式的意思就是新的经过 $k$ 以下的中间节点的结果就是旧的路径经过 $k-1$ 的中间节点与原路径的并，如下图所示

    ![](https://picures.oss-cn-beijing.aliyuncs.com/img/kleene_recursive.png)

有了 Kleene 闭包，我们就可以来找 $s$（起始节点）到 $t$ 终止节点

### 重建 RE 并验证等价

接下来所谓的重建其实就是透过 Kleene 闭包来重建正则表达式，也就是说我们要找的目标就是 $R_{ij}^{k}, i=s0, j=f, k=S$，我们透过不断增加 $k$ 值来找到我们的最终结果，接下来直接使用示例来解说吧（在此我们先将示例的状态重新编号）

> 示例1: $a(b|c)^{*}$

![](https://picures.oss-cn-beijing.aliyuncs.com/img/min_dfa_sample_1_retag.png)

- 初始化状态：$k = -1$

    $R_{00}^{-1} = \epsilon$\
    $R_{01}^{-1} = a$\
    $R_{10}^{-1} = \oslash$\
    $R_{11}^{-1} = b|c|\epsilon$

- $k = 0$

    $R_{00}^{0} = R_{00}^{-1} (R_{00}^{-1})^{\ast} R_{00}^{-1} | R_{00}^{-1} = \epsilon (\epsilon)^{\ast} \epsilon       = \epsilon$\
    $R_{01}^{0} = R_{00}^{-1} (R_{00}^{-1})^{\ast} R_{01}^{-1} | R_{01}^{-1} = \epsilon (\epsilon)^{\ast} a              = a$\
    $R_{10}^{0} = R_{10}^{-1} (R_{00}^{-1})^{\ast} R_{00}^{-1} | R_{10}^{-1} = \oslash  (\epsilon)^{\ast} \oslash        = \oslash$\
    $R_{11}^{0} = R_{10}^{-1} (R_{00}^{-1})^{\ast} R_{01}^{-1} | R_{11}^{-1} = \oslash  (\epsilon)^{\ast} (b|c|\epsilon) = b|c|\epsilon$

- $k = 1$

    $R_{00}^{1} = R_{01}^{0} (R_{11}^{0})^{\ast} R_{00}^{0} | R_{10}^{0} = a              (b|c|\epsilon)^{\ast} \oslash|\epsilon              = \epsilon$\
    $R_{01}^{1} = R_{01}^{0} (R_{11}^{0})^{\ast} R_{01}^{0} | R_{11}^{0} = a              (b|c|\epsilon)^{\ast} (b|c|\epsilon)|a              = a(b|c)^{\ast}$\
    $R_{10}^{1} = R_{11}^{0} (R_{11}^{0})^{\ast} R_{00}^{0} | R_{10}^{0} = (b|c|\epsilon) (b|c|\epsilon)^{\ast} \oslash|\oslash               = \oslash$\
    $R_{11}^{1} = R_{11}^{0} (R_{11}^{0})^{\ast} R_{01}^{0} | R_{11}^{0} = (b|c|\epsilon) (b|c|\epsilon)^{\ast} (b|c|\epsilon)|(b|c|\epsilon) = (b|c)^{\ast}$

最后我们使用 Kleene 闭包重建出来的正则表达式 $R_{01}^{1} = a(b|c)^{\ast}$ 与原表达式相同，证明新的 DFA 与原正则表达式为等价。

> 示例2: $(a|b)^{*}abb$

![](https://picures.oss-cn-beijing.aliyuncs.com/img/min_dfa_sample_2_retag.png)

- 初始化状态：$k = -1$

    $R_{00}^{-1} = b|\epsilon$\
    $R_{01}^{-1} = a$\
    $R_{02}^{-1} = \oslash$\
    $R_{03}^{-1} = \oslash$\
    $R_{10}^{-1} = \oslash$\
    $R_{11}^{-1} = a|\epsilon$\
    $R_{12}^{-1} = b$\
    $R_{13}^{-1} = \oslash$\
    $R_{20}^{-1} = \oslash$\
    $R_{21}^{-1} = a$\
    $R_{22}^{-1} = \epsilon$\
    $R_{23}^{-1} = b$\
    $R_{30}^{-1} = b$\
    $R_{31}^{-1} = a$\
    $R_{32}^{-1} = \oslash$\
    $R_{33}^{-1} = \epsilon$

- $k = 0$

    $R_{00}^{0} = R_{00}^{-1} (R_{00}^{-1})^{\ast} R_{00}^{-1} | R_{00}^{-1} = (b|\epsilon) (b|\epsilon)^{\ast} (b|\epsilon) | (b|\epsilon) = b^{\ast}$\
    $R_{01}^{0} = R_{00}^{-1} (R_{00}^{-1})^{\ast} R_{01}^{-1} | R_{01}^{-1} = (b|\epsilon) (b|\epsilon)^{\ast} a            | a            = b^{\ast}a$\
    $R_{02}^{0} = R_{00}^{-1} (R_{00}^{-1})^{\ast} R_{02}^{-1} | R_{02}^{-1} = (b|\epsilon) (b|\epsilon)^{\ast} \oslash      | \oslash      = \oslash$\
    $R_{03}^{0} = R_{00}^{-1} (R_{00}^{-1})^{\ast} R_{03}^{-1} | R_{03}^{-1} = (b|\epsilon) (b|\epsilon)^{\ast} \oslash      | \oslash      = \oslash$\
    $R_{10}^{0} = R_{10}^{-1} (R_{00}^{-1})^{\ast} R_{00}^{-1} | R_{10}^{-1} = \oslash      (b|\epsilon)^{\ast} (b|\epsilon) | \oslash      = \oslash$\
    $R_{11}^{0} = R_{10}^{-1} (R_{00}^{-1})^{\ast} R_{01}^{-1} | R_{11}^{-1} = \oslash      (b|\epsilon)^{\ast} a            | (a|\epsilon) = a|\epsilon$\
    $R_{12}^{0} = R_{10}^{-1} (R_{00}^{-1})^{\ast} R_{02}^{-1} | R_{12}^{-1} = \oslash      (b|\epsilon)^{\ast} \oslash      | b            = b$\
    $R_{13}^{0} = R_{10}^{-1} (R_{00}^{-1})^{\ast} R_{03}^{-1} | R_{13}^{-1} = \oslash      (b|\epsilon)^{\ast} \oslash      | \oslash      = \oslash$\
    $R_{20}^{0} = R_{20}^{-1} (R_{00}^{-1})^{\ast} R_{00}^{-1} | R_{20}^{-1} = \oslash      (b|\epsilon)^{\ast} (b|\epsilon) | \oslash      = \oslash$\
    $R_{21}^{0} = R_{20}^{-1} (R_{00}^{-1})^{\ast} R_{01}^{-1} | R_{21}^{-1} = \oslash      (b|\epsilon)^{\ast} a            | a            = a$\
    $R_{22}^{0} = R_{20}^{-1} (R_{00}^{-1})^{\ast} R_{02}^{-1} | R_{22}^{-1} = \oslash      (b|\epsilon)^{\ast} \oslash      | \epsilon     = \epsilon$\
    $R_{23}^{0} = R_{20}^{-1} (R_{00}^{-1})^{\ast} R_{03}^{-1} | R_{23}^{-1} = \oslash      (b|\epsilon)^{\ast} \oslash      | b            = b$\
    $R_{30}^{0} = R_{30}^{-1} (R_{00}^{-1})^{\ast} R_{00}^{-1} | R_{30}^{-1} = b            (b|\epsilon)^{\ast} (b|\epsilon) | b            = b^{\ast}$\
    $R_{31}^{0} = R_{30}^{-1} (R_{00}^{-1})^{\ast} R_{01}^{-1} | R_{31}^{-1} = b            (b|\epsilon)^{\ast} a            | a            = b^{\ast}a$\
    $R_{32}^{0} = R_{30}^{-1} (R_{00}^{-1})^{\ast} R_{02}^{-1} | R_{32}^{-1} = b            (b|\epsilon)^{\ast} \oslash      | \oslash      = \oslash$\
    $R_{33}^{0} = R_{30}^{-1} (R_{00}^{-1})^{\ast} R_{03}^{-1} | R_{33}^{-1} = b            (b|\epsilon)^{\ast} \oslash      | \epsilon     = \epsilon$

- $k = 1$

    $R_{00}^{1} = R_{01}^{0} (R_{11}^{0})^{\ast} R_{10}^{0} | R_{00}^{0} = (b^{\ast}a)  (a|\epsilon)^{\ast} \oslash      | b^{\ast}     = b^{\ast}$\
    $R_{01}^{1} = R_{01}^{0} (R_{11}^{0})^{\ast} R_{11}^{0} | R_{01}^{0} = (b^{\ast}a)  (a|\epsilon)^{\ast} (a|\epsilon) | (b^{\ast}a)  = b^{\ast}a^{+}$\
    $R_{02}^{1} = R_{01}^{0} (R_{11}^{0})^{\ast} R_{12}^{0} | R_{02}^{0} = (b^{\ast}a)  (a|\epsilon)^{\ast} b            | \oslash      = b^{\ast}a^{+}b$\
    $R_{03}^{1} = R_{01}^{0} (R_{11}^{0})^{\ast} R_{13}^{0} | R_{03}^{0} = (b^{\ast}a)  (a|\epsilon)^{\ast} \oslash      | \oslash      = \oslash$\
    $R_{10}^{1} = R_{11}^{0} (R_{11}^{0})^{\ast} R_{10}^{0} | R_{10}^{0} = (a|\epsilon) (a|\epsilon)^{\ast} \oslash      | \oslash      = \oslash$\
    $R_{11}^{1} = R_{11}^{0} (R_{11}^{0})^{\ast} R_{11}^{0} | R_{11}^{0} = (a|\epsilon) (a|\epsilon)^{\ast} (a|\epsilon) | (a|\epsilon) = a^{\ast}$\
    $R_{12}^{1} = R_{11}^{0} (R_{11}^{0})^{\ast} R_{12}^{0} | R_{12}^{0} = (a|\epsilon) (a|\epsilon)^{\ast} b            | b            = a^{\ast}b$\
    $R_{13}^{1} = R_{11}^{0} (R_{11}^{0})^{\ast} R_{13}^{0} | R_{13}^{0} = (a|\epsilon) (a|\epsilon)^{\ast} \oslash      | \oslash      = \oslash$\
    $R_{20}^{1} = R_{21}^{0} (R_{11}^{0})^{\ast} R_{10}^{0} | R_{20}^{0} = a            (a|\epsilon)^{\ast} \oslash      | \oslash      = \oslash$\
    $R_{21}^{1} = R_{21}^{0} (R_{11}^{0})^{\ast} R_{11}^{0} | R_{21}^{0} = a            (a|\epsilon)^{\ast} (a|\epsilon) | a            = a^{+}$\
    $R_{22}^{1} = R_{21}^{0} (R_{11}^{0})^{\ast} R_{12}^{0} | R_{22}^{0} = a            (a|\epsilon)^{\ast} b            | \epsilon     = a^{+}b|\epsilon$\
    $R_{23}^{1} = R_{21}^{0} (R_{11}^{0})^{\ast} R_{13}^{0} | R_{23}^{0} = a            (a|\epsilon)^{\ast} \oslash      | b            = b$\
    $R_{30}^{1} = R_{31}^{0} (R_{11}^{0})^{\ast} R_{10}^{0} | R_{30}^{0} = (b^{\ast}a)  (a|\epsilon)^{\ast} \oslash      | b^{\ast}     = b^{\ast}$\
    $R_{31}^{1} = R_{31}^{0} (R_{11}^{0})^{\ast} R_{11}^{0} | R_{31}^{0} = (b^{\ast}a)  (a|\epsilon)^{\ast} (a|\epsilon) | (b^{\ast}a)  = b^{\ast}a^{+}$\
    $R_{32}^{1} = R_{31}^{0} (R_{11}^{0})^{\ast} R_{12}^{0} | R_{32}^{0} = (b^{\ast}a)  (a|\epsilon)^{\ast} b            | \oslash      = b^{\ast}a^{+}b$\
    $R_{33}^{1} = R_{31}^{0} (R_{11}^{0})^{\ast} R_{13}^{0} | R_{33}^{0} = (b^{\ast}a)  (a|\epsilon)^{\ast} \oslash      | \epsilon     = \epsilon$

- $k = 2$

    $R_{00}^{2} = R_{02}^{1} (R_{22}^{1})^{\ast} R_{20}^{1} | R_{00}^{1} = (b^{\ast}a^{+}b)  (a^{+}b|\epsilon)^{\ast} \oslash           | b^{\ast}          = b^{\ast}$\
    $R_{01}^{2} = R_{02}^{1} (R_{22}^{1})^{\ast} R_{21}^{1} | R_{01}^{1} = (b^{\ast}a^{+}b)  (a^{+}b|\epsilon)^{\ast} a^{+}             | (b^{\ast}a^{+})   = b^{\ast}a^{+}((ba^{+})^{+}|\epsilon)$\
    $R_{02}^{2} = R_{02}^{1} (R_{22}^{1})^{\ast} R_{22}^{1} | R_{02}^{1} = (b^{\ast}a^{+}b)  (a^{+}b|\epsilon)^{\ast} (a^{+}b|\epsilon) | (b^{\ast}a^{+}b)  = b^{\ast}(a^{+}b)^{+}$\
    $R_{03}^{2} = R_{02}^{1} (R_{22}^{1})^{\ast} R_{23}^{1} | R_{03}^{1} = (b^{\ast}a^{+}b)  (a^{+}b|\epsilon)^{\ast} b                 | \oslash           = b^{\ast}(a^{+}b)^{+}b$\
    $R_{10}^{2} = R_{12}^{1} (R_{22}^{1})^{\ast} R_{20}^{1} | R_{10}^{1} = (a^{\ast}b)       (a^{+}b|\epsilon)^{\ast} \oslash           | \oslash           = \oslash$\
    $R_{11}^{2} = R_{12}^{1} (R_{22}^{1})^{\ast} R_{21}^{1} | R_{11}^{1} = (a^{\ast}b)       (a^{+}b|\epsilon)^{\ast} a^{+}             | a^{\ast}          = a^{\ast}((ba^{+})^{+}|\epsilon)$\
    $R_{12}^{2} = R_{12}^{1} (R_{22}^{1})^{\ast} R_{22}^{1} | R_{12}^{1} = (a^{\ast}b)       (a^{+}b|\epsilon)^{\ast} (a^{+}b|\epsilon) | (a^{\ast}b)       = a^{\ast}b(a^{+}b)^{\ast}$\
    $R_{13}^{2} = R_{12}^{1} (R_{22}^{1})^{\ast} R_{23}^{1} | R_{13}^{1} = (a^{\ast}b)       (a^{+}b|\epsilon)^{\ast} b                 | \oslash           = a^{\ast}b(a^{+}b)^{\ast}b$\
    $R_{20}^{2} = R_{22}^{1} (R_{22}^{1})^{\ast} R_{20}^{1} | R_{20}^{1} = (a^{+}b|\epsilon) (a^{+}b|\epsilon)^{\ast} \oslash           | \oslash           = \oslash$\
    $R_{21}^{2} = R_{22}^{1} (R_{22}^{1})^{\ast} R_{21}^{1} | R_{21}^{1} = (a^{+}b|\epsilon) (a^{+}b|\epsilon)^{\ast} a^{+}             | a^{+}             = (a^{+}b)^{\ast}a^{+}$\
    $R_{22}^{2} = R_{22}^{1} (R_{22}^{1})^{\ast} R_{22}^{1} | R_{22}^{1} = (a^{+}b|\epsilon) (a^{+}b|\epsilon)^{\ast} (a^{+}b|\epsilon) | (a^{+}b|\epsilon) = (a^{+}b)^{\ast}$\
    $R_{23}^{2} = R_{22}^{1} (R_{22}^{1})^{\ast} R_{23}^{1} | R_{23}^{1} = (a^{+}b|\epsilon) (a^{+}b|\epsilon)^{\ast} b                 | b                 = (a^{+}b)^{\ast}b$\
    $R_{30}^{2} = R_{32}^{1} (R_{22}^{1})^{\ast} R_{20}^{1} | R_{30}^{1} = (b^{\ast}a^{+}b)  (a^{+}b|\epsilon)^{\ast} \oslash           | b^{\ast}          = b^{\ast}$\
    $R_{31}^{2} = R_{32}^{1} (R_{22}^{1})^{\ast} R_{21}^{1} | R_{31}^{1} = (b^{\ast}a^{+}b)  (a^{+}b|\epsilon)^{\ast} a^{+}             | (b^{\ast}a^{+})   = b^{\ast}(a^{+}b)^{\ast}a^{+}$\
    $R_{32}^{2} = R_{32}^{1} (R_{22}^{1})^{\ast} R_{22}^{1} | R_{32}^{1} = (b^{\ast}a^{+}b)  (a^{+}b|\epsilon)^{\ast} (a^{+}b|\epsilon) | (b^{\ast}a^{+}b)  = b^{\ast}(a^{+}b)^{+}$\
    $R_{33}^{2} = R_{32}^{1} (R_{22}^{1})^{\ast} R_{23}^{1} | R_{33}^{1} = (b^{\ast}a^{+}b)  (a^{+}b|\epsilon)^{\ast} b                 | \epsilon          = b^{\ast}(a^{+}b)^{+}|\epsilon$

- $k = 3$

    $R_{00}^{3} = R_{03}^{2} (R_{33}^{2})^{\ast} R_{30}^{2} | R_{00}^{2} = (b^{\ast}(a^{+}b)^{+}b)         (b^{\ast}(a^{+}b)^{+}|\epsilon)^{\ast} b^{\ast}                        | b^{\ast}$\
    $R_{01}^{3} = R_{03}^{2} (R_{33}^{2})^{\ast} R_{31}^{2} | R_{01}^{2} = (b^{\ast}(a^{+}b)^{+}b)         (b^{\ast}(a^{+}b)^{+}|\epsilon)^{\ast} b^{\ast}(a^{+}b)^{\ast}a^{+}    | b^{\ast}a^{+}((ba^{+})^{+}|\epsilon)$\
    $R_{02}^{3} = R_{03}^{2} (R_{33}^{2})^{\ast} R_{32}^{2} | R_{02}^{2} = (b^{\ast}(a^{+}b)^{+}b)         (b^{\ast}(a^{+}b)^{+}|\epsilon)^{\ast} b^{\ast}(a^{+}b)^{+}            | b^{\ast}(a^{+}b)^{+}$\
    $R_{03}^{3} = R_{03}^{2} (R_{33}^{2})^{\ast} R_{33}^{2} | R_{03}^{2} = (b^{\ast}(a^{+}b)^{+}b)         (b^{\ast}(a^{+}b)^{+}|\epsilon)^{\ast} (b^{\ast}(a^{+}b)^{+}|\epsilon) | b^{\ast}(a^{+}b)^{+}b$\
    $R_{10}^{3} = R_{13}^{2} (R_{33}^{2})^{\ast} R_{30}^{2} | R_{10}^{2} = (a^{\ast}b(a^{+}b)^{\ast}b)     (b^{\ast}(a^{+}b)^{+}|\epsilon)^{\ast} b^{\ast}                        | \oslash$\
    $R_{11}^{3} = R_{13}^{2} (R_{33}^{2})^{\ast} R_{31}^{2} | R_{11}^{2} = (a^{\ast}b(a^{+}b)^{\ast}b)     (b^{\ast}(a^{+}b)^{+}|\epsilon)^{\ast} b^{\ast}(a^{+}b)^{\ast}a^{+}    | a^{\ast}((ba^{+})^{+}|\epsilon)$\
    $R_{12}^{3} = R_{13}^{2} (R_{33}^{2})^{\ast} R_{32}^{2} | R_{12}^{2} = (a^{\ast}b(a^{+}b)^{\ast}b)     (b^{\ast}(a^{+}b)^{+}|\epsilon)^{\ast} b^{\ast}(a^{+}b)^{+}            | a^{\ast}b(a^{+}b)^{\ast}$\
    $R_{13}^{3} = R_{13}^{2} (R_{33}^{2})^{\ast} R_{33}^{2} | R_{13}^{2} = (a^{\ast}b(a^{+}b)^{\ast}b)     (b^{\ast}(a^{+}b)^{+}|\epsilon)^{\ast} (b^{\ast}(a^{+}b)^{+}|\epsilon) | a^{\ast}b(a^{+}b)^{\ast}b$\
    $R_{20}^{3} = R_{23}^{2} (R_{33}^{2})^{\ast} R_{30}^{2} | R_{20}^{2} = (a^{+}b)^{\ast}b                (b^{\ast}(a^{+}b)^{+}|\epsilon)^{\ast} b^{\ast}                        | \oslash$\
    $R_{21}^{3} = R_{23}^{2} (R_{33}^{2})^{\ast} R_{31}^{2} | R_{21}^{2} = (a^{+}b)^{\ast}b                (b^{\ast}(a^{+}b)^{+}|\epsilon)^{\ast} b^{\ast}(a^{+}b)^{\ast}a^{+}    | (a^{+}b)^{\ast}a^{+}$\
    $R_{22}^{3} = R_{23}^{2} (R_{33}^{2})^{\ast} R_{32}^{2} | R_{22}^{2} = (a^{+}b)^{\ast}b                (b^{\ast}(a^{+}b)^{+}|\epsilon)^{\ast} b^{\ast}(a^{+}b)^{+}            | (a^{+}b)^{\ast}$\
    $R_{23}^{3} = R_{23}^{2} (R_{33}^{2})^{\ast} R_{33}^{2} | R_{23}^{2} = (a^{+}b)^{\ast}b                (b^{\ast}(a^{+}b)^{+}|\epsilon)^{\ast} (b^{\ast}(a^{+}b)^{+}|\epsilon) | (a^{+}b)^{\ast}b$\
    $R_{30}^{3} = R_{33}^{2} (R_{33}^{2})^{\ast} R_{30}^{2} | R_{30}^{2} = (b^{\ast}(a^{+}b)^{+}|\epsilon) (b^{\ast}(a^{+}b)^{+}|\epsilon)^{\ast} b^{\ast}                        | b^{\ast}$\
    $R_{31}^{3} = R_{33}^{2} (R_{33}^{2})^{\ast} R_{31}^{2} | R_{31}^{2} = (b^{\ast}(a^{+}b)^{+}|\epsilon) (b^{\ast}(a^{+}b)^{+}|\epsilon)^{\ast} b^{\ast}(a^{+}b)^{\ast}a^{+}    | b^{\ast}(a^{+}b)^{\ast}a^{+}$\
    $R_{32}^{3} = R_{33}^{2} (R_{33}^{2})^{\ast} R_{32}^{2} | R_{32}^{2} = (b^{\ast}(a^{+}b)^{+}|\epsilon) (b^{\ast}(a^{+}b)^{+}|\epsilon)^{\ast} b^{\ast}(a^{+}b)^{+}            | b^{\ast}(a^{+}b)^{+}$\
    $R_{33}^{3} = R_{33}^{2} (R_{33}^{2})^{\ast} R_{33}^{2} | R_{33}^{2} = (b^{\ast}(a^{+}b)^{+}|\epsilon) (b^{\ast}(a^{+}b)^{+}|\epsilon)^{\ast} (b^{\ast}(a^{+}b)^{+}|\epsilon) | b^{\ast}(a^{+}b)^{+}|\epsilon$

最后的 $k = 3$ 就不展开了，我们只需要简化 $R_{03}^{3}$ 就可以了：\
由 $R_{03}^{3} = (b^{\ast}(a^{+}b)^{+}b)(b^{\ast}(a^{+}b)^{+}|\epsilon)^{\ast}(b^{\ast}(a^{+}b)^{+}|\epsilon)|b^{\ast}(a^{+}b)^{+}b$\
$= b^{\ast}(a^{+}b)^{+}b(b^{\ast}(a^{+}b)^{+})^{\ast}$\
$= b^{\ast}a^{\ast}ab(a^{\ast}ab)^{\ast}b(b^{\ast}a^{\ast}ab(a^{\ast}ab)^{\ast})^{\ast}$\
$= b^{\ast}(a^{\ast}ab)^{\ast}a^{\ast}abb(b^{\ast}(a^{\ast}ab)^{\ast}a^{\ast}ab)^{\ast}$\
$= (a|b)^{*}abb$

有点偷吃步hhh，不过反正结果上是一样的，这样的 Kleene 闭包也是可以自动化的所以不需要过于纠结必须用手算hh。

## 最终版本的 DFA

到此就全部完成啦，我们经由 RE -> NFA -> DFA -> 最小化 DFA -> 验证 DFA 这几个步骤，终于构建出了最终版本的 DFA 了，撒花～

> 示例1: $a(b|c)^{*}$

![](https://picures.oss-cn-beijing.aliyuncs.com/img/min_dfa_sample_1_retag.png)

> 示例2: $(a|b)^{*}abb$

![](https://picures.oss-cn-beijing.aliyuncs.com/img/min_dfa_sample_2_retag.png)


# 结语

从 RE 构建 DFA 的三部曲就到这边结束啦，由于最后的`验证 DFA`使用 Kleene 闭包其实比较繁琐，读者可以尝试自己实现自动化这个过程的程序（作者估计是写不出来了hhh）。

