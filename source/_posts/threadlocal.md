---
title: Java源码 - TheadLocal
date: 2018-02-28 16:43:26
tags:
	- ThreadLocal
	- JDK
categories:
	- SourceCode
	- JDK
---

> 源码基于JDK1.8

通常的用法:

```java
private static ThreadLocal<Map> localContext = new ThreadLocal<Map>();

public static void putLocalContext(Object key, Object value) {
    Map<Object, Object> context = localContext.get();
    if (context == null) {
        context = new HashMap<Object, Object>();
        localContext.set(context);
    }
    context.put(key, value);
}
```
<!-- more -->
### ThreadLocalMap
该类是`Thread`类的一个内部类, 且其定义标志为`static class ThreadLocalMap`. 通过控制`ThreadLocalMap`可见范围, 避免通过类似`t.getThreadLocals()`方法访问, 保证访问必须通过`ThreadLocal`, 进一步避免不合理操作导致的并发冲突.

`ThreadLocalMap`内部主要的属性有

```java
private Entry[] table;
private static final int INITIAL_CAPACITY = 16;

static class Entry extends WeakReference<ThreadLocal<?>> {
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```
其内部使用`Entry`数组保存数据, 该数据的默认容量是`INITIAL_CAPACITY`, 即16. 数据内的元素是继承了`WeakReference`的`Entry`对象. 
使用`WeakReference`是为了做到在程序中, 如果除了`ThreadLocalMap`引用了该`ThreadLocal`, 没有其他引用, 则回收该`ThreadLocal`. `ThreadLocal`内部通过`cleanSomeSlots()`并调用`expungeStaleEntry()`来释放对存入对象的引用, 同时将`table`内对应位置的元素置为`null`. 这样避免内存泄漏.
```java
// `ThreadLocal`不是static, 且线程被复用时, 每次都往`Thread`内放一个新的`ThreadLocal`, 如果不用`WeakReference`的话就会内存泄露
public void test() {
	ThreadLocal<Object> localContext = new ThreadLocal<Object>();
	// do something
}
```
### set()方法
```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```
直接看`map.set()`方法实现
```java
private void set(ThreadLocal<?> key, Object value) {
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);

    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();

        if (k == key) {
            e.value = value;
            return;
        }

        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }

    tab[i] = new Entry(key, value);
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
```
其中, `i`代表元素本来应该存放的位置下标, 从`i`开始查找, 如果有一个下标上没有元素("坑"没被占), 则将该元素存入此下标上, 此时元素实际存储位置不同于其原本应该存储的位置(即`i`), 其在其后面. 
存储完数据后会调用一次`cleanSomeSlots()`, 该方法是用来清理过期的元素, 如果没有元素被清理, 而且`table`内元素超过其容量的`2/3`则需要`rehash`.
```java
private boolean cleanSomeSlots(int i, int n) {
    boolean removed = false;
    Entry[] tab = table;
    int len = tab.length;
    do {
        i = nextIndex(i, len);
        Entry e = tab[i];
        if (e != null && e.get() == null) {
            n = len;
            removed = true;
            i = expungeStaleEntry(i);
        }
    } while ( (n >>>= 1) != 0);
    return removed;
}
```
`cleanSomeSlots()`方法会循环log2(n)次, 也就是从`i`开始遍历, 但不一定会遍历完所有元素, 
```java
private int expungeStaleEntry(int staleSlot) {
    Entry[] tab = table;
    int len = tab.length;

    tab[staleSlot].value = null;
    tab[staleSlot] = null;
    size--;

    Entry e;
    int i;
    for (i = nextIndex(staleSlot, len);
         (e = tab[i]) != null;
         i = nextIndex(i, len)) {
        ThreadLocal<?> k = e.get();
        if (k == null) {
            e.value = null;
            tab[i] = null;
            size--;
        } else {
            int h = k.threadLocalHashCode & (len - 1);
            if (h != i) {
                tab[i] = null;
                while (tab[h] != null)
                    h = nextIndex(h, len);
                tab[h] = e;
            }
        }
    }
    return i;
}
```
首先通过`tab[staleSlot].value = null;`释放对存入`ThreadLocal`的数据的引用, 然后一直向后遍历, 把所有的被回收的`ThreadLocal`都清理掉. 在else分支内所做的事, 是把之前由于开放寻址导致元素实际存储位置偏移的元素往前挪.

`set()`方法的`for`循环内, 首先如果一开始`i`上就没被占, 此时元素就存储在它本来应该在的位置, 即下标`i`上. 如果`k == null`, 也就是对应的`ThreadLocal`被回收了, 则执行`replaceStaleEntry()`方法. 这个方法好长啊, 不看了~~
下面看下`rehash`的实现
```java
private void rehash() {
    expungeStaleEntries();

    if (size >= threshold - threshold / 4)
        resize();
}
```
其中`expungeStaleEntries`是从0开始遍历数组, 如果数组元素`e != null && e.get() == null`, 即对应的`ThreadLocal`被回收, 则调用`expungeStaleEntry()`方法.
而`resize()`方法则是将数据容量翻倍, 然后元素导入.


### get()方法
```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```
直接看`getEntry()`的实现
```java
private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}
```
首先, 通过`int i = key.threadLocalHashCode & (table.length - 1);`获取待获取元素的下标, 然后获取元素. 
如果获取不到数据则调用`getEntryAfterMiss()`方法
```java
private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
    Entry[] tab = table;
    int len = tab.length;

    while (e != null) {
        ThreadLocal<?> k = e.get();
        if (k == key)
            return e;
        if (k == null)
            expungeStaleEntry(i);
        else
            i = nextIndex(i, len);
        e = tab[i];
    }
    return null;
}
```
该方法会从i开始一直查找到最后一个元素, 在遍历过程中如果`e.get() == null`, 即`ThreadLocal`被垃圾回收了, 则执行`expungeStaleEntry()`以清理相关数据.
