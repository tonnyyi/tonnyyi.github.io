---
title: java 中资源文件的加载方式
tags:
  - java
  - resource
categories:
  - java
date: 2019-11-12 14:08:32
---

## 项目结构
```
│ 
├── src
│   └── main
│       ├── java
│       │   └── com
│       │       └── test
│       │           ├── ResourceTest.java
│       │           └── Resource.java
│       └── resources
│           ├── conf
│           │   └── config.json
│           └── application.properties
└── pom.xml
```

## 读取方式
```java
public class ResourceTest {

    public static void main(String[] args) {
        // 1、通过Class的getResource方法
        String a1 = ResourceTest.class.getResource("/com/test/Resource.class").getPath();
        String a2 = ResourceTest.class.getResource("Resource.class").getPath();
        String a3 = ResourceTest.class.getResource("/application.properties").getPath();
        String a4 = ResourceTest.class.getResource("../../application.properties").getPath();
        String a5 = ResourceTest.class.getResource("/conf/config.json").getPath();
        String a6 = ResourceTest.class.getResource("../../conf/config.json").getPath();

        // 2、通过本类的ClassLoader的getResource方法
        String b1 = ResourceTest.class.getClassLoader().getResource("com/test/Resource.class").getPath();
        String b2 = ResourceTest.class.getClassLoader().getResource("application.properties").getPath();
        String b3 = ResourceTest.class.getClassLoader().getResource("conf/config.json").getPath();

        // 3、通过ClassLoader的getSystemResource方法
        String c1 = ClassLoader.getSystemClassLoader().getResource("com/test/Resource.class").getPath();
        String c2 = ClassLoader.getSystemClassLoader().getResource("application.properties").getPath();
        String c3 = ClassLoader.getSystemClassLoader().getResource("conf/config.json").getPath();

        // 4、通过ClassLoader的getSystemResource方法
        String d1 = ClassLoader.getSystemResource("com/test/Resource.class").getPath();
        String d2 = ClassLoader.getSystemResource("application.properties").getPath();
        String d3 = ClassLoader.getSystemResource("conf/config.json").getPath();

        // 5、通过Thread方式
        String e1 = Thread.currentThread().getContextClassLoader().getResource("com/test/Resource.class").getPath();
        String e2 = Thread.currentThread().getContextClassLoader().getResource("application.properties").getPath();
        String e3 = Thread.currentThread().getContextClassLoader().getResource("conf/config.json").getPath();
    }
}
```
总结规律如下:
- `Class.getResource()`的资源获取如果以 `/` 开头, 则从根路径开始搜索资源
- `Class.getResource()`的资源获取如果不以 `/` 开头, 则从当前类所在的路径开始搜索资源
- `ClassLoader.getResource()`的资源获取**不能以 `/` 开头**, 统一从根路径开始搜索资源

## 源码分析

首先看一下`Class`类中的两个方法实现
```java
public InputStream getResourceAsStream(String name) {
    name = resolveName(name);
    ClassLoader cl = getClassLoader0();
    if (cl==null) {
        // A system class.
        return ClassLoader.getSystemResourceAsStream(name);
    }
    return cl.getResourceAsStream(name);
}

public java.net.URL getResource(String name) {
    name = resolveName(name);
    ClassLoader cl = getClassLoader0();
    if (cl==null) {
        // A system class.
        return ClassLoader.getSystemResource(name);
    }
    return cl.getResource(name);
}
```

可以看到, `Class`类先通过`resoveName`方法解析出资源文件路径, 然后委托`ClassLoader`去加载资源的, 首先看一下`resolveName`方法的实现
```java
private String resolveName(String name) {
    if (name == null) {
        return name;
    }
    if (!name.startsWith("/")) {
        Class<?> c = this;
        while (c.isArray()) {
            c = c.getComponentType();
        }
        String baseName = c.getName();
        int index = baseName.lastIndexOf('.');
        if (index != -1) {
            name = baseName.substring(0, index).replace('.', '/')
                +"/"+name;
        }
    } else {
        name = name.substring(1);
    }
    return name;
}
```

核心逻辑就是
- 如果资源路径入参以`/`开头则截取 `/` 后面的内容作为资源路径
- 否则通过截取当前类的全限定名称获取当前类所在的包, 然后将包分隔符`.`替换为`/`, 例: `com.test.ResourceTest` -> `com/test/`, 最后拼接上资源路径入参作为资源文件的路径

再看一下`ClassLoader`的两个方法实现

```java
public URL getResource(String name) {
    URL url;
    if (parent != null) {
        url = parent.getResource(name);
    } else {
        url = getBootstrapResource(name);
    }
    if (url == null) {
        url = findResource(name);
    }
    return url;
}

public InputStream getResourceAsStream(String name) {
    URL url = getResource(name);
    try {
        return url != null ? url.openStream() : null;
    } catch (IOException e) {
        return null;
    }
}
```

`getResourceAsStream`方法也是先调用同类的`getResource`方法, 所以重点看一下`getResource`方法的实现. 该方法首先使用了**双亲委派机制**来加载资源, 最终的父类`ClassLoader`使用`getBootstrapClassPath`去加载资源, 如果父类没加载到还会`findResource`去加载, 因此`ClassLoader`子类可以通过覆盖`findResource`方法来实现自定义的资源加载实现.

本示例中就是通过`AppClassLoader`的 `findResource`方法(实际是继承自`URLClassLoader`)加载到的资源, 该方法在`URLClassLoader`中实现如下
```java
public URL findResource(final String name) {
    /*
     * The same restriction to finding classes applies to resources
     */
    URL url = AccessController.doPrivileged(
        new PrivilegedAction<URL>() {
            public URL run() {
                return ucp.findResource(name, true);
            }
        }, acc);

    return url != null ? ucp.checkURL(url) : null;
}
```