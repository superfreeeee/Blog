# select 源码解析

@[TOC](文章目录)

<!-- TOC -->

- [select 源码解析](#select-源码解析)
- [正文](#正文)
  - [0. 功能](#0-功能)
  - [1. 代码结构](#1-代码结构)
    - [1.1 HTMLSelectElement](#11-htmlselectelement)
    - [1.2 HTMLInputElement、HTMLTextAreaElement](#12-htmlinputelementhtmltextareaelement)
    - [1.3 其他 Element](#13-其他-element)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 正文

## 0. 功能

select 库的功能是使用户能够以编程命令式的方式选择某个 html 元素节点，并同步获取元素内容文本

## 1. 代码结构

- `/src/select.js`（阅读笔记：`/src/select.js`）

```js
function select(element) {
  var selectedText;

  if (element.nodeName === 'SELECT') {
    // ...
  } else if (element.nodeName === 'INPUT' || element.nodeName === 'TEXTAREA') {
    // ...
  } else {
    // ...
  }

  return selectedText;
}

module.exports = select;
```

代码结构就一个 `select.js` 文件，主流程分成三种目标元素来处理

### 1.1 HTMLSelectElement

对于 `<select>` 元素

- `/src/select.js`（阅读笔记：`/src/select.js`）

```js
  if (element.nodeName === 'SELECT') {
    element.focus();

    selectedText = element.value;
```

直接使用 select 元素的 value 代表选取的目标文本

### 1.2 HTMLInputElement、HTMLTextAreaElement

- `/src/select.js`（阅读笔记：`/src/select.js`）

```js
  } else if (element.nodeName === 'INPUT' || element.nodeName === 'TEXTAREA') {
    var isReadOnly = element.hasAttribute('readonly');

    if (!isReadOnly) {
      element.setAttribute('readonly', '');
    }

    element.select();
    element.setSelectionRange(0, element.value.length);

    if (!isReadOnly) {
      element.removeAttribute('readonly');
    }

    selectedText = element.value;
```

对于 input 和 textarea 元素，首先需要在选取的过程确保 readonly，避免内容被修改；使用

```js
    element.select();
    element.setSelectionRange(0, element.value.length);
```

选择输入框后选取输入框的内容文本

### 1.3 其他 Element

- `/src/select.js`（阅读笔记：`/src/select.js`）

```js
    if (element.hasAttribute('contenteditable')) {
      element.focus();
    }

    var selection = window.getSelection();
    var range = document.createRange();

    range.selectNodeContents(element);
    selection.removeAllRanges();
    selection.addRange(range);

    selectedText = selection.toString();
```

对于其他 Element，我们使用了 Selection API 来实现，先获取 window 上的 Selection 对象(`window.getSelection`)，然后创建 Range 对象(`document.createRange`)，使用 `range.selectNodeContents` 方法选取目标元素，并挂载到 selection 上 `selection.addRange(range)`

# 其他资源

## 参考连接

| Title                           | Link                                                                                                                                                 |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| select - npm                    | [https://www.npmjs.com/package/select](https://www.npmjs.com/package/select)                                                                         |
| zenorocha/select - github       | [https://github.com/zenorocha/select](https://github.com/zenorocha/select)                                                                           |
| HTMLSelectElement - MDN         | [https://developer.mozilla.org/en-US/docs/Web/API/HTMLSelectElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLSelectElement)             |
| HTMLInputElement.select() - MDN | [https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLInputElement/select](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLInputElement/select) |
| HTMLTextAreaElement - MDN       | [https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLTextAreaElement](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLTextAreaElement)         |
| HTMLElement - MDN               | [https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement)                         |
| Window.getSelection - MDN       | [https://developer.mozilla.org/zh-CN/docs/Web/API/Window/getSelection](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/getSelection)         |
| Document.createRange() - MDN    | [https://developer.mozilla.org/zh-CN/docs/Web/API/Document/createRange](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/createRange)       |
| Range - MDN                     | [https://developer.mozilla.org/zh-CN/docs/Web/API/Range](https://developer.mozilla.org/zh-CN/docs/Web/API/Range)                                     |

## 阅读笔记参考

[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/select-1.1.2](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/select-1.1.2)
