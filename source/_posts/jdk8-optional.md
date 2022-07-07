---
title: java8-optional
tags:
  - jdk
categories:
  - jdk
  - source
date: 2019-03-29 09:52:53
---

# 问题
在调用一个方法后需要链式的获取某个字段, 比如:

```java
// 调用接口, 获取用户名
ResponseDTO dto = xxxServcice.foo();
String username = dto.getUser().getUserName();
```
但是上面的方法可能会抛出空指针异常, 因此, 为了程序的健壮性, 我们一般在每一步的`get`前都需要进行非空判断:

```java
// 调用接口, 获取用户名
ResponseDTO dto = xxxServcice.foo();
if (dto != null) {
    User user = dto.getUser();
    if (user != null){
        String username = user.getUserName();
    }
}
```
如果字段获取链路过深, 那么这些非空判断非常多, 很烦. 下面介绍下 如何使用 jdk8 中的`Optional`类解决这个问题.

# Optional
## API
首先看一下`Optional`都有哪些方法

### 构造类方法
有三个静态 `empty()`, `ofNullAble()` 和 `of()` 可以创建`Optional`对象
```java
// 显示的创建一个包含 Null 值的 Optional 对象
Optional<String> empty = Optional.empty();

String username = null;
// 创建一个值可以为空的 Optional 对象
Optional<String> nullable = Optional.ofNullable(username);

// 创建一个值不能为空的 Optional 对象, 值为空时会抛出空指针异常
Optional<String> of = Optional.of(username);
```

### 判断类方法
就一个实例方法`isPresent()`用来判断值是否为`null`
```java
Optional<String> optional = Optional.of(username);
// 如果值不为 null(不推荐这种用法, 与原来的非空判断写法没有区别)
if (optional.isPresent()) {
    // 获取值
    username = optional.get();
} else {
    throw new RuntimeException("username cannot null")
}
```

### 获取值类方法
一共有4个方法, `get()`, `orElse(T)`, `orElseGet(Supplier)` 和 `orElseThrow(Supplier)`, 对应着当值为`null`时的不同处理策略
```java
// 值为 null 时返回默认值
username = optional.orElse("张三");

// 值为 null 时调用Supplier接口实现来获取值
username = optional.orElseGet(() -> {
    return yyService.get();
});

// 值为 null 时直接抛出异常, 该异常对象由Supplier接口实现创建
username = optional.orElseThrow(() -> {
    return new IllegalArgumentException("username cannot be null");
});

// 如果值为 null 抛出默认的NoSuchElementException异常
username = of.get();
```

### 其他

```java
// 当值不为 null 时消费该值, 接受一个 Consumer 接口实现
optional.ifPresent(username -> {
    // 
});

// 对值进行过滤, 通过则返回当前 Optional 对象, 否则调用 empty() 返回空值对象
optional = optional.filter(StringUtils::isNotBlank);

// 该方法会将入参Function接口实现返回值包装成 Optional 对象, 从而实现链式操作
Optional<String> s1 = of.map(s -> {
    return s + "12";
}).map(String::toUpperCase);

// 类似 map, 但入参Function接口实现的返回值类型是 Optional 对象了
// 且要求该对象不能为 null
Optional<String> s1 = of.flatMap(s -> {
    return Optional.of(s + "12");
}).map(String::toUpperCase);
```

## 最佳实践
回到上面的问题, 看看如何使用`Optional`解决联系空值判断的问题.

```java
String username = Optional.ofNullable(dto)
        .map(ResponseDTO::getUser)
        .map(User::getUsername)
        .orElseThrow(() -> {
            return new RuntimeException("username cannot be null");
        });

// do something with username
```

1. **尽量避免在程序中直接调用`Optional`对象的`get()`和`isPresent()`方法**
2. **避免使用`Optional`类型声明实体类的属性**, `Optional`是没有继承`Serializable`接口的, 所以一旦用了, 该属性就没法被序列化的
3. **Optional 主要用作返回类型以构建流畅的API**, **不要使用`Optional`作为方法的入参**, 这样做会让代码变得复杂. 

## JDK9增强
上面介绍的 API 都是基于 JDK8 的, JDK 9 为 `Optional` 类添加了三个方法: `or()`,`ifPresentOrElse()` 和 `stream()`.

1. `or()` 方法与 `orElse()` 和 `orElseGet()` 类似, 它们都在对象为空的时候提供了替代情况. 

    ```java
     User result = Optional.ofNullable(user)
          .or(() -> Optional.of(new User("default","1234")))
          .get();
    ```

2. `ifPresentOrElse()` 方法需要两个参数: 一个 `Consumer` 和一个 `Runnable`. 如果对象包含值, 会执行 `Consumer` 的动作, 否则运行 Runnable.

    ```java
    Optional.ofNullable(user)
        .ifPresentOrElse(
            u -> logger.info("User is:" + u.getEmail()), 
            () -> logger.info("User not found")
        );
    ```

3. `stream()` 方法把`Optional`实例转换为 `Stream` 对象, 从而可以使用大量的`Stream`接口
   
    ```java
    List<String> emails = Optional.ofNullable(user)
      .stream()
      .filter(u -> u.getEmail() != null && u.getEmail().contains("@"))
      .map(u -> u.getEmail())
      .collect(Collectors.toList());
    ```
