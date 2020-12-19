# Java 进阶: Reflect 反射机制（动态获取类内部结构和对象内容、调用方法）

@[TOC](文章目录)

<!-- TOC -->

- [Java 进阶: Reflect 反射机制（动态获取类内部结构和对象内容、调用方法）](#java-进阶-reflect-反射机制动态获取类内部结构和对象内容调用方法)
  - [简介](#简介)
  - [参考](#参考)
  - [完整测试代码地址](#完整测试代码地址)
- [正文](#正文)
  - [获取类定义结构](#获取类定义结构)
    - [可获取内部信息总览](#可获取内部信息总览)
  - [成员变量 Field](#成员变量-field)
    - [Field 方法](#field-方法)
    - [Field 信息](#field-信息)
    - [Field 动态获取对象内容](#field-动态获取对象内容)
  - [方法 Method](#方法-method)
    - [Method 方法](#method-方法)
    - [Method 信息](#method-信息)
    - [Method 动态调用方法](#method-动态调用方法)
  - [构造函数 Constructor](#构造函数-constructor)
    - [Constructor 方法](#constructor-方法)
    - [Constructor 信息](#constructor-信息)
    - [Constructor 动态创建对象实例](#constructor-动态创建对象实例)
- [结语](#结语)

<!-- /TOC -->

## 简介

前一篇：<a href="https://blog.csdn.net/weixin_44691608/article/details/111403969">Java 基础: 浅谈类型基础 - Class 对象</a>带我们初步认识了 Class 对象有关类型的信息，然而我们真正希望做的是透过反射动态获取对象中的值或是调用对象的方法。老样子，还是我们的老朋友 `Class 对象`。

## 参考

<table>
  <tr>
    <td>五分钟看懂ClassLoader</td>
    <td><a href="https://www.jianshu.com/p/554c138ca0f5">https://www.jianshu.com/p/554c138ca0f5</a></td>
  </tr>
  <tr>
    <td>JAVA Reflect反射详解</td>
    <td><a href="https://blog.csdn.net/u014209205/article/details/80542114">https://blog.csdn.net/u014209205/article/details/80542114</a></td>
  </tr>
  <tr>
    <td>试用Java中的反射reflect之getDeclaredMethods和getMethods</td>
    <td><a href="https://www.cnblogs.com/jianjianjiao/articles/1853409.html">https://www.cnblogs.com/jianjianjiao/articles/1853409.html</a></td>
  </tr>
  <tr>
    <td>Java Field getBoolean()用法及代码示例</td>
    <td><a href="https://vimsky.com/examples/usage/field-getboolean-method-in-java-with-examples.html">https://vimsky.com/examples/usage/field-getboolean-method-in-java-with-examples.html</a></td>
  </tr>
</table>

## 完整测试代码地址

先附上文中用上的完整测试代码地址：<a href="https://github.com/superfreeeee/Blog-code/tree/main/back_end/java/java_reflect">https://github.com/superfreeeee/Blog-code/tree/main/back_end/java/java_reflect</a>

# 正文

## 获取类定义结构

前一篇我们演示了如何获取并检查类的各种信息（访问修饰符、类/接口/枚举/注解类型、blablabla），下面我们要来获取类的内部结构

### 可获取内部信息总览

首先我们先来看看我们可以获取哪些内部信息

| Method                                  | Description          |
| --------------------------------------- | -------------------- |
| Field[] getFields()                     | 获取成员变量列表     |
| Field[] getDeclaredFields()             | 获取自有成员变量列表 |
| Method[] getMethods()                   | 获取方法列表         |
| Method[] getDeclaredMethods()           | 获取自有方法列表     |
| Constructor[] getConstructors()         | 获取构造函数列表     |
| Constructor[] getDeclaredConstructors() | 获取自有构造函数列表 |
| Class[] getClasses()                    | 获取内部类列表       |
| Class[] getDeclaredClasses()            | 获取自有内部类列表   |

应该可以很清楚的发现四种内容都分为 `getXXX` 和 `getDeclaredXXX` 两个方法，其中的差异在于：

- `getXXX` 获取所有从祖先类、接口继承或实现的访问修饰符为 `public` 的成员
- `getDeclaredXXX` 获取所有该类自己定义的所有(任意描述符)成员

看看下面的用例就知道了

> Demo.java

```java
package com.example.reflect.test1;

// 建立不同访问权限的各种成员
public class Demo {
    // 成员变量
    private String privateField;
    protected String protectedField;
    String defaultField;
    public String publicField;

    // 构造函数
    private Demo() {}
    protected Demo(int i) {}
    Demo(float f) {}
    public Demo(double d) {}

    // 方法
    private void privateMethod() {}
    protected void protectedMethod() {}
    void defaultMethod() {}
    public void publicMethod() {}

    // 内部类
    private class PrivateInnerClass {}
    protected class ProtectedInnerClass {}
    class DefaultInnerClass {}
    public class PublicInnerClass {}
}
```

> Test.java

```java
package com.example.reflect.test1;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.util.Arrays;

public class Test {
    public static void main(String[] args) {
        Class<Demo> demoClass = Demo.class;

        // 查询成员变量
        System.out.println("----- Demo fields ----- ");
        for(Field field : demoClass.getFields()) {
            System.out.println("field: " + field.getName() + ", type: " + field.getType().getName());
        }
        System.out.println("----- Demo declared fields ----- ");
        for(Field field : demoClass.getDeclaredFields()) {
            System.out.println("field: " + field.getName() + ", type: " + field.getType().getName());
        }
        // 查询方法
        System.out.println("----- Demo methods ----- ");
        for(Method method : demoClass.getMethods()) {
            System.out.println("method: " + method.getName() + ", params: " + Arrays.toString(method.getParameterTypes()));
        }
        System.out.println("----- Demo declared methods ----- ");
        for(Method method : demoClass.getDeclaredMethods()) {
            System.out.println("method: " + method.getName() + ", params: " + Arrays.toString(method.getParameterTypes()));
        }
        // 查询构造函数
        System.out.println("----- Demo constructors ----- ");
        for(Constructor constructor : demoClass.getConstructors()) {
            System.out.println("constructor: " + constructor.getName() + ", params: " + Arrays.toString(constructor.getParameterTypes()));
        }
        System.out.println("----- Demo declared constructors ----- ");
        for(Constructor constructor : demoClass.getDeclaredConstructors()) {
            System.out.println("constructor: " + constructor.getName() + ", params: " + Arrays.toString(constructor.getParameterTypes()));
        }
        // 查询内部类
        System.out.println("----- Demo classes ----- ");
        for(Class c : demoClass.getClasses()) {
            System.out.println("class: " + c.getName() + ", modifiers: " + c.getModifiers());
        }
        System.out.println("----- Demo declared classes ----- ");
        for(Class c : demoClass.getDeclaredClasses()) {
            System.out.println("class: " + c.getName() + ", modifiers: " + c.getModifiers());
        }
    }
}
```

> 输出

```
----- Demo fields ----- 
field: publicField, type: java.lang.String
----- Demo declared fields ----- 
field: privateField, type: java.lang.String
field: protectedField, type: java.lang.String
field: defaultField, type: java.lang.String
field: publicField, type: java.lang.String
----- Demo methods ----- 
method: publicMethod, params: []
method: wait, params: [long, int]
method: wait, params: [long]
method: wait, params: []
method: equals, params: [class java.lang.Object]
method: toString, params: []
method: hashCode, params: []
method: getClass, params: []
method: notify, params: []
method: notifyAll, params: []
----- Demo declared methods ----- 
method: privateMethod, params: []
method: protectedMethod, params: []
method: defaultMethod, params: []
method: publicMethod, params: []
----- Demo constructors ----- 
constructor: com.example.reflect.test1.Demo, params: [double]
----- Demo declared constructors ----- 
constructor: com.example.reflect.test1.Demo, params: []
constructor: com.example.reflect.test1.Demo, params: [int]
constructor: com.example.reflect.test1.Demo, params: [float]
constructor: com.example.reflect.test1.Demo, params: [double]
----- Demo classes ----- 
class: com.example.reflect.test1.Demo$PublicInnerClass, modifiers: 1
----- Demo declared classes ----- 
class: com.example.reflect.test1.Demo$PublicInnerClass, modifiers: 1
class: com.example.reflect.test1.Demo$DefaultInnerClass, modifiers: 0
class: com.example.reflect.test1.Demo$ProtectedInnerClass, modifiers: 4
class: com.example.reflect.test1.Demo$PrivateInnerClass, modifiers: 2
```

下面我们将对前三种成员类型一一说明（第四种就是内部的 Class 类型，与一般的 Class 对象无异）

## 成员变量 Field

第一种成员类型是成员变量，定义为 `java.lang.reflect.Field` 类型

### Field 方法

首先先来看看 Field 可以调用的方法接口

| Method                                 | Usage                          |
| -------------------------------------- | ------------------------------ |
| String getName()                       | 获取变量名                     |
| Class getType()、Type getGenericType() | 获取变量类型                   |
| Class getDeclaringClass()              | 获取成员所属类型               |
| int getModifiers()                     | 获取变量访问修饰符             |
| T getT(Object)                         | 动态获取参数对象实际成员变量值 |

### Field 信息

> FieldDemo.java：成员变量示例类型

```java
package com.example.reflect.test2;

public class FieldDemo {
    private int privateInt; // private 变量
    protected int protectedInt; // protected 变量
    int defaultInt; // default 变量
    public int publicInt; // public 变量
    public static int publicStaticInt; // public static 变量

    private String stringField; // String 类型
    private Object objectField; // Object 类型
    private int[] arrayFiled; // 数组类型

}
```

> 测试方法

```java
void testFieldInfo() {
    System.out.println("##### testFieldInfo #####");
    Class<FieldDemo> fieldDemoClass = FieldDemo.class;

    // getFields
    System.out.println("----- fields ----- ");
    for (Field field : fieldDemoClass.getFields()) {
        System.out.println(field.getName());
    }
    // getDeclaredFields
    System.out.println("----- declared fields ----- ");
    for(Field field : fieldDemoClass.getDeclaredFields()) {
        System.out.println(field.getName());
    }
    // fields detail
    System.out.println("----- declared fields detail ----- ");
    for(Field field : fieldDemoClass.getDeclaredFields()) {
        System.out.println("name: " + field.getName());
        System.out.println("\ttype: " + field.getType());
        System.out.println("\tgenericType: " + field.getGenericType());
        System.out.println("\tdeclaringClass: " + field.getDeclaringClass());
        System.out.println("\tmodifiers: " + field.getModifiers());
    }
    System.out.println();
}
```

> 输出

```
##### testFieldInfo #####
----- fields ----- 
publicInt
publicStaticInt
----- declared fields ----- 
privateInt
protectedInt
defaultInt
publicInt
publicStaticInt
stringField
objectField
arrayFiled
----- declared fields detail ----- 
name: privateInt
	type: int
	genericType: int
	declaringClass: class com.example.reflect.test2.FieldDemo
	modifiers: 2
name: protectedInt
	type: int
	genericType: int
	declaringClass: class com.example.reflect.test2.FieldDemo
	modifiers: 4
name: defaultInt
	type: int
	genericType: int
	declaringClass: class com.example.reflect.test2.FieldDemo
	modifiers: 0
name: publicInt
	type: int
	genericType: int
	declaringClass: class com.example.reflect.test2.FieldDemo
	modifiers: 1
name: publicStaticInt
	type: int
	genericType: int
	declaringClass: class com.example.reflect.test2.FieldDemo
	modifiers: 9
name: stringField
	type: class java.lang.String
	genericType: class java.lang.String
	declaringClass: class com.example.reflect.test2.FieldDemo
	modifiers: 2
name: objectField
	type: class java.lang.Object
	genericType: class java.lang.Object
	declaringClass: class com.example.reflect.test2.FieldDemo
	modifiers: 2
name: arrayFiled
	type: class [I
	genericType: class [I
	declaringClass: class com.example.reflect.test2.FieldDemo
	modifiers: 2
```


### Field 动态获取对象内容

> 测试方法

```java
void extractFieldInfo() throws NoSuchFieldException, IllegalAccessException {
    System.out.println("##### extractFieldInfo #####");
    FieldDemo demo1 = new FieldDemo();
    demo1.publicInt = 1;
    FieldDemo demo2 = new FieldDemo();
    demo2.publicInt = 2;

    Class<FieldDemo> fieldDemoClass = FieldDemo.class;
    Field publicInt = fieldDemoClass.getDeclaredField("publicInt"); // throws NoSuchFieldException
    System.out.println("demo1.publicInt: " + publicInt.getInt(demo1));
    System.out.println("demo2.publicInt: " + publicInt.getInt(demo2));
    System.out.println();
}
```

> 输出

```
##### extractFieldInfo #####
demo1.publicInt: 1
demo2.publicInt: 2
```

## 方法 Method

第二种是方法，定义为 `java.lang.reflect.Method` 

### Method 方法

| Method                                                         | Usage                                |
| -------------------------------------------------------------- | ------------------------------------ |
| String getName()                                               | 获取方法名                           |
| Class getReturnType()、Type getGenericReturnType()             | 获取返回类型                         |
| Class[] getParameterTypes()、Type[] getGenericParameterTypes() | 获取参数类型列表                     |
| Class[] getExceptionTypes()、Type[] getGenericExceptionTypes() | 获取抛出异常类型列表                 |
| Class getDeclaringClass()                                      | 获取定义方法的类                     |
| Object getDefaultValue()                                       | 获取默认返回值                       |
| int getModifiers()                                             | 获取方法访问修饰符                   |
| void setAccessible(boolean)                                    | 设置调用时访问权限                   |
| Object invoke(Object, Object ...args)                          | 传入调用方法对象和参数列表，调用函数 |

### Method 信息

> MethodDemo.java

```java
package com.example.reflect.test3;

public class MethodDemo {

    private String name;

    public MethodDemo(String name) {
        this.name = name;
    }

    private void privateMethod() {}
    protected void protectedMethod() {}
    void defaultMethod() {}
    public void publicMethod() {}
    public static void publicStaticMethod() {}
    private void privateMethodWithParams(int i, int j) {
        System.out.println("invoke privateMethodWithParams from object: [name=" + name + "], with params: [i=" + i + ", j=" + j + "]");
    }
    void defaultMethodWithExceptions() throws NoSuchMethodException, IllegalArgumentException {}
}
```

> 测试方法

```java
void testMethodInfo() {
    System.out.println("##### testMethodInfo #####");
    Class<MethodDemo> c = MethodDemo.class;
    System.out.println("----- methods -----");
    for (Method method : c.getMethods()) {
        System.out.println(method.getName());
    }

    System.out.println("----- declared methods -----");
    for (Method method : c.getDeclaredMethods()) {
        System.out.println(method.getName());
    }

    System.out.println("----- declared methods detail -----");
    for (Method method : c.getDeclaredMethods()) {
        System.out.println("name: " + method.getName());
        System.out.println("\treturnType: " + method.getReturnType());
        System.out.println("\tgenericReturnType: " + method.getGenericReturnType());
        System.out.println("\tparameterTypes: " + Arrays.toString(method.getParameterTypes()));
        System.out.println("\tgenericParameterTypes: " + Arrays.toString(method.getGenericParameterTypes()));
        System.out.println("\texceptionTypes: " + Arrays.toString(method.getExceptionTypes()));
        System.out.println("\tgenericExceptionTypes: " + Arrays.toString(method.getGenericExceptionTypes()));
        System.out.println("\tdeclaringClass: " + method.getDeclaringClass());
        System.out.println("\tdefaultValue: " + method.getDefaultValue());
        System.out.println("\tmodifiers: " + method.getModifiers());
    }

    System.out.println();
}
```

> 输出

```
##### testMethodInfo #####
----- methods -----
publicMethod
publicStaticMethod
wait
wait
wait
equals
toString
hashCode
getClass
notify
notifyAll
----- declared methods -----
privateMethod
protectedMethod
privateMethodWithParams
defaultMethod
publicMethod
publicStaticMethod
defaultMethodWithExceptions
----- declared methods detail -----
name: privateMethod
	returnType: void
	genericReturnType: void
	parameterTypes: []
	genericParameterTypes: []
	exceptionTypes: []
	genericExceptionTypes: []
	declaringClass: class com.example.reflect.test3.MethodDemo
	defaultValue: null
	modifiers: 2
name: protectedMethod
	returnType: void
	genericReturnType: void
	parameterTypes: []
	genericParameterTypes: []
	exceptionTypes: []
	genericExceptionTypes: []
	declaringClass: class com.example.reflect.test3.MethodDemo
	defaultValue: null
	modifiers: 4
name: privateMethodWithParams
	returnType: void
	genericReturnType: void
	parameterTypes: [int, int]
	genericParameterTypes: [int, int]
	exceptionTypes: []
	genericExceptionTypes: []
	declaringClass: class com.example.reflect.test3.MethodDemo
	defaultValue: null
	modifiers: 2
name: defaultMethod
	returnType: void
	genericReturnType: void
	parameterTypes: []
	genericParameterTypes: []
	exceptionTypes: []
	genericExceptionTypes: []
	declaringClass: class com.example.reflect.test3.MethodDemo
	defaultValue: null
	modifiers: 0
name: publicMethod
	returnType: void
	genericReturnType: void
	parameterTypes: []
	genericParameterTypes: []
	exceptionTypes: []
	genericExceptionTypes: []
	declaringClass: class com.example.reflect.test3.MethodDemo
	defaultValue: null
	modifiers: 1
name: publicStaticMethod
	returnType: void
	genericReturnType: void
	parameterTypes: []
	genericParameterTypes: []
	exceptionTypes: []
	genericExceptionTypes: []
	declaringClass: class com.example.reflect.test3.MethodDemo
	defaultValue: null
	modifiers: 9
name: defaultMethodWithExceptions
	returnType: void
	genericReturnType: void
	parameterTypes: []
	genericParameterTypes: []
	exceptionTypes: [class java.lang.NoSuchMethodException, class java.lang.IllegalArgumentException]
	genericExceptionTypes: [class java.lang.NoSuchMethodException, class java.lang.IllegalArgumentException]
	declaringClass: class com.example.reflect.test3.MethodDemo
	defaultValue: null
	modifiers: 0
```

### Method 动态调用方法

> 测试方法

```java
void invokeMethod() throws NoSuchMethodException, IllegalAccessException, InvocationTargetException {
    System.out.println("##### invokeMethod #####");
    MethodDemo demo1 = new MethodDemo("demo1");
    MethodDemo demo2 = new MethodDemo("demo2");

    Class<MethodDemo> c = MethodDemo.class;
    Method m = c.getDeclaredMethod("privateMethodWithParams", int.class, int.class);
    m.setAccessible(true); // 方法内部引用私有变量，需要设置 setAccessible(true) 才能正常调用
    m.invoke(demo1, 10, 20);
    m.invoke(demo2, 30, 40);
    System.out.println();
}
```

> 输出

```
##### invokeMethod #####
invoke privateMethodWithParams from object: [name=demo1], with params: [i=10, j=20]
invoke privateMethodWithParams from object: [name=demo2], with params: [i=30, j=40]
```

## 构造函数 Constructor

第三种是构造函数，定义为 `java.lang.reflect.Constructor`

### Constructor 方法

| Method                                                         | Usage              |
| -------------------------------------------------------------- | ------------------ |
| String getName()                                               | 构造函数名（类名） |
| Class[] getParameterTypes()、Type[] getGenericParameterTypes() | 获取参数列表       |
| int getModifiers()                                             | 获取访问修饰符     |

### Constructor 信息

> ConstructorDemo.java

```java
package com.example.reflect.test4;

public class ConstructorDemo {
    private ConstructorDemo() {}
    protected ConstructorDemo(int i) {
        System.out.println("create ConstructorDemo with i=" + i);
    }
    ConstructorDemo(float f) {}
    public ConstructorDemo(double d) {}
}
```

> 测试方法

```java
void testConstructorInfo() {
    System.out.println("##### testConstructorInfo #####");
    Class<ConstructorDemo> c = ConstructorDemo.class;
    System.out.println("----- constructors -----");
    for (Constructor constructor : c.getConstructors()) {
        System.out.println(constructor.getName() + ", params: " + Arrays.toString(constructor.getParameterTypes()));
    }

    System.out.println("----- declared constructors -----");
    for (Constructor constructor : c.getDeclaredConstructors()) {
        System.out.println(constructor.getName() + ", params: " + Arrays.toString(constructor.getParameterTypes()));
    }

    System.out.println("----- declared constructors detail -----");
    for (Constructor constructor : c.getDeclaredConstructors()) {
        System.out.println("name: " + constructor.getName());
        System.out.println("\tparameterTypes: " + Arrays.toString(constructor.getParameterTypes()));
        System.out.println("\tgenericParameterTypes: " + Arrays.toString(constructor.getGenericParameterTypes()));
        System.out.println("\tmodifiers: " + constructor.getModifiers());
    }

    System.out.println();
}
```

> 输出

```
##### testConstructorInfo #####
----- constructors -----
com.example.reflect.test4.ConstructorDemo, params: [double]
----- declared constructors -----
com.example.reflect.test4.ConstructorDemo, params: [double]
com.example.reflect.test4.ConstructorDemo, params: [float]
com.example.reflect.test4.ConstructorDemo, params: [int]
com.example.reflect.test4.ConstructorDemo, params: []
----- declared constructors detail -----
name: com.example.reflect.test4.ConstructorDemo
	parameterTypes: [double]
	genericParameterTypes: [double]
	modifiers: 1
name: com.example.reflect.test4.ConstructorDemo
	parameterTypes: [float]
	genericParameterTypes: [float]
	modifiers: 0
name: com.example.reflect.test4.ConstructorDemo
	parameterTypes: [int]
	genericParameterTypes: [int]
	modifiers: 4
name: com.example.reflect.test4.ConstructorDemo
	parameterTypes: []
	genericParameterTypes: []
	modifiers: 2
```

### Constructor 动态创建对象实例

> 测试方法

```java
void invokeConstructor() throws NoSuchMethodException, InstantiationException, IllegalAccessException, InvocationTargetException {
    System.out.println("##### invokeConstructor #####");

    Class<ConstructorDemo> c = ConstructorDemo.class;
    Constructor constructor = c.getDeclaredConstructor(int.class);
    ConstructorDemo demo1 = c.cast(constructor.newInstance(1));
    ConstructorDemo demo2 = c.cast(constructor.newInstance(2));

    System.out.println();
}
```

> 输出

```
##### invokeConstructor #####
create ConstructorDemo with i=1
create ConstructorDemo with i=2
```

本人自认为代码注释的很清楚了，见文知义，就不多加解释了

# 结语

呼终于搞定，中间输出了很多很多信息，`Field`、`Method`、`Constructor`、`Class` 等都拥有很多附带信息，这些类型相当于是把 .class 文件的类型信息都暴露出来，所以真正应用的时候可以选需要检查的部分和方法即可。

而这也是 Spring 框架中实现`依赖注入(DI = Dependency Injection)` 的底层原理，距离 Spring 实现原理有更进一步了，下一篇将要从 Spring Boot 的注解切入，先来看看 Java 原生的注解具备什么样的能力，来了解在 Spring 启动过程中所扮演的角色。
