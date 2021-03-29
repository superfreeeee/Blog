# JS 动画基础: 细说 requestAnimationFrame

@[TOC](文章目录)

<!-- TOC -->

- [JS 动画基础: 细说 requestAnimationFrame](#js-动画基础-细说-requestanimationframe)
  - [简介](#简介)
  - [参考](#参考)
  - [完整示例代码](#完整示例代码)
- [正文](#正文)
  - [`setInterval` 实现动画](#setinterval-实现动画)
    - [`setInterval` 的限制](#setinterval-的限制)
  - [`requestAnimationFrame` 实现动画](#requestanimationframe-实现动画)
    - [rAF 基础实现](#raf-基础实现)
    - [rAF 定时调用：timestamp](#raf-定时调用timestamp)
    - [rAF 取消回调：`cancelAnimationFrame`](#raf-取消回调cancelanimationframe)
    - [渲染、动画分离](#渲染动画分离)
    - [最终封装版本](#最终封装版本)
  - [`requestAnimationFrame` 特性 & 使用注意事项](#requestanimationframe-特性--使用注意事项)
- [结语](#结语)

<!-- /TOC -->

## 简介

相信大家都知道动画的基础就是在于快速的画面刷新，常见的刷新频率单位是 **每秒传输帧数(FPS = Frames Per Second)** ，有时我们也直接简称为 **帧** 。在游戏或是视频中，24 帧以上对于肉眼来说就能算是流畅不卡顿，而常见的刷新频率通常为 60 FPS(或是 75 FPS)。

同样的，我们透过浏览器访问网页也需要进行页面刷新。对于浏览器来说所谓的网页也就是一个用 html 格式所描述的一个动态画面，DOM 元素的移动、改变、滚轮等就相当于是在页面的一次刷新中对元素的偏移、变形进行重新计算，最后 **重新渲染** 在浏览器的页面中，而这就是所谓的页面刷新。

下面我们将从熟悉的 `setTimout` 起手，到引入本篇的主角 `requestAnimationFrame`(下文将交叉使用 rAF 用以表示该接口)，以及对于 rAF 的使用进行说明并给出范例。

## 参考

<table>
  <tr>
    <td>requestAnimationFrame</td>
    <td><a href="https://blog.csdn.net/qq_37606901/article/details/79774101?utm_source=app&app_version=4.5.5">https://blog.csdn.net/qq_37606901/article/details/79774101?utm_source=app&app_version=4.5.5</a></td>
  </tr>
  <tr>
    <td>requestAnimationFrame</td>
    <td><a href="https://www.jianshu.com/p/f6d933670617?ivk_sa=1024320u">https://www.jianshu.com/p/f6d933670617?ivk_sa=1024320u</a></td>
  </tr>
  <tr>
    <td>requestAnimationFrame详解</td>
    <td><a href="https://www.jianshu.com/p/fa5512dfb4f5?ivk_sa=1024320u">https://www.jianshu.com/p/fa5512dfb4f5?ivk_sa=1024320u</a></td>
  </tr>
  <tr>
    <td>window.requestAnimationFrame-MDN</td>
    <td><a href="https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestAnimationFrame">https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestAnimationFrame</a></td>
  </tr>
  <tr>
    <td>DOMHighResTimeStamp-MDN</td>
    <td><a href="https://developer.mozilla.org/zh-CN/docs/Web/API/DOMHighResTimeStamp">https://developer.mozilla.org/zh-CN/docs/Web/API/DOMHighResTimeStamp</a></td>
  </tr>
</table>

## 完整示例代码

<a href="https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_requestAnimationFrame">https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_requestAnimationFrame</a>

# 正文

## `setInterval` 实现动画

要说到定时刷新，刚接触 web 前端的人大多会联想到 `setInterval` 这个 API。"不就是定时吗？那就使用 `setInterval` 设定一个定时任务来定期刷新页面，实现动画"。是的这个思路是可以的，给出代码

```js
// sample_setInterval.js
const start = () => {
  const text = document.querySelector('.text')
  const maxWidth = /[0-9]*/.exec(getComputedStyle(text).width)

  let w = 0
  const renderId = setInterval(() => {
    for (let i = 0; i < 10000; i++) {
      for (let j = 0; j < 10000; j++) {
        const k = i * j
      }
    }
    if (w < maxWidth) {
      console.log(`do at ${performance.now()}`)
      w++
      text.style.width = `${w}px`
    } else {
      console.log('animation finished')
      clearInterval(renderId)
    }
  }, 16)
}

document.querySelector('.btn').addEventListener('click', start)
```

- 实现效果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_requestAnimationFrame_sample1.gif)

这边我们要实现的动画就是文字框的宽度从 $0px$ 伸长到 $300px$

### `setInterval` 的限制

上面的代码看起来没啥问题，实现效果也是正常。但是我们忽略了 `setInterval` 的特性。它本质上是一个'定时'任务，然而这个定时却不是指'每 xxx 毫秒执行一次'，而是'每xxx毫秒之后执行一次'。由于 js 的 **事件循环(Event Loop)** 机制特性，`setInterval` 的定时任务并不能保证在固定时间后立即执行，而仅仅只是在计时结束后重新将任务放回异步队列等待下一次的执行。

这时候如果主线程代码或是动画函数过于繁忙/密集可能会产生 **掉帧** 的现象，如下代码

```js
// sample_setInterval.js
const start = () => {
  const text = document.querySelector('.text')
  const maxWidth = /[0-9]*/.exec(getComputedStyle(text).width)

  let w = 0
  const renderId = setInterval(() => {
    for (let i = 0; i < 10000; i++) {
      for (let j = 0; j < 10000; j++) {
        const k = i * j
      }
    }
    if (w < maxWidth) {
      console.log(`do at ${performance.now()}`)
      w++
      text.style.width = `${w}px`
    } else {
      console.log('animation finished')
      clearInterval(renderId)
    }
  }, 16)
}

document.querySelector('.btn').addEventListener('click', start)
```

我们在每次的渲染函数前面加上一个非常耗时(一亿次的循环)的操作来代表实际应用中的复杂逻辑和耗时操作

- 实现效果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_requestAnimationFrame_sample2.gif)

我们可以看到，明明同样设定间隔为 $16ms$，但是动画的速度大打折扣。我们截取最后的几个输出： 

```
...
do at 16795.655000023544
do at 16838.365000032354
do at 16877.054999989923
do at 16930.690000008326
do at 16969.79000000283
animation finished
```

发现每次间隔因为耗时操作的原因，实际上大约每 $40ms$ 才执行了一次动画动作，这时如果浏览器是 60 FPS(每 $16ms$ 为一帧)的频率，那实际上每 2~3 帧才进行了一次修改，也就是差生了所谓的 **掉帧** 现象

## `requestAnimationFrame` 实现动画

为了防止掉帧的现象产生，我们应该使用另一个更符合我们需求的 API: `requestAnimationFrame`。首先先来看看 MDN 上对于 rAF 的描述：

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_requestAnimationFrame_mdn.png)

简单来说，rAF 会在 **下次页面刷新之前调用回调函数**。接下来有关特性和使用的样式下面我们一一来讲解

### rAF 基础实现

由于我们的主题围绕着动画的实现，所以先给出一个基础般的 rAF 使用方式

```js
// sample_simple.js
const start = () => {
  const text = document.querySelector('.text')
  const maxWidth = /[0-9]*/.exec(getComputedStyle(text).width)

  let w = 0
  const render = () => {
    if (w < maxWidth) {
      console.log(`do at ${performance.now()}`)
      w += 5
      text.style.width = `${w}px`
      requestAnimationFrame(render)
    } else {
      console.log('animation finished')
    }
  }
  requestAnimationFrame(render)
}

document.querySelector('.btn').addEventListener('click', start)
```

代码的逻辑就是向 rAF 传入想要在下一帧刷新前调用的函数，并在回调函数里面递归调用 rAF 来实现动画效果

- 实现效果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_requestAnimationFrame_sample3.gif)

我们可以从图中看到默认的刷新率近似于每 $16ms$ 一次调用，也就是 60 FPS(测试时使用 chrome 浏览器)

### rAF 定时调用：timestamp

然而 rAF 并不是用就完事了，**默认** 情况下，rAF 的调用时机是与屏幕刷新率相关的，rAF 在 **每一帧刷新前调用回调函数**。如此一来回调函数调用的时机就取决于浏览器的刷新率和当时主线程的调度情况。为了避免浏览器刷新率的干扰，同时我们希望动画在不同环境下都能有近似的刷新速率，我们可以利用 rAF 传给回调函数的第一个参数来进行定时刷新。

rAF 调用回调函数的时候会传入一个 `DOMHighResTimeStamp` 类型的参数，简单来说它的意义就是调用该回调的时候的时间戳，其值是与 `performance.now()` 相同的，下面给出 **每 $50ms$ 调用渲染一次** 的实现

```js
// sample_time.js
const [period, offset] = [50, 5]

const start = (period, offset) => {
  const text = document.querySelector('.text')
  const maxWidth = /[0-9]*/.exec(getComputedStyle(text).width)

  let w = 0
  let lastTime = performance.now()
  console.log(`start time = ${lastTime}`)
  const render = (timestamp) => {
    if (w < maxWidth) {
      if (timestamp - lastTime > period) {
        console.log(`do at = ${timestamp}`)
        w += offset
        text.style.width = `${w}px`
        lastTime = timestamp
      }
      requestAnimationFrame(render)
    } else {
      console.log('animation finished')
    }
  }
  requestAnimationFrame(render)
}

document.querySelector('.btn').addEventListener('click', () => {
  start(50, 5)
})
```

- 实现效果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_requestAnimationFrame_sample4.gif)

我们可以看到每次执行动画回调的时间间隔被我们控制到 $50ms$ 了而不是默认的 $16ms$ 一次

### rAF 取消回调：`cancelAnimationFrame`

到此我们已经具备所有的动画实现基础了：
- 我们使用 rAF 的递归调用来实现连续动画
- 使用传入回调函数的 timestamp 来控制动画速率和调用时机

然而有的时候我们可能发生了一些意外状况需要取消当前的回调调用，这时候我们就可以使用 `cancelAnimationFrame` API。与 `setTimeout/clearTimeout` 的组合雷同，调用 `requestAnimationFrame` 会返回一个 rAF 的 $id$，透过调用 `cancelAnimationFrame(id)` 就能够取消预定好的回调函数。

下面我们延续上面使用的动画，在另外设置一个定时器在一秒后取消动画回调

```js
// sample_clean.js
const [period, offset] = [50, 5]

const start = (period, offset) => {
  const text = document.querySelector('.text')
  const maxWidth = /[0-9]*/.exec(getComputedStyle(text).width)

  let w = 0
  let lastTime = performance.now()
  console.log(`start time = ${lastTime}`)
  let id = null
  const render = (timestamp) => {
    // console.log(timestamp)
    if (w < maxWidth) {
      if (timestamp - lastTime > period) {
        console.log(`do at = ${timestamp}`)
        w += offset
        text.style.width = `${w}px`
        lastTime = timestamp
      }
      id = requestAnimationFrame(render)
    } else {
      console.log('animation finished')
    }
  }
  id = requestAnimationFrame(render)
  return () => {
    console.log(`cancel at ${performance.now()}`)
    cancelAnimationFrame(id)
  }
}

document.querySelector('.btn').addEventListener('click', () => {
  const cancel = start(50, 5)
  setTimeout(cancel, 1000)
})
```

- 实现效果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_requestAnimationFrame_sample5.gif)

我们可以看到在 1 秒钟后取消了动画，所以页面就停在当下不再更新

### 渲染、动画分离

到此为止其实就是所有可能会用到的 API 了，下面我们针对实践方案还有实际开发场景进行指责的分离和封装

首先第一步通常情况下我们应该要将动画回调函数与真正的计时器(递归 rAF 调用)进行分离，相当于是将 **页面刷新时机** 和 **真正的刷新操作** 进行隔离，方便我们对两个部分分别进行扩展或修改

```js
// sample_separate_render.js
const setRender = (offset) => {
  const text = document.querySelector('.text')
  const maxWidth = /[0-9]*/.exec(getComputedStyle(text).width)

  let w = 0
  const render = () => {
    // console.log(timestamp)
    const hasNext = w < maxWidth
    if (hasNext) {
      w += offset
      text.style.width = `${w}px`
    }
    return hasNext
  }
  return render
}

const startAnimation = (render, period = null) => {
  console.log('start animation')
  let hasNext = true
  let renderWrapper
  if (period) {
    let lastTime = performance.now()
    console.log(`start time = ${lastTime}`)
    renderWrapper = (timestamp) => {
      if (timestamp - lastTime > period) {
        console.log(`do at = ${timestamp}`)
        hasNext = render()
        lastTime = timestamp
      }
      if (hasNext) {
        requestAnimationFrame(renderWrapper)
      } else {
        console.log('animation finished')
      }
    }
  } else {
    renderWrapper = (timestamp) => {
      console.log(`do at = ${timestamp}`)
      hasNext = render()
      if (hasNext) {
        requestAnimationFrame(renderWrapper)
      } else {
        console.log('animation finished')
      }
    }
  }
  requestAnimationFrame(renderWrapper)
}

document.querySelector('.btn').addEventListener('click', () => {
  startAnimation(setRender(5), 50)
})
```

这边我们将动画的回调封装到 `setRender` 方法当中，而动画回调时机的控制则封装到 `startAnimation` 方法之中(分成 **指定间隔/默认刷新率** 两种模式)

由于实现效果与上面相同，这边就不再截图，有兴趣可以到代码仓库将目标文件拉下来运行

### 最终封装版本

最终版本我们将回调函数执行时机的控制封装到一个 **动画控制器(AnimationController)** 之中

```js
// index.js
class AnimationController {
  constructor(render, reset, period = null) {
    this.render = render
    this.period = period
    this.reset = reset
  }

  // 设置动画回调
  setRender(render = () => {}) {
    this.render = render
  }

  // 设置间隔
  setPeriod(period = null) {
    this.period = period
  }

  // 开始动画
  start() {
    const { render, period } = this
    this.stop()
    this.reset()
    this.hasNext = true

    const next = (hasNext) => {
      if (hasNext) {
        this.animationId = requestAnimationFrame(this.renderWrapper)
      } else {
        console.log('animation finished')
        this.animationId = null
      }
    }

    if (period) {
      let lastTime = performance.now() - period
      this.renderWrapper = (timestamp) => {
        if (timestamp - lastTime > period) {
          console.log(`do at = ${timestamp}`)
          this.hasNext = render()
          lastTime = timestamp
        }
        next(this.hasNext)
      }
    } else {
      this.renderWrapper = (timestamp) => {
        console.log(`do at = ${timestamp}`)
        this.hasNext = render()
        next(this.hasNext)
      }
    }
    this.animationId = requestAnimationFrame(this.renderWrapper)
  }

  // 停止动画
  stop() {
    if (this.animationId) {
      cancelAnimationFrame(animationId)
      this.animationId = null
    }
  }
}

const createRender = (offset) => {
  const text = document.querySelector('.text')
  const maxWidth = /[0-9]*/.exec(getComputedStyle(text).width)

  let w = 0
  const render = () => {
    const hasNext = w < maxWidth
    if (hasNext) {
      w += offset
      text.style.width = `${w}px`
    }
    return hasNext
  }
  const reset = () => {
    w = 0
  }
  return [render, reset]
}

const controller = new AnimationController(...createRender(10), 50)

window.controller = controller
window.start = controller.start.bind(controller)
window.stop = controller.stop.bind(controller)
window.reset = controller.reset

document.querySelector('.btn').addEventListener('click', start)
```

## `requestAnimationFrame` 特性 & 使用注意事项

最后给出几个对于 rAF 的总结

- 特性

1. rAF 会把每一帧中的所有 DOM 操作集中起来，在一次重绘或回流中就完成；重绘或回流的时机则是根据屏幕的刷新率来决定
2. 对于隐藏元素、后台脚本(其他分页)，rAF 将不会进行重绘/回流，大大节省了 cpu/gpu 的性能开销和内存使用

- 应用

1. 函数节流：由于 rAF 的执行时机依赖于屏幕刷新率，并将同一帧内的操作合并，也就相当于浏览器提供了一个默认的函数节流的借力点

# 结语

最近春招面试告一段路，在面试中发现了许多不足的部分，同时也累积了许多新的知识点需要去探索，之后会重新恢复更新博客。同时由于一些考量和自我反省，之后的博客会希望往更精更深的知识点或实践场景去探索、以提升博客内容质量，并以分享总结好的知识点为出发点继续之后的更新。
