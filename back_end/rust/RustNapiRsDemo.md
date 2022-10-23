# Rust: 基于 napi-rs 开发 Node.js 原生模块

@[TOC](文章目录)

<!-- TOC -->

- [Rust: 基于 napi-rs 开发 Node.js 原生模块](#rust-基于-napi-rs-开发-nodejs-原生模块)
- [完整代码示例](#完整代码示例)
- [背景 & napi](#背景--napi)
- [环境/工具链准备](#环境工具链准备)
- [创建项目](#创建项目)
- [打包 & 测试](#打包--测试)
- [参考链接](#参考链接)

<!-- /TOC -->

# 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/back_end/rust/rust_napi_rs_test](https://github.com/superfreeeee/Blog-code/tree/main/back_end/rust/rust_napi_rs_test)

# 背景 & napi

作为前端人都知道 V8 引擎以及在此之上的 Node.js 的出现，改写了 JavaScript 这门语言的命运。JS 不再只能运行在浏览器上，与此相关的工具链也蓬勃发展。

不过作为 web 开发的工程师或许比较少接触到所谓的 C++ 模块，充其量就是基于 Node.js 对外暴露的 API 来写点工程化相关的胶水代码。

实际上 Node.js 除了在 JS 的层级上提供了众多内置模块之外，还提供了一层称为 n-api 的 C++ 级别的抽象接口层，我们可以通过 napi 编写一个 C++ 模块直接在更底层的位置调用 Node.js 的 API。

但是老实说作者就是一个 C++ 菜鸡，也不是特别喜欢用 C++ 写东西。因此今天要带给大家的是基于 napi-rs 框架，使用 Rust 语言编写来编写并编译成原生模块。

Let‘s go

# 环境/工具链准备

首先是环境准备，不用说 rust 相关的编译器肯定是要整好的，`rustc`、`cargo` 等工具就不多说了，自己上官网一次整好吧

[传送门：Rust 官网](https://www.rust-lang.org/tools/install)

（可选）除此之外，还有就是在全局装一个 `@napi-rs/cli`，该框架提供了一个类似脚手架的命令行工具，用于创建基础项目框架

```bash
pnpm i -g @napi-rs/cli
```

这里要用 npm、yarn、pnpm 都随便，就不多说了

# 创建项目

基础工具链都装好之后，就可以来创建项目啦，使用 `napi new` 然后选择适合自己的配置选项（其实也没几个）

![](https://picures.oss-cn-beijing.aliyuncs.com/img/rust_napi_rs_demo_1_new.png)

创建好之后我们可以看到一个预先配置好的模版，有 napi 构建项目的入口

- `build.rs`

```rs
extern crate napi_build;

fn main() {
  napi_build::setup();
}
```

然后是作为三方库时候的主入口 `lib.rs`

- `/src/lib.rs`

```rs
#![deny(clippy::all)]

#[macro_use]
extern crate napi_derive;

#[napi]
pub fn sum(a: i32, b: i32) -> i32 {
    a + b
}
```

甚至连测试用的测试用例也给我们准备好了

- `/__test__/index.spec.mjs`

```js
import test from 'ava'

import { sum } from '../index.js'

test('sum from native', (t) => {
  t.is(sum(1, 2), 3)
})
```

# 打包 & 测试

接下来我们只要运行 `pnpm build` 运行 build 指令就会开始构建啦

运行完成之后我们会在根目录下面看到三个新文件

- `index.d.ts`

第一个是我们打包出来之后的库函数的类型定义

![](https://picures.oss-cn-beijing.aliyuncs.com/img/rust_napi_rs_demo_2_type.png)

- `index.js`

第二个则是用于磨平平台差异的 binding 用的代码

![](https://picures.oss-cn-beijing.aliyuncs.com/img/rust_napi_rs_demo_3_binding.png)

第三个 `xxx.node` 则是 Rust 代码经过编译，将 napi 调用一同编译过后生成的类似动态运行库，副档名 `.node` 说明是专属于 Node.js 的原生模块，其实本质上就是 MacOS 下 `.dylib` 或是 Windows 下 `.dll` 的动态库文件

最后都编译完成之后，我们就可以运行 `pnpm test` 测试一下功能是否正常

![](https://picures.oss-cn-beijing.aliyuncs.com/img/rust_napi_rs_demo_4_test.png)

整体体验上还是非常丝滑。虽然还没有真正用上 napi 暴露出来的 Node.js 的底层 API，但是光是能够接入 Rust 语言这一步就能看得出来真实投入使用时候的潜力。

# 参考链接

| Title                                                | Link                                                                                                     |
| ---------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| 从暴力到 NAN 再到 NAPI——Node.js 原生模块开发方式变迁 | [https://cnodejs.org/topic/5957626dacfce9295ba072e0](https://cnodejs.org/topic/5957626dacfce9295ba072e0) |
| napi-rs                                              | [https://napi.rs/](https://napi.rs/)                                                                     |
| Install Rust - Rust                                  | [https://www.rust-lang.org/tools/install](https://www.rust-lang.org/tools/install)                       |
| napi-rs - Github                                     | [https://github.com/napi-rs/napi-rs](https://github.com/napi-rs/napi-rs)                                 |
