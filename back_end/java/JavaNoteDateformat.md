# Java 踩坑笔记: YYYY-MM-dd 的误用

@[TOC](文章目录)

<!-- TOC -->

- [Java 踩坑笔记: YYYY-MM-dd 的误用](#java-踩坑笔记-yyyy-mm-dd-的误用)
  - [简介](#简介)
  - [参考](#参考)
  - [完整示例代码](#完整示例代码)
- [正文](#正文)
  - [测试代码](#测试代码)
  - [问题来源](#问题来源)
- [结语](#结语)

<!-- /TOC -->

## 简介

可能对于熟悉 java 的开发者来说已经是老生常谈了，本篇将要来说说在 `SimpleDateFormat` 中使用 `YYYY-MM-dd` 可能会带来的问题。

## 参考

<table>
  <tr>
    <td>听说又有兄弟因为用YYYY-MM-dd被锤了...</td>
    <td><a href="https://blog.csdn.net/dyc87112/article/details/111948436?utm_source=app&app_version=4.5.0">https://blog.csdn.net/dyc87112/article/details/111948436?utm_source=app&app_version=4.5.0</a></td>
  </tr>
  <tr>
    <td>SimpleDateFormat(Java Platform SE 8)</td>
    <td><a href="https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html">https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html</a></td>
  </tr>
</table>

## 完整示例代码

<a href="https://github.com/superfreeeee/Blog-code/tree/main/back_end/java/java_note_dateformat">https://github.com/superfreeeee/Blog-code/tree/main/back_end/java/java_note_dateformat</a>

# 正文

## 测试代码

废话不多说，让我们直接来看看哪段代码出啥毛病了

```java
/* Tests */
public class Tests {

    private SimpleDateFormat df1 = new SimpleDateFormat("YYYY-MM-dd");
    private SimpleDateFormat df2 = new SimpleDateFormat("yyyy-MM-dd");

    @Test
    public void test() {
        Calendar calendar = Calendar.getInstance();

        for (int date = 25; date <= 28; date++) {
            calendar.set(2020, 11, date);
            show(calendar.getTime());
        }
    }

    private void show(Date date) {
        System.out.printf("YYYY-MM-dd: %s, yyyy-MM-dd: %s\n", df1.format(date), df2.format(date));
    }
}
```

- 输出

```
YYYY-MM-dd: 2020-12-25, yyyy-MM-dd: 2020-12-25
YYYY-MM-dd: 2020-12-26, yyyy-MM-dd: 2020-12-26
YYYY-MM-dd: 2021-12-27, yyyy-MM-dd: 2020-12-27
YYYY-MM-dd: 2021-12-28, yyyy-MM-dd: 2020-12-28
```

嗯？我们输出的应该是2020年12月的25~28号，但是我们看到使用`YYYY-MM-dd`格式化的后两个日期的年份变成2021年了。

## 问题来源

根据 Java 8 Doc 文档的描述

![](https://picures.oss-cn-beijing.aliyuncs.com/img/java_text_simpledateformat.png)

我们可以看到格式化表达式中 `Y` 的类型是 **Week year**，而所谓的 week year 指的是根据**该周所属的年份**进行表示，也就是说只要该周(周日到周六)中经历的1月1日，那么这一天的年份就属于下一年的日期了

![](https://picures.oss-cn-beijing.aliyuncs.com/img/calendar_2020_12.png)

从日历中我们就可以很清楚的看到，12/25、26是属于2020年的，而12/27、28则是与2021/1/1同一周，所以在`YYYY-MM-dd`的表示下为2021年。

# 结语

除非像是银行账单或是其他特殊用途的日期计算方法，大部分时间我们还是希望那一天是哪一年就显示哪一年，所以尽量还是使用`yyyy-MM-dd`，只在需要的时候使用`YYYY-MM-dd`。
