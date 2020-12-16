# Spring Cloud 实战: 手把手带你用 Netflix Eureka 实现微服务注册/发现（Service Discovery）

@[TOC](文章目录)

<!-- TOC -->

- [Spring Cloud 实战: 手把手带你用 Netflix Eureka 实现微服务注册/发现（Service Discovery）](#spring-cloud-实战-手把手带你用-netflix-eureka-实现微服务注册发现service-discovery)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [Service Discovery 回顾](#service-discovery-回顾)
  - [Netflix Eureka 架构](#netflix-eureka-架构)
  - [项目启动](#项目启动)
    - [项目结构](#项目结构)
    - [搭建项目外壳](#搭建项目外壳)
    - [Eureka Server 注册中心](#eureka-server-注册中心)
    - [Provider 微服务](#provider-微服务)
    - [Consumer 微服务](#consumer-微服务)
  - [完整项目内容](#完整项目内容)
    - [最终项目结构 & 配置信息 & 接口信息](#最终项目结构--配置信息--接口信息)
    - [Eureka Server（eureka-server 模块）](#eureka-servereureka-server-模块)
    - [Provider（eureka-service-provider 模块）](#providereureka-service-provider-模块)
    - [Consumer（eureka-service-consumer 模块）](#consumereureka-service-consumer-模块)
- [结语](#结语)

<!-- /TOC -->

## 简介

前一篇：<a href="https://blog.csdn.net/weixin_44691608/article/details/111189830">Spring Cloud: Overview 概述</a>，对 Spring Cloud 的所有成员有大概的说明。

本篇将要来实现第一个 Spring Cloud 模块：`使用 Spring Cloud Netflix Eureka 实现 Service Discovery`

## 参考

<table>
  <tr>
    <td>Eureka 概念</td>
    <td><a href="https://blog.csdn.net/u013084266/article/details/100105965">https://blog.csdn.net/u013084266/article/details/100105965</a></td>
  </tr>
  <tr>
    <td>Spring Cloud 入门教程(一): 服务注册</td>
    <td><a href="https://www.cnblogs.com/chry/p/7248947.html">https://www.cnblogs.com/chry/p/7248947.html</a></td>
  </tr>
  <tr>
    <td>idea创建springcloud项目图文教程(EurekaServer注册中心)（六）</td>
    <td><a href="https://blog.csdn.net/hcmony/article/details/77855158">https://blog.csdn.net/hcmony/article/details/77855158</a></td>
  </tr>
  <tr>
    <td>idea创建springcloud项目图文教程(创建服务提供者)（七）</td>
    <td><a href="https://blog.csdn.net/hcmony/article/details/77855843">https://blog.csdn.net/hcmony/article/details/77855843</a></td>
  </tr>
  <tr>
    <td>idea创建springcloud项目图文教程(创建消费者)（八）</td>
    <td><a href="https://blog.csdn.net/hcmony/article/details/77855903">https://blog.csdn.net/hcmony/article/details/77855903</a></td>
  </tr>
</table>

# 正文

## Service Discovery 回顾

首先我们先来回顾一下微服务的`服务注册/发现(Service Discovery)`功能

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springcloud_service_discovery2.png)

当我们将服务拆分成一个个`微服务(MicroService)`时我们可以建立一个`注册中心(Register Center)`，之后所有服务启动时都需要向注册中心`注册(Register)`自己的服务，其他微服务想要调用时就可以从注册中心获取可用的服务信息。

## Netflix Eureka 架构

为了实现服务注册的特性，本篇采用 `Spring Cloud Netflix Eureka` 依赖来达成整个目标。

与我们前面提到的类似，Eureka 的注册中心(Register Server)称为一个 `Eureka Server`，其他每一个微服务(MicroService)则被视为一个 `Eureka Client`。

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_concept.png)

当然我们也可以修改配置后启动多个 `Eureka Server` 并互相发现，实现注册中心的高可用性

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_concept2.png)

实现细节像是什么续约啊、心跳检测啥的用于保证服务持续可用啥的这边就不过多说明了，可以查看参考链接或是自行找寻相关材料吧，这边主要专注在实现上。

## 项目启动

接下来我们就手把手 使用 `Netflix Eureka` 带你实现一个简单的微服务注册项目

### 项目结构

首先先给出项目的目标架构图

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_structure.png)

此次实践中我们将构建一个 Eureka Server 和两个 Eureka Client（分别是 provider 和 consumer），同时 consumer 将透过 `restTemplate` 向 provider 请求服务。

- 架构缺陷

这边的实践存在一个瑕疵：consumer 向 provider 请求服务时仅仅只是简单的 HTTP 请求，与外部的一般调用者无意，也就是说我们还没有用到 Eureka Server 提供的`服务发现(Discovery)`，而仅仅实现了向 Eureka Server `注册服务(Register)`的部分。下一篇我们将使用 `Feign` 来真正利用 Eureka Server 带来的服务发现功能。

### 搭建项目外壳

> 搭建空项目

接下来我们就开始搭建项目内容了。首先先利用 IDEA 创建空的项目，详细步骤如下：

1. 点击 `New Project` 创建新的项目

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_project1.png)

2. 选择 `Empty Project` 创建一个空项目，接下来所有微服务都放在该目录之下

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_project2.png)

3. 输入项目名称 `spring_cloud_eureka`（可自定）

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_project3.png)

### Eureka Server 注册中心

> 创建 Eureka Server 注册中心

1. 选择 `File -> New -> Module` 新增模块

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_server1.png)

2. 使用 `Spring Initializr` 创建 Spring 项目（IDEA 新的插件好像是叫 `Spring Assistant`，一个东西）

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_server2.png)

3. 输入一些 maven 项目配置信息（项目名称、Java 版本、包结构等）

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_server3.png)

4. 选择 Spring 依赖，选 `Spring Cloud Discovery -> Eureka Server`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_server4.png)

5. 项目路径就放在刚刚创建的空项目之下就行了：`/spring_cloud_eureka/eureka-server`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_server5.png)

如果建好项目之后看到下面长成这幅德行

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_server5_1.png)

那就到 `File -> Project Structure -> Module` 将 `src/main/java` 指定为 `Sources` 就好了

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_server5_2.png)

> 修改依赖配置 pom.xml

6. 建好项目之后第一件事先来修改 `pom.xml` 的配置信息（最后会附上详细文件内容）

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_server6.png)

- Spring Boot 版本：`2.3.2.RELEASE`
- Spring Cloud 版本：`Hoxton.SR6`

> 修改入口类

7. 在入口类 `EurekaServerApplication` 添加 `@EnableEurekaServer`，声明为一个注册中心

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_server7.png)

> 修改项目配置 application.yml

8. 将 `application.properties` 改名为 `application.yml` 并填入配置信息

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_server8.png)

```yml
server:
  port: 8800 # 启动端口

eureka:
  client:
    register-with-eureka: false # 不向 eureka 注册
    fetch-registry: false # 表明自己就是 eureka server
  instance:
    secure-port-enabled: false
```

> 启动并查看可视化界面

9. 直接在 IDEA 启动项目之后可以访问 `http://localhost:8800` 查看可视化界面

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_server9.png)

到此我们的 Eureka Server 就创建完毕啦，不过后面的路还长着呢hhh

### Provider 微服务

接下来我们要创建第二个模块：Provider 微服务，基本上与前面创建 Eureka Server 模块的顺序几乎一样（实在是不想再截一轮了就不放图了），不同的之处就用表格列出吧：

| 对应步骤            | 属性                         | 值                                                        |
| ------------------- | ---------------------------- | --------------------------------------------------------- |
| 3(maven 配置)       | Artifact Id                  | eureka-service-provider                                   |
|                     | Project name                 | eureka-service-provider                                   |
|                     | Package name                 | com.example.provider                                      |
| 4(Spring 依赖选择)  | Eureka Client 依赖           | Spring Cloud Discovery.Eureka Discovery Client            |
| 5(项目路径位置)     | Module name                  | eureka-service-provider                                   |
|                     | Content root                 | xxx/spring_cloud_eureka/eureka-service-provider           |
|                     | Module file location         | xxx/spring_cloud_eureka/eureka-service-provider           |
| 6(依赖配置 pom.xml) | Eureka Client 依赖(不用修改) | 使用了 `spring-cloud-starter-netflix-eureka-client` 依赖  |
|                     | Web 项目依赖                 | 新增 `spring-boot-starter-web` 依赖                       |
| 7(入口类声明)       | Annotation                   | 改为添加 `@EnableEurekaClient` 注释，声明为 Eureka Client |

第 8 步开始来加入 Provider 的服务内容

> Provider 服务 & 配置

1. 首先我们先创立一个实体类 `User`

- `User.java`

```java
package com.example.provider;

public class User {

    private static int count = 0;
    private Integer id;
    private String name;
    private String password;

    public User() {
        id = count++;
    }

    public User(String name, String password) {
        id = count++;
        this.name = name;
        this.password = password;
    }
    // 省略 id 的 getter
    // 省略 name、password 的 getter/setter
}
```

9. 接下来我们建立 `ProviderController` 并加入我们的服务接口

- `ProviderController.java`

```java
package com.example.provider;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.HashMap;
import java.util.Map;

@RestController // 声明为 Controller 类
public class ProviderController {

    // 获取配置信息的端口
    @Value("${server.port}")
    private String port;

    // 记录接下来的用户信息（没有持久化）
    private Map<String, User> userTable = new HashMap<>();

    // 默认路由
    @GetMapping("/")
    public String info() {
        return "Default route from port: " + port;
    }

    // 查看所有组册用户
    @GetMapping("/users")
    public Map<String, User> getUserTable() {
        return userTable;
    }

    // 注册接口
    @GetMapping("/register")
    public String register(String name, String password) {
        // 检查参数
        if (name.length() == 0) {
            return "User name is required.";
        }
        if (password.length() == 0) {
            return "User password is required.";
        }
        // 用户名查重
        if (userTable.containsKey(name)) {
            return "User " + name + " is already exist.";
        }
        // 建立并加入用户
        User user = new User(name, password);
        userTable.put(user.getName(), user);
        return "Register success with id: " + user.getId() + ".";
    }

    // 登入接口
    @GetMapping("/login")
    public String login(String name, String password) {
        if (!userTable.containsKey(name)) {
            return "User " + name + " didn't exist.";
        }
        User user = userTable.get(name);
        if (!user.getPassword().equals(password)) {
            return "Incorrect password.";
        }
        return "Login success with user id: " + user.getId() + ".";
    }
}
```

10. 最后一样需要将 `application.properties` 改名为 `application.yml` 并填入配置信息

```yml
server:
  port: 8801 # 启动端口

eureka:
  instance:
    appname: service-provider # 实例名称
  client:
    service-url:
      defaultZone: http://localhost:8800/eureka/ # 注册中心

spring:
  application:
    name: service-provider # 服务名称
```

这边不同的地方在于我们透过设置 `eureka.client.service-url.defaultZone` 来指定注册中心，接下来我们就可以在浏览器访问服务了

> 测试

11. 我们先启动前面的 Eureka Server 然后启动我们的 Provider，并访问 `http://localhost:8800` 查看我们的注册中心

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_provider1.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_provider2.png)

12. 接下来我们分别测试 Provider 的四个接口是否正确生效

- `http://localhost:8801`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_provider3.png)

- `http://localhost:8801/register?name=user1&password=123`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_provider4.png)

- `http://localhost:8801/register?name=user2&password=456`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_provider5.png)

- `http://localhost:8801/users`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_provider6.png)

- `http://localhost:8801/login?name=user1&password=123`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_provider7.png)

确定好 Provider 服务可以了之后，接下来轮到 Consumer 服务

### Consumer 微服务

第三个模块是 Consumer 服务，与前两个一样，也是经过 `File -> New -> Module -> ...` 等一系列步骤创建，详细步骤就不展开了，一样给出配置信息：

| Stage               | Property                     | Value / Action                                            |
| ------------------- | ---------------------------- | --------------------------------------------------------- |
| 3(maven 配置)       | Artifact Id                  | eureka-service-consumer                                   |
|                     | Project name                 | eureka-service-consumer                                   |
|                     | Package name                 | com.example.consumer                                      |
| 4(Spring 依赖选择)  | Eureka Client 依赖           | Spring Cloud Discovery.Eureka Discovery Client            |
| 5(项目路径位置)     | Module name                  | eureka-service-consumer                                   |
|                     | Content root                 | xxx/spring_cloud_eureka/eureka-service-consumer           |
|                     | Module file location         | xxx/spring_cloud_eureka/eureka-service-consumer           |
| 6(依赖配置 pom.xml) | Eureka Client 依赖(不用修改) | 使用了 `spring-cloud-starter-netflix-eureka-client` 依赖  |
|                     | Web 项目依赖                 | 新增 `spring-boot-starter-web` 依赖                       |
| 7(入口类声明)       | Annotation                   | 改为添加 `@EnableEurekaClient` 注释，声明为 Eureka Client |

我们一样从第 8 步开始

> Consumer 服务 & 配置

在 Consumer 我们打算的事不太多，直接代理 Provider 提供的服务就行了

8. 创建实体类 `User`（内容与 Provider 的相仿，用于接受请求结果）

- `User.java`

```java
package com.example.consumer;

public class User {

    private Integer id;
    private String name;
    private String password;
    // 省略 id、name、password 的 getter
}
```

9. 接下来我们在主入口 `EurekaServiceConsumerApplication.main` 的下方注入一个 `RestTemplate`，等下请求 Provider 服务会用到

- `EurekaServiceConsumerApplication.java`

```java
package com.example.consumer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@EnableEurekaClient
@SpringBootApplication
public class EurekaServiceConsumerApplication {

	public static void main(String[] args) {
		SpringApplication.run(EurekaServiceConsumerApplication.class, args);
	}

    // 注入一个 RestTemplate 的 Bean
	@Bean
	public RestTemplate restTemplate() {
		return new RestTemplate();
	}
}
```

10. 再来我们创建 `ConsumerController` 来定义我们的接口

```java
package com.example.consumer;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import java.util.Map;

@RestController
public class ConsumerController {

    // 配置文件中的端口
    @Value("${server.port}")
    private String port;

    // 自动装配刚刚注入的 RestTemplate
    @Autowired
    private RestTemplate restTemplate;

    // Provider 的路由
    private String providerUrl = "http://localhost:8801";

    @GetMapping("/")
    public String info() {
        return "Default route from port: " + port;
    }

    /* 
        下面三个方法都是直接代理 Provider 的服务
     */
    @GetMapping("/users")
    public Map<String, User> users() {
        String url = providerUrl + "/users";
        // 使用 getForObject 快速发起 GET 请求并获取 body
        Map<String, User> res = restTemplate.getForObject(url, Map.class);
        return res;
    }

    @GetMapping("/register")
    public String register(String name, String password) {
        String url = providerUrl + "/register?name=" + name + "&password=" + password;
        String res = restTemplate.getForObject(url, String.class);
        return "Proxy from " + port + ": " + res;
    }

    @GetMapping("/login")
    public String login(String name, String password) {
        String url = providerUrl + "/login?name=" + name + "&password=" + password;
        String res = restTemplate.getForObject(url, String.class);
        return "Proxy from " + port + ": " + res;
    }
}
```

11. 最后的最后一样要配置一下 `application.yml`

- `application.yml`

```yml
server:
  port: 8802 # 启动端口

eureka:
  instance:
    appname: service-consumer # 实例名称
  client:
    service-url:
      defaultZone: http://localhost:8800/eureka/ # 注册中心

spring:
  application:
    name: service-consumer # 服务名称
```

这边基本跟 Provider 一致（同为 Eureka Client），只需要修改端口(port)和服务名称(application.name、instance.appname)就好了

> 启动 & 测试

12. 接下来跟测试 Provider 的时候一样，先分别启动 Eureka Server、Provider、Consumer，然后到 `8800` 查看 Eureka Server 的注册情形

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_consumer1.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_consumer2.png)

我们可以看到 Provider 跟 Consumer 都正确的注册到 Eureka Server 了

13. 最后简单测试一下 Consumer 的接口

- `http://localhost:8802`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_consumer3.png)

- `http://localhost:8802/register?name=user1&password=123`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_consumer4.png)

- `http://localhost:8801/register?name=user2&password=456`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_consumer5.png)

- `http://localhost:8802/users`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_consumer6.png)

由于 Consumer 只是代理 Provider 的操作，所以不论是 8801 或是 8802 注册的用户都保存在 Provider 的 userTable 中

- `http://localhost:8802/login?name=user1&password=123`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/netflix_eureka_sample_consumer7.png)

## 完整项目内容

最后给出最终版本的项目完整内容

### 最终项目结构 & 配置信息 & 接口信息

> 最终项目结构

```
/spring_cloud_eureka
├── eureka-server
│   ├── pom.xml
│   └── src
│       ├── main
│       │   ├── java
│       │   │   └── com
│       │   │       └── example
│       │   │           └── server
│       │   │               └── EurekaServerApplication.java
│       │   └── resources
│       │       └── application.yml
│       └── test // 省略
├── eureka-service-consumer
│   ├── pom.xml
│   └── src
│       ├── main
│       │   ├── java
│       │   │   └── com
│       │   │       └── example
│       │   │           └── consumer
│       │   │               ├── ConsumerController.java
│       │   │               ├── EurekaServiceConsumerApplication.java
│       │   │               └── User.java
│       │   └── resources
│       │       └── application.yml
│       └── test // 省略
└── eureka-service-provider
    ├── pom.xml
    └── src
        ├── main
        │   ├── java
        │   │   └── com
        │   │       └── example
        │   │           └── provider
        │   │               ├── EurekaServiceProviderApplication.java
        │   │               ├── ProviderController.java
        │   │               └── User.java
        │   └── resources
        │       └── application.yml
        └── test // 省略
```

> 服务端口信息

| Service       | name                    | Port |
| ------------- | ----------------------- | ---- |
| Eureka Server | eureka-server           | 8800 |
| Provider      | eureka-service-provider | 8801 |
| Consumer      | eureka-service-consumer | 8802 |

> 接口信息

| Service   | API                     | Usage                          |
| --------- | ----------------------- | ------------------------------ |
| 默认1     | localhost:8801          | Provider 的默认端口            |
| 默认2     | localhost:8802          | Consumer 的默认端口            |
| 注册1     | localhost:8801/register | 在 Provider 注册用户           |
| 注册2     | localhost:8802/register | 代理 Provider 的注册接口       |
| 登陆1     | localhost:8801/login    | 在 Provider 登陆               |
| 登陆2     | localhost:8802/login    | 代理 Provider 的登陆接口       |
| 用户列表1 | localhost:8801/users    | 查看 Provider 上的所有用户信息 |
| 用户列表2 | localhost:8802/users    | 查看 Provider 上的所有用户信息 |

### Eureka Server（eureka-server 模块）

> pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.3.2.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.example</groupId>
	<artifactId>eureka-server</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>eureka-server</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Hoxton.SR6</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

	<repositories>
		<repository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/milestone</url>
		</repository>
	</repositories>

</project>
```

> application.yml

```yml
server:
  port: 8800 # 启动端口

eureka:
  client:
    register-with-eureka: false # 不向 eureka 注册
    fetch-registry: false # 表明自己就是 eureka server
  instance:
    secure-port-enabled: false
```

> EurekaServerApplication.java

```java
package com.example.server;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(EurekaServerApplication.class, args);
	}

}
```

### Provider（eureka-service-provider 模块）

> pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.3.2.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.example</groupId>
	<artifactId>eureka-service-provider</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>eureka-service-provider</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Hoxton.SR6</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

	<repositories>
		<repository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/milestone</url>
		</repository>
	</repositories>

</project>
```

> application.yml

```yml
server:
  port: 8801 # 启动端口

eureka:
  instance:
    appname: service-provider # 实例名称
  client:
    service-url:
      defaultZone: http://localhost:8800/eureka/ # 注册中心

spring:
  application:
    name: service-provider # 服务名称
```

> EurekaServiceProviderApplication.java

```java
package com.example.provider;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@EnableEurekaClient
@SpringBootApplication
public class EurekaServiceProviderApplication {

	public static void main(String[] args) {
		SpringApplication.run(EurekaServiceProviderApplication.class, args);
	}

}
```

> ProviderController.java

```java
package com.example.provider;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.HashMap;
import java.util.Map;

@RestController
public class ProviderController {

    @Value("${server.port}")
    private String port;

    private Map<String, User> userTable = new HashMap<>();

    @GetMapping("/")
    public String info() {
        return "Default route from port: " + port;
    }

    @GetMapping("/users")
    public Map<String, User> getUserTable() {
        return userTable;
    }

    @GetMapping("/register")
    public String register(String name, String password) {
        if (name.length() == 0) {
            return "User name is required.";
        }
        if (password.length() == 0) {
            return "User password is required.";
        }
        if (userTable.containsKey(name)) {
            return "User " + name + " is already exist.";
        }
        User user = new User(name, password);
        userTable.put(user.getName(), user);
        return "Register success with id: " + user.getId() + ".";
    }

    @GetMapping("/login")
    public String login(String name, String password) {
        if (!userTable.containsKey(name)) {
            return "User " + name + " didn't exist.";
        }
        User user = userTable.get(name);
        if (!user.getPassword().equals(password)) {
            return "Incorrect password.";
        }
        return "Login success with user id: " + user.getId() + ".";
    }
}
```

> User.java

```java
package com.example.provider;

public class User {

    private static int count = 0;
    private Integer id;
    private String name;
    private String password;

    public User() {
        id = count++;
    }

    public User(String name, String password) {
        id = count++;
        this.name = name;
        this.password = password;
    }

    public Integer getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

### Consumer（eureka-service-consumer 模块）

> pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.3.2.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.example</groupId>
	<artifactId>eureka-service-consumer</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>eureka-service-consumer</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Hoxton.SR6</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

	<repositories>
		<repository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/milestone</url>
		</repository>
	</repositories>

</project>
```

> application.yml

```yml
server:
  port: 8802 # 启动端口

eureka:
  instance:
    appname: service-consumer # 实例名称
  client:
    service-url:
      defaultZone: http://localhost:8800/eureka/ # 注册中心

spring:
  application:
    name: service-consumer # 服务名称
```

> EurekaServiceConsumerApplication.java

```java
package com.example.consumer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@EnableEurekaClient
@SpringBootApplication
public class EurekaServiceConsumerApplication {

	public static void main(String[] args) {
		SpringApplication.run(EurekaServiceConsumerApplication.class, args);
	}

	@Bean
	public RestTemplate restTemplate() {
		return new RestTemplate();
	}
}

```

> ConsumerController.java

```java
package com.example.consumer;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import java.util.Map;

@RestController
public class ConsumerController {

    @Value("${server.port}")
    private String port;

    @Autowired
    private RestTemplate restTemplate;

    private String providerUrl = "http://localhost:8801";

    @GetMapping("/")
    public String info() {
        return "Default route from port: " + port;
    }

    @GetMapping("/users")
    public Map<String, User> users() {
        String url = providerUrl + "/users";
        Map<String, User> res = restTemplate.getForObject(url, Map.class);
        return res;
    }

    @GetMapping("/register")
    public String register(String name, String password) {
        String url = providerUrl + "/register?name=" + name + "&password=" + password;
        String res = restTemplate.getForObject(url, String.class);
        return "Proxy from " + port + ": " + res;
    }

    @GetMapping("/login")
    public String login(String name, String password) {
        String url = providerUrl + "/login?name=" + name + "&password=" + password;
        String res = restTemplate.getForObject(url, String.class);
        return "Proxy from " + port + ": " + res;
    }
}
```

> User.java

```java
package com.example.consumer;

public class User {

    private Integer id;
    private String name;
    private String password;

    public Integer getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public String getPassword() {
        return password;
    }
}
```

# 结语

呼！终于结束了，其实整个服务也没啥内容，光是配置就弄了好久，而且好像该弄个代码仓库了这样一个个复制贴上也不是办法hhh。

下一篇将带来使用 Feign 实现根据 Eureka Server 注册中心中的服务注册信息来实现内部的`服务间调用(service-to-service calls)`。
