# Java 基础: 命令行使用 javac + jar 原生命令打包

@[TOC](文章目录)

<!-- TOC -->

- [Java 基础: 命令行使用 javac + jar 原生命令打包](#java-基础-命令行使用-javac--jar-原生命令打包)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [命令解析](#命令解析)
  - [默认打包（jar_test1）](#默认打包jar_test1)
  - [`MANIFEST.MF` 说明文件](#manifestmf-说明文件)
  - [自定义 MANIFEST.MF](#自定义-manifestmf)
- [结语](#结语)

<!-- /TOC -->

## 简介

自从写 Java 以来就没离开过 IDE，以前稍稍看过 maven 的概念也不是很懂。本篇就回到最纯粹的 Java 项目，使用 `javac + jar` 两个指令将项目打包成 jar 包，并简单说明 `MANIFEST.MF` 文件的作用。

## 参考

<table>
  <tr>
    <td>Java命令行下Jar包打包小结</td>
    <td><a href="https://www.jb51.net/article/131101.htm">https://www.jb51.net/article/131101.htm</a></td>
  </tr>
</table>

# 正文

## 命令解析

首先我们先简单介绍一下我们接下来要用的几个命令：`javac`、`jar`

> javac：编译源文件，实现 .java -> .class 的转换

![](https://picures.oss-cn-beijing.aliyuncs.com/img/java_package_javac_options.png)

> jar：打包成 JAR 包

![](https://picures.oss-cn-beijing.aliyuncs.com/img/java_package_jar_options.png)

接下来我们会用到的选项其实不多

| 使用选项 | 说明                              |
| -------- | --------------------------------- |
| javac -d | 指定编译目标目录                  |
| jar c    | 建立新的目标 jar 包（必覆盖旧的） |
| jar v    | 在输出流打印详细信息              |
| jar f    | 指定目标 jar 包名称               |
| jar m    | 指定内部结构配置文件              |
| jar -C   | 指定打包目录                      |

看说明看不懂就算了，用了才知道hhh。接下来会给出两个简易项目打包的过程

## 默认打包（jar_test1）

第一个场景是没有任何分包，同时也不编写任何配置文件，活生生的一个裸项目（哪天在生产环境看到应该会晕倒了），不过作为初入门使用 javac 倒是比较友好hhh

> 项目结构

```
/jar_test1
├── Makefile
├── Main.java
├── Test1.java
└── Test2.java
```

我们建立三个 Java 类：`Main.java`、`Test1.java`、`Test2.java`；以及接下来使用 make 指令的配置文件 `Makefile`

> 各文件内容

- `Test1.java`

```java
public class Test1 {
    public void display() {
        System.out.println("this is test1");
    }
}
```

- `Test2.java`

```java
public class Test2 {
    public void display() {
        System.out.println("this is test2");
    }
}
```

- `Main.java`

```java
public class Main {
    public static void main(String[] args) {
        for (String arg : args) {
            System.out.println("param: " + arg);
        }
        Test1 test1 = new Test1();
        test1.display();
        Test2 test2 = new Test2();
        test2.display();
    }
}
```

> 运行配置

- `Makefile`

```makefile
SOURCES := $(wildcard *.java) # 源文件：.java
CLASSES := $(patsubst %.java, %.class, $(SOURCES)) # 编译后中间文件：.class
TARGET := jar_test1.jar # 目标 jar 包

.PHONY : init clean run1 run2

init : clean
	javac -d . $(SOURCES)
	jar cvf $(TARGET) $(CLASSES)

run1 :
	java -jar jar_test1.jar

run2 :
	java -cp jar_test1.jar Main

clean :
	rm -f $(CLASSES) $(TARGET)
```

> 运行结果

第一个例子比较简单，我们直接指定 `-d .` 表示编译到当前目录下，然后再使用 `jar` 生成 jar 包，运行情况如下：

![](https://picures.oss-cn-beijing.aliyuncs.com/img/java_package_jar_test1_1.png)

但是当我们打算直接运行该 jar 包（run1）就会变成下面这样

![](https://picures.oss-cn-beijing.aliyuncs.com/img/java_package_jar_test1_2.png)

这是因为默认的 `MANIFEST.MF` 没有指定运行主类，我们只要在运行时指定主类并将 jar 包作为依赖的方式来启动就行了（run2）

![](https://picures.oss-cn-beijing.aliyuncs.com/img/java_package_jar_test1_3.png)

## `MANIFEST.MF` 说明文件

上面的例子提到一个 `MANIFEST.MF` 文件，但我们什么都没写啊？这是因为 `jar` 命令在你没提供的情况下帮你补上了，下面我们将 `jar_test1.jar` 解压缩能看到如下项目结构

```
/jar_test1.jar
├── META-INF
│   └── MANIFEST.MF
├── Main.class
├── Test1.class
└── Test2.class
```

我们可以看到三个刚刚写的类以及一个 `META-INF/MANIFEST.MF` 它的内容是（可能根据编译器的来源不同有不同的声明）

```
Manifest-Version: 1.0
Created-By: 1.8.0_265 (Amazon.com Inc.)
```

第一个例子中我们改成使用 `java -cp` 的方式来指定运行主类，另一种做法则是可以修改 `MANIFEST.MF` 的内容就能够跟我们预想的一样直接使用 `java -jar` 来运行一个 jar 包

```
Manifest-Version: 1.0
Created-By: 1.8.0_265 (Amazon.com Inc.)
Main-Class: Main
```

我们在 MANIFEST.MF 中加入 `Main-Class` 以指定运行主类。另外如果你的项目依赖了其他 jar 包，也可以在打包的时候一并包入，然后在 MANIFEST.MF 中加入 `Class-Path` 属性声明依赖，有点像是递归声明依赖(classpath)的感觉，如下：

```
Manifest-Version: 1.0
Created-By: 1.8.0_265 (Amazon.com Inc.)
Main-Class: Main
Class-Path: DepA.jar DepB.jar DepC.jar
```

## 自定义 MANIFEST.MF

上面提到我们可以修改 MANIFEST.MF，不过正确的做法不是在打包完之后再拆开来改，这样有点蠢；我们可以借助 `jar m` 选项在打包时选择我们写好的 MANIFEST.MF 说明文件

第二个打包示例我们再为每个类加入`包(package)`的特性

> 项目结构

```
/jar_test2
├── MANIFEST.MF
├── Makefile
└── src
    └── sample
        ├── main
        │   └── Main.java
        ├── pkg1
        │   └── Test1.java
        └── pkg2
            └── Test2.java
```

> 各文件内容

- `Test1.java`

```java
package sample.pkg1;

public class Test1 {
    public void display() {
        System.out.println("this is Test1 from sample.pkg1");
    }
}
```

- `Test2.java`

```java
package sample.pkg2;

public class Test2 {
    public void display() {
        System.out.println("this is Test2 from sample.pkg2");
    }
}
```

- `Main.java`

```java
package sample.main;

import sample.pkg1.Test1;
import sample.pkg2.Test2;

public class Main {
    public static void main(String[] args) {
        for (String arg : args) {
            System.out.println("param: " + arg);
        }
        Test1 test1 = new Test1();
        test1.display();
        Test2 test2 = new Test2();
        test2.display();
    }
}
```

- `MANIFEST.MF`

```java
Mainvest-Version: 1.0
Main-Class: sample.main.Main
```

> 运行配置

- `Makefile`

```makefile
SOURCES = $(wildcard src/*/*/*.java)
JAR = jar_test2.jar
CONFIG = MANIFEST.MF
PARAMS = 1 2 3 4

.PHONY : init clean build

build : clean
	mkdir target
	javac -d target $(SOURCES)
	jar cvfm $(JAR) $(CONFIG) -C target/ .

clean :
	rm -rf target $(JAR)

run :
	java -jar $(JAR) $(PARAMS)
```

> 运行结果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/java_package_jar_test2_1.png)

# 结语

本篇简单展示使用最原汁原味的 `javac + jar` 指令来编译打包 jar 包，`MANIFEST.MF` 本质上就是一个对于 jar 包内部结构的声明文件（更多内部配置细节可选项可在自己查一查），其实那些 IDE 或是其他打包工具也都是在这些原生功能上的封装和扩展甚至自动化的成果，理解最底层最原始样貌多少能加深对 jar 包的理解，之后看到复杂项目打包后的结果也不会那么迷茫了hhh。
