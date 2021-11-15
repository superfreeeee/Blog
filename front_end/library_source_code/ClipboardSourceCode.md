# Clipboard.js 源码解析

@[TOC](文章目录)

<!-- TOC -->

- [Clipboard.js 源码解析](#clipboardjs-源码解析)
- [正文](#正文)
  - [0. 源码解析都在看什么？](#0-源码解析都在看什么)
  - [1. 源码解析路线 & 手段](#1-源码解析路线--手段)
  - [2. Clipboard 源码解析](#2-clipboard-源码解析)
    - [2.1 源码目录结构](#21-源码目录结构)
    - [2.2 基础使用](#22-基础使用)
    - [2.3 源码入口：Clipboard 类](#23-源码入口clipboard-类)
    - [2.4 配置参数解析 resolveOptions](#24-配置参数解析-resolveoptions)
      - [2.4.1 defaultAction](#241-defaultaction)
      - [2.4.2 defaultTarget](#242-defaulttarget)
      - [2.4.3 defaultText](#243-defaulttext)
    - [2.5 监听事件 listenClick](#25-监听事件-listenclick)
    - [2.6 事件触发 onClick](#26-事件触发-onclick)
      - [2.6.1 默认事件 ClipboardActionDefault](#261-默认事件-clipboardactiondefault)
      - [2.6.2 复制 copy - ClipboardActionCopy](#262-复制-copy---clipboardactioncopy)
      - [2.6.3 纯文本元素载体 createFakeElement](#263-纯文本元素载体-createfakeelement)
      - [2.6.4 复制/剪贴核心 API：document.execCommand](#264-复制剪贴核心-apidocumentexeccommand)
      - [2.6.3 剪贴 cut - ClipboardActionCut](#263-剪贴-cut---clipboardactioncut)
      - [2.6. 事件回调 emit('success' | 'error')](#26-事件回调-emitsuccess--error)
    - [2.7 静态方法 copy、cut](#27-静态方法-copycut)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [阅读笔记参考](#阅读笔记参考)

<!-- /TOC -->

# 正文

## 0. 源码解析都在看什么？

本篇的主要目标和关注点，但不仅限于以下几点

- 源代码组织结构（代码架构）
- 三方库工程化构建方式
- 功能实现核心 API

## 1. 源码解析路线 & 手段

- 从应用着手

根据实际使用该三方库的方式为起点探入实现细节

- 打断点调试观察运行过程

可以使用一些 IDE 附带的打断点方式，跟随源代码的执行顺序更直接的体验代码运行过程和实际步骤

## 2. Clipboard 源码解析

下面马上进入源码细节，Clipboard.js 本身并不是太复杂的三方库，可以说是比较轻便好用的

### 2.1 源码目录结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/clipboard_source_code_1_package_structure.png)

源码的目录结构如下，还是比较简单的

- `/src/clipboard.js` 为主入口
- `/common` 为辅助函数
- `/actions` 为核心操作方法

### 2.2 基础使用

Clipboard.js 的使用方式可以参考[官方推荐用法](https://www.npmjs.com/package/clipboard)下面我们直接进入代码细节

### 2.3 源码入口：Clipboard 类

根据基础使用我们知道整个库的核心在于一个 Clipboard 类，结构如下

- `/src/clipboard.js`（阅读笔记：`/src/clipboard.js/1_structure.js`）

```js
import Emitter from 'tiny-emitter';
import listen from 'good-listener';
import ClipboardActionDefault from './actions/default';
import ClipboardActionCut from './actions/cut';
import ClipboardActionCopy from './actions/copy';

function getAttributeValue(suffix, element) {/* ... */}

class Clipboard extends Emitter {
  constructor(trigger, options) {/* ... */}
  resolveOptions(options = {}) {/* ... */}
  listenClick(trigger) {/* ... */}
  onClick(e) {/* ... */}
  defaultAction(trigger) {/* ... */}
  defaultTarget(trigger) {/* ... */}
  static copy(target, options = { container: document.body }) {/* ... */}
  static cut(target) {/* ... */}
  static isSupported(action = ['copy', 'cut']) {/* ... */}
  defaultText(trigger) {/* ... */}
  destroy() {/* ... */}
}

export default Clipboard;
```

首先我们从构造函数开始看

- `/src/clipboard.js`（阅读笔记：`/src/clipboard.js/3_constructor.js`）

```js
class Clipboard extends Emitter {
  // ...

  constructor(trigger, options) {
    super();

    this.resolveOptions(options);
    this.listenClick(trigger);
  }

  // ...
}

export default Clipboard;
```

核心步骤有两步：解析配置参数 optiosn、监听点击事件

### 2.4 配置参数解析 resolveOptions

- `/src/clipboard.js`（阅读笔记：`/src/clipboard.js/4_resolveOptions.js`）

```js
class Clipboard extends Emitter {
  // ...

  resolveOptions(options = {}) {
    // 操作
    this.action =
      typeof options.action === 'function'
        ? options.action
        : this.defaultAction;
    // 目标
    this.target =
      typeof options.target === 'function'
        ? options.target
        : this.defaultTarget;
    // 文本
    this.text =
      typeof options.text === 'function' ? options.text : this.defaultText;
    // 容器
    this.container =
      typeof options.container === 'object' ? options.container : document.body;
  }

  // ...
}

export default Clipboard;
```

根据官方用法对照，核心配置参数有四个

- `action` 操作行为，可选值有 `'copy' | 'cut'`
- `target` 操作目标，也就是复制的文本对象参考
- `text` 则是不指定目标的时候可以直接指定复制文本
- `container` 则是挂载元素的容器，通常直接使用默认的 body 就行了

我们还能看到前三个参数都提供了缺省实现

#### 2.4.1 defaultAction

- `/src/clipboard.js`（阅读笔记：`/src/clipboard.js/4_1_defaultAction.js`）

```js
class Clipboard extends Emitter {
  // ...

  defaultAction(trigger) {
    return getAttributeValue('action', trigger);
  }

  // ...
}

export default Clipboard;
```

我们可以看到 defaultAciton 的缺省实现就是调用 `getAttributeValue` 方法

```js

function getAttributeValue(suffix, element) {
  const attribute = `data-clipboard-${suffix}`;

  if (!element.hasAttribute(attribute)) {
    return;
  }

  return element.getAttribute(attribute);
}
```

`getAttributeValue` 我们可以看到本质上就是从目标元素拿到 `data-clipboard-xxx` 的属性值，正好与使用范例一一对应

#### 2.4.2 defaultTarget

- `/src/clipboard.js`（阅读笔记：`/src/clipboard.js/4_2_defaultTarget.js`）

```js
class Clipboard extends Emitter {
  // ...

  defaultTarget(trigger) {
    const selector = getAttributeValue('target', trigger);

    if (selector) {
      return document.querySelector(selector);
    }
  }

  // ...
}

export default Clipboard;
```

`data-clipboard-target` 的值为目标元素的 CSS 选择器，在此转换成目标元素

#### 2.4.3 defaultText

- `/src/clipboard.js`（阅读笔记：`/src/clipboard.js/4_3_defaultText.js`）

```js
class Clipboard extends Emitter {
  // ...

  defaultText(trigger) {
    return getAttributeValue('text', trigger);
  }

  // ...
}

export default Clipboard;
```

defaultText 则是直接取 `data-clipboard-text` 的属性值

### 2.5 监听事件 listenClick

经过 resolveOptions 解析配置参数后，我们已经确立了 `this.action、this.target、this.text、this.container` 四个属性

第二步就是监听目标元素的点击事件

- `/src/clipboard.js`（阅读笔记：`/src/clipboard.js/5_listenClick.js`）

```js
class Clipboard extends Emitter {
  // ...

  listenClick(trigger) {
    this.listener = listen(trigger, 'click', (e) => this.onClick(e));
  }

  destroy() {
    this.listener.destroy();
  }

  // ...
}

export default Clipboard;
```

这里用了 `good-listener` 库的 listen 方法，有兴趣可以再去爬爬源码；同时提供了 destory 方法来销毁监听函数

### 2.6 事件触发 onClick

在构造函数中注册好后，之后就是点击按钮后触发复制/剪贴的操作了，也就对应着 onClick 方法的实现

- `/src/clipboard.js`（阅读笔记：`/src/clipboard.js/6_onClick.js`）

```js
class Clipboard extends Emitter {
  // ...

  onClick(e) {
    const trigger = e.delegateTarget || e.currentTarget;
    const selectedText = ClipboardActionDefault({
      action: this.action(trigger),
      container: this.container,
      target: this.target(trigger),
      text: this.text(trigger),
    });

    // Fires an event based on the copy operation result.
    this.emit(selectedText ? 'success' : 'error', {
      action: this.action,
      text: selectedText,
      trigger,
      clearSelection() {
        if (trigger) {
          trigger.focus();
        }
        document.activeElement.blur();
        window.getSelection().removeAllRanges();
      },
    });
  }

  // ...
}

export default Clipboard;
```

整个方法还是很单纯，一个是调用 `ClipboardActionDefault` 方法触发剪贴行为，第二个则是使用 `tiny-emitter` 库提供的事件机制来触发用户透过 `on` 方法监听的回调函数

#### 2.6.1 默认事件 ClipboardActionDefault

这里最核心的函数就是这个 `ClipboardActionDefault`，前面我们看到 ClipboardActionDefault 接受的参数都是经过调用获取实时的值

- `/src/actions/default.js`（阅读笔记：`/src/actions/default.js`）

```js
import ClipboardActionCut from './cut';
import ClipboardActionCopy from './copy';

const ClipboardActionDefault = (options = {}) => {
  const { action = 'copy', container, target, text } = options;

  // 验证 action
  if (action !== 'copy' && action !== 'cut') {
    throw new Error('Invalid "action" value, use either "copy" or "cut"');
  }

  // 验证 target
  if (target !== undefined) {
    // 必须是 Element 元素
    if (target && typeof target === 'object' && target.nodeType === 1) {
      if (action === 'copy' && target.hasAttribute('disabled')) {
        throw new Error(
          'Invalid "target" attribute. Please use "readonly" instead of "disabled" attribute'
        );
      }

      if (
        action === 'cut' &&
        (target.hasAttribute('readonly') || target.hasAttribute('disabled'))
      ) {
        throw new Error(
          'Invalid "target" attribute. You can\'t cut text from elements with "readonly" or "disabled" attributes'
        );
      }
    } else {
      throw new Error('Invalid "target" value, use a valid Element');
    }
  }

  // 直接指定文本优先
  if (text) {
    return ClipboardActionCopy(text, { container });
  }

  // 按目标元素剪贴
  if (target) {
    return action === 'cut'
      ? ClipboardActionCut(target)
      : ClipboardActionCopy(target, { container });
  }
};

export default ClipboardActionDefault;
```

整个 `ClipboardActionDefault` 方法都在做参数的校验，真正发挥功能的则是最后的两块，分别使用了 `ClipboardActionCopy` 来复制、`ClipboardActionCut` 来剪贴

#### 2.6.2 复制 copy - ClipboardActionCopy

- `/src/actions/copy.js`（阅读笔记：`/src/actions/copy.js`）

```js
import select from 'select';
import command from '../common/command';
import createFakeElement from '../common/create-fake-element';

const ClipboardActionCopy = (
  target,
  options = { container: document.body }
) => {
  let selectedText = '';
  if (typeof target === 'string') {
    // 纯文本
    const fakeElement = createFakeElement(target);
    options.container.appendChild(fakeElement);
    selectedText = select(fakeElement);
    command('copy');
    fakeElement.remove();
  } else {
    // 目标元素文本
    selectedText = select(target);
    command('copy');
  }
  return selectedText;
};

export default ClipboardActionCopy;
```

复制行为可能是针对纯文本，又或是目标元素的文本；这里使用了 `select` 库来实现选择目标元素行为

#### 2.6.3 纯文本元素载体 createFakeElement

对于纯文本则还要提供一个额外的文本框外壳来实现

- `/src/common/create-fake-element.js`（阅读笔记：`/src/common/create-fake-element.js`）

```js
export default function createFakeElement(value) {
  const isRTL = document.documentElement.getAttribute('dir') === 'rtl';
  const fakeElement = document.createElement('textarea');
  // Prevent zooming on iOS
  fakeElement.style.fontSize = '12pt';
  // Reset box model
  fakeElement.style.border = '0';
  fakeElement.style.padding = '0';
  fakeElement.style.margin = '0';
  // Move element out of screen horizontally
  fakeElement.style.position = 'absolute';
  fakeElement.style[isRTL ? 'right' : 'left'] = '-9999px';
  // Move element to the same position vertically
  let yPosition = window.pageYOffset || document.documentElement.scrollTop;
  fakeElement.style.top = `${yPosition}px`;

  fakeElement.setAttribute('readonly', '');
  fakeElement.value = value;

  return fakeElement;
}
```

本质上就是提供一个 `textarea` 元素作为文本的载体

#### 2.6.4 复制/剪贴核心 API：document.execCommand

最后使用 `document.execCommand` 作为实现功能的核心 API

- `/src/common/command.js`（阅读笔记：`/src/common/command.js`）

```js
export default function command(type) {
  try {
    return document.execCommand(type);
  } catch (err) {
    return false;
  }
}
```

同时 Clipboard 类也提供的对于 API 兼容性的检查

- `/src/clipboard.js`（阅读笔记：`/src/clipboard.js/2_support.js`）

```js
class Clipboard extends Emitter {
  // ...

  static isSupported(action = ['copy', 'cut']) {
    const actions = typeof action === 'string' ? [action] : action;
    let support = !!document.queryCommandSupported;

    actions.forEach((action) => {
      support = support && !!document.queryCommandSupported(action);
    });

    return support;
  }

  // ...
}

export default Clipboard;
```

#### 2.6.3 剪贴 cut - ClipboardActionCut

剪贴事件则比较单纯，只可能是针对元素目标来执行

- `/src/actions/cut.js`（阅读笔记：`/src/actions/cut.js`）

```js
const ClipboardActionCut = (target) => {
  const selectedText = select(target);
  command('cut');
  return selectedText;
};

export default ClipboardActionCut;
```

#### 2.6. 事件回调 emit('success' | 'error')

- `/src/clipboard.js`（阅读笔记：`/src/clipboard.js/6_onClick.js`）

```js
class Clipboard extends Emitter {
  // ...

  onClick(e) {
    // ...

    this.emit(selectedText ? 'success' : 'error', {
      action: this.action,
      text: selectedText,
      trigger,
      clearSelection() {
        if (trigger) {
          trigger.focus();
        }
        document.activeElement.blur();
        window.getSelection().removeAllRanges();
      },
    });
  }

  // ...
}

export default Clipboard;
```

剪贴事件本身完成之后才会调用回调函数 `'success' | 'error'`，借用了 `tiny-emitter` 的事件通道

### 2.7 静态方法 copy、cut

最后的最后 Clipboard 还提供静态方法共使用者进行命令式调用

- `/src/clipboard.js`（阅读笔记：`/src/clipboard.js/7_static.js`）

```js
class Clipboard extends Emitter {
  // ...

  static copy(target, options = { container: document.body }) {
    return ClipboardActionCopy(target, options);
  }

  static cut(target) {
    return ClipboardActionCut(target);
  }

  // ...
}

export default Clipboard;
```

# 其他资源

## 参考连接

| Title                            | Link                                                                                                         |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| clipboard - npm                  | [https://www.npmjs.com/package/clipboard](https://www.npmjs.com/package/clipboard)                           |
| good-listener - npm              | [https://www.npmjs.com/package/good-listener](https://www.npmjs.com/package/good-listener)                   |
| tiny-emitter - npm               | [https://www.npmjs.com/package/tiny-emitter](https://www.npmjs.com/package/tiny-emitter)                     |
| select - npm                     | [https://www.npmjs.com/package/select](https://www.npmjs.com/package/select)                                 |
| elementNode.nodeType值对应的类型 | [https://blog.csdn.net/hqmln/article/details/83915637](https://blog.csdn.net/hqmln/article/details/83915637) |

## 阅读笔记参考

[https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/clipboard-2.0.8](https://github.com/superfreeeee/Blog-code/tree/main/source_code_research/clipboard-2.0.8)
