# OpenGL 学习实录5: 基础光照 & 材质

@[TOC](文章目录)

<!-- TOC -->

- [OpenGL 学习实录5: 基础光照 & 材质](#opengl-学习实录5-基础光照--材质)
- [系列文章](#系列文章)
- [正文](#正文)
  - [1. 光照场景](#1-光照场景)
  - [2. 基础光照](#2-基础光照)
  - [3. 加上材质](#3-加上材质)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 系列文章

- [OpenGL 学习实录1: 基于 MacOS + Clion 配置 OpenGL 运行环境](https://blog.csdn.net/weixin_44691608/article/details/120806889)
- [OpenGL 学习实录2: 基础绘制初试](https://blog.csdn.net/weixin_44691608/article/details/120807296)
- [OpenGL 学习实录3: 深入着色器 - 纹理](https://blog.csdn.net/weixin_44691608/article/details/120816987)
- [OpenGL 学习实录4: 坐标系统 & 摄像机](https://blog.csdn.net/weixin_44691608/article/details/120868046)

# 正文

## 1. 光照场景

- 场景物体：一个立方体 + 一个光源立方体

1. 创建顶点位置数组

```cpp
float vertices[] = {
  // 详细看代码
}
```

2. 构建缓冲对象 & 填充着色器参数(VAO 为立方体、lightVAO 为光源立方体)

```cpp
GLuint VAO, VBO, lightVAO;
glGenVertexArrays(1, &VAO);
glGenVertexArrays(1, &lightVAO);
glGenBuffers(1, &VBO);

glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

//    box VAO
glBindVertexArray(VAO);
glBindBuffer(GL_ARRAY_BUFFER, VBO);
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void *) 0);
glEnableVertexAttribArray(0);

//    light VAO
glBindVertexArray(lightVAO);
glBindBuffer(GL_ARRAY_BUFFER, VBO);
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void *) 0);
glEnableVertexAttribArray(0);
```

3. 创建光源 & 加上立方体/光源颜色

```cpp
glm::vec3 lightPos = glm::vec3(1.2f, 1.0f, 2.0f);
```

```cpp
objectShader.use();
objectShader.setVec3("objectColor", 1.0f, 0.5f, 0.31f);
objectShader.setVec3("lightColor", 1.0f, 1.0f, 1.0f);
```

4. 渲染立方体和光源立方体

```cpp
objectShader.use();
objectShader.setVec3("lightPos", lightPos);

glm::mat4 model = glm::mat4(1.0f);
glm::mat4 view = camera.GetViewMatrix();
glm::mat4 projection = glm::perspective(glm::radians(camera.zoom),
                                    (float) SCR_WIDTH / (float) SCR_HEIGHT,
                                    0.1f,
                                    100.0f);
objectShader.setMat4("model", model);
objectShader.setMat4("view", view);
objectShader.setMat4("projection", projection);

glBindVertexArray(VAO);
glDrawArrays(GL_TRIANGLES, 0, 36);
```

```cpp
lightShader.use();

model = glm::mat4(1.0f);
model = glm::translate(model, lightPos); // reset position
model = glm::scale(model, glm::vec3(0.2f)); // smaller

lightShader.setMat4("model", model);
lightShader.setMat4("view", view);
lightShader.setMat4("projection", projection);

glBindVertexArray(lightVAO);
glDrawArrays(GL_TRIANGLES, 0, 36);
```

5. 修改片段着色器(使用物体颜色与光源颜色合成真实颜色)

```glsl
#version 410 core
out vec4 FragColor;

uniform vec3 objectColor;
uniform vec3 lightColor;

void main() {
    FragColor = vec4(lightColor * objectColor, 1.0);
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/open_gl_learning_5_basic_light_materials_result1.png)

## 2. 基础光照

- 视觉颜色 = 物体本身颜色 + 光源颜色共同作用
- 光照模型：冯氏光照模型(Phong Lighting Model) = 环境光 + 漫反射光 + 镜面光

1. 在片段着色器中分别添加环境光 = 光照基值

```glsl
int main() {
    // 1. 环境光照
    float ambientStrength = 0.1;
    vec3 ambient = ambientStrength * lightColor;
}
```

2. 漫反射光照 = 垂直反射分量强度(主要)；计算漫反射需要片段位置、法向量、光源位置

```glsl
in vec3 FragPos; // 着色片段位置
in vec3 Normal; // 法向量

uniform vec3 lightPos; // 光源位置
uniform vec3 lightColor;

int main() {
    // ...

    vec3 norm = normalize(Normal);// 片段法向量
    vec3 lightDir = normalize(lightPos - FragPos);// 片段 - 光源向量
    float diff = max(dot(norm, lightDir), 0.0);
    vec3 diffuse = diff * lightColor;
```

3. 镜面光照 = 反射给观察者的直线光强度

```glsl
    float specularStrength = 0.5;
    vec3 viewDir = normalize(viewPos - FragPos);
    vec3 reflectDir = reflect(-lightDir, norm);
    float spec = pow(max(dot(viewDir, reflectDir), 0.0), 32);
    vec3 specular = specularStrength * spec * lightColor;
```

4. 将三种光线颜色合成真实的视觉颜色

```glsl
    vec3 result = (ambient + diffuse + specular) * objectColor;
```

5. 与顶点着色器传递片段位置与法向量(使用法线矩阵抵消模型矩阵带来的变形效果)

```glsl
#version 410 core
layout (location = 0) in vec3 aPos;// 顶点坐标
layout (location = 1) in vec3 aNormal;// 顶点法向量

out vec3 FragPos;
out vec3 Normal;

uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;

void main()
{
    FragPos = vec3(model * vec4(aPos, 1.0));
    Normal = mat3(transpose(inverse(model))) * aNormal; // 法线矩阵

    gl_Position = projection * view * model * vec4(aPos, 1.0f);
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/open_gl_learning_5_basic_light_materials_result2.png)

## 3. 加上材质

- 材质实际上就是三种反射光线的强度、色值变化的不同

1. 定义材质矩阵

```glsl
struct Material {
    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
    float shininess;
};

uniform Material material;
```

2. 重新定义光源矩阵，更细致的调整光源强度的影响

```glsl
struct Light {
    vec3 position;

    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
};

uniform Light light;
```

3. 重新计算环境光照

```glsl
    // 1. 环境光照
    vec3 ambient = light.ambient * material.ambient;
```

4. 漫反射光照

```glsl
    vec3 norm = normalize(Normal);// 片段法向量
    vec3 lightDir = normalize(light.position - FragPos);// 片段 - 光源向量
    float diff = max(dot(norm, lightDir), 0.0);
    vec3 diffuse = light.diffuse * (diff * material.diffuse);
```

5. 镜面光照

```glsl
    // 3. 镜面高光
    vec3 viewDir = normalize(viewPos - FragPos);
    vec3 reflectDir = reflect(-lightDir, norm);
    float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);
    vec3 specular = light.specular * (spec * material.specular);
```

6. 合成光线

```glsl
    vec3 result = ambient + diffuse + specular;
    FragColor = vec4(result, 1.0);
```

7. 定义材质和光线强度

```glsl
    // set material
    objectShader.setVec3("material.ambient", 1.0f, 0.5f, 0.31f);
    objectShader.setVec3("material.diffuse", 1.0f, 0.5f, 0.31f);
    objectShader.setVec3("material.specular", 0.5f, 0.5f, 0.5f);
    objectShader.setFloat("material.shininess", 32.0f);

    objectShader.setVec3("light.ambient", 0.2f, 0.2f, 0.2f);
    objectShader.setVec3("light.diffuse", 0.5f, 0.5f, 0.5f);
    objectShader.setVec3("light.specular", 1.0f, 1.0f, 1.0f);
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/open_gl_learning_5_basic_light_materials_result3.png)

# 其他资源

## 参考连接

| Title              | Link                                                                                                                             |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------------- |
| 材质 - LearnOpenGL | [https://learnopengl-cn.github.io/02%20Lighting/03%20Materials/](https://learnopengl-cn.github.io/02%20Lighting/03%20Materials/) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/others/open_gl/open_gl_basic_light_materials](https://github.com/superfreeeee/Blog-code/tree/main/others/open_gl/open_gl_basic_light_materials)
