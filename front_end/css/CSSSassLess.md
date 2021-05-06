# CSS 预处理器: Sass/Scss & Less

@[TOC](文章目录)

<!-- TOC -->

- [CSS 预处理器: Sass/Scss & Less](#css-预处理器-sassscss--less)
- [前言](#前言)
  - [Why 预处理器？](#why-预处理器)
  - [Sass？Scss？](#sassscss)
- [正文](#正文)
  - [1. 预处理器的编译工具](#1-预处理器的编译工具)
    - [Less 编译](#less-编译)
    - [Sass/Scss 编译](#sassscss-编译)
  - [2. Gulp 实现自动化编译](#2-gulp-实现自动化编译)
  - [3. 预处理器功能](#3-预处理器功能)
    - [3.0 预处理器功能概述](#30-预处理器功能概述)
    - [3.1 变量(Variables)](#31-变量variables)
      - [3.1.1 基本使用](#311-基本使用)
    - [3.2 嵌套(Nesting)](#32-嵌套nesting)
      - [3.2.1 基本使用](#321-基本使用)
      - [3.2.2 父选择器](#322-父选择器)
    - [3.3 混合(Mixins)](#33-混合mixins)
      - [3.3.1 基本使用](#331-基本使用)
      - [3.3.2 带参数的混合器](#332-带参数的混合器)
      - [3.3.3 不输出混合器](#333-不输出混合器)
    - [3.4 继承(Extensions)](#34-继承extensions)
      - [3.4.1 基本使用](#341-基本使用)
      - [3.4.2 纯虚样式](#342-纯虚样式)
    - [3.5 运算(Operations) & 函数(Functions)](#35-运算operations--函数functions)
      - [3.5.1 基本使用](#351-基本使用)
      - [3.5.2 自定义函数](#352-自定义函数)
    - [3.6 控制指令(Control Directives)](#36-控制指令control-directives)
      - [3.6.1 条件分支 if](#361-条件分支-if)
      - [3.6.2 列表遍历 each](#362-列表遍历-each)
      - [3.6.3 循环遍历 for](#363-循环遍历-for)
    - [3.7 注释(Comments)](#37-注释comments)
    - [3.8 导入(Importing)](#38-导入importing)
  - [4. 附录：webpack 打包编译嵌入](#4-附录webpack-打包编译嵌入)
    - [4.1 下载依赖](#41-下载依赖)
    - [4.2 配置文件](#42-配置文件)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码实现](#完整代码实现)

<!-- /TOC -->

# 前言

CSS 层叠样式表作为 HTML 的极大增强功能，使得空洞简陋的 html 页面套用丰富的样式规则和页面特效，然而我们今天要来介绍的是 CSS 预处理器(Preprocessor)。

## Why 预处理器？

为什么需要预处理器呢？写过一点 CSS 的前端朋友常常会在开发业务的时候遇到一些情况：需要使用精确的选择器来定位元素、常常写重复的样式逻辑、想要使用优先级来覆盖一些原有样式而刻意增加选择器权重、... 等。以上逻辑有时候不能说不好，也都是一些正规可接受的解决方案，但是问题在于：**太麻烦了！**

为了一些基本的业务逻辑而陷于 CSS 语法的泥沼，不断编写重复的 CSS 代码如下：

```css
.header > .title > .left > .inner > .a {/* ... */}
.header > .title > .left > .inner > .b {/* ... */}
.header > .title > .left > .inner > .c {/* ... */}
```

是不是看了就觉得烦，但是没办法，这就是 CSS。

所以我们就可以使用 CSS 预处理器，提供开发者更强大的语法支持，并透过工具自动转换成浏览器能识别的基本 CSS 语法，用了预处理器之后上述语法可以变为：

```less
.header {
    > .title {
        > .left {
            > .inner {
                > .a {/* ... */}
                > .b {/* ... */}
                > .c {/* ... */}
            }
        }
    }
}
```

这样看起来是不是舒服多了，而且当我们需要在不同层级增加扩展的时候就特别方便了。

下面我们将要来介绍 Sass 和 Less 两种最有名的 CSS 预处理器。

## Sass？Scss？

Sass 是包名称，编写语法风格使用缩进代替原生 CSS 的大括号；Scss 是使用 css 代码风格(大括号)的 Sass，通常还是用后者(也不知道作者怎么想的，可能就对缩进的样式情有独钟，进而斗胆破坏 CSS 的编码风格，这不找麻烦吗hhh)

# 正文

Sass 和 Less 的特性啊还是一些官方描述、宣传句子就不说了，简单来说都是一种 CSS 的增强语法。

## 1. 预处理器的编译工具

说白了，预处理器就是在真实使用 CSS 前经过一次编译的过程，将增强的语法功能转换回原生的 CSS 语法。所以，第一步我们要先来了解 Sass 和 Less 到底该怎么编译

### Less 编译

先来说比较单纯的 less。`Less.js` 本来就是采用 js 实现的预处理器，所以想当然他就是一个基本的 node 包，所以我们可以直接使用 `npm/yarn` 包管理工具来下载并使用：

```bash
$ yarn add less -D
$ yarn lessc <source-file> <target-file>
```

less 编译的核心命令为 `lessc`，传入源文件路径和目标文件名/路径就行了。另外需要注意按惯例 less 的语法需要写在 `.less` 扩展名的文件下，这也很好理解

### Sass/Scss 编译

1. Ruby 版本：Sass 比较特别的是使用 Ruby 实现的编译器(可能是当时最潮的 web 领域用的编程脚本吧，在 Node 出生之前)，所以要使用最原本的 Sass 编译器还要下一个 Ruby。

2. node-sass / dart-sass：第二种方案是使用 Node 环境下的 Sass，可能依据情况还要依赖底层 libsass(我是用 mac 自带 ruby 所以没啥问题，windows 用户可能会出现 node-sass 不能用，那就用 dart-sass 吧)

```bash
$ yarn add node-sass
$ yarn node-sass <source-file> <target-file>
```

编译选项也差不多，就是源文件跟目标文件

## 2. Gulp 实现自动化编译

再再配置一下，待会要用到好多文件能，总不能一个个手动编译吧。这边用上 gulp 简单构建一个自动编译的配置，在根目录下创建 `gulpfile.js`

- `gulpfile.js`

```js
const gulp = require('gulp')
const less = require('gulp-less')
const sass = require('gulp-sass')

const tasks = [
  {
    name: 'less',
    src: 'src/less/*.less',
    plugin: less,
    option: {},
    dest: 'lib/less',
  },
  {
    name: 'scss',
    src: 'src/scss/*.scss',
    plugin: sass,
    option: { outputStyle: 'expanded' },
    dest: 'lib/scss',
  },
]

tasks.forEach(({ name, src, plugin, option, dest }) => {
  gulp.task(name, (done) => {
    gulp.src(src).pipe(plugin(option)).pipe(gulp.dest(dest))
    done()
  })
})

gulp.task('default', gulp.parallel(...tasks.map((task) => task.name)))

gulp.task('watch', (done) => {
  tasks.forEach(({ name, src }) => {
    console.log(`watch '${src}' and do '${name}'`)
    gulp.watch(src, gulp.task(name))
  })
  done()
})
```

简单来说我们配置了：

- 一个 `less` 任务用于编译 `/src/less` 目录下的所有 `.less` 文件到 `/lib/less` 目录下
- 一个 `scss` 任务用于编译 `/src/scss` 目录下的所有 `.scss` 文件到 `/lib/scss` 目录下
- 一个 `default` 默认任务并行执行 `less`、`scss` 两个任务
- 一个 `watch` 任务用于监听文件变化，实时执行 `less`、`scss` 两个任务

这边 gulp 不是重点，就是方便测试，下面马上进入正题

## 3. 预处理器功能

### 3.0 预处理器功能概述

这边列出我们接下来要介绍的预处理器能力：

- 变量(Variables)
- 嵌套(Nesting)
- 混合(Mixins)
- 继承(Extensions)
- 运算(Operations) & 函数(Functions)
- 控制指令(Control Directives)
- 注释(Comments)
- 导入(Importing)

，可以说是不管在 Less 还是 Sass 都存在自己的实现方式，实现的模式也是大同小异，所以用哪个其实都差不多，看个人喜好。好了不bb了，马上来看看 CSS 预处理器的强大之处。

### 3.1 变量(Variables)

首先第一个特性是解决 CSS 最让人诟病的一个问题：变量的复用。

常常我们需要在好多地方复用同一个颜色或是固定的颜色风格，或是在不同样式之间维护相同的属性值。在原生 CSS 的规则之下我们只能手动的维护，到了 CSS3 才能够使用 `--var` + `var()` 的组合招来实现，然而预处理器早就实现了这个功能，甚至比原生的都要灵活：

- Less 使用 `@` 符号声明变量
- Sass/Scss 使用 `$` 符号声明变量

#### 3.1.1 基本使用

预处理器中，变量可以替换为 **属性值**、**属性名**、**选择器**，分别都有不同的写法

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_sass_less_var.png)

### 3.2 嵌套(Nesting)

第二项能力就是在前言中提到的，将丑陋的多层选择器转换为嵌套的形式。这个特性的实现 Less 和 Sass 的几乎一致，就是将样式表放到外层选择器样式表之中，进行 **嵌套定义**

#### 3.2.1 基本使用

第一种是基本的使用姿态，直接嵌套、使用 `>`、`+`、`[space]` 关键字来表达不同的 CSS 组选择器，语法和效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_sass_less_nesting_basic.png)

#### 3.2.2 父选择器

比较特别的是，默认的嵌套行为是使用 `A B` 的祖先-子代选择器，而且必定是外层选择器作为前导，如果我们想要复用外层选择器或是改变选择器次序则可以使用 `&` 符号替代外层选择器：

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_sass_less_nesting_father.png)

### 3.3 混合(Mixins)

第三种能力是混合，也就是我们最需要的样式复用。我们知道 CSS 本身就是透过选择器的方式对不同的元素应用样式表来完成复用，然而该方式的粒度还是略显繁杂，衍生出 **原子 CSS** 的实现思想。但是在预处理器的能力之下我们就能够轻松的透过 **混合(Mixins)** 来完成样式的复用

#### 3.3.1 基本使用

第一种是混合的基本使用：

- Less 直接将样式写在别的样式里面就有了
- Sass/Scss 需要使用 `@mixin` + `@include` 关键字来实现

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_sass_less_mixins_basic.png)

#### 3.3.2 带参数的混合器

更进一步，我们可以透过传递参数的方式来特制需要混入的样式，两者的实现方式非常相似，就是在混合器的选择器/名称后面加上形式参数列表(还可以使用默认参数)：

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_sass_less_mixins_parametric.png)

#### 3.3.3 不输出混合器

Sass/Scss 默认混合器就是一个特别的样式，不用 `@include` 不会出现；然而 Less 却是能将一般的样式也作为混合器混入其他样式，所以我们可以在样式后面加上空参数列表 `selector()` 来声明为一个纯混合器而不会直接放到编译后结果之中。

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_sass_less_mixins_not_output.png)

### 3.4 继承(Extensions)

继承的复用形式又与混合不太一样：

- 混合：将 `A` 的样式混合到 `B` 的样式当中
- 继承：`A` 继承 `B` 的意思就是，对于所有套用 `B` 的样式表 `A` 也应该套用，也就是会将继承其他类的选择器添加到继承目标的所有样式表选择器上

#### 3.4.1 基本使用

基本的使用方式：

- Less 使用 `:extend(father)` 伪类来引入继承的特性
- Sass/Scss 则使用 `@extend` 关键字来实现继承

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_sass_less_extend_basic.png)

#### 3.4.2 纯虚样式

跟 C++ 的纯虚函数或是 Java 的接口相似，我们希望父类只是一个抽象，不会直接出现在编译结果当中，Sass/Scss 提供 `%` 符号作为类的前缀，可以表示一个纯虚样式表的实现，只有当被继承的时候才会引入

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_sass_less_extend_not_output.png)

### 3.5 运算(Operations) & 函数(Functions)

接下来一个很重要的能力时运算和函数的能力。原生 CSS 提供的函数和运算方法寥寥无几(可以说就只有 `calc` 一个)，预处理器提供了我们更多的运算空间，同时提供了大量的内置函数和自定义函数的能力。

不过需要注意的是即便看起来是根据变量来运算，但其实一切都会在编译后消失，所以真正最后编译的样式结果其实还是静态的，只是更方便开发者定义变量值运算逻辑和确定计算过程而已。

#### 3.5.1 基本使用

基本的运算方法可以分为：

- 数字运算
- 字符串运算
- 颜色创建/颜色修改
- 其他内置函数

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_sass_less_operations_functions_basic.png)

详细的内置函数扩展可以查询各预处理器官网 API 说明

#### 3.5.2 自定义函数

虽然两者都提供了一些基本运算方法和内置函数，有时候我们还是想要定义自己的函数，预处理器就厉害了，为 CSS 加入了类似命令式编程的语法，编译原理就是这么强大，这都能转：

- Less 使用 `Mixins + Maps` 的组合来模拟实现函数的效果
- Sass/Scss 提供 `@function + @return` 的关键字来实现更平易近人的函数调用体验

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_sass_less_operations_functions_custom_function.png)

### 3.6 控制指令(Control Directives)

快进入到尾声了，预处理器还提供了类似条件判断、循环等控制语句的模拟实现，逻辑判断和大量生成也是难不倒预处理器的

#### 3.6.1 条件分支 if

两种预处理器都提供了条件分支的 if 实现：

- Less 使用 `if(condition, true-result, false-result)` 内置方法实现
- Sass/Scss 则使用 `@if` 关键字进行条件编译

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_sass_less_control_directives_if.png)

#### 3.6.2 列表遍历 each

在一般的 for 循环之前，我们先说说遍历既有列表的 each 方法：

- Less 使用 `each(list, stylesheet)` 方法实现，样式表内部使用 `@value` 来拿到每次遍历的值
- Sass/Scss 使用 `@each var in list` 的方法实现

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_sass_less_control_directives_each.png)

#### 3.6.3 循环遍历 for

- Less 使用了 `each + range` 的组合来模拟实现 for 循环的效果
- Sass/Scss 使用 `@for var from a through b` 的关键字实现

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_sass_less_control_directives_for.png)

### 3.7 注释(Comments)

CSS 注释倒是比较简单，原生 CSS 只存在 `/* */` 多行注释的形式，预处理器虽然允许 `//` 单行注释，但是会在编译后将其省略

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_sass_less_comment.png)

### 3.8 导入(Importing)

最后一个就是导入外部样式表的能力，原生 CSS 也提供 `@import` 的语法，所以这边其实也差不多

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_sass_less_import.png)

## 4. 附录：webpack 打包编译嵌入

在正式业务流程中不会想再自己一个个编译打包，通常会选择一个自动化打包工具来使用，这边以 webpack 为例：

对于 webpack 来说一切模块皆 js，所以 `.less`、`.scss` 这种不伦不类的扩展不认识，就要用特定的 loader 进行编译后转化为标准 js 嵌入

### 4.1 下载依赖

首先是编译选项的依赖，webpack 核心的就不列了

- Less 编译工具 & loader

```bash
$ yarn add less less-loader -D
```

- Sass/Scss 编译工具 & loader

```bash
$ yarn add node-sass sass-loader -D
```

如同前面提过的 node-sass 用不了就用 dart-sass 吧，别死磕了

### 4.2 配置文件

主要就是配置 loader，在 `module.rules` 下添加规则：

- `webpack.config.js`

```js
module.exports = {
  // ...
  module: {
    rules: [
      // ...
      {
        test: /\.less$/,
        use: ['style-loader', 'css-loader', 'less-loader'],
      },
      {
        test: /\.scss$/,
        use: ['style-loader', 'css-loader', 'sass-loader'],
      },
      // ...
    ],
  },
  // ...
}
```

简单来说就是使用 `less-loader/sass-loader` 将 `.less/.scss` 文件转换为普通 CSS 之后就能够套用 `css-loader` 和 `style-loader` 转化为正常模块了

# 结语

CSS 预处理为开发者剩下大把的代码维护成本，同时也提高代码的可读性以及可维护性。Less 和 Sass 的使用上也基本差不多，学好一个要再转也是很快的，本篇供大家参考，欢迎指教。

# 其他资源

## 参考连接

<table>
  <tr>
    <td>Less 官方-中文</td>
    <td><a href="https://less.bootcss.com/">https://less.bootcss.com/</a></td>
  </tr>
  <tr>
    <td>Sass 官网</td>
    <td><a href="https://sass-lang.com/">https://sass-lang.com/</a></td>
  </tr>
  <tr>
    <td>Sass 中文网</td>
    <td><a href="https://www.sass.hk/">https://www.sass.hk/</a></td>
  </tr>
  <tr>
    <td>Gulp 中文网</td>
    <td><a href="https://v3.gulpjs.com.cn/">https://v3.gulpjs.com.cn/</a></td>
  </tr>
  <tr>
    <td>用gulp如何编译多个文件夹下的less</td>
    <td><a href="https://segmentfault.com/q/1010000009691680">https://segmentfault.com/q/1010000009691680</a></td>
  </tr>
  <tr>
    <td>使用gulp编译sass</td>
    <td><a href="https://www.cnblogs.com/wuzhiquan/p/6738018.html">https://www.cnblogs.com/wuzhiquan/p/6738018.html</a></td>
  </tr>
  <tr>
    <td>SASS学习系列之三--------- node-sass 自动编译scss 文件</td>
    <td><a href="https://blog.csdn.net/wx11408115/article/details/78032443">https://blog.csdn.net/wx11408115/article/details/78032443</a></td>
  </tr>
  <tr>
    <td>串行方式运行任务，亦即，任务依赖</td>
    <td><a href="https://v3.gulpjs.com.cn/docs/recipes/running-tasks-in-series/">https://v3.gulpjs.com.cn/docs/recipes/running-tasks-in-series/</a></td>
  </tr>
  <tr>
    <td>gulp构建项目（三）：gulp-watch监听文件改变、新增、删除</td>
    <td><a href="https://blog.csdn.net/guang_s/article/details/84672449">https://blog.csdn.net/guang_s/article/details/84672449</a></td>
  </tr>
  <tr>
    <td>Webpack安装(2)-打包css、scss、less(包括编译、分离)</td>
    <td><a href="https://blog.csdn.net/lhtzbj12/article/details/79188447">https://blog.csdn.net/lhtzbj12/article/details/79188447</a></td>
  </tr>
  <tr>
    <td>webpack---less+热更新 使用</td>
    <td><a href="https://www.cnblogs.com/chen-cong/p/8323958.html">https://www.cnblogs.com/chen-cong/p/8323958.html</a></td>
  </tr>
</table>

## 完整代码实现

<a href="https://github.com/superfreeeee/Blog-code/tree/main/front_end/css/css_sass_less">https://github.com/superfreeeee/Blog-code/tree/main/front_end/css/css_sass_less</a>
