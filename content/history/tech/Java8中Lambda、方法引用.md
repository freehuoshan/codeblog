---
title: Java8中Lambda、方法引用
id: 17
date: 2023-04-10 18:19:16
auther: FREEDOM
cover: https://codeblog.net/upload/2021/04/images%20(1)-d9a63e4e76744be3831911005fa15c0b.png
excerpt: 函数式接口段落引用函数式接口就是只定义一个抽象方法的接口（无论有多少个默认方法），可以通过 @FunctionalInterface 加以标识。Lambda 是函数式接口的实现，类似于匿名函数，只是它的实现只有参数和实现体，省去了重复的模板代码更加简洁。函数式接口的抽象方法定义可以视作 Lambda
permalink: /archives/java8-zhong-lambda-fang-fa-yin-yong
categories:
 - code
tags: 
 - java8
---

## 函数式接口


> 段落引用函数式接口就是只定义一个抽象方法的接口（无论有多少个默认方法），可以通过 @FunctionalInterface 加以标识。

Lambda 是函数式接口的实现，类似于匿名函数，只是它的实现只有参数和实现体，省去了重复的模板代码更加简洁。函数式接口的抽象方法定义可以视作 Lambda 表达式的函数签名。函数签名由参数和返回值两部分构成。Java8 提供了大部分常用函数接口可供日常开发使用。



函数式签名定义了一个函数的输入与输出，函数体实现由 Lambda 表达式完成。Lambda 的输入与输出应与其函数签名的输入输出匹配。

例如:

```java
 //函数签名
void printAttrs(T t);
 //Lambda
(Apple apple) -> System.out.println(apple.getColor());
(apple) -> System.out.println(apple.getColor());
```



对于常见函数签名，java　已经提供了很多，可以直接使用。https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html

![image.png](https://codeblog.net/upload/2021/03/image-38ec604496e14f2280bddea59a8069fb.png)

## 方法引用


Lambda 和方法引用都是将行为作为参数传递到一个方法中，达到传递行为的目的。只是 Lambda 是函数接口中函数签名的实现，而方法引用是直接引用已经存在的方法传递到其他方法中。

```java
 //引用普通方法
Integer::parseInt
List::contains
//引用构造函数
 Apple::new
```

可以将 Lambda 与方法引用统一理解，它们都是用于向方法传递行为，Lambda 是函数接口一个全新的实现，方法引用是引用已经存在的方法作为函数接口的实现。



## 总结


Java 为了向方法传递行为更加方便和灵活经过了这些演变: 多态->匿名类->Lambda->方法引用

多态:

在java8 以前如果一个方法中某个行为为了使其更加灵活，向这个方法传入行为的方式是通过一个参数，该参数是一个接口，可以通过向该接口参数传递不同的实现类达到传递不同行为的目的。

匿名类:

在java8 之前，如果向一个方法中传递行为，不想使用多态的方式，因为多态需要定义很多实现类，如果这个实现类只会使用这一次，只能使用匿名类。

Lambda:

java8 为向一个方法传递行为提供了更简易的方式，因为多态需要定义多个不同的实现类，匿名类的写法太繁琐且不必要，所以提供了Lambda 更加简洁。

方法引用:

Lambda适用于代码只使用一次的情况，方法引用可以引用已经存在的代码，减少重复。