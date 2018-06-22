---
title: Java源码 - Collection - HashMap
tags:
- HashMap
- JDK
categories:
- SourceCode
- JDK
---

> 源码基于JDK1.7

![hierarchy](hierarchy.png)

在Java集合类里, 很多类的声明里既继承一个父类又显示的实现父类已经实现的接口(比如HashMap继承了AbstractMap, 而AbstractMap是实现了Map接口的, 但HashMap的类声明还是显示的实现Map接口), 其实Spring也有类似的写法, 关于这一点stackOverflow上有人[这么解释](http://stackoverflow.com/a/3854763/1330573):

> It's a way to remember that this class really implements that interface.It won't have any bad effect and it can help to understand the code without going through the complete hierarchy of the given class.

意思是这样可以很清晰的看出这个类确实是实现了某个接口, 而且这么做没有任何副作用, 反而可以不用看完类的完整继承链就能理解代码.

### 成员变量

- `static final int DEFAULT_INITIAL_CAPACITY = 1 << 4`: 默认的容量, 即2^4=16
- `static final int MAXIMUM_CAPACITY = 1 << 30`: 最大容量, 2^30=1,073,741,824, 10亿多
- `static final float DEFAULT_LOAD_FACTOR = 0.75f`: 默认的加载因子
- `static final Entry<?,?>[] EMPTY_TABLE = {}`: 一个空数组, 用来被所有实例共享
- `transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE`: 用来保存元素的数组, 其长度必须是2的n次方.
- `transient int size`: Map内保存的元素个数
- `int threshold`: 直译为"门槛", 当`保存的元素个数 * loadFactor`达到该值时就需要rehash
- `final float loadFactor`: 加载因子
- `transient int modCount`: 保存Map更新的次数
- `static final int ALTERNATIVE_HASHING_THRESHOLD_DEFAULT = Integer.MAX_VALUE`: 
- `transient int hashSeed = 0`

### 构造函数

- `HashMap()`: 方法体`this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR)`
- `HashMap(int initialCapacity)`: 方法体`this(initialCapacity, DEFAULT_LOAD_FACTOR)`
- `HashMap(int initialCapacity, float loadFactor)`: 指定容量和加载因子

所以重点看看第三个构造方法实现:

```java
if (initialCapacity < 0)
    throw new IllegalArgumentException("Illegal initial capacity: " +
                                       initialCapacity);
if (initialCapacity > MAXIMUM_CAPACITY)
    initialCapacity = MAXIMUM_CAPACITY;
if (loadFactor <= 0 || Float.isNaN(loadFactor))
    throw new IllegalArgumentException("Illegal load factor: " +
                                       loadFactor);

this.loadFactor = loadFactor;
threshold = initialCapacity;
init();
```

主要是验证参数范围, 最后的`init()`方法在`HashMap`是空的方法体.





