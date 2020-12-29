# Spring Cloud 实战: Open Feign 强化版的 Ribbon，实现更优雅的声明式服务发现/调用

@[TOC](文章目录)

<!-- TOC -->

- [Spring Cloud 实战: Open Feign 强化版的 Ribbon，实现更优雅的声明式服务发现/调用](#spring-cloud-实战-open-feign-强化版的-ribbon实现更优雅的声明式服务发现调用)
  - [简介](#简介)
  - [参考](#参考)
  - [完整示例代码](#完整示例代码)
- [正文](#正文)
  - [Feign 简介](#feign-简介)
    - [声明式服务调用（与 Ribbon 比较）](#声明式服务调用与-ribbon-比较)
  - [项目实战](#项目实战)
    - [项目抽象结构](#项目抽象结构)
    - [依赖版本 & 目录结构 & 服务接口说明](#依赖版本--目录结构--服务接口说明)
    - [关键配置](#关键配置)
      - [各项目依赖(pom.xml)](#各项目依赖pomxml)
      - [各项目运行配置(application.yml)](#各项目运行配置applicationyml)
    - [关键代码实现](#关键代码实现)
      - [Eureka Server 服务注册中心](#eureka-server-服务注册中心)
      - [Feign Service 服务提供者](#feign-service-服务提供者)
      - [Feign Client 服务使用者（重点！）](#feign-client-服务使用者重点)
    - [测试结果](#测试结果)
- [结语](#结语)

<!-- /TOC -->

## 简介

前一篇<a href="https://blog.csdn.net/weixin_44691608/article/details/111361970">Spring Cloud 实战: 使用 Netflix Ribbon 实现服务发现 & 负载均衡(Service Discovery & Load Balancing)</a>我们在建立 Eureka 服务注册的基础之上，使用 Eureka 集成的 Ribbon 进行服务发现并调用，利用的是 RestTemplate 对象并自行组合请求路由的形式如下

- `RibbonClientApplication.java`

```java
@RibbonClient(name = "ribbon-client", configuration = RibbonConfig.class)
public class RibbonClientApplication {
    public static void main(String[] args) {
    SpringApplication.run(RibbonClientApplication.class, args);
  }

  @LoadBalanced
  @Bean
  public RestTemplate restTemplate() {
    return new RestTemplate();
  }
}
```

- `ClientController`

```java
@RestController
public class ClientController {
    @Autowired
    private RestTemplate restTemplate;
    private String ribbonServiceInfoUrl = "http://ribbon-service/info";

    public String access() {
        // ...
        String res = restTemplate.getForObject(ribbonServiceInfoUrl, String.class);
        // ...
    }
}
```

这样的调用逻辑会陷入一个问题是我们需要在每次请求时（`getForObject`）由程序员自行组合请求路由、参数列表、返回类型，这样使得服务提供者(provider)和服务使用者(consumer)之间的依赖过于紧密。因此我们接下来将使用 Feign 更优雅的执行服务发现和调用。

## 参考

<table>
  <tr>
    <td>微服务实战SpringCloud之Feign简介及使用</td>
    <td><a href="https://www.jianshu.com/p/8bca50cb11d8">https://www.jianshu.com/p/8bca50cb11d8</a></td>
  </tr>
  <tr>
    <td>Spring Cloud 入门教程(六)： 用声明式REST客户端Feign调用远端HTTP服务</td>
    <td><a href="https://www.cnblogs.com/chry/p/7266916.html">https://www.cnblogs.com/chry/p/7266916.html</a></td>
  </tr>
  <tr>
    <td></td>
    <td><a href=""></a></td>
  </tr>
</table>

## 完整示例代码

<a href="https://github.com/superfreeeee/Blog-code/tree/main/back_end/spring/spring_cloud_feign">https://github.com/superfreeeee/Blog-code/tree/main/back_end/spring/spring_cloud_feign</a>

# 正文

## Feign 简介

关于 Feign 服务的描述在参考资料中或是其他网上资源都有详细的解说，这边抓出几个比较核心的特点进行说明：

- 默认透过 Eureka Server/Client 进行服务注册（透过 Eureka Client 的实例名称查找对应服务，在后面的 `@FeignClient` 注解中用到）
- 默认使用 Ribbon 来进行负载均衡和调用（已经集成到 `openfeign` 依赖当中）
- 声明式服务调用，提供形似调用本地服务的开发体验

### 声明式服务调用（与 Ribbon 比较）

Feign 的一大特色就是声明式服务调用，对比于 Ribbon 的编程式实现，我们对两者进行比较

|                              |         Ribbon          |         Feign          |
| ---------------------------- | :---------------------: | :--------------------: |
| 使用对象                     |      RestTemplate       |      FeignService      |
| 请求路由、参数、返回类型定义 |        编程实现         |       声明为接口       |
| 调用特定服务                 | 统一使用 `xxxForObject` | 直接调用接口声明的方法 |

下面就是一种 Feign 声明式服务的实现样例（可以与前一篇的 Ribbon 做对比）

```java
@FeignClient("feign-service")
public interface FeignService {

    @GetMapping("/")
    public String defaultRoute();

    @GetMapping("/users")
    public Response getUsers();

    @PostMapping("/signup")
    public String signUp(@RequestBody User user);
}
```

接下来我们马上就来实践如何使用 Feign 来更优雅的实现微服务发现并调用

## 项目实战

接下来马上就写一个 demo 试试

### 项目抽象结构

老样子首先先给出我们这次要实现的项目结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/spring_cloud_feign_sample_structure.png)

系统中存在三种角色（括号内为服务名）：

- Eureka Server(eureka-server)：服务注册中心
- Feign Service(feign-service)：服务提供者
- Feign Client(feign-client)：服务使用者

Feign 使用体现在 feign-client 调用 feign-service 服务的时候，我们接着看

### 依赖版本 & 目录结构 & 服务接口说明

> Spring Boot & Spring Cloud 版本选用

- Spring Boot: `2.3.2.RELEASE`
- Spring Cloud: `Hoxton.SR6`

```xml
<!-- spring boot version -->
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>2.3.2.RELEASE</version>
  <relativePath/> <!-- lookup parent from repository -->
</parent>

<properties>
  <java.version>1.8</java.version>
  <!-- spring cloud version -->
  <spring-cloud.version>Hoxton.SR6</spring-cloud.version>
</properties>
```

下面给出项目目录结构和各微服务接口说明

> 目录结构

```
/spring-cloud-feign
├── eureka-server
│   ├── pom.xml
│   └── src
│       └── main
│           ├── java/com/example/server
│           │   └── EurekaServerApplication.java
│           └── resources
│               └── application.yml
├── feign-client
│   ├── pom.xml
│   └── src
│       └── main
│           ├── java/com/example/client
│           │   ├── ClientController.java
│           │   ├── FeignClientApplication.java
│           │   ├── FeignService.java
│           │   ├── Response.java
│           │   └── User.java
│           └── resources
│               └── application.yml
└── feign-service
    ├── pom.xml
    └── src
        └── main
            ├── java/com/example/service
            │   ├── FeignServiceApplication.java
            │   ├── Response.java
            │   ├── ServiceController.java
            │   └── User.java
            └── resources
                └── application.yml
```

> 接口说明

| 所属服务      | 接口路由 | 接口说明                          |
| ------------- | -------- | --------------------------------- |
| feign-service | /        | 默认路由                          |
| feign-service | /users   | 获取用户列表                      |
| feign-service | /signup  | 获取                              |
| feign-client  | /        | 使用 service 的默认路由           |
| feign-client  | /users   | 获取 feign-service 的用户列表     |
| feign-client  | /signup  | 代理 feign-service 的 signup 服务 |

### 关键配置

#### 各项目依赖(pom.xml)

首先给出各个项目的 pom.xml 关键配置和 application.yml 配置

- `eureka-server/pom.xml`

```xml
<!-- eureka server 依赖 -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

- `feign-service/pom.xml`

```xml
<!-- eureka client 依赖 -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

<!-- spring mvc 相关依赖 -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

- `feign-client/pom.xml`

```xml
<!-- eureka client 依赖 -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

<!-- feign 依赖 -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>

<!-- spring mvc 相关依赖 -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

#### 各项目运行配置(application.yml)

第二部分给出各个服务的项目配置 application.yml

- `eureka-server/src/main/resources/application.yml`

```yml
server:
  port: 8800

eureka:
  instance:
    appname: eureka-server
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:
      defaultZone: http://localhost:8800/eureka/

spring:
  application:
    name: eureka-server
```

- `feign-service/src/main/resources/application.yml`

```yml
server:
  port: 8801 # & 8802 透过运行参数 --server.port 指定端口

eureka:
  instance:
    appname: feign-service
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:8800/eureka/

spring:
  application:
    name: feign-service
```

- `feign-client/src/main/resources/application.yml`

```yml
server:
  port: 8803

eureka:
  instance:
    appname: feign-client
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:8800/eureka/

spring:
  application:
    name: feign-client

# feign 配置
feign:
  client:
    config:
      default:
        # 超时设置
        connectTimeout: 5000
        readTimeout: 5000
        loggerLevel: basic
```

接下来到关键的代码实现

### 关键代码实现

#### Eureka Server 服务注册中心

这个服务本身没啥重点，配置完了之后加上 `@EnableEurekaServer` 注解就完事了

- `EurekaServerApplication.java`：入口类

```java
@EnableEurekaServer // 加在这
@SpringBootApplication
public class EurekaServerApplication {
	public static void main(String[] args) {
		SpringApplication.run(EurekaServerApplication.class, args);
	}
}
```

#### Feign Service 服务提供者

虽然这边取名为 Feign Service，但是本质上就是单纯的服务提供着与 Feign 无关，所以实现也与前一篇 Ribbon 用到的类似

- `FeignServiceApplication.java`：入口类

```java
package com.example.service;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@EnableEurekaClient // 作为 Eureka Client 向服务中心注册服务
@SpringBootApplication
public class FeignServiceApplication {
	public static void main(String[] args) {
		SpringApplication.run(FeignServiceApplication.class, args);
	}
}
```

- `ServiceController.java`：控制层

```java
package com.example.service;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.Map;

@RestController
public class ServiceController {

    @Value("${server.port}")
    private Integer port;

    // 暂时保存用户信息，未实现持久化
    private Map<Integer, User> userTable = new HashMap<>();

    /* 一共提供三个接口 */
    @GetMapping("/")
    public String defaultRoute() {
        return "Default response from feign-service at " + port;
    }

    @GetMapping("/users")
    public Response getUsers() {
        return new Response("users at " + port, new ArrayList<>(userTable.values()));
    }

    @PostMapping("/signup")
    public String signUp(@RequestBody User user) {
        user.setId(userTable.size());
        userTable.put(user.getId(), user);
        return "Sign up user by id=" + user.getId() + " at port " + port;
    }
}
```

- `User.java`：实体类

```java
package com.example.service;

public class User {
    private Integer id;
    private String name;
    private String password;
    // getter & setter
}
```

- `Response.java`：请求结果封装

```java
package com.example.service;

public class Response {

    private String msg;
    private Object data;
    // getter & setter & full-params constructor
}
```

#### Feign Client 服务使用者（重点！）

终于到我们的主角了，也就是透过 Feign 的声明式自动查找并调用服务

- `FeignClientApplication.java`：入口类

```java
package com.example.client;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.cloud.openfeign.FeignClient;

@EnableFeignClients // 扫描 FeignClient 并自动注入 Bean
@EnableEurekaClient
@SpringBootApplication
public class FeignClientApplication {
	public static void main(String[] args) {
		SpringApplication.run(FeignClientApplication.class, args);
	}
}
```

- `FeignService.java`：声明式服务（重点！）

```java
package com.example.client;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;

@FeignClient("feign-service") // 给出服务名
public interface FeignService {

    // 服务声明为接口，会自动创建对应的 RestTemplate
    @GetMapping("/")
    public String defaultRoute();

    @GetMapping("/users")
    public Response getUsers();

    @PostMapping("/signup")
    public String signUp(@RequestBody User user);
}
```

- `User.java & Response.java`：实体类和返回结果封装，与 Service 中的一致，这边就不贴了

- `ClientController.java`：控制层

```java
package com.example.client;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
public class ClientController {

    // FeignService 声明接口之后就能透过 Autowired 自动装配 @EnableFeignClients 扫描时创建的 Bean 实例
    @Autowired
    private FeignService feignService;

    @Value("${server.port}")
    private Integer port;

    @GetMapping("/")
    public String proxyDefault() {
        return "[Client at " + port + "]" + feignService.defaultRoute();
    }

    @GetMapping("/users")
    public Response getUsers() {
        return feignService.getUsers();
    }

    @PostMapping("/signup")
    public String signup(@RequestBody User user) {
        return feignService.signUp(user);
    }
}
```

### 测试结果

1. 首先我们现依序启动各个服务：eureka-server、feign-service(at 8801)、feign-service(at 8802)、feign-client，并到 8800 查看服务启动和注册情形

![](https://picures.oss-cn-beijing.aliyuncs.com/img/spring_cloud_feign_sample_1.png)

2. 检查两个 feign-service 服务是否正常，并添加几个用户做示例（使用 `Apifox` 进行测试）

- 默认路由

![](https://picures.oss-cn-beijing.aliyuncs.com/img/spring_cloud_feign_sample_2.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/spring_cloud_feign_sample_3.png)

- 添加用户

![](https://picures.oss-cn-beijing.aliyuncs.com/img/spring_cloud_feign_sample_4.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/spring_cloud_feign_sample_5.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/spring_cloud_feign_sample_6.png)

- 查看当前用户列表

![](https://picures.oss-cn-beijing.aliyuncs.com/img/spring_cloud_feign_sample_7.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/spring_cloud_feign_sample_8.png)

3. feign-client 可以查看并调用两个 service 的方法（服务），这边仅仅以 /users 接口做示例

![](https://picures.oss-cn-beijing.aliyuncs.com/img/spring_cloud_feign_sample_9.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/spring_cloud_feign_sample_10.png)

# 结语

到此就搞定啦，相比于 Ribbon 粗暴的直接组合请求路由和传入返回类型，Feign 的声明式服务显得更优雅，也更能体现微服务服务发现的透明度，调用其他服务的方法就好像在调用项目自己内部的方法一般，这正是 Feign 最方便的地方。
