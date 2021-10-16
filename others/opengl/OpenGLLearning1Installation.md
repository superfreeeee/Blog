# OpenGL 学习实录1: 基于 MacOS + Clion 配置 OpenGL 运行环境

@[TOC](文章目录)

<!-- TOC -->

- [OpenGL 学习实录1: 基于 MacOS + Clion 配置 OpenGL 运行环境](#opengl-学习实录1-基于-macos--clion-配置-opengl-运行环境)
- [正文](#正文)
  - [1. 相关库简介 & 安装](#1-相关库简介--安装)
    - [1.1 概念](#11-概念)
    - [1.2 安装](#12-安装)
  - [2. 配置运行环境](#2-配置运行环境)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

## 1. 相关库简介 & 安装

- OpenGL 核心图形绘制接口库
- GLFW 系统调用接口图形库
- glad 系统硬件驱动接入库

### 1.1 概念

三者联系：GLFW 封装系统调用操作(创建窗口、监听键盘输入) & 初始化 OpenGL 上下文，OpenGL 定义核心的图形绘制方法&接口，glad 自动查找硬件加速接口注入 OpenGL 的方法实现

### 1.2 安装

1. OpenGL

MacOS 系统自带 OpenGL 库，无需另外安装

查看 Mac 系统对应 OpenGL 版本：[https://support.apple.com/zh-cn/HT202823](https://support.apple.com/zh-cn/HT202823)

2. GLFW

使用 homebrew 工具安装

```bash
$ brew install glfw3
```

安装完毕可在 `/usr/local/Cellar/` 目录下可见

![](https://picures.oss-cn-beijing.aliyuncs.com/img/open_gl_learning_1_installation_1_glfw_install.png)

3. glad

于 web 上生成：[https://glad.dav1d.de/](https://glad.dav1d.de/)

选项：`Specification=OpenGL`、`Profile=Core`、`API:gl=<OpenGL version>`，最后点击右下角的 `GENERATE` 按钮生成所需的配置文件

![](https://picures.oss-cn-beijing.aliyuncs.com/img/open_gl_learning_1_installation_2_glad_create.png)

下载压缩包 `glad.zip` 文件后，将 `/include/` 目录下的两个子目录扔进 `/usr/local/include/` 目录下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/open_gl_learning_1_installation_3_glad_include.png)

而另一个 `/src/glad.c` 文件等下还会用到

## 2. 配置运行环境

在 Clion 中创建一个全新的 C++ 项目

![](https://picures.oss-cn-beijing.aliyuncs.com/img/open_gl_learning_1_installation_4_project_init.png)

将刚才的 `/src/glad.c` 文件跟默认生成的 `main.cpp` 文件扔到新项目的 `src` 目录下如下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/open_gl_learning_1_installation_5_project_update.png)

然后修改 `CMakeLists.txt` 配置文件（记得 glfw 改成你的版本号）

```cmake
cmake_minimum_required(VERSION 3.17)
project(open_gl_install)

set(CMAKE_CXX_STANDARD 14)

set(LOCAL_H /usr/local/include)
include_directories(${LOCAL_H})

set(GLFW_LINK /usr/local/Cellar/glfw/3.3.4/lib/libglfw.3.dylib)
link_libraries(${OPENGL} ${GLFW_LINK})

add_executable(OpenGL src/glad.c src/main.cpp)

if (APPLE)
    target_link_libraries(OpenGL "-framework OpenGL")
    target_link_libraries(OpenGL "-framework GLUT")
endif ()
```

最后在 `/src/main.cpp` 文件引入 glad、GLFW 库文件就大功告成啦（顺序不能乱）

```cpp
#include <glad//glad.h>
#include <GLFW/glfw3.h>
#include <iostream>

using namespace std;

int main() {
    cout << "Hello, World!" << endl;
    return 0;
}
```

# 其他资源

## 参考连接

| Title                                         | Link                                                                                                                                                                       |
| --------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Mac CLion下OpenGL环境配置                     | [https://www.cnblogs.com/shayue/p/Mac-CLion-xiaOpenGL-huan-jing-pei-zhi.html](https://www.cnblogs.com/shayue/p/Mac-CLion-xiaOpenGL-huan-jing-pei-zhi.html)                 |
| 创建窗口 - LearnOpenGL CN                     | [https://learnopengl-cn.github.io/01%20Getting%20started/02%20Creating%20a%20window/](https://learnopengl-cn.github.io/01%20Getting%20started/02%20Creating%20a%20window/) |
| 使用 OpenCL 和 OpenGL 图形处理程序的 Mac 电脑 | [https://support.apple.com/zh-cn/HT202823](https://support.apple.com/zh-cn/HT202823)                                                                                       |
| Glad 文件生成网站                             | [https://glad.dav1d.de/](https://glad.dav1d.de/)                                                                                                                           |
| glfw glew glm glut 傻傻分不清楚               | [https://shengyu7697.github.io/glfw-glew-glm-glut/](https://shengyu7697.github.io/glfw-glew-glm-glut/)                                                                     |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/others/open_gl/open_gl_install](https://github.com/superfreeeee/Blog-code/tree/main/others/open_gl/open_gl_install)
