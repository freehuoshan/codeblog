---
title: Java8 中的默認方法
id: 11
date: 2023-04-10 18:19:13
auther: FREEDOM
cover: https://codeblog.net/upload/2021/04/java8int-9f02ff648405404687f3b52b943b0004.png
excerpt: java8 引入的默認方法允許開發者向接口添加新方法而不會影響到已經存在的該接口的實現，它爲接口提供了更大的靈活性，當實現類沒有提供該方法的實現時默認方法會作爲其默認實現。　例如如下代碼public interface oldInterface {  public void existingMet
permalink: /archives/java8zhong-de-mo-ren-fang-fa
categories:
 - code
tags: 
 - java8
---

java8 引入的默認方法允許開發者向接口添加新方法而不會影響到已經存在的該接口的實現，它爲接口提供了更大的靈活性，當實現類沒有提供該方法的實現時默認方法會作爲其默認實現。

　例如如下代碼:
```java
public interface oldInterface {
  public void existingMethod();
  default public void newDefaultMethod() {
    System.out.println("New default method"
    " is added in interface");
  }
}
```
下面的代碼使用 JDK8　會成功編譯:

```java
public class oldInterfaceImpl implements oldInterface {
  public void existingMethod() {
    // existing implementation is here…
  }
}
```
```java
oldInterfaceImpl obj = new oldInterfaceImpl ();
// print “New default method add in interface”
obj.newDefaultMethod();
```
## 爲什麼要引入默認方法

重新設計已有的 JDK 總是非常複雜的。修改　JDK 中的一個接口,會破壞所有使用它的實現類，這意味着添加任何新方法會破壞數百萬行代碼。所以接口引入默認方法是作爲　JDK 保持向後兼容的一種實現機制。    

  一個接口中提供默認方法不會影響實現類，實現類也可以重寫接口提供的默認方法。

　Java8 的 JDK 擴展了集合框架，而且 forEach　方法被添加到了整個集合中（與　lambda 結合使用）。如果使用常規方式，代碼應該是這樣的:

```java
public interface Iterable<T> {
 public void forEach(Consumer<? super T> consumer);
}
```
但是因爲這樣，該接口的每個實現類都會編譯錯誤，所以引入一個默認方法是爲了不影響已經存在的實現。

下面是接口 Iterable 接口的默認方法:
```java
public interface Iterable<T> {
  public default void forEach(Consumer<? super T> consumer) {
    for (T t : this) {
      consumer.accept(t);
    }
  }
}
```
 同樣機制被用於 JDK 的　Stream 接口以免破壞實現類。

## 默認方法與抽象類的差異

介紹完默認方法，現在接口似乎與抽象類沒什麼不一樣。然而，在 java8　中他們依然是不同的概念。

  抽象類可以定義構造方法，而且可以定義屬性。與之相比，默認方法只能調用其他接口方法，而且接口不可以定義屬性變量。因此接口和抽象方法基於需求用於不同目的。

## 默認方法與多重繼承的歧義問題

因爲 java 類可以實現多個接口，而每個接口可以定義一個名字簽名相同的默認方法。這些繼承方法會造成衝突。

見以下代碼:

```java
public interface InterfaceA { 
  default void defaultMethod(){ 
    System.out.println("Interface A default method"); 
  } 
}

public interface InterfaceB {
  default void defaultMethod(){
    System.out.println("Interface B default method");
  }
}

public class Impl implements InterfaceA, InterfaceB  {
}

```
以上代碼編譯會報以下錯誤
```java
java: class Impl inherits unrelated defaults for defaultMethod() 
```
爲了解決這個問題，我們需要提供一個默認方法的實現:

```java
 public class Impl implements InterfaceA, InterfaceB {
   public void defaultMethod(){
   }
}
```
進一步,如果我們想要調用父接口中的一個默認實現而不想要自己實現，我們可以像下面這樣做:

```java
public class Impl implements InterfaceA, InterfaceB {
  public void defaultMethod(){
    // existing code here..
    InterfaceA.super.defaultMethod();
  }
}
```
## 默認方法與常規方法的不同

與常規方法相比默認方法除了有　default 修飾符外，常規方法除了可以修改方法參數外還可以修改類屬性，但是默認方法只能訪問方法參數，因爲接口沒有狀態屬性。

　　總之，默認方法可以讓我們向已有接口添加方法而不破壞這些接口的實現類

　　當我們實現一個包含默認方法的接口時，遵循一下規則:

1. 如果沒有重寫默認方法將會繼承接口默認方法
2. 重寫默認方法和我們其他方法在子類中重寫相似
3. 重定義默認方法爲抽象方法，將會強迫子類重寫它

翻譯自:https://dzone.com/articles/interface-default-methods-java

