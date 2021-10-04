# SpringBoot 实战: 静态资源分发配置

@[TOC](文章目录)

<!-- TOC -->

- [SpringBoot 实战: 静态资源分发配置](#springboot-实战-静态资源分发配置)
- [正文](#正文)
  - [1. 默认配置](#1-默认配置)
  - [2. 在 application.yml 中配置](#2-在-applicationyml-中配置)
  - [3. 实现 WebMvcConfigurer 配置](#3-实现-webmvcconfigurer-配置)
    - [3.1 实现按命名空间划分静态资源](#31-实现按命名空间划分静态资源)
  - [4. 总结](#4-总结)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

## 1. 默认配置

- `classpath:static/`、`classpath:public/`、`classpath:resources/`、`classpath:META_INF/resources/` 四个目录下的文件默认作为静态资源分发

- 步骤

1. 创建一个 SpringBoot 项目
2. 向默认 classpath 目录添加如下文件

```
/src/main/resources
├── application.yml
├── META-INF
│   └── resources
│       └── from_meta_inf_resources.png
├── public
│   └── from_public.png
├── resources
│   └── from_resources.png
└── static
    └── from_static.png
```

3. 按文件名访问默认根目录

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_static_resources_1_default_static.png) ![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_static_resources_2_default_public.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_static_resources_3_default_resources.png) ![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_static_resources_4_default_meta_inf_resources.png)

## 2. 在 application.yml 中配置

- 会覆盖默认的配置
- 一样仅能将静态资源目录映射到根路径下

1. 配置文件

- `/src/main/resources/application.yml`

```yml
spring:
  web:
    resources:
      static-locations:
        - classpath:templates/
```

2. 添加静态资源

```
/src/main/resources
├── application.yml
└── templates
    └── from_templates.png
```

3. 访问静态资源

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_static_resources_5_static_locations.png)

## 3. 实现 WebMvcConfigurer 配置

- 会覆盖 `application.yml` 的配置

1. 添加 `WebMvcConfig` 配置类（实现 `WebMvcConfigurer` 接口）

- `/src/main/java/com/example/demo/WebMvcConfig.java`

将 static 目录和 custom 目录作为静态资源目录

```java
package com.example.demo;

@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
       registry.addResourceHandler("/**").addResourceLocations("classpath:static/", "classpath:custom/");
    }
}
```

2. 添加静态资源
3. 
```
/src/main/resources
├── application.yml
└── custom
    └── from_custom.png
```

3. 访问资源

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_static_resources_6_webmvcconfigurer_root.png)

### 3.1 实现按命名空间划分静态资源 

- 目标：静态资源分发按命名空间划分
- 一样使用实现 `WebMvcConfigurer` 的方式

1. 修改配置类

- `/src/main/java/com/example/demo/WebMvcConfig.java`

```java
package com.example.demo;

@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/static/**").addResourceLocations("classpath:static/");
        registry.addResourceHandler("/public/**").addResourceLocations("classpath:public/");
    }
}
```

2. 访问静态资源

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_static_resources_7_webmvcconfigurer_namespace_static.png) ![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_static_resources_8_webmvcconfigurer_namespace_public.png)

## 4. 总结

- 静态资源分配配置方法
  - A. 默认配置
  - B. `application.yml` 配置文件
  - C. 实现 `WebMvcConfigurer` 接口的配置类
- 效果优先级（覆盖）
  - C > B > A
- 推荐：使用 C 方法，同时使用命名空间区分不同资源类型或模块

# 其他资源

## 参考连接

| Title                                 | Link                                                                                                                     |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| SpringBoot中配置Web静态资源路径——详解 | [https://blog.csdn.net/wangxin1949/article/details/89016428](https://blog.csdn.net/wangxin1949/article/details/89016428) |
| springboot之静态资源路径间的较量      | [https://zhuanlan.zhihu.com/p/30436022](https://zhuanlan.zhihu.com/p/30436022)                                           |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/back_end/spring/spring_boot_static_resources](https://github.com/superfreeeee/Blog-code/tree/main/back_end/spring/spring_boot_static_resources)
