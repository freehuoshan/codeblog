---
title: Java 向方法(函数)传递行为有几种方式
id: 19
date: 2023-04-10 18:19:16
auther: FREEDOM
cover: https://codeblog.net/upload/2021/04/download%20(3)-1e1c8a2be4074680801c4f21334deb4f.png
excerpt: 不管是程序语言的设计者还是程序语言的使用者－程序员除了追求程序运行的运行速度，还有开发程序的速度，然而他们还要追求程序的简单易读。所以他们希望程序运行的速度更快、开发速度更快、程序更简单易读。所谓开发程序的原则－&gt;不要重复，就是基于此。为了实现这三大目标程序语言设计者智慧发挥到了极点，例如－&
permalink: /archives/javaxiang-fang-fa--han-shu--chuan-di-xing-wei-you-ji-zhong-fang-shi
categories:
 - code
tags: 
 - java
---

不管是程序语言的设计者还是程序语言的使用者－程序员除了追求程序运行的运行速度，还有开发程序的速度，然而他们还要追求程序的简单易读。所以他们希望程序运行的速度更快、开发速度更快、程序更简单易读。所谓开发程序的原则－>不要重复，就是基于此。为了实现这三大目标程序语言设计者智慧发挥到了极点，例如－>面向对象、范型、Lambda...,程序语言的使用者总结了各种设计模式、工具库、开发框架....

  据说好的程序员就是有更好抽象能力的程序员，一次看 youtube TED　linus 评判两个代码片段哪个更好，他认为没有 if 的那个代码片段更好，因为 if 针对特殊情况做处理而没有 if 的代码端也实现了同样的功能，但是它更通用。换句话说没有 if 的那段代码抽象层次更高。

 向方法传递行为也是为了实现更高层次的抽象以实现上边的三大目标。


## 利用多态


 当某个方法内部某个行为需要根据传入的参数而发生改变时，将该行为作为参数传入可以获得最大的灵活性。在 java8 之前可以利用 java 的多态性，将该参数设置为一个接口类型，通过为该函数传入该接口参数类型的不同实现类，达到将多态方法的行为通过方法参数的方式传入方法。
```java
 class Apple{
  String color;
  int weight;
}

List<Apple> filter(List<Apple> apples, AppleFilter filter){
  List<Apple> result = new ArrayList<>();
  for(Apple apple : apples){
    if(filter.filt(apple)){
      result.add(apple);
    }
  }
} 
interface AppleFilter{
  boolean filt(Apple apple);
}

class ColorAppleFilter{
  boolean filt(Apple apple){
    return "green".equals(aapple.getColor);
  }
}
```

## 利用匿名函数


  第一种方法需要为每种行为创建一个实现类，如果这个实现类只使用一次可以通过匿名类的方式，更加方便。

```java
filter(apples, new AppleFilter(){
boolean filt(Apple apple){
    return "green".equals(apple.getColor);
  }
});
```

　匿名函数是第一种的特殊情况，创建了一个只使用一次的实现类。

## 利用Lambda


　　匿名函数固然相比于第一种方法为每种行为定义一个实现类简单很多（如果该行为只使用一次），但是代码显的繁琐，只有函数体是显的有用，其他都是多余的，Java8 提出了 Lambda.

```java
 filter(apples, (Apple apple) -> "green".equals(apple.getColor));
```

　Lambda 可以看做是匿名类的变体，也是创建了一个只使用一次的实现类，只是其接口是函数式接口(该接口只有一个抽象方法)。

## 方法引用

  java8 提供了 Lambda 已经足够好了，但是 lambda 和匿名类一样其存在在于其行为只会使用一次，如果你将要写的 lambda 其行为已经在其他类中或者库中存在，可以直接使用 Java8 提供的方法引用直接引用已经存在的方法即可。
```java
public class Apple {
　private String color;
　private int weight;
 
　public static boolean isGreen(Apple apple){
　　return "red".equals(apple.getColor());
　}
｝
 
public interface AppleFilter {
　　boolean filte(Apple apple);
}
 
filteApple(apples, Apple::isGreen);
 
public static void filteApple(List<Apple> apples, Predicate<Apple> fuc){
　List<Apple> result = new ArrayList<>();
　for (Apple apple: apples) {
 
　　if(fuc.test(apple)){
　　　result.add(apple);
　　}
　 
　}
}
```
  方法引用与　Lambda 本质是一致的都是为函数接口提供一个实现，只是 lambda 创建一个全新的实现而　方法引用是引用其他方法作为函数接口的实现。不管是lambda 还是方法引用的方法其方法签名必须与函数接口（要传入的目标方法的参数接口）的函数签名是一致的。

## 总结
这几种方式其本质是一致的都是为接口提供不同的实现(行为)，通过方法参数传入方法中。

1.多态:通过为接口提供不同实现类

2.匿名函数:为接口创建一个一次性实现类

3.Lambda:为函数接口中的函数签名创建一个一次性实现。

4.方法引用:为函数接口中的函数签名寻找一个已经存在的方法作为其实现。



方法2,3,4 都可以认为是方法1的变体，都是利用了多态性。java8 增加了 lamda 和方法引用这种语法糖以增加传递行为的灵活性。

