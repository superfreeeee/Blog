# SpringBoot + Web 項目啟動

@[TOC](文章目錄)

<!-- TOC -->

- [SpringBoot + Web 項目啟動](#springboot--web-項目啟動)
    - [簡介](#簡介)
    - [參考](#參考)
- [正文](#正文)
    - [Creation 創建項目](#creation-創建項目)
        - [by Spring Initializr](#by-spring-initializr)
        - [by IDEA](#by-idea)
        - [Project Structure 項目結構](#project-structure-項目結構)
    - [SpringWeb HelloWorld](#springweb-helloworld)
        - [Overview 總覽](#overview-總覽)
        - [ResponseVO.java](#responsevojava)
        - [DemoController.java](#democontrollerjava)
        - [DemoApplication.java](#demoapplicationjava)
        - [Sample](#sample)
- [結語](#結語)

<!-- /TOC -->

## 簡介

Spring Framework 作為基於 Java(Kotlin, Groovy 也能開發)的後端框架，本篇主要來介紹如何創建一個 web 後端服務器的搭建，詳情可以到官方查看相關教程（參考一鏈結）。

## 參考

<table>
    <tr>
        <td>Getting Started</td>
        <td><a href="https://spring.io/guides/gs/spring-boot/">https://spring.io/guides/gs/spring-boot/</a></td>
    </tr>
</table>

# 正文

本篇介紹兩個創建 SpringBoot + web 項目的入口，其實本質上是同一個

## Creation 創建項目

創建項目的入口可以從 Spring 官方的網頁版 Spring Initializr 創建，也有透過 IDEA 的入口進行創建。

項目創建主要就是建立一個 maven 項目(其他篇詳細說明)，然後自動引入 springboot-starter 依賴，再加上我們選擇的 spring web 依賴，

### by Spring Initializr

Spring 官方提供的項目創建  入口：<a href="https://start.spring.io/">https://start.spring.io/</a>

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/spring_initializr.png)

### by IDEA

IDEA 官網下載：<a href="https://www.jetbrains.com/idea/">https://www.jetbrains.com/idea/</a>

1. 打開 IDEA，選擇 `Create New Project`

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/idea1.png)

2. 選擇 Spring Initializr，本質上 IDEA 也是透過官方提供的 Spring Initializr 進行創建

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/idea2.png)

3. 填寫項目描述和配置（Group, Artifact, version，基本上就是 maven 項目配置）

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/idea3.png)

4. 選擇 Spring 相關依賴，本次項目選擇 `Spring Web`，其他篇將介紹其他常用依賴選項。

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/idea4.png)

5. 最後選擇項目創建路徑就創建成功了

### Project Structure 項目結構

這邊只展示出核心文件和項目架構

```tree
.
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── example
    │   │           └── demo
    │   │               └── DemoApplication.java
    │   └── resources
    │       └── application.properties
    └── test
        └── java
            └── com
                └── example
                    └── demo
                        └── DemoApplicationTests.java
```

- pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.7.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

一個 maven 項目的依賴版本定義寫在 `pom.xml` 裡面，而源代碼則寫在 `src` 裡面，`com/example/demo` 則作為項目代碼的根目錄，`DemoApplication.java` 為啟動類

## SpringWeb HelloWorld

Spring 作為一個成熟的框架提供很多能力與配置選項，而本篇文章介紹的 SpringBoot 使得配置輕量化，大量的配置默認化使 SprintBoot 項目能開箱即用。

這裡簡單介紹 SpringMVC 項目的 HelloWorld 等級代碼，更多的妙用請查看其他成熟的 Spring 項目或是其他篇的教學介紹常用依賴。

### Overview 總覽

SpringMVC 的思想是分層式的體系結構，大致上可分為三層：

1. Controller 控制層：控制 http 請求的分流，為每一個請求 url 對應一個方法
2. Service 服務層：為了符合"高內聚，低耦合"的核心思想，通常 service 層的實現會分為接口定義(Service)和實現類(ServiceImpl)
3. DAO(Data Access Object) 數據層：為 Service 邏輯和數據庫連接的中間層，透過其他數據庫相關的依賴(JDBC、Hibernate、Mybatis)可以抽象出數據庫訪問的數據接口，甚至客製化 SQL 語句。SpringBoot 項目就可以從數據庫中解耦出來，透過 java 對象和方法的使用來操作數據庫。

本篇只需要使用到控制層，有關服務層和數據層的使用詳見後面幾篇：

### ResponseVO.java

我們先來定義返回對象的格式，在 `com/example/demo` 根目錄下創建 `vo` 包，並創建 `ResponseVO.java`：

```java
package com.example.demo.vo;

public class ResponseVO {

    private Boolean success;

    private String message;

    private Object content;

    public static ResponseVO buildSuccess() {
        ResponseVO responseVO = new ResponseVO();
        responseVO.setMessage("build success");
        return responseVO;
    }

    public static ResponseVO buildSuccess(Object content) {
        ResponseVO responseVO = new ResponseVO();
        responseVO.setContent(content);
        responseVO.setMessage("build success");
        return responseVO;
    }
}
```

- 說明：此處省略 getter/setter 函數，但是由於序列化需要，至少必須實現所有成員變量的 getter 方法

### DemoController.java

我們定義好要返回的數據類型之後，開始構建控制層的代碼：

在 `com/example/demo` 根目錄下創建 `controller` 包，並創建 `DemoController.java`：

```java
package com.example.demo.controller;

import com.example.demo.vo.ResponseVO;
import org.springframework.web.bind.annotation.*;

@RestController
public class DemoController {

    @GetMapping("/hello")
    public ResponseVO hello() {
        return ResponseVO.buildSuccess("Hello Worl");
    }
}
```

在這邊我們定義了一個 url 為 `/hello` 的接口，將返回一個 `ResponseVO` 對象，當響應請求的時候 ResponseVO 將會被序列化成一個 JSON 對象放入 Response body 裡面。

### DemoApplication.java

在項目創建完成的時候就已經存在 `com/example/demo` 目錄下的 `DemoApplication.java` 便是整個 SpringBoot 項目的入口類，不需要做任何修改。

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

### Sample

添加完以上代碼之後，啟動 `DemoApplication.java`，並到瀏覽器輸入 url 發送請求：

```bash
http://localhost:8080/hello
```

這時瀏覽器就會返回經由 Controller 層接受請求並返回的序列化後的 ResponseVO 對象：

```json
{ "success": null, "message": "build success", "content": "Hello World" }
```

# 結語

到此我們已經能夠創建最基礎的 SpringBoot 項目了！可以說是超級簡單，開箱即用。本篇僅簡單介紹 SpringBoot 項目的創建和 SpringMVC 的簡單代碼，有關 Spring 的構造原理（IOC、DI）等會在其他篇介紹。後面我們還會介紹更多中間件和數據庫與 SpringBoot 的集成（如 Mybatis、Redis、Spring Security、Spring JDBC 等），後續 SpringCloud 也是基於 SpringBoot 的基礎上進行開發。
