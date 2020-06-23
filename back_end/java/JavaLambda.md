# Java 基礎：Lambda 表達式

@[TOC](文章目錄)

<!-- TOC -->

- [Java 基礎：Lambda 表達式](#java-基礎lambda-表達式)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Lambda 表達式](#lambda-表達式)
  - [@FunctionalInterface](#functionalinterface)
  - [Example](#example)
    - [Customer Functional Interface 自定義函數式接口](#customer-functional-interface-自定義函數式接口)
      - [原始用法：匿名內部類實現](#原始用法匿名內部類實現)
      - [使用 Lambda + 顯示類型](#使用-lambda--顯示類型)
      - [使用 Lambda + 類型推導](#使用-lambda--類型推導)
    - [java.lang.Runnable](#javalangrunnable)
    - [java.util.Comparator](#javautilcomparator)
    - [其他](#其他)
- [結語](#結語)

<!-- /TOC -->

## 簡介

今天要來介紹 Java 中的 Lambda 表達式。Java8 的時候引入了 Lambda 表達式，本質上他就是一個`匿名函數`，在 Python 或是 JavaScript 這些語言中，函數屬於`一級公民(First-class citizen)`，意思就是`函數(Function)`與一般變量的層級是相同的，一樣可以透過名字來訪問：

```js
// In JavaScript
function func1() {}
console.log(func1)
// [Function: func1]

const func2 = function () {}
console.log(func2)
// [Function: func2]
```

但是在 C++ 和 Java 這類語言中，`函數(Function)`是作為`類(Class)`的`方法(Method)`的形式來表現的：

```java
// In Java
public class Human {
  private String name;

  public void greeting() {
    System.out.println("Hello I am " + name);
  }
}
```

我們馬上來揭開 Java8 的 Lambda 表達式的神秘面紗。

## 參考

<table>
  <tr>
    <td>JAVA 8 函数式接口 - Functional Interface</td>
    <td><a href="https://www.cnblogs.com/chenpi/p/5890144.html">https://www.cnblogs.com/chenpi/p/5890144.html</a></td>
  </tr>
  <tr>
    <td>Java8 API</td>
    <td><a href="https://docs.oracle.com/javase/8/docs/api/">https://docs.oracle.com/javase/8/docs/api/</a></td>
  </tr>
</table>

# 正文

## Lambda 表達式

我們簡單描述一下 Lambda 表達式的意義，它是由 lambda 演算衍伸出來的應用，從語言層面上來看他就是一個匿名函數，可以將一個函數作為參數傳入另一個函數，這樣的能力和應用在 JS 的應用中是非常常見的(如`回調函數 callback`)。

在許多語言中都支持(Python、JavaScrip、C#、Java8 ...)，大部分都是使用`箭頭函數`的形式來表現，這邊舉一個使用 JS 的例子：

```js
const add = (x, y) => x + y
const greeting = (name) => {
  console.log(`hello ${name}`)
}
```

## @FunctionalInterface

但是在 Java 中，除了基本類型(Primitive type)之外，操作的基本單位是`類(Class)`，透過`引用類型(Reference type)`來訪問透過類創建出來的`對象實例(Object)`。

那為了要在 Java 中實現 Lambda 表達式就引入了一個注解：`@FunctionalInterface`。這個注解必須裝飾在`接口類(Interface)`上，並且這個接口`只能存在一個方法`，也就是接下來我們要直接聲明的匿名函數本體。由於 Java 是強類型語言，以 `@FunctionalInterface` 注解聲明的類就好像是這個匿名函數的`類型`一樣：

- Runnable.java

```java
@FunctionalInterface
public interface Runnable {
  public abstract void run();
}
```

`@FunctionalInterface` 類也能夠存在多個方法，但是除了我們自己定義的函數接口之外，只能存在另外三種方法：

1. 方法簽名都必須與 Object 類的 public 一樣
2. 存在`默認實現(default)`
3. `靜態(static)`方法

- Comparator.java

```java
@FunctionalInterface
public interface Comparator<T> {
  // 主方法，也就是使用 Lambda 時實現的方法
  int compare(T o1, T o2);

  // 與 Object 相同函數簽名的方法
  boolean equals(Object obj);

  // 存在默認實現 defautl 的方法
  default Comparator<T> reversed() {
    return Collections.reverseOrder(this);
  }

  // 靜態方法 static
  public static <T extends Comparable<? super T>> Comparator<T> reverseOrder() {
    return Collections.reverseOrder();
  }

  // ...
}
```

## Example

接下來我們舉幾個例子就更清楚

### Customer Functional Interface 自定義函數式接口

我們舉一元運算符和二元運算符做例子，先看函數式接口定義

- UnaryOperator.java

```java
@FunctionalInterface
public interface UnaryOperator<T> {
  T exec(T t);
}
```

- BinaryOperator.java

```java
@FunctionalInterface
public interface BinaryOperator<T> {
  T exec(T t1, T t2);
}
```

接下來的使用我們需要區分成下面幾種

#### 原始用法：匿名內部類實現

第一種使用方式是在 Java8 以前沒有 Lambda 的時候，需要使用匿名內部類的方式實現接口

```java
UnaryOperator<Integer> minus = new UnaryOperator<Integer>() {
  @Override
  public Integer exec(Integer integer) {
    return -integer;
  }
};
BinaryOperator<Integer> add = new BinaryOperator<Integer>() {
  @Override
  public Integer exec(Integer t1, Integer t2) {
    return t1 + t2;
  }
};
System.out.println(minus.exec(123));  // -123
System.out.println(add.exec(1, 2));  // 3
```

#### 使用 Lambda + 顯示類型

接下來就可以使用 Lambda 來簡化（表達式形式：`() -> {}`），第一個版本是最麻煩的 Lambda 表達式，需要加上完整的類型

```java
// 直接返回值
UnaryOperator<Integer> minusWithType = (Integer i) -> -i;
// 加上函數體
BinaryOperator<Integer> addWithType = (Integer x, Integer y) -> {
  return x + y;
}

System.out.println(minusWithType.exec(123));  // -123
System.out.println(addWithType.exec(1, 2));  // 3
```

#### 使用 Lambda + 類型推導

Lambda 表達式還能夠省略參數的類型（接口定義已經指定過了），Java 使用 invokedynamic 將會自動實現`類型推導(Type Inference)`，這才是最簡潔版本的 Lambda 也是它最迷人之處。

```java
UnaryOperator<Integer> minusWithInference = i -> -i;
BinaryOperator<Integer> addWithInference = (x, y) -> x + y;

System.out.println(minusWithInference.exec(123));  // -123
System.out.println(addWithInference.exec(1, 2));  // 3
```

到此我們已經完全學會怎麼樣定義自己的`函數式接口(@FunctionalInterface)`啦，就是這麼簡單啦！接下來我們舉幾個 JDK 中已經有的函數式接口做例子和使用範例吧。

### java.lang.Runnable

首先第一個例子就是大名鼎鼎的 `Runnable.java`，多線程必學的接口沒有之一。先來看看定義：

- Runnable.java

```java
@FunctionalInterface
public interface Runnable {
  public abstract void run();
}
```

我們可以看到這個接口全部就只有一個 `run` 方法，接下來我們分別使用原始的`匿名內部類(Anonymous inner-class)`以及 `Lambda 表達式`兩種實現方式

- 原始用法：Anonymous inner-class 匿名內部類

```java
new Thread(new Runnable() {
  @Override
  public void run() {
    System.out.println("run");
  }
}).start();
```

- 使用 Lambda 表達式

```java
new Thread(() -> {
  System.out.println("run");
}).start();
```

### java.util.Comparator

第二個例子是實現比較時會用到的接口 `Comparator<T>`，一樣先上定義

- Comparator.java

```java
@FunctionalInterface
public interface Comparator<T> {
  int compare(T o1, T o2);
  // ...
  // 省略其他方法
}
```

`Comparator<T>` 定義了一些默認方法和靜態方法，這邊主要專注在函數式接口的主要函數 `compare` 上，接下來一樣分成原始寫法和 Lambda 寫法

- 原始寫法

```java
Integer[] numArray = new Integer[]{3,2,4,1,5,5,1,4,2,3};
List<Integer> nums = new ArrayList<>();
Collections.addAll(nums, numArray);

nums.sort(new Comparator<Integer>() {
  @Override
  public int compare(Integer o1, Integer o2) {
    return o1 - o2;
  }
});
System.out.println(nums);
// [1, 1, 2, 2, 3, 3, 4, 4, 5, 5]

nums.sort(new Comparator<Integer>() {
  @Override
  public int compare(Integer o1, Integer o2) {
    return o2 - o1;
  }
});
System.out.println(nums);
// [5, 5, 4, 4, 3, 3, 2, 2, 1, 1]
```

- Lambda 寫法

```java
Integer[] numArray = new Integer[]{3,2,4,1,5,5,1,4,2,3};
List<Integer> nums = new ArrayList<>();
Collections.addAll(nums, numArray);

nums.sort((a, b) -> a - b);
System.out.println(nums);
// [1, 1, 2, 2, 3, 3, 4, 4, 5, 5]

nums.sort((a, b) -> b - a);
System.out.println(nums);
// [5, 5, 4, 4, 3, 3, 2, 2, 1, 1]
```

有沒有發現寫法簡化了不只一點呢，還不快用起來

### 其他

其他有用到 `@FunctionalInterface` 的接口還有如下這些：

- `java.util.concurrent.Callable`

```java
@FunctionalInterface
public interface Callable<V> {
  // 主方法
  V call() throws Exception;
}
```

- `java.util.function.Consumer`

```java
@FunctionalInterface
public interface Consumer<T> {
  // 主方法
  void accept(T t);

  default Consumer<T> andThen(Consumer<? super T> after) {
    Objects.requireNonNull(after);
    return (T t) -> { accept(t); after.accept(t); };
  }
}
```

- `java.util.function.Predicate`

```java
@FunctionalInterface
public interface Predicate<T> {
  // 主方法
  boolean test(T t);

  default Predicate<T> and(Predicate<? super T> other) {
    Objects.requireNonNull(other);
    return (t) -> test(t) && other.test(t);
  }
  default Predicate<T> negate() {
    return (t) -> !test(t);
  }

  default Predicate<T> or(Predicate<? super T> other) {
    Objects.requireNonNull(other);
    return (t) -> test(t) || other.test(t);
  }
  static <T> Predicate<T> isEqual(Object targetRef) {
    return (null == targetRef)
          ? Objects::isNull
          : object -> targetRef.equals(object);
  }
}

```

- `java.util.function.Supplier`

```java
@FunctionalInterface
public interface Supplier<T> {
  // 主方法
  T get();
}
```

還有其他用到 `@FunctionalInterface` 的接口這邊就不一一列出了。之後我們會獨立出一篇專門講解 `java.util.funciton` 包中的類和方法，是 Lambda 表達式寫法更多的擴展用法

# 結語

本篇我們學習了在 Java 中使用 Lambda 表達式，雖然定義上比 JavaScript 的`箭頭函數(arrow function)` 要麻煩許多，但是在類型限定上卻更為健壯且安全。下一篇將要介紹 `java.util.funciton` 包中關於 Lambda 表達式更多的擴展用法，敬請期待。
