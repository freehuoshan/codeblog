---
title: Java8 中新添加的日期、时间 API
id: 7
date: 2023-04-10 18:19:14
auther: FREEDOM
cover: https://codeblog.net/upload/2021/04/download%20(1)-4bc963d9bce54b4ead2fb4d7770e2073.png
excerpt: 概览Java8 中引入了新的日期和时间 API 以解决之前版本中　java.util.Date 和 java.util.Calendar 的缺点。　　在开始介绍之前，我们先了解下旧版本的日期、时间 API 有哪些缺点以及 Java8 是如何解决的。　　我们还会研究下 java.time 包下的一些核
permalink: /archives/java8-zhong-xin-tian-jia-de-ri-qi-shi-jian-api
categories:
 - code
tags: 
 - java8
---

1. ## 概览

Java8 中引入了新的日期和时间 API 以解决之前版本中　java.util.Date 和 java.util.Calendar 的缺点。

　　在开始介绍之前，我们先了解下旧版本的日期、时间 API 有哪些缺点以及 Java8 是如何解决的。

　　我们还会研究下 java.time 包下的一些核心类，例如: LocalDate、LocalTime、LocalDateTime、ZonedDateTime、Period、Duration　和他们支撑的 API.

2. ## 已存在日期时间 API 的缺点

- 线程安全-Date 和 Claendar 类不是线程安全的，将让人头痛的并发问题留给了开发者而且需要为处理线程安全问题书写大量代码。然而，java8 中新引入的时期和时间 API 是不可修改的而且线程安全，所以开发者不会再为处理日期和时间的并发并发问题而头痛。
- 易于理解的 API 设计: Date 和　Calendar API 糟糕的设计使得即使是最常用的操作都很难。新的日期时间 API 遵循　ISO 标准，一致的日期、时间、持续时间、时间间隔　模型，各种各样的工具方法支持大多数常用操作。
- 时区-使用老版本 API 开发者需要大量逻辑代码处理时区问题，然而新的　API 可以使用 Local 和 ZonedDate 时间 API 处理时区问题。

3. ## 使用 LocalDate、LocalTime 和 LocalDateTime

最常使用的类是: LocalDate,LocalTime,LocalDateTime 就像它们的名字分别代表本地 日期和时间。

  这些类主要用于不需要明确时区的情况，这里的介绍会覆盖最常用的情况。

- ### 使用 LocalDate
LocalDate 代表的一个日期，在 ISO 中的格式是 (yyyy-MM-dd),不包含时间.

   它可以用来存储例如生日、付款日等。

　　创建当前日期可以基于当前系统时钟创建:
```java
LocalDate localDate = LocalDate.now();
```
LocalDate 代表指定年月日的一个特定日期，可以使用方法 of 或者 parse 创建，例如下边代码片段创建了一个代表 2015 年 2 月 20 日　的 LocalDate:
```java
LocalDate.of(2015, 02, 20);
LocalDate.parse("2015-02-20");
```
LocalDate 提供了很多工具方法用于获取各种各样的信息。让我们快速看一下这些 API 方法。
下面的代码创建了一个当前日期的 LocalDate 然后添加一天:

```java
LocalDate tomorrow = LocalDate.now().plusDays(1);
```
下面是在当前日期减去一个月，注意一下它是如何接受一个枚举类作为时间单位的。

```java
LocalDate previousMonthSameDay = LocalDate.now().minus(1, ChronoUnit.MONTHS);
```
　下面两个代码片段，我们解析日期 "2016-06-12" 分别获取该日期是星期几以及是当前月的第几天。
```java
DayOfWeek sunday = LocalDate.parse("2016-06-12").getDayOfWeek();
int　twelve = LocalDate.parse("2016-06-12").getDayOfMonth();
```
我们也可以检查一个日期是否为闰年，例如下面的例子我们检查今年是否为闰年:

```java
boolean leapYear = LocalDate.now().isLeapYear();
```
我们也可以检查两个日期哪个更早或者更迟。

```java
boolean　notBefore = LocalDate.parse("2016-06-12")
  .isBefore(LocalDate.parse("2016-06-11"));
 
boolean　isAfter = LocalDate.parse("2016-06-12")
  .isAfter(LocalDate.parse("2016-06-11"));
```
可以获得指定日期的时间边界，例如，一个日期的时间起始边界是  00:00:00 ，该日期所在的月份起始日是，1号:

```java
LocalDateTime beginningOfDay = LocalDate.parse("2016-06-12").atStartOfDay();
LocalDate firstDayOfMonth = LocalDate.parse("2016-06-12")
  .with(TemporalAdjusters.firstDayOfMonth());
```

- ### 使用 LocalTime

LocalTime 代表时分秒，没有日期

就像　LocalDate , 可以基于当前系统时钟使用方法 of 或者 parse 创建一个 LocalTime 实例，下面让我们快速看一下常用的方法:
基于当前系统时钟创建一个　 LocalTime 实例:

```java
LocalTime now = LocalTime.now();
```
下面我们通过解析一个字符创建了一个代表 06:30AM 的 LocalTime 实例:

```java
LocalTime sixThirty = LocalTime.parse("06:30");
```
工厂方法 of 可以用于创建一个 LocalTime 实例,例如下面代码片段使用工厂方法创建了一个代表 06:30AM 的 LocalTime 实例:

```java
LocalTime sixThirty = LocalTime.of(6, 30);
```

下面的例子中我们通过解析一个字符串创建了一个 LocalTime 并使用 plus 方法添加了一个小时，结果应该是 07:30AM:

```java
LocalTime sevenThirty = LocalTime.parse("06:30").plus(1, ChronoUnit.HOURS);
```
可以通过各种 get 方法获取指定的时间单位值,例如时、分、秒。

```java
int six = LocalTime.parse("06:30").getHour();
```

我们也可以比较两个时间的先后，例如西面代码就是比较两个 LocalTime ，结果应该是 True:

```java
boolean isbefore = LocalTime.parse("06:30").isBefore(LocalTime.parse("07:30"));
```
可以通过　LocalTime 类中的常量获取一天中的最迟，最早和中午时间，对于查询数据库中指定时间范围内的数据会非常有用,例如下面的代码代表 23:59:59

```java
LocalTime maxTime = LocalTime.MAX
```
- ### 使用 LocalDateTime

LocalDateTime 用于代表日期和时间的组合.

当我们需要日期和时间的组合时，这是我们最常使用的类。该类提供了各种 API 方法，让我们看一下最常使用的用法:
就像　LocalDate 和 LocalTime ,可以获取当前系统时钟的 LocalTime 实例:

```java
LocalDateTime.now();
```
下面的代码示例展示了如何通过工厂方法 of 和 parse 获取一个实例，创建的　LocalDateTime 实例代表　2015　年　2　月 20 日　06:30AM

```java
LocalDateTime.of(2015, Month.FEBRUARY, 20, 06, 30);
LocalDateTime.parse("2015-02-20T06:30:00");
```
而且提供了一些工具方法可以在指定时间单位上加减，例如，年月日时分秒，下面代码示例展示了方法 plus 和 minus 。这些 API 与 LocalDate 和 LocalTime 中对应方法作用相同:

```java
localDateTime.plusDays(1);
localDateTime.minusHours(2);
```
类似于日期和时间类中的 get 方法， LocalDateTime 也提供了一些 get  方法用于获取指定时间单位值，例如下面代码实例获取一个实例的月份:
```java
localDateTime.getMonth();
```

4. ## 使用 ZonedDateTime API

当我们需要对指定日期和时间处理时区问题时， Java8 提供了 ZonedDateTime。ZoneId 是用于标识不同时区的标识符，有大约 40 个不同的时区。

下面代码片段创建了一个巴黎时区:
```java
ZoneId zoneId = ZoneId.of("Europe/Paris");
```
可以获取所有 Zone Ids:
```java
Set<String> allZoneIds = ZoneId.getAvailableZoneIds();
```
LocalDateTime 可以转换为指定时区:
```java
ZonedDateTime zonedDateTime = ZonedDateTime.of(localDateTime, zoneId);
```
ZonedDateTime 提供了 parse 方法用于解析时间与时区:

```java
ZonedDateTime.parse("2015-05-03T10:15:30+01:00[Europe/Paris]");
```
使用時區的另一種方法是使用OffsetDateTime。 OffsetDateTime是帶有偏移量的日期時間的不可變表示形式。此類存儲所有日期和時間字段（精度為納秒）以及與UTC /格林威治的偏移量。

OffSetDateTime 实例可以像下面通过 ZoneOffSet 创建。这里我们创建了一个 LocalDateTime 代表 2015 年 2 月 20 日 6:30AM

```java
LocalDateTime localDateTime = LocalDateTime.of(2015, Month.FEBRUARY, 20, 06, 30);
```
然後，通過創建ZoneOffset將時間增加了兩個小時,並為localDateTime實例進行設置，：
```java
ZoneOffset offset = ZoneOffset.of("+02:00");
 
OffsetDateTime offSetByTwo = OffsetDateTime
  .of(localDateTime, offset);
```
现在我们创建了一个 LocalTime 2015-02-20 06:30 +02:00. 接下来介绍如何使用 Period 和 Duration 类修改日期和时间。

5. ## 使用 Period 和 Duration

Period 类代表以年月日为单位的时间量，Duration 类代表以秒和纳秒为单位的时间量。
- ### 使用 Period

Period 类被用于修改指定日期和获取两个时间的时间差:

```java
LocalDate initialDate = LocalDate.parse("2007-05-10");
```

该日期可以像下面代码片段使用 Period 操作:

```java
LocalDate finalDate = initialDate.plus(Period.ofDays(5));
```

Period 类有各种 get 方法例如 getYears、getMonths、getDays 从一个 Period 对象获取相应值。下面代码我们想要获取两个日期的时间差，返回值应该是 5:

```java
int five = Period.between(initialDate, finalDate).getDays();
```
Period 获取两个日期的时间差可以通过  ChronoUnit.between 指定单位:

```java
long five = ChronoUnit.DAYS.between(initialDate, finalDate);
```
返回值应该是 5,接下来我们继续研究 Duration 类。

- ### 使用 Duration

类似于 Period, Duration 类用于处理时间。下面的代码我们创建了一个 LocalTime 代表 06:30 然后添加了　30 秒　生成一个代表　06:30:30am 的　LocalTime 。

```java
LocalTime initialTime = LocalTime.of(6, 30, 0);
 
LocalTime finalTime = initialTime.plus(Duration.ofSeconds(30));
```
两个时刻的时间差可以通过　Duration 或者通过指定时间单位获得，在下边第一个代码段我们使用 Duration 的 between() 方法获取 finalTime 和 initialTime 的时间差并以秒为单位返回:

```java
long thirty = Duration.between(initialTime, finalTime).getSeconds();
```
第二个例子中我们使用 ChronoUnit 类的 between() 方法执行同样的操作:

```java
long thirty = ChronoUnit.SECONDS.between(initialTime, finalTime);
```
接下来我们看一下如何将旧有 Date 和 Calendar 为新的 Date/Time.

6. ## 兼容 Date 和 Calendar

Java8 添加了 toInstant() 方法帮助将旧有 Date 和 Calendar 实例转换为新 Date Time API,就像下面代码片段:
```java
LocalDateTime.ofInstant(date.toInstant(), ZoneId.systemDefault());
LocalDateTime.ofInstant(calendar.toInstant(), ZoneId.systemDefault());
```
LocalDateTime 可以通过 unix 纪元秒构造，就像下面代码的结果 LocalDateTime 代表 2016-06-13T11:34:50

```java
LocalDateTime.ofEpochSecond(1465817690, 0, ZoneOffset.UTC);
```
下面我们关注日期和时间的格式化。

7. ## 日期和时间的格式化

Java8 提供的 API 使得格式化 Date 和 Time 非常容易。

```java
LocalDateTime localDateTime = LocalDateTime.of(2015, Month.JANUARY, 25, 6, 30);
```
下面代码通过 ISO 日期格式格式化本地日期，结果应该是 2015-01-25:

```java
String localDateString = localDateTime.format(DateTimeFormatter.ISO_DATE);
```
DateTimeFormatter 提供了各种各样的标准格式，也可以为格式化方法提供自定义模式，像下面，返回的 LocalDate 应该是 2015/01/25:

```java
localDateTime.format(DateTimeFormatter.ofPattern("yyyy/MM/dd"));
```
作為格式化選項的一部分，我們可以採用格式為SHORT，LONG或MEDIUM的格式。下面的代码会输出LocalDateTime 通过 25-Jan-2015, 06:30:00 

```java
localDateTime
  .format(DateTimeFormatter.ofLocalizedDateTime(FormatStyle.MEDIUM))
  .withLocale(Locale.UK);
```
下面我们介绍一些 Java8 核心日期时间 API 的替代方案。

8. ## 反向移植和替代选项

- ### 使用 Threeten 项目

對於正在從Java 7或Java 6遷移到Java 8並希望使用日期和時間API的組織，Threeten 項目提供了反向移植功能。開發人員可以使用該項目中可用的類來實現與新的Java 8 Date and Time API相同的功能，一旦他們移至Java 8，就可以切換軟件包。

```xml
<dependency>
    <groupId>org.threeten</groupId>
    <artifactId>threetenbp</artifactId>
    <version>1.3.1</version>
</dependency>
```
- ### Joda-Time 库

另一个可替代 Java8 日期和时间 API 的是 Joda-Time 库，事实上 Java8 日期时间 API 由 Joda-Time 库作者(Stephen Colebourne) 和 Oracle 公共领导开发。这个库提供了几乎所有 Java8 日期时间 API支持的功能。
```xml
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.9.4</version>
</dependency>
```
9. ## 总结

Java 8提供了一組具有一致API設計的豐富API，以簡化開發。


翻译自: https://www.baeldung.com/java-8-date-time-intro 