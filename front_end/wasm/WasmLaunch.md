# WASM 初探: JS 调用 C 模块

@[TOC](文章目录)

<!-- TOC -->

- [WASM 初探: JS 调用 C 模块](#wasm-初探-js-调用-c-模块)
- [正文](#正文)
  - [0. 环境背景](#0-环境背景)
  - [1. 安装 emcc 编译器](#1-安装-emcc-编译器)
  - [2. 准备 C 源码 & 编译成 wasm](#2-准备-c-源码--编译成-wasm)
  - [3. JS 调用 WASM](#3-js-调用-wasm)
  - [4. 更多测试：JS 与 C 代码效率比较、分析](#4-更多测试js-与-c-代码效率比较分析)
  - [5. 遗留问题](#5-遗留问题)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

## 0. 环境背景

- JS 运行环境：Node
- C 编译工具：emcc(emscripten)

## 1. 安装 emcc 编译器

[https://emscripten.org/docs/getting_started/downloads.html](https://emscripten.org/docs/getting_started/downloads.html)

1. 下载源码

```bash
$ git clone https://github.com/emscripten-core/emsdk.git
$ cd emsdk
```

2. 源码安装

```bash
$ ./emsdk install latest
$ ./emsdk activate latest
```

3. 环境变量

```bash
$ source ./emsdk_env.sh
```

设置环境变量后才能在命令行使用 emcc 指令，可以选择加入到如 `~/.bash_profile` 的启动脚本中

4. 检查是否安装成功

```bash
$ emcc -v
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/wasm_launch_1_emcc_install.png)

## 2. 准备 C 源码 & 编译成 wasm

这里就不使用官方的带 Web Server 的示例了，直接编译一个作为库函数的模块

1. 编写 C 代码

- `/basic/main.c`

```c
int add(int x, int y) {
    return x + y;
}
```

2. 接下来写一个 makefile 脚本来完成编译

- `/basic/makefile`

```makefile
init:
	emcc main.c \
	-o main.wasm \
	-Os \
	-s WASM=1 \
	-s SIDE_MODULE=1
```

3. 执行编译

```bash
$ make
```

## 3. JS 调用 WASM

JS 要直接调用 WASM 需要依赖一个 WebAssembly 对象，具体写法如下

- `/basic/index.c`

```js
const fs = require('fs');

const src = new Uint8Array(fs.readFileSync('./main.wasm'));

const env = {
  memoryBase: 0,
  tableBase: 0,
  memory: new WebAssembly.Memory({
    initial: 256,
  }),
  table: new WebAssembly.Table({
    initial: 2,
    element: 'anyfunc',
  }),
  abort: () => {
    throw 'abort';
  },
};

WebAssembly.instantiate(src, { env })
  .then((res) => {
    const _instance = res.instance;
    const { add } = _instance.exports;

    console.log(`add(1, 2)=${add(1, 2)}`);
  })
  .catch((err) => {
    console.error('err', err);
    return Promise.reject(err);
  });
```

首先将 wasm 文件读成二进制文件流(Uint8Array 类型)，然后声明一个 env 环境配置对象，最后使用 `WebAssembly.instantiate` 来载入具体脚本，可以在 `res.instance.exports` 中找到导出的方法

![](https://picures.oss-cn-beijing.aliyuncs.com/img/wasm_launch_2_basic.png)

## 4. 更多测试：JS 与 C 代码效率比较、分析

接下来我们分别在 C 与 JS 中写出两个相同逻辑的方法，并进行测试（具体测试脚本不详细说明，主要展示比较用的函数逻辑）

分别是一个三次方的算术运算方法，以及一个斐波那契数列方法

- `/test/test.c`

```c
int sum(int times) {
    int ans = 0;
    for(int k=0; k<times; k++) {
        for(int j=0; j<times; j++) {
            for(int i=0; i<times; i++) {
                ans += i;
            }
        }
    }
    return ans;
}

int fac(int n) {
    return n <= 2 ? 1 : fac(n - 1) + fac(n - 2);
}
```

- `/test/test.js`

```js
const sum = (times) => {
  let ans = 0;
  for (let k = 0; k < times; k++) {
    for (let j = 0; j < times; j++) {
      for (let i = 0; i < times; i++) {
        ans += i;
      }
    }
  }
  return ans;
};

const fac = (n) => (n <= 2 ? 1 : fac(n - 1) + fac(n - 2));

module.exports = {
  sumInJs: sum,
  facInJs: fac,
};
```

测试用代码如下

- `/test/index.js`

```js
const createSumTest = (sum) => () => {
  sum(1500);
};

const createFacTest = (fac) => () => {
  fac(40);
};
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/wasm_launch_3_test.png)

- 指标含义
  - in C: C 编译后的 wasm 模块
  - in Js: 原始 js 模块
  - delta: 时间差
  - times: C 版本对于 Js 版本的执行效率倍数

我们发现对于大量循环代码 C 语言版本估计经过某种优化，几乎只耗费了个位数毫秒，但是 JS 版本甚至需要花费数秒的时间

第二个测试斐波那契函数，则有可能是由于调用栈的问题，而产生了接近 2.5 倍的效率

## 5. 遗留问题

看起来确实提升了不少效率，然而对于 wasm 的使用还是存在几个问题

- 未成功进行三方库的链接
- I/O 相关操作，乃至系统级操作函数的映射

以上问题作者暂时还没研究出来，本篇就仅仅只是对于 wasm 的简单尝试哈

# 其他资源

## 参考连接

| Title                                       | Link                                                                                                                             |
| ------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Developer's Guide                           | [https://webassembly.org/getting-started/developers-guide/](https://webassembly.org/getting-started/developers-guide/)           |
| Compiling a New C/C++ Module to WebAssembly | [https://developer.mozilla.org/en-US/docs/WebAssembly/C_to_wasm](https://developer.mozilla.org/en-US/docs/WebAssembly/C_to_wasm) |
| Download and install - Emscripten           | [https://emscripten.org/docs/getting_started/downloads.html](https://emscripten.org/docs/getting_started/downloads.html)         |
| WebAssembly完全入门——了解wasm的前世今身     | [https://www.cnblogs.com/detectiveHLH/p/9928915.html](https://www.cnblogs.com/detectiveHLH/p/9928915.html)                       |
| 初探WebAssembly                             | [https://www.cnblogs.com/detectiveHLH/p/9881640.html](https://www.cnblogs.com/detectiveHLH/p/9881640.html)                       |
| WebAssembly之emcc编译命令                   | [https://www.jianshu.com/p/d5a60bc43940](https://www.jianshu.com/p/d5a60bc43940)                                                 |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/wasm/wasm_launch](https://github.com/superfreeeee/Blog-code/tree/main/front_end/wasm/wasm_launch)
