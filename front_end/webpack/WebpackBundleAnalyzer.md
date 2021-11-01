# Webpack 插件分享: webpack-bundle-analyzer 打包文件分析工具

@[TOC](文章目录)

<!-- TOC -->

- [Webpack 插件分享: webpack-bundle-analyzer 打包文件分析工具](#webpack-插件分享-webpack-bundle-analyzer-打包文件分析工具)
- [正文](#正文)
  - [0. 为什么需要分析？](#0-为什么需要分析)
  - [1. 功能 & 范例](#1-功能--范例)
  - [2. 引入方式](#2-引入方式)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

## 0. 为什么需要分析？

前端工程化越来越庞大，对于三方库、源代码的编译、打包等能力要求越来越高，如果一不小心打了太大的东西进去会造成客户端渲染的时候严重延迟，影响用户体验

## 1. 功能 & 范例

webpack-bundle-analyzer 插件提供了一种对打包体积文件大小的可视化展示，并支持简单的搜索，是当我们需要对打包成果进行详细控制的时候非常好用的一个参考对象

下面两张图分别是利用 `@youxian/cli` 构建前端应用时，在开发环境和生产环境下分别利用 webpack-bundle-analyzer 分析时看到的分析结果

- 开发环境

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_bundle_analyzer_1_dev.png)

- 生产环境

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_bundle_analyzer_2_prod.png)

## 2. 引入方式

- 安装依赖

```bash
$ yarn add webpack-bundle-analyzer -D
```

- 配置文件引入插件：`webpack.config.js`

```js
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');

module.exports = {
    // ...
    plugins: [
        // ...
        new BundleAnalyzerPlugin()
    ]
}
```

- 详细参数用法

自己去查！[https://github.com/webpack-contrib/webpack-bundle-analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer)

# 其他资源

## 参考连接

| Title                                            | Link                                                                                                                     |
| ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------ |
| webpack-bundle-analyzer打包文件分析工具          | [https://www.jianshu.com/p/0c05faee0975](https://www.jianshu.com/p/0c05faee0975)                                         |
| webpack打包优化解决方案                          | [https://segmentfault.com/a/1190000011138081](https://segmentfault.com/a/1190000011138081)                               |
| webpack-bundle-analyzer - npm                    | [https://www.npmjs.com/package/webpack-bundle-analyzer](https://www.npmjs.com/package/webpack-bundle-analyzer)           |
| webpack-contrib/webpack-bundle-analyzer - github | [https://github.com/webpack-contrib/webpack-bundle-analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer) |
| Awesome webpack #webpack plugins                 | [https://webpack.js.org/awesome-webpack/#webpack-plugins](https://webpack.js.org/awesome-webpack/#webpack-plugins)       |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/webpack/webpack_bundle_analyzer](https://github.com/superfreeeee/Blog-code/tree/main/front_end/webpack/webpack_bundle_analyzer)
