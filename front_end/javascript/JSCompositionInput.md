# JS 事件: Composition Input 组合输入事件(中文输入事件监听)

@[TOC](文章目录)

<!-- TOC -->

- [JS 事件: Composition Input 组合输入事件(中文输入事件监听)](#js-事件-composition-input-组合输入事件中文输入事件监听)
- [前言](#前言)
- [正文](#正文)
  - [组合输入事件：compositionstart、compositionupdate、compositionend](#组合输入事件compositionstartcompositionupdatecompositionend)
  - [输入阶段](#输入阶段)
  - [代码示例](#代码示例)
    - [1. 非受控组件](#1-非受控组件)
    - [2. 受控组件](#2-受控组件)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

今天跟大家分享一个比较有趣的小知识。我们都知道 html 中监听 `<input>` 标签的输入相关事件有 `input`、`change` 等，其中在 React 中我们又通常都使用 `onChange` 来监听输入事件。

今天要介绍的是与中文输入类似的组合输入事件监听

# 正文

## 组合输入事件：compositionstart、compositionupdate、compositionend

废话不多说，主角直接上场。

在 `<input>` 标签下我们有三个事件名称可以监听组合输入的事件，不论是简体中文或是繁体中文或是其他任何输入会被替换成最终输入的文字的语言都一样

- `compositionstart`：组合输入开始
- `compositionupdate`：组合输入更新
- `compositionend`：组合输入结束

## 输入阶段

接下来我们明确一下组合输入的阶段

1. 开始阶段：处于特别的输入法并敲下第一个字符，这时候就会触发 `compositionstart` 事件
2. 更新阶段：在组合输入的状态下继续输入更多字符，这时候触发的是 `compositionupdate`
3. 结束阶段：这边要注意的一个点，就是有时候字生成不一定代表输入结束，像繁体中文输入法中输入完成生成中文字符之后还可以进行选字，这时候实际上要再输入 enter 结束输入状态，这时候才会触发 `compositionend`

## 代码示例

接下来我们就在 React 中试试

### 1. 非受控组件

- `/src/App.tsx`

第一个实验我们先不对 input 的 value 进行控制，也就是所谓的不受控组件，而仅仅只是监听事件来看看他的顺序

```ts
import React, { useState } from 'react'

export default function App() {
  const handleChange = (e) => {
    console.log('onChange:', e.target.value)
  }

  const handleCompositionStart = (e) => {
    console.log('onCompositionStart:', e.target.value)
  }

  const handleCompositionUpdate = (e) => {
    console.log('onCompositionUpdate:', e.target.value)
  }

  const handleCompositionEnd = (e) => {
    console.log('onCompositionEnd:', e.target.value)
  }

  return (
    <div>
      <h1>JS 事件: Composition Input 组合输入事件</h1>
      <input
        type="text"
        onChange={handleChange}
        onCompositionStart={handleCompositionStart}
        onCompositionUpdate={handleCompositionUpdate}
        onCompositionEnd={handleCompositionEnd}
      />
    </div>
  )
}
```

- 非组合文字输入

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_composition_input_sample1.gif)

我们看到实际上普通的输入是不会触发组合输入相关的事件的

- 中文输入

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_composition_input_sample2.gif)

我们改成输入简体中文就可以看到触发了组合事件

### 2. 受控组件

这时候你可能会想，那 React 推荐的受控组件输入法会不会影响组件，这时候我们可以看看下面的例子

- `/src/App.tsx`

```ts
import React, { useState } from 'react'

export default function App() {
  const [value, setValue] = useState('')

  const handleChange = (e) => {
    console.log('onChange:', e.target.value)
    setValue(e.target.value)
  }

  const handleCompositionStart = (e) => {
    console.log('onCompositionStart:', e.target.value)
  }

  const handleCompositionUpdate = (e) => {
    console.log('onCompositionUpdate:', e.target.value)
  }

  const handleCompositionEnd = (e) => {
    console.log('onCompositionEnd:', e.target.value)
  }

  return (
    <div>
      <h1>JS 事件: Composition Input 组合输入事件</h1>
      <input
        type="text"
        value={value}
        onChange={handleChange}
        onCompositionStart={handleCompositionStart}
        onCompositionUpdate={handleCompositionUpdate}
        onCompositionEnd={handleCompositionEnd}
      />
    </div>
  )
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_composition_input_sample3.gif)

我们可以看到，虽然 change 确实也会捕捉输入过程的中间状态，但是实际上是不会打断组合输入的。

# 结语

更多关于组合输入事件监听的特性可以自己尝试，其实对于一般的输入处理我们并不需要特别利用组合输入来监听事件，但是我们有时候会希望只在特定时间例如文字完成瞬间才检查输入的时候，就可以用到相关事件监听来实现。

本篇的宗旨主要是抛出来分享有这么个事件能利用，供大家参考。

# 其他资源

## 参考连接

| Title                  | Link                                                                                                                                   |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| CompositionEvent - MDN | [https://developer.mozilla.org/en-US/docs/Web/API/CompositionEvent](https://developer.mozilla.org/en-US/docs/Web/API/CompositionEvent) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_composition_input](https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_composition_input)
