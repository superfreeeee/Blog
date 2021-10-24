# OpenGL 学习实录6: 进阶光照(光照纹理 & 投光物)

@[TOC](文章目录)

<!-- TOC -->

- [OpenGL 学习实录6: 进阶光照(光照纹理 & 投光物)](#opengl-学习实录6-进阶光照光照纹理--投光物)
- [系列文章](#系列文章)
- [正文](#正文)
  - [1. 材质纹理](#1-材质纹理)
  - [2. 多种光源](#2-多种光源)
    - [2.1 定向光源](#21-定向光源)
    - [2.2 点光源](#22-点光源)
    - [2.3 聚光源](#23-聚光源)
    - [2.4 合成光线](#24-合成光线)
  - [3. 布置场景 & 结果](#3-布置场景--结果)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 系列文章

- [OpenGL 学习实录1: 基于 MacOS + Clion 配置 OpenGL 运行环境](https://blog.csdn.net/weixin_44691608/article/details/120806889)
- [OpenGL 学习实录2: 基础绘制初试](https://blog.csdn.net/weixin_44691608/article/details/120807296)
- [OpenGL 学习实录3: 深入着色器 - 纹理](https://blog.csdn.net/weixin_44691608/article/details/120816987)
- [OpenGL 学习实录4: 坐标系统 & 摄像机](https://blog.csdn.net/weixin_44691608/article/details/120868046)
- [OpenGL 学习实录5: 基础光照 & 材质](https://blog.csdn.net/weixin_44691608/article/details/120914725)

# 正文

## 1. 材质纹理

前一篇我们学会了根据参数调整光源反射来表现材质，现在我们更进一步加上纹理，并在纹理上表现出材质效果

我们只需要把材质对象关于漫反射和高亮反射的属性改成纹理对象就可以了

```glsl
struct Material {
    sampler2D diffuse;// 光照纹理
    sampler2D specular;// 镜面纹理
    float shininess;
};
in vec2 TexCoords;
```

接下来计算三种光线分量的时候要乘上纹理对象

```glsl
vec3 ambient = light.ambient * vec3(texture(material.diffuse, TexCoords));
vec3 diffuse = light.diffuse * diff * vec3(texture(material.diffuse, TexCoords));
vec3 specular = light.specular * spec * vec3(texture(material.specular, TexCoords));
```

## 2. 多种光源

上一节[OpenGL 学习实录5: 基础光照 & 材质](https://blog.csdn.net/weixin_44691608/article/details/120914725)使用的光源为点光源系统，现实场景还存在多种光照系统(定向光ex太阳、聚光ex手电筒)

### 2.1 定向光源

定向光只需要方向与三个参数

```glsl
struct DirLight {
    vec3 direction;

    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
};
uniform DirLight dirLight;// 定向光
```

接下来将定向光的计算封装成一个方法

```glsl
vec3 CalcDirLight(DirLight dirLight, vec3 normal, vec3 viewDir);
```

首先计算光照方向

```glsl
vec3 CalcDirLight(DirLight light, vec3 normal, vec3 viewDir) {
    vec3 lightDir = normalize(-light.direction);
```

计算漫反射和镜面光的分量

```glsl
    // 漫反射
    float diff = max(dot(normal, lightDir), 0.0);

    // 镜面光
    vec3 reflectDir = reflect(-lightDir, normal);
    float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);
```

最后得出最终光源颜色

```glsl
    vec3 ambient = light.ambient * vec3(texture(material.diffuse, TexCoords));
    vec3 diffuse = light.diffuse * diff * vec3(texture(material.diffuse, TexCoords));
    vec3 specular = light.specular * spec * vec3(texture(material.specular, TexCoords));

    return (ambient + diffuse + specular);
}
```

### 2.2 点光源

点光源也是类似

```glsl
struct PointLight {
    vec3 position;

    float constant;
    float linear;
    float quadratic;

    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
};
uniform PointLight pointLights[NR_POINT_LIGHTS];// 点光源

vec3 CalcPointLight(PointLight light, vec3 normal, vec3 fragPos, vec3 viewDir);
```

首先是两种光线的分量计算

```glsl
vec3 CalcPointLight(PointLight light, vec3 normal, vec3 fragPos, vec3 viewDir) {
    vec3 lightDir = normalize(light.position - fragPos);

    // 漫反射
    float diff = max(dot(normal, lightDir), 0.0);

    // 镜面光
    vec3 reflectDir = reflect(-lightDir, normal);
    float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);
```

接下来是点光源还需要计算光线衰减

```glsl
    // 衰减
    float distance = length(light.position - FragPos);
    float attenuation = 1.0 / (light.constant + light.linear * distance + light.quadratic * distance * distance);
```

最终结果再乘上衰减效果

```glsl
    vec3 ambient = light.ambient * vec3(texture(material.diffuse, TexCoords));
    vec3 diffuse = light.diffuse * diff * vec3(texture(material.diffuse, TexCoords));
    vec3 specular = light.specular * spec * vec3(texture(material.specular, TexCoords));

    return (ambient + diffuse + specular) * attenuation;
}
```

### 2.3 聚光源

聚光灯基本就跟点光源一致

```glsl
struct SpotLight {
    vec3 position;
    vec3 direction;
    float cutOff;
    float outerCutOff;

    float constant;
    float linear;
    float quadratic;

    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
};
uniform SpotLight spotLight;// 聚光

vec3 CalcSpotLight(SpotLight light, vec3 normal, vec3 fragPos, vec3 viewDir);
```

```glsl
vec3 CalcSpotLight(SpotLight light, vec3 normal, vec3 fragPos, vec3 viewDir) {
    vec3 lightDir = normalize(light.position - fragPos);

    // 漫反射
    float diff = max(dot(normal, lightDir), 0.0);

    // 镜面光
    vec3 reflectDir = reflect(-lightDir, normal);
    float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);

    // 衰减
    float distance = length(light.position - FragPos);
    float attenuation = 1.0 / (light.constant + light.linear * distance + light.quadratic * distance * distance);
```

与点光源不同的是，我们还需要再多计算一个光照角度(theta)，并计算光照强度(intensity)

```glsl
    // 聚光增强
    float theta = dot(lightDir, normalize(-light.direction));
    float epsilon = light.cutOff - light.outerCutOff;
    float intensity = clamp((theta - light.outerCutOff) / epsilon, 0.0, 1.0);

    vec3 ambient = light.ambient * vec3(texture(material.diffuse, TexCoords));
    vec3 diffuse = light.diffuse * diff * vec3(texture(material.diffuse, TexCoords));
    vec3 specular = light.specular * spec * vec3(texture(material.specular, TexCoords));

    return (ambient + diffuse + specular) * attenuation * intensity;
}
```

### 2.4 合成光线

有了三种光源，片段着色器的主方法就是计算所有光源的结果并加总

```glsl
void main() {
    vec3 norm = normalize(Normal);
    vec3 viewDir = normalize(viewPos - FragPos);

    vec3 result = CalcDirLight(dirLight, norm, viewDir);
    for (int i = 0; i < NR_POINT_LIGHTS; i++) {
        result += CalcPointLight(pointLights[i], norm, FragPos, viewDir);
    }
    result += CalcSpotLight(spotLight, norm, FragPos, viewDir);

    FragColor = vec4(result, 1.0);
}
```

## 3. 布置场景 & 结果

`main.cpp` 中进行顶点设置、物体模型、光源模型的配置，着色器的各个光源属性配置，光照纹理配置等[前几篇](#系列文章)都有了，对代码细节有兴趣的可以看下面的[源代码](#完整代码示例)，最终效果如下

1. 正面

![](https://picures.oss-cn-beijing.aliyuncs.com/img/open_gl_learning_6_advanced_light_result1.png)

2. 侧面

![](https://picures.oss-cn-beijing.aliyuncs.com/img/open_gl_learning_6_advanced_light_result2.png)

3. 聚光效果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/open_gl_learning_6_advanced_light_result3.png)

# 其他资源

## 参考连接

| Title                   | Link                                                                                                                                             |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| 多光源 - LearnOpenGL CN | [https://learnopengl-cn.github.io/02%20Lighting/06%20Multiple%20lights/](https://learnopengl-cn.github.io/02%20Lighting/06%20Multiple%20lights/) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/others/open_gl/open_gl_advanced_light](https://github.com/superfreeeee/Blog-code/tree/main/others/open_gl/open_gl_advanced_light)
