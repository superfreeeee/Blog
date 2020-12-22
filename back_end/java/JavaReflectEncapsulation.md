# Java 应用: Reflect 封装，一次实现使用字符串查找字段、调用方法、打破 private 限制

@[TOC](文章目录)

<!-- TOC -->

- [Java 应用: Reflect 封装，一次实现使用字符串查找字段、调用方法、打破 private 限制](#java-应用-reflect-封装一次实现使用字符串查找字段调用方法打破-private-限制)
  - [简介](#简介)
  - [参考](#参考)
  - [完整示例代码](#完整示例代码)
- [正文](#正文)
  - [实现目标：`ReflectUtils` 工具类](#实现目标reflectutils-工具类)
  - [具体实现](#具体实现)
    - [MyAnnotation 自定义注解](#myannotation-自定义注解)
    - [Demo 调用目标](#demo-调用目标)
    - [主角：ReflectUtils 封装反射方法的工具类](#主角reflectutils-封装反射方法的工具类)
      - [实现要点](#实现要点)
    - [测试检验](#测试检验)
- [结语](#结语)

<!-- /TOC -->

## 简介

前一篇<a href="https://blog.csdn.net/weixin_44691608/article/details/111413301">Java 进阶: Reflect 反射机制（动态获取类内部结构和对象内容、调用方法）</a>介绍了 Java 的反射原理，以及透过反射来获取各样的类信息。

本篇将要在此基础之上，封装反射的方法。每次想用反射都要找 Class 找方法（包括确定参数类型、返回类型），实在是太麻烦了，所以我们将稍微封装一下反射，使我们能够直接透过`对象(obj)`和`方法名字(name)(字符串)`就能够获取字段(field)或是调用函数(method)。

## 参考

<table>
  <tr>
    <td></td>
    <td><a href=""></a></td>
  </tr>
</table>

## 完整示例代码

<a href="https://github.com/superfreeeee/Blog-code/tree/main/back_end/java/java_relfect_encapsulation">https://github.com/superfreeeee/Blog-code/tree/main/back_end/java/java_relfect_encapsulation</a>

# 正文

## 实现目标：`ReflectUtils` 工具类

在正式开始讲解代码之前，我们先来明确一下本篇的实现目标。透过 Java Reflect 的反射机制，我们可以使用 Class 对象获取各种类型信息，以及动态获取运行时对象的信息。现在我们打算封装出一下几个方法（所有方法封装到 `ReflectUtils` 工具类之中）：

> ReflectUtils.java：实现接口

```java
public abstract class ReflectUtils {
    // 根据字段名 name，获取对象 obj 的字段
    public static Object getField(Object obj, String name);
    // 根据方法名 name 和传入参数 args，调用对象 obj 的方法
    public static Object invokeMethod(Object obj, String name, Object... args);
    // 根据注解名 name，获取类 c 定义的注解
    public static Annotation getAnnotation(Class c, String name);
    // 根据注解名 name 和属性类型 type(可以是字段 Field 或是方法 Method) 以及属性名 target，获取类 c 在属性上定义的注解
    public static Annotation getAnnotation(Class c, String name, ElementType type, String target);
}
```

下面我们就来看看实现

## 具体实现

### MyAnnotation 自定义注解

首先 Java 内置的注解都不太好使，所以先自定义一个注解待会能用。

> MyAnnotation.java：自定义注解

```java
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
    String value() default "";
}
```

### Demo 调用目标

接下来我们写一个待会要用于动态获取字段、调用方法、获取注解的对象类型定义。

> Demo.java

```java
@MyAnnotation("Class Demo")
public class Demo {

    @MyAnnotation("Field field")
    private String field = "default string of Demo.field";

    @MyAnnotation("Method f")
    private void f() {
        System.out.println("invoke function f from Demo");
    }

    private String g() {
        return "return String from Demo.g()";
    }
}
```

### 主角：ReflectUtils 封装反射方法的工具类

接下来就是我们的主角，封装反射方法的工具类 `ReflectUtils`

> ReflectUtils.java

```java

import java.lang.annotation.Annotation;
import java.lang.annotation.ElementType;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class ReflectUtils {
    // 根据字段名获取字段
    public static Object getField(Object obj, String name) throws NoSuchFieldException, IllegalAccessException {
        if (obj == null) throw new NullPointerException();
        Class c = obj.getClass(); // 获取 Class 对象
        Field field = c.getDeclaredField(name); // 获取指定字段 Field
        field.setAccessible(true); // 设置可访问性，屏蔽 private
        return field.get(obj); // 动态获取对象 obj 字段
    }

    // 根据方法名和参数调用方法
    public static Object invokeMethod(Object obj, String name, Object... args) throws NoSuchMethodException, IllegalAccessException, InvocationTargetException {
        if (obj == null) throw new NullPointerException();
        Class c = obj.getClass();
        Method method;
        // 获取具体方法 Method
        if (args.length == 0) {
            // 无参数列表
            method = c.getDeclaredMethod(name);
        } else {
            // 获取参数类型列表
            Class[] paramTypes = new Class[args.length];
            for (int i = 0; i < args.length; i++) {
                paramTypes[i] = args[i].getClass();
            }
            method = c.getDeclaredMethod(name, paramTypes);
        }
        method.setAccessible(true); // 设置可访问性屏蔽 private
        if (method.getReturnType() == void.class) {
            // 返回类型为 void
            method.invoke(obj, args);
            return null; // 调用后返回 null
        } else {
            // 存在返回值，一律返回 Object
            return method.invoke(obj, args);
        }
    }

    // 根据名称获取类的注解
    public static Annotation getAnnotation(Class c, String name) {
        for (Annotation annotation : c.getDeclaredAnnotations()) {
            // 检查注解列表是否存在匹配注解名，避免需要获取特定注解的 Class 对象
            if (annotation.annotationType().getSimpleName().equals(name)) {
                return annotation;
            }
        }
        return null;
    }

    // 根据名称、属性类型、属性名获取注解
    public static Annotation getAnnotation(Class c, String name, ElementType type, String target) throws NoSuchFieldException, NoSuchMethodException, ClassNotFoundException {
        switch (type) {
            case FIELD: // 字段上的注解
                Field field = c.getDeclaredField(target);
                for (Annotation annotation : field.getAnnotations()) {
                    if (annotation.annotationType().getSimpleName().equals(name)) {
                        return annotation;
                    }
                }
                break;
            case METHOD: // 方法上的注解
                Method method = c.getDeclaredMethod(target);
                for (Annotation annotation : method.getAnnotations()) {
                    if (annotation.annotationType().getSimpleName().equals(name)) {
                        return annotation;
                    }
                }
                break;
        }
        return null;
    }
}
```

#### 实现要点

相关的方法实现在上面的代码注释中都有说明，这边特别提出几个实现要点

1. `Field.setAccessible(true)` 设置可访问性，相当于屏蔽了 private 访问修饰符的作用，使得用户可以访问任意字段（使修饰访问符失效）
2. `Method.setAccessible(true)` 设置可访问性，避免方法内部调用私有变量引起的访问权限异常(IllegalAccessException)
3. `invokeMethod` 方法内部需要区分
   1. 方法类型：`无参数`和`有参数(需要根据参数类型列表才能找到正确的方法，相当于完整的函数签名)`
   2. 返回类型：`void`和`Object`类型是不同的，需要加以区分
4. `getAnnotation` 方法中针对 Annotation 类型不能使用 `getClass` 方法（会返回`$Proxy`），而应该使用 `annotationType` 来获取正确的 Class 对象

### 测试检验

最后就来用用看封装好的工具类吧，直接透过字符串获取字段或调用方法，真香

> Test.java

```java
import java.lang.annotation.ElementType;

public class Test {
    public static void main(String[] args) throws Exception {
        Demo demo = new Demo();
        System.out.println("demo.field: " + ReflectUtils.getField(demo, "field"));
        System.out.println("demo.methodReturnVoid(): " + ReflectUtils.invokeMethod(demo, "methodReturnVoid"));
        System.out.println("demo.methodReturnString(): " + ReflectUtils.invokeMethod(demo, "methodReturnString"));
        System.out.println("demo.methodWithParams(1, 2): " + ReflectUtils.invokeMethod(demo, "methodWithParams", 1, 2));
        System.out.println("MyAnnotation on Demo: " + ReflectUtils.getAnnotation(Demo.class, "MyAnnotation"));
        System.out.println("MyAnnotation on Demo.field: " + ReflectUtils.getAnnotation(Demo.class, "MyAnnotation", ElementType.FIELD, "field"));
        System.out.println("MyAnnotation on Demo.method: " + ReflectUtils.getAnnotation(Demo.class, "MyAnnotation", ElementType.METHOD, "methodReturnVoid"));
    }
}
```

> 输出结果

```
demo.field: default string of Demo.field
invoke methodReturnVoid from Demo
demo.methodReturnVoid(): null
demo.methodReturnString(): return String from Demo.gmethodReturnString)
demo.methodWithParams(1, 2): 3
MyAnnotation on Demo: @MyAnnotation(value=Class Demo)
MyAnnotation on Demo.field: @MyAnnotation(value=Field field)
MyAnnotation on Demo.method: @MyAnnotation(value=Method methodReturnVoid)
```

# 结语

本篇应用 Java 的反射机制并加以封装，实现 `ReflectUtils` 用于直接根据字符串来`动态`获取对象信息(Field、Annotaion)或是调用方法(method)，供读者参考。
