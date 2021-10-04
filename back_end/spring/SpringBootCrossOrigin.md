# SpringBoot 实战: 跨域配置（添加 Cors 跨域策略实现）

@[TOC](文章目录)

<!-- TOC -->

- [SpringBoot 实战: 跨域配置（添加 Cors 跨域策略实现）](#springboot-实战-跨域配置添加-cors-跨域策略实现)
- [正文](#正文)
  - [1. 接口实现](#1-接口实现)
  - [2. 前端直接访问](#2-前端直接访问)
  - [3. 添加跨域配置后访问](#3-添加跨域配置后访问)
  - [4. 总结](#4-总结)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

## 1. 接口实现

简单创建一个 SpringBoot 项目，添加 `DemoController`

- `/be/src/main/java/com/example/demo/DemoController.java`

```java
package com.example.demo;

@RestController
public class DemoController {

    @GetMapping("/{name}")
    public String greeting(@PathVariable String name) {
        return "Hello " + name;
    }
}
```

接下来直接使用浏览器访问测试接口正常运作

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_cross_origin_1_test_api.png)

## 2. 前端直接访问

- `/fe/index.js`

```js
console.log('>>> index.js loaded');

fetch('http://localhost:8080/superfree')
  .then((res) => res.text())
  .then((res) => {
    console.log(`res: ${res}`);
  });
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_cross_origin_2_fail.png)

因跨域问题而失败

## 3. 添加跨域配置后访问

添加 `WebMvcConfig` 配置类，实现 `WebMvcConfigurer` 接口

- `/be/src/main/java/com/example/demo/WebMvcConfig.java`

```java
package com.example.demo;

@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**") // 配置路由
                .allowCredentials(true) // 添加凭证
                .allowedOriginPatterns("*") // 允许跨域路由正则
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTION") // 允许跨域方法
                .allowedHeaders("*"); // 允许传送 Http 头部
    }
}
```

配置完成之后再次从前端尝试

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_cross_origin_3_success.png)

## 4. 总结

- SprinBoot 实现跨域方法
  - 实现 `WebMvcConfigurer` 接口，重载 `addCorsMappings` 方法
  - 根据需要添加自己的配置项

# 其他资源

## 参考连接

| Title                        | Link                                                                                                   |
| ---------------------------- | ------------------------------------------------------------------------------------------------------ |
| SpringBoot解决跨域的三种方式 | [https://www.cnblogs.com/antLaddie/p/14751540.html](https://www.cnblogs.com/antLaddie/p/14751540.html) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/back_end/spring/spring_boot_cross_origin](https://github.com/superfreeeee/Blog-code/tree/main/back_end/spring/spring_boot_cross_origin)
