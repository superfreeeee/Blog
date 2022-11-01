# Webpack 配置: 自定义网站图标 favicon

@[TOC](文章目录)

<!-- TOC -->

- [Webpack 配置: 自定义网站图标 favicon](#webpack-配置-自定义网站图标-favicon)
- [实现核心技术: html-webpack-plugin 插件](#实现核心技术-html-webpack-plugin-插件)
- [完整代码示例](#完整代码示例)
- [参考连接](#参考连接)

<!-- /TOC -->

# 实现核心技术: html-webpack-plugin 插件

我们可以透过 html-webpack-plugin 来实现自定义 favicon 的配置

从原生的角度来说，我们要自定义 favicon 需要在 html 文件的 `<head>` 标签内加上这句话

```html
<link rel="shortcut icon" type="image/icon" href="/favicon.ico">
```

在使用 webpack 打包的场景下我们则通过 html-webpack-plugin 插件来实现 icon 注入

# 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/webpack/webpack_html_favicon](https://github.com/superfreeeee/Blog-code/tree/main/front_end/webpack/webpack_html_favicon)

配置方式

- `webpack.config.js`

```js
module.exports = {
  // ...
  plugins: [
    new HtmlWebpackPlugin({
      // filename, titile, template ...
      
      favicon: './public/favicon32.ico',
    }),
  ],
}
```

效果如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/webpack_html_favicon_1.png)

# 参考连接

| Title                                      | Link                                                                                                                     |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------ |
| jantimon/html-webpack-plugin - Github      | [https://github.com/jantimon/html-webpack-plugin#options](https://github.com/jantimon/html-webpack-plugin#options)       |
| React 中使用 webpack 动态添加 favicon 图标 | [https://blog.csdn.net/m0_37128507/article/details/83269084](https://blog.csdn.net/m0_37128507/article/details/83269084) |
| ico 图标制作                               | [https://www.dute.org/ico-converter](https://www.dute.org/ico-converter)                                                 |
