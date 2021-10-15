# JS 进阶: 深入理解键盘事件 Keyboard Event

@[TOC](文章目录)

<!-- TOC -->

- [JS 进阶: 深入理解键盘事件 Keyboard Event](#js-进阶-深入理解键盘事件-keyboard-event)
- [正文](#正文)
  - [1. 基础 API](#1-基础-api)
  - [2. 基础事件：keydown、keyup](#2-基础事件keydownkeyup)
  - [3. 应用：一次按下 + 释放仅触发一次事件](#3-应用一次按下--释放仅触发一次事件)
  - [4. 应用：组合键事件监听封装](#4-应用组合键事件监听封装)
  - [5. 应用：计算按压时间](#5-应用计算按压时间)
  - [6. 应用：定制定时器超越事件触发间隔限制](#6-应用定制定时器超越事件触发间隔限制)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

## 1. 基础 API

第一部分是基础 API

| 事件名  | 描述     | 事件类型      |
| ------- | -------- | ------------- |
| keydown | 按下按键 | KeyboardEvent |
| keyup   | 释放按键 | KeyboardEvent |

- KeyboardEvent 标准属性

| 属性        | 描述                             | 类型/示例值 |
| ----------- | -------------------------------- | ----------- |
| key         | 按键字面量                       |             |
| code        | 按键编号                         |             |
| altKey      | `alt/option` 是否按下            |             |
| ctrlKey     | `ctrl` 是否按下                  |             |
| shiftKey    | `shift` 是否按下                 |             |
| metaKey     | `meta/command` 是否按下          |             |
| repeat      | 是否为按压下重复触发事件         |             |
| isComposing | 是否在组合输入过程（如拼音输入） |             |

使用的示例如下

```js
const onKeydownBasic = (e) => {
  console.group('keyboard event');
  console.log('event', e);
  console.log('event.code', e.code);
  console.log('event.key', e.key);
  console.log('event.altKey', e.altKey);
  console.log('event.ctrlKey', e.ctrlKey);
  console.log('event.shiftKey', e.shiftKey);
  console.log('event.metaKey', e.metaKey);
  console.log('event.getModifierState()', e.getModifierState('Alt'));
  console.groupEnd();
};

document.addEventListener('keydown', onKeydownBasic);
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_keyboard_event_1_basic.png)

## 2. 基础事件：keydown、keyup

第二个我们来测试两个事件的不同

```js
const onKeydownUp = (e, isDown) => {
  console.log(`key: ${e.key} ${isDown ? 'down' : 'up'}`);
};

const testUpAndDown = () => {
  document.addEventListener('keydown', (e) => onKeydownUp(e, true));
  document.addEventListener('keyup', (e) => onKeydownUp(e, false));
};

testUpAndDown();
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_keyboard_event_2_up_and_down.png)

同时我们发现如果按住不放则会不断出发 keydown 事件，直到释放按键时触发一次 keyup 为止

## 3. 应用：一次按下 + 释放仅触发一次事件

第一个应用是透过 repeat 属性，让我们只在按下第一次的时候触发事件

```js
const onKeydownOnce = (e) => {
  !e.repeat && console.log(`press ${e.key}`);
};

document.addEventListener('keydown', onKeydownOnce);
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_keyboard_event_3_once.png)

## 4. 应用：组合键事件监听封装

第二个应用则是对组合键进行封装，允许用户注册带组合键的监听函数

```js
class KeyboardHandler {
  static ALT = 'A';
  static SHIFT = 'S';
  static CTRL = 'C';
  static META = 'M';

  handlerConfigs = new Map();

  constructor() {
    this.listen();
  }

  addHandler(key, opts, handler) {
    const id = this.createId(key, opts);
    let handlers = this.handlerConfigs.get(id);
    if (!handlers) {
      handlers = new Set();
      this.handlerConfigs.set(id, handlers);
    }
    if (!handlers.has(handler)) {
      handlers.add(handler);
    }
  }

  removeHandler(key, opts, handler) {
    const id = this.createId(key, opts);
    const handlers = this.handlerConfigs.get(id) || new Set();
    if (handlers.has(handler)) {
      handlers.delete(handler);
    }
    if (handlers.size === 0) {
      this.handlerConfigs.delete(id);
    }
  }

  createId(key, opts) {
    const supportKeys = [KeyboardHandler.ALT, KeyboardHandler.SHIFT, KeyboardHandler.CTRL, KeyboardHandler.META];
    const config = {
      [KeyboardHandler.ALT]: false,
      [KeyboardHandler.SHIFT]: false,
      [KeyboardHandler.CTRL]: false,
      [KeyboardHandler.META]: false,
      ...opts,
    };
    let id = `${key.toLowerCase()}`;
    for (const key of supportKeys) {
      id += `-${key}${config[key] ? 1 : 0}`;
    }
    return id;
  }

  listen() {
    document.addEventListener('keydown', this.onKeydown);
    document.addEventListener('keyup', this.onKeyup);
  }

  stopListen() {
    document.removeEventListener('keydown', this.onKeydown);
    document.removeEventListener('keyup', this.onKeyup);
  }

  onKeydown = (e) => {
    const opts = {
      [KeyboardHandler.ALT]: e.altKey,
      [KeyboardHandler.SHIFT]: e.shiftKey,
      [KeyboardHandler.CTRL]: e.ctrlKey,
      [KeyboardHandler.META]: e.metaKey,
    };
    const id = this.createId(e.key, opts);
    const handlers = this.handlerConfigs.get(id);
    handlers && handlers.forEach((handler) => handler());
  };

  onKeyup = (e) => {};
}

const testCombineHandler = () => {
  const keyboardHandler = new KeyboardHandler();
  console.log(keyboardHandler);

  keyboardHandler.addHandler('a', {}, () => {
    console.log('press a');
  });
  keyboardHandler.addHandler('a', { [KeyboardHandler.SHIFT]: true }, () => {
    console.log('press a + shift');
  });
};

testCombineHandler();
```

代码还算是比较简单的可以自己看看，这里简单带过：
- `addHandler`、`removeHandler` 是处理监听函数的
- `createId` 透过对 key 值和辅助键生成唯一的 id 作为导出事件的 key
- `onKeydown` 在按下按键时依据键盘状态组合成 id，查找相应的监听函数并触发

生成的 ID 如下图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_keyboard_event_4_combine_keydown.png)

## 5. 应用：计算按压时间

下一个应用，我们可以利用先 keydown 后 keyup 的特性来计算按键被持续按压的时间

```js
const pressedKeys = new Map();
const onKeydownTime = (e) => {
  if (!pressedKeys.has(e.key)) {
    pressedKeys.set(e.key, performance.now());
  }
};

const onKeyupTime = (e) => {
  const endTime = performance.now();
  const startTime = pressedKeys.get(e.key) || endTime;
  pressedKeys.delete(e.key);

  const pressTime = Math.round(endTime - startTime);

  console.log(`key=${e.key}, press time=${pressTime}`);
};

const testCountingTime = () => {
  document.addEventListener('keydown', onKeydownTime);
  document.addEventListener('keyup', onKeyupTime);
};

testCountingTime();
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_keyboard_event_5_count_time.png)

## 6. 应用：定制定时器超越事件触发间隔限制

最后一个应用比较特别，我们前面提过持续按压按键会重复触发 keydown 事件，但是有时侯依赖事件是危险的，因为键盘的连续 keydown 事件有点太慢了，如下测试

```js
let id = 0;
let time = 0;
const onKeydownDelay = (e) => {
  if (!time) {
    time = performance.now();
  }
  const current = performance.now();
  const delay = Math.round(current - time);
  console.log(`keydown(${id++}): ${e.key}, delay=${delay}`);
  time = current;
};

document.addEventListener('keydown', onKeydownDelay);
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_keyboard_event_6_event_delay.png)

我们发现第一次事件触发后会延迟 500ms 才触发第二次，接下来大约每 80ms 触发一次键盘事件，这对于需要频繁响应用户操作的应用（如游戏）来说有些太慢了，因此我们可以自己定制一个计时器，来超越事件触发的频率

```js
class KeyboardEventScheduler {
  delay;
  timer = null;

  keysConfig = new Map();
  activeKeys = new Set();

  constructor(delay = 1000) {
    this.delay = delay;
  }

  setHandler(key, handler) {
    let handlers = this.keysConfig.get(key);
    if (!handlers) {
      handlers = [];
      this.keysConfig.set(key, handlers);
    }
    handlers.push(handler);
  }

  clearHandler(key, handler) {
    const handlers = this.keysConfig.get(key);
    if (handlers.includes(handler)) {
      handlers.splice(handlers.indexOf(handler), 1);
    }
    if (handlers.length === 0) {
      this.keysConfig.delete(key);
    }
  }

  clearTimer() {
    console.log('clearTimer');
    clearInterval(this.timer);
    this.timer = null;
  }

  setTimer() {
    console.log('setTimer');
    const callHandlers = () => {
      this.activeKeys.forEach((key) => {
        this.keysConfig.get(key).forEach((handler) => handler());
      });
    };
    this.timer = setInterval(callHandlers, this.delay);
    callHandlers();
  }

  pressKey(key) {
    if (!this.activeKeys.has(key) && this.keysConfig.has(key)) {
      this.activeKeys.add(key);
      if (!this.timer) {
        this.setTimer();
      }
    }
  }

  releaseKey(key) {
    if (this.activeKeys.has(key)) {
      this.activeKeys.delete(key);
      if (this.activeKeys.size === 0) {
        this.clearTimer();
      }
    }
  }
}
```

核心流程解释：
- `setHandler`、`clearHandler` 负责维护监听函数
- `clearTimer`、`setTimer` 负责维护定制的计时器
- `pressKey`、`releaseKey` 则负责控制按键的标记

我们就可以通过自定义的计时器，轮训按键标志后不断自动触发事件，而键盘事件仅仅是起到一个控制标记的作用，如此一来我们便能够超越事件触发的频率（这里使用 `setTimeInterval`，在 Chrome 浏览器测试下极限为每 5～8ms 触发一次左右）

接下来我们来比较依赖原生事件触发频率与使用定制计时器的区别

- 依赖原生事件频率

```js
const testSchedulerOrigin = () => {
  const box = document.querySelector('.board .box');
  let boxLeft = Number.parseInt(getComputedStyle(box).left);

  document.addEventListener('keydown', (e) => {
    console.log(`press at ${Math.round(performance.now())}`);
    if (e.key === 'ArrowRight') {
      boxLeft = Math.min(300 - 30, boxLeft + 30);
      box.style.left = `${boxLeft}px`;
    } else if (e.key === 'ArrowLeft') {
      boxLeft = Math.max(0, boxLeft - 30);
      box.style.left = `${boxLeft}px`;
    }
  });
};
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_keyboard_event_7_schedule_origin.gif)

与前面的按压测试相同，第一次 500ms 的间隔，后面约 80ms 的间隔

- 定制计时器

```js
const testScheduler = () => {
  const scheduler = new KeyboardEventScheduler(30);
  const box = document.querySelector('.board .box');
  let boxLeft = Number.parseInt(getComputedStyle(box).left);
  scheduler.setHandler('ArrowRight', () => {
    console.log(`press at ${Math.round(performance.now())}`);
    boxLeft = Math.min(300 - 30, boxLeft + 30);
    box.style.left = `${boxLeft}px`;
  });
  scheduler.setHandler('ArrowLeft', () => {
    console.log(`press at ${Math.round(performance.now())}`);
    boxLeft = Math.max(0, boxLeft - 30);
    box.style.left = `${boxLeft}px`;
  });

  document.addEventListener('keydown', (e) => {
    !e.repeat && scheduler.pressKey(e.key);
  });

  document.addEventListener('keyup', (e) => {
    scheduler.releaseKey(e.key);
  });
};
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_keyboard_event_8_schedule.gif)

定制计时器我们设定为 30ms 的间隔，实际上最高可以达到约 5～8ms 的间隔，同时物体的移动也更为流畅

# 其他资源

## 参考连接

| Title                    | Link                                                                                                                             |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------- |
| KeyboardEvent - MDN      | [https://developer.mozilla.org/zh-CN/docs/Web/API/KeyboardEvent](https://developer.mozilla.org/zh-CN/docs/Web/API/KeyboardEvent) |
| javascript KeyboardEvent | [https://blog.csdn.net/claroja/article/details/103987800](https://blog.csdn.net/claroja/article/details/103987800)               |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_keyboard_event](https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_keyboard_event)
