# Babel 插件开发: 数组负数索引语法糖

@[TOC](文章目录)

<!-- TOC -->

- [Babel 插件开发: 数组负数索引语法糖](#babel-插件开发-数组负数索引语法糖)
- [正文](#正文)
  - [1. Doc](#1-doc)
  - [2. 核心流程](#2-核心流程)
  - [3. 核心包简单尝试](#3-核心包简单尝试)
  - [4. 插件开发实战：数组负数索引支持](#4-插件开发实战数组负数索引支持)
    - [4.1 插件解构规则](#41-插件解构规则)
    - [4.2 插件配置](#42-插件配置)
    - [4.3 插件实现](#43-插件实现)
    - [4.4 t.memberExpression 小坑分享](#44-tmemberexpression-小坑分享)
    - [4.5 运行成果](#45-运行成果)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

在前端开发用 Babel 也有一段时间了，今天给大家介绍到底如何开发一个自定义的 Babel 插件，加入自己的语法糖！

## 1. Doc

Babel 插件的文档感觉比较老旧了，可能因为真正的核心开发者比较少在干这个hh，主要的标准语法插件也都有官方的人写了，所以文档大部分是以前的

给大家推荐一个

- [Babel Plugin Handbook（英文版）](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/en/plugin-handbook.md)
- [Babel Plugin Handbook（中文版）](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md)

这里需要注意的是文档里面的包的名字有时候比较混乱，可能需要自己脑补猜一下

## 2. 核心流程

Babel 的架构中核心流程可以分为三步：`解析 -> 转换 -> 生成`

相信大家都听的老掉牙了，这里就不赘述细节了，可以自己看参考链接多补一点概念，然后看看上面的手册看看几个核心类型

实际上这三个步骤也就正好对应三个包：

- 解析：`@babel/parser`
- 转换（遍历）：`@babel/traverse`
- 生成：`@babel/generator`

## 3. 核心包简单尝试

下面我们先来试试一个直接引入 babel 的库来试试

- `/src/test.js`

这里很搞的是 babel 导出变量的方法比较特别，导致我们需要使用下面这种引入才能找到正确的函数

```js
import { parse } from '@babel/parser';
import _traverse from '@babel/traverse';
import _generate from '@babel/generator';

const traverse = _traverse.default;
const generate = _generate.default;
```

下面是我们要传入的源代码

```js
const code = `function greeting() {
  console.log('Hello World');
}

greeting();
`;
```

最后我们分别经过三个步骤就能够得到最终生成的源代码

```js
const ast = parse(code);

traverse(ast, {
  MemberExpression(p) {
    console.log('MemberExpression', p.node);
  },
});

const result = generate(ast, {}, code);
```

`parse` 方法将源代码映射为 AST；`traverse` 会遍历 AST，并接受第二个参数使用 Visitor 模式进行访问；最后 `generate` 函数生成转译后的代码，还可以传入源代码进行比较并生成源码映射表

最终的结果也给大家展示一下

- 解析后的 AST

```js
parsed AST Node {
  type: 'File',
  start: 0,
  end: 67,
  loc: SourceLocation {
    start: Position { line: 1, column: 0 },
    end: Position { line: 6, column: 0 },
    filename: undefined,
    identifierName: undefined
  },
  errors: [],
  program: Node {
    type: 'Program',
    start: 0,
    end: 67,
    loc: SourceLocation {
      start: [Position],
      end: [Position],
      filename: undefined,
      identifierName: undefined
    },
    sourceType: 'script',
    interpreter: null,
    body: [ [Node], [Node] ],
    directives: []
  },
  comments: []
}
```

- 遍历过程

源代码只有一个表达式解析为 `MemberExpression` 类型

```js
console.log('Hello World');
```

```js
MemberExpression Node {
  type: 'MemberExpression',
  start: 24,
  end: 35,
  loc: SourceLocation {
    start: Position { line: 2, column: 2 },
    end: Position { line: 2, column: 13 },
    filename: undefined,
    identifierName: undefined
  },
  object: Node {
    type: 'Identifier',
    start: 24,
    end: 31,
    loc: SourceLocation {
      start: [Position],
      end: [Position],
      filename: undefined,
      identifierName: 'console'
    },
    name: 'console'
  },
  computed: false,
  property: Node {
    type: 'Identifier',
    start: 32,
    end: 35,
    loc: SourceLocation {
      start: [Position],
      end: [Position],
      filename: undefined,
      identifierName: 'log'
    },
    name: 'log'
  }
}
```

- 生成结果

```js
Generated Result {
  code: "function greeting() {\n  console.log('Hello World');\n}\n\ngreeting();",
  map: null,
  rawMappings: undefined
}
```

关于解析后的 AST 节点类型还有相关详细细节都可以从 `@babel/types` 里面找到，同时它也提供了类型判断、对象生成等方法，是等下我们真正开发插件的时候的好帮手

## 4. 插件开发实战：数组负数索引支持

最后我们给大家展示一个支持负数索引数组的插件开发。

我们知道在 javascript 里面会会将 `[]` 内的索引转为字符串后再进行索引，所以如果写的是负数只会变成查找 `-x` 的键，而不会从数组尾部开始查找，因此这里我们开发一个支持负数索引的插件转换

### 4.1 插件解构规则

- `/src/plugin.js`

首先我们先看到开发插件的时候规定的写法规则

```js
/**
 * 支持数组负数索引
 * @param {*} param0
 * @returns
 */
export default function ({ types: t }) {
  return {
    visitor: {},
  };
}
```

其实本质上就是要导出一个默认函数，这个函数返回一个对象，并将遍历的时候需要用到的 Visitor 写在 visitor 对应的值当中

而函数默认接受的参数就是 babel 本身，可以将 `@babel/types` 以 `types` 键解构出来

### 4.2 插件配置

当我们要使用自定义插件的时候，可以直接在配置文件里面写上相对路径就可以了

- `.babelrc`

```json
{
  "plugins": ["./src/plugin.js"]
}
```

### 4.3 插件实现

最后我们来看看插件的内容要怎么写

- `/src/plugin.js`

```js
/**
 * 支持数组负数索引
 * @param {*} param0
 * @returns
 */
export default function ({ types: t }) {
  return {
    visitor: {
      MemberExpression(path) {
        /**
         * 满足形式
         * MemberExpression {
         *   object: Identifier | MemberExpression {},
         *   property: UnaryExpression {
         *     prefix: true,
         *     operator: '-',
         *     arguments: NumericLiteral {}
         *   }
         * }
         */
        const { object: obj, property: prop } = path.node || {};

        const isObjMatch =
          obj && (t.isIdentifier(obj) || t.isMemberExpression(obj));
        const isPropMatch = prop && t.isUnaryExpression(prop);

        if (!isObjMatch || !isPropMatch) {
          return;
        }
```

首先一开始我们先对遍历到的 `node` 节点进行先查，我们只转换 `MemberExpression { object: Identifier | MemberExpression, property: UnaryExpression }` 形式的表达式

```js
        const { prefix, operator, argument: arg } = prop;

        const isPropIndexMatch =
          prefix &&
          operator === '-' &&
          t.isNumericLiteral(arg) &&
          arg.value > 0;

        if (!isPropIndexMatch) {
          return;
        }
```

接下来我们还要再对 `property` 进行检查，确定索引的参数是负数(`prefix = true` 且 prop 的类型必须是 `NumericLiteral`)

最后我们将 `x[-i]` 转换成 `x[x.length - i]` 的形式

```js
        /**
         * obj[prop]
         * 转换为
         * obj[obj.length - prop.arg.value]
         */
        // obj.length
        const len = t.memberExpression(obj, t.identifier('length'));
        // prop.value
        const val = t.numericLiteral(arg.value);
        // len - val
        const binExp = t.binaryExpression('-', len, val);
        // obj[obj.length - prop.value]
        const newNode = t.memberExpression(obj, binExp, true);

        path.replaceWith(newNode);
      },
    },
  };
}
```

这里就用到很多 types 的函数来进行节点的生成，当然像是 obj 我们就可以直接复用原来的那个对象就行了

### 4.4 t.memberExpression 小坑分享

Note: 这里的 `t.memberExpression` 表达式需要传入第三个参数 `computed = true`，才能够允许使用 `BinaryExpression` 作为 `property`，否则会报下面这个错

```
Property property of MemberExpression expected node to be of a type ["Identifier","PrivateName"] but instead got "BinaryExpression"
```

### 4.5 运行成果

最后给大家看看成果

- 编译目标代码 `/src/index.js`

```js
const a = [1, 2, 3, { b: [1, 2, 3, 4] }, 4, 5];
console.log(`a =`, a);
console.log(`a[0] =`, a[0]);
console.log(`a[-1] =`, a[-1]);
console.log(`a[+1] =`, a[+1]);
console.log(`a['-1'] =`, a['-1']);
console.log(`a[-3] =`, a[-3]);
console.log(`a[-3].b[-2] =`, a[-3].b[-2]);
```

- 编译成果 `/lib/index.js`

```js
const a = [1, 2, 3, {
  b: [1, 2, 3, 4]
}, 4, 5];
console.log(`a =`, a);
console.log(`a[0] =`, a[0]);
console.log(`a[-1] =`, a[a.length - 1]);
console.log(`a[+1] =`, a[+1]);
console.log(`a['-1'] =`, a['-1']);
console.log(`a[-3] =`, a[a.length - 3]);
console.log(`a[-3].b[-2] =`, a[a.length - 3].b[a[a.length - 3].b.length - 2]);
```

然后我们就可以直接运行一下编译结果

```
yarn run v1.22.17
$ babel ./src/index.js -d lib/ && node lib/index
Successfully compiled 1 file with Babel (74ms).
a = [ 1, 2, 3, { b: [ 1, 2, 3, 4 ] }, 4, 5 ]
a[0] = 1
a[-1] = 5
a[+1] = 2
a['-1'] = undefined
a[-3] = { b: [ 1, 2, 3, 4 ] }
a[-3].b[-2] = 3
✨  Done in 0.33s.
```

# 其他资源

## 参考连接

有时候还是推荐大家多看一点官方文档比较完整，或是直接从源码里面查相关的类型定义会更好，看别人写的产物总是容易踩坑

| Title                                                                                                                                                   | Link                                                                                                                                                                                                   |
| ------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| babel/babel - Github                                                                                                                                    | [https://github.com/babel/babel](https://github.com/babel/babel)                                                                                                                                       |
| Babel 插件手册                                                                                                                                          | [https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md) |
| @babel/types                                                                                                                                            | [https://babeljs.io/docs/en/babel-types](https://babeljs.io/docs/en/babel-types)                                                                                                                       |
| babel 7 全套                                                                                                                                            | [http://xaber.co/2019/09/07/babel-7-%E5%85%A8%E5%A5%97/](http://xaber.co/2019/09/07/babel-7-%E5%85%A8%E5%A5%97/)                                                                                       |
| babel 7 插件开发相关                                                                                                                                    | [https://zhuanlan.zhihu.com/p/81878859](https://zhuanlan.zhihu.com/p/81878859)                                                                                                                         |
| AST Explorer                                                                                                                                            | [https://astexplorer.net/](https://astexplorer.net/)                                                                                                                                                   |
| 手把手教你开发一个babel-plugin                                                                                                                          | [https://blog.csdn.net/weixin_34405354/article/details/88733010](https://blog.csdn.net/weixin_34405354/article/details/88733010)                                                                       |
| \[Bug\]: TypeError: traverse is not a function when using @babel/traverse in node.js                                                                    | [https://www.giters.com/babel/babel/issues/13855](https://www.giters.com/babel/babel/issues/13855)                                                                                                     |
| TypeError: Property property of MemberExpression expected node to be of a type \["Identifier","PrivateName"\] but instead got "BinaryExpression" #10139 | [https://github.com/babel/babel/issues/10139](https://github.com/babel/babel/issues/10139)                                                                                                             |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/babel/babel_plugin_custom_array_index](https://github.com/superfreeeee/Blog-code/tree/main/front_end/babel/babel_plugin_custom_array_index)
