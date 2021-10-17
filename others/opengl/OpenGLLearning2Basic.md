# OpenGL 学习实录2: 基础绘制初试

@[TOC](文章目录)

<!-- TOC -->

- [OpenGL 学习实录2: 基础绘制初试](#opengl-学习实录2-基础绘制初试)
- [正文](#正文)
  - [0. 概念 & 步骤](#0-概念--步骤)
  - [1. 实做记录](#1-实做记录)
    - [1.1 项目目录结构 & CmakeLists 配置](#11-项目目录结构--cmakelists-配置)
    - [1.2 程序结构](#12-程序结构)
    - [1.3 初始化 GLFW & 建立窗口](#13-初始化-glfw--建立窗口)
    - [1.4 构建着色器程序](#14-构建着色器程序)
    - [1.5 创建数据缓冲对象](#15-创建数据缓冲对象)
    - [1.6 绘制程序 & 退出时的资源清理](#16-绘制程序--退出时的资源清理)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

完成前一篇[OpenGL 学习实录1: 基于 MacOS + Clion 配置 OpenGL 运行环境](https://blog.csdn.net/weixin_44691608/article/details/120806889)的运行环境配置，第二篇记录关于第一次尝试绘制的步骤

## 0. 概念 & 步骤

- OpenGL 与 Web 的 Canvas 相似，都是透过所谓的上下文的绘图模式(Canvas2D 的 RenderingContext，OpenGL 的全局空间等)
- 数据对象
  - OpenGL 操作的对象为缓冲对象(Buffer Object)，需要透过如 `glGenBuffers` 方法来创建
  - 缓冲对象分为`顶点缓冲对象(Vertex Buffer Object = VBO)`、`元素/索引缓冲对象(Element Buffer Object = EBO / Index Buffer Object = IBO)`
  - 使用`顶点数列对象(Vertex Array Object = VAO)`来记录缓冲对象、索引对象的集合
- 渲染对象
  - 顶点着色器(VertexShader)
  - 片段着色器(FragmentShader)
  - 着色器程序(ShaderProgram)

## 1. 实做记录

### 1.1 项目目录结构 & CmakeLists 配置

参照 [OpenGL 学习实录1: 基于 MacOS + Clion 配置 OpenGL 运行环境](https://blog.csdn.net/weixin_44691608/article/details/120806889) 配置完成后的目录结构

### 1.2 程序结构

操作步骤伪代码

```
int main() {
    初始化 GLFW
    检查 OpenGL 版本
    创建 GLFW 窗口

    使用 glad 检查 OpenGL 接口绑定
    创建顶点着色器
    创建片段着色器
    创建着色器程序 & 与前两个着色器绑定
    绑定完成后可清理着色器
    
    创建顶点数据并写入缓冲区
    
    while(不应关闭窗口) {
        处理用户输入

        具体绘制流程
            清理屏幕
            绑定着色器程序
            绑定绘制缓冲对象
            绘制图形

        接受用户输入事件
        交换视频缓冲区(渲染上屏)
    }

    释放缓冲对象
    结束 GLFW 窗口
    return 0;
}
```

### 1.3 初始化 GLFW & 建立窗口

- 程序头文件

```cpp
#include <glad/glad.h>
#include <GLFW/glfw3.h>
#include <iostream>

using namespace std;

int main() {
    // ...
}
```

- 初始化 GLFW

```cpp
int main() {
//    1. 初始化 glfw
    if (glfwInit() == GLFW_FALSE) {
        cout << "Fail to initialize GLFW" << endl;
        return -1;
    }

//    2. glfw 配置项(OpenGL 版本号、绘制模式)
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 4);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 1);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

#ifdef __APPLE__
    glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);
#endif

//    3. 创建 glfw 窗口
    GLFWwindow *window = glfwCreateWindow(800, 600, "LearnOpenGL", NULL, NULL);
    // 3.1 检查窗口是否创建成功
    if (window == NULL) {
        cout << "Failed to create GLFW window" << endl;
        glfwTerminate();
        return -1;
    }
    // 3.2 加入到当前上下文
    glfwMakeContextCurrent(window);
    // 3.3 窗口大小更新
    glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);

// more
}
```

```cpp
// 窗口大小更新处理函数
void framebuffer_size_callback(GLFWwindow *window, int width, int height) {
    // 重制可见视图
    glViewport(0, 0, width, height);
}
```

### 1.4 构建着色器程序

第二部分在于定制 OpenGL 开放的着色器程序，主要定制顶点着色器和片段着色器

- 顶点着色器

```glsl
#version 410 core
layout (location = 0) in vec3 aPos;
void main() {
    gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
}
```

- 片段着色器

```glsl
#version 410 core
out vec4 color;
void main() {
    color = vec4(1.0f, 0.5f, 0.2f, 1.0f);
}
```

作为 C++ 字符串

```cpp
// 顶点着色器 glsl 源码
const char *vertexShaderSource = "#version 410 core\n"
                                 "layout (location = 0) in vec3 aPos;\n"
                                 "void main() {\n"
                                 "    gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);\n"
                                 "}\0";

// 片段着色器 glsl 源码
const char *fragmentShaderSource = "#version 410 core\n"
                                   "out vec4 color;\n"
                                   "void main() {\n"
                                   "    color = vec4(1.0f, 0.5f, 0.2f, 1.0f);\n"
                                   "}\0";
```

- 在程序中创建 & 编译

```cpp
int main() {
    // ...

//    4. 使用 glad 初始化 OpenGL 接口(检查)
    if (!gladLoadGLLoader((GLADloadproc) glfwGetProcAddress)) {
        cout << "Failed to initialize GLAD" << endl;
        return -1;
    }

//    5. 顶点着色器
    unsigned int vertexShader = glCreateShader(GL_VERTEX_SHADER);
    glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
    glCompileShader(vertexShader);
    checkShaderCompile(vertexShader);

//    6. 片段着色器
    unsigned int fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
    glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);
    glCompileShader(fragmentShader);
    checkShaderCompile(fragmentShader);

//    7. 着色器程序
    unsigned int shaderProgram = glCreateProgram();
    glAttachShader(shaderProgram, vertexShader);
    glAttachShader(shaderProgram, fragmentShader);
    glLinkProgram(shaderProgram);
    checkProgramCompile(shaderProgram);

    glDeleteShader(vertexShader);
    glDeleteShader(fragmentShader);

    // ...
}
```

Link 成功之后就能够 Delete 释放着色器了

着色器、着色程序的编译结果检查函数如下

```cpp
void checkShaderCompile(unsigned int shader) {
    //    检查着色器编译结果
    int success;
    char infoLog[512];
    glGetShaderiv(shader, GL_COMPILE_STATUS, &success);
    if (!success) {
        glGetShaderInfoLog(shader, 512, NULL, infoLog);
        cout << "ERROR::SHADER::COMPILATION_FAILED\n" << infoLog << endl;
    }
}

void checkProgramCompile(unsigned int program) {
    //    检查着色程序编译结果
    int success;
    char infoLog[512];
    glGetProgramiv(program, GL_LINK_STATUS, &success);
    if (!success) {
        glGetProgramInfoLog(program, 512, NULL, infoLog);
        cout << "ERROR::PROGRAM::COMPILATION_FAILED\n" << infoLog << endl;
    }
}
```

### 1.5 创建数据缓冲对象

输入设备存在各种图形表示结构，需要将其写入缓冲对象才能够交由 OpenGL 进行操作和控制

```cpp
int main() {
    // ...

//    8. 顶点数据
    float vertices[] = {
            0.5f, 0.5f, 0.0f,
            0.5f, -0.5f, 0.0f,
            0.0f, 0.0f, 0.0f,
            -0.5f, -0.5f, 0.0f,
            -0.5f, 0.5f, 0.0f,
    };

    // 顶点索引
    unsigned int indices[] = {
            0, 1, 2,
            2, 3, 4
    };

//    顶点数组对象 VAO、顶点缓冲对象 VBO、EBO
    unsigned int VAO, VBO, EBO;
    glGenVertexArrays(1, &VAO);
    glGenBuffers(1, &VBO);
    glGenBuffers(1, &EBO);

    glBindVertexArray(VAO);
//    三角形顶点数组写入缓冲对象 VBO
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
    glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);

//    设置顶点属性指针
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void *) 0);
    glEnableVertexAttribArray(0);

    glBindBuffer(GL_ARRAY_BUFFER, 0);
    glBindVertexArray(0);

    // ...
}
```

### 1.6 绘制程序 & 退出时的资源清理

现在我们有了绑定好着色器的着色器程序，以及保存了顶点数据的缓冲对象，最后一部分就是进行图形的绘制和每帧的刷新实现了

```cpp
int main() {
    // ...

    while (!glfwWindowShouldClose(window)) {
        processInput(window);

//    刷新屏幕
        glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT);

//    绑定着色器程序
        glUseProgram(shaderProgram);
//    基于缓冲对象绘制图形
        glBindVertexArray(VAO);
//        glDrawArrays(GL_TRIANGLES, 0, 3); // 简单绘制
        glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);

//    获取用户操作 & 缓冲上屏
        glfwPollEvents();
        glfwSwapBuffers(window);
    }

// 清理缓冲对象 & 着色器程序资源
    glDeleteVertexArrays(1, &VAO);
    glDeleteBuffers(1, &VBO);
    glDeleteBuffers(1, &EBO);
    glDeleteProgram(shaderProgram);

    glfwTerminate();
    return 0;
}
```

# 其他资源

## 参考连接

| Title                                                     | Link                                                                                                                                                             |
| --------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| OpenGL 学习实录1: 基于 MacOS + Clion 配置 OpenGL 运行环境 | [https://blog.csdn.net/weixin_44691608/article/details/120806889](https://blog.csdn.net/weixin_44691608/article/details/120806889)                               |
| 你好，三角形 - LearnOpenGL CN                             | [https://learnopengl-cn.github.io/01%20Getting%20started/04%20Hello%20Triangle/](https://learnopengl-cn.github.io/01%20Getting%20started/04%20Hello%20Triangle/) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/others/open_gl/open_gl_basic](https://github.com/superfreeeee/Blog-code/tree/main/others/open_gl/open_gl_basic)
