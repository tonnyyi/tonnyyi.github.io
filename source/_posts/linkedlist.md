---
title: Java源码 - Collection - LinkedList
tags:
  - LinkedList
  - JDK
categories:
  - SourceCode
  - JDK
date: 2016-09-12 12:08:19
---


> 源码基于JDK1.7

`LinkedList`不仅实现了`List`接口, 同时还实现了`Deque`接口, 所以还可以当双端队列去用.
<!-- more -->
![hierarchy](hierarchy.png)
### 成员变量

- `transient int size = 0`: 保存的元素的个数
- `transient Node<E> first; transient Node<E> last`: 分别指向头元素和尾元素的引用
### 构造函数

#### `LinkedList()`

方法体是空的, 略过

#### `LinkedList(Collection<? extends E> c)`

```java
public LinkedList(Collection<? extends E> c) {
	this();
	addAll(c);
}
```

重点看`allAll(c)`这行, 这个方法最终调用的是`addAll(int index, Collection<? extends E> c)`, 其中第一个参数传递的是`size`成员变量, 而`c`就是构造方法的入参. 这是该方法的实现: 

```java
// 参数index表示从列表的什么位置开始插入集合的元素
public boolean addAll(int index, Collection<? extends E> c) {
    checkPositionIndex(index);	// 第一段

  	// 第二段
    Object[] a = c.toArray();
    int numNew = a.length;
    if (numNew == 0)
        return false;

  	// 第三段
    Node<E> pred, succ;
    if (index == size) {
        succ = null;
        pred = last;
    } else {
        succ = node(index);
        pred = succ.prev;
    }

  	// 第四段
    for (Object o : a) {
        @SuppressWarnings("unchecked") E e = (E) o;
        Node<E> newNode = new Node<>(pred, e, null);
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        pred = newNode;
    }

  	// 第五段
    if (succ == null) {
        last = pred;
    } else {
        pred.next = succ;
        succ.prev = pred;
    }

    size += numNew;
    modCount++;
    return true;
}
```

第一行调用的方法是判断下标越界, 代码表示就是: 如果`!(index >= 0 && index <= size)`则抛出异常, 其中`index`是入参, `size`是成员变量. 

第三段代码用来确定待插入下下标上的节点`succ`和它指向前的节点`prev`, 插入新节点时需要修改`prev`的`next`指向到新元素. `if`判断是否是从尾部开始插入, 在`else`分支内有个`node(index)`方法, 该方法就是获取下标为`index`的元素. 该方法使用一点优化: 如果`index`在前半段则从头往后遍历, 否则从后尾向前遍历. 实现如下:

```java
Node<E> node(int index) {
    // assert isElementIndex(index);

    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}	
```

从这个方法可以看出, `LinkedList`的随机访问性能较差, 因为每次获取一个元素都需要遍历, 时间复杂度为`O(n)`.

第四段代码开始添加新节点, 首先构造一个新节点: 

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

如果待插入位置的前节点`prev`为`null`则将带插入集合内的第一个元素作为头结点, 否则将`prev`的先后链接指向新节点, 同时将新节点赋给`prev`, 这样不停遍历从而将集合内所有元素*串*起来.

第五段代码是完成双端链表内的指向, 如果是从列表尾部开始插入, 则列表的尾元素赋值为最后一个插入的新节点. 否则需要将最后的新节点的向后链接指向原先下标为`index`的元素, 又因为`LinkedList`实现了`Deque`接口, 可以看做是双端队列, 所以需要将原先下标为`index`的元素的向前链接指向最后一个插入的新节点. 至此插入完成. 

最后是更新列表的元素个数属性`size`和更新次数属性`modeCount`.

### 主要方法

#### List接口

##### `add(E e)`

即向列表尾部插入一个元素, 方法体很简单, 只是调用`linkLast(e)`之后返回`true`, 所以直接看`linkLast()`方法的实现:

```java
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```

也很简单, 构造一个新的节点, 如果当前列表内尾节点为`null`, 则将新节点作为列表的首节点, 否则将尾节点的先后链接指向新节点. 最后更新列表元素个数和更新次数属性.

##### `E get(int index)`

获取指定下标的元素, 方法体如下:

```java
public E get(int index) {
  	checkElementIndex(index);
  	return node(index).item;
}
```

第一行用于检测下标是否越界, 实际逻辑很简单: 如果`!(index >= 0 && index < size)`就抛出异常, 然后获取节点.

##### `E remove(int index)`

实现:

```java
public E remove(int index) {
  	checkElementIndex(index);
  	return unlink(node(index));
}
```

先检测下标是否越界, 然后获取节点后调用`unlinke()`, 该方法实现:

```java
E unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }

    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }

    x.item = null;
    size--;
    modCount++;
    return element;
}
```

如果待移除节点是头节点则将第二个节点赋值为头结点, 否则将待移除节点的前一个节点的后向链接指向下一个节点(说的有点绕口, 其实即使移除节点n时, 将n-1节点的next赋值为n+1节点). 如果移除的是尾节点, 将待移除节点的前一个节点赋值为尾节点, 否则将n+1节点的prev赋值为n-1节点. 至此整个链表完整地*串*起来了. 接着清除节点内保存的数据, 更新链表内节点的个数和链表更新次数. 

#### Queue接口

##### `E poll()`  

获取并移除头节点, 方法实现很简单:

```java
public E poll() {
  	final Node<E> f = first;
  	return (f == null) ? null : unlinkFirst(f);
}
```

##### `E peek()`

 获取但不移除头节点, 实现如下:

```java
public E peek() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}	
```

##### `boolean offer(E e)`

在队列尾部添加节点, 通过调用`add(E e)`实现的.

### 迭代

`LinkedList`的`iterator()`方法并无自己的实现, 即调用的是父类`AbstractSequentialList`的`iterator()`方法, 该方法在调用父类`AbstractList`的`listIterator()`接口最终返回一个`ListIterator`对象. 下面是方法实现

```java
public ListIterator<E> listIterator() {
    return listIterator(0);
}

public ListIterator<E> listIterator(final int index) {
  	rangeCheckForAdd(index);
  	return new ListItr(index);
}
```

其中`rangeCheckForAdd(int index``)`方法实现就一段如果`index < 0 || index > size()`则抛出异常, 即下标不能越界. 然后来看看`ListItr`的源码:

```java
private class ListItr extends Itr implements ListIterator<E> {
    ListItr(int index) {
        cursor = index;
    }

    public boolean hasPrevious() {
        return cursor != 0;
    }

    public E previous() {
        checkForComodification();
        try {
            int i = cursor - 1;
            E previous = get(i);
            lastRet = cursor = i;
            return previous;
        } catch (IndexOutOfBoundsException e) {
            checkForComodification();
            throw new NoSuchElementException();
        }
    }

    public int nextIndex() {
        return cursor;
    }

    public int previousIndex() {
        return cursor-1;
    }

    public void set(E e) {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();

        try {
            AbstractList.this.set(lastRet, e);
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    public void add(E e) {
        checkForComodification();

        try {
            int i = cursor;
            AbstractList.this.add(i, e);
            lastRet = -1;
            cursor = i + 1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }
}
```

比`Iterator`多了几个判断接口和`add`,`set`方法, 其中`add`用来在当前位置添加新元素, 而`set`则是更新当前下标位置为新的元素.


