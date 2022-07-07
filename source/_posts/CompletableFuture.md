---
title: CompletableFuture
tags:
  - Java
  - CompletableFuture
categories:
  - Java
  - concurrent
date: 2022-04-02 15:08:26
---

`Future`是Java5中增加的, 用来表示一个异步执行的结果. 其`isDone`方法可以检查执行是否完成, `get`方法可以**阻塞**主线程, 直至执行完成返回结果. 也开始使用`cancel`方法终止任务. 

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
  ExecutorService pool = Executors.newFixedThreadPool(10);
  Future<Integer> future = pool.submit(() -> {
    // 长时间的异步计算...
    return 100;
  });
  
  while (!future.isDone()) {
    Integer result = future.get();
  }
}
```



虽然提供了异步执行的能力, 但对于获取结果, 只能使用轮询或阻塞的方式. 在Java8中, 新增了`CompletableFuture`, 提供了函数式编程和接口回调的能力, 功能强大, 简化了异步编程.



## 创建CompletableFuture对象

```java
# 静态方法, 返回一个已经计算好的CompletableFuture
public static <U> CompletableFuture<U> completedFuture(U value);

# 为一段异步执行的代码 创建CompletableFuture对象
public static CompletableFuture<Void> runAsync(Runnable runnable)
public static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor)
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier)
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor)
```

> 以`Async`结尾的方法, 默认使用`ForkJoinPool.commonPool()`作为线程池执行异步代码, 当然也可以使用自定义的线程池

- `runAsync`方法以`Runnable`为入参, 因此`CompletableFuture`的计算结果为空
- `supplyAsync`方法以`Supplier<U>`为入参, 计算结果类型为`U`

```java
CompletableFuture<Integer> f = CompletableFuture.supplyAsync(() -> {
  return 100;
});
Integer result = f.get();
```



## 计算完成后的处理

```java
public CompletableFuture<T> whenComplete(BiConsumer<? super T,? super Throwable> action)
public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T,? super Throwable> action)
public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T,? super Throwable> action, Executor executor)
public CompletableFuture<T> exceptionally(Function<Throwable,? extends T> fn)
```

- `whenComplete**`方法是在计算结束后(包括异常结束)触发, 执行相关**消费**动作(动作没有返回值), 且返回新的`CompletableFuture`包含原始结果和异常
- `exceptionally`则是在计算抛出异常时触发, 返回新的`CompletableFuture`, 当原始对象计算异常时, 新的`CompletableFuture`的计算值为fn的结果; 如果原始对象正常完成, 则fn不被执行且新的`CompletableFuture`的计算结果和原始对象计算结果一致. 即:**有异常时执行fn, 否则结果和前次执行保持一致**

**这两类方法不是排他的**, 加入计算发生异常则两类方法都触发. 这几个方法都会返回`CompletableFuture`, 所以可以链式无限调用.

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> 1 / 0)
  .whenComplete((integer, throwable) -> {	// 返回新的CompletableFuture包含原始结果和异常
      System.out.println("whenComplete " + integer + ",e: " + throwable);
  })
  .exceptionally(throwable -> {
      System.out.println("exceptionally e:" + throwable);
      return -1;
  }).whenComplete((integer, throwable) -> {
      System.out.println("whenComplete 2");
  });
System.out.println(future.get());

// whenComplete null,e: java.util.concurrent.CompletionException: java.lang.ArithmeticException: / by zero
// exceptionally e:java.util.concurrent.CompletionException: java.lang.ArithmeticException: / by zero
// whenComplete 2
// -1

如果1/0改成 1/1, exceptionally入参fn不被执行且计算结果和前一次一致
// whenComplete 1,e: null
// whenComplete 2 1,e: null
// 1
```



下面这组`handle**`方法也返回`CompletableFuture`对象, 但是**计算值永远都是新的**, 不管上次计算是正常还是异常结束

```java
public <U> CompletableFuture<U> handle(BiFunction<? super T,Throwable,? extends U> fn)
public <U> CompletableFuture<U> handleAsync(BiFunction<? super T,Throwable,? extends U> fn)
public <U> CompletableFuture<U> handleAsync(BiFunction<? super T,Throwable,? extends U> fn, Executor executor)
```



## 转换

```java
public <U> CompletableFuture<U> thenApply(Function<? super T,? extends U> fn)
public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn)
public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn, Executor executor)
```

这组方法的功能是当原来的`CompletableFuture`计算完成转换, 将结果传递给`fn`, 将`fn`的结果作为新的`CompletableFuture`对象的技术结果, 因此相当于将`CompletableFuture<T>`转换成`CompletableFuture<U>`

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    return 100;
});
CompletableFuture<String> f =  future.thenApplyAsync(i -> i * 10).thenApply(i -> i.toString());
System.out.println(f.get()); //"1000" 如果feature执行异常, 则此处会抛异常
```

与`handle**`的区别是这组方法仅处理正常的计算值, 而`handle`会处理正常和异常计算



## 纯消费

```java
public CompletableFuture<Void> thenAccept(Consumer<? super T> action)
public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action)
public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action, Executor executor)
```

看参数就能明白, 由于入参是`Consumer`, 所以只有输入, 没有输出

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    return 100;
});
CompletableFuture<Void> f =  future.thenAccept(System.out::println);
System.out.println(f.get());
```



下面这组方法则是同时消费两个`CompletionStage`(当两个都正常完成时), `runAfterBoth`不使用任意一个计算结果

```java
public <U> CompletableFuture<Void> thenAcceptBoth(CompletionStage<? extends U> other, BiConsumer<? super T,? super U> action)
public <U> CompletableFuture<Void> thenAcceptBothAsync(CompletionStage<? extends U> other, BiConsumer<? super T,? super U> action)
public <U> CompletableFuture<Void> thenAcceptBothAsync(CompletionStage<? extends U> other, BiConsumer<? super T,? super U> action, Executor executor)
```

示例

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    return 100;
});
CompletableFuture<Void> f =  future.thenAcceptBoth(CompletableFuture.completedFuture(10), (x, y) -> System.out.println(x * y));
System.out.println(f.get());
```



下面这组方法在计算完成时会执行一个`Runnable`, 完全不使用计算结果

```java
public CompletableFuture<Void> thenRun(Runnable action)
public CompletableFuture<Void> thenRunAsync(Runnable action)
public CompletableFuture<Void> thenRunAsync(Runnable action, Executor executor)
public CompletableFuture<Void> runAfterBoth(CompletionStage<?> other,  Runnable action)
```

示例

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    return 100;
});
CompletableFuture<Void> f =  future.thenRun(() -> System.out.println("finished"));
System.out.println(f.get());
```



## 组合

```java
public <U> CompletableFuture<U> thenCompose(Function<? super T,? extends CompletionStage<U>> fn)
public <U> CompletableFuture<U> thenComposeAsync(Function<? super T,? extends CompletionStage<U>> fn)
public <U> CompletableFuture<U> thenComposeAsync(Function<? super T,? extends CompletionStage<U>> fn, Executor executor)
```

当原`CompletableFuture`正常完成时, 传递计算值并生成新的`CompletableFuture`, 实现类似 A --> B --> C 的串行执行效果

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
  return 100;
});
CompletableFuture<String> f = future.thenCompose(i -> {
  return CompletableFuture.supplyAsync(() -> {
    return (i * 10) + "";
  });
});
System.out.println(f.get()); //1000
```



下面这组方法则能够实现并行执行的效果, 最后对执行结果进行处理

```java
public <U,V> CompletableFuture<V> thenCombine(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn)
public <U,V> CompletableFuture<V> thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn)
public <U,V> CompletableFuture<V> thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn, Executor executor)
```

功能上`thenCombine`类似`thenAcceptBoth`, 但后者是纯消费, 它的函数没有返回值.

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    return 100;
});
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
    return "abc";
});
CompletableFuture<String> f =  future.thenCombine(future2, (x,y) -> y + "-" + x);
System.out.println(f.get()); //abc-100
```



## 任一结果

```java
public CompletableFuture<Void> acceptEither(CompletionStage<? extends T> other, Consumer<? super T> action)
public CompletableFuture<Void> acceptEitherAsync(CompletionStage<? extends T> other, Consumer<? super T> action)
public CompletableFuture<Void> acceptEitherAsync(CompletionStage<? extends T> other, Consumer<? super T> action, Executor executor)
  
public <U> CompletableFuture<U> applyToEither(CompletionStage<? extends T> other, Function<? super T,U> fn)
public <U> CompletableFuture<U> applyToEitherAsync(CompletionStage<? extends T> other, Function<? super T,U> fn)
public <U> CompletableFuture<U> applyToEitherAsync(CompletionStage<? extends T> other, Function<? super T,U> fn, Executor executor)
```

- `acceptEither`方法是当任意一个CompletionStage完成的时候，`action`就会被执行。这个方法返回`CompletableFuture<Void>`

- `applyToEither`方法是当任意一个CompletionStage完成的时候，`fn`会被执行，它的返回值会当作新的`CompletableFuture<U>`的计算结果。

```java
Random rand = new Random();
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    try {
        Thread.sleep(10000 + rand.nextInt(1000));
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return 100;
});
CompletableFuture<Integer> future2 = CompletableFuture.supplyAsync(() -> {
    try {
        Thread.sleep(10000 + rand.nextInt(1000));
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return 200;
});
// 返回值不一定, 可能是100或200, 看哪个future先执行完
CompletableFuture<String> f =  future.applyToEither(future2,i -> i.toString());
```



## 辅助方法

```java
public static CompletableFuture<Void> allOf(CompletableFuture<?>... cfs)
public static CompletableFuture<Object> anyOf(CompletableFuture<?>... cfs)
```

这两个方法用来组合更多个`CompletableFuture`

- `allOf`方法是当所有的`CompletableFuture`都执行完后执行计算。
- `anyOf`方法是当任意一个`CompletableFuture`执行完后就会执行计算，计算的结果相同。

`anyOf`和`applyToEither`有所不同不同.`anyOf`接受任意多的CompletableFuture, 但是`applyToEither`只是判断两个CompletableFuture, `anyOf`返回值的计算结果是参数中其中一个CompletableFuture的计算结果, `applyToEither`返回值的计算结果却是要经过`fn`处理的.

```java
Random rand = new Random();
CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(() -> {
    try {
        Thread.sleep(10000 + rand.nextInt(1000));
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return 100;
});
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
    try {
        Thread.sleep(10000 + rand.nextInt(1000));
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return "abc";
});
//CompletableFuture<Void> f =  CompletableFuture.allOf(future1,future2);
CompletableFuture<Object> f =  CompletableFuture.anyOf(future1,future2);
System.out.println(f.get());
```















参考: [Java CompletableFuture 详解](https://colobu.com/2016/02/29/Java-CompletableFuture/)
