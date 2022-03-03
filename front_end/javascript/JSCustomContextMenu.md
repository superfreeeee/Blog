# JS: 自定义右键操作列表 Context Menu

@[TOC](文章目录)

<!-- TOC -->

- [JS: 自定义右键操作列表 Context Menu](#js-自定义右键操作列表-context-menu)
- [默认 Context Menu](#默认-context-menu)
- [contextmenu 事件](#contextmenu-事件)
- [完整代码示例](#完整代码示例)
- [参考连接](#参考连接)

<!-- /TOC -->

# 默认 Context Menu

使用页面的时候，使用右键点击页面会弹出类似下面的可选操作列表，这个列表在浏览器中称为 `Context Menu`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_custom_context_menu_1_default.png)

实际上我们也可以像下面这样自定义自己的 Context Menu

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_custom_context_menu_2_custom.png)

- 上图应用：drawio

# contextmenu 事件

而我们要自己掌控这个右键点击事件有两种方式

- 第一种：contextmenu 事件

我们可以透过阻止 `contextmenu` 的默认行为，然后打开自己的 contextmenu 列表

```js
element.addEventListener('contextmenu', (e) => {
  e.preventDefault();
  openCustomContextMenu()
  
  // ...
}
```

- 第二种：mousedown 事件

第二种我们可以使用 mousedown 事件，并判断 `event.button` 的值来监听右键（需要注意的是这里依旧需要阻止默认的 contextmenu 事件，否则默认列表的优先级是更高的）

```js
element.addEventListener('contextmenu', (e) => {
  e.preventDefault();
});
element.addEventListener('mousedown', (e) => {
  if (e.button === 2) {
    openCustomContextMenu()
    
    // ...
  }
});
```

关于 `event.button` 的值分类可以参考 API 说明

# 完整代码示例

最后提供一个例子，整个页面分成三个区域：

- 红色为禁用默认 contextmenu
- 绿色为自定义 contextmenu
- 其余部分为默认 contextmenu

效果大致如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_custom_context_menu_3_sample.png)

有兴趣的可以下下来玩一玩～

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_custom_context_menu](https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_custom_context_menu)

# 参考连接

| Title                                     | Link                                                                                                                                                     |
| ----------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Element: contextmenu event - MDN web docs | [https://developer.mozilla.org/en-US/docs/Web/API/Element/contextmenu_event](https://developer.mozilla.org/en-US/docs/Web/API/Element/contextmenu_event) |
| DocumentFragment - MDN web docs           | [https://developer.mozilla.org/zh-TW/docs/Web/API/DocumentFragment](https://developer.mozilla.org/zh-TW/docs/Web/API/DocumentFragment)                   |
