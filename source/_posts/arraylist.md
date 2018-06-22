---
title: Java源码 - Collection - ArrayList
tags:
  - ArrayList
  - JDK
categories:
  - SourceCode
  - JDK
date: 2016-08-26 14:45:50
---


> 源码基于JDK1.7

![hierarchy](hierarchy.png)

### 成员变量
- `private static final int DEFAULT_CAPACITY`: 默认容量, 值为10
- `private transient Object[] elementData`: 实际存储元素的数组
- `private static final Object[] EMPTY_ELEMENTDATA = {};`: 空数组, 默认使用该数组存储元素
- `private int size`: 保存的元素的个数, 并**不是数组的长度**, 加入移除元素, 该值需要减1, 而数组长度, 即`lenght`属性是不变的.
- `private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8`: 数组最大长度, 按注释说的, 减8是因为某些虚拟机会保留部分`header words`, 超过`Integer.MAX_VALUE - 8`可能会导致`OutOfMemoryError`.
<!-- more -->
### 构造函数
#### `ArrayList()` 
初始时使用的是空数组`EMPTY_ELEMENTDATA `, 但当添加元素时会对数组扩容, 扩容时使用默认容量`DEFAULT_CAPACITY`, 即10

```java
public ArrayList() {
    super();
    this.elementData = EMPTY_ELEMENTDATA;
}
```
- `super()`调用的是`AbstractList`的无参构造方法, 该方法体为空, 所以不用关心. 
- `this.elementData = EMPTY_ELEMENTDATA`用于初始化元素存储的数组, 因为后续添加元素时调用的是`Arrays.copyOf(T[] original, int newLength)`方法, 所以数组不能为null.


#### `ArrayList(int initialCapacity)`
指定初始容量

```java
public ArrayList(int initialCapacity) {
    super();
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Capacity: " + initialCapacity);
    this.elementData = new Object[initialCapacity];
}
```
没什么特殊的, 创建一个长度为*initialCapacity*的Object数组用于存储数据.

#### `ArrayList(Collection<? extends E> c)` 
基于已有的集合创建

```java
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    size = elementData.length;
    // c.toArray might (incorrectly) not return Object[] (see 6260652)
    if (elementData.getClass() != Object[].class)
        elementData = Arrays.copyOf(elementData, size, Object[].class);
}
```
注释部分的意思是`c.toArray()`返回的不一定是`Object`类型数组, 这是JDK的一个[Bug](http://bugs.java.com/bugdatabase/view_bug.do?bug_id=6260652), 这个问题[StatckOverFlow](http://stackoverflow.com/a/8521523/1330573)上有讨论. 下面这段代码演示了这个问题: 

```java
List<Object> l = new ArrayList<Object>(Arrays.asList("foo", "bar"));

// Arrays.asList("foo", "bar").toArray() produces String[], 
// and l is backed by that array

l.set(0, new Object()); // Causes ArrayStoreException, because you cannot put
                        // arbitrary Object into String[]           
```

### 常用方法
#### `add(E e)`

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```
主要看看`ensureCapacityInternal()`方法, 该方法要做的是为数组扩容, 传入的参数是数组需要的最小长度, 如果数组当前长度小于该值, 则需要扩容.

```java
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    ensureExplicitCapacity(minCapacity);
}
```
该方法主要保证第一次向使用`ArrayList()`构造的list中存放元素时数组容量够用. 接下来就看看`ensureExplicitCapacity()`方法的实现: 

```java
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
```
要注意的是`modCount`, 该属性是继承自`AbstractList`, 含义是当前list结构被更新次数, 主要用于防止迭代时list被修改, 文档在[这里](https://docs.oracle.com/javase/7/docs/api/java/util/AbstractList.html#modCount). 关于并发更新`ArrayList`, `Vector`等抛出`ConcurrentModificationException `异常的解释, 可以看看这篇[博客](http://www.cnblogs.com/dolphin0520/p/3933551.html). 
接着如果数组容量不足, 则调用`grow()`扩容.

```java
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```
从4 ~ 6行代码中可以看出, jdk会先尝试扩容到1.5倍, 如果还是不够则直接扩容到`minCapacity`. 如果扩容后的容量超过最大容量会调用`hugeCapacity()`确定最终的容量, 该方法主要的代码是`return (minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE : MAX_ARRAY_SIZE`. 
确定容量后就调用`Arrays.copyOf()`将数据拷贝到一个新的更大数组内, 数组的容量就是`newCapacity`. 该方法内调用的都是`native`方法来进行数组拷贝了.

#### `add(int index, E element)`
向指定位置插入数据

```java
public void add(int index, E element) {
    rangeCheckForAdd(index);

    ensureCapacityInternal(size + 1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}
```
相比上面的方法多了两行代码, 其中`rangeCheckForAdd(index)`很简单, 就是保证待插入的位置合法, 实际的逻辑就一行`index > size || index < 0`, 如果`true`就抛出异常. `System.arraycopy(elementData, index, elementData, index + 1, size - index)`是将下标`index`之后的元素向后挪一位, 从而把`index`下标的位置让出来.

#### `get(int index)`
获取指定位置的元素

```java
public E get(int index) {
    rangeCheck(index);

    return elementData(index);
}
```
`rangeCheck()`很简单, 如果`index >= size`就抛异常, 即获取的元素的下标不能超过保存的元素的个数. 接着`elementData()`返回数组指定下标的元素, 具体实现就一行`return (E) elementData[index];`.

#### `remove(int index)`
移除指定位置的元素

```java
public E remove(int index) {
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}
```
`rangeCheck`上面已经解释过了, `modCount++`增加了list结构更新的次数, 接着就是将`index+1`之后的数据以拷贝覆盖的方式前移一位, 最后将末尾数据清除. 当然如果`size - index - 1 > 0`, 即移除的是最后一位元素, 则直接将其赋值为`null`就行了. 最后返回被移除的元素.

#### `remove(Object o)`
移除指定元素

```java
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}
```
没什么特殊的, 就是遍历数组, 调用`fastRemove()`移除元素. 该方法的实现与`remove(int index)`基本一致, 只去除了`rangeCheck(index);`, 不检查下标范围, 还有就是没有`return`.

### 迭代
调用`iterator()`获取迭代器, 方法内部返回一个`Itr`实例, 这是一个内部类, 完整实现: 

```java
private class Itr implements Iterator<E> {
    int cursor;       // index of next element to return
    int lastRet = -1; // index of last element returned; -1 if no such
    int expectedModCount = modCount;

    public boolean hasNext() {
        return cursor != size;
    }

    @SuppressWarnings("unchecked")
    public E next() {
        checkForComodification();
        int i = cursor;
        if (i >= size)
            throw new NoSuchElementException();
        Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length)
            throw new ConcurrentModificationException();
        cursor = i + 1;
        return (E) elementData[lastRet = i];
    }

    public void remove() {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();

        try {
            ArrayList.this.remove(lastRet);
            cursor = lastRet;
            lastRet = -1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```
先看看每个成员变量的含义: 

- `cursor`: 游标, 指向当前元素的下标
- `lastRet`: 最后返回的元素的下标
- `expectedModCount`: 保存期望的`modCount`, 防止并发修改

`next()`用于获取元素, 并将游标向后移动一位.
`remove()`用于移除当前元素, 之后设置游标`cursor`为当前移除的元素的下标, 其实就是前移了一位. 将`lastRet`置为`-1`, 还要更新`expectedModCount`为新的`modCount`, 防止抛出`ConcurrentModificationException`.
`ArrayList.this.elementData`中的`this`用于在内部类访问外部类的实例. [StatckOverFlow](http://stackoverflow.com/a/5666168/1330573)上有相关的讨论.

**如果迭代时移除下标小于`cursor`的元素, 则被移除元素之后的所有元素都向前挪了一位, 这样等于把`cursor`向后跳过一位, 造成一个数据没被读取被直接跳过.**

### 总结
- `ArrayList`使用数组保存元素, 所以`get(int index)`, 即随机访问的速度很快, 但是插入, 移除元素时需要*搬迁*移动数组内的元素, 所以性能较差.
