---
title: Java8的使用：时间
date: 2018-09-06 23:38:13
tags:
  - 方法
  - 技巧
categories:
  - Java
---

在开发中不可避免都要对时间进行处理，尤其在处理财务方面的需求都时候，就经常需要考虑时间的问题。由于Java8之前标准库在时间方面的问题，对于时间都处理就十分都麻烦，所以在Java8的时候就根据第三方都Jodo库实现了一个新的时间库time库，我现在就简单的介绍下使用，也算式我最近经常使用时间库进行编程操作都一个总结吧：

**time库都操作基本都是函数式的**

# 时间转换

在对时间进行处理都时候，最常见的两个操作就是将字符串转换成时间或者在各种时间格式之间机型转换，这在之前基本上是要依赖`SimpleDateFormat`来进行操作，但是这样会存在许多问题，其中时区就可以能是其中一个问题

## 字符串转换时间

对于年月日或者年月日时分秒这种比较标准都时间格式来说，我们可以直接使用自带都方法进行转换:

```java
// 年-月-日
LocalDate.parse("date string");

// 年-月-日 时:分:秒
LocalDateTime.parse("date string");

// 年-月 这种一般在财务计算中常见
YearMonth.parse("date string");

```

对于一些需要自定义都时间格式来说，我们可以使用Java8自带都时间格式函数来进行转换：

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
LocalDate.parse("date string", formatter);
```

## 时间转字符串

在转换成时间都时候，如果式向数据比较标准的时间格式，其实直接使用`toString()`方法或者`String.valueOf()`就可以很好的处理时间数据了。但是向转换到一写特使格式都时候，我们同样可以使用`DateTimeFormatter`来完成我们都操作：

```java
LocalDate localDate = LocalDate.now();
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd LLLL yyyy");
String formattedString = localDate.format(formatter);
```

# 时间处理

对于时间都处理，一般分为取间隔，时间的比较，时间的增加，这些操作在标准库中都有比较好的实现，我们可以直接取过来使用：

## 时间间隔

在使用这个方法取得时间的间隔都时候，会存在一个问题：按月都时候不是开闭包含都；精确到天或者月的时候，最后一个月不足自然的半个月都时候式不算一个月都。

```java
long intervalMonth = ChronoUnit.MONTHS.between(startDate, endDate);
```

## 时间增减及比较

这个直接取时间库中都函数就可以完成：

```java
// 在这里，时间都操作式函数式的，每次都返回一个新的时间对象
LocalDate newDate = oldDate.plusMonths(1);

// 比较精都和时间的精度有关，两个日期比较就精确到天
boolean isBefore = sourceDate.isBefore(targetDate);
```

## 特殊操作

在进行时间处理都时候，经常需要取这个月有多少天，这个月都月末是什么时候的操作

```java
// 这个月都最后一天
YearMonth sourceDate = targetDate.with(TemporalAdjusters.lastDayOfMonth())

// 这个月的天数
int length = targetDate.lengthOfMonths();
```

# 时间转换

在以前，在对时间进行相互转换都时候需要借助字符串作为中间态，但是使用Java8的标准库之后，时间对象可以直接从大精度转换成小精度：

```txt
LocalDateTime -> LocalDate -> YearMonth
```

当由小精度转换成大精度都时候，只需要补全下精度就行了：

```java
LocalDate sourceDate = LocalDate.now();
LocalDateTime targetDate = sourceDate.atTime(20, 11, 22);
```
