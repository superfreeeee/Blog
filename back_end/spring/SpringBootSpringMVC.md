# SpringBoot：SpringMVC Web服務端框架

@[TOC](文章目錄)

<!-- TOC -->

- [SpringBoot：SpringMVC Web 服務端框架](#springbootspringmvc-web-服務端框架)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Concept 概念](#concept-概念)
  - [Dependency 依賴說明](#dependency-依賴說明)
  - [起步](#起步)
    - [Project Structure 項目目錄結構](#project-structure-項目目錄結構)
    - [Configuration 配置項](#configuration-配置項)
    - [Application 應用入口](#application-應用入口)
    - [Controller 控制層](#controller-控制層)
    - [Service 服務層](#service-服務層)
    - [Dao 持久層](#dao-持久層)
    - [Run 啟動項目](#run-啟動項目)
- [結語](#結語)

<!-- /TOC -->

## 簡介

當今 Java 語言的框架中幾乎是 Spring 獨佔鱉頭，同時因為 SpringBoot 的出現使得 Spring Framework 令人詬病的繁瑣配置，以及從 XML 的 Bean 聲明方式轉變成使用注釋註冊 Bean，不僅配置更加油好靈活，也更貼合最原本的 Java 語言本身的寫法，對於開發者來說更為友善。

本篇將要介紹 SpringMVC 框架，儘管經過一些寫法上的包裝，其本質還是根據 SpringMVC 這個依賴進行開發，那麼現在就來看看到底該怎麼寫吧。

## 參考

<table>
  <tr>
    <td>spring-projects/spring-boot-github</td>
    <td><a href="https://github.com/spring-projects/spring-boot/blob/v2.2.7.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-web/pom.xml">https://github.com/spring-projects/spring-boot/blob/v2.2.7.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-web/pom.xml</a></td>
  </tr>
  <tr>
    <td>spring-Boot之spring-boot-starter</td>
    <td><a href="https://blog.csdn.net/zhou920786312/article/details/84324915">https://blog.csdn.net/zhou920786312/article/details/84324915</a></td>
  </tr>
</table>

# 正文

## Concept 概念

首先先來介紹一下 SpringMVC 的抽象概念，作為一個分層模式的體系結構，SpringMVC 將整個服務端系統分成三層：

1. Controller 控制層：控制請求的收發
2. Service 服務層：主要邏輯業務
3. Dao 持久層：系統和數據庫交互的接口

類圖表示如下：

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/layers.png)

其中值得注意的一點是，由於實際業務邏輯可能會出現多個服務層模塊間的複雜依賴關係，所以抽象出一個接口，使得控制層和服務層實現都依賴於 Service 接口，從而實現面對對象的依賴倒置原則(DIP)，並且真正組合業務層實例間的依賴關係可以藉由 SpringBoot 提供的 `@Autowired` 注釋，背後原理為 Spring Framwork 的`依賴注入(DI)`，是 Spring 的核心技術之一，有興趣的可以查閱相關資料。

## Dependency 依賴說明

在開始使用之前，我們應該要先了解我們到底引入了哪些包。本篇博客使用 maven 作為依賴管理器，配置文件為根目錄下的 `pom.xml`，使用的技術為 `SpringMVC` + `Mybatis` + `MySQL`。

本篇的所使用的主要依賴說明如下：

1. `spring-boot-starter-web`：Web 開發主要依賴
2. `mybatis-spring-boot-starter`：使用 mybatis 作為本篇的 ORM
3. `mysql-connector-java`：使用 mysql 連接池
4. `spring-boot-starter-test`：JUnit 測試依賴

配置文件內容如下：

- pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <!-- 版本控制依賴 -->
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.0.RELEASE</version>
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
    <!-- web 開發依賴，包含 mvc -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- mybatis 相關依賴 -->
    <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-starter</artifactId>
      <version>2.1.3</version>
    </dependency>
    <!-- mysql 相關依賴 -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <scope>runtime</scope>
    </dependency>
    <!-- JUnit 相關依賴 -->
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

其中 `<parent>` 裡面引入了 `sprint-boot-start-parent` 依賴，作為 SpringBoot 項目的`統一版本依賴`，這樣的好處是下面引入的 Spring 相關依賴都不需要自己寫版本號了，也更利於版本統一管理。

而上面第二重要的是引入了 `spring-boot-start-web`，作為本項目的核心，我們要先來瞭接一下這個依賴裡面都包含了哪些子依賴，以下是 github 上對應版本(本篇使用 2.2.7RELEASE，各版本差異可自行查閱相關文檔)的 `pom.xml` 中的主要依賴：

- spring-booot-start-web 2.2.7RELEASE - pom.xml

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-json</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
    <exclusions>
      <exclusion>
        <groupId>org.apache.tomcat.embed</groupId>
        <artifactId>tomcat-embed-el</artifactId>
      </exclusion>
    </exclusions>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
  </dependency>
</dependencies>
```

稍微介紹一下其中主要的依賴：

- `spring-boot-starter`：用於 SpringBoot 啟動的相關依賴
- `spring-boot-starter-json`：用於 RESTFul 風格時，將對象序列化的相關依賴
- `spring-boot-starter-tomcat`：作為虛擬機的容器，內置的 tomcat 依賴
- `spring-boot-starter-web`、`spring-boot-starter-webmvc`：各層註解，以及 web 服務端開發會用到的註解的相關依賴

Spring 項目都是透過這種方式多層配置各種依賴，最後暴露出 `spirng-boot-starter-xxx` 作為單一依賴接口，善加選擇便能夠輕易的以簡潔的配置文件引入所有需要的依賴。

這邊就不遞歸深究每一個依賴了，有興趣的可以自己查文檔，接下來就開始建立我們的項目吧！

## 起步

創建 SpringBoot 項目的方法在我的<a href="https://blog.csdn.net/weixin_44691608/article/details/106079234">上一篇：SpringBoot 項目啟動</a>有介紹，這邊就不重複了

### Project Structure 項目目錄結構

首先我們先建立如下目錄結構：

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/structure.png)

- 在 `com.example.demo` 目錄下：
  - 創建 `/controller/DemoController.java`
  - 創建 `/service/DemoService.java`
  - 創建 `/service/Impl/DemoServiceImpl.java`
  - 創建 `/dao/DemoMapper.java`
  - 創建 `/vo/ResponseVO.java`
  - 創建 `/vo/UserVO.java`
  - 創建 `/po/User.java`
- 在 `resources` 目錄下
  - 創建 `/mapper/DemoMapper.xml`
  - 創建 `/sql/spring_demo.sql`
  - 將 `application.properties` 改名為 `application.yml`

### Configuration 配置項

首先我們先修改 `application.yml` 中的配置如下：

- application.yml

```yml
spring:
  datasource:
    # 數據庫的路徑，mysql 默認端口號為 3306，本項目使用的數據庫名稱為 spring_demo
    url: jdbc:mysql://localhost:3306/spring_demo
    username: root # 改成自己的數據庫使用者名稱
    password: root # 改成自己的密碼

mybatis:
  # 配置 Mybatis 的配置文件位置
  mapper-locations: classpath:mapper/*Mapper.xml
```

### Application 應用入口

接下來就可以開始編寫我們的服務端代碼了，首先我們建立 Spring 項目的時候，就會存在一個 `DemoApplication.java` 類作為我們的應用入口，這時候我們需要加一個用於掃描映射文件的註解，其中參數傳入 `dao` 包的路徑 `com.example.demo.dao`：

- DemoApplication.java

```java
package com.example.demo;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@MapperScan("com.example.demo.dao")
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}
```

### Controller 控制層

接下來跳到我們的控制層(Controller)，這層的 Bean 主要負責接收外部發送的請求，並進行解析轉發任務，最終回覆請求者。由於我們使用 RESTful 風格進行開發，所以統一返回一個 `ResponseVO` 類型的對象，定義如下：

- ResponseVO.java

```java
package com.example.demo.vo;

public class ResponseVO {
    private Boolean success;
    private String message;
    private Object content;

    public static ResponseVO buildSimple(Boolean success) {
        ResponseVO res = new ResponseVO();
        res.setSuccess(success);
        return res;
    }

    public static ResponseVO buildSuccess(Object content) {
        ResponseVO res = new ResponseVO();
        res.setSuccess(true);
        res.setContent(content);
        return res;
    }

    public static ResponseVO buildFailure(String message) {
        ResponseVO res = new ResponseVO();
        res.setSuccess(false);
        res.setMessage(message);
        return res;
    }
    // 以下需要添加所有屬性的 getter 和 setter 這邊就不列出來了
}
```

接下來我們來添加 Controller 類的具體內容：

- DemoController.java

```java
package com.example.demo.controller;

import com.example.demo.service.DemoService;
import com.example.demo.vo.ResponseVO;
import com.example.demo.vo.UserVO;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/demo/api")
public class DemoController {

    @Autowired
    DemoService demoService;

    @PostMapping("/user")
    public ResponseVO addUser(@RequestBody UserVO userVO) {
        boolean res = demoService.addUser(userVO);
        return ResponseVO.buildSimple(res);
    }

    @GetMapping("/user/{userId}")
    public ResponseVO getUserInfo(@PathVariable("userId") Integer userId) {
        UserVO userVO = demoService.getUser(userId);
        if(userVO == null) {
            return ResponseVO.buildSimple(false);
        } else {
            return ResponseVO.buildSuccess(userVO);
        }
    }
}
```

可以看到 Controller 用到了 `ResponseVO` 作為返回的對象以及 `UserVO` 作為傳入的參數，以下為 `UserVO` 的定義：

- UserVO.java

```java
package com.example.demo.vo;

import com.example.demo.po.User;

public class UserVO {
    private Integer userId;
    private String name;
    private String password;

    public static UserVO fromUser(User user) {
        UserVO userVO = new UserVO();
        userVO.userId = user.getUserId();
        userVO.name = user.getName();
        userVO.password = user.getPassword();
        return userVO;
    }
    // 同樣，以下為 getter/setter
}
```

到此我們已經完成了控制層的定義，我們可以看到 `DemoController.java` 將主要的業務邏輯分發給 `DemoService.java`，並且使用 SpringBoot 提供的 `@Autowired` 註解自動注入依賴，並對 Service 層返回的結果經處理後包裝成 ResponseVO 返回。

### Service 服務層

有了控制層作為請求的攔截，負責請求的接收和回覆，`服務層(Service)`就能夠專注於構造服務層的接口，並且由控制層來調用服務層提供的接口完成任務。

由於服務層的不同模塊之間可能出現複雜的相互依賴關係，所以這邊通常會構造出一個 Service 接口層，將真正的依賴關係以組合的形式延遲到實現(`Impl`)時才注入，便能實現依賴倒置原則(DIP)

`DemoService.java` 接口以及 `DemoServiceImpl.java` 實現代碼如下：

- DemoService.java

```java
package com.example.demo.service;

import com.example.demo.vo.UserVO;

public interface DemoService {

    boolean addUser(UserVO userVO);

    UserVO getUser(Integer userId);
}
```

- DemoServiceImpl.java

```java
package com.example.demo.service.Impl;

import com.example.demo.dao.DemoMapper;
import com.example.demo.po.User;
import com.example.demo.service.DemoService;
import com.example.demo.vo.UserVO;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class DemoServiceImpl implements DemoService {

    @Autowired
    DemoMapper demoMapper;

    @Override
    public boolean addUser(UserVO userVO) {
        User user = User.fromUserVO(userVO);
        try {
            int res = demoMapper.insertUser(user);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    @Override
    public UserVO getUser(Integer userId) {
        try {
            User user = demoMapper.selectUserById(userId);
            UserVO userVO = UserVO.fromUser(user);
            return userVO;
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
}
```

由 `DemoService.java` 來向外聲明服務層的接口，並由 `DemoServiceImpl.java` 實現並組合相關依賴的模塊(`DemoMapper`、其他服務層模塊等)

### Dao 持久層

控制層負責消息的轉發，服務層為實現主要業務邏輯，接下來就是`持久層(Dao)`負責實現與數據庫的連接。本篇使用 Mybatis + MySQL 的組合，其他 ORM 選擇還有 Hibernate、spring JDBC 等，數據庫也有多種選擇如 Oracle。

持久層最主要的意義在於將業務邏輯與數據庫交互細節分離：向上(Service)，服務層調用持久層提供的接口來操作數據庫，向下(database)由 Mybatis 配置負責定義各接口對於數據庫操作的真正實現。

首先我們需要定義 `DemoMapper.java` 作為持久層的接口聲明：

- DemoMapper.java

```java
package com.example.demo.dao;

import com.example.demo.po.User;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;
import org.springframework.stereotype.Repository;

@Mapper
@Repository
public interface DemoMapper {

    int insertUser(User user);

    User selectUserById(@Param("userId") Integer userId);
}
```

由於 Mybatis 有兩種配置方式：

1. 將 sql 語句作為註解寫在 Mapper 類當中，適合小型或快速開發的項目
2. 獨立出一個 `xxxMapper.xml` 作為 Mybatis 的配置文件，集中聲明 sql 語句，適合較大型項目的開發，並且統一管理以及可擴展性表現更佳

這邊使用 `DemoMapper.xml` 作為獨立的配置文件：

- DemoMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.demo.dao.DemoMapper">

    <insert id="insertUser" parameterType="com.example.demo.po.User" useGeneratedKeys="true" keyProperty="userId">
        INSERT INTO user (name, password) VALUES (#{name}, #{password})
    </insert>

    <select id="selectUserById" resultMap="User">
        SELECT * FROM user WHERE user_id=#{userId}
    </select>


    <resultMap id="User" type="com.example.demo.po.User">
        <result column="user_id" property="userId"></result>
        <result column="name" property="name"></result>
        <result column="password" property="password"></result>
    </resultMap>
</mapper>
```

並且建立 `spring_demo.sql` 管理我們的數據庫表結構：

- spring_demo.sql

```sql
DROP TABLE IF EXISTS user;

CREATE TABLE user
(
	user_id int auto_increment,
	name varchar(20) NOT NULL,
	password varchar(20) NOT NULL ,
	PRIMARY KEY (user_id)
);
```

### Run 啟動項目

到此我們已經完全構建我們的簡易項目了，這邊使用 Postman 作為接口測試工具，接下來附上一些運行時，以及兩個接口的測試截圖：

- 啟動項目

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/runtime.png)

- 測試 addUser 接口

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/test1.png)

- 測試 getUserInfo 接口

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/test2.png)

# 結語

本篇使用 SpringMVC + MyBatis + MySQL 的框架構建項目，主要演示了最簡單最簡單的使用方式，真正寫項目的時候基本就是照這個思路不斷的擴展功能。其中每一層的用法細節會在後面的篇章單獨剝離出來詳細解說。SpringBoot 作為一個擴展性良好的框架，其強大的依賴注入的能力能夠輕易的將眾多第三方依賴庫中的類作為 Bean 注入到框架內，以極少的配置實現依賴注入，接下來作者也會寫幾篇 SpringBoot 整合其他庫的使用介紹。
