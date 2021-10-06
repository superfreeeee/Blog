# ES6 特性: 模版字符串 Template String

@[TOC](文章目录)

<!-- TOC -->

- [ES6 特性: 模版字符串 Template String](#es6-特性-模版字符串-template-string)
- [正文](#正文)
  - [1. 特性：字符串拼接](#1-特性字符串拼接)
  - [2. 特性：字符串模版作为函数](#2-特性字符串模版作为函数)
  - [3. 应用：字符串表达式转对象](#3-应用字符串表达式转对象)
  - [4. 应用：生成内联 CSS 样式(style 样式)](#4-应用生成内联-css-样式style-样式)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

## 1. 特性：字符串拼接

原来使用 + 号拼接字符串 => 改为使用反引号与 `${}` 表达式组合字符串

1. ES5 以前

```js
const greeting = 'Hello ' + name + '!';
```

2. ES6 模版字符串

```js
const greeting = `Hello ${name}!`
```

## 2. 特性：字符串模版作为函数

除了作为字符串之外，还可以将模版字符串接在函数后面，利用字符串模版作为函数的参数

- 语法
  - 第一个参数为字符串字面量的类数组对象，还有一个 `raw` 属性保存所有字符串的原始字符
  - 后续参数为不定长的表达式，使用 `...` 扩展运算符变为表达式数组

```js
function fn(templates, ...expressions) {/* ... */}

fn`blablabla`
```

- 基础用法：将所有表达式代表的字符串转为大写

```js
const createGreeting = (name) => `Hello ${name}!`;
console.log(createGreeting('superfree'));

const createGreeting2 = ((strs, nameExp) => {
  // strs[0] === 'Hello '
  // strs[1] === '!'
  // nameExp === (name) => name.toUpperCase()
  return (name) => `${strs[0]}${nameExp(name)}${strs[1]}`;
})`Hello ${(name) => name.toUpperCase()}!`;

console.log(createGreeting2('superfree'));
```

## 3. 应用：字符串表达式转对象

对于第二种的字符串表达式作为函数参数不仅能返回字符串，实际上能返回任意东西

- 示例：将字符串解析为对象

```js
const createObj = (strs, ...expressions) => {
  console.log('strs', strs);
  console.log('expressions', expressions);
  const obj = {};
  let i = expressions.length;
  while (i-- > 0) {
    obj[strs[i]] = expressions[i];
  }
  return obj;
};

const name = 'superfree';
const age = 21;
const obj = createObj`name${name}age${age}`;
console.log('obj', obj);
```

## 4. 应用：生成内联 CSS 样式(style 样式)

- 应用2: 对象映射为 CSS 内联样式

```js
const createStyleFactory = (strs, ...expressions) => {
  return (props) => {
    let i = strs.length - 1;
    let s = strs[i];
    while (i-- > 0) {
      s = strs[i] + expressions[i](props) + s;
    }
    return s;
  };
};

const createStyle = createStyleFactory`
  width: ${(props) => props.width};
  height: ${(props) => props.height};
  background: ${(props) => props.background || '#000'};
`;

console.log(createStyle({ width: 111, height: 222 }));

document
  .querySelector('.block')
  .setAttribute('style', createStyle({ width: '200px', height: '40px', background: '#f0f0f0' }));
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/es6_template_string_1_app_css.png)

# 其他资源

## 参考连接

| Title                               | Link                                                                                                                                                                       |
| ----------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Template literals - MDN             | [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals) |
| ES6 JavaScript 模板字符串與標籤模板 | [https://blog.csdn.net/weixin_46803507/article/details/106579538](https://blog.csdn.net/weixin_46803507/article/details/106579538)                                         |
| 三种CSS样式-内联、嵌入、外部        | [https://www.cnblogs.com/dyfblogs/p/11391655.html](https://www.cnblogs.com/dyfblogs/p/11391655.html)                                                                       |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/es6/es6_template_string](https://github.com/superfreeeee/Blog-code/tree/main/front_end/es6/es6_template_string)
