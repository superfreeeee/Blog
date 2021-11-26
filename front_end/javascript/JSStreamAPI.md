# JS Web API: Stream API 解析

@[TOC](文章目录)

<!-- TOC -->

- [JS Web API: Stream API 解析](#js-web-api-stream-api-解析)
- [正文](#正文)
  - [1. Stream 概念 & 类型](#1-stream-概念--类型)
  - [2. ReadableStream](#2-readablestream)
  - [3. WritableStream](#3-writablestream)
  - [4. pipeTo 传递数据](#4-pipeto-传递数据)
    - [4.1 自定义流对接](#41-自定义流对接)
    - [4.2 pipeTo](#42-pipeto)
  - [5. TransformStream & pipeThrough](#5-transformstream--pipethrough)
    - [5.1 TransformStream 的创建](#51-transformstream-的创建)
    - [5.2 自定义流通道](#52-自定义流通道)
    - [5.3 TransformStream + pipeThrough](#53-transformstream--pipethrough)
  - [6. 应用](#6-应用)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

## 1. Stream 概念 & 类型

写过一些其他语言的人应该对流不陌生，流本身可以说是对缓冲的一种抽象，将发送和接收的角色抽象出来，并透过对流的详细设计来控制数据流的效率

在 JS 当中 Stream API 大致可以分为以下三种流

1. ReadableStream 可读流
2. WritableStream 可写流
3. TransformStream 转换流

以下是我们参考的 MDN 教程和一些基础例子

传送门：[Streams API - MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Streams_API)

下面我们一个个来看分别都是怎么用的

## 2. ReadableStream

第一种 ReadableStream 依名字来能知道对于消费者来说希望从这个流中获取数据；而从定义的角度来说就是从某个地方接收到数据之后写入这个可读流当中

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_stream_api_1_readableStream.png)

接下来我们就先来看看可读流的创建

- `index.js`

```js
const readableStream = new ReadableStream({
  start(controller) {
    controller.enqueue('H');
    controller.enqueue('e');
    controller.enqueue('l');
    controller.enqueue('l');
    controller.enqueue('o');
    controller.enqueue(' ');
    controller.enqueue('W');
    controller.enqueue('o');
    controller.enqueue('r');
    controller.enqueue('l');
    controller.enqueue('d');
    controller.close();
  },
});
```

`ReadableStream` 构造函数接受一个配置对象，允许我们重写多个方法，最主要我们常用的就是这个 `start(controller)` 方法，然后我们可以使用 `controller.enqueue` 来向可读流写入数据，这个 start 内部是同步异步都没关系，最终都是以 `close` 表示没有新的数据了

而第一种消费可读流的方法就是直接创建一个 Reader

```js
const reader = readableStream.getReader();
let result;
result = await reader.read();
console.log(`[testReadableStreamCreate] result =`, result);
result = await reader.read();
console.log(`[testReadableStreamCreate] result =`, result);
```

- 输出

```
testReadableStreamCreate
  [testReadableStreamCreate] result = {value: 'H', done: false}
  [testReadableStreamCreate] result = {value: 'e', done: false}
```

这里我们可以看到每个 read 对应了 controller.enqueue 写入的数据，也就是说我们必须不断的调用 read 方法直到 `done: true`

## 3. WritableStream

可写流作为 ReadableStream 的反面，相当于是一个数据的接受者，可以从其他的可读流中接受流数据

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_stream_api_2_writableStream.png)

这时候与 ReadableStream 一样，数据并不是一次过来的，而每一次接受到的片段则称为 chunk

WritableStream 的构造方法如下

- `index.js`

```js
const writableStream = new WritableStream({
  write: (chunk) =>
    new Promise((resolve, reject) => {
      console.log(`[${prefix}] chunk =`, chunk);
      resolve();
    }),
  close() {},
});
```

这里比较重要的是一个 write 方法，而这个方法接受一个 chunk 作为参数，返回一个 Promise，会在每次数据传入的时候被调用

与 ReadableStream 一样，我们也有主动向 WritableStream 写入数据的方法，就是先创建 Writer 然后调用 write 方法

```js
const writer = writableStream.getWriter();
await writer.write('Hello');
await writer.write(' ');
await writer.write('World');
```

- 输出

```
testWritableStreamCreate
  [testWritableStreamCreate] chunk = Hello
  [testWritableStreamCreate] chunk = 
  [testWritableStreamCreate] chunk = World
```

## 4. pipeTo 传递数据

介绍完两个基础流，接下来我们来看看常规的用法

从概念上来说我们拿到一大块数据，依次写入 ReadableStream 之后，再通过 pipe 的方式使数据'流'到 WritableStream，然后我们就可以在另一边来消费数据

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_stream_api_3_pipeTo.png)

### 4.1 自定义流对接

首先我们先来看看如果我们想要自己去对接两个流（$ReadableStream \to WritableStream$）要怎么做

- `index.js`

```js
const testCustomPipeTo = async () => {
  const reader = new ReadableStream(createReadableStreamSource()).getReader();
  const writer = new WritableStream(
    createWritableStreamSource('testCustomPipeTo')
  ).getWriter();

  const tryRead = async () => {
    const { done, value } = await reader.read();
    if (done) {
      return;
    }
    await writer.ready;
    writer.write(value);

    await tryRead();
  };
  await tryRead();
};
```

重点在这个 `tryRead` 方法上，调用 `reader.read` 方法获取数据后，写入到 writer 对象上

### 4.2 pipeTo

Stream API 还提供了另一个更方便的方式，让我们可以不需要自己来管理流数据的速率，就是使用 `pipeTo` 方法

```js
const testPipeTo = async () => {
  const readableStream = new ReadableStream(createReadableStreamSource());
  const writableStream = new WritableStream(
    createWritableStreamSource('testPipeTo')
  );
  await readableStream.pipeTo(writableStream);
};
```

- 输出

```
testPipeTo
  [testPipeTo] chunk = H
  [testPipeTo] chunk = e
  [testPipeTo] chunk = l
  [testPipeTo] chunk = l
  [testPipeTo] chunk = o
  [testPipeTo] chunk = 
  [testPipeTo] chunk = W
  [testPipeTo] chunk = o
  [testPipeTo] chunk = r
  [testPipeTo] chunk = l
  [testPipeTo] chunk = d
```

可以看到使用 `pipeTo` 方法的时候，浏览器会自动在数据准备好同时 writer 空闲的时候自动的进行数据传递

## 5. TransformStream & pipeThrough

除了两个基础的 Stream 之外，最后一种 TransformStream 可以说是一种中间流。

我们可以把流想象成一个管道分成很多节，ReadableStream 是注水口而 WritableStream 是出水口，最后的 TransformStream 就好像中间的水管，可以对数据流进行加工，然后传递下去，甚至我们可以进行过滤、阻断等多种实现

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_stream_api_4_transformStream_pipeThrough.png)

### 5.1 TransformStream 的创建

TransformStream 的构造方法我们关注的是 transform 方法

- `index.js`

```js
const decoder = new TextDecoder();
const transformStream = new TransformStream({
  transform: (chunk, controller) => {
    controller.enqueue(decoder.decode(chunk, { stream: true }));
  },
});
```

第一个参数相当于作为 WritableStream 的角色，接受一个 chunk 数据；而第二个参数则是作为 ReadableStream 将加工过的数据向下传递到下一个流当中

### 5.2 自定义流通道

再使用 API 之前，我们已经可以先来尝试看看如何在两个 Stream 之间插入一个 TransformStream 了

首先我们的 ReadableStream 不再简单的传递数据，而是使用 `TextEncoder` 对字符串编码创建 ArrayBuffer 对象

```js
const testTransformStreamCustom = async () => {
  const readableStream = new ReadableStream({
    start(controller) {
      const encoder = new TextEncoder();
      const chunks = encoder.encode('Hello World', { stream: true });
      chunks.forEach((chunk) => controller.enqueue(chunk));
      controller.close();
    },
  });
```

这时候接受数据的 WritableStream 就需要添加一个 `TextDecoder` 对字符串进行解码，然后才能消费真正的字符串

```js
  const writableStream = new WritableStream({
    write(chunk) {
      const decoder = new TextDecoder();
      return new Promise((resolve, reject) => {
        const buffer = new ArrayBuffer(2);
        const view = new Uint16Array(buffer);
        view[0] = chunk;
        console.log(`decode = ${decoder.decode(view, { stream: true })}`);

        setTimeout(() => {
          resolve();
        }, 100);
      });
    },
    close() {
      console.log('writableStream in testTransformStreamCustom closed');
    },
  });
```

最后我们使用 pipeTo 对接

```js
  await readableStream.pipeTo(writableStream);
};
```

- 输出

```
testTransformStreamCustom
  decode = H
  decode = e
  decode = l
  decode = l
  decode = o
  decode = 
  decode = W
  decode = o
  decode = r
  decode = l
  decode = d
  writableStream in testTransformStreamCustom closed
```

### 5.3 TransformStream + pipeThrough

但是上面这个写法使我们需要改动 ReadableStream 和 WritableStream 其实不太好，我们应该像上面提过的水管一样，将数据的编码解码封装成一个中间流，保持每个 stream 的纯粹性

```js
const testTransformStream = async () => {
  const decoder = new TextDecoder();
  const transformStream = new TransformStream({
    transform: (chunk, controller) => {
      controller.enqueue(decoder.decode(chunk, { stream: true }));
    },
  });

  const readableStream = new ReadableStream(createReadableStreamSource());
  const writableStream = new WritableStream(
    createWritableStreamSource('testTransformStream')
  );
  await readableStream
    .pipeThrough(new TextEncoderStream())
    .pipeThrough(transformStream)
    .pipeTo(writableStream);
};
```

- 输出

```
testTransformStream
  [testTransformStream] chunk = H
  [testTransformStream] chunk = e
  [testTransformStream] chunk = l
  [testTransformStream] chunk = l
  [testTransformStream] chunk = o
  [testTransformStream] chunk =  
  [testTransformStream] chunk = W
  [testTransformStream] chunk = o
  [testTransformStream] chunk = r
  [testTransformStream] chunk = l
  [testTransformStream] chunk = d
```

## 6. 应用

代码的部分告一段落，下面我们看看 Stream 到底能应用在什么地方

- 非即时数据：当我们有一个不间断的数据不断产生的时候，恐怕没有人想去自己写一个 Promise 管理，会相当的麻烦，相反的是我们可以写成一个 Stream，然后不断的写入数据直到结束的时候调用 close 来关闭流
- 大数据传输/网络状况不佳：另一个就是与网络有关的情况了，当我们下载一个很大的数据的时候，数据总是分片出现的，我们使用流式写法的话就可以在小部分过来的时候马上进行数据的加工和处理；同时我们也能够更精准的控制数据流的移动，更方便我们进行监控、阻断、重试等方法

# 其他资源

## 参考连接

| Title                     | Link                                                                                                                         |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Streams API - MDN         | [https://developer.mozilla.org/zh-CN/docs/Web/API/Streams_API](https://developer.mozilla.org/zh-CN/docs/Web/API/Streams_API) |
| mdn/dom-examples - Github | [https://github.com/mdn/dom-examples/tree/master/streams](https://github.com/mdn/dom-examples/tree/master/streams)           |
| 精读《web streams》       | [https://mp.weixin.qq.com/s/K-KkZXt4Xj_g_I9R9BFJIQ](https://mp.weixin.qq.com/s/K-KkZXt4Xj_g_I9R9BFJIQ)                       |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_stream_api](https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_stream_api)
