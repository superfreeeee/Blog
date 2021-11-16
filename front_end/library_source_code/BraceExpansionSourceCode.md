# brace-expansion 源码解析

@[TOC](文章目录)

<!-- TOC -->

- [brace-expansion 源码解析](#brace-expansion-源码解析)
- [正文](#正文)
  - [0. 基本信息](#0-基本信息)
  - [1. 源码解析](#1-源码解析)
    - [1.0 代码结构](#10-代码结构)
    - [1.1 主入口 expandTop](#11-主入口-expandtop)
    - [1.2 转译/反转译 escapeBraces、unescapeBraces](#12-转译反转译-escapebracesunescapebraces)
    - [1.3 递归展开 expand](#13-递归展开-expand)
      - [1.3.1 不存在 {}](#131-不存在-)
      - [1.3.2 忽略 ${}](#132-忽略-)
      - [1.3.3 展开 {}](#133-展开-)
      - [1.3.4 非序列 => 保留原字符串](#134-非序列--保留原字符串)
      - [1.3.5 捕捉序列参数](#135-捕捉序列参数)
      - [1.3.6 展开序列](#136-展开序列)
      - [1.3.7 返回结果](#137-返回结果)
  - [2. 小结](#2-小结)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 正文

## 0. 基本信息

- version：`v2.0.1`
- 核心功能：模仿 Bash 程序规范的 brace expansion 特性

示例可以参阅 README（[传送门：brace-expansion - npm](https://www.npmjs.com/package/brace-expansion)）

## 1. 源码解析

一样先来看看项目结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/brace_expansion_source_code_0_structure.png)

也是单一个 `index.js`

### 1.0 代码结构

接下来看进去代码结构

- `index.js`（阅读笔记：`/index.js/0_structure.js`）

```js
const balanced = require('balanced-match');

function numeric(str) {}

// 替换省略字符
function escapeBraces(str) {}

// 换回来呵呵
function unescapeBraces(str) {}

// 匹配 {,,,} 模式
function parseCommaParts(str) {}

// 顶层入口函数
function expandTop(str) {}

// str => {str}
function embrace(str) {}

// 是否存在前导 0
function isPadded(el) {}

// i <= y
function lte(i, y) {}

// i >= y
function gte(i, y) {}

// 递归展开
function expand(str, isTop) {}

module.exports = expandTop;
```

可以看到主入口就是一个 `expandTop`，马上继续探进去

### 1.1 主入口 expandTop

- `index.js`（阅读笔记：`/index.js/1_expandTop.js`）

```js
/**
 * @param {string} str
 */
/* 主入口 */
function expandTop(str) {
  if (!str) return [];

  // I don't know why Bash 4.3 does this, but it does.
  // Anything starting with {} will have the first two bytes preserved
  // but *only* at the top level, so {},a}b will not expand to anything,
  // but a{},b}c will be expanded to [a}c,abc].
  // One could argue that this is a bug in Bash, but since the goal of
  // this module is to match Bash's rules, we escape a leading {}

  // 忽略了 {} 作为起始字符
  if (str.slice(0, 2) === '{}') {
    str = '\\{\\}' + str.slice(2);
  }

  return expand(escapeBraces(str), true).map(unescapeBraces);
}
```

作者也是很逗hh留了段注释，但是实际上测试发现，不管是空 `{}` 还是 `{aaa}` 没有逗号分离，不管放在什么位置其实都会被留下来的说，因此这个转译实际上有点多余了

### 1.2 转译/反转译 escapeBraces、unescapeBraces

接下来作者用上了两个方法来进行特殊字符的转译和反转译

- `index.js`（阅读笔记：`/index.js/2_escapeBraces.js`）

```js
const escSlash = '\0SLASH' + Math.random() + '\0';
const escOpen = '\0OPEN' + Math.random() + '\0';
const escClose = '\0CLOSE' + Math.random() + '\0';
const escComma = '\0COMMA' + Math.random() + '\0';
const escPeriod = '\0PERIOD' + Math.random() + '\0';

/**
 * @param {string} str
 */
/* 替换特殊字符 */
function escapeBraces(str) {
  return str.split('\\\\').join(escSlash)
            .split('\\{').join(escOpen)
            .split('\\}').join(escClose)
            .split('\\,').join(escComma)
            .split('\\.').join(escPeriod);
}
```

将字符串中带 `\` 的转译字符换掉，解析完成之后再用下面的方法转译回来

- `index.js`（阅读笔记：`/index.js/10_unescapeBraces.js`）

```js
/**
 * @param {string} str
 */
/* 把省略字符换回来呵呵 */
function unescapeBraces(str) {
  return str.split(escSlash).join('\\')
            .split(escOpen).join('{')
            .split(escClose).join('}')
            .split(escComma).join(',')
            .split(escPeriod).join('.');
}
```

### 1.3 递归展开 expand

下面就是整个库的核心方法：`expand`，递归展开不同层级的 `{}` 括号并返回字符串数组

- `index.js`（阅读笔记：`/index.js/3_expand.js`）

整个方法需要拆分好几段，要看完整的源码笔记把最下面的链接代码下下来吧！

#### 1.3.1 不存在 {}

```js
/**
 * @param {string} str
 * @param {boolean} [isTop]
 */
function expand(str, isTop) {
  /** @type {string[]} */
  const expansions = [];

  // 检查是否存在顶层花括号
  const m = balanced('{', '}', str);

  // 1. 不存在顶层花括号 => 直接返回原字符串
  if (!m) return [str];
```

第一种情况考虑有没有 `{}` 需要展开的部分

#### 1.3.2 忽略 ${}

接下来如果存在 `{}` 但是是 `${}` 的形式，还是忽略并直接返回（参考 Bash 规范标准）

```js
  // no need to expand pre, since it is guaranteed to be free of brace-sets
  const pre = m.pre;
  const post = m.post.length ? expand(m.post, false) : ['']; // 括号后内容递归展开

  if (/\$$/.test(m.pre)) {
    // 2.1 ${} 的组合不展开 body
    for (let k = 0; k < post.length; k++) {
      const expansion = pre + '{' + m.body + '}' + post[k];
      expansions.push(expansion);
    }
```

#### 1.3.3 展开 {}

其他情况就是说确实存在一个 `{}` 对需要展开

```js
    // 3. 展开 {} 的内容
    const isNumericSequence = /^-?\d+\.\.-?\d+(?:\.\.-?\d+)?$/.test(m.body); // x..y or x..y..z
    const isAlphaSequence = /^[a-zA-Z]\.\.[a-zA-Z](?:\.\.-?\d+)?$/.test(m.body); // a..b or a..b..c
    const isSequence = isNumericSequence || isAlphaSequence; // 以上两种形式
    const isOptions = m.body.indexOf(',') >= 0; // 是否存在 ,
```

首先先检查要展开的内容(`m.body`)是哪种形式：可以是 `{x..y..z}`，也可以是 `{a,b,c}`

#### 1.3.4 非序列 => 保留原字符串

```js
    if (!isSequence && !isOptions) {
      // 3.1 不是 seq 也不是 option => 就一个内容

      // {a},b}
      // 兼容 {a},b} 的写法（老实说我觉得不该兼容hh）
      if (m.post.match(/,.*\}/)) {
        // 忽略 当前后括号，回填 escClose 并递归调用
        str = m.pre + '{' + m.body + escClose + m.post;
        return expand(str);
      }

      // 无需兼容的情况就是返回当前字符串 {} 内容直接保留
      return [str];
    }
```

如果两种序列都不是，表示 body 部分只有一个字符串，根据 bash 的规则应该保留后返回

#### 1.3.5 捕捉序列参数

确定至少是其中一种序列之后就要展开内容，首先先展开序列本身，确定两种情况下的序列参数

```js
    // 以下必为 seq 或 option 其中一个
    // 4. 拆分 .. 或是 ,
    let n;
    if (isSequence) {
      // seq 拆分 {a..b..c}
      n = m.body.split(/\.\./);
    } else {
      // options 拆分 {a,b,c}
      n = parseCommaParts(m.body);
      if (n.length === 1) {
        // x{{a,b}}y ==> x{a}y x{b}y
        // 兼容上面这种的嵌套 => 重新展开 n[0]，然后回填 {}
        n = expand(n[0], false).map(embrace);
        if (n.length === 1) {
          return post.map(function (p) {
            return m.pre + n[0] + p;
          });
        }
      }
    }
```

- 对于数字序列：`n = [x, y, z]`
- 对于字母序列：`n = [a, b, c, ...]`

#### 1.3.6 展开序列

有了参数就可以展开序列了，首先是对于数字序列

```js
    let N;

    if (isSequence) {
      // 5.1 a..b..c 形式的展开
      const x = numeric(n[0]);
      const y = numeric(n[1]);
      const width = Math.max(n[0].length, n[1].length); // 保留数字宽度
      let incr = n.length == 3 ? Math.abs(numeric(n[2])) : 1; // 是否存在 step 参数
      let test = lte;
      const reverse = y < x; // 倒序
      if (reverse) {
        incr *= -1;
        test = gte;
      }
      const pad = n.some(isPadded); // 需不需要前导 0

      N = [];
      // 从 x -> y，incr 递增步长、test 结束测试
      for (let i = x; test(i, y); i += incr) {
        let c;
        if (isAlphaSequence) {
          // 2.2.3.1 字母序列
          c = String.fromCharCode(i);
          if (c === '\\') c = '';
        } else {
          // 2.2.3.2 数字序列
          c = String(i);
          if (pad) {
            const need = width - c.length;
            // 补 0
            if (need > 0) {
              const z = new Array(need + 1).join('0');
              if (i < 0) c = '-' + z + c.slice(1);
              else c = z + c;
            }
          }
        }
        N.push(c);
      }
    }
```

先转化为数字/字母ascii码，然后遍历(`incr` 为步长、`test` 为终止测试)，然后一步步的将结果推到 N 里面

对于 `,` 序列就比较简单，直接写入就可以了

```js
    else {
      // 5.2 a,b,c 形式的展开
      N = [];

      for (let j = 0; j < n.length; j++) {
        N.push.apply(N, expand(n[j], false));
      }
    }
```

#### 1.3.7 返回结果

最后我们只需要把 N 的结果与 pre 拼接一下就是我们的输出了

```js
    // 6. N 为 body 展开后的字符串序列
    for (let j = 0; j < N.length; j++) {
      for (let k = 0; k < post.length; k++) {
        const expansion = pre + N[j] + post[k];
        // 顶层 && 非字母序列 && 字符串为空的时候不用填入
        if (!isTop || isSequence || expansion) expansions.push(expansion);
      }
    }
  }

  return expansions;
}
```

## 2. 小结

整个库的目的在于复现 Bash 规范的 brace-expansion 特性，所以比较多的字符串操作，整体来说非常的琐碎hh，不过也展示出编程时应该留意各处的细节，包括对于异常流程的处理把控。

# 其他资源

## 参考连接

| Title                    | Link                                                                                               |
| ------------------------ | -------------------------------------------------------------------------------------------------- |
| brace-expansion - npm    | [https://www.npmjs.com/package/brace-expansion](https://www.npmjs.com/package/brace-expansion)     |
| brace-expansion - Github | [https://github.com/juliangruber/brace-expansion](https://github.com/juliangruber/brace-expansion) |

## 阅读笔记参考

[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/brace-expansion-2.0.1](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/brace-expansion-2.0.1)
