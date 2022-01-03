# HTML 进阶: Web Components 原生组件技术

@[TOC](文章目录)

<!-- TOC -->

- [HTML 进阶: Web Components 原生组件技术](#html-进阶-web-components-原生组件技术)
- [Web Components 概念 & 技术核心](#web-components-概念--技术核心)
- [1. Custom Elements 自定义标签](#1-custom-elements-自定义标签)
  - [1.1 注册 Web Component](#11-注册-web-component)
  - [1.2 生命周期钩子](#12-生命周期钩子)
- [2. Shadow DOM](#2-shadow-dom)
- [3. Template 模版 & Slot 插槽](#3-template-模版--slot-插槽)
  - [3.1 Template 模版的作用](#31-template-模版的作用)
  - [3.2 Slot 插槽的作用](#32-slot-插槽的作用)
- [小结](#小结)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# Web Components 概念 & 技术核心

前端开发中存在许多框架，大多数的响应式框架不外乎是用于开发"组件"。今天要来介绍的是浏览器提供的原生 JS 的组件开发标准：Web Components

在前端原生开发里面，与组件最近似的概念就是所谓的标签，或是我们可以说组件应该是一组标签与内部数据逻辑为一个整体。

Web Components 的实现技术核心主要有三大点

1. Custom Elements 自定义标签
2. Shadow DOM 影子DOM
3. Template & Slot 模版与插槽

其中 Custom Elements 还提供了所谓的生命周期钩子，让我们能监听自定义组件的属性变化、创建/销毁时机等

下面我们透过实际编码来一一介绍

# 1. Custom Elements 自定义标签

第一项技术是自定义标签

## 1.1 注册 Web Component

自定义标签的入口是透过 `CustomElementRegistry.prototype.define` 注册一个自定义的组件，用法如下

- `index.js`

```js
customElements.define('popup-info', PopUpInfo);
```

而作为第二个参数的组件类型，则至少必须继承 `HTMLElement` 类型

```js
class PopUpInfo extends HTMLElement {/* ... */}
```

当然你也可以继承更具体的其他内置类型，如下

```js
class WordCount extends HTMLParagraphElement {/* ... */}
customElements.define('word-count', WordCount, { extends: 'p' });
```

## 1.2 生命周期钩子

对于这个继承 HTMLElement 的组件类，我们可以利用生命周期钩子来监听组件的变化

- `connectedCallback` 在元素添加到页面的时候触发
- `disconnectedCallback` 在元素移除的时候触发
- `adoptedCallback` 在元素移动的时候触发
- `attributeChangedCallback` 在元素属性修改的时候触发
  - `static observedAttributes` 返回需要观察的属性列表

下面为一个简单的例子

- `index.js`

```js
class Square extends HTMLElement {
  // Specify observed attributes so that
  // attributeChangedCallback will work
  static get observedAttributes() {
    return ['c', 'l'];
  }

  constructor() {
    // Always call super first in constructor
    super();

    const shadow = this.attachShadow({ mode: 'open' });

    const div = document.createElement('div');
    const style = document.createElement('style');
    shadow.appendChild(style);
    shadow.appendChild(div);
  }

  connectedCallback() {
    console.log('Custom square element added to page.');
    updateStyle(this);
  }

  disconnectedCallback() {
    console.log('Custom square element removed from page.');
  }

  adoptedCallback() {
    console.log('Custom square element moved to new page.');
  }

  attributeChangedCallback(name, oldValue, newValue) {
    console.log('Custom square element attributes changed.');
    console.log(name, oldValue, newValue);
    updateStyle(this);
  }
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/html_web_components_1_lifecycle.png)

从上图能看到每次点击 update，修改 `l` 与 `c` 各会触发一次 `attributeChangedCallback` 钩子

![](https://picures.oss-cn-beijing.aliyuncs.com/img/html_web_components_2_lifecycle_remove.png)

点击移除的时候调用 `remove`，则触发了 `disconnectedCallback` 钩子

# 2. Shadow DOM

前面介绍完自定义标签作为我们的组件载体，以及生命周期钩子的运用；接下来则是填充组件的血肉，也就是具体的元素集合与样式。

这里用到的 Shadow DOM 技术实际上早就在浏览器环境下投入使用，例如 `<video>` 元素的播放控件，实际上就是隐藏的 Shadow DOM 部分。

下面我们就利用 Shadow DOM 来向自定义标签的 Shadow DOM 添加子元素

- `index.html`

首先先在 html 写好我们要的标签（如果当前浏览器不支持 Web Components 或是自定义标签尚未注册时，则会作为默认标签处理）

```html
<popup-info
  img="static/default.jpg"
  text="Your card validation code (CVC) is an extra
security feature — it is the last 3 or 4
numbers on the back of your card."
></popup-info>
```

- `index.js`

接下来我们进行注册，创建了一个弹出框，在 hover 的时候浮出

```js
class PopUpInfo extends HTMLElement {
  constructor() {
    // Always call super first in constructor
    super();

    // Create a shadow root
    var shadow = this.attachShadow({ mode: 'open' });

    // Create spans
    var wrapper = document.createElement('span');
    wrapper.setAttribute('class', 'wrapper');
    var icon = document.createElement('div');
    icon.setAttribute('class', 'icon');
    icon.setAttribute('tabindex', 0);
    var info = document.createElement('div');
    info.setAttribute('class', 'info');

    // Take attribute content and put it inside the info span
    var text = this.getAttribute('text');
    info.textContent = text;

    // Insert icon
    var imgUrl;
    if (this.hasAttribute('img')) {
      imgUrl = this.getAttribute('img');
    } else {
      imgUrl = 'img/default.png';
    }
    var img = document.createElement('img');
    img.src = imgUrl;
    icon.appendChild(img);

    // Create some CSS to apply to the shadow dom
    var style = document.createElement('style');

    style.textContent = `
    .wrapper {
      position: relative;
      display: inline-block;
    }
    .icon {
      height: 100%;
    }
    .info {
      position: absolute;
      bottom: 20px;
      right: -10px;
      display: inline-block;
      width: 200px;
      padding: 10px;
      border: 1px solid black;
      border-radius: 10px;
      transform: translateX(100%);
      transition: 0.6s all;
      background: white;
      opacity: 0;
      font-size: 0.8rem;
      z-index: 3;
    }
    img {
      width: 2rem
    }
    .icon:hover + .info,
    .icon:focus + .info {
      opacity: 1;
    }
    `;

    // attach the created elements to the shadow dom

    shadow.appendChild(style);
    shadow.appendChild(wrapper);
    wrapper.appendChild(icon);
    wrapper.appendChild(info);
  }
}

// Define the new element
customElements.define('popup-info', PopUpInfo);
```

下图是上述组件的预览效果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/html_web_components_3_shadow_dom_sample.png)

我们打开调试面板，可以看到前面代码中创建并插入子元素的 shadow 将作为元素的 `shadow-root` 元素插入到实际 DOM 当中

![](https://picures.oss-cn-beijing.aliyuncs.com/img/html_web_components_4_shadow_dom_detail.png)

这里需要注意的是 Shadow DOM 内部的元素与外部 DOM 是隔离的空间，也就是外部的样式表、`querySelector` 方法等与 Shadow DOM 内部是不互通的；作为替代方案我们可以在 Shadow DOM 使用 `<link>` 引入外部样式表即可、另外我们还可以先查找到目标元素，使用 `targetElement.shadowRoot.querySelector` 来查找 Shadow DOM 内部元素（需要使用 `mode: open`，当然还有其他 hack 方式[参考链接](https://blog.revillweb.com/open-vs-closed-shadow-dom-9f3d7427d1af)）

# 3. Template 模版 & Slot 插槽

最后一部分是关于模版与插槽在 web components 上的应用。

## 3.1 Template 模版的作用

前面我们可以看到就算是使用了 Web Components 的技术，如果只使用 JavaScript 的话还是免不了不断调用 `document.createElement` 这种琐碎的操作，但是通常我们又不愿意轻易使用大开销的 `innerHTML`，这时候我们就可以用到一种特别的标签 `<template>` 标签

- `index.html`

```html
<template id="element-details-template">
  <style>
    /* skip styles ... */
  </style>
  <details>
    <summary>
      <span>
        <code class="name">
          &lt;<slot name="element-name">NEED NAME</slot>&gt;
        </code>
        <i class="desc">
          <slot name="description">NEED DESCRIPTION</slot>
        </i>
      </span>
    </summary>
    <div class="attributes">
      <h4><span>Attributes</span></h4>
      <slot name="attributes"><p>None</p></slot>
    </div>
  </details>
  <hr />
</template>
```

顾名思义它就是一个模版，在浏览器中不会将 template 标签的内容打印出来，但是我们一样可以使用 `querySelector` 来获取子 DOM 树，拷贝一份下来就可以用于创建新的组件元素啦！

因此我们有如下代码

- `index.js`

```js
class ElementDetails extends HTMLElement {
  constructor() {
    super();

    const template = document.getElementById(
      'element-details-template'
    ).content;

    const shadowRoot = this.attachShadow({ mode: 'closed' }).appendChild(
      template.cloneNode(true)
    );
    this._root = shadowRoot;
  }
}
customElements.define('element-details', ElementDetails);
```

透过获取 `template` 元素，我们就不再需要一个个的创建元素了

## 3.2 Slot 插槽的作用

我们可以看到前面的例子在 `<template>` 模版当中出现了 `<slot>` 标签，他也是另一个比较特别的标签之一，对于 `<slot>` 或是命名插槽 `<slot name>`，实际渲染的时候他会检测是否有用户传入的元素用于替换插槽，或是渲染原始插槽中默认的内容

例如我们在 HTML 中插入两段上述组件的实例

- `index.html`

```html
<element-details>
  <span slot="element-name">slot</span>
  <span slot="description">
    A placeholder inside a web component that users can fill with their
    own markup, with the effect of composing different DOM trees together.
  </span>
  <dl slot="attributes">
    <dt>name</dt>
    <dd>The name of the slot.</dd>
  </dl>
</element-details>

<element-details>
  <span slot="element-name">template</span>
  <span slot="description">
    A mechanism for holding client- side content that is not to be
    rendered when a page is loaded but may subsequently be instantiated
    during runtime using JavaScript.
  </span>
</element-details>
```

透过 `slot="xxx"` 的方式指定插槽位置，可以得到如下结果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/html_web_components_5_template_slot.png)

我们可以看到第一个组件将每个插槽都填满了，第二个组件没有传入 `slot="attributes"` 的插槽内容，因此渲染了默认内容 `None`

# 小结

本篇对于 Web Components 技术的介绍就到此为止。该技术在目前来说还是相对较新，标准也还不算完全定下来，使用上可能也没有其他大框架这么广泛。但是作为原生 JS 的技术，实际上与现有组件定义结构大同小异，甚至就算在 React、Vue 这种框架底下，将组件的编译逻辑映射成 Web Components 的实现也不是不行。

以后也继续跟进 Web 技术的发展，供大家参考哈

# 其他资源

## 参考连接

| Title                                  | Link                                                                                                                                   |
| -------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| Web Components - MDN                   | [https://developer.mozilla.org/zh-CN/docs/Web/Web_Components](https://developer.mozilla.org/zh-CN/docs/Web/Web_Components)             |
| Open vs. Closed Shadow DOM             | [https://blog.revillweb.com/open-vs-closed-shadow-dom-9f3d7427d1af](https://blog.revillweb.com/open-vs-closed-shadow-dom-9f3d7427d1af) |
| javascript在指定的元素前或后插入新元素 | [https://www.feiniaomy.com/post/109.html](https://www.feiniaomy.com/post/109.html)                                                     |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/html/html_web_components](https://github.com/superfreeeee/Blog-code/tree/main/front_end/html/html_web_components)
