# Spring Boot 基础: 使用 `@ConfigurationProperties` 实现自定义配置属性的自动装配

@[TOC](文章目录)

<!-- TOC -->

- [Spring Boot 基础: 使用 `@ConfigurationProperties` 实现自定义配置属性的自动装配](#spring-boot-基础-使用-configurationproperties-实现自定义配置属性的自动装配)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [提前预览](#提前预览)
  - [`@Value`](#value)
  - [`@ConfigurationProperties`](#configurationproperties)
  - [`@EnableConfigurationProperties`](#enableconfigurationproperties)
  - [`spring-boot-configuration-processor`](#spring-boot-configuration-processor)
  - [完整代码示例](#完整代码示例)
- [结语](#结语)

<!-- /TOC -->

## 简介

在 Spring Boot 的实践中我们有时会需要针对整个项目进行全局配置，可能是应用的部署信息、依赖配置信息，甚至是服务需要的全局参数等。通常我们都会将配置信息写在 `resources/application.yml(或是 .properties)` 内，接下来我们可以透过几种手段来提取配置信息，接下来让我娓娓道来。

## 参考

<table>
  <tr>
    <td>SPRINGBOOT用@CONFIGURATIONPROPERTIES获取配置文件值</td>
    <td><a href="https://www.cnblogs.com/lihaoyang/p/10223339.html">https://www.cnblogs.com/lihaoyang/p/10223339.html</a></td>
  </tr>
  <tr>
    <td>Spring Boot 配置文件注入，在yml中可以自动提示</td>
    <td><a href="https://www.cnblogs.com/vianzhang/p/13615123.html">https://www.cnblogs.com/vianzhang/p/13615123.html</a></td>
  </tr>
  <tr>
    <td>SPRINGBOOT项目启动成功后执行一段代码的两种方式</td>
    <td><a href="https://www.cnblogs.com/zuidongfeng/p/9926471.html">https://www.cnblogs.com/zuidongfeng/p/9926471.html</a></td>
  </tr>
  <tr>
    <td>Spring Boot中自定义属性的代码完成</td>
    <td><a href="https://www.mdoninger.de/2015/05/16/completion-for-custom-properties-in-spring-boot.html">https://www.mdoninger.de/2015/05/16/completion-for-custom-properties-in-spring-boot.html</a></td>
  </tr>
</table>

# 正文

## 提前预览

首先我们先预览下面提到的各个注解或依赖的作用：

1. `@Value` 引入单个配置属性值
2. `@ConfigurationProperties` 声明配置属性的映射类型
3. `@EnableConfigurationProperties` 允许挂载配置对象，同时也是配置对象注册 Bean 的入口
4. `spring-boot-configuration-processor` 自动生成配置属性说明文件，提供编程时的代码提示

## `@Value`

首先我们第一个想到的方法是使用 `@Value("${xxx}")` 注解来获取配置文件的配置项，使用示例如下：

> application.yml

```yml
server:
  port: 8800
```

> DemoController.java

```java
@RestController
class DemoController {
    
    @Value("${server.port}")
    private Integer port;

    @GetMapping("/")
    public String info() {
        return "Default route from localhost:" + port;
    }
}
```

这样一来我们就完成配置项的引入，但是如果需要配置的项目越来越多结构越来越复杂呢？还要像这样一个个引入吗？

> application.yml

```yml
my:
  prop:
    name: superfree
    port: 8800
    exclude-users:
      - user1
      - user2
      - user3
    server:
      name: my-server
      url: http://localhost:8801
    client:
      name: my-client
      url: http://localhost:8802
```

> DemoController.java

```java
@RestController
class DemoController {
    
    @Value("${my.prop.name}")
    private String name;
    @Value("${my.prop.port}")
    private Integer port;
    @Value("${my.prop.exclude-users}")
    private List<String> excludeUsers;
    @Value("${my.prop.server.name}")
    private String serverName;
    @Value("${my.prop.server.url}")
    private String serverUrl;
    @Value("${my.prop.client.name}")
    private String clientName;
    @Value("${my.prop.client.url}")
    private String clientUrl;
    // ...
}
```

难不成要像这样一个个引入吗？这样也太丑而且有够麻烦hhh，所以通常我们选择采取另一个做法

## `@ConfigurationProperties`

`@ConfigurationProperties` 这个注解可以声明一个配置属性对应的类型，然后我们就可以使用自动装配的功能，声明方式如下：

> application.yml：配置项结构

```yml
my:
  prop:
    name: superfree
    port: 8800
```

> MyProperties.java

```java

// 透过 prefix 参数声明前缀
@ConfigurationProperties(prefix = "my.prop")
public class MyProperties {

    // 与配置文件属性一一对应
    private String name;
    private Integer port;

    // getter/setter for name & port
}
```

## `@EnableConfigurationProperties`

配置文件和映射类型都写好了，接下来我们必须找到一个装配该类型的地方。

一种方法是将该配置类加上 `@Component`，作为一个"组件"注入

```java
@Component // 声明为独立的组件
@ConfigurationProperties(prefix = "my.prop")
public class MyProperties {
    // ...
```

第二种做法是使用 `@EnableConfigurationProperties` 将配置类挂载到我们要使用的类上，而这个类通常可以是某个配置类或是其他组件(Component)

```java
@Configuration // or @Component
@EnableConfigurationProperties(MyProperties.class)
public class MyConfig {

    // 使用 EnableConfigurationProperties 之后就可以透过 @Autowired 实现自动装配
    @Autowired
    private MyProperties myProperties;
    // ...
```

or

```java
@RestController
@EnableConfigurationProperties(MyProperties.class)
public class DemoController {

    @Autowired
    private MyProperties myProperties;
    // ...
```

透过这种方式我们就可以借由声明配置项的映射类型即可实现自动装配，非常便利。

## `spring-boot-configuration-processor`

最后一件事情是，如果我们的服务被打包成第三方依赖，或是本身开发过程中，我们可能希望 IDE 能更智能的根据映射对象来提示我们到底该配置哪些属性，我们就可以用上 `spring-boot-configuration-processor` 依赖，在 maven 添加下方依赖：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-configuration-processor</artifactId>
  <optional>true</optional>
</dependency>
```

这时候我们再手动的调用 `mvn clean package`（IDEA 有提供可视化按钮直接使用），我们就会在打包后的 target 项目中看到关于配置项的结构说明文件 `spring-configuration-metadata.json`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_configuration_processor1.png)

这样一来使用我们的包作为依赖的开发者，他的 IDE 就能够根据这个文件在 yml 中给出相应的代码提示。

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_configuration_processor2.png)

若我们想要在自己开发的过程中也享受到代码提示我们可以选择将 `META-INF` 目录整个复制到 resoureces 目录之下（如下图），即可实现相同效果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_configuration_processor3.png)

## 完整代码示例

最后我们给出完整示例的关键代码，详细完整项目<a href="https://github.com/superfreeeee/Blog-code/tree/main/back_end/spring/spring_boot_configuration_properties">在这里</a>

> application.yml：配置项目（实现嵌套对象配置）

```yml
my:
  prop:
    name: superfree
    port: 8800
    exclude-users:
      - user1
      - user2
      - user3
    server:
      name: my-server
      url: http://localhost:8801
    client:
      name: my-client
      url: http://localhost:8802
```

> MyProperties.java：配置项类型

```java

@ConfigurationProperties(prefix = "my.prop")
public class MyProperties {

    private String name;
    private Integer port;
    private List<String> excludeUsers;
    private ServerProperties server;
    private ClientProperties client;
    // toString, getter/setter
}
```

> ServerProperties.java：嵌套类型1

```java
// 注意，这边就不需要 ConfigurationProperties 了
public class ServerProperties {

    private String name;
    private String url;
    // toString, getter/setter
}
```

> ClientProperties.java：嵌套类型2

```java
public class ClientProperties {

    private String name;
    private String url;
    // toString, getter/setter
}
```

> Runner.java：测试类

```java
@Component
@EnableConfigurationProperties(MyProperties.class)
public class Runner implements ApplicationRunner {

    @Autowired
    private MyProperties myProperties; // 自动装配配置项

    @Override
    public void run(ApplicationArguments args) throws Exception {
        show();
    }

    public void show() {
        System.out.println("----- show my.prop -----");
        System.out.println(myProperties);
    }
}
```

> 输出

```
----- show my.prop -----
MyProperties{name='superfree', port=8800, excludeUsers=[user1, user2, user3], server=ServerProperties{name='my-server', url='http://localhost:8801'}, client=ClientProperties{name='my-client', url='http://localhost:8802'}}
```

# 结语

好啦到此结束啦，大家都学会如何优雅的引入配置文件的内容了吗，再也不用看到一大票丑丑的 `@Value` 了。
