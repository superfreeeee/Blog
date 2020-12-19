# Java 基础: 浅谈类型基础 - Class 对象

@[TOC](文章目录)

<!-- TOC -->

- [Java 基础: 浅谈类型基础 - Class 对象](#java-基础-浅谈类型基础---class-对象)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [浅谈 Class 对象的前生后世](#浅谈-class-对象的前生后世)
    - [Class 对象的加载时机](#class-对象的加载时机)
  - [初识 Class 对象](#初识-class-对象)
  - [Class 类型方法](#class-类型方法)
    - [获取类信息](#获取类信息)
    - [获取类型继承链](#获取类型继承链)
    - [创建对象实例 & 类型转换](#创建对象实例--类型转换)
    - [类型检查](#类型检查)
    - [完整代码仓库](#完整代码仓库)
- [结语](#结语)

<!-- /TOC -->

## 简介

本文的起因要说到最近在研究 Spring Cloud 的多种服务。由于入门时直接从 Spring Boot 入门，大部分的服务或依赖都已经封装的非常完善，开箱即用，反而对于底层的运行机制不慎了解。作为一个有抱负的程序员（bushi，不能接受这样懵懵懂懂瞎用，所以就打算从应用底层的 `Java 反射机制(Reflect)`从头学习一遍。

本篇将作为学习 Java 反射的预备知识，在真正进入反射之前我们先来了解以下什么是 `Class 对象`，它在 Java 运行的过程中扮演什么样的角色。

## 参考

<table>
  <tr>
    <td>Thinking in Java - 原书第 4 版</td>
    <td><a href=""></a></td>
  </tr>
  <tr>
    <td>深入理解 Java 虚拟机 - 周志明</td>
    <td><a href=""></a></td>
  </tr>
  <tr>
    <td>Java中Class对象详解</td>
    <td><a href="https://blog.csdn.net/mcryeasy/article/details/52344729">https://blog.csdn.net/mcryeasy/article/details/52344729</a></td>
  </tr>
  <tr>
    <td>java中类加载时机</td>
    <td><a href="https://www.cnblogs.com/aspirant/p/9036010.html">https://www.cnblogs.com/aspirant/p/9036010.html</a></td>
  </tr>
</table>

# 正文

## 浅谈 Class 对象的前生后世

要运行 Java 就得要从 JVM（Java Virtual Machine）谈起。学过一点 Java 的应该都知道，与 C/C++ 直接编译成二进制文件不同，Java 会将源文件编译成 `.class` 字节码，并在运行时由 JVM 动态加载。

在 Java 的世界中万物皆对象(Object)，而在强类型的限制下每个对象势必有自己的类型，Java `编译(compile)`时就会为每个类型(class)生成一个 .class 文件。接下来就轮到`运行(runtime)`时的主角 JVM 登场了，JVM 解析 .class 文件的过程可粗略分成三个阶段：

- 加载(Load)：由`类加载器(Class Loader)`通过`全限定名(包名+类名)`加载代表 .class 文件的二进制流，并由该字节码生成一个 `Class 对象`。

- 链接(Link)：验证 .class 文件中的信息有效性和正确性、静态域空间分配和初始化、常量池符号引用。

- 初始化(initialize)：真正开始运行对象内容，静态块初始化(`static {}`)、若存在父类则优先初始化父类。

这边我们最关心的是第一个阶段的 `Class 对象`，它也是 JVM 运行时最核心的部分，有了 Class 对象才有类型的存在。

备注：有关 JVM 运行机制、.class 文件结构、类加载器(Class Loader)等内容可以参考其他资料，本篇不再细说。

### Class 对象的加载时机

Class 对象通常会在`第一次使用`时被 JVM 加载，而所谓的"第一次使用"可以是：

- 创建对象实例（使用 `new` 关键字）
- 访问静态域（`XXX.field` 或是 `XXX.func()`）
- 透过反射(`java.lang.reflect`)机制对类方法进行调用时
- 子类正在被加载
- 运行时指定的主类（运行主入口 `main` 所在的类）

也就是说 JVM 并不会一次创建甚至加载好所有类型，而是`按需加载(懒加载)`。这部分就使得 JVM 有很大的可伸缩性，我们可以修改类加载的规则，也可以在运行后才动态的输入二进制流来添加新的类型，使得 Java 语言天生能和网络结合在一起，只要存在于更大的网络之中就能够动态接受二进制流以识别新的类型。

## 初识 Class 对象

说了这么多，我们马上就来看看 Class 对象到底都躲在哪里

```java
package com.example.classobject.test1;

public class Test {
    public static void main(String[] args) {
        Class c = Test.class;
        System.out.println("Test.class: " + c);
        Test t1 = new Test();
        Test t2 = new Test();
        Test t3 = new Test();
        System.out.println(c == t1.getClass());
        System.out.println(c == t2.getClass());
        System.out.println(c == t3.getClass());
    }
}
```

```
Test.class: class com.example.classobject.test1.Test
true
true
true
```

最简单的方法我们直接调用 `Test.class` 就可以拿到该类型的 Class 对象，我们也可以调用对象实例的 `getClass()` 方法获取 Class 对象。注意！由于 Class 对象是与类型一一对应的，所以不管是直接调用 `Test.class` 或是 `t1.getClass()`，其所指向的 Class 对象都是同一个，如下图所示

![](https://picures.oss-cn-beijing.aliyuncs.com/img/java_class_object_relationship.png)

## Class 类型方法

下面我们简单介绍以下 Class 类型的可用接口（这边仅仅谈论与类型相关的，`Field`、`Method` 相关的留到反射的篇幅说明）

> 静态方法

| Static Method                | Description                                                 |
| ---------------------------- | ----------------------------------------------------------- |
| static Class forName(String) | 根据传入的全限定名查找 Class 对象（第一次访问会进行类加载） |

> 获取类信息方法

| Method                    | Description                   |
| ------------------------- | ----------------------------- |
| String getName()          | 获取类型名称                  |
| String getSimpleName()    | 获取简单名称（省略包名）      |
| String getCanonicalName() | 获取标准完整名称              |
| Class getSuperclass()     | 获取父类的 Class 对象         |
| Class[] getInterfaces()   | 获取实现接口的 Class 对象列表 |

> 构造方法 & 类型转换

| Method               | Description  |
| -------------------- | ------------ |
| Object newInstance() | 创建类型实例 |
| T cast(Class\<T\>)   | 类型转换     |

> 类型检查

| Method             | Description      |
| ------------------ | ---------------- |
| isMemberClass()    | 检查是否为内部类 |
| isInterface()      | 检查是否为接口   |
| isEnum()           | 检查是否为枚举   |
| isAnnotation()     | 检查是否为注解   |
| isArray()          | 检查是否为数组   |
| isPrimitive()      | 检查是否为内置   |
| isAnonymousClass() | 检查是否为匿名类 |
| isLocalClass()     | 检查是否为局部类 |

下面我们就给出几个 Class 对象方法测试

### 获取类信息

> 类型定义

```java
/* IA.java */
package com.example.classobject.test1;
public interface IA {}

/* IB.java */
package com.example.classobject.test1;
public interface IB {}

/* IC.java */
package com.example.classobject.test1;
public interface IC {}

/* A.java */
package com.example.classobject.test1;
public class A {
    A() {}
    A(int i) {}
}
/* B.java */
package com.example.classobject.test1;
public class B extends A implements IA, IB, IC {
    B() { super(1); }
}
```

> 测试类

```java
package com.example.classobject.test1;

public class Main {

    /**
     * 查看 Class 对象信息
     * @param c
     */
    private static void info(Class c) {
        System.out.println("----- info -----");
        System.out.println("Class name: " + c.getName()); // 获取类型名称
        System.out.println("is interface: " + c.isInterface()); // 检查是否为接口
        System.out.println("Simple name: " + c.getSimpleName()); // 简单名称
        System.out.println("Canonical name: " + c.getCanonicalName()); // 全限定名
    }

    public static void main(String[] args) {
        Class c = null;
        try {
            // 获取 B 类型的 Class 对象
            c = Class.forName("com.example.classobject.test1.B");
        } catch (ClassNotFoundException e) {
            System.out.println("B not found");
            System.exit(1);
        }

        info(c); // 展示 B 类型的 Class 对象信息
        for(Class i : c.getInterfaces()) {
            info(i); // 展示 B 类型的所有接口 Class 信息
        }

        Class pc = c.getSuperclass(); // 获取父类即 A 的 Class 对象
        Object p = null;

        try {
            p = pc.newInstance(); // 创建父类实例
        } catch (InstantiationException e) {
            System.out.println("unable to instantiate");
            System.exit(1);
        } catch(IllegalAccessException e) {
            System.out.println("unable to access");
            System.exit(1);
        }
        info(p.getClass()); // 由对象获取 Class 信息（类型为 A）
    }
}
```

> 输出

```
----- info -----
Class name: com.example.classobject.test1.B
is interface: false
Simple name: B
Canonical name: com.example.classobject.test1.B
----- info -----
Class name: com.example.classobject.test1.IA
is interface: true
Simple name: IA
Canonical name: com.example.classobject.test1.IA
----- info -----
Class name: com.example.classobject.test1.IB
is interface: true
Simple name: IB
Canonical name: com.example.classobject.test1.IB
----- info -----
Class name: com.example.classobject.test1.IC
is interface: true
Simple name: IC
Canonical name: com.example.classobject.test1.IC
----- info -----
Class name: com.example.classobject.test1.A
is interface: false
Simple name: A
Canonical name: com.example.classobject.test1.A
```

### 获取类型继承链

第二个例子我们演示如何获取一个类型的完整继承链

> 类型定义

```java
/* A.java */
package com.example.classobject.test2;
public class A {}

/* B.java */
package com.example.classobject.test2;
public class B extends A {}

/* C.java */
package com.example.classobject.test2;
public class C extends B {}
```

> 测试类

```java
package com.example.classobject.test2;

public class Main {

    private static void printInheritList(Class c) {
        Class pc;
        // 递归检索父类，直到 null（Object.class.getSuperclass() 为 null）
        while ((pc = c.getSuperclass()) != null) {
            System.out.println(c.getName() + " extends " + pc.getName());
            c = pc;
        }
    }

    public static void main(String[] args) {
        try {
            Class c = Class.forName("com.example.classobject.test2.C");
            printInheritList(c);
        } catch (ClassNotFoundException e) {
            System.out.println("Class C not found");
            System.exit(1);
        }
    }
}
```

> 输出

```
com.example.classobject.test2.C extends com.example.classobject.test2.B
com.example.classobject.test2.B extends com.example.classobject.test2.A
com.example.classobject.test2.A extends java.lang.Object
```

我们可以看到最简单的类型 A 默认是继承 Object 类型的，也就是所有对象（类型）的祖先类型必定是 Object。

### 创建对象实例 & 类型转换

第三个例子我们透过 Class 对象来动态创建类型实例，并给出一般类型转换之外的另一种方法。

> 类型定义

- `SuperClass.java`

```java
package com.example.classobject.test3;

public class SuperClass {

    // 无参数构造函数
    public SuperClass() {
        System.out.println("SuperClass()");
    }

    // 带参数构造函数
    public SuperClass(int i) {
        System.out.println("SuperClass(int i)");
    }

}
```

- `SubClass.java`

```java
package com.example.classobject.test3;

public class SubClass extends SuperClass {

    // 无参数构造函数
    public SubClass() {
        super(1);
        System.out.println("SubClass()");
    }
}
```

> 测试类

```java
package com.example.classobject.test3;

public class Main {

    public static void main(String[] args) {
        testSuperClassInstance();
        testTypeCasting();
    }

    private static void testSuperClassInstance() {
        try {
            Class<SubClass> c = SubClass.class; // 子类 Class 对象
            SubClass subClass = c.newInstance(); // 创建子类实例
            System.out.println("after create subClass");

            // pc.newInstance return type "capture of ? super SubClass"
            Class<? super SubClass> pc = c.getSuperclass(); // 父类 Class对象
            Object superClass = pc.newInstance(); // 创建父类实例
            System.out.println("after create superClass");
        } catch (IllegalAccessException e) {
            System.out.println("Illegal access");
            System.exit(1);
        } catch (InstantiationException e) {
            System.out.println("Instantiation error");
            System.exit(1);
        }
    }

    private static void testTypeCasting() {
        SuperClass sup = new SubClass();
        Class<SubClass> subType = SubClass.class;
        // 另一种类型转换 cast()
        // SubClass.class.cast(obj) 与 (SubClass) obj 等价
        SubClass sub = subType.cast(sup); // equals: sub = (SubClass) sup;
    }
}
```

> 输出

```
SuperClass(int i)
SubClass()
after create subClass
SuperClass()
after create superClass
SuperClass(int i)
SubClass()
```

我们可以看到，当我们使用 `newInstance()` 方法只能调用默认构造函数，待到反射的时候我们再来看看要如何调用带参数构造函数（良好的代码风格可以使用`设计模式`中的工厂方法来取代重载构造函数）。

### 类型检查

最后一个例子是检测 Class 对象的类型。在 Java 中一个 class 可能有各种形态：一般类型、内部类型、枚举类型、注解类型、数组类型、内置类型、匿名类型、局部类型，这些信息 Class 对象也能识别！马上来看看

> 类型定义

- `A.java`

```java
package com.example.classobject.test4;
// 一般类型
public class A {
    // 内部类型
    public class B {
    }
}
```

- `C.java`

```java
package com.example.classobject.test4;
// 接口类型
public interface C {}
```

- `D.java`

```java
package com.example.classobject.test4;
// 枚举类型
public enum D {}
```

- `E.java`

```java
package com.example.classobject.test4;
// 注解类型
public @interface E {}
```

- `F.java`

```java
package com.example.classobject.test4;

public abstract class F {
    public static F createAnonymous() {
        // 匿名类型
        return new F() {
        };
    }

    public static F createLocal() {
        // 局部类型
        class G extends F {}
        return new G();
    }
}
```

> 测试类

```java
package com.example.classobject.test4;

public class Test {

    public static void main(String[] args) {
        check(A.class); // 一般类
        check(A.B.class); // 内部类
        check(C.class); // 接口类
        check(D.class); // 枚举类
        check(E.class); // 注解类
        check(int.class); // 内置类
        check(new int[0].getClass()); // 数组类
        check(F.createAnonymous().getClass()); // 匿名类
        check(F.createLocal().getClass()); // 局部类
    }

    private static void check(Class c) {
        System.out.println("----- check " + c.getSimpleName() + " -----");
        System.out.println("isMemberClass: " + c.isMemberClass() + ", isInterface: " + c.isInterface() + ", isEnum: " + c.isEnum() + ", isAnnotation: " + c.isAnnotation());
        System.out.println("isArray: " + c.isArray() + ", isPrimitive: " + c.isPrimitive() + ", isAnonymousClass: " + c.isAnonymousClass() + ", isLocalClass: " + c.isLocalClass());
    }
}
```

> 输出

```
----- check A -----
isMemberClass: false, isInterface: false, isEnum: false, isAnnotation: false
isArray: false, isPrimitive: false, isAnonymousClass: false, isLocalClass: false
----- check B -----
isMemberClass: true, isInterface: false, isEnum: false, isAnnotation: false
isArray: false, isPrimitive: false, isAnonymousClass: false, isLocalClass: false
----- check C -----
isMemberClass: false, isInterface: true, isEnum: false, isAnnotation: false
isArray: false, isPrimitive: false, isAnonymousClass: false, isLocalClass: false
----- check D -----
isMemberClass: false, isInterface: false, isEnum: true, isAnnotation: false
isArray: false, isPrimitive: false, isAnonymousClass: false, isLocalClass: false
----- check E -----
isMemberClass: false, isInterface: true, isEnum: false, isAnnotation: true
isArray: false, isPrimitive: false, isAnonymousClass: false, isLocalClass: false
----- check int -----
isMemberClass: false, isInterface: false, isEnum: false, isAnnotation: false
isArray: false, isPrimitive: true, isAnonymousClass: false, isLocalClass: false
----- check int[] -----
isMemberClass: false, isInterface: false, isEnum: false, isAnnotation: false
isArray: true, isPrimitive: false, isAnonymousClass: false, isLocalClass: false
----- check  -----
isMemberClass: false, isInterface: false, isEnum: false, isAnnotation: false
isArray: false, isPrimitive: false, isAnonymousClass: true, isLocalClass: false
----- check G -----
isMemberClass: false, isInterface: false, isEnum: false, isAnnotation: false
isArray: false, isPrimitive: false, isAnonymousClass: false, isLocalClass: true
```

输出看起来有些吓人，不过就是一些类型信息

### 完整代码仓库

<a href="https://github.com/superfreeeee/Blog-code/tree/main/back_end/java/java_class_object">https://github.com/superfreeeee/Blog-code/tree/main/back_end/java/java_class_object</a>

# 结语

本篇稍微介绍了 Class 对象的来由还有一些基本类型相关的方法，下一期将开始正式进入 `Java 反射`的环节。
