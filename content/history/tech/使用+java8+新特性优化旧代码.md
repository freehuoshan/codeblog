---
title: 使用 java8 新特性优化旧代码
id: 14
date: 2023-04-10 18:19:15
auther: FREEDOM
cover: https://codeblog.net/upload/2021/04/java8-features-5cf20f59f198451c8834b12330baf64c.png
excerpt: java8 很重要的进步就是提供了 lambda、方法引用、StreamAPI.这些新添加的特性终极目的就是为了提高 java 代码的简洁易读。lambda 替换匿名类在 java8 之前对于只使用一次的行为只能使用匿名类的方式，但是 Java8中Lambda、方法引用 提供了更加简洁的方式处理此类
permalink: /archives/shi-yong-java8xin-te-xing-you-hua-jiu-dai-ma
categories:
 - code
tags: 
 - java8
---

java8 很重要的进步就是提供了 lambda、方法引用、StreamAPI.这些新添加的特性终极目的就是为了提高 java 代码的简洁易读。
![image.png](https://codeblog.net/upload/2021/03/image-226b1b7de7ae4eb89e0f5ef7bce10bfe.png)

## lambda 替换匿名类

在 java8 之前对于只使用一次的行为只能使用匿名类的方式，但是 Java8中Lambda、方法引用 提供了更加简洁的方式处理此类情况。

```java
Runnable r1 = new Runnable() {
 @Override
 public void run() {
  System.out.println("hello world");
 }
};
//java8
Runnable r2 = ()-> System.out.println("hello world");
```
如果已经存在多年或者正在开发的项目中有大量使用匿名类，像上边那样，重构为 java8 lambda 的方式更为简洁。但是匿名类的使用通常是调用方法时作为参数传递，而该方法有可能是重载的情况，如果多个方法是重载的情况而且其函数参数签名也一致，这种情况会发生冲突。

![image.png](https://codeblog.net/upload/2021/03/image-c564bd5518c1478b80f9e1b7660ddad8.png)

```java
Job job = new Job();
 
//before java8
job.doSomeThing(new Task() {
 @Override
 public void execute() {
  System.out.println("do some tasks in java7");
 }
});
 
job.doSomeThing(new Runnable() {
 @Override
 public void run() {
  System.out.println("do some job in java7");
 }
});
 
//java8
//job.doSomeThing(() -> System.out.println("do some tasks in java8"));
 
job.doSomeThing((Task) () -> System.out.println("do some tasks in java8"));
job.doSomeThing((Runnable) () -> System.out.println("do some job in java8"));

interface Task{
 public void execute();
}
 
class Job{
 
 public void doSomeThing(Task task){
  task.execute();
 }
 
 public void doSomeThing(Runnable r){
  r.run();
 }
}

```

## 方法引用替代 Lambda

如果 lambda 中有大量逻辑代码，最好将其放到一个方法中以简明易懂的方式描述其功能，然后以方法引用的方式使用，这样可以不用详细查看代码便可理解整个数据处理流程。

  Java8 提供的 Collectors API 中提供了大量常用数据处理方法，如果可以满足需求优先引用这些静态方法而不是自己再次使用 lambda 的方式实现一遍。

## StreamAPI 替换命令式处理

使用 java8 提供的 StreamAPI 替换以前处理数据代码中使用循环遍历等命令式编程代码。可以大大提高代码质量。  

## 设计模式的再优化

java 中大多数设计模式都是利用的接口和多态性以更方便的方式传递行为，增加传递行为的灵活性。但是 java8 提供了 lambda 和 方法引用，如果将其应用到这些设计模式的使用中可以增加更多的灵活性。比如某些时候可以之前传入一个 lambda 作为一个实现而不需要单独创建一个特定的实现类。