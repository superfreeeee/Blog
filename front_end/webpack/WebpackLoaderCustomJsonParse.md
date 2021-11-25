# Webpack: Loader 开发分享

@[TOC](文章目录)

<!-- TOC -->

- [Webpack: Loader 开发分享](#webpack-loader-开发分享)
- [正文](#正文)
  - [1. Concept 概念](#1-concept-概念)
  - [2. Configuration 配置实例](#2-configuration-配置实例)
  - [3. Custom 自定义 Loader](#3-custom-自定义-loader)
    - [3.1 配置自定义 Loader](#31-配置自定义-loader)
    - [3.2 内嵌 loader（路径指定）](#32-内嵌-loader路径指定)
    - [3.3 Loader 写法](#33-loader-写法)
  - [4. 实战：jsonc-loader](#4-实战jsonc-loader)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

## 1. Concept 概念

Webpack 的理念告诉我们万物皆模块（`Module`），然而一个完整的前端网页不仅仅只有 JavaScript，还会有其他各类文件的参与（CSS、HTML、静态资源）

因此 Webpack 作为一个 Module Bundler 将整个打包过程拆分成不同的模块分别负责不同的任务，详细就不展开说了。

今天要来搞的就是所谓的 Loader 的部分，一言以蔽之就是：

> Loader 负责将非 JS 的文件转换成 JS 模块后导入

例如我们常会用的

```js
import './xxx.css'
```

实际上这样导入就不是标准 JavaScript 能看懂的范畴了，因此就要借助 Loader 的能力转换成 JS 模块后再导入

## 2. Configuration 配置实例

在 Wwebpack 中配置 Loader 的方法官方也写的很详细了，简单来说就是写在配置文件的 `module.rules` 项当中

- `webpack.config.js`

```js
module.exports = {
  module: {
    rules: [
      // ...
    ]
  }
}
```

第一种写法只配置单一个 loader

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.html$/,
        use: 'html-loader'
      }
    ]
  }
}
```

要使用多个 loader 可以传入一个数组，生效的顺序为由后往前

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      }
    ]
  }
}
```

另外如果想要针对某一个 loader 传入配置参数，将名字改成对象就可以了

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.module.(sass|scss)$/,
        use: [
          'style-loader',
          {
            loader: 'css-loader',
            options: {
              modules: {
                localIdentName: '[path][name]__[local]--[hash:base64:5]',
              },
              sourceMap: true,
            },
          },
          'sass-loader',
        ],
      },
    ]
  }
}
```

如上面最后一个例子，配置了一个启用了 CSS Module 和 Sass 语法的解析，先后经过 `sass-loader、css-loader、style-loader`，最后被引入代码当中

其中为 `css-loader` 特别配置了模块化方案的参数

## 3. Custom 自定义 Loader

那么我们要怎么自己来写一个 loader 呢，就好像 Vue 一样写了一个 loader 来支持 `.vue` 文件的解析一样，看着就很屌

### 3.1 配置自定义 Loader

配置自定义 loader 有很多种方式

1. 第一种：引入相对路径（只用一次的时候）

```js
const path = require('path');

module.exports = {
  //...
  module: {
    rules: [
      {
        test: /\.js$/,
        use: [
          {
            loader: path.resolve('path/to/loader.js'),
            options: {
              /* ... */
            },
          },
        ],
      },
    ],
  },
};
```

2. 第二种：配置 loader 查找范围（推荐）

```js
const path = require('path');

module.exports = {
  //...
  resolveLoader: {
    modules: ['node_modules', path.resolve(__dirname, 'loaders')],
  },
};
```

使用第二种就可以直接在 loader 属性写名字就行了

### 3.2 内嵌 loader（路径指定）

当然要使用内联的 loader 的时候就必须使用第二种写法

```js
require('./loader!./lib');
```

像是上面这样写 webpack 就会知道加载 `./lib` 这个东西之后要先经过 `loader` 的转换

内嵌的时候甚至也能为 loader 配置参数

```js
require('./loader1?xyz!loader2!./resource?rrr');
```

这样写老实说可读性还是挺差的，所以通常在 loader 的 runtime 写法中比较常见，正常用的时候还是避免内嵌到路径当中

### 3.3 Loader 写法

接下来进入正题，loader 到底该怎么写。

首先先看看模块结构

```js
module.exports = function (source) {
  // loader
};
```

一个 loader 必须导出一个 funciton 函数（这里应该是不能用箭头函数的，因为 webpack 会通过 `this` 将 context 注入进来，不过我也不太确定hh，`this` 还是要学好啊hh）

loader 的返回就很多种了

- 同步一个返回值：直接 `return output`
- 同步多个返回值：使用 `this.callback`
- 异步：使用 `callback = this.async()`

详细使用还是去查文档吧，这里写了也没意思

## 4. 实战：jsonc-loader

最后我们给出一个 loader 的实战，用于解析 jsonc 格式的文件

由于 Node 默认就支持导入 `.json` 文件，但是意外的发现不能用 jsonc，所以可以来试一把，主要任务就是把注释删了就可以了

- `/loaders/jsonc-loader`

```js
const removeComment = (s) => {
  return s.replace(/\/\/.*$/gm, '');
};

const removeWhiteSpace = (s) => {
  return s.replace(/[ \n\t]*/g, '');
};

module.exports = function (source) {
  console.log(`Solve jsonc: '${source}'`);

  const noComment = removeComment(source);
  console.log(`noComment: '${noComment}'`);

  const noWhiteSpace = removeWhiteSpace(noComment);
  console.log(`noWhiteSpace: '${noWhiteSpace}'`);

  const data = JSON.parse(noWhiteSpace);
  console.log(`Parsed data:`, data);

  const res = JSON.stringify(data);
  console.log(`res: ${res}`);

  return `module.exports = ${res}`;
}
```

直接上代码，还是比较简单的，使用正则表达式分别删除注释、清理空白符、使用 parse 反序列化确保 json 格式正确，最后反序列化并加上 `module.exports` 作为模块默认导出对象

写好 loader 之后要去配置文件里面配置一下

- `webpack.config.js`

```js
const path = require('path');

const config = {
  // ...
  resolveLoader: {
    modules: [path.resolve(__dirname, 'loaders'), 'node_modules'],
  },
  module: {
    rules: [
      {
        test: /\.jsonc/,
        use: 'jsonc-loader',
      },
    ],
  },
};

module.exports = config;
```

首先是 `resolveLoader` 来配置 loader 目录，然后加入一个 rule

最后的测试代码稍微贴一下

- `/src/index.js`

```js
console.log('sample', require('./sample.jsonc'));
```

然后分别使用 Node 直接运行 & 使用 webpack 编译后运行的结果

- Node 直接运行脚本

```bash
$ node src/index.js
Hello world
~/webpack_loader_custom_json_parse/src/sample.jsonc:3
  "name": "superfree",
        ^

SyntaxError: Unexpected token ':'
```

- Webpack 编译后运行

```bash
$ webpack && node dist/main.js

✔ Webpack
  Compiled successfully in 124.86ms

asset main.js 283 bytes [emitted] [minimized] (name: main)
./src/index.js 103 bytes [built] [code generated]
./src/sample.jsonc 70 bytes [built] [code generated]
webpack 5.64.3 compiled successfully in 127 ms
Hello world
sample { name: 'superfree', age: 22, courses: [ 'math', 'cs' ] }
```

# 其他资源

## 参考连接

| Title                                               | Link                                                                                                                                         |
| --------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| Writing a Loader - Webpack                          | [https://webpack.js.org/contribute/writing-a-loader/](https://webpack.js.org/contribute/writing-a-loader/)                                   |
| Loader Interface - Webpack                          | [https://webpack.js.org/api/loaders/](https://webpack.js.org/api/loaders/)                                                                   |
| require(id) - Node                                  | [https://nodejs.org/dist/latest-v16.x/docs/api/modules.html#requireid](https://nodejs.org/dist/latest-v16.x/docs/api/modules.html#requireid) |
| JSON Loader - npm                                   | [https://www.npmjs.com/package/json-loader](https://www.npmjs.com/package/json-loader)                                                       |
| webpack-contrib/json-loader - Github                | [https://github.com/webpack-contrib/json-loader](https://github.com/webpack-contrib/json-loader)                                             |
| 手把手教你撸一个 Webpack Loader                     | [https://blog.csdn.net/lszy16/article/details/79162960](https://blog.csdn.net/lszy16/article/details/79162960)                               |
| 简单实用的webpack-html-include-loader（附开发详解） | [https://cloud.tencent.com/developer/article/1607507](https://cloud.tencent.com/developer/article/1607507)                                   |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/webpack/webpack_loader_custom_json_parse](https://github.com/superfreeeee/Blog-code/tree/main/front_end/webpack/webpack_loader_custom_json_parse)
