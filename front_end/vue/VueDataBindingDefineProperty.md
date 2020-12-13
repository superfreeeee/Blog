# Vue 源码实现: Data Binding 双向数据绑定（使用 Object.defineProperty 实现）

@[TOC](文章目录)

<!-- TOC -->

- [Vue 源码实现: Data Binding 双向数据绑定（使用 Object.defineProperty 实现）](#vue-源码实现-data-binding-双向数据绑定使用-objectdefineproperty-实现)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [实现目标](#实现目标)
  - [实现架构](#实现架构)
  - [具体实现](#具体实现)
    - [项目结构 & 静态内容初始化](#项目结构--静态内容初始化)
    - [核心代码](#核心代码)
    - [对象间交互顺序 & 图](#对象间交互顺序--图)
      - [input 输入](#input-输入)
      - [修改 `$data` 属性](#修改-data-属性)
- [结语](#结语)

<!-- /TOC -->

## 简介

本篇要来干大事了（bushi，大名鼎鼎的 Vue 作为一个最潮的 MVVM 框架，实现了双向数据绑定和虚拟 dom 等多项复杂技术。本篇将要尝试实现 Vue2 所使用的方法(借助 `Object.defineProperty`)来实现双向数据绑定，走起

## 参考

<table>
  <tr>
    <td>vue 双向数据绑定的实现学习（二）- 监听器的实现</td>
    <td><a href="https://www.cnblogs.com/adouwt/p/10039900.html">https://www.cnblogs.com/adouwt/p/10039900.html</a></td>
  </tr>
</table>

# 正文

## 实现目标

首先我们先来明确我们的实现目标：`v-model` 实现双向数据绑定，也就是`修改绑定变量`时同时`更新输入框的值`；或是`输入框输入`时`更新绑定变量的值`，图示如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/data_binding_object_simple.png)

我们的目标就是将左边的 `input` 输入框与右侧 data 中的 `name` 变量绑定起来（透过形如 `<input v-model="name">` 的形式）

那么接下来我们要做的就是创建一个管理中心(MVVM)来帮我们完成这件事情：

![](https://picures.oss-cn-beijing.aliyuncs.com/img/data_binding_object_abstract.png)

## 实现架构

在开始上代码之前，我们先来谈谈抽象的实现架构。前面我们提到我们的实现目标就是创建一个 MVVM 来帮我们完成双向绑定的工作，而这个 MVVM 内部又可以细分成三种成员：

- `Compiler 模版渲染器`：通过编译特定模版代码之后渲染成实际 dom
- `Watcher 观察者`：观察`绑定变量`是否被修改，并通知 Compiler 重新渲染
- `Dispatcher 调度中心`：负责决定通知哪个观察者更新，`绑定变量`修改时通知调度中心进而通知观察者更新，如下图所示

![](https://picures.oss-cn-beijing.aliyuncs.com/img/data_binding_object.png)

## 具体实现

接下来我们就来着手实现模拟 Vue2 中的双向数据绑定技术

注意：这里的实现代码有非常多的漏洞，如：

- 未实现`模版渲染 Compiler`的解析，而是直接修改目标 dom 的内容
- 实现中只绑定了一个 `name` 属性
  - 所以从头到尾只存在一个 `Watcher`，所以整个过程 `Dispatcher.target` 并未更动也未做区分
  - 绑定到实例的 `$prop` 直接设为 `name`，当存在多个绑定属性的时候就必须加以区分

项目的构成参考了前一篇<a href="https://blog.csdn.net/weixin_44691608/article/details/111072904">Express 实战: 使用 express-static 处理静态资源</a>的方式创建的项目

### 项目结构 & 静态内容初始化

首先给出项目结构：

```
/data-binding-defineProperty
.
├── package.json
├── server.js       // 静态资源服务器入口
├── src/
│   ├── compiler.js     // Compiler（模版渲染）
│   ├── dispatcher.js   // Dispatcher（调度中心）
│   ├── favicon32.ico
│   ├── index.css       // 模版样式
│   ├── index.html      // 模版代码
│   ├── main.js         // 主入口
│   ├── observe.js      // 观察方法（observe、reactive）
│   ├── vue.js          // Vue (MVVM)
│   └── watcher.js      // Watcher（观察者）
└── yarn.lock
```

接下来给出几个静态文件的初始化内容

> package.json：加入 `start` 作为启动命令

```json
{
  "name": "js_data_binding_vue2",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.17.1",
    "express-static": "^1.2.6"
  }
}
```

> server.js

```js
const express = require('express')
const expressStatic = require('express-static')

const app = express()

app.use(expressStatic('./src'))

const port = 3000

app.listen(port, () => {
  console.log(`server listen at: http://localhost:${port}`)
})
```

> index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>data-binding Vue2</title>
    <link rel="icon" href="favicon32.ico">
    <link rel="stylesheet" href="index.css">
</head>
<body>
    <div id="app">
        <input type="text" class="input" v-model="name">
        <h1 class="content">{{ name }}</h1>
    </div>

    <script src="main.js" type="module"></script>
</body>
</html>
```

> index.css

```css
#app {
    width: 500px;
    margin: 100px auto;
    text-align: center;
}
```

### 核心代码

接下来给出具体的核心代码实现，相关说明请看代码注释：

> main.js：主入口

```js
import Vue from './vue.js'

// 创建 MVVM 类，并暴露成全局变量 vm 供访问
window.vm = new Vue({
  el: '#app',             // 模版选择器，即替换目标
  data: {                 // 数据项
    name: 'default name'
  }  
})
```

> vue.js：MVVM 类实现

```js
import observe from './observe.js'
import Compiler from './compiler.js'

// MVVM 主类
export default function Vue (options) {
  // 初始化各属性
  this.$options = options // 传入参数备份
  this.$el = document.querySelector(options.el) // 选到实际 dom 元素
  this.$data = options.data // 数据项备份
  Object.keys(this.$data).forEach(key => {
    this.$prop = key // 目前只有一个绑定属性 name
  })
  this.init() // 初始化
}

Vue.prototype.init = function () {
  // 初始化时递归观察 this.$data 数据项
  observe(this.$data)
  // 创建模版渲染对象，于自身绑定（传入 vm）
  new Compiler(this)
}
```

> observe.js：观察函数实现

```js
import Dispatcher from './dispatcher.js'

// 观察（绑定）数据项
export default function observe (data) {
  // 只对 object 作用
  if (!data || typeof data !== 'object') {
    return
  }
  // 激活 reactive 对象的每个键
  Object.keys(data).forEach(key => {
    reactive(data, key, data[key])
  })
}

// 激活函数
function reactive (data, key, value) {
  // 对每个键创建独有的调度中心 Dispatcher
  const dp = new Dispatcher()
  // 使用 Object.defineProperty 设置成访问器属性（getter/setter）
  Object.defineProperty(data, key, {
    get () {
      // 访问属性时检查当前访问者是否已经订阅该属性
      if (Dispatcher.target && !dp.subs.includes(Dispatcher.target)) {
        dp.addSub(Dispatcher.target)
      }
      return value
    },
    set (newValue) {
      if (value !== newValue) {
        // 实际的值透过闭包绑定到局部变量 value 上
        value = newValue
        // 每次更新就透过 Dispatcher 更新（notify 将通知所有 subs）
        dp.notify()
      }
    }
  })
  // 递归观察
  observe(value)
}
```

> dispatcher.js：调度中心实现

```js
export default function Dispatcher () {
  // 订阅者列表，是一个 Watcher 列表
  this.subs = []
}

Dispatcher.target = null

Dispatcher.prototype.notify = function () {
  // 通知更新时提醒所有观察者更新（调用 sub.update()）
  this.subs.forEach(sub => {
    sub.update()
  })
}

Dispatcher.prototype.addSub = function (sub) {
  // 添加订阅
  this.subs.push(sub)
}
```

> watcher.js：观察者实现

```js
import Dispatcher from './dispatcher.js'

// 观察者（订阅者）
export default function Watcher (vm, prop, callback) {
  this.vm = vm
  this.$prop = prop
  this.value = this.get()
  this.callback = callback
}

Watcher.prototype.get = function () {
  Dispatcher.target = this
  const value = this.vm.$data[this.$prop]
  return value
}

Watcher.prototype.update = function () {
  const value = this.vm.$data[this.$prop]
  const oldValue = this.value
  // 观察者更新时检查当前保留数据（this.value 将与实际 dom 展示数据同步）
  // 与 新数据（data.name 为实际绑定数据）
  if (oldValue !== value) {
    // 不相同时则更新 this.value 并通知 Compiler 更新 dom（callback 为 Compiler 传入的更新 dom 函数）
    this.value = value
    this.callback(this.value)
  }
}
```

> compiler.js：模版渲染实现（非常简陋的实现）

```js
import Watcher from './watcher.js'

// 模版渲染器
export default function Compiler (vm) {
  this.vm = vm
  this.$el = vm.$el
  this.fragment = null
  this.init()
}

Compiler.prototype.init = function () {
  // 初始化时使用 data 的值替换 dom 展示的内容
  // 这边没有实现模版解析，而是直接指定 data.name 并替换标签内容（textContent）
  let value = this.vm.$data.name
  document.querySelector('.input').value = value
  document.querySelector('.content').textContent = value

  // 为观察属性（prop）创建相应的观察者，并传入能够更新模版内容的回调函数（callback）
  // 这边只有一个 data.name 属性，并且回调函数直接修改指定标签内容
  // 正常实现是需要遇上方模版解析语法配合，在虚拟 dom 上修改相应标签
  new Watcher(this.vm, this.vm.$prop, value => {
    document.querySelector('.input').value = value
    document.querySelector('.content').textContent = value
  })

  // 为输入框添加监听函数
  document.querySelector('.input').addEventListener('input', e => {
    const targetValue = e.target.value
    if (value !== targetValue) {
      // 输入框的值修改时直接修改绑定变量的值
      this.vm.$data.name = targetValue
      // 并直接更新模版内容
      document.querySelector('.input').value = targetValue
      document.querySelector('.content').textContent = targetValue
    }
  }, false) // 默认 false 为冒泡事件
}
```

### 对象间交互顺序 & 图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/data_binding_object_interaction.png)

所谓双向绑定就是不管我们是(1)修改绑定变量(2)输入框输入哪种操作另一种都会同步，所以我们分别从两个路线来看对象间是如何交互的：

#### input 输入

当我们在输入框进行输入时会产生 `input` 事件，并进入到 `compiler.js` 的

```js
// 为输入框添加监听函数
document.querySelector('.input').addEventListener('input', e => {
  const targetValue = e.target.value
  if (value !== targetValue) {
    // 输入框的值修改时直接修改绑定变量的值
    this.vm.$data.name = targetValue
    // 并直接更新模版内容
    document.querySelector('.input').value = targetValue
    document.querySelector('.content').textContent = targetValue
  }
}, false) // 默认 false 为冒泡事件
```

这段处理函数，他会直接修改 `$data.name` 的值并更新模版，而修改 `$data.name` 则会触发 `observe.js` 中定义的访问器属性（`set` 方法）

```js
// 使用 Object.defineProperty 设置成访问器属性（getter/setter）
Object.defineProperty(data, key, {
  get () {
    // 访问属性时检查当前访问者是否已经订阅该属性
    if (Dispatcher.target && !dp.subs.includes(Dispatcher.target)) {
      dp.addSub(Dispatcher.target)
    }
    return value
  },
  set (newValue) {
    if (value !== newValue) {
      // 实际的值透过闭包绑定到局部变量 value 上
      value = newValue
      // 每次更新就透过 Dispatcher 更新（notify 将通知所有 subs）
      dp.notify()
    }
  }
})
```

这时候就会更新闭包中的 `value` 并通知相应的 `Dispatcher` 进行更新 `notify`，`notify` 函数则会通知所有观察者进行更新（`watcher.js`）

```js
Watcher.prototype.update = function () {
  const value = this.vm.$data[this.$prop]
  const oldValue = this.value
  // 观察者更新时检查当前保留数据（this.value 将与实际 dom 展示数据同步）
  // 与 新数据（data.name 为实际绑定数据）
  if (oldValue !== value) {
    // 不相同时则更新 this.value 并通知 Compiler 更新 dom（callback 为 Compiler 传入的更新 dom 函数）
    this.value = value
    this.callback(this.value)
  }
}
```

观察者 `Watcher` 就会根据 `Compiler` 提供的回调函数更新模版内容

#### 修改 `$data` 属性

第二种可能是直接修改 `$data.name` 的值，就会经过 `set` -> `dp.notify()` -> `sub.update()` -> `callback()` 的路径更新绑定标签的值啦。

# 结语

本篇模仿 Vue2 使用 `Object.defineProperty` 实现双向数据绑定，不过最重要的是缺少了模版解析和渲染的部分（这很重要，应该只修改 `{{}}` 所覆盖的范围而不是修改整个标签内容，同时应该为每个 `{{}}` 引用创建相应的观察者 `Watcher` 才是正确的实现），供大家参考。
