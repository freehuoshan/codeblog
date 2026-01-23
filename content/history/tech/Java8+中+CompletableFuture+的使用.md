---
title: Java8 中 CompletableFuture 的使用
id: 8
date: 2023-04-10 18:19:13
auther: FREEDOM
cover: https://codeblog.net/upload/2021/04/download-fa75ce62b4c440138c94877d52d9a384.jpeg
excerpt: 介绍这篇文章介绍 Java8 中为改善　Concurrency API （并发 API）引入的类　CompletableFuture 的使用Java 中的异步计算异步计算不好理解。通常我们希望所有计算都按照步骤顺序执行，线性思维。但是异步计算中回调动作分散在代码各处而且互相深度嵌套，如果再考虑到错误
permalink: /archives/java8zhong-completablefuturede-shi-yong
categories:
 - code
tags: 
 - java8
---

1. ## 介绍

这篇文章介绍 Java8 中为改善　Concurrency API （并发 API）引入的类　CompletableFuture 的使用

2. ## Java 中的异步计算

异步计算不好理解。通常我们希望所有计算都按照步骤顺序执行，线性思维。但是异步计算中回调动作分散在代码各处而且互相深度嵌套，如果再考虑到错误处理，会更难以理解。

接口　Future 在　Java5 中被引入作为异步计算的结果，但是它没有为计算的组合提供任何方法，也没有为可能发生的错误提供处理解决方案。

　Java8 引入了 Completable，除了 Future 接口，它也实现了　CompletionStage 接口。该接口为异步计算中的步骤指定了规范，使得这些步骤可以与其他步骤组合。

　CompleteableFuture 同时也是一个由五十多个不同方法组成的构件块和框架，这些方法用于异步计算步骤的编排，组合，以及错误处理等。

　如此庞大的　API 会让人不是所措，但是大多数方法用于我们常用的异步计算情况，通过几个不同的用例你就会明白。

3. ## 将 CompletableFuture 作为一个简单　Future 使用

首先，CompletableFuture 类实现了 Future 接口，所以你可以将其作为　Future 的实现使用，但是需要有异步任务完成的逻辑代码。

　例如，你可以使用 CompletableFuture 的无参构造器创建一个实例以表示异步计算将来完成后的结果，在将来某个时间点当异步计算完成后使用　complete 方法将其结果包裹后发给消费者。然后消费者使用其 get 方法阻塞当前线程直到异步计算完成并返回结果。

  在下边的例子中，我们有一个方法，首先创建了一个 CompletableFuture 实例，然后将一些计算任务拆分到另一个的线程中，接下来立马返回了刚才创建的　Future 实例。

  在拆分的计算中，我们等待了 500 毫秒以模拟耗时任务，然后将　“hello” 字符串作为计算结果传给刚才创建的实例，以标识该 Future 事件已经完成并返回了结果。

```java
public　Future<String> calculateAsync() throws　InterruptedException {
    CompletableFuture<String> completableFuture  = new　CompletableFuture<>();
 
    Executors.newCachedThreadPool().submit(() -> {
        Thread.sleep(500);
        completableFuture.complete("Hello");
        return　null;
    });
 
    return　completableFuture;
}
```
上边拆分计算我们使用了 Executor API 创建新的线程，我们完全可以使用最原始的穿件线程的方式，例如:

```java
new Thread(() -> {
  Thread.sleep(500)
  coompletableFuture.complete(Hello)
}).start()  
```
作用与上边 4-8　行的代码一致。或者也可以使用其他创建线程的 API。

  当我们调用上边的方法，会获得一个 Future 实例，然后可以使用 get 方法获得结果，该方法会阻塞当前线程直到返回值（如果当代码执行到此处，异步计算已经完成可以立马获得值，否则会等待异步计算返回值）。

  同时我们注意到 get 返回抛出了几个受检异常，ExecutionException (当执行计算遇到问题，会抛出该异常)，InterruptedException(当执行线程被中断，会抛出该异常)

```java
Future<String> completableFuture = calculateAsync();
 
// ... 
 
String result = completableFuture.get();
assertEquals("Hello", result);
```

如果已经知道了一个计算的结果，可以将该结果作为参数传入 completedFuture 方法，然后调用　get 方法不会被阻塞，会将该值作为结果立马返回。

```java
Future<String> completableFuture = CompletableFuture.completedFuture("Hello");
 
// ...
 
String result = completableFuture.get();
assertEquals("Hello", result);
```
某些情况下，你有可能想要取消一个异步任务。

　例如，有可能你没有找到一个值，接下去的计算没有任何意义，你希望到此结束该异步计算。这种情况下对于　Future 而言可以使用它的 cancel 方法，该方法接受一个布尔参数，但是它对　CompletableFuture 却没有影响，CompletableFuture 没有使用中断控制它的处理流程。

　 下面是修改后的异步方法:

```java
public Future<String> calculateAsyncWithCancellation() throws InterruptedException {
    CompletableFuture<String> completableFuture = new CompletableFuture<>();
 
    Executors.newCachedThreadPool().submit(() -> {
        Thread.sleep(500);
        completableFuture.cancel(false);
        return null;
    });
 
    return completableFuture;
}
```
当我们使用 get　方法获得结果时，它会抛出　CancellationException 

```java
Future<String> future = calculateAsyncWithCancellation();
future.get(); // CancellationException
```

4. ## CompletableFuture 封装计算逻辑

使用类似上边的代码我们可以实现任何并行操作，但是如果我们不想要总是写这种冗余模板代码，让异步代码更加简单。

  CompletableFutue 提供的静态方法 runAsync 和　supplyAsync 允许我们通过　Runnalbe 和　Supplier 函数类型创建实例。

  Runnable 和 Supplier　都是函数接口(只包含一个方法的接口),所以可以以 lambda 的方式传递他们的实例。

　Runnable 接口用于创建线程，它不会返回值。

　Supplier 接口会返回值。

　当使用 lambda 作为 Supplier 实例传入 supplyAsync ，如下例:
```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "Hello");
 
// ...
 
assertEquals("Hello", future.get());
```

5. ## 处理异步计算的结果

处理结果通常的方式是将其传递给一个方法。方法 thenApply 就是: 接受一个函数接口实例，该实例会处理结果并返回一个包裹处理结果的 Future

```java
CompletableFuture<String> completableFuture
  = CompletableFuture.supplyAsync(() -> "Hello");
 
CompletableFuture<String> future = completableFuture
 .thenApply(s -> s + " World");
 
assertEquals("Hello World", future.get());
```
如果你不需要为后面的 Future 链返回值，你可以使用方法 thenAccept 方法，该方法以　Comsumer 函数接口类型为参数，返回 void ，这时如果再调用 get 方法会返回一个 Void 类型实例。

```java
CompletableFuture<String> completableFuture
  = CompletableFuture.supplyAsync(() -> "Hello");
 
CompletableFuture<Void> future = completableFuture
  .thenAccept(s -> System.out.println("Computation returned: " + s));
 
future.get();
```
最后如果你既不需要计算的结果也不需要返回结果，你可以传递一个 Runnable 的 Lambda 实例给 thenRun 方法。例如下面代码:

```java
CompletableFuture<String> completableFuture 
  = CompletableFuture.supplyAsync(() -> "Hello");
 
CompletableFuture<Void> future = completableFuture
  .thenRun(() -> System.out.println("Computation finished."));
 
future.get();
```
6. ## 组合 Futures

CompletableFuture API 最重要的功能就是可以将一系列计算步骤中的 CompletableFuture 实例链接起来。

  这条计算链的结果依然是一个 CompletableFuture ,可以进一步链接组合。这种方式在函数编程语言中几乎无处不在，也被称为 Monadic 设计模式。

  下面是一个我们使用 thenCompose 方法将俩那个 Futures 按照顺序链接起来的例子。

  该方法需要一个函数接口作为参数，返回一个 CompletableFuture 实例。这个函数接口类型的参数是上一步计算的结果。在下一步计算中我们可以使用基于该值进一步计算:

```java
CompletableFuture<String> completableFuture 
  = CompletableFuture.supplyAsync(() -> "Hello")
    .thenCompose(s -> CompletableFuture.supplyAsync(() -> s + " World"));
 
assertEquals("Hello World", completableFuture.get());
```
方法 thenCompose 和 thenApply 是实现 Monadic 模式的基础构件。与 Stream 和 Optional 类中的 map 、flatMap 方法非常相似。

  这些方法都是接受一个函数接口实例作为参数用于处理计算结果，但是 thenCompose(flatMap) 接收的函数参数会返回另一个同类型的对象（嵌套 future/Stream）,这样的功能结构使得我们可以灵活将这些类型以非常灵活的方式组合。

  如果你希望独立执行两个 Futures 事件然后基于它们的值作某些计算，可以使用 thenCombine 方法，该方法接收两个参数，一个 Future（第二个独立 Future 事件的结果） ,和一个函数-该函数有两个参数分别是上一步计算(第一个独立 Future 事件)的结果和第一个参数的结果。

```java
CompletableFuture<String> completableFuture 
  = CompletableFuture.supplyAsync(() -> "Hello")
    .thenCombine(CompletableFuture.supplyAsync(
      () -> " World"), (s1, s2) -> s1 + s2));
 
assertEquals("Hello World", completableFuture.get());
```
如果你不需要将最终的结果传递给后边的 Future 链，可以使用 thenAcceptBoth 方法:

```java
CompletableFuture future = CompletableFuture.supplyAsync(() -> "Hello")
  .thenAcceptBoth(CompletableFuture.supplyAsync(() -> " World"),
    (s1, s2) -> System.out.println(s1 + s2));
```

7. ## thenApply() 和 thenCompose() 的不同

我们上边已经介绍了 thenApply() 和 thenCompose() 都是用于将不同的 CompletableFuture调用链接起来，但是这两个方法的功能不同

### thenApply():

该方法用于处理上一步调用的结果。但是有一点需要注意，如果 lambda 返回的是 Future，会形成嵌套。

```java
CompletableFuture<Integer> finalResult = compute().thenApply(s-> s + 1);
```
### thenCompose():

与 thenApply() 不同点在于，它会将包含的嵌套的 Future 扁平化，类似于 Stream API 中 map 和 flatMap() 的区别:
```java
CompletableFuture<Integer> computeAnother(Integer i){
    return CompletableFuture.supplyAsync(() -> 10 + i);
}
CompletableFuture<Integer> finalResult = compute().thenCompose(this::computeAnother);
```

8. ## 并行运行多个 Future

有时我希望能够以并行的方式运行多个 Future 事件，当所有事件执行完毕后基于结果再执行进一步的计算。

  CompleteableFuture 的静态方法就是为这种需求而提供的。

```java
CompletableFuture<String> future1  
  = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<String> future2  
  = CompletableFuture.supplyAsync(() -> "Beautiful");
CompletableFuture<String> future3  
  = CompletableFuture.supplyAsync(() -> "World");
 
CompletableFuture<Void> combinedFuture 
  = CompletableFuture.allOf(future1, future2, future3);
 
// ...
 
combinedFuture.get();
 
assertTrue(future1.isDone());
assertTrue(future2.isDone());
assertTrue(future3.isDone());
```

注意到该方法返回的是结果是 Void ，无法返回这些计算合并后的结果。

幸运的事，CompletableFuture 的 join() 方法配合 Stream API 让事情变得更简单:

```java
String combined = Stream.of(future1, future2, future3)
  .map(CompletableFuture::join)
  .collect(Collectors.joining(" "));
 
assertEquals("Hello Beautiful World", combined);
```

9. ## 处理错误

为了处理异步计算链的错误， CompletableFuture 提供了一个 handle 方法以替代传统同步计算中捕获异常的方式 。该方接受两个参数:一个是计算结果(如果计算成功)，另一个是抛出的异常(如果计算未能成功运行)。

下面的例子中，如果为提供 name 导致计算失败，在handle中返回一个默认值:

```java
String name = null;
 
// ...
 
CompletableFuture<String> completableFuture  
  =  CompletableFuture.supplyAsync(() -> {
      if(name == null) {
          throw new RuntimeException("Computation error!");
      }
      return "Hello, " + name;
 })}).handle((s, t) -> s != null ? s : "Hello, Stranger!");
 
assertEquals("Hello, Stranger!", completableFuture.get());
```
上边我们在 handle 中以提供默认值的方式处理了如果计算未能成功，我们也可以通过使用 completeExceptionally 方法直接抛出:

```java
String name = null;
 
// ...
 
CompletableFuture<String> completableFuture  
       =  CompletableFuture.supplyAsync(() -> {
            if(name == null) {
              throw new RuntimeException("Computation error!");
            }
           return "Hello, " + name;
})}).handle((s, t) -> completableFuture.completeExceptionally(t));   
 
assertEquals("Hello, Stranger!", completableFuture.get());
```
上边我们将异常直接抛出，那么当我们调用　get 方法时可以通过　try/catch 方式处理，或者再向上抛出。

10. ## 异步方法

CompletableFuture 类中的大多数方法都有后缀带有 Async 的两种变体。这些方法通常是为了将相应的计算运行在另一个线程中。

　不带有　Async 后缀的方法，其中的代码会运行在当前调用者同一个线程中。带有　Async 但是不带有 Executor 参数的方法会使用　ForkJoinPool.commonPool() 获取的 Executor 实现的普通 fork/join 线程池。带有　Async 和 Executor 参数的方法会使用传入的　Executor。

　 下面的代码与我们之前的使用方式唯一可以看到的区别是 thenApplyAsync ，但是其真正的区别发生在幕后，该方法中的代码被包裹到一个 ForkJoinTask 实例中。这使得我们可以并行计算，可以利用更多的计算机资源。

```java
CompletableFuture<String> completableFuture  
  = CompletableFuture.supplyAsync(() -> "Hello");
 
CompletableFuture<String> future = completableFuture
  .thenApplyAsync(s -> s + " World");
 
assertEquals("Hello World", future.get());
```
11. ## JDK9 中的 CompletableFuture API

  Java9 通过以下改变加强了 Completable API

- 添加了新的工厂方法
- 支持延时和超时
- 改进了对子类的支持

引入的新的 API

- Executor defaultExecutor()
- CompletableFuture<U> newIncompleteFuture()
- CompletableFuture<T> copy()
- CompletionStage<T> minimalCompletionStage()
- CompletableFuture<T> completeAsync(Supplier<? extends T> supplier, Executor executor)
- CompletableFuture<T> completeAsync(Supplier<? extends T> supplier)
- CompletableFuture<T> orTimeout(long timeout, TimeUnit unit)
- CompletableFuture<T> completeOnTimeout(T value, long timeout, TimeUnit unit)

添加了新的静态工具方法:

- Executor delayedExecutor(long delay, TimeUnit unit, Executor executor)
- Executor delayedExecutor(long delay, TimeUnit unit)
- <U> CompletionStage<U> completedStage(U value)
- <U> CompletionStage<U> failedStage(Throwable ex)
- <U> CompletableFuture<U> failedFuture(Throwable ex)

最後，為了解決超時問題，Java 9引入了另外兩個新功能：

- orTimeout()
- completeOnTimeout()

12. ## 总结

这篇文件介绍了 CompletableFuture 类中方法的通常用法。

翻译自: https://www.baeldung.com/java-completablefuture


