# Rust 构建 Wasm 模块

@[TOC](文章目录)

<!-- TOC -->

- [Rust 构建 Wasm 模块](#rust-构建-wasm-模块)
- [正文](#正文)
  - [1. 安装](#1-安装)
    - [1.1 使用 Rustup 安装 Rust](#11-使用-rustup-安装-rust)
    - [1.2 安装 wasm-pack](#12-安装-wasm-pack)
    - [1.3 CLion 配置](#13-clion-配置)
  - [2. 手动构建 Rust to Wasm 项目](#2-手动构建-rust-to-wasm-项目)
    - [2.1 创建项目](#21-创建项目)
    - [2.2 配置文件 Cargo.toml](#22-配置文件-cargotoml)
    - [2.3 模块导出函数 & 打包](#23-模块导出函数--打包)
    - [2.4 前端项目展示](#24-前端项目展示)
  - [3. 使用 wasm-pack 模版构建](#3-使用-wasm-pack-模版构建)
    - [3.1 配置文件补全](#31-配置文件补全)
    - [3.2 模块导出函数 & 打包](#32-模块导出函数--打包)
    - [3.3 前端 @wasm-tool/wasm-pack-plugin 插件](#33-前端-wasm-toolwasm-pack-plugin-插件)
    - [3.4 引入 wasm 模块 & 运行效果](#34-引入-wasm-模块--运行效果)
  - [4. 小结](#4-小结)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

今天来给大家介绍如何使用 Rust 构建 Wasm 模块

## 1. 安装

第一部分是相关依赖安装

### 1.1 使用 Rustup 安装 Rust

首先是安装 Rust 的部分，如果读者使用 `brew install rust` 直接安装 Rust 模块的话，建议还是删掉重新使用 Rustup 安装完整工具链hh（由于我们的 Wasm 编译相当于是一个独立的跨平台编译目标，因此必须使用完整的 rustup 工具链，否则后面的构建过程会失败）

- 官方推荐安装方式

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

或是你比较喜欢使用 Homebrew 的话也可以用 Homebrew 来安装

- Homebrew

```bash
brew install rustup
```

然后下面再执行命令来进行环境变量初始化

- 初始化环境变量

```bash
source ~/.cargo/env
```

或是你可以把下面的路径加入到你的 `~/.zshrc` 或是 `~/.bash_profile` 里面

```bash
export PATH=$HOME/.cargo/bin:$PATH
```

最后在终端里面测试一下是否安装成功

```bash
% rustup --version
rustup 1.24.3 (ce5817a94 2021-05-31)
info: This is the version for the rustup toolchain manager, not the rustc compiler.
info: The currently active `rustc` version is `rustc 1.56.1 (59eed8a2a 2021-11-01)`
```

```bash
% rustc --version
rustc 1.56.1 (59eed8a2a 2021-11-01)
```

```bash
% cargo --version
cargo 1.56.0 (4ed5d137b 2021-10-04)
```

### 1.2 安装 wasm-pack

第二个重要的工具是一个团队针对 Rust 对于 Wasm 的支持开发一个全面的 wasm-pack 工具，我们可以使用 `cargo` 进行安装

```bash
% cargo install wasm-pack
% wasm-pack --version
wasm-pack 0.10.1
```

### 1.3 CLion 配置

最后我们使用的 IDE 是 CLion，先安装 Rust 插件然后配置 Rust 工具链路径

创建项目之前可以先在  里面配置工具链路径（当然要先安装好 Rust 插件啦）

- 插件安装：`Preference > Plugins 搜寻 Rust`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/rust_wasm_1_plugin.png)

- Rust 工具链路径配置：`Preference > Language & Frameworks > Rust`
  - Toolchain location：`~/.cargo/bin`
  - Standard Library：`~/.rustup/toolchains/stable-aarch64-apple-darwin/lib/rustlib/src/rust`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/rust_wasm_2_env.png)

通常选好第一个第二个它会自动补全的

## 2. 手动构建 Rust to Wasm 项目

接下来第一种我们先尝试直接使用刚才安装好的工具手动构建一个 Rust to Wasm 项目

### 2.1 创建项目

创建项目的部分我们直接使用 `cargo` 提供的默认选项

```bash
cargo new --lib manual
```

或是使用 CLion 提供的等价图形化选项

![](https://picures.oss-cn-beijing.aliyuncs.com/img/rust_wasm_3_clion_lib.png)

### 2.2 配置文件 Cargo.toml

创建好项目之后我们先不急着写代码，先配置好 `Cargo.toml` 文件配置

- `/manual/Cargo.toml`

```toml
[package]
name = "rust_wasm"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[lib]
crate-type = ["cdylib"]

[dependencies]
wasm-bindgen = "0.2.78"

[package.metadata.wasm-pack.profile.release]
wasm-opt = false
```

首先指定构建目标 `crate-type = ["cdylib"]`，然后记得关闭 `wasm-opt = false`

### 2.3 模块导出函数 & 打包

接下来就是库模块的主入口写一个导出函数

- `/manual/src/lib.rs`

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
extern "C" {
    fn alert(s: &str);
}

#[wasm_bindgen]
pub fn greet(name: &str) {
    alert(&format!("[manulaRustModule] Hello, {}!", name));
}
```

加上 `#[wasm_bindgen]` 等下打包的时候才会被编译为导出函数

下面我们就可以配置打包命令了

- CLion 执行命令

使用 CLion 的话选择 wasm-pack 命令

![](https://picures.oss-cn-beijing.aliyuncs.com/img/rust_wasm_4_manual_web.png)

- 命令行

如果你是使用命令行的话，则是运行下面的指令

```bash
wasm-pack build --target web
```

本质上我们就是要使用 `--target web` 打包成 Web 浏览器能直接读取的方式

### 2.4 前端项目展示

针对第一种方式，由于 wasm-pack 默认打包成了 ESM 模块，我们使用 webpack 来支持一下 ESM，简单的 webpack 配置我们就先不提了，所以我们直接看主入口

- `/index.js`

```js
import * as manualRustModule from './manual/pkg/rust_wasm';
console.log('manulaRustModule', manualRustModule);

manualRustModule.default().then(() => {
  manualRustModule.greet('superfree');
});
```

- 结果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/rust_wasm_5_manual_res.png)

我们可以看到由于我们直接使用 `wasm-pack --target web` 的打包方式，所以我们导出的模块使用 `default` 方法进行初始化然后调用模块下的 `greet` 方法

## 3. 使用 wasm-pack 模版构建

前面的方法还是比较琐碎的，接下来第二种方法我们可以直接使用 wasm-pack 提供的模版创建项目

更偷懒的我们直接使用 CLion 提供的安装选项哈

![](https://picures.oss-cn-beijing.aliyuncs.com/img/rust_wasm_6_template.png)

### 3.1 配置文件补全

创建好模版之后，我们还是要稍微修改一下配置文件，否则打包的时候老是警告hh

- `/pack_template/Cargo.toml`

```toml
[package.metadata.wasm-pack.profile.release]
wasm-opt = false
```

一样是关闭 `wasm-opt = false` 来避免打包错误

### 3.2 模块导出函数 & 打包

导出函数的部分其实写法也是差不多

- `/pack_template/src/lib.rs`

```rust
mod utils;

use wasm_bindgen::prelude::*;

// When the `wee_alloc` feature is enabled, use `wee_alloc` as the global
// allocator.
#[cfg(feature = "wee_alloc")]
#[global_allocator]
static ALLOC: wee_alloc::WeeAlloc = wee_alloc::WeeAlloc::INIT;

#[wasm_bindgen]
extern {
    fn alert(s: &str);
}

#[wasm_bindgen]
pub fn greet(name: &str) {
    alert(&format!("[packTemplateModule] Hello {}!", name));
}
```

打包则是直接使用默认的 `wasm-pack build` 就可以了，默认采用的是 `--target bundler`，也就是打包成适合 webpack 或其他打包工具适合的导入方式

![](https://picures.oss-cn-beijing.aliyuncs.com/img/rust_wasm_7_builde.png)

### 3.3 前端 @wasm-tool/wasm-pack-plugin 插件

打包成功之后，我们前端也要做出相应的改变，引入 `@wasm-tool/wasm-pack-plugin` 插件，来对 Rust 项目进行初始化

- `webpack.config.js`

```js
const path = require('path');
const webpack = require('webpack');

const HtmlWebpackPlugin = require('html-webpack-plugin');
const WasmPackPlugin = require('@wasm-tool/wasm-pack-plugin');

module.exports = {
  mode: 'development',
  entry: './index.js',
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html',
    }),
    new WasmPackPlugin({
      crateDirectory: path.resolve(__dirname, 'pack_template'),
    }),
    // Have this example work in Edge which doesn't ship `TextEncoder` or
    // `TextDecoder` at this time.
    new webpack.ProvidePlugin({
      TextDecoder: ['text-encoding', 'TextDecoder'],
      TextEncoder: ['text-encoding', 'TextEncoder'],
    }),
  ],
  experiments: {
    asyncWebAssembly: true,
  },
};
```

这里有几个注意点

1. 使用 `WasmPackPlugin` 配置 Rust 项目根目录 `crateDirectory: path.resolve(__dirname, 'pack_template')`
2. 开启 `experiments.asyncWebAssembly: true` 选项

### 3.4 引入 wasm 模块 & 运行效果

最后直接引入 wasm 代码

- `index.js`

```js
import * as packTemplateModule from './pack_template/pkg';

console.log('packTemplateModule', packTemplateModule);
packTemplateModule.greet('superfree');
```

来看看运行效果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/rust_wasm_8_pack_template_res.png)

## 4. 小结

尝试完 Rust to Wasm 的支持之后，不得不说体验上是比 C++ & emscription 的体验要好很多（传送门：[WASM 初探: JS 调用 C 模块](https://blog.csdn.net/weixin_44691608/article/details/120649157)）

wasm-pack 团队的努力让我们可以轻松使用 Rust 来编写 wasm 模块，同时直接默认支持前端工具链的 bundler，可以说是非常契合前端工程化

# 其他资源

## 参考连接

| Title                                                   | Link                                                                                                                                   |
| ------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| 安裝 Rust - Rust                                        | [https://www.rust-lang.org/zh-TW/tools/install](https://www.rust-lang.org/zh-TW/tools/install)                                         |
| Quickstart - Hello wasm-pack!                           | [https://rustwasm.github.io/docs/wasm-pack/quickstart.html](https://rustwasm.github.io/docs/wasm-pack/quickstart.html)                 |
| Introduction - The wasm-bindgen Guide                   | [https://rustwasm.github.io/docs/wasm-bindgen/introduction.html](https://rustwasm.github.io/docs/wasm-bindgen/introduction.html)       |
| Compiling from Rust to WebAssembly - MDN                | [https://developer.mozilla.org/en-US/docs/WebAssembly/Rust_to_wasm](https://developer.mozilla.org/en-US/docs/WebAssembly/Rust_to_wasm) |
| no prebuilt wasm-opt binaries error #913 - Github issue | [https://github.com/rustwasm/wasm-pack/issues/913](https://github.com/rustwasm/wasm-pack/issues/913)                                   |
| 使用 Rust 编写更快的 React 组件                         | [https://mp.weixin.qq.com/s/QcnZn-_SefY2ysIz1SLX0w](https://mp.weixin.qq.com/s/QcnZn-_SefY2ysIz1SLX0w)                                 |
| WASM 初探: JS 调用 C 模块                               | [https://blog.csdn.net/weixin_44691608/article/details/120649157](https://blog.csdn.net/weixin_44691608/article/details/120649157)     |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/back_end/rust/rust_wasm](https://github.com/superfreeeee/Blog-code/tree/main/back_end/rust/rust_wasm)
