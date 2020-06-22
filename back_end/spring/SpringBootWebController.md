# SpringBoot：Web - Controller Annotations 控制層常用注解

@[TOC](文章目錄)

<!-- TOC -->

- [SpringBoot：Web - Controller Annotations 控制層常用注解](#springbootweb---controller-annotations-控制層常用注解)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Overview 概述](#overview-概述)
  - [`@RestController` = `@Controller` + `@ResponseBody`](#restcontroller--controller--responsebody)
  - [`@RequestMapping`](#requestmapping)
    - [1. `value`（默認）路由匹配模式](#1-value默認路由匹配模式)
    - [2. `method` 請求方法](#2-method-請求方法)
    - [3. `consumes` 請求內容類型、`produces` 返回內容類型](#3-consumes-請求內容類型produces-返回內容類型)
    - [4. `params` 請求參數、`headers` 請求頭](#4-params-請求參數headers-請求頭)
  - [請求參數解析](#請求參數解析)
    - [`@PathVariable`](#pathvariable)
    - [`@RequestParam`](#requestparam)
    - [`@RequestBody`](#requestbody)
  - [自動注入依賴 `@Autowired`](#自動注入依賴-autowired)
- [結語](#結語)

<!-- /TOC -->

## 簡介

<a href="https://blog.csdn.net/weixin_44691608/article/details/106692801">前一篇</a>

<a href="https://blog.csdn.net/weixin_44691608/article/details/106692801">前一篇</a>介紹了 SpringBoot 的 Web 開發項目基礎架構。接下來幾篇將一層層分別介紹，本篇主要介紹 `Controller` 控制層的常用注解的使用

## 參考

<table>
  <tr>
    <td>Spring Web Annotations</td>
    <td><a href="https://www.baeldung.com/spring-mvc-annotations">https://www.baeldung.com/spring-mvc-annotations</a></td>
  </tr>
  <tr>
    <td>Spring web-官方</td>
    <td><a href="https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-controller">https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-controller</a></td>
  </tr>
</table>

# 正文

本篇主要講解各個注解的用法，配置的加載和依賴注入等將不在這裡說明原理和運行機制

## Overview 概述

首先列出我們將要介紹的所有控制層會用到的注解（更多更完整的解釋請移步參考二鏈接：<a href="https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-controller">Spring web-官方</a>）

- `@RestController`
  - `@Controller`
  - `@ResponseBody`
- `@RequestMapping`
  - `@GetMapping`
  - `@PostMapping`
  - `@PutMapping`
  - `@DeleteMapping`
- 參數映射
  - `@PathVariable`
  - `@RequestParam`
  - `@RequestBody`
- `@Autowired` 依賴注入

## `@RestController` = `@Controller` + `@ResponseBody`

由於 Spring 的核心架構是一個 IOC Container，其他所有類實例都作為 Bean 被註冊到容器(Container)中。

`@Controller` 是 `@Component` 的擴展，代表 Spring 將此類作為`控制器類`解析和注入。

而 `@ResponseBody` 的作用是將返回的對象解析成 JSON 格式並寫入到 Http response 的 ResponseBody 中。這個注解是因應 RESTful 風格的服務端架構，由於 RESTful API 將只返回 JSON 格式的對象，所以透過 `@ResponseBody` 可以自動將返回對象透過 jackson 包解析成 JSON 對象並寫入 ResponseBody。

以下為使用範例：

- DemoController.java

```java
@RestController
public class DemoController {}
```

等價於下面兩種形式：

```java
@Controller
@RequestBody
public class DemoController {}
```

```java
@Controller
public class DemoController {

  @RequestBody
  public Object someMethod() {}
}
```

## `@RequestMapping`

然而只有 `@Controller` 來表示控制器類還不夠，我們還需要使用 `@RequestMapping` 真正將請求路由一一映射到處理方法上

`@RequestMapping` 有六個參數，使用時寫在方法上，如果寫在類上將作為父路徑，而其他參數作為所有路由的默認匹配

### 1. `value`（默認）路由匹配模式

匹配規則如下：

| Pattern       | Description                               | Example                                                                                            |
| ------------- | ----------------------------------------- | -------------------------------------------------------------------------------------------------- |
| ?             | 匹配單個任意字符                          | `/demo/t?st.html`<br/>匹配<br/>`/demo/test.html`<br/>`/demo/t3st.html`                             |
| \*            | 匹配多個任意字符                          | `/demo/*.html`<br/>匹配<br/>`/demo/main.html`<br/>`/demo/test.html`                                |
| \*\*          | 匹配多個路徑段(以 `/` 分段)               | `/resources/**`<br/>匹配<br/>`/resources/static/js/index.js`<br/>`/resources/static/css/index.css` |
| {name}        | 匹配路徑參數（將在 `@PathVariable` 解釋） | `/demo/api/user/{id}`                                                                              |
| {name:regexp} | 可做正則表達式限定                        | `/demo/api/user/{id:[0-9]+}`                                                                       |

- example

```java
@RestController
@RequestMapping("/demo/api")  // `/demo/api` 作為公路路由，也就是父級路徑
public class DemoController {
    // 匹配路由：`/demo/api/user`
    @RequestMapping("/user")
    public ResponseVO addUser() {}

    // 匹配路由 `/demo/123`、`/demo/456`
    @RequestMapping("/user/{userId:[0-9]+}")
    public ResponseVO getUserInfo(@PathVariable Integer userId) {}
}
```

### 2. `method` 請求方法

支持所有 Http 的請求方法，常用的有 `GET`、`POST`、`PUT`、`DELETE`。同時支持 `@xxxMapping` 作為 `@RequestMapping(method = RequestMethod.xxx)` 的縮寫

- example

```java
@RestController
@RequestMapping("/demo/api")
public class DemoController {
    // POST 方法
    @RequestMapping("/user", method = RequestMethod.POST)
    // 等價於 @PostMapping("/user")
    public ResponseVO addUser() {}

    // GET 方法
    @GetMapping("/user/{userId:[0-9]+}")
    public ResponseVO getUserInfo(@PathVariable Integer userId) {}
}
```

### 3. `consumes` 請求內容類型、`produces` 返回內容類型

`consumes` 對應了 Http Request 中的 `Content-Type`，如：`application/json`、`text/html`；而 `produces` 指定返回內容類型

### 4. `params` 請求參數、`headers` 請求頭

必須有指定`請求參數`(url 中 `?` 附帶的參數)和`請求頭(headers)`才匹配請求

## 請求參數解析

Http 請求時後的參數主要可以有三種類型：

1. 寫在於路由中間的，使用 `@PathVariable`，例如：`/demo/api/user/123`
2. 作為請求參數，置於 `?` 後，使用 `@RequestParam`，例如：`/demo/api/user?userId=123`
3. 寫在 RequestBody 中，使用 `@RequestBody`

### `@PathVariable`

路由中間的參數，也就是前面提到的在 `@RequestMapping` 中使用 `{name}` 作為參數名。注解接受兩個參數：

1. `value`（默認）、`name`：變量名，對應 `{name}` 中的 name
2. `required`：指定必須參數，默認為 `true`

- example

```java
@RestController
@RequestMapping("/demo/api")
public class DemoController {
    // PathVariable 變量名與默認參數名不同時可缺省
    @GetMapping("/user/{userId:[0-9]+}")
    public ResponseVO getUserInfo(@PathVariable Integer userId) {}
    // groupId 為非必須參數
    @GetMapping("/allUser/{groupId:[0-9]+}")
    public ResposneVO updateUser(@PathVariable(required = false) Integer groupId)
}
```

### `@RequestParam`

請求附帶參數，也就是 `?` 後面的附帶的參數。注解接受三個參數：

1. `value`（默認）、`name`：變量名，對應 `?key=value` 中的 key
2. `required`：指定必須參數，默認為 `true`
3. `defaultValue`：指定默認值

- example

```java
@RestController
@RequestMapping("/demo/api")
public class DemoController {
    // 匹配請求路由：/demo/api/user?userId=123
    @GetMapping("/user")
    public ResponseVO getUserInfo(@RequestParam Integer userId) {}
    // 參數可缺省時啟用默認值
    @GetMapping("/allUser")
    public ResposneVO updateUser(@RequestParam(required = false, defaultValue = "123") Integer groupId)
}
```

### `@RequestBody`

參數寫在請求主體(RequestBody)中。只有一個參數：

1. `required`：指定必須參數，默認為 `true`

- example

```java
@RestController
@RequestMapping("/demo/api")
public class DemoController {
    // RequestBody 中的 JSON 對象將被映射為 UserVO 類
    @PostMapping("/user")
    public ResponseVO addUser(@RequestBody UserVO userVO) {}
    // 非必須參數
    @PutMapping("/user")
    public ResponseVO updateUser(@RequestBody(required = false) UserVO userVO) {}
}
```

## 自動注入依賴 `@Autowired`

`Controller` 類作為控制層，勢必要依賴於對應服務層類，我們可以使用 `@Autowired` 讓 SpringBoot 自動為我們注入依賴

```java
@RestController
@RequestMapping("/demo/api")
public class DemoController {

    // 自動注入服務層類（需加上 @Service 注解）
    @Autowired
    DemoService demoService;

    @PostMapping("/user")
    public ResponseVO addUser(@RequestBody UserVO userVO) {
        boolean res = demoService.addUser(userVO);
        return ResponseVO.buildSimple(res);
    }

    @GetMapping("/user/{userId}")
    public ResponseVO getUserInfo(@PathVariable Integer userId) {
        UserVO userVO = demoService.getUser(userId);
        if(userVO == null) {
            return ResponseVO.buildSimple(false);
        } else {
            return ResponseVO.buildSuccess(userVO);
        }
    }
}
```

# 結語

本篇簡單總結 Spring Web 開發中 Controller 控制層常用的注解，其他還有異常處理 `@ExceptionHandler`、定時服務 `@EnableScheduling` 等其他功能，將留到後面幾篇介紹。
