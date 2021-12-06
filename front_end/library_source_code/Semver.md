# semver 源码解析(Npm library)

@[TOC](文章目录)

<!-- TOC -->

- [semver 源码解析(Npm library)](#semver-源码解析npm-library)
- [正文](#正文)
  - [0. 基本信息](#0-基本信息)
  - [1. 源码解析](#1-源码解析)
    - [1.1 核心类型](#11-核心类型)
    - [1.2 SemVer 类型实现核心](#12-semver-类型实现核心)
      - [1.2.1 SemVer 构造函数](#121-semver-构造函数)
      - [1.2.2 SemVer.prototype.compare 比较](#122-semverprototypecompare-比较)
      - [1.2.3 包装函数](#123-包装函数)
    - [1.3 Comparator 类型实现核心](#13-comparator-类型实现核心)
      - [1.3.1 Comparator 构造函数](#131-comparator-构造函数)
      - [1.3.2 Comparator.prototype.intersects 计算交集](#132-comparatorprototypeintersects-计算交集)
    - [1.4 Range 类型实现核心](#14-range-类型实现核心)
      - [1.4.1 Range 构造函数](#141-range-构造函数)
      - [1.4.2 Range.prototype.parseRange 解析原始字符串](#142-rangeprototypeparserange-解析原始字符串)
      - [1.4.3 包装函数](#143-包装函数)
    - [1.5 小结](#15-小结)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 正文

## 0. 基本信息

- version：`v7.3.5`
- 功能：SemVer 版本号规范基础操作实现

SemVer 为语意化版本号的规约，广泛应用在多数包管理器进行包的版本管理规约。semver 是一个非常有用的包版本管理

传送门：[Semantic Versioning 2.0.0](https://semver.org/)

## 1. 源码解析

下面我们进行源码的解读，由于整个库的代码量还是比较大的，所以这里我们挑几个重点讲解，详细完整源码可以参考阅读笔记的注释

### 1.1 核心类型

参考 SemVer 规约，semver 构建了三种基本类型

1. `SemVer`：保存并解析 $X.Y.Z$ 类型格式的版本号
2. `Comparator`：则是在 SemVer 类型之外多保存了一个比较符号，可以用于代表一个单侧区间，如 `>=0.0.0`
3. `Range`：可以保存特殊符号如 `~`、`^`、`-` 的特殊区间，内部使用 Comparator[] 数组来表示区间

### 1.2 SemVer 类型实现核心

接下来看到核心类型 SemVer 的实现，我们只挑几个方法来解析

#### 1.2.1 SemVer 构造函数

一个类最重要的莫过于构造函数

- `/classes/semver.js`

第一部分属于参数解析，也可以看到注释里面总结的 SemVer 类型属性

```js
class SemVer {
  /**
   * 初始化版本号
   * options : 配置选项
   * loose             : 标志 - 是否使用放松规则
   * includePrerelease : 标志 - 是否附带预发版本
   * raw : 原始 version 版本号
   * major      : 主版本号
   * minor      : 次版本号
   * patch      : 更新版本号
   * prerelease : 预发版本号
   * build      : 构建版本号
   * @param {*} version
   * @param {*} options
   * @returns
   */
  // ? Read
  constructor(version, options) {
    options = parseOptions(options); // 解析配置参数

    if (version instanceof SemVer) {
      // 1. version 为 SemVer 实例
      if (
        version.loose === !!options.loose &&
        version.includePrerelease === !!options.includePrerelease
      ) {
        // loose 模式 && includePrerelease 模式下返回相同实例
        return version;
      } else {
        version = version.version;
      }
    } else if (typeof version !== 'string') {
      // 保证 version 为字符串
      throw new TypeError(`Invalid Version: ${version}`);
    }

    if (version.length > MAX_LENGTH) {
      // 检查 verison 长度限制
      throw new TypeError(`version is longer than ${MAX_LENGTH} characters`);
    }

    // =============== 参数解析 ===============
```

接下来的正式解析过程直接利用 re 模块提供的正则表达式进行匹配

```js
    debug('SemVer', version, options);
    this.options = options;
    this.loose = !!options.loose;
    // this isn't actually relevant for versions, but keep it so that we
    // don't run into trouble passing this.options around.
    this.includePrerelease = !!options.includePrerelease;

    // 匹配版本号
    const m = version.trim().match(options.loose ? re[t.LOOSE] : re[t.FULL]);

    if (!m) {
      throw new TypeError(`Invalid Version: ${version}`);
    }
```

分别记录 major、minor、patch、prerelease、build 五部分的版本号

```js
    this.raw = version;

    // these are actually numbers
    this.major = +m[1]; // 主版本号
    this.minor = +m[2]; // 次版本号
    this.patch = +m[3]; // patch 版本号

    // 确保三个版本号皆为有效数字
    if (this.major > MAX_SAFE_INTEGER || this.major < 0) {
      throw new TypeError('Invalid major version');
    }

    if (this.minor > MAX_SAFE_INTEGER || this.minor < 0) {
      throw new TypeError('Invalid minor version');
    }

    if (this.patch > MAX_SAFE_INTEGER || this.patch < 0) {
      throw new TypeError('Invalid patch version');
    }

    // numberify any prerelease numeric ids
    if (!m[4]) {
      this.prerelease = [];
    } else {
      this.prerelease = m[4].split('.').map((id) => {
        if (/^[0-9]+$/.test(id)) {
          const num = +id;
          if (num >= 0 && num < MAX_SAFE_INTEGER) {
            // 纯数字预发版本
            return num;
          }
        }
        // 普通标识符版本
        return id;
      });
    }

    // build 版本
    this.build = m[5] ? m[5].split('.') : [];
    this.format();
  }
}
```

#### 1.2.2 SemVer.prototype.compare 比较

有了 SemVer 类最重要的就是这个 compare 方法了

- `/classes/semver.js`

```js
  /**
   * 比较 SemVer 对象
   * @param {*} other
   * @returns
   */
  // ? Read
  compare(other) {
    debug('SemVer.compare', this.version, this.options, other);
    if (!(other instanceof SemVer)) {
      // 非 SemVer 对象
      if (typeof other === 'string' && other === this.version) {
        // 简单比较一下字符串
        return 0;
      }
      // 转换成 SemVer 对象
      other = new SemVer(other, this.options);
    }

    // 比较 version
    if (other.version === this.version) {
      return 0;
    }

    // 先比较主版本再比较预发版本
    return this.compareMain(other) || this.comparePre(other);
  }
```

比较主版本号的方法 `compareMain`

```js
  /**
   * 比较主版本号：major、minor、patch
   * @param {*} other
   * @returns
   */
  // ? Read
  compareMain(other) {
    if (!(other instanceof SemVer)) {
      // 非 SemVer 的先转换
      other = new SemVer(other, this.options);
    }

    // 按序比较 major、minor、patch
    return (
      compareIdentifiers(this.major, other.major) ||
      compareIdentifiers(this.minor, other.minor) ||
      compareIdentifiers(this.patch, other.patch)
    );
  }
```

以及比较预发版本号 `comparePre`

```js
  /**
   * 比较预发版本号
   * @param {*} other
   * @returns
   */
  // ? Read
  comparePre(other) {
    if (!(other instanceof SemVer)) {
      // 转换成 SemVer 对象
      other = new SemVer(other, this.options);
    }

    // NOT having a prerelease is > having one
    // 1. 先简单根据 length 比较，没有 prerelease 的最大
    if (this.prerelease.length && !other.prerelease.length) {
      return -1;
    } else if (!this.prerelease.length && other.prerelease.length) {
      return 1;
    } else if (!this.prerelease.length && !other.prerelease.length) {
      return 0;
    }

    let i = 0;
    do {
      const a = this.prerelease[i];
      const b = other.prerelease[i];
      debug('prerelease compare', i, a, b);
      // prerelease 愈多组的优先级越高（相当于是 prerelease 里的 X < X.Y < X.Y.Z）
      if (a === undefined && b === undefined) {
        // 长度相同
        return 0;
      } else if (b === undefined) {
        // b 短  => a > b => 1
        return 1;
      } else if (a === undefined) {
        // a 短  => a < b => -1
        return -1;
      } else if (a === b) {
        // a,b 相同，比较下一个
        continue;
      } else {
        // a,b 非空且不同，比较 ID
        return compareIdentifiers(a, b);
      }
    } while (++i);
  }
```

#### 1.2.3 包装函数

最后 semver 包导出的函数实际上就是包装 SemVer 类的 compare 方法

例如版本相等方法

- `/functions/eq.js`

```js
const compare = require('./compare');
// ? Read
// a = b ?
const eq = (a, b, loose) => compare(a, b, loose) === 0;
module.exports = eq;
```

- `/functions/compare.js`

```js
const SemVer = require('../classes/semver');
/**
 * 比较两个版本号
 *   分别构建 SemVer，然后使用 compare 方法比较
 * @param {*} a
 * @param {*} b
 * @param {*} loose
 * @returns
 */
// ? Read
const compare = (a, b, loose) =>
  new SemVer(a, loose).compare(new SemVer(b, loose));

module.exports = compare;
```

其他更多接口就不细说

### 1.3 Comparator 类型实现核心

Comparator 是包装了版本号前面的比较符号类型

#### 1.3.1 Comparator 构造函数

首先看到构造函数

- `/classes/comparator.js`

```js
// 任意 SemVer 版本号
const ANY = Symbol('SemVer ANY');

/**
 * 比较类型
 *   保存目标范围（比较符号 + 比价版本）
 *   test       测试给定版本号是否符合当前 Comparator
 *   intersects 检查两个 Comparator 是否存在交集
 */
// ? Read
// hoisted class for cyclic dependency
class Comparator {
  // ? getter for Symbol('SemVer ANY')
  static get ANY() {
    return ANY;
  }

  /**
   * 构造函数
   * options : 配置选项
   * loose   : 模糊模式
   * value   : 比较符号 + 比较目标版本号
   * @param {*} comp
   * @param {*} options
   * @returns
   */
  // ? Read
  constructor(comp, options) {
    options = parseOptions(options);

    if (comp instanceof Comparator) {
      if (comp.loose === !!options.loose) {
        return comp;
      } else {
        comp = comp.value;
      }
    }

    // ========== 参数校验 ==========

    debug('comparator', comp, options);
    this.options = options;
    this.loose = !!options.loose;
    this.parse(comp);

    // 值 = 比较符号 + 比较目标版本号
    if (this.semver === ANY) {
      this.value = '';
    } else {
      this.value = this.operator + this.semver.version;
    }

    debug('comp', this);
  }
```

主要的解析函数封装在了 parse 方法，继续往下看

```js
  /**
   * 解析比较字符串
   * operator : 比较符号
   * semver   : 比较目标版本号
   * @param {*} comp
   */
  // ? Read
  parse(comp) {
    const r = this.options.loose ? re[t.COMPARATORLOOSE] : re[t.COMPARATOR];
    const m = comp.match(r);

    if (!m) {
      throw new TypeError(`Invalid comparator: ${comp}`);
    }

    this.operator = m[1] !== undefined ? m[1] : '';
    if (this.operator === '=') {
      // = 等价于空
      this.operator = '';
    }

    // if it literally is just '>' or '' then allow anything.
    if (!m[2]) {
      // 无版本号  => 匹配任意版本
      this.semver = ANY;
    } else {
      // 构建比较目标
      this.semver = new SemVer(m[2], this.options.loose);
    }
  }
```

parse 方法一样使用正则表达式匹配之后，分成 `operator` 和 `semver` 两部分

#### 1.3.2 Comparator.prototype.intersects 计算交集

对于 Comparator 类型来说也有 test 方法检验给定版本号是否符合区间。不过另一个也很重要的部分是计算两个 Comparator 是否产生交集

- `/classes/comparator.js`

第一部分一样先做参数校验

```js
  /**
   * 检验两个版本号是否相交（交集 != null）
   * @param {*} comp
   * @param {*} options
   * @returns
   */
  // ? Read
  intersects(comp, options) {
    if (!(comp instanceof Comparator)) {
      throw new TypeError('a Comparator is required');
    }

    if (!options || typeof options !== 'object') {
      options = {
        loose: !!options,
        includePrerelease: false,
      };
    }

    // ========== 参数校验 ==========
```

接下来是处理一下两个参数的 operator，`=` 的情况下使用 Range 对象来比较

```js
    // 其中一方无比较符号 => 作为 Range.test 参数
    if (this.operator === '') {
      // this 无比较函数
      if (this.value === '') {
        // value 为空 === ANY ===> true
        return true;
      }
      // 创建 Range 类比较
      return new Range(comp.value, options).test(this.value);
    } else if (comp.operator === '') {
      // comp 无比较函数
      if (comp.value === '') {
        return true;
      }
      return new Range(this.value, options).test(comp.semver);
    }
```

对于两个 Comparator 都有比较符号的时候，计算两个区间是否存在非空交集

```js
    // 标志
    // 都是 > or >=
    const sameDirectionIncreasing =
      (this.operator === '>=' || this.operator === '>') &&
      (comp.operator === '>=' || comp.operator === '>');
    // 都是 < or <=
    const sameDirectionDecreasing =
      (this.operator === '<=' || this.operator === '<') &&
      (comp.operator === '<=' || comp.operator === '<');
    // version 相同
    const sameSemVer = this.semver.version === comp.semver.version;
    // 都是 ?=
    const differentDirectionsInclusive =
      (this.operator === '>=' || this.operator === '<=') &&
      (comp.operator === '>=' || comp.operator === '<=');
    // this < comp && this 为 > && comp 为 <
    // 即 this.version <= this <= comp <= comp.version
    const oppositeDirectionsLessThan =
      cmp(this.semver, '<', comp.semver, options) &&
      (this.operator === '>=' || this.operator === '>') &&
      (comp.operator === '<=' || comp.operator === '<');
    // this > comp && this 为 < && comp 为 >
    // 即 comp.version <= comp <= this <= this.version
    const oppositeDirectionsGreaterThan =
      cmp(this.semver, '>', comp.semver, options) &&
      (this.operator === '<=' || this.operator === '<') &&
      (comp.operator === '>=' || comp.operator === '>');

    return (
      // 方向相同
      sameDirectionIncreasing ||
      sameDirectionDecreasing ||
      // 版本相同且都带 =
      (sameSemVer && differentDirectionsInclusive) ||
      // 方向存在交集
      oppositeDirectionsLessThan ||
      oppositeDirectionsGreaterThan
    );
  }
```

由于 Comparator 主要还是为 Range 类型服务，因此我们就不讨论具体实现方法，直接往下看到 Range 对象的实现

### 1.4 Range 类型实现核心

Range 比起 Comparator 又更高级一些，能够完整定义一个版本号区间，同时允许参数传入带特殊符号（如 `~ ^ -`）的版本号描述

#### 1.4.1 Range 构造函数

一样先来看构造函数

- `/classes/range.js`

```js
/**
 * 版本号区间
 */
// ? Read
// hoisted class for cyclic dependency
class Range {
  /**
   * 构造函数
   * raw     : 原始字符串
   * set     : 范围集合
   * options : 配置选项
   * loose             : 标志 - 简易模式
   * includePrerelease : 标志 - 是否包含预发版本
   * set     : Comparator[] 范围数组
   * @param {*} range : ;
   * @param {*} options
   * @returns
   */
  // ? Read
  constructor(range, options) {
    options = parseOptions(options);
```

第一种如果传入的是一个 Range 对象，则按类似拷贝构造的形式直接完成当前对象的创建

```js
    if (range instanceof Range) {
      // 拷贝构造
      if (
        range.loose === !!options.loose &&
        range.includePrerelease === !!options.includePrerelease
      ) {
        return range;
      } else {
        return new Range(range.raw, options);
      }
    }
```

如果参数是一个 Comparator，表示是一个简单的单侧区间，直接作为 `set` 属性值并返回

```js
    if (range instanceof Comparator) {
      // 使用 Comparator 构造
      // just put it in the set and return
      this.raw = range.value;
      this.set = [[range]];
      this.format();
      return this;
    }
```

最后则是字符串形式的情况，我们需要进行字符串的匹配

```js
    this.options = options;
    this.loose = !!options.loose;
    this.includePrerelease = !!options.includePrerelease;

    // First, split based on boolean or ||
    this.raw = range;
    this.set = range
      .split(/\s*\|\|\s*/) // 按 || 划分
      // map the range to a 2d array of comparators
      .map((range) => this.parseRange(range.trim()))
      // throw out any comparator lists that are empty
      // this generally means that it was not a valid range, which is allowed
      // in loose mode, but will still throw if the WHOLE range is invalid.
      .filter((c) => c.length);
```

最后对解析后的 `set` 进行区间的校验和过滤

```js
    // 范围数组不可为空
    if (!this.set.length) {
      throw new TypeError(`Invalid SemVer Range: ${range}`);
    }

    // if we have any that are not the null set, throw out null sets.
    if (this.set.length > 1) {
      // keep the first one, in case they're all null sets
      const first = this.set[0];
      this.set = this.set.filter((c) => !isNullSet(c[0])); // 过滤空范围
      if (this.set.length === 0) this.set = [first];
      // 保留第一个串
      else if (this.set.length > 1) {
        // if we have any that are *, then the range is just *
        for (const c of this.set) {
          if (c.length === 1 && isAny(c[0])) {
            this.set = [c]; // 匹配任意版本
            break;
          }
        }
      }
    }

    this.format();
  }
```

#### 1.4.2 Range.prototype.parseRange 解析原始字符串

Range 对象对于字符串的解析还是比较复杂

- `/classes/range.js`

首先先去 cache 里面查找是否存在符合条件的 Range

```js
  /**
   * 解析范围字符串 => Comparator[]
   * @param {*} range : ;
   * @returns
   */
  // ? Read
  parseRange(range) {
    range = range.trim();

    // memoize range parsing for performance.
    // this is a very hot path, and fully deterministic.
    const memoOpts = Object.keys(this.options).join(',');
    const memoKey = `parseRange:${memoOpts}:${range}`; // 'parseRange : options 序列 : range' 作缓存 key
    const cached = cache.get(memoKey);
    if (cached) return cached; // 是否已缓存
```

接下来先对特殊符号进行转换，如：
- `v1 - v2` 转换成 `>=v1 <=v2`
- `~ v1 ^ v2` 裁剪空白符 `~v1 ^v2`

```js
    const loose = this.options.loose;

    // 转义连字号写法
    // `1.2.3 - 1.2.4` => `>=1.2.3 <=1.2.4`
    const hr = loose ? re[t.HYPHENRANGELOOSE] : re[t.HYPHENRANGE];
    range = range.replace(hr, hyphenReplace(this.options.includePrerelease));
    debug('hyphen replace', range);

    // 裁剪空格
    // `> 1.2.3 < 1.2.5` => `>1.2.3 <1.2.5`
    range = range.replace(re[t.COMPARATORTRIM], comparatorTrimReplace);
    debug('comparator trim', range, re[t.COMPARATORTRIM]);

    // `~ 1.2.3` => `~1.2.3`
    range = range.replace(re[t.TILDETRIM], tildeTrimReplace);

    // `^ 1.2.3` => `^1.2.3`
    range = range.replace(re[t.CARETTRIM], caretTrimReplace);

    // 聚合空白符
    // normalize spaces
    range = range.split(/\s+/).join(' ');
```

最后则是使用 `parseComparator` 方法转换成 Comparator[] 数组，作为 set 返回

```js
    // At this point, the range is completely trimmed and
    // ready to be split into comparators.

    const compRe = loose ? re[t.COMPARATORLOOSE] : re[t.COMPARATOR];
    const rangeList = range
      .split(' ')
      .map((comp) => parseComparator(comp, this.options)) // 转义特殊字符
      .join(' ')
      .split(/\s+/)
      // >=0.0.0 is equivalent to *
      .map((comp) => replaceGTE0(comp, this.options)) // 转义 0.0.0
      // in loose mode, throw out any that are not valid comparators
      .filter(this.options.loose ? (comp) => !!comp.match(compRe) : () => true) // 舍去 loose 模式下不合法比较符号
      .map((comp) => new Comparator(comp, this.options)); // 构建 Comparator 数组

    // if any comparators are the null set, then replace with JUST null set
    // if more than one comparator, remove any * comparators
    // also, don't include the same comparator more than once
    const l = rangeList.length;
    const rangeMap = new Map();
    for (const comp of rangeList) {
      if (isNullSet(comp)) return [comp]; // 存在空集合  => 直接返回
      rangeMap.set(comp.value, comp); // value => comp
    }
    if (rangeMap.size > 1 && rangeMap.has('')) rangeMap.delete(''); // 移除 '' 通配符

    const result = [...rangeMap.values()];
    cache.set(memoKey, result); // key => Comparator[]
    return result;
  }
```

#### 1.4.3 包装函数

对于 Range 类型也导出了一些函数作为对外接口，以 `max-satisfying` 为例

- `/ranges/max-satisfying.js`

首先根据参数构建 Range 对象

```js
const SemVer = require('../classes/semver');
const Range = require('../classes/range');

/**
 * 找出最大的符合 range 条件的 version
 * @param {*} versions 
 * @param {*} range 
 * @param {*} options 
 * @returns 
 */
// ? Read
const maxSatisfying = (versions, range, options) => {
  let max = null;
  let maxSV = null;
  let rangeObj = null;
  try {
    rangeObj = new Range(range, options);
  } catch (er) {
    return null;
  }
```

然后遍历 versions 找出符合条件（使用 `Range.test`）的最大（使用 `SemVer.compare`）版本号

```js
  versions.forEach((v) => {
    if (rangeObj.test(v)) {
      // v 处于 range 区间内
      // satisfies(v, range, options)
      if (!max || maxSV.compare(v) === -1) {
        // compare(max, v, true)
        // maxSV < v => max = v
        max = v;
        maxSV = new SemVer(max, options);
      }
    }
  });
  return max;
};
module.exports = maxSatisfying;
```

### 1.5 小结

SemVer 规范还是比较普遍使用的，透过这次对于 semver 库的源码解析，也对版本号的定义理解更进一步

# 其他资源

## 参考连接

| Title                                                              | Link                                                                                           |
| ------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------- |
| semver - npm                                                       | [https://www.npmjs.com/package/semver](https://www.npmjs.com/package/semver)                   |
| npm/node-semver - Github                                           | [https://github.com/npm/node-semver](https://github.com/npm/node-semver)                       |
| Semantic Versioning 2.0.0                                          | [https://semver.org/](https://semver.org/)                                                     |
| Key words for use in RFCs to Indicate Requirement Levels - rfc2119 | [https://datatracker.ietf.org/doc/html/rfc2119](https://datatracker.ietf.org/doc/html/rfc2119) |

## 阅读笔记参考

[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/semver-7.3.5](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/semver-7.3.5)
