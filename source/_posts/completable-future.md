---
title: completable-future
tags:
  - java
  - concurrent
categories:
  - java
date: 2021-12-15 15:52:12
---





### 创建CompletableFuture对象

`CompletableFuture.completedFuture`是一个静态辅助方法，用来返回一个已经计算好结果的`CompletableFuture`(结果就是入参)。

```java
public static <U> CompletableFuture<U> completedFuture(U value)
```

下面4个静态方法可以为一段异步代码创建`CompletableFuture`对象

```java
public static CompletableFuture<Void> 	runAsync(Runnable runnable)
public static CompletableFuture<Void> 	runAsync(Runnable runnable, Executor executor)
public static <U> CompletableFuture<U> 	supplyAsync(Supplier<U> supplier)
public static <U> CompletableFuture<U> 	supplyAsync(Supplier<U> supplier, Executor executor)
```

以`Async`结尾并且没有指定`Executor`的方法会使用`ForkJoinPool.commonPool()`作为它的线程池执行异步代码

> `ForkJoinPool.commonPool()`是当前JVM进程上的所有 CompletableFuture、并行 Stream 共享的，commonPool的目标场景是非阻塞的CPU密集型任务，其线程数默认为CPU数量减1，所以对于我们用java常做的IO密集型任务，默认线程池是远远不够使用的；在双核及以下机器上，默认线程池又会退化为为每个任务创建一个线程，相当于没有线程池。
>
> 因此, **复杂的应用中推荐使用自定义线程池创建`CompletableFuture`对象**

> `ForkJoinPool.commonPool()`中使用的默认线程池中的线程不会继承父线程的ClassLoader, 有时候回出现找不到类的情况, 修复方法如下:
>
> ```java
> ClassLoader contextClassLoader = Thread.currentThread().getContextClassLoader();
> 
> CompletableFuture<Void> task1 = CompletableFuture.runAsync(() -> {
> Thread.currentThread().setContextClassLoader(contextClassLoader);
> });
> ```
>
> 或者干脆使用自定义线程池也可以解决



### 计算完成时的处理

`CompletableFuture`类实现了`CompletionStage`和`Future`接口，所以还是可以通过阻塞或者轮询的方式获得结果，但**这种方式不推荐使用**.

```java
public T 	get()
public T 	get(long timeout, TimeUnit unit)
public T 	getNow(T valueIfAbsent)
public T 	join()
```

当`CompletableFuture`的计算结果完成，或者抛出异常的时候，我们可以执行特定的`Action`。主要是下面的方法

```java
public CompletableFuture<T> 	whenComplete(BiConsumer<? super T,? super Throwable> action)
public CompletableFuture<T> 	whenCompleteAsync(BiConsumer<? super T,? super Throwable> action)
public CompletableFuture<T> 	whenCompleteAsync(BiConsumer<? super T,? super Throwable> action, Executor executor)
public CompletableFuture<T>     exceptionally(Function<Throwable,? extends T> fn)
```

可以看到`Action`的类型是`BiConsumer<? super T,? super Throwable>`，它可以处理正常的计算结果，或者异常情况。方法不以`Async`结尾，意味着`Action`使用相同的线程执行，而`Async`可能会使用其它的线程池的线程去执行。

`exceptionally`方法是当原始的CompletableFuture抛出异常的时候调用，并且返回一个新的CompletableFuture. 如果原始的CompletableFuture正常计算完后，这个新的CompletableFuture也计算完成，它的值和原始的CompletableFuture的计算的值相同。也就是这个`exceptionally`方法用来处理异常的情况。

下面一组方法计算完成或者抛出异常的时候，会触发, 返回CompletableFuture对象，结果由`BiFunction`参数计算而得。因此这组方法兼有`whenComplete`和转换的两个功能。

```java
public <U> CompletableFuture<U> 	handle(BiFunction<? super T,Throwable,? extends U> fn)
public <U> CompletableFuture<U> 	handleAsync(BiFunction<? super T,Throwable,? extends U> fn)
public <U> CompletableFuture<U> 	handleAsync(BiFunction<? super T,Throwable,? extends U> fn, Executor executor)
```



### 转换

由于回调风格的实现，我们不必因为等待一个计算完成而阻塞着调用线程，而是告诉`CompletableFuture`当计算完成的时候请执行某个`function`。而且我们还可以将这些操作串联起来，或者将`CompletableFuture`组合起来。

```java
public <U> CompletableFuture<U> 	thenApply(Function<? super T,? extends U> fn)
public <U> CompletableFuture<U> 	thenApplyAsync(Function<? super T,? extends U> fn)
public <U> CompletableFuture<U> 	thenApplyAsync(Function<? super T,? extends U> fn, Executor executor)
```

这一组函数的功能是当原来的CompletableFuture计算完后，将结果传递给函数`fn`，将`fn`的结果作为新的`CompletableFuture`计算结果。因此它的功能相当于将`CompletableFuture<T>`转换成`CompletableFuture<U>`。



### 只消费结果

上面的方法是当计算完成的时候，会生成新的计算结果(`thenApply`, `handle`)，或者返回同样的计算结果`whenComplete`，`CompletableFuture`还提供了一种处理结果的方法，只对结果执行`Action`,而不返回新的计算值，因此计算值为`Void`:

```java
public CompletableFuture<Void> 	thenAccept(Consumer<? super T> action)
public CompletableFuture<Void> 	thenAcceptAsync(Consumer<? super T> action)
public CompletableFuture<Void> 	thenAcceptAsync(Consumer<? super T> action, Executor executor)
```

`thenAcceptBoth`以及相关方法提供了类似的功能，当两个CompletionStage都正常完成计算的时候，就会执行提供的`action`，它用来组合另外一个异步的结果。
`runAfterBoth`是当两个CompletionStage都正常完成计算的时候,执行一个Runnable，这个Runnable并不使用计算的结果。

```java
public <U> CompletableFuture<Void> 	thenAcceptBoth(CompletionStage<? extends U> other, BiConsumer<? super T,? super U> action)
public <U> CompletableFuture<Void> 	thenAcceptBothAsync(CompletionStage<? extends U> other, BiConsumer<? super T,? super U> action)
public <U> CompletableFuture<Void> 	thenAcceptBothAsync(CompletionStage<? extends U> other, BiConsumer<? super T,? super U> action, Executor executor)
public     CompletableFuture<Void> 	runAfterBoth(CompletionStage<?> other,  Runnable action)
```

下面一组方法当计算完成的时候会执行一个Runnable,与`thenAccept`不同，Runnable并不使用CompletableFuture计算的结果。

```java
public CompletableFuture<Void> 	thenRun(Runnable action)
public CompletableFuture<Void> 	thenRunAsync(Runnable action)
public CompletableFuture<Void> 	thenRunAsync(Runnable action, Executor executor)
```



### 组合

这一组方法接受一个Function作为参数，这个Function的输入是当前的CompletableFuture的计算值，返回结果将是一个新的CompletableFuture，这个新的CompletableFuture会组合原来的CompletableFuture和函数返回的CompletableFuture。因此它的功能类似:

```java
public <U> CompletableFuture<U> 	thenCompose(Function<? super T,? extends CompletionStage<U>> fn)
public <U> CompletableFuture<U> 	thenComposeAsync(Function<? super T,? extends CompletionStage<U>> fn)
public <U> CompletableFuture<U> 	thenComposeAsync(Function<? super T,? extends CompletionStage<U>> fn, Executor executor)
```

`thenCompose`返回的对象并不一是函数`fn`返回的对象，如果原来的`CompletableFuture`还没有计算出来，它就会生成一个新的组合后的CompletableFuture。

而下面的一组方法`thenCombine`用来复合另外一个CompletionStage的结果。它的功能类似：

```java
public <U,V> CompletableFuture<V> 	thenCombine(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn)
public <U,V> CompletableFuture<V> 	thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn)
public <U,V> CompletableFuture<V> 	thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn, Executor executor)
```

两个CompletionStage是并行执行的，它们之间并没有先后依赖顺序，`other`并不会等待先前的`CompletableFuture`执行完毕后再执行。其实从功能上来讲,它们的功能更类似`thenAcceptBoth`，只不过`thenAcceptBoth`是纯消费，它的函数参数没有返回值，而`thenCombine`的函数参数`fn`有返回值。



### 任一

`thenAcceptBoth`和`runAfterBoth`是当两个CompletableFuture都计算完成，而我们下面要了解的方法是当任意一个CompletableFuture计算完成的时候就会执行。

```java
public CompletableFuture<Void> 	acceptEither(CompletionStage<? extends T> other, Consumer<? super T> action)
public CompletableFuture<Void> 	acceptEitherAsync(CompletionStage<? extends T> other, Consumer<? super T> action)
public CompletableFuture<Void> 	acceptEitherAsync(CompletionStage<? extends T> other, Consumer<? super T> action, Executor executor)
public <U> CompletableFuture<U> 	applyToEither(CompletionStage<? extends T> other, Function<? super T,U> fn)
public <U> CompletableFuture<U> 	applyToEitherAsync(CompletionStage<? extends T> other, Function<? super T,U> fn)
public <U> CompletableFuture<U> 	applyToEitherAsync(CompletionStage<? extends T> other, Function<? super T,U> fn, Executor executor)
```

`acceptEither`方法是当任意一个CompletionStage完成的时候，`action`这个消费者就会被执行。这个方法返回`CompletableFuture<Void>。

`applyToEither`方法是当任意一个CompletionStage完成的时候，`fn`都会被执行，它的返回值会当作新的`CompletableFuture<U>`的计算结果。



### 辅助方法

下面介绍的这两个方法用来组合多个CompletableFuture。

```java
public static CompletableFuture<Void> 	    allOf(CompletableFuture<?>... cfs)
public static CompletableFuture<Object> 	anyOf(CompletableFuture<?>... cfs)
```

`allOf`方法是当所有的`CompletableFuture`都执行完后执行计算。

`anyOf`方法是当任意一个`CompletableFuture`执行完后就会执行计算，计算的结果相同。



参考:

[Java CompletableFuture 详解](https://colobu.com/2016/02/29/Java-CompletableFuture/)
