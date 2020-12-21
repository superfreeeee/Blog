# Java 基础: Annotation 注解说明(Spring 建设基础)

@[TOC](文章目录)

<!-- TOC -->

- [Java 基础: Annotation 注解说明(Spring 建设基础)](#java-基础-annotation-注解说明spring-建设基础)
  - [简介](#简介)
  - [参考](#参考)
  - [完整示例代码](#完整示例代码)
- [正文](#正文)
  - [注解的作用](#注解的作用)
  - [注解类型结构](#注解类型结构)
    - [Annotation 注解类型](#annotation-注解类型)
    - [RetentionPolicy 生命周期](#retentionpolicy-生命周期)
    - [ElementType 可作用类型](#elementtype-可作用类型)
  - [内置注解](#内置注解)
    - [内置注解总览](#内置注解总览)
    - [`@Documented`](#documented)
    - [`@Retention`](#retention)
    - [`@Target`](#target)
    - [`@Inherited`](#inherited)
  - [自定义注解](#自定义注解)
    - [自定义注解语法](#自定义注解语法)
    - [自定义注解应用](#自定义注解应用)
      - [一般注解](#一般注解)
      - [带属性注解](#带属性注解)
      - [配合注解自动解析路由](#配合注解自动解析路由)
- [结语](#结语)

<!-- /TOC -->

## 简介

本篇是受到 Spring Boot 的启发。在 Spring 的发展历程中 Spring Boot 的出现使得开发者能从繁杂的 xml 配置中解放出来，透过 Java 原生的注解功能给予开发者全 Java 的优雅体验。

然而 Java 原生的注解所提供的功能其实是非常简单的，就跟名字的字面意义一样，`注解(Annotation)`本身就只是`附加信息`的作用仅此而已。那些解析、Bean 注册、DI 等额外功能只是基于注解解析所达成的额外功能。接下来马上就来看看我们要怎么使用注解吧。

## 参考

<table>
  <tr>
    <td>Java 注解（Annotation）</td>
    <td><a href="https://www.runoob.com/w3cnote/java-annotation.html">https://www.runoob.com/w3cnote/java-annotation.html</a></td>
  </tr>
  <tr>
    <td>Java注释@interface的用法</td>
    <td><a href="https://www.cnblogs.com/liaojie970/p/7879917.html">https://www.cnblogs.com/liaojie970/p/7879917.html</a></td>
  </tr>
  <tr>
    <td>SpringBoot中的@AliasFor注解介绍</td>
    <td><a href="https://blog.csdn.net/u012043390/article/details/89391518">https://blog.csdn.net/u012043390/article/details/89391518</a></td>
  </tr>
</table>

## 完整示例代码

<a href=""></a>

# 正文

## 注解的作用

注解 Annotation 在 Java 源代码中扮演的角色仅仅只是`附加信息`的作用，这点非常重要，再次强调。

下面像是 `RetentionPolicy 生命周期`指的仅仅只是这些信息能够存活的周期，而`不能主动执行任何操作`，这点必须铭记在心。

## 注解类型结构

首先我们先来看看注解的类型结构

![](https://picures.oss-cn-beijing.aliyuncs.com/img/java_annotation_type_structure.png)

每个注解会实现 `Annotation` 接口，而每个 Annotation 会包含一个 `RetentionPolicy(生命周期)` 和多个 `ElementType(可作用类型)`

### Annotation 注解类型

首先来看看注解的定义结构

> Annotation.java

```java
package java.lang.annotation;

public interface Annotation {
    boolean equals(Object obj);
    int hashCode();
    String toString();
    Class<? extends Annotation> annotationType();
}
```

跟 Object 几乎一致。下面我们会提到，我们使用 `class` 关键字声明一个类，声明注解则要使用 `@interface` 关键字，`annotationType` 则会返回注解实际定义的类型(即 `? extends Annotation` 的 `?`)

### RetentionPolicy 生命周期

一个注解会有一个唯一的 `RetentionPolicy 生命周期`，用于指定注解存活(作用)的周期，有以下可选值：

> RetentionPolicy.java

```java
package java.lang.annotation;

public enum RetentionPolicy {
    SOURCE, // 源码级别，仅仅作用于编译期
    CLASS, // 字节码级别，会被编译写入到 .class 文件内，但不会在运行时使用到
    RUNTIME // 运行时级别，能在运行时动态的被 JVM 查询
}
```

再次复习，`注解`的作用仅仅只是`附带信息`，并不能执行任何操作，**注解起到的作用都是由其他部件解析特定注解时所进行的特定操作**

在自定义注解的时候我们可以透过加上 `@Retention(RetentionPolicy.XXX)` 来指定自定义注解的生命周期

### ElementType 可作用类型

最后一个成员是 `ElementType 可作用类型`。在 Java 中注解能够修饰的成员被归类为下列几种：

> ElementType.java

```java
package java.lang.annotation;

public enum ElementType {
    TYPE, // 类型，可以是 class, interface, enum, annotation 等
    FIELD, // 字段（成员变量）
    METHOD, // 方法
    PARAMETER, // 参数
    CONSTRUCTOR, // 构造函数
    LOCAL_VARIABLE, // 局部变量
    ANNOTATION_TYPE, // 注解类型专用
    PACKAGE, // 包
    TYPE_PARAMETER, // 类型参数（范型）
    TYPE_USE // 类型使用（实际类型名称）
}
```

在实际自定义的时候我们可以使用 `@Target(ElementType.XXX)` 或是 `@Target({ElementType.XXX, ...})` 指定多个目标


## 内置注解

下面我们稍微介绍一下注解，以及几个接下来我们自定义注解是要用到的内置注解

### 内置注解总览

下面列出所有 Java 内置提供的注解和说明

| Annotation           | Usage                                            | Version      |
| -------------------- | ------------------------------------------------ | ------------ |
| @Deprecated          | 标记方法为过时的方法                             |              |
| @Override            | 标记方法为重写(Override)方法，编译期检查         |              |
| @Documented          | 标记注解是否包含到文档中                         |              |
| @SuppressWarnings    | 标记为可忽略的警告                               |              |
| @Retention           | 指定注解的生命周期(RetentionPolicy)              |              |
| @Target              | 指定注解的可作用目标(ElementType)                |              |
| @Inherited           | 标记注解为可继承注解(即子类也会带有父类的该注解) |              |
| @SafeVarargs         | 忽略参数为范型变量的警告                         | since Java 7 |
| @FunctionalInterface | 标记接口为函数式接口(只存在唯一方法)             | since Java 8 |
| @Repeatable          | 标记注解为可重用注解(对同一目标重复使用)         | since Java 8 |

下面我们介绍通常自定义注解时会用到的四个注解定义。这四个注解的`可作用对象(ElementType)`都是`ElementType.ANNOTATION_TYPE 注解类型`，也就是仅仅只能用于注解`注解类型(@interface)`的注解（绕口hh），通常我们称这类(就这四个)注解为`元注解(Meta Annotation)`。

### `@Documented`

第一个是声明注解会出现在文档内部，来看看定义

> Documented.java

```java
package java.lang.annotation;

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Documented {
}
```

`@Documented` 说明他会被编入 javadoc 内；`@Retention(RetentionPolicy.RUNTIME)` 说明能够在运行时动态查找到该注解；`@Target(ElementType.ANNOTATION_TYPE)` 说明他只能附加在注解类型上

### `@Retention`

第二个是 `@Retention`，用于指定注解的生命周期

> Retention.java

```java
package java.lang.annotation;

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Retention {
    RetentionPolicy value();
}
```

### `@Target`

第三个是 `@Target`，用于指定注解可作用的目标

> Target.java

```java
package java.lang.annotation;

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    ElementType[] value();
}
```

### `@Inherited`

最后一个是 `@Inherited`，用于声明注解可被继承

> Inherited.java

```java
package java.lang.annotation;

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Inherited {
}
```

## 自定义注解

说了半天我们终于要开始自定义注解了！

### 自定义注解语法

首先先来看看如何定义一个注解

```java
@Retention
@Target
public @interface <annotation-name> {
    // member
}
```

我们需要使用 `@interface` 关键字声明该类型为注解类型(会实现 `Annotation` 接口)，然后使用 `@Retention` 指定生命周期；并使用 `@Target` 指定可作用对象。

- 如果未添加 `@Retention`，则默认会使用 `RetentionPolicy.CLASS`，字节码级别的声明周期
- 如果未添加 `@Target`，则默认能够作用在所有目标类型上

### 自定义注解应用

最后我们举几个自定义注解的例子，并使用 Class 对象来获取注解信息（对 Class 对象不熟可以参看前两篇：<a href="https://blog.csdn.net/weixin_44691608/article/details/111403969">Java 基础: 浅谈类型基础 - Class 对象</a>、<a href="https://blog.csdn.net/weixin_44691608/article/details/111413301">Java 进阶: Reflect 反射机制（动态获取类内部结构和对象内容、调用方法）</a>）

#### 一般注解

第一种我们先演示最基本的自定义注解，不包含任何的属性

> SimpleAnnotation.java：自定义注解

```java
package com.example.annotation.test1;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.FIELD, ElementType.METHOD})
public @interface SimpleAnnotation {
}
```

> SimpleAnnotationUsage.java：使用自定义注解

```java
package com.example.annotation.test1;

@SimpleAnnotation
public class SimpleAnnotationUsage {

    @SimpleAnnotation
    private String field;

    private String unAnnotatedField;

    @SimpleAnnotation
    public void method() {}

    private void unAnnotatedMethod() {}
}
```

> Test.java：查看注解信息（使用 Class 类型的 `getDeclaredAnnotations` 方法）

```java
package com.example.annotation.test1;

import java.lang.annotation.Annotation;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.util.Arrays;

public class Test {
    public static void main(String[] args) {
        Class<SimpleAnnotationUsage> c = SimpleAnnotationUsage.class;
        // 查看类的注解
        System.out.println("----- Declared annotations from SimpleAnnotationUsage -----");
        for (Annotation annotation : c.getDeclaredAnnotations()) {
            System.out.println(annotation.annotationType().getSimpleName());
        }
        // 查看字段（成员变量）的注解
        System.out.println("----- Declared fields from SimpleAnnotationUsage -----");
        for (Field field : c.getDeclaredFields()) {
            System.out.println("Declared annotations from SimpleAnnotationUsage." + field.getName() + ": " + Arrays.toString(field.getAnnotations()));
        }
        // 查看方法的注解
        System.out.println("----- Declared methods from SimpleAnnotationUsage -----");
        for(Method method : c.getDeclaredMethods()) {
            System.out.println("Declared annotations from SimpleAnnotationUsage." + method.getName() + ": " + Arrays.toString(method.getAnnotations()));
        }
    }
}
```

> 输出结果

```
----- Declared annotations from SimpleAnnotationUsage -----
SimpleAnnotation
----- Declared fields from SimpleAnnotationUsage -----
Declared annotations from SimpleAnnotationUsage.field: [@com.example.annotation.test1.SimpleAnnotation()]
Declared annotations from SimpleAnnotationUsage.unAnnotatedField: []
----- Declared methods from SimpleAnnotationUsage -----
Declared annotations from SimpleAnnotationUsage.unAnnotatedMethod: []
Declared annotations from SimpleAnnotationUsage.method: [@com.example.annotation.test1.SimpleAnnotation()]
```

#### 带属性注解

第二个例子我们声明一个可以附带属性的注解，这边我们仿造 Spring MVC 里面的 `@Controller` 和 `@RequestMapping` 注解，注解定义如下：

> Controller.java：控制器类注解

```java
package com.example.annotation.test2;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Controller {
}
```

> RequestMapping.java：请求接口注解

```java
package com.example.annotation.test2;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface RequestMapping {
    String name(); // 接口名称
    String path(); // 接口路由
    Class[] params() default {}; // 接口参数
}
```

> DemoController.java：使用自定义注解（带属性）

```java
package com.example.annotation.test2;

@Controller
public class DemoController {

    @RequestMapping(name = "foo", path = "/foo")
    public void foo() {}

    @RequestMapping(name = "bar", path = "/bar", params = {int.class, double.class})
    public void bar(int i, double d) {}
}
```

> Test.java：查看注解和注解属性

```java
package com.example.annotation.test2;

import java.lang.reflect.Method;
import java.util.Arrays;

public class Test {
    public static void main(String[] args) {
        Class<DemoController> c = DemoController.class;
        System.out.println("DemoController's annotations: " + Arrays.toString(c.getDeclaredAnnotations()));
        for (Method method : c.getDeclaredMethods()) {
            System.out.println("method: name=" + method.getName() + ", with annotations: " + Arrays.toString(method.getDeclaredAnnotations()));
            RequestMapping requestMapping = method.getAnnotation(RequestMapping.class);
            System.out.println("\tname: " + requestMapping.name());
            System.out.println("\tpath: " + requestMapping.path());
            System.out.println("\tparamsType: " + Arrays.toString(requestMapping.params()));
        }
    }
}
```

> 输出结果

```
DemoController's annotations: [@com.example.annotation.test2.Controller()]
method: name=bar, with annotations: [@com.example.annotation.test2.RequestMapping(params=[int, double], name=bar, path=/bar)]
	name: bar
	path: /bar
	paramsType: [int, double]
method: name=foo, with annotations: [@com.example.annotation.test2.RequestMapping(params=[], name=foo, path=/foo)]
	name: foo
	path: /foo
	paramsType: []
```

到此就是注解的简单应用。注意：注解的附带属性声明形式为一个方法(只读 readonly)。

不过我们注意到一点，每次都要指定注解属性有些麻烦，而且我们还希望能够使用`默认参数`、`别名`、`自动解析方法`的功能，接下来请看最后一个例子

#### 配合注解自动解析路由

最后一个例子我们透过`反射(reflect)`来自动解析接口参数，并提供接口路由设置`别名(alias)`。这边模拟实现了一个极简易版的 `@AliasFor` 和 `AnnotationUtils` 实现，后面会再另写一篇专门解析 Spring 对于注解的妙用

> AliasFor.java：设置别名注解

```java
package com.example.annotation.test3;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD) // 用于修饰注解附带属性，属于方法目标
public @interface AliasFor {
    // 不指定属性时默认为 value()
    String value() default "";
}
```

> Controller.java：控制器类注解

```java
package com.example.annotation.test3;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Controller {
}
```

> RequestMapping.java：接口注解

```java
package com.example.annotation.test3;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface RequestMapping {

    // 默认读取 value，于 path 互为别名
    @AliasFor("path")
    String value() default "";
    @AliasFor("value")
    String path() default "";
}
```

> DemoController.java：使用接口，模拟 MVC 的控制层(Controller)

```java
package com.example.annotation.test3;

@Controller
public class DemoController {

    @RequestMapping("/foo")
    public void foo() {}

    @RequestMapping(path = "/bar")
    public void bar() {}
}
```

> AnnotationUtils.java：控制层注解解析类（带测试入口）

```java
package com.example.annotation.test3;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class AnnotationUtils {
    public void solve(Class<?> c) throws NoSuchMethodException, IllegalAccessException, InvocationTargetException {
        // 检查传入类是否标记为 @Controller
        if (c.getDeclaredAnnotation(Controller.class) == null) {
            throw new RuntimeException("class didn't declared with @Controller");
        }
        // 遍历所有方法
        for (Method method : c.getDeclaredMethods()) {
            System.out.println("method: " + method.getName() + ", annotations: " + Arrays.toString(method.getDeclaredAnnotations()));
            // 只处理标记为 @RequestMapping 的方法作为接口
            if (method.getAnnotation(RequestMapping.class) == null) continue;

            // 获取接口注解内容
            RequestMapping requestMapping = method.getAnnotation(RequestMapping.class);
            Class<RequestMapping> cr = RequestMapping.class;
            List<String> params = new ArrayList<>();
            // 遍历接口注解的所有属性
            for (Method attr : cr.getDeclaredMethods()) {
                Object val = attr.invoke(requestMapping);
                AliasFor aliasFor;
                // 如果未传入值且存在别名，则引用别名
                if (val.equals(attr.getDefaultValue()) && (aliasFor = attr.getDeclaredAnnotation(AliasFor.class)) != null) {
                    val = cr.getDeclaredMethod(aliasFor.value()).invoke(requestMapping);
                }
                params.add(attr.getName() + ": " + val);
            }
            System.out.println("RequestMapping with params: {" + String.join(", ", params) + "}");
        }
    }

    // 测试入口
    public static void main(String[] args) throws NoSuchMethodException, IllegalAccessException, InvocationTargetException {
        new AnnotationUtils().solve(DemoController.class);
    }
}
```

> 输出结果

```
method: bar, annotations: [@com.example.annotation.test3.RequestMapping(value=, path=/bar)]
RequestMapping with params: {value: /bar, path: /bar}
method: foo, annotations: [@com.example.annotation.test3.RequestMapping(value=/foo, path=)]
RequestMapping with params: {value: /foo, path: /foo}
```

# 结语

本篇从注解的类型、接口，到介绍内置注解和元注解，最后实现自定义注解的语法和三个例子，希望读者全面了解 Java 注解(Annotation) 的全貌。

最后的最后再次强调：**注解只是附带信息，而不进行任何操作。**如 Spring 对于注解的应用核心部分在于类似第三个例子的 `AnnotationUtils`，需要依赖应用主动解析实现。
