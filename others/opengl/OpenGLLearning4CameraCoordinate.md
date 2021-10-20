# OpenGL 学习实录4: 坐标系统 & 摄像机

@[TOC](文章目录)

<!-- TOC -->

- [OpenGL 学习实录4: 坐标系统 & 摄像机](#opengl-学习实录4-坐标系统--摄像机)
- [系列文章](#系列文章)
- [正文](#正文)
  - [1. 坐标系统变换](#1-坐标系统变换)
  - [2. 矩阵运算库 glm](#2-矩阵运算库-glm)
  - [3. 构建摄像机](#3-构建摄像机)
    - [3.1 更多盒子(模型矩阵)](#31-更多盒子模型矩阵)
    - [3.2 摄像机封装](#32-摄像机封装)
      - [3.2.0 属性解析](#320-属性解析)
      - [3.2.1 摄像机移动(键盘响应)](#321-摄像机移动键盘响应)
      - [3.2.2 视角旋转(滑鼠响应)](#322-视角旋转滑鼠响应)
      - [3.2.3 视角缩放(滑鼠滚轮响应)](#323-视角缩放滑鼠滚轮响应)
      - [3.2.4 构建观察矩阵](#324-构建观察矩阵)
    - [3.3 投影矩阵裁切可见空间](#33-投影矩阵裁切可见空间)
    - [3.4 更新顶点着色器](#34-更新顶点着色器)
  - [4. 最终效果](#4-最终效果)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 系列文章

- [OpenGL 学习实录1: 基于 MacOS + Clion 配置 OpenGL 运行环境](https://blog.csdn.net/weixin_44691608/article/details/120806889)
- [OpenGL 学习实录2: 基础绘制初试](https://blog.csdn.net/weixin_44691608/article/details/120807296)
- [OpenGL 学习实录3: 深入着色器 - 纹理](https://blog.csdn.net/weixin_44691608/article/details/120816987)

# 正文

前一篇我们制作了一个盒子，并涂上两张图片作为纹理，本篇将要介绍的则是绘制 3D 场景时通用的坐标系统变换方法，构建更多的盒子，并利用坐标变换来模拟摄像机的运转、也就是模拟用户视角的移动

## 1. 坐标系统变换

首先是坐标系统，默认的坐标系统为
- x 方向向`右`为正
- y 方向向`上`为正
- z 方向指向`屏幕外`为正

但是我们要构建 3D 场景的时候就需要对整个模型进行映射，坐标系统分为以下几个

- `局部空间 Local Space`(物体空间 Object Space)
  - 描述物体本身坐标系，通常以 `(0,0,0)` 为原点
- `世界空间 World Space`
  - 模拟物体在 3D 世界中的坐标，也就是改把我们的物体放到该放的地方(偏移)，进行适当的变形(旋转、缩放)
  - `世界空间 = 模型矩阵 Model Matrix * Local Space`
- `观察空间 View Space`(摄像机空间 Camera Space / 视觉空间 Eye Space)
  - 用户观察世界的角度，也就是所谓的摄像机看向世界的角度(透过对整个场景进行变形来模拟摄像机的运行)
  - `观察空间 = 观察矩阵 View Matrix * World Space`
- `剪裁空间 Clip Space`
  - 用户可见的视野范围，也就是指定用户可见的最近、最远距离，将剪裁空间以外的物体舍弃
  - `剪裁空间 = 投影矩阵 Projection Matrix * View Space`
- `屏幕空间 Screen Space`
  - 最后屏幕空间就是投影到屏幕上的样子，这个 OpenGL 会自动帮我们完成

## 2. 矩阵运算库 glm

前面提到一堆坐标的变换，而这些坐标都是一些向量，变换都是一堆矩阵，我们将使用的向量/矩阵等运算的数学库 `glm`，下面是我们后续会用到的几个常见函数

- 向量 `glm::vecX`
- 矩阵 `glm::matX`
- 平移 `glm::translate(matrix, vec)`
- 旋转 `glm::rotate(matrix, radian, vec)`
- 缩放 `glm::scale(matrix, vec)`
- 正射投影 `glm::ortho(sx, tx, sy, ty, sz, tz)`
- 透视投影 `glm::perspective(fov, w/h, sz, tz);`

## 3. 构建摄像机

接下来我们会经历几个阶段

- 定义物体模型，也就是每个方块的原始坐标、颜色等
- 定义物体位置，使用`模型矩阵`将每个物体映射到目标位置 = 世界空间
- 接下来模拟摄像机的运作，使用`观察矩阵`将整个场景进行变形 = 观察空间
- 最后根据观察空间，我们使用`投影矩阵`对观察空间的物体进行裁切 = 剪裁空间
- 接下来 OpenGL 会负责将剪裁空间内的物体绘制到我们的 2D 屏幕上啦

### 3.1 更多盒子(模型矩阵)

首先前面我们已经定义过一个基础的盒子对象了，这时候我们扩展一下，变出十个盒子

- 顶点坐标 & 索引数组

```cpp
float vertices[] = {
        // position          // texture
        -0.5f, -0.5f, -0.5f, 0.0f, 0.0f,
        0.5f, -0.5f, -0.5f, 1.0f, 0.0f,
        0.5f, 0.5f, -0.5f, 1.0f, 1.0f,
        -0.5f, 0.5f, -0.5f, 0.0f, 1.0f,

        -0.5f, -0.5f, 0.5f, 0.0f, 0.0f,
        0.5f, -0.5f, 0.5f, 1.0f, 0.0f,
        0.5f, 0.5f, 0.5f, 1.0f, 1.0f,
        -0.5f, 0.5f, 0.5f, 0.0f, 1.0f,

        -0.5f, 0.5f, 0.5f, 1.0f, 0.0f,
        -0.5f, 0.5f, -0.5f, 1.0f, 1.0f,
        -0.5f, -0.5f, -0.5f, 0.0f, 1.0f,
        -0.5f, -0.5f, 0.5f, 0.0f, 0.0f,

        0.5f, 0.5f, 0.5f, 1.0f, 0.0f,
        0.5f, 0.5f, -0.5f, 1.0f, 1.0f,
        0.5f, -0.5f, -0.5f, 0.0f, 1.0f,
        0.5f, -0.5f, 0.5f, 0.0f, 0.0f,

        -0.5f, -0.5f, -0.5f, 0.0f, 1.0f,
        0.5f, -0.5f, -0.5f, 1.0f, 1.0f,
        0.5f, -0.5f, 0.5f, 1.0f, 0.0f,
        -0.5f, -0.5f, 0.5f, 0.0f, 0.0f,

        -0.5f, 0.5f, -0.5f, 0.0f, 1.0f,
        0.5f, 0.5f, -0.5f, 1.0f, 1.0f,
        0.5f, 0.5f, 0.5f, 1.0f, 0.0f,
        -0.5f, 0.5f, 0.5f, 0.0f, 0.0f,
};

unsigned int indices[] = {
        0, 1, 2,
        2, 3, 0,

        4, 5, 6,
        6, 7, 4,

        8, 9, 10,
        10, 11, 8,

        12, 13, 14,
        14, 15, 12,

        16, 17, 18,
        18, 19, 16,

        20, 21, 22,
        22, 23, 20,
};
```

- 构建缓冲对象

```cpp
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

- 循环内渲染盒子对象

上面的所有顶点和索引只能构建一个盒子，接下来我们定义十个盒子，并分别指定每个的实际坐标偏移

```cpp
glm::vec3 cubePositions[] = {
        glm::vec3(0.0f, 0.0f, 0.0f),
        glm::vec3(2.0f, 5.0f, -15.0f),
        glm::vec3(-1.5f, -2.2f, -2.5f),
        glm::vec3(-3.8f, -2.0f, -12.3f),
        glm::vec3(2.4f, -0.4f, -3.5f),
        glm::vec3(-1.7f, 3.0f, -7.5f),
        glm::vec3(1.3f, -2.0f, -2.5f),
        glm::vec3(1.5f, 2.0f, -2.5f),
        glm::vec3(1.5f, 0.2f, -1.5f),
        glm::vec3(-1.3f, 1.0f, -1.5f)
};
```

然后渲染时构建模型矩阵进行映射

```cpp
for (unsigned int i = 0; i < 10; i++) {
    glm::mat4 model = glm::mat4(1.0f);
    // 偏移
    model = glm::translate(model, cubePositions[i]);

    float angle = 20.0f * i; // 起始旋转角度 20 * i
    model = glm::rotate(model, glm::radians(angle), glm::vec3(1.0f, 0.3f, 0.5f));

    GLint modelLoc = glGetUniformLocation(ourShader.ID, "model");
    glUniformMatrix4fv(modelLoc, 1, GL_FALSE, glm::value_ptr(model));

    glDrawElements(GL_TRIANGLES, 36, GL_UNSIGNED_INT, 0);
}
```

这样应该就能看到画面上有 10 个盒子了

### 3.2 摄像机封装

完成了局部空间到世界空间的映射，也就是初步构建好我们的世界空间场景了，接下来要进行观察空间的映射，也就是创造一个摄像机

这里我们写的是一种 FPS 摄像机，存在一些限制，代码里面会提到

首先构建一个 `camera.h` 头文件

- `camera.h`

```cpp
//
// Created by 超悠閒 on 2021/10/20.
//

#ifndef OPEN_GL_CAMERA_COORDINATE_CAMERA_H
#define OPEN_GL_CAMERA_COORDINATE_CAMERA_H

#include <glad/glad.h>
#include <glm/glm.hpp>
#include <glm/gtc/matrix_transform.hpp>

enum Camera_Movement {
    FORWARD,
    BACKWARD,
    LEFT,
    RIGHT
};

// Default camera values
const float DEFAULT_YAW = -90.0f;
const float DEFAULT_PITCH = 0.0f;
const float DEFAULT_SPEED = 5.0f;
const float DEFAULT_SENSITIVITY = 0.1f;
const float DEFAULT_ZOOM = 45.0f;

class Camera {
public:
    glm::vec3 position; // 相机位置
    glm::vec3 front;    // 相机前景中心
    glm::vec3 up;       // 相机上向量
    glm::vec3 right;    // 相机右向量
    glm::vec3 worldUp;  // 世界空间上向量

    float yaw;   // 水平旋转角
    float pitch; // 镜头仰角
    float moveSpeed;        // 移动速度
    float mouseSensitivity; // 鼠标灵敏度
    float zoom;  // 镜头缩放

    Camera(glm::vec3 position = glm::vec3(0.0f, 0.0f, 0.0f),
           glm::vec3 up = glm::vec3(0.0f, 1.0f, 0.0f),
           float yaw = DEFAULT_YAW,
           float pitch = DEFAULT_PITCH);

    Camera(float posX, float posY, float posZ, float upX, float upY, float upZ, float yaw, float pitch);

    glm::mat4 GetViewMatrix();

    void ProcessKeyboard(Camera_Movement direction, float deltaTime);

    void ProcessMouseMovement(float xoffset, float yoffset, GLboolean constrainPitch = true);

    void ProcessMouseScroll(float yoffset);

private:
    void updateCameraVectors();

};

#endif //OPEN_GL_CAMERA_COORDINATE_CAMERA_H
```

后面则是这个文件的实现

#### 3.2.0 属性解析

在开始之前我们先来仔细看一下摄像机类都有哪些属性要用

```cpp
glm::vec3 position; // 相机位置
glm::vec3 front;    // 相机前景中心
glm::vec3 up;       // 相机上向量
glm::vec3 right;    // 相机右向量
glm::vec3 worldUp;  // 世界空间上向量

float yaw;   // 水平旋转角
float pitch; // 镜头仰角
float moveSpeed;        // 移动速度
float mouseSensitivity; // 鼠标灵敏度
float zoom;  // 镜头缩放
```

首先我们会使用一个摄像机的坐标 `position`，然后定义一个摄像机前方单位长度的坐标 `front`，来定位摄像机的朝向，接下来我们可以根据世界空间的向上向量 `worldUp` 与相机法线向量 `position - front` 叉乘得到右向量`right`，最后再用方向向量与右向量叉乘得到上向量，透过使用这三个互相垂直的向量，我们就能够表现任意角度的视角

而下面几个变量则是表示一些基础量，并且后续将根据用户操作来改变值

#### 3.2.1 摄像机移动(键盘响应)

第一种操作是摄像机本身进行前后左右的移动

```cpp
void Camera::ProcessKeyboard(Camera_Movement direction, float deltaTime) {
    float velocity = this->moveSpeed * deltaTime;
    if (direction == FORWARD) {
        this->position += this->front * velocity;
    } else if (direction == BACKWARD) {
        this->position -= this->front * velocity;
    } else if (direction == LEFT) {
        this->position -= this->right * velocity;
    } else if (direction == RIGHT) {
        this->position += this->right * velocity;
    }
}
```

本质上就是根据移动方向改变相机位置 `position`

#### 3.2.2 视角旋转(滑鼠响应)

第二种是视角的旋转，我们透过检查滑鼠的移动来模拟，上下移动改变仰角，左右移动改变视角

```cpp
void Camera::ProcessMouseMovement(float xoffset, float yoffset, GLboolean constrainPitch) {
    xoffset *= this->mouseSensitivity;
    yoffset *= this->mouseSensitivity;

    this->yaw += xoffset;
    this->pitch += yoffset;

    // make sure that when pitch is out of bounds, screen doesn't get flipped
    if (constrainPitch) {
        if (this->pitch > 89.0f) {
            this->pitch = 89.0f;
        } else if (this->pitch < -89.0f) {
            this->pitch = -89.0f;
        }
    }

    // update Front, Right and Up Vectors using the updated Euler angles
    this->updateCameraVectors();
}
```

- `yaw` 表示垂直仰角
- `pitch` 表示水平视角

由这两个向量我们可以计算出更新后的方向向量

```cpp
void Camera::updateCameraVectors() {
    glm::vec3 front;
    front.x = cos(glm::radians(this->yaw)) * cos(glm::radians(this->pitch));
    front.y = sin(glm::radians(this->pitch));
    front.z = sin(glm::radians(this->yaw)) * cos(glm::radians(this->pitch));
    this->front = glm::normalize(front);

    this->right = glm::normalize(glm::cross(this->front, this->worldUp));
    this->up = glm::normalize(glm::cross(this->right, this->front));
}
```

#### 3.2.3 视角缩放(滑鼠滚轮响应)

视角的缩放则要放到投影矩阵的部分才会用到，这里仅仅只是记录用户的操作

```cpp
void Camera::ProcessMouseScroll(float yoffset) {
    float zoom = this->zoom -= (float) yoffset;
    if (zoom < 1.0f) {
        this->zoom = 1.0f;
    } else if (zoom > 45.0f) {
        this->zoom = 45.0f;
    }
}
```

#### 3.2.4 构建观察矩阵

到此观察矩阵所需的要素都备齐了，接下来使用 `lookAt` 来生成观察矩阵

```cpp
glm::mat4 Camera::GetViewMatrix() {
    return glm::lookAt(this->position, this->position + this->front, this->up);
}
```

并且修改一下主入口的代码

- `/src/main.cpp`

首先构建一个摄像机

```cpp
// camera
Camera camera(glm::vec3(0.0f, 0.0f, 3.0f));
float lastX = SCR_WIDTH / 2.0f;
float lastY = SCR_HEIGHT / 2.0f;
bool firstMouse = true;
```

然后加上几个操作监听函数

```cpp
int main() {
    // ...
    
    glfwSetInputMode(window, GLFW_CURSOR, GLFW_CURSOR_DISABLED); // 锁定滑鼠
    glfwSetCursorPosCallback(window, mouse_callback);
    glfwSetScrollCallback(window, scroll_callback);

    // ...


    while (!glfwWindowShouldClose(window)) {
        processInput(window);

```

先使用 `glfwSetInputMode` 隐藏用户鼠标，鼠标移动使用 `glfwSetCursorPosCallback` 设置监听函数，滚轮使用 `glfwSetScrollCallback` 响应，用户输入跟之前一样使用 `processInput` 进行响应

对于鼠标移动，我们计算好偏移量之后更新摄像机

```cpp
void mouse_callback(GLFWwindow *window, double xPos, double yPos) {
    if (firstMouse) {
        lastX = xPos;
        lastY = yPos;
        firstMouse = false;
    }

    float xOffset = xPos - lastX;
    float yOffset = lastY - yPos;
    lastX = xPos;
    lastY = yPos;

    camera.ProcessMouseMovement(xOffset, yOffset);
}
```

缩放效果一样

```cpp
void scroll_callback(GLFWwindow *window, double xoffset, double yoffset) {
    camera.ProcessMouseScroll(yoffset);
}
```

最后在渲染过程中创建使用`观察矩阵`，将世界空间映射到观察空间当中

```cpp
        glm::mat4 view = camera.GetViewMatrix();
        GLint viewLoc = glGetUniformLocation(ourShader.ID, "view");
        glUniformMatrix4fv(viewLoc, 1, GL_FALSE, glm::value_ptr(view));
```

### 3.3 投影矩阵裁切可见空间

最后一部分是前面我们使用滚轮操作和摄像机记录了用户的缩放行为，最后我们只需要构建一个投影矩阵就能够看到正确的剪裁空间了

```cpp
        glm::mat4 projection = glm::mat4(1.0f);
        projection = glm::perspective(glm::radians(camera.zoom), (float) SCR_WIDTH / (float) SCR_HEIGHT, 0.5f, 100.0f);
        GLint projectionLoc = glGetUniformLocation(ourShader.ID, "projection");
        glUniformMatrix4fv(projectionLoc, 1, GL_FALSE, glm::value_ptr(projection));
```

### 3.4 更新顶点着色器

模型搭建好了，坐标系统映射也全部准备好了，最后就在顶点着色器里面加入一半常见的通用坐标变换系统啦

- `vertex.glsl`

```glsl
#version 410 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec2 aTexCoord;

out vec2 TexCoord;

uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;

void main()
{
    gl_Position = projection * view * model * vec4(aPos, 1.0f);
    TexCoord = aTexCoord;
}
```

## 4. 最终效果

最终效果我们可以使用 `WSAD` 进行前后左右移动，使用鼠标移动模拟视角的旋转，最后使用鼠标的滚轮模拟缩放效果，给出几个效果图如下

- 从正面看

![](https://picures.oss-cn-beijing.aliyuncs.com/img/open_gl_learning_4_camera_coordinate_result1.png)

- 从右侧看

![](https://picures.oss-cn-beijing.aliyuncs.com/img/open_gl_learning_4_camera_coordinate_result2.png)

- 从背面看

![](https://picures.oss-cn-beijing.aliyuncs.com/img/open_gl_learning_4_camera_coordinate_result3.png)

# 其他资源

## 参考连接

| Title                   | Link                                                                                                                                         |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| 摄像机 - LearnOpenGL CN | [https://learnopengl-cn.github.io/01%20Getting%20started/09%20Camera/](https://learnopengl-cn.github.io/01%20Getting%20started/09%20Camera/) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/others/open_gl/open_gl_camera_coordinate](https://github.com/superfreeeee/Blog-code/tree/main/others/open_gl/open_gl_camera_coordinate)
