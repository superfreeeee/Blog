# OpenGL 学习实录3: 深入着色器 - 纹理

@[TOC](文章目录)

<!-- TOC -->

- [OpenGL 学习实录3: 深入着色器 - 纹理](#opengl-学习实录3-深入着色器---纹理)
- [系列文章](#系列文章)
- [正文](#正文)
  - [1. 阶段目标](#1-阶段目标)
  - [2. 着色器封装](#2-着色器封装)
  - [3. 纹理填充](#3-纹理填充)
    - [3.1 纹理基本用法 API](#31-纹理基本用法-api)
    - [3.2 问题加载封装](#32-问题加载封装)
    - [3.4 纹理坐标映射](#34-纹理坐标映射)
    - [3.3 多纹理实现](#33-多纹理实现)
  - [4. 主流程梳理](#4-主流程梳理)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 系列文章

- [OpenGL 学习实录1: 基于 MacOS + Clion 配置 OpenGL 运行环境](https://blog.csdn.net/weixin_44691608/article/details/120806889)
- [OpenGL 学习实录2: 基础绘制初试](https://blog.csdn.net/weixin_44691608/article/details/120807296)

# 正文

## 1. 阶段目标

本篇的实现基于[OpenGL 学习实录2: 基础绘制初试](https://blog.csdn.net/weixin_44691608/article/details/120807296)的基础，实现

- 着色器代码的封装
- 以纹理代替简单颜色值进行填充
- 多纹理管道进行填充

## 2. 着色器封装

之前我们的着色器代码以 C 语言形式的字符串来表示，很丑又很难用如下

```c
const char *vertexShaderSource = "#version 410 core\n"
                                 "layout (location = 0) in vec3 aPos;\n"
                                 "void main() {\n"
                                 "    gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);\n"
                                 "}\0";
```

现在我们可以将字符串的内容写在一个单独的文件内，然后再编写一点简单的代码进行加载和编译

- `vertex.glsl`

```glsl
#version 410 core

layout (location = 0) in vec3 aPos;

void main() {
    gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
}
```

首先我们建立头文件

- `shader.h`

```cpp
//
// Created by 超悠閒 on 2021/10/17.
//

#ifndef OPEN_GL_TEMPLATE_SHADER_H
#define OPEN_GL_TEMPLATE_SHADER_H

#include <glad/glad.h>

#include <string>
#include <fstream>
#include <sstream>
#include <iostream>

using namespace std;

class Shader {
public:
    unsigned int ID;

    Shader(const GLchar *vertexPath, const GLchar *fragmentPath);

    void use();

    void setBool(const string &name, bool value) const;

    void setInt(const string &name, int value) const;

    void setFloat(const string &name, float value) const;

private:
    void checkCompileErrors(unsigned int shader, string type);
};

#endif //OPEN_GL_TEMPLATE_SHADER_H
```

创建一个 Shader 类，用于加载和编译着色器代码；`setXxx` 则是指定着色器运行时的 `uniform` 全局变量的注入

关于实现我们一步步解说

- `shader.cpp`

首先是构造函数，调用构造函数的时候需要传入两个字符串，分别是顶点着色器和片段着色器的文件路径，由于他们通常是两个一组来组成一个着色程序，因此我们一个 Shader 对象便代表着一个着色器程序

```cpp
#include "shader.h"

#include <glad/glad.h>
#include <string>
#include <fstream>
#include <sstream>
#include <iostream>

using namespace std;

Shader::Shader(const GLchar *vertexPath, const GLchar *fragmentPath) {
```

接下来创建关于文件 I/O 的变量，并将文件内容本身读进来

```cpp

    string vertexCode, fragmentCode;
    ifstream vShaderFile, fShaderFile;

    vShaderFile.exceptions(ifstream::failbit | ifstream::badbit);
    fShaderFile.exceptions(ifstream::failbit | ifstream::badbit);

//    读取着色器源文件
    try {
        vShaderFile.open(vertexPath);
        fShaderFile.open(fragmentPath);

        stringstream vShaderStream, fShaderStream;

        vShaderStream << vShaderFile.rdbuf();
        fShaderStream << fShaderFile.rdbuf();

        vertexCode = vShaderStream.str();
        fragmentCode = fShaderStream.str();
    } catch (ifstream::failure &e) {
        cout << "ERROR::SHADER::FILE_NOT_SUCCESSFULLY_READ" << endl;
    }

    const char *vShaderCode = vertexCode.c_str();
    const char *fShaderCode = fragmentCode.c_str();
```

最后一步则是跟之前一样，分别创建顶点着色器和片段着色器并编译，最后链接成一个着色器程序后可以释放两个原始着色器

```cpp
//    创建着色器
    unsigned int vertexShader;
    vertexShader = glCreateShader(GL_VERTEX_SHADER);
    glShaderSource(vertexShader, 1, &vShaderCode, NULL);
    glCompileShader(vertexShader);
    checkCompileErrors(vertexShader, "VERTEX");

    unsigned int fragmentShader;
    fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
    glShaderSource(fragmentShader, 1, &fShaderCode, NULL);
    glCompileShader(fragmentShader);
    checkCompileErrors(fragmentShader, "FRAGMENT");

//    创建着色器程序
    ID = glCreateProgram();
    glAttachShader(ID, vertexShader);
    glAttachShader(ID, fragmentShader);
    glLinkProgram(ID);
    checkCompileErrors(ID, "PROGRAM");
    glDeleteShader(vertexShader);
    glDeleteShader(fragmentShader);
}
```

使用的时候我们就可以如下

```cpp
    Shader ourShader("../shader/vertex.glsl", "../shader/fragment.glsl");
```

`use` 方法其实就是 `glUseProgram` 方法的别名

```cpp
void Shader::use() {
    glUseProgram(ID);
}
```

`setXxx` 方法则是使用 `glUniform1x` API 来注入着色器里面定义的 `uniform` 类型变量

```cpp
void Shader::setBool(const string &name, bool value) const {
    glUniform1i(glGetUniformLocation(ID, name.c_str()), (int) value);
}

void Shader::setInt(const string &name, int value) const {
    glUniform1i(glGetUniformLocation(ID, name.c_str()), value);
}

void Shader::setFloat(const string &name, float value) const {
    glUniform1f(glGetUniformLocation(ID, name.c_str()), value);
}
```

## 3. 纹理填充

我们将着色器代码与 C++ 代码进行分离之后整个程序的逻辑就变得更清晰了，接下来是本篇的重点：对顶点平面使用纹理(也就是图片素材包)，将纹理素材的像素点映射到目标图形上进行填充

### 3.1 纹理基本用法 API

与着色器、缓冲对象一样，我们总是需要使用 `glGenXxx` 方法创建 OpenGL 的对象。对于纹理对象我们可以这样创建

```cpp
GLuint texture;
glGenTextures(1, &texture);
```

接下来我们要绑定(也就是选中)该纹理对象并进行配置

```cpp
glBindTexture(GL_TEXTURE_2D, texture);

glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

`glTexParameteri` 方法指定纹理对象的参数，我们要配置的是二维纹理对象的属性使用 `GL_TEXTURE_2D`，`GL_TEXTURE_WRAP_S`、`GL_TEXTURE_WRAP_S` 就是指 x 方向和 y 方向的纹理绘制方式，使用 `GL_REPEAT` 表示重复填充，`GL_TEXTURE_MIN_FILTER`、`GL_TEXTURE_MAG_FILTER` 则是指后续进行缩放的时候关于纹理采样的时候的像素采样算法

参数设定完毕之后我们需要加在纹理图片并构建纹理对象（我们使用 `stb_image.h` 库进行纹理图片的加载 [nothings/stb - Github](https://github.com/nothings/stb/blob/master/stb_image.h)）

下载 `stb_image.h`，文件后，再额外创建一个 `stb_image.cpp` 作为配置文件

- `stb_image.cpp`

```cpp
#define STB_IMAGE_IMPLEMENTATION
#include "stb_image.h"
```

最后我们只要使用 `stbi_load` 加载原始代码，使用 `glTexImage2D + glGenerateMipmap` 创建纹理对象，并使用 `stbi_image_free` 释放素材图片资源即可

```cpp
int width, height, nrChannels;
stbi_set_flip_vertically_on_load(true);
unsigned char *data;
data = stbi_load(filename, &width, &height, &nrChannels, 0);
if (data) {
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, data);
    glGenerateMipmap(GL_TEXTURE_2D);
} else {
    cout << "Failed to load texture: " << filename << endl;
}
stbi_image_free(data);
```

### 3.2 问题加载封装

纹理加载本身也是一个非常重复的过程，我们选择封装一下

首先建立一个头文件

- `texture.h`

```cpp
//
// Created by 超悠閒 on 2021/10/17.
//

#ifndef OPEN_GL_TEMPLATE_TEXTURE_H
#define OPEN_GL_TEMPLATE_TEXTURE_H

#include <glad/glad.h>
#include "shader.h"

void loadTexture(const char *filename, GLuint *texture, int hasAlpha);

#endif //OPEN_GL_TEMPLATE_TEXTURE_H
```

接下来是实现，也就是将刚才的基础 API 封装成一个方法

- `texture.cpp`

```cpp
//
// Created by 超悠閒 on 2021/10/17.
//

#include "texture.h"
#include "stb_image.h"
#include <glad/glad.h>

void loadTexture(const char *filename, GLuint *texture, int hasAlpha) {
    int width, height, nrChannels;
    stbi_set_flip_vertically_on_load(true);
    unsigned char *data;

//    初始化纹理对象
    glGenTextures(1, texture);
    glBindTexture(GL_TEXTURE_2D, *texture);

//    设置纹理对象参数
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);

//    加载纹理图片
    data = stbi_load(filename, &width, &height, &nrChannels, 0);
    if (data) {
        GLenum colorFormat = hasAlpha ? GL_RGBA : GL_RGB;
        glTexImage2D(GL_TEXTURE_2D, 0, colorFormat, width, height, 0, colorFormat, GL_UNSIGNED_BYTE, data);
        glGenerateMipmap(GL_TEXTURE_2D);
    } else {
        cout << "Failed to load texture: " << filename << endl;
    }

//   释放纹理数据资源
    stbi_image_free(data);

    glBindTexture(GL_TEXTURE_2D, 0); // unbind texture
}
```

之后我们就可以这样来创建纹理对象

```cpp
GLuint texture1;
loadTexture("../static/container.jpeg", &texture1, false);
```

### 3.4 纹理坐标映射

加载纹理图片还没完，我们需要将它传给片段着色器进行使用，这时候我们可以选择使用 `uniform` 的特性，将 texture 写入一个 OpenGL 全局变量(`uniform` 修饰的变量)，这样一来片段着色器就能够使用了

```cpp
// 绑定一次就好
ourShader.use();
ourShader.setInt("texture1", 0);

// 每次渲染前调用
glActiveTexture(GL_TEXTURE0);
glBindTexture(GL_TEXTURE_2D, texture1);
```

首先先选用当前着色器 `ourShader.use()`，然后绑定名为 `texture1` 的 uniform 变量为编号 0 的纹理单元；接下来每次绘制时先使用 `glActiveTexture` 激活编号为 0 的纹理单元，然后告诉 OpenGL 要使用的纹理对象为我们刚刚创建的纹理对象 `texture1`(注意一个是纹理对象 `texture1`，一个是片段着色器里面的全局变量 `texture1`)

接下来在片段着色器里面就可以使用名为 `texture1`，并绑定某个纹理对象的变量了

- `fragment.glsl`

```glsl
#version 410 core
in vec3 ourColor;
in vec2 TexCoord;

out vec4 FragColor;

uniform sampler2D texture1;

void main() {
    FragColor = texture(texture1, TexCoord);
}
```

`texture` 函数使用全局变量 `texture1` 指向的纹理对象来进行像素映射

### 3.3 多纹理实现

前面我们提到了关于纹理单元的选用，OpenGL 保证至少有 16 个纹理单元能用(`GL_TEXTURE0~15`)，多纹理对象实际上就是按 3.3 步骤创建多个纹理对象，然后再重复 3.4 绑定到多个全局变量上即可

- 创建纹理对象 & 绑定纹理单元编号

```cpp
GLuint texture1, texture2;
loadTexture("../static/container.jpeg", &texture1, false);
loadTexture("../static/awesomeface.png", &texture2, true);

ourShader.use();
ourShader.setInt("texture1", 0);
ourShader.setInt("texture2", 1);
```

- 每次渲染时传入对应的纹理对象

```cpp
while (!glfwWindowShouldClose(window)) {
    // ...

    glActiveTexture(GL_TEXTURE0);
    glBindTexture(GL_TEXTURE_2D, texture1);
    glActiveTexture(GL_TEXTURE1);
    glBindTexture(GL_TEXTURE_2D, texture2);

    // ...
}
```

- 最后在片段着色器同时使用两个纹理对象进行渲染

```glsl
#version 410 core
in vec3 ourColor;
in vec2 TexCoord;

out vec4 FragColor;

uniform sampler2D texture1;
uniform sampler2D texture2;

void main() {
    FragColor = mix(texture(texture1, TexCoord), texture(texture2, TexCoord), 0.2);
}
```

`mix` 能将两个纹理对象的颜色按比例进行混合，`0.2` 表示按 `0.8 * texture1 + 0.2 * texture2` 的比例进行混合

## 4. 主流程梳理

最后来一个大总汇，梳理主流程上的各个步骤

- `main.cpp`

首先是 GLFW 的初始化、窗口创建等，都没啥好说了

```cpp
#include <glad/glad.h>
#include <GLFW/glfw3.h>
#include <iostream>
#include "shader.h"
#include "stb_image.h"
#include "texture.h"

using namespace std;

void framebuffer_size_callback(GLFWwindow *window, int width, int height);

void processInput(GLFWwindow *window);

int main() {
    if (glfwInit() == GLFW_FALSE) {
        cout << "Fail to initialize GLFW" << endl;
        return -1;
    }

    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 4);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 1);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

#ifdef __APPLE__
    glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);
#endif

    GLFWwindow *window = glfwCreateWindow(800, 600, "LearnOpenGL", NULL, NULL);
    if (window == NULL) {
        cout << "Failed to create GLFW window" << endl;
        glfwTerminate();
        return -1;
    }
    glfwMakeContextCurrent(window);
    glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);
```

接下来是使用刚才在 2. 封装的着色器程序来加载着色器

```cpp
    if (!gladLoadGLLoader((GLADloadproc) glfwGetProcAddress)) {
        cout << "Failed to initialize GLAD" << endl;
        return -1;
    }

    Shader ourShader("../shader/vertex.glsl", "../shader/fragment.glsl");
```

有了着色器就可以着手建立顶点对象和索引对象了

```cpp
    float vertices[] = {
            // position         // colors           // texture1
            1.0f,  1.0f, 0.0f,  1.0f, 0.0f, 0.0f,   1.0f, 1.0f, // 右上
            1.0f, -1.0f, 0.0f,  0.0f, 1.0f, 0.0f,   1.0f, 0.0f, // 右下
           -1.0f, -1.0f, 0.0f,  0.0f, 0.0f, 1.0f,   0.0f, 0.0f, // 左下
           -1.0f,  1.0f, 0.0f,  1.0f, 1.0f, 0.0f,   0.0f, 1.0f, // 左上
    };

    unsigned int indices[] = {
            0, 1, 3,
            1, 2, 3,
    };

    unsigned int VAO, VBO, EBO;
    glGenVertexArrays(1, &VAO);
    glGenBuffers(1, &VBO);
    glGenBuffers(1, &EBO);

    glBindVertexArray(VAO);
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
    glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);
```

这里要注意的是，我们的顶点对象结构变成每 8 个 float 一个顶点，因此第五个参数表示步长都是 `8 * sizeof(float)`
- 第一个参数表示 position，分别是 1、2、3
- 第二个参数表示数字个数，也就是向量长度：顶点坐标和颜色都是三维，纹理坐标则为二维
- 第三个参数为数据类型，都为 float
- 第四个参数表示是否标准化，现在还没做到
- 第五个参数表示点与点的间隔即步长，为每 8 个 float 一个点
- 第六个参数表示起始偏移量，顶点坐标偏移量为 0，颜色为 3，纹理坐标为 6

```cpp
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 8 * sizeof(float), (void *) 0);
    glEnableVertexAttribArray(0);
    glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 8 * sizeof(float), (void *) (3 * sizeof(float)));
    glEnableVertexAttribArray(1);
    glVertexAttribPointer(2, 2, GL_FLOAT, GL_FALSE, 8 * sizeof(float), (void *) (6 * sizeof(float)));
    glEnableVertexAttribArray(2);
```

完成顶点缓冲对象的创建后就是纹理对象的创建和加载资源，并绑定一下片段着色器要用的 `uniform` 变量

```cpp
    GLuint texture1, texture2;
    loadTexture("../static/container.jpeg", &texture1, false);
    loadTexture("../static/awesomeface.png", &texture2, true);

    ourShader.use();
    ourShader.setInt("texture1", 0);
    ourShader.setInt("texture2", 1);
```

接下来在渲染的流程中，我们在原本调用 `ourShader.use + glDrawElements` 进行绘制之前，要加入纹理对象的绑定

```cpp
    while (!glfwWindowShouldClose(window)) {
        processInput(window);

        glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT);

//    将纹理对象绑定到指定的 uniform 全局变量当中
        glActiveTexture(GL_TEXTURE0);
        glBindTexture(GL_TEXTURE_2D, texture1);
        glActiveTexture(GL_TEXTURE1);
        glBindTexture(GL_TEXTURE_2D, texture2);

        ourShader.use();
        glBindVertexArray(VAO);
        glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);

        glfwSwapBuffers(window);
        glfwPollEvents();
    }

    glDeleteVertexArrays(1, &VAO);
    glDeleteBuffers(1, &VBO);
    glDeleteBuffers(1, &EBO);

    glfwTerminate();
    return 0;
}
```

大功告成啦，秀个图爽一下hh

![](https://picures.oss-cn-beijing.aliyuncs.com/img/open_gl_learning_3_shader_texture_result.png)

# 其他资源

## 参考连接

| Title                 | Link                                                                                                                                             |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| 纹理 - LearnOpenGL CN | [https://learnopengl-cn.github.io/01%20Getting%20started/06%20Textures/](https://learnopengl-cn.github.io/01%20Getting%20started/06%20Textures/) |
| nothings/stb - Github | [https://github.com/nothings/stb/blob/master/stb_image.h](https://github.com/nothings/stb/blob/master/stb_image.h)                               |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/others/open_gl/open_gl_shader](https://github.com/superfreeeee/Blog-code/tree/main/others/open_gl/open_gl_shader)
