# Flask 踩坑笔记: localhost:5000 无法访问/host,port 配置无效(Error: Failed to load resource: the server responded with a status of 403 ())

@[TOC](文章目录)

<!-- TOC -->

- [Flask 踩坑笔记: localhost:5000 无法访问/host,port 配置无效(Error: Failed to load resource: the server responded with a status of 403 ())](#flask-踩坑笔记-localhost5000-无法访问hostport-配置无效error-failed-to-load-resource-the-server-responded-with-a-status-of-403-)
- [0. 项目背景](#0-项目背景)
- [1. 问题描述](#1-问题描述)
- [2. 问题排查 & 解决方案](#2-问题排查--解决方案)
  - [2.1 问题一：localhost:5000 无法访问](#21-问题一localhost5000-无法访问)
  - [2.2 问题二：修改域名/端口无效](#22-问题二修改域名端口无效)
- [3. 小结](#3-小结)
- [参考连接](#参考连接)
- [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 0. 项目背景

- Web 框架：Flask
- IDE：PyCharm

# 1. 问题描述

基于 PyCharm 可视化界面创建并启动 Flask 应用，无法访问 `http://localhost:5000`，修改 `app.run(host= , port= )` 启动参数也无效

![](https://picures.oss-cn-beijing.aliyuncs.com/img/python_note_flask_origin_3_error.png)

# 2. 问题排查 & 解决方案

## 2.1 问题一：localhost:5000 无法访问

可能原因有

- host 不对
- 5000 端口占用

1. host 不对

Flask 框架默认使用 `127.0.0.1` 回环地址作为启动域名，因此无法使用 `localhost` 访问

- 解决方案：改成使用 `http://127.0.0.1::5000` 访问

2. 5000 端口占用

第二种可能是 5000 端口已经被占用了，输入命令查看端口信息

```bash
netstat -lat
```

- 解决方案：换个端口呗，改成使用 `app.run(port=8900)`，挑个你喜欢的数字哈

## 2.2 问题二：修改域名/端口无效

原因大致就是你用了 PyCharm 的启动命令

![](https://picures.oss-cn-beijing.aliyuncs.com/img/python_note_flask_origin_1_cmd.png)

当然 JetBrain 应该是有其他配置办法，不过实在是太懒了，直接改成简单的 python 启动方式吧

![](https://picures.oss-cn-beijing.aliyuncs.com/img/python_note_flask_origin_2_new.png)

按上图选择左侧的一般 Python 脚本，然后在右侧 `Script Path` 指定启动脚本即可

# 3. 小结

总结就是两个点避免乱七八糟的问题hh

1. 不要使用 PyCharm 的 Flask 启动脚本，当成普通的 Python 脚本就好了
2. 域名设为 `localhost` 让 `localhost`、`127.0.0.1` 都能访问；换个大一点的端口吧

```py
if __name__ == '__main__':
    app.run(host="localhost", port=8900)
```

# 参考连接

| Title                                                      | Link                                                                                                                                                                           |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Why does localhost:5000 not work in Flask? - stackoverflow | [https://stackoverflow.com/questions/46127005/why-does-localhost5000-not-work-in-flask](https://stackoverflow.com/questions/46127005/why-does-localhost5000-not-work-in-flask) |
| flask启动时host和port配置未生效【flask】                   | [https://blog.csdn.net/weixin_44041700/article/details/112591946](https://blog.csdn.net/weixin_44041700/article/details/112591946)                                             |
| python-----flask项目端口设置无效                           | [https://www.cnblogs.com/xiaodai0/p/10460751.html](https://www.cnblogs.com/xiaodai0/p/10460751.html)                                                                           |

# 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/back_end/python/python_note_flask_origin](https://github.com/superfreeeee/Blog-code/tree/main/back_end/python/python_note_flask_origin)
