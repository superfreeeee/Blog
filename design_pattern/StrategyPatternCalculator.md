# 设计模式: Strategy Pattern 策略模式

@[TOC](文章目錄)

<!-- TOC -->

- [设计模式: Strategy Pattern 策略模式](#设计模式-strategy-pattern-策略模式)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [應用場景](#應用場景)
  - [Roles 角色](#roles-角色)
  - [舉例：計算器（Java 實現）](#舉例計算器java-實現)
    - [角色對應](#角色對應)
    - [Classes Definition 類定義](#classes-definition-類定義)
    - [Benefit 優勢](#benefit-優勢)
- [結語](#結語)

<!-- /TOC -->

## 簡介

GoF 四人幫所寫的 Design Pattern 堪稱 OOP 的聖典，裏頭的 23 種策略模式至今都不斷的在使用。本篇介紹的策略模式(Strategy Pattern)屬於行為型設計模式，著重在對於行為的封裝，與之相似的抽象工廠(Abstract Factory)則是著重在對象的創建上，需要將兩者區分清楚。

## 參考

<table>
    <tr>
        <td>Strategy Design Pattern in Java – Example Tutorial</td>
        <td><a href="https://www.journaldev.com/1754/strategy-design-pattern-in-java-example-tutorial">https://www.journaldev.com/1754/strategy-design-pattern-in-java-example-tutorial</a></td>
    </tr>
</table>

# 正文

## 應用場景

在同樣的上下文(context)中，某種複雜的計算目標可能有多種算法選擇，這時就可以考慮使用策略模式，將多種複雜的計算抽象成策略，可以將算法細節和上下文對象分離開來，使得客戶(Client)能在不改動上下文方法和策略接口情的況下添加新的計算策略。

## Roles 角色

![](https://wtfhhh.oss-cn-beijing.aliyuncs.com/strategy_pattern_uml.png)

- Context：表示上下文環境，存在複雜操作所需要的計算參數。
- Strategy：策略接口，抽象出計算策略的公共接口，可選的接受一些必要參數。
- ConcreteStrategy：具體策略，實現具體計算策略

## 舉例：計算器（Java 實現）

### 角色對應

- Context <-> IntegerCalculator
- Strategy <-> OperationStrategry
- ConcreteStrategy <-> AddOperation, SubOperation, MulOperation, DivOperation

### Classes Definition 類定義

- Calculator：上下文對象

```java
public class IntegerCalculator {

    private HashMap<String, OperationStrategy> operations;

    // 默認註冊四種計算策略
    public IntegerCalculator() {
        operations = new HashMap<String, OperationStrategy>();
        registerOperation("add", new AddOperation());
        registerOperation("sub", new SubOperation());
        registerOperation("mul", new MulOperation());
        registerOperation("div", new DivOperation());
    }

    // 處理計算
    public void solve(String cmd) {
        String[] args = cmd.split(" ");
        String op = args[0];
        int x = Integer.parseInt(args[1]);
        int y = Integer.parseInt(args[2]);

        OperationStrategy operation = operations.get(op);
        if(operation == null) {
            System.out.println("unknown operation");
        } else {
            int result = operation.calculate(x, y);
            System.out.println("cmd: " + cmd);
            System.out.println("result: " + result);
        }
    }

    // 可由外部添加計算策略
    public void registerOperation(String op, OperationStrategy strategy) {
        operations.put(op, strategy);
    }
}
```

- OperationStrategy：抽象策略接口

```java
public interface OperationStrategy {
    int calculate(int x, int y);
}

```

- xxxOperation：具體計算策略實現

```java
public class AddOperation implements OperationStrategy {
    public int calculate(int x, int y) {
        return x + y;
    }
}

public class SubOperation implements OperationStrategy {

    public int calculate(int x, int y) {
        return x - y;
    }
}

public class MulOperation implements OperationStrategy {
    public int calculate(int x, int y) {
        return x * y;
    }
}

public class DivOperation implements OperationStrategy {
    public int calculate(int x, int y) {
        if(y == 0) {
            return -1;
        }
        return x / y;
    }
}
```

- Client：客戶端

```java
public class Client {
    public static void main(String[] args) {
        String[] commands = new String[]{
                "add 200 70",
                "sub 200 70",
                "mul 200 70",
                "div 200 70",
                "gcd 200 70"
        };
        IntegerCalculator calculator = new IntegerCalculator();
        // 添加 "計算最大公因數" 的策略
        calculator.registerOperation("gcd", new OperationStrategy() {
            public int calculate(int x, int y) {
                if(x < 0 || y < 0) {
                    return -1;
                }
                if(x < y) {
                    x = x ^ y;
                    y = x ^ y;
                    x = x ^ y;
                }
                int tmp;
                while(y > 0) {
                    tmp = y;
                    y = x % y;
                    x = tmp;
                }
                return x;
            }
        });
        // 對每條指令進行計算
        for(String cmd : commands) {
            calculator.solve(cmd);
        }
    }
}
```

- Result 運行結果

```
cmd: add 200 70
result: 270
cmd: sub 200 70
result: 130
cmd: mul 200 70
result: 14000
cmd: div 200 70
result: 2
cmd: gcd 200 70
result: 10
```

### Benefit 優勢

策略模式的好處在於，複雜計算被抽象成一個策略接口，具體使用的對象由外部或構造函數注入，可在不修改 Client 和 Context 的情況下添加不同的計算策略，符合 OCP 原則：對擴展計算策略開放，對修改 Client 和 Context 的方法關閉。

# 結語

本篇介紹的策略模式可以很好的抽象出某種複雜行為，提升代碼的可修改性。所謂的策略模式僅僅是一個思想，實際運用的時候不應該被代碼的形式或語言特性所約束，只要是在符合 OOP 的幾大原則之下，都可以對實現代碼做出更進一步的抽象和封裝。
