# minimatch 源码解析

@[TOC](文章目录)

<!-- TOC -->

- [minimatch 源码解析](#minimatch-源码解析)
- [正文](#正文)
  - [0. 基本信息](#0-基本信息)
  - [1. 源码解析](#1-源码解析)
    - [1.1 项目展开](#11-项目展开)
    - [1.2 Minimatch](#12-minimatch)
      - [1.2.1 构造函数](#121-构造函数)
      - [1.2.2 make 初始化表达式解析](#122-make-初始化表达式解析)
      - [1.2.3 裁剪前导 !](#123-裁剪前导-)
      - [1.2.4 花括号展开](#124-花括号展开)
      - [1.2.5 分段 & parse 正则表达式转换](#125-分段--parse-正则表达式转换)
      - [1.2.6 正则表达式转换：第一部分 - 初始化状态](#126-正则表达式转换第一部分---初始化状态)
      - [1.2.7 正则表达式转换：第二部分 - 循环解析](#127-正则表达式转换第二部分---循环解析)
        - [1.2.7.1 状态扩展符](#1271-状态扩展符)
        - [1.2.7.2 分组符号](#1272-分组符号)
        - [1.2.7.3 或符号](#1273-或符号)
        - [1.2.7.4 类运算符](#1274-类运算符)
        - [1.2.7.5 一般字符](#1275-一般字符)
      - [1.2.8 正则表达式转换：第三部分 - 残局处理](#128-正则表达式转换第三部分---残局处理)
        - [1.2.8.1 未闭合情况处理](#1281-未闭合情况处理)
        - [1.2.8.2 转义标志](#1282-转义标志)
        - [1.2.8.3 反义表达式回看](#1283-反义表达式回看)
        - [1.2.8.4 正则表达式生成](#1284-正则表达式生成)
      - [1.2.9 正则表达式组生成](#129-正则表达式组生成)
      - [1.2.10 makeRe 生成完整表达式](#1210-makere-生成完整表达式)
      - [1.2.11 match 匹配字符串](#1211-match-匹配字符串)
      - [1.2.12 matchOne 单一模式匹配](#1212-matchone-单一模式匹配)
    - [1.3 minimatch](#13-minimatch)
      - [1.3.1 filter 过滤函数](#131-filter-过滤函数)
      - [1.3.2 braceExpand 花括号扩展](#132-braceexpand-花括号扩展)
      - [1.3.3 makeRe 创建正则表达式](#133-makere-创建正则表达式)
      - [1.3.4 match 匹配方法](#134-match-匹配方法)
    - [1.4 默认配置](#14-默认配置)
      - [1.4.1 Minimatch.defaults](#141-minimatchdefaults)
      - [1.4.2 minimatch.defaults](#142-minimatchdefaults)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 正文

## 0. 基本信息

- version：`v3.0.4`
- 功能：花括号展开（brace-expansion）、glob 表达式匹配、globstar 扩展（**）
- Npm Repository: [https://www.npmjs.com/package/minimatch](https://www.npmjs.com/package/minimatch)

## 1. 源码解析

minimatch 的内容还是比较多，主要为基于 js 对 Bash 的 glob 解析的复现，同时顺便学习到底 glob 表达式是如何解析的

![](https://picures.oss-cn-beijing.aliyuncs.com/img/minimatch_source_code_0_structure.png)

核心代码集中在 `minimatch.js` 单文件中，我们继续往下看

### 1.1 项目展开

首先先来看一下 `minimatch.js` 文件展开的大致结构

- `minimatch.js`（阅读笔记：`/minimatch.js/0_structure.js`）

```js
module.exports = minimatch;
minimatch.Minimatch = Minimatch;

var path = { sep: '/' };
try {
  path = require('path');
} catch (er) {}

var GLOBSTAR = (minimatch.GLOBSTAR = Minimatch.GLOBSTAR = {});
var expand = require('brace-expansion');

/* 2_4_parse */
function charSet(s) {}

/* 1_1_filter */
minimatch.filter = filter;
function filter(pattern, options) {}

/* 3_3_ext */
function ext(a, b) {}

/* 1_0_defaults */
minimatch.defaults = function (def) {};

/* 2_0_defaults */
Minimatch.defaults = function (def) {};

/* 1_minimatch */
function minimatch(p, pattern, options) {}

/* 2_Minimatch */
function Minimatch(pattern, options) {}

/* 2_1_make */
Minimatch.prototype.debug = function () {};

/* 2_1_make */
Minimatch.prototype.make = make;
function make() {}

/* 2_2_parseNegate */
Minimatch.prototype.parseNegate = parseNegate;
function parseNegate() {}

/* 1_2_braceExpand */
minimatch.braceExpand = function (pattern, options) {};

/* 2_3_braceExpand */
Minimatch.prototype.braceExpand = braceExpand;
function braceExpand(pattern, options) {}

/* 2_4_parse */
Minimatch.prototype.parse = parse;
function parse(pattern, isSub) {}

/* 1_3_makeRe */
minimatch.makeRe = function (pattern, options) {};

/* 2_5_makeRe */
Minimatch.prototype.makeRe = makeRe;
function makeRe() {}

/* 1_4_match */
minimatch.match = function (list, pattern, options) {};

/* 2_6_match */
Minimatch.prototype.match = match;
function match(f, partial) {}

/* 2_7_matchOne */
Minimatch.prototype.matchOne = function (file, pattern, partial) {};

/* 3_1_globUnescape */
function globUnescape(s) {}

/* 3_2_regExpEscape */
function regExpEscape(s) {}
```

注释附上每个方法对应的阅读笔记文件

### 1.2 Minimatch

虽然 readme 推荐我们直接用默认的 minimatch 函数，但是实际上他就是 Minimatch 类再套一层壳，所以我们先直接进入到核心类的定义和实现

#### 1.2.1 构造函数

- `minimatch.js`（阅读笔记：`/minimatch.js/2_Minimatch.js`）

```js
/* 核心类 */
function Minimatch(pattern, options) {
  if (!(this instanceof Minimatch)) {
    return new Minimatch(pattern, options);
  }

  if (typeof pattern !== 'string') {
    throw new TypeError('glob pattern string required');
  }

  if (!options) options = {};
  pattern = pattern.trim();

  // ========== 以上为参数校验 ==========

  // windows support: need to use /, not \
  if (path.sep !== '/') {
    // 将 sep 改为 /
    pattern = pattern.split(path.sep).join('/');
  }

  this.options = options;
  this.set = [];
  this.pattern = pattern;
  this.regexp = null;
  this.negate = false;
  this.comment = false; // 模式为注释
  this.empty = false; // 模式为空

  // make the set of regexps etc.
  this.make(); // 初始化
}
```

构造函数实现了参数校验、基础属性初始化，然后调用 `make` 方法进行真正的表达式解析初始化

#### 1.2.2 make 初始化表达式解析

整个 make 的逻辑比较长，可能会跨两三个小标题

- `minimatch.js`（阅读笔记：`/minimatch.js/2_1_make.js`）

```js
// normalizes slashes.
var slashSplit = /\/+/;

Minimatch.prototype.debug = function () {};

Minimatch.prototype.make = make;

/* 初始化 Minimatch */
function make() {
  // don't do it more than once.
  if (this._made) return;

  var pattern = this.pattern;
  var options = this.options;

  // empty patterns and comments match nothing.
  // 1. 匹配模式为注释 =>
  if (!options.nocomment && pattern.charAt(0) === '#') {
    this.comment = true;
    return;
  }
  // 2. 匹配模式为空 =>
  if (!pattern) {
    this.empty = true;
    return;
  }
```

首先第一部分先检查两种可以提前结束解析的情况：

1. 表达式为注释
2. 表达式为空

#### 1.2.3 裁剪前导 !

接下来下一步我们需要先进行表达式前导 ! 的裁剪

- `minimatch.js`（阅读笔记：`/minimatch.js/2_1_make.js`）

```js
  // step 1: figure out negation, etc.
  // 3. 裁剪前导 !
  this.parseNegate();
```

- `minimatch.js`（阅读笔记：`/minimatch.js/2_2_parseNegate.js`）

```js
/* 裁剪前导 ! */
Minimatch.prototype.parseNegate = parseNegate;
function parseNegate() {
  var pattern = this.pattern;
  var negate = false;
  var options = this.options;
  var negateOffset = 0;

  // 压制前导 !
  if (options.nonegate) return;

  // 记录 negate 个数
  for (var i = 0, l = pattern.length; i < l && pattern.charAt(i) === '!'; i++) {
    negate = !negate;
    negateOffset++;
  }

  // 裁剪 pattern 前导 !
  if (negateOffset) this.pattern = pattern.substr(negateOffset);
  this.negate = negate;
}
```

本质上就是计算多少个前导 `!`，然后初始化 `this.negate` 标志

#### 1.2.4 花括号展开

接下来在真正开始转换 glob 表达式之前，先完成花括号展开的特性(brace-expansion)，然后初始化 `debug` 输出

- `minimatch.js`（阅读笔记：`/minimatch.js/2_1_make.js`）

```js
  // step 2: expand braces
  // 4. 花括号展开 => [p1, p2, p3, ...]
  var set = (this.globSet = this.braceExpand());

  if (options.debug) this.debug = console.error; // debug 输出，默认为 () => {}

  this.debug(this.pattern, set);
```

- `minimatch.js`（阅读笔记：`/minimatch.js/2_3_braceExpand.js`）

```js
Minimatch.prototype.braceExpand = braceExpand;

/* 花括号扩展(brace-expansion 特性) */
function braceExpand(pattern, options) {
  // 作者你实在是有点懒啊，参数好好的传一下可以吗
  if (!options) {
    if (this instanceof Minimatch) {
      options = this.options;
    } else {
      options = {};
    }
  }

  pattern = typeof pattern === 'undefined' ? this.pattern : pattern;

  if (typeof pattern === 'undefined') {
    throw new TypeError('undefined pattern');
  }

  // ========== 从 Minimatch 实例上抽取参数 ==========

  // options.nobrace: 压制 brace-expansion 特性
  if (options.nobrace || !pattern.match(/\{.*\}/)) {
    // shortcut. no need to expand.
    return [pattern];
  }

  return expand(pattern);
}
```

本质上是借用了 `brace-expansion` 库的功能（[传送门：brace-expansion 源码解析](https://blog.csdn.net/weixin_44691608/article/details/121361324)）

#### 1.2.5 分段 & parse 正则表达式转换

接下来是按 `/` 分段，并将每一段转换成正则表达式

- `minimatch.js`（阅读笔记：`/minimatch.js/2_1_make.js`）

```js
  // step 3: now we have a set, so turn each one into a series of path-portion
  // matching patterns.
  // These will be regexps, except in the case of "**", which is
  // set to the GLOBSTAR object for globstar behavior,
  // and will not contain any / characters
  // 5. 按 / 分开 => [[p1], [p2], [p3], ...]
  set = this.globParts = set.map(function (s) {
    return s.split(slashSplit);
  });

  this.debug(this.pattern, set);

  // glob --> regexps
  // 6. 将 glob 表达式片段转 regexp
  set = set.map(function (s, si, set) {
    return s.map(this.parse, this);
  }, this);

  this.debug(this.pattern, set);
```

#### 1.2.6 正则表达式转换：第一部分 - 初始化状态

整个正则表达式转换比较长(差不多 400 行)，我们分成几块来看

- `minimatch.js`（阅读笔记：`/minimatch.js/2_4_parse.js`）

第一部分是关于转换状态的初始化

```js
var plTypes = {
  '!': { open: '(?:(?!(?:', close: '))[^/]*?)' }, // !(xxx) in glob => (?:(?!(?:xxx))[^/]*?) in reg
  '?': { open: '(?:', close: ')?' }, // ?(xxx) in glob => (?:xxx)? in reg
  '+': { open: '(?:', close: ')+' }, // +(xxx) in glob => (?:xxx)+ in reg
  '*': { open: '(?:', close: ')*' }, // *(xxx) in glob => (?:xxx)* in reg
  '@': { open: '(?:', close: ')' }, // @(xxx) in glob => (?:xxx)@ in reg
};

// any single thing other than /
// don't need to escape / when using new RegExp()
var qmark = '[^/]';

// * => any number of characters
var star = qmark + '*?';

// characters that need to be escaped in RegExp.
var reSpecials = charSet('().*{}+?[]^$\\!');

// "abc" -> { a:true, b:true, c:true }
function charSet(s) {
  return s.split('').reduce(function (set, c) {
    set[c] = true;
    return set;
  }, {});
}
// parse a component of the expanded set.
// At this point, no pattern may contain "/" in it
// so we're going to return a 2d array, where each entry is the full
// pattern, split on '/', and then turned into a regular expression.
// A regexp is made at the end which joins each array with an
// escaped /, and another full one which joins each regexp with |.
//
// Following the lead of Bash 4.1, note that "**" only has special meaning
// when it is the *only* thing in a path portion.  Otherwise, any series
// of * is equivalent to a single *.  Globstar behavior is enabled by
// default, and can be disabled by setting options.noglobstar.
Minimatch.prototype.parse = parse;
var SUBPARSE = {};

/* 解析路径片段 */
function parse(pattern, isSub) {
  // 0. 模式以 64KB 为上限
  if (pattern.length > 1024 * 64) {
    throw new TypeError('pattern is too long');
  }

  var options = this.options;

  // shortcuts
  // 1. 关闭 GLOBSTAR 模式
  if (!options.noglobstar && pattern === '**') return GLOBSTAR;
  if (pattern === '') return '';

  var re = '';
  var hasMagic = !!options.nocase;
  var escaping = false;
  // ? => one single character
  var patternListStack = [];
  var negativeLists = [];
  var stateChar;
  var inClass = false;
  var reClassStart = -1;
  var classStart = -1;
  // . and .. never match anything that doesn't start with .,
  // even when options.dot is set.
  // 2. 处理 glob 字符串的特殊起始字符
  var patternStart =
    pattern.charAt(0) === '.'
      ? '' // anything
      : // not (start or / followed by . or .. followed by / or end)
      options.dot
      ? '(?!(?:^|\\/)\\.{1,2}(?:$|\\/))'
      : '(?!\\.)';
  var self = this;

  // 清理 stateChar 状态字符
  function clearStateChar() {
    if (stateChar) {
      // we had some state-tracking character
      // that wasn't consumed by this pass.
      switch (stateChar) {
        case '*':
          re += star;
          hasMagic = true;
          break;
        case '?':
          re += qmark;
          hasMagic = true;
          break;
        default:
          re += '\\' + stateChar;
          break;
      }
      self.debug('clearStateChar %j %j', stateChar, re);
      stateChar = false;
    }
  }
```

#### 1.2.7 正则表达式转换：第二部分 - 循环解析

- `minimatch.js`（阅读笔记：`/minimatch.js/2_4_parse.js`）

下一步就是要循环解析表达式的字符

```js
  // 3. 循环处理表达式
  for (var i = 0, len = pattern.length, c; i < len && (c = pattern.charAt(i)); i++) {
    this.debug('%s\t%s %s %j', pattern, i, re, c);

    // skip over any that are escaped.
    // 3.1 忽略特殊字符
    if (escaping && reSpecials[c]) {
      re += '\\' + c;
      escaping = false;
      continue;
    }

    // 3.2 处理扩展 glob 的状态字符
    switch (c) {} // switch
  } // for
```

##### 1.2.7.1 状态扩展符

对于特殊字符我们分开来看，对于状态扩展符，记录到 `stateChar`

```js
      case '/':
        // completely not allowed, even escaped.
        // Should already be path-split by now.
        return false;

      case '\\':
        clearStateChar();
        escaping = true;
        continue;

      // the various stateChar values
      // for the "extglob" stuff.
      case '?':
      case '*':
      case '+':
      case '@':
      case '!':
        this.debug('%s\t%s %s %j <-- stateChar', pattern, i, re, c);

        // all of those are literals inside a class, except that
        // the glob [!a] means [^a] in regexp
        // [!a] in glob => [^a] in reg
        if (inClass) {
          this.debug('  in class');
          if (c === '!' && i === classStart + 1) c = '^';
          re += c;
          continue;
        }

        // if we already have a stateChar, then it means
        // that there was something like ** or +? in there.
        // Handle the stateChar, then proceed with this one.
        self.debug('call clearStateChar %j', stateChar);
        clearStateChar();
        stateChar = c;
        // if extglob is disabled, then +(asdf|foo) isn't a thing.
        // just clear the statechar *now*, rather than even diving into
        // the patternList stuff.
        if (options.noext) clearStateChar();
        continue;
```

##### 1.2.7.2 分组符号

接下来对于分组符(`()`)，使用 `patternListStack` 栈来保存状态

```js
      case '(':
        // 在 [] 中
        if (inClass) {
          re += '(';
          continue;
        }

        // 没有前导状态字符
        if (!stateChar) {
          re += '\\(';
          continue;
        }

        patternListStack.push({
          type: stateChar,
          start: i - 1,
          reStart: re.length,
          open: plTypes[stateChar].open,
          close: plTypes[stateChar].close,
        });
        // negation is (?:(?!js)[^/]*)
        // 你的 open 不拿起来用一下吗
        re += stateChar === '!' ? '(?:(?!(?:' : '(?:';
        this.debug('plType %j %j', stateChar, re);
        stateChar = false;
        continue;

      case ')':
        // 在 [] 中或是没有前半的 ( 在栈中
        if (inClass || !patternListStack.length) {
          re += '\\)';
          continue;
        }

        clearStateChar();
        hasMagic = true;
        var pl = patternListStack.pop();
        // negation is (?:(?!js)[^/]*)
        // The others are (?:<pattern>)<type>
        re += pl.close;
        if (pl.type === '!') {
          negativeLists.push(pl);
        }
        pl.reEnd = re.length;
        continue;
```

##### 1.2.7.3 或符号

对于或运算符(`|`)，则是直接映射为正则表达式的或运算符

```js
      case '|':
        // [] 中、非 () 中、escaping 属于转义字符
        if (inClass || !patternListStack.length || escaping) {
          re += '\\|';
          escaping = false;
          continue;
        }

        clearStateChar();
        re += '|';
        continue;
```

##### 1.2.7.4 类运算符

对于类(`[]`)运算符，使用 `inClass` 标志保存状态

```js
      // these are mostly the same in regexp and glob
      case '[':
        // swallow any state-tracking char before the [
        clearStateChar();

        // [] 中
        if (inClass) {
          re += '\\' + c;
          continue;
        }

        // 顶层 []
        inClass = true;
        classStart = i;
        reClassStart = re.length;
        re += c;
        continue;

      case ']':
        //  a right bracket shall lose its special
        //  meaning and represent itself in
        //  a bracket expression if it occurs
        //  first in the list.  -- POSIX.2 2.8.3.2
        // [] 之间至少要有一个字符
        if (i === classStart + 1 || !inClass) {
          re += '\\' + c;
          escaping = false;
          continue;
        }

        // handle the case where we left a class open.
        // "[z-a]" is valid, equivalent to "\[z-a\]"
        if (inClass) {
          // split where the last [ was, make sure we don't have
          // an invalid re. if so, re-walk the contents of the
          // would-be class to re-translate any characters that
          // were passed through as-is
          // TODO: It would probably be faster to determine this
          // without a try/catch and a new RegExp, but it's tricky
          // to do safely.  For now, this is safe and works.
          var cs = pattern.substring(classStart + 1, i);
          try {
            RegExp('[' + cs + ']');
          } catch (er) {
            // not a valid class!
            var sp = this.parse(cs, SUBPARSE);
            re = re.substr(0, reClassStart) + '\\[' + sp[0] + '\\]';
            hasMagic = hasMagic || sp[1];
            inClass = false;
            continue;
          }
        }

        // finish up the class.
        hasMagic = true;
        inClass = false;
        re += c;
        continue;
```

##### 1.2.7.5 一般字符

最后默认情况对于一般字符我们直接写入正则表达式

```js
      default:
        // swallow any state char that wasn't consumed
        clearStateChar();

        if (escaping) {
          // no need
          escaping = false;
        } else if (reSpecials[c] && !(c === '^' && inClass)) {
          // 特殊字符全部转义掉
          re += '\\';
        }

        re += c;
    } // switch
  } // for
```

#### 1.2.8 正则表达式转换：第三部分 - 残局处理

##### 1.2.8.1 未闭合情况处理

对于 `[]`、`()` 等成对符号未闭合的情况需要特殊处理

- `minimatch.js`（阅读笔记：`/minimatch.js/2_4_parse.js`）

对于 `[]` 类未闭合

```js
  // handle the case where we left a class open.
  // "[abc" is valid, equivalent to "\[abc"
  // 4. 收拾没有关闭的 []
  if (inClass) {
    // split where the last [ was, and escape it
    // this is a huge pita.  We now have to re-walk
    // the contents of the would-be class to re-translate
    // any characters that were passed through as-is
    cs = pattern.substr(classStart + 1);
    sp = this.parse(cs, SUBPARSE);
    re = re.substr(0, reClassStart) + '\\[' + sp[0];
    hasMagic = hasMagic || sp[1];
  }
```

对于 `()` 未闭合

```js
  // handle the case where we had a +( thing at the *end*
  // of the pattern.
  // each pattern list stack adds 3 chars, and we need to go through
  // and escape any | chars that were passed through as-is for the regexp.
  // Go through and escape them, taking care not to double-escape any
  // | chars that were already escaped.
  // 5. 收拾没有关闭的 +(
  for (pl = patternListStack.pop(); pl; pl = patternListStack.pop()) {
    var tail = re.slice(pl.reStart + pl.open.length);
    this.debug('setting tail', re, pl);
    // maybe some even number of \, then maybe 1 \, followed by a |
    tail = tail.replace(/((?:\\{2}){0,64})(\\?)\|/g, function (_, $1, $2) {
      if (!$2) {
        // the | isn't already escaped, so escape it.
        $2 = '\\';
      }

      // need to escape all those slashes *again*, without escaping the
      // one that we need for escaping the | character.  As it works out,
      // escaping an even number of slashes can be done by simply repeating
      // it exactly after itself.  That's why this trick works.
      //
      // I am sorry that you have to see this.
      return $1 + $1 + $2 + '|';
    });

    this.debug('tail=%j\n   %s', tail, tail, pl, re);
    var t = pl.type === '*' ? star : pl.type === '?' ? qmark : '\\' + pl.type;

    hasMagic = true;
    re = re.slice(0, pl.reStart) + t + '\\(' + tail;
  }
```

##### 1.2.8.2 转义标志

除了未闭合类之外，还有字符转义的部分

- `minimatch.js`（阅读笔记：`/minimatch.js/2_4_parse.js`）

```js
  // handle trailing things that only matter at the very end.
  clearStateChar();
  if (escaping) {
    // trailing \\
    re += '\\\\';
  }

  // only need to apply the nodot start if the re starts with
  // something that could conceivably capture a dot
  // 6. 是否添加起始头部
  var addPatternStart = false;
  switch (re.charAt(0)) {
    case '.':
    case '[':
    case '(':
      addPatternStart = true;
  }
```

##### 1.2.8.3 反义表达式回看

除了分组，最前面我们解析 `negate` 反义表达式的结果也要在这里用上，加上反义表达式的前后包装

- `minimatch.js`（阅读笔记：`/minimatch.js/2_4_parse.js`）

```js
  // Hack to work around lack of negative lookbehind in JS
  // A pattern like: *.!(x).!(y|z) needs to ensure that a name
  // like 'a.xyz.yz' doesn't match.  So, the first negative
  // lookahead, has to look ALL the way ahead, to the end of
  // the pattern.
  // 7. ! 否定匹配需要回看
  for (var n = negativeLists.length - 1; n > -1; n--) {
    var nl = negativeLists[n];

    var nlBefore = re.slice(0, nl.reStart);
    var nlFirst = re.slice(nl.reStart, nl.reEnd - 8);
    var nlLast = re.slice(nl.reEnd - 8, nl.reEnd);
    var nlAfter = re.slice(nl.reEnd);

    nlLast += nlAfter;

    // Handle nested stuff like *(*.js|!(*.json)), where open parens
    // mean that we should *not* include the ) in the bit that is considered
    // "after" the negated section.
    var openParensBefore = nlBefore.split('(').length - 1;
    var cleanAfter = nlAfter;
    for (i = 0; i < openParensBefore; i++) {
      cleanAfter = cleanAfter.replace(/\)[+*?]?/, '');
    }
    nlAfter = cleanAfter;

    var dollar = '';
    if (nlAfter === '' && isSub !== SUBPARSE) {
      dollar = '$';
    }
    var newRe = nlBefore + nlFirst + nlAfter + dollar + nlLast;
    re = newRe;
  }
```

##### 1.2.8.4 正则表达式生成

最后还有几个琐碎的边界条件，然后最后生成正则表达式

- `minimatch.js`（阅读笔记：`/minimatch.js/2_4_parse.js`）

```js
  // if the re is not "" at this point, then we need to make sure
  // it doesn't match against an empty path part.
  // Otherwise a/* will match a/, which it should not.
  // 8. 确保非空模式不匹配空串
  if (re !== '' && hasMagic) {
    re = '(?=.)' + re;
  }

  // 9. 根据 addPatternStart 添加模式头部
  if (addPatternStart) {
    re = patternStart + re;
  }

  // parsing just a piece of a larger pattern.
  if (isSub === SUBPARSE) {
    return [re, hasMagic];
  }

  // skip the regexp for non-magical patterns
  // unescape anything in it, though, so that it'll be
  // an exact match against a file etc.
  // 10. 对于不存在任何特殊匹配的模式
  if (!hasMagic) {
    return globUnescape(pattern);
  }

  // 11. 构建正则表达式
  var flags = options.nocase ? 'i' : ''; // 忽略大小写
  try {
    var regExp = new RegExp('^' + re + '$', flags);
  } catch (er) {
    // If it was an invalid regular expression, then it can't match
    // anything.  This trick looks for a character after the end of
    // the string, which is of course impossible, except in multi-line
    // mode, but it's not a /m regex.
    return new RegExp('$.');
  }

  regExp._glob = pattern;
  regExp._src = re;

  return regExp;
}
```

#### 1.2.9 正则表达式组生成

看到这里就完成正则表达式生成了

- `minimatch.js`（阅读笔记：`/minimatch.js/2_1_make.js`）

```js
  // filter out everything that didn't compile properly.
  // 7. 过滤匹配失败的表达式
  set = set.filter(function (s) {
    return s.indexOf(false) === -1;
  });

  this.debug(this.pattern, set);

  this.set = set;
}
```

回到 `make` 函数，我们的 glob 表达式经过分段、转正则之后，会生成一个二维的正则表达式组(一维为花括号扩展成的多个表达式，二维为一个表达式分段后的多个片段)

整个 make 的过程确定了 `this.set` 的内容

#### 1.2.10 makeRe 生成完整表达式

完成 make 之后我们就可以基于 set 生成一个完整的正则表达式

- `minimatch.js`（阅读笔记：`/minimatch.js/2_5_makeRe.js`）

```js
// ** when dots are allowed.  Anything goes, except .. and .
// not (^ or / followed by one or two dots followed by $ or /),
// followed by anything, any number of times.
var twoStarDot = '(?:(?!(?:\\/|^)(?:\\.{1,2})($|\\/)).)*?';

Minimatch.prototype.makeRe = makeRe;
function makeRe() {
  // 手动发起 regexp 的创建
  if (this.regexp || this.regexp === false) return this.regexp;

  // at this point, this.set is a 2d array of partial
  // pattern strings, or "**".
  //
  // It's better to use .match().  This function shouldn't
  // be used, really, but it's pretty convenient sometimes,
  // when you just want to work with a regex.
  // 1. 保证 this.set 为转变后的正则表达式片段
  var set = this.set;

  // 解析失败
  if (!set.length) {
    this.regexp = false;
    return this.regexp;
  }
  var options = this.options;

  var twoStar = options.noglobstar
    ? star // '*'
    : options.dot
    ? twoStarDot // '**.'
    : twoStarNoDot; // '**'
  var flags = options.nocase ? 'i' : '';
```

前面主要还是一些参数的校验，最核心的部分就是下面这一块

```js
  var re = set
    .map(function (pattern) {
      return pattern
        .map(function (p) {
          return p === GLOBSTAR
            ? twoStar // p === GLOBSTAR === {}
            : typeof p === 'string'
            ? regExpEscape(p) // p === string(plain text)
            : p._src; // p === regexp
        })
        .join('\\/'); // s1/s2/s3/...
    })
    .join('|'); // p1|p2|p3|...
```

对于每个表达式，的每个片段进行整合

下面再对最后产物进行一点修饰后返回

```js
  // must match entire pattern
  // ending in a * or ** will make it less strict.
  re = '^(?:' + re + ')$'; // 头尾

  // can match anything, as long as it's not this.
  if (this.negate) re = '^(?!' + re + ').*$'; // 取反

  try {
    // 构建完整正则表达式
    this.regexp = new RegExp(re, flags);
  } catch (ex) {
    // 构建失败
    this.regexp = false;
  }
  return this.regexp;
}
```

#### 1.2.11 match 匹配字符串

生成好正则表达式就可以真正来匹配实际的目标字符串了

- `minimatch.js`（阅读笔记：`/minimatch.js/2_6_match.js`）

一开始先对目标字符串进行一些预处理

```js
Minimatch.prototype.match = match;

/* 匹配字符串 */
function match(f, partial) {
  this.debug('match', f, this.pattern);
  // short-circuit in the case of busted things.
  // comments, etc.
  // 1. 匹配注释
  if (this.comment) return false;
  // 2. 匹配空串
  if (this.empty) return f === '';

  // 3. 匹配 /
  if (f === '/' && partial) return true;

  var options = this.options;

  // windows: need to use /, not \
  // 4. 统一分界符 /
  if (path.sep !== '/') {
    f = f.split(path.sep).join('/');
  }

  // treat the test path as a set of pathparts.
  // 5. 按 / 拆分 => f = []
  f = f.split(slashSplit);
  this.debug(this.pattern, 'split', f);

  // just ONE of the pattern sets in this.set needs to match
  // in order for it to be valid.  If negating, then just one
  // match means that we have failed.
  // Either way, return on the first hit.

  // 6. 获取分解好的模式 => this.set = []
  var set = this.set;
  this.debug(this.pattern, 'set', set);
```

接下来第二个部分是循环检查是否存在任何一个表达式能匹配目标字符串

```js
  // Find the basename of the path by looking for the last non-empty segment
  var filename; // basename
  var i;
  for (i = f.length - 1; i >= 0; i--) { // 后往前
    filename = f[i];
    if (filename) break;
  }

  // 7. 测试命中
  for (i = 0; i < set.length; i++) {
    var pattern = set[i];
    var file = f;
    if (options.matchBase && pattern.length === 1) {
      file = [filename];
    }
    var hit = this.matchOne(file, pattern, partial);
    if (hit) {
      if (options.flipNegate) return true;
      return !this.negate;
    }
  }

  // didn't get any hits.  this is success if it's a negative
  // pattern, failure otherwise.
  // 8. 未命中
  if (options.flipNegate) return false;
  return this.negate;
}
```

这里的核心就是这个 `matchOne` 函数，检查了目标路径片段与一个正则表达式片段是否匹配

#### 1.2.12 matchOne 单一模式匹配

`matchOne` 就是检查目标与当前模式是否匹配

- `minimatch.js`（阅读笔记：`/minimatch.js/2_7_matchOne.js`）

整个匹配循环过程分成三大块，第一块是匹配解析失败的表达式

```js
// set partial to true to test if, for example,
// "/a/b" matches the start of "/*/b/*/d"
// Partial means, if you run out of file before you run
// out of pattern, then that's fine, as long as all
// the parts match.
/* 匹配表达式片段 */
Minimatch.prototype.matchOne = function (file, pattern, partial) {
  var options = this.options;

  this.debug('matchOne', { this: this, file: file, pattern: pattern });

  this.debug('matchOne', file.length, pattern.length);

  for (var fi = 0, pi = 0, fl = file.length, pl = pattern.length; fi < fl && pi < pl; fi++, pi++) {
    this.debug('matchOne loop');
    var p = pattern[pi];
    var f = file[fi];

    this.debug(pattern, p, f);

    // should be impossible.
    // some invalid regexp stuff in the set.
    // 1. 模式匹配失败 => p === false
    if (p === false) return false;
```

第二部分是匹配 globstar 的扩展特性

```js
    // 2. 匹配 **
    if (p === GLOBSTAR) {
      this.debug('GLOBSTAR', [pattern, p, f]);

      // "**"
      // a/**/b/**/c would match the following:
      // a/b/x/y/z/c
      // a/x/y/z/b/c
      // a/b/x/b/x/c
      // a/b/c
      // To do this, take the rest of the pattern after
      // the **, and see if it would match the file remainder.
      // If so, return success.
      // If not, the ** "swallows" a segment, and try again.
      // This is recursively awful.
      //
      // a/**/b/**/c matching a/b/x/y/z/c
      // - a matches a
      // - doublestar
      //   - matchOne(b/x/y/z/c, b/**/c)
      //     - b matches b
      //     - doublestar
      //       - matchOne(x/y/z/c, c) -> no
      //       - matchOne(y/z/c, c) -> no
      //       - matchOne(z/c, c) -> no
      //       - matchOne(c, c) yes, hit
      var fr = fi;
      var pr = pi + 1;
      // 2.1 ** 作为尾部
      if (pr === pl) {
        this.debug('** at the end');
        // a ** at the end will just swallow the rest.
        // We have found a match.
        // however, it will not swallow /.x, unless
        // options.dot is set.
        // . and .. are *never* matched by **, for explosively
        // exponential reasons.
        for (; fi < fl; fi++) {
          // '**' 不匹配 '.' '../' '.xxx'
          if (file[fi] === '.' || file[fi] === '..' || (!options.dot && file[fi].charAt(0) === '.')) return false;
        }
        return true;
      }

      // ok, let's see if we can swallow whatever we can.
      // 2.2 贪婪模式
      while (fr < fl) {
        var swallowee = file[fr];

        this.debug('\nglobstar while', file, fr, pattern, pr, swallowee);

        // XXX remove this slice.  Just pass the start index.
        if (this.matchOne(file.slice(fr), pattern.slice(pr), partial)) {
          // 递归匹配后半部成功 => true
          this.debug('globstar found match!', fr, fl, swallowee);
          // found a match.
          return true;
        } else {
          // can't swallow "." or ".." ever.
          // can only swallow ".foo" when explicitly asked.
          // 遇到 ** 不匹配的 => 完成 ** 匹配
          if (swallowee === '.' || swallowee === '..' || (!options.dot && swallowee.charAt(0) === '.')) {
            this.debug('dot detected!', file, fr, pattern, pr);
            break;
          }

          // ** swallows a segment, and continue.
          this.debug('globstar swallow a segment, and continue');
          fr++;
        }
      }

      // no match was found.
      // However, in partial mode, we can't say this is necessarily over.
      // If there's more *pattern* left, then
      // partial 模式匹配到当前最佳结果
      if (partial) {
        // ran out of file
        this.debug('\n>>> no match, partial?', file, fr, pattern, pr);
        if (fr === fl) return true;
      }
      return false;
    }
```

globstar 特性的实现就是尽可能匹配多个目录片段

第三部分则是匹配简单字符串

```js
    // something other than **
    // non-magic patterns just have to match exactly
    // patterns with magic have been turned into regexps.
    var hit;
    if (typeof p === 'string') {
      // 3. 匹配简单字符串
      if (options.nocase /* 忽略大小写 */) {
        hit = f.toLowerCase() === p.toLowerCase();
      } else {
        hit = f === p;
      }
      this.debug('string match', p, f, hit);
    } else {
      // 4. 匹配正则表达式
      hit = f.match(p);
      this.debug('pattern match', p, f, hit);
    }

    if (!hit) return false;
  }
```

最后是一些边界条件的检查

```js
  // Note: ending in / means that we'll get a final ""
  // at the end of the pattern.  This can only match a
  // corresponding "" at the end of the file.
  // If the file ends in /, then it can only match a
  // a pattern that ends in /, unless the pattern just
  // doesn't have any more for it. But, a/b/ should *not*
  // match "a/b/*", even though "" matches against the
  // [^/]*? pattern, except in partial mode, where it might
  // simply not be reached yet.
  // However, a/b/ should still satisfy a/*

  // now either we fell off the end of the pattern, or we're done.
  if (fi === fl && pi === pl) {
    // 5. 完成所有片段匹配
    // ran out of pattern and filename at the same time.
    // an exact hit!
    return true;
  } else if (fi === fl) {
    // 6. 完成目标路径匹配
    // ran out of file, but still had pattern left.
    // this is ok if we're doing the match as part of
    // a glob fs traversal.
    return partial;
  } else if (pi === pl) {
    // 7. 完成模式匹配 => 目标路径以 / 结尾
    // ran out of pattern, still have file left.
    // this is only acceptable if we're on the very last
    // empty segment of a file with a trailing slash.
    // a/* should match a/b/
    var emptyFileEnd = fi === fl - 1 && file[fi] === '';
    return emptyFileEnd;
  }

  // should be unreachable.
  // 7. 未知路径
  throw new Error('wtf?');
};
```

### 1.3 minimatch

看完整个 Minimatch 核心类的实现，实际上默认导出的 minimatch 就是 Minimatch 的代理，所以大部分的方法就是 Minimatch 的封装

#### 1.3.1 filter 过滤函数

minimatch 在 Minimatch 之外额外提供了一个过滤函数，能够过滤符合条件的文件字符串

- `minimatch.js`（阅读笔记：`/minimatch.js/1_1_filter.js`）

```js
minimatch.filter = filter;

/* 生成过滤函数 */
function filter(pattern, options) {
  options = options || {};
  return function (p, i, list) {
    return minimatch(p, pattern, options);
  };
}
```

返回一个类似 `match` 方法的函数，有一点使用科里化提前绑定的意思

#### 1.3.2 braceExpand 花括号扩展

第二个函数 braceExpand 实际上就等价于 Minimatch 的版本

- `minimatch.js`（阅读笔记：`/minimatch.js/1_2_braceExpand.js`）

```js
// Brace expansion:
// a{b,c}d -> abd acd
// a{b,}c -> abc ac
// a{0..3}d -> a0d a1d a2d a3d
// a{b,c{d,e}f}g -> abg acdfg acefg
// a{b,c}d{e,f}g -> abdeg acdeg abdeg abdfg
//
// Invalid sets are not expanded.
// a{2..}b -> a{2..}b
// a{b}c -> a{b}c
/* 与 Minimatch.braceExpand 等价 */
minimatch.braceExpand = function (pattern, options) {
  return braceExpand(pattern, options);
};
```

#### 1.3.3 makeRe 创建正则表达式

也是照搬 Minimatch

- `minimatch.js`（阅读笔记：`/minimatch.js/1_3_makeRe.js`）

```js
/* 与 Minimatch.makeRe 等价（创建新的 Minimatch 状态隔离） */
minimatch.makeRe = function (pattern, options) {
  return new Minimatch(pattern, options || {}).makeRe();
};
```

#### 1.3.4 match 匹配方法

- `minimatch.js`（阅读笔记：`/minimatch.js/1_4_match.js`）

```js
/* Minimatch.match */
minimatch.match = function (list, pattern, options) {
  options = options || {};
  var mm = new Minimatch(pattern, options);
  list = list.filter(function (f) {
    return mm.match(f);
  });
  if (mm.options.nonull && !list.length) {
    list.push(pattern);
  }
  return list;
};
```

### 1.4 默认配置

如 axios 一样，其实很多库都有类似的实现，就是提供一个 `defaults` 属性或是方法保存一些默认的配置

#### 1.4.1 Minimatch.defaults

- `minimatch.js`（阅读笔记：`/minimatch.js/2_0_defaults.js`）

```js
/* minimatch.defaults 代理 + 返回自身支持链式 */
Minimatch.defaults = function (def) {
  if (!def || !Object.keys(def).length) return Minimatch;
  return minimatch.defaults(def).Minimatch;
};
```

代理到 minimatch 方法上

#### 1.4.2 minimatch.defaults

- `minimatch.js`（阅读笔记：`/minimatch.js/1_0_defaults.js`）

```js
/* 默认配置 defaults */
minimatch.defaults = function (def) {
  if (!def || !Object.keys(def).length) return minimatch;

  var orig = minimatch;

  var m = function minimatch(p, pattern, options) {
    return orig.minimatch(p, pattern, ext(def, options));
  };

  m.Minimatch = function Minimatch(pattern, options) {
    return new orig.Minimatch(pattern, ext(def, options));
  };

  return m;
};
```

minimatch 库采用返回代理的方式，避免污染全局 Minimatch 类，不过比较琐碎

# 其他资源

## 参考连接

| Title                          | Link                                                                                                                       |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------- |
| minimatch - npm                | [https://www.npmjs.com/package/minimatch](https://www.npmjs.com/package/minimatch)                                         |
| minimatch - Github             | [https://github.com/isaacs/minimatch](https://github.com/isaacs/minimatch)                                                 |
| Bash Extended Globbing         | [https://www.linuxjournal.com/content/bash-extended-globbing](https://www.linuxjournal.com/content/bash-extended-globbing) |
| glob (programming) - wikipedia | [https://en.wikipedia.org/wiki/Glob_(programming)](https://en.wikipedia.org/wiki/Glob_(programming))                       |

## 阅读笔记参考

[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/minimatch-3.0.4](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/minimatch-3.0.4)
