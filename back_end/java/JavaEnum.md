# Java 基礎：Enum 枚舉

@[TOC](文章目錄)

<!-- TOC -->

- [Java 基礎：Enum 枚舉](#java-基礎enum-枚舉)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [常數定義](#常數定義)
  - [Enum 枚舉](#enum-枚舉)
    - [Definition 定義](#definition-定義)
      - [name 方法](#name-方法)
      - [ordinal 方法](#ordinal-方法)
    - [1. Basic Usage 基本用法](#1-basic-usage-基本用法)
      - [values 方法](#values-方法)
      - [valueOf 方法](#valueof-方法)
    - [2. Properties 成員變量](#2-properties-成員變量)
    - [3. Implement 實現接口](#3-implement-實現接口)
    - [4. Singleton 單例模式](#4-singleton-單例模式)
- [結語](#結語)

<!-- /TOC -->

## 簡介

本篇將要介紹 Java 中的`枚舉類型(Enum)`，枚舉類型的作用在於限定實例（或是說`對象(object)`的個數），有點像是常數集合的替代，但是它比一般的常數更強大也更安全，接下來將會介紹枚舉和一般常數的區別以及用法。

## 參考

<table>
  <tr>
    <td>Java 列舉(Enum)範例</td>
    <td><a href="https://matthung0807.blogspot.com/2017/10/java-enum.html">https://matthung0807.blogspot.com/2017/10/java-enum.html</a></td>
  </tr>
  <tr>
    <td>Java 枚举源码分析</td>
    <td><a href="https://blog.jrwang.me/2016/java-enum/">https://blog.jrwang.me/2016/java-enum/</a></td>
  </tr>
</table>

# 正文

## 常數定義

首先我們先來看到最基本的常數定義。當我們想要定義一組常數或是一個固定值的時候，為了避免`魔術值(magic number)` 的出現，我們很理所當然地會想到下列用法：

```java
public class Task {
  // Task State
  public static final int START = 1;
  public static final int ONGOING = 2;
  public static final int END = 3;
  public static final int ERROR = -1;
  // Task other logic
}
```

但是這時候可能會出現問題，第一個是你可能`沒辦法限定狀態的個數`，因為類中可能混雜了其他常量或是業務功能，我們希望`將限定狀態(枚舉狀態)的值歸納到同一個類中`，如下

```java
public class Task {
  // Task other logic
  class TaskState {
    public static final int START = 1;
    public static final int ONGOING = 2;
    public static final int END = 3;
    public static final int ERROR = -1;
  }
}
```

第二個問題是沒辦法進行有效的`類型檢查`，同時也不能保證`枚舉值的可靠性`，因為普通的常數定義可能出現如下情況

```java
public static final int START = 1;
public final int ONGOING = 2;
public static int END = 3;
public int ERROR = -1;
public static final String ALIVE = "keep alive";
```

接下來我們將使用 `Enum 枚舉`來歸納我們的限定類型

## Enum 枚舉

### Definition 定義

首先我們先來看一下枚舉類的定義的核心部分：

```java
public abstract class Enum<E extends Enum<E>> implements Comparable<E>, Serializable {
  private final String name;
  public final String name() {
    return name;
  }

  private final int ordinal;
  public final int ordinal() {
    return ordinal;
  }

  protected Enum(String name, int ordinal) {
    this.name = name;
    this.ordinal = ordinal;
  }

  public String toString() {
    return name;
  }
}
```

一個枚舉類最重要的兩個屬性 `name` 表示字面量（也就是對象名）、`ordinal` 表示順序（表示定義的索引）；同時我們看到 `toString` 方法可以看到當你直接打印一個枚舉對象時，將會直接輸出他的 `name` 本質上就是一個 String

#### name 方法

成員變量 `name` 保存了枚舉對象的字面量，也就是他的名字，而我們可以使用與他同名的方法來訪問

```java
Color.BLUE.name() // BLUE
```

#### ordinal 方法

`ordinal` 則紀錄著枚舉對象的序號，也就是定義的位置，它同樣也定義了一個同名方法用於訪問

```java
Color.BLUE.ordinal // 2
```

### 1. Basic Usage 基本用法

接下來我們看第一個例子，也是最簡單的僅僅定義限定值，沒有任何其他操作

```java
public enum Color {
  RED, GREEN, BLUE
}
```

我們先嘗試反編譯一下這段代碼

```java
public final class Color extends java.lang.Enum<Color> {
  public static final Color RED;
  public static final Color GREEN;
  public static final Color BLUE;
  public static Color[] values();
  public static Color valueOf(java.lang.String);
  static {};
}
```

我們可以看到所謂的`枚舉(Enum)`本質上就是創建多個類作為 `public static final` 的靜態變量，並且附加了兩個新方法：`values()` 和 `valueOf`

#### values 方法

使用 `Color.values()` 方法可以返回所有枚舉對象

```java
Color[] colors = Color.values();
for(Color color : colors) {
  System.out.println(color);
}

// output:
// RED
// GREEN
// BLUE
```

#### valueOf 方法

原本我們可以透過字面量 `Color.xxx` 來拿到對應的實例，我們還可以透過使用 `valueOf` 傳入字面量的字符串來獲取對象，這樣的靈活性和可編程性更高

```java
Color blue = Color.valueOf("BLUE")
```

### 2. Properties 成員變量

上面構造了最基礎的枚舉對象，僅僅是作為一個類別的符號，接下來我們想要擴展一下每一個枚舉對象

```java
public enum CustomerException {
  ControllerException(0, "unknown controller exception"),
  ServiceException(1, "unknown service exception"),
  DaoException(2, "unknown dao exception");

  private int code;
  private String message;

  CustomerException(int code, String message) {
    this.code = code;
    this.message = message;
  }

  public int getCode() {
    return code;
  }

  public String getMessage() {
    return message;
  }

  @Override
  public String toString() {
    return this.code + "\n" + this.message;
  }
}
```

- 測試

```java
CustomerException controllerException = CustomerException.ControllerException;
System.out.println(controllerException);
CustomerException serviceException = CustomerException.ServiceException;
System.out.println(serviceException.getCode());

// output:
// 0
// unknown controller exception
// 1
```

我們發現其實枚舉對象就是普通的對象實例，而 `ControllerException(0, "unknown controller exception")` 這種語法就好像是調用它的構造函數一樣 ，並且成員變量聲明為 `private` 使用 getter/setter 來調用。

### 3. Implement 實現接口

第三個例子我們將要使用枚舉類來實現接口

- IAnimal.java

```java
public interface IAnimal {
  void spark();
}
```

- Animal.java

```java
public enum Animal implements IAnimal {
  DOG("dog") {
    public void spark() {
      System.out.println("wang wang");
    }
  },
  CAT("cat") {
    public void spark() {
      System.out.println("miao miao");
    }
  };

  private String type;

  Animal(String type) {
    this.type = type;
  }
}
```

- Test.java

```java
Animal dog = Animal.DOG;
dog.spark();
Animal cat = Animal.CAT;
cat.spark();

// output:
// wang wang
// miao miao
```

如此一來枚舉類幾乎和一般的 class 沒什麼兩樣，也能夠實現接口(interface)，語法形式

```java
CAT("cat") {
  public void spark() {
    System.out.println("miao miao");
  }
};
```

是不是很像我們在表達式中直接實現抽象類時需要傳入類的接口實現一樣。

### 4. Singleton 單例模式

最後一個例子我們講解透過`枚舉(Enum)`來實現單例模式

- Singleton.java

```java
public enum Singleton {
  INSTANCE;

  private int id;

  Singleton() {
    this.id = 0;
  }

  public void doSomthing() {
    System.out.println("do something with same instance: id = " + id);
  }
}
```

- Test.java

```java
Singleton s1 = Singleton.INSTANCE;
s1.doSomthing();
Singleton s2 = Singleton.valueOf("INSTANCE");
s2.doSomthing();
System.out.println("s1 == s2 ? " + (s1 == s2));

// output:
// do something with same instance: id = 0
// do something with same instance: id = 0
// s1 == s2 ? true
```

我們將實例定義為枚舉類的唯一枚舉對象，就能夠實現單例模式啦！同時他也是線程(thread)安全的喔！供大家參考

# 結語

本篇介紹了枚舉類的基本形式，`name`、`ordinal`、`values`、`valueOf` 四個方法，還有使用`枚舉(Enum)`實現`接口(Interface)`，最後我們展示了一下最基礎的單例模式用法。從多線程的角度來看枚舉就是單例模式最好的實現，可以避免複雜的鎖的邏輯。下篇我們將會介紹使用枚舉配合 `@ExceptionHandler` 來實現 SpringMVC 中的全局異常處理，就可以不用在業務邏輯中間插一堆 `try ... catch` 塊了，不僅能夠將異常處理獨立出來，還能透簡化代碼提升可讀性，非常實用。
