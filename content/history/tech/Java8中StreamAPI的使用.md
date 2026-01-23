---
title: Java8中StreamAPI的使用
id: 16
date: 2023-04-10 18:19:16
auther: FREEDOM
cover: https://codeblog.net/upload/2021/04/download%20(2)-e041d1129f444be9b2986ed7f7756c7a.png
excerpt: 在 java8 之前我們處理數據的方式大體可以這樣講：通過集合(Collection)API 例如，List、Map 這些以特定常用的數據結構存儲數據，使用for、while 等循環遍歷的方式對原集合中的數據進行修改或者組裝到新的集合中。這個過程可以抽象為 數據源-&gt;循環遍歷-&gt;對每個
permalink: /archives/java8-zhong-streamapi-de-shi-yong
categories:
 - code
tags: 
 - java8
---

在 java8 之前我們處理數據的方式大體可以這樣講：通過集合(Collection)API 例如，List、Map 這些以特定常用的數據結構存儲數據，使用for、while 等循環遍歷的方式對原集合中的數據進行修改或者組裝到新的集合中。這個過程可以抽象為: 數據源->循環遍歷->對每個數據元素處理->組裝數據->處理結果。

1. 數據源->通常是一個數據集（數據）
2. 循環遍歷->遍歷數據源中每個元素 (行為)
3. 對每個元素處理->針對每個元素進行特殊處理(行為)
4. 組裝數據->配合2,3的結果組裝數據(行為)
5. 處理結果->4的結果，一個數據或者一個數據集(數據)
  
StreamAPI 與 CollectionAPI 不存在競爭關係，CollectionAPI 重點在於數據結構，StreamAPI 重點在於處理數據的行為。在 Java8以前我們使用 CollectionAPI 以特定數據結構存儲數據，但是關於數據的處理行為例如:循環遍歷、對於每個數據如何處理、組裝數據等細節需要程序員手動實現每個細節，然而這些數據處理行為通常很重複導致大量重複模板代碼，在 java8 以前由於傳遞行為不夠靈活，對與這種尷尬似乎沒有很好的解決辦法。

Java8 提供了 lambda 和方法引用使得傳遞行為的靈活性大大提高，對於處理數據中的重複行為可以以更優雅的方式實現。StreamAPI 便是專門為了優化數據處理而提供的一套 API

## 使用StreamAPI處理數據的方式

在以上提到的處理數據的流程中 1,5 不用多說，一個是數據源一個是數據結果，他們不涉及行為只使用 CollectionAPI 存儲數據即可。關於循環遍歷 java8 以前沒有對數據處理中的行為提供API，需要我們自己實現遍歷，這也是之前處理數據大量重複代碼的主要原因，但是 java8 提供的 StreamAPI 已經將循環遍歷內置，不需要程序員再手動寫這些重複的遍歷模板代碼，只要向StreamAPI 提供你要遍歷的數據集即可。關於針對每個元素的處理行為，可以通過 java8 提供的 lambda/方法引用向 StreamAPI 傳遞。關於數據的組裝，在java8之前也需要我們手動實現，但是 java8 的 StreamAPI 提供了大量組裝數據相關 api 可以滿足常用數據結構的組裝方式，如果無法滿足特殊需求，程序員可以實現自己的組裝器。

因为 StreamAPI 已经将循环遍历内置，可以推断 StreamAPI 会为就数据处理和数据组装提供相应API.

没错，只需要向 StreamAPI 传入数据集， 然后调用StreamAPI提供的数据处理方法向其传入针对每个元素想要执行的行为，然后调用 StreamAPI 提供的数据组装收集方法向其传入组装器(StreamAPI 提供的或者自定义的)，便完成了整个数据处理流程。

```java
List<Apple> list = Arrays.asList(
                new Apple("red", 20, "china"),
                new Apple("green", 30, "us"),
                new Apple("blue", 30, "china"),
                new Apple("red", 20, "us"),
                new Apple("green", 40, "us"),
                new Apple("red", 20, "china")
        );
 
        Map<String, List<Apple>> map = list.stream()
                .filter(apple -> "china".equals(apple.getCountry()))
                .collect(groupingBy(Apple::getColor));
 
 
//        Integer totalWeight = list.stream().collect(reducing(0, Apple::getWeight, (x, y) -> x + y));
        System.out.println(map);
```
这段代码将来自china 的苹果按照颜色分组，组装到一个 map 结构中并打印输出。

## 生成数据源

将 collectionAPI 的数据结构的数据使用 StreamAPI 处理需要先将 Collection 装换为 Stream 。例如上例中:

```java
list.stream()
```

可以将其类比为要将装在箱子中的产品放到流水线上准备加工的准备过程。将静态数据结构转换为流结构

- 创建空流: Stream.empty()
- list生成流: list.stream()
- 数据生成流: Arrays.stram(arr)
- 值生成流: Stream.of("a", "b", "c")
- 文件生成流: File.lines(file)
- 函数生成流: Stream.iterate(0, n-> n+2),Stream.generate(Math::random)

## 处理数据

StreamAPI 针对处理数据的不同需求提供了大量可用的方法。

- 筛选数据方法 filter
   该方法要求传入的参数是函数式接口 Predicate, 其函数签名是 (T t) -> boolean, 所以使用中向其传入符合其函数签名的 lambda 或者方法引用即可。传入的行为会应用到待处理数据集的每个数据上用于筛选。

- 去除重复项方法 distinct
   该方法会去除掉数据集中重复的元素，根据数据流 Stream 中元素的 equals 和 hashCode 决定是否为重复项。

- 只提取部分元素 limit
   该方法会舍弃流中其他元素，只保留指定个数的元素

- 跳过部分元素 skip
   该方法会跳过流中前 n 个元素

- 将流中处理的元素映射或者转换为其他元素 map
- 将嵌套流扁平化 flatMap
   该方法是map 的特殊情况，map 用于将非流数据装换为另一个非流数据，flatMap 用于将流数据装换为非流数据(也就是将子流中的数据汇入到当前流中)

- 是否匹配,这些方法的参数与 filter 类似也是传入一个函数签名为 (T t)-> boolean 的函数接口用于查找和匹配
	- 是否有匹配的数据 anyMatch
	- 是否所有数据匹配 allMatch
	- 是否都不匹配 noneMatch
- 查找数据，这些方法参数与匹配方法类似，只是这些方法返回的不是 boolean 值，而是数据元素。
	- 查找符合参数行为的任何数据元素 findAny
	- 查找符合参数行为的第一个数据元素 findFirst
- 用于求和或者最大值最小值，将结果归结为一个数据的操作: reduce (该操作功能强大，要想了解需要深入学习)
- 以上所有方法都是基于 Stream 流，java8 提供了专门针对数值计算的数值流 IntStream、DoubleStream、LongStream
	- Stream 转换为数值流 mapToInt, mapToDouble, mapToLong
	- 数值流转换为对象流 Stream: boxed()


## 收集组装


  StreamAPI 提供的 collect 方法用于组装数据，将结果数据组装为自己想要的数据结构。该方法要求传入一个组装器，可以使用 java8 提供的，也可以使用自己创建的。java8已经提供的组装器有:

- 将流中所有数据元素收集到一个 List 数据结构中： toList
- 将流中所有数据元素收集到一个 Set 数据结构中: toSet
- 将流中所有数据元素收集到一个指定的数据结构中: toCollection
- 计算流中数据元素的个数: counting
- 对流中数据元素一个整数属性求和: summingInt
- 对流中数据元素一个整数属性求平均值: averagingInt
- 将流中所有元素以字符串方式连接起来: joining
- 获取流中最大元素 maxBy
- 获取流中最小元素 minBy
- 将流中元素根据自己需求归结为一个值: reducing
- 包裹另一个包装器对其结果应用转换函数: collectingAndThen
- 对流中数据元素进行分组、多级分组: groupingBy
- 对流中数据元素进行分区: partitioningBy, 是分组的特殊情况