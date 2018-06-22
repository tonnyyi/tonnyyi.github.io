---
title: AQS
tags:
  - Java
  - Concurrent
  - AQS
categories:
  - Java
  - Concurrent
date: 2018-05-21 16:06:20
---

总体流程: 
ReentrantLock.lock -> FairSync.lock() -> AbstractQueuedSynchronizer.acquire() -> FairSync.tryAcquire() 如果失败-> AQS.acquireQueued(AQS.addWaiter()) + thread.interrupt()

```java
// AbstractQueuedSynchronizer
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```
<!-- more -->
### 1. 尝试获取锁`FairSync.tryAcquire()`
```java 
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```
首先通过`AQS.getState()`获取同步状态:
#### 1.1 如果为0则表示当前锁对象未被持有
通过`hasQueuedPredecessors()`判断是否有其他在排队的线程, 如果: 
- 已经有(头节点后面跟着其他线程), 公平锁则返回false, 因为先到先得嘛
- 如果没有则尝试CAS设置锁状态从0设置为1, 失败时返回false
- 设置`AbstractOwnableSynchronizer`的`exclusiveOwnerThread`排他锁持有线程为当前线程, 返回true

#### 1.2 如果所以被持有, 并且是当前线程持有
设置锁状态值+1, 表示锁已被获取的次数+1, 返回true

#### 1.3 锁被其他线程持有则返回false, 获取锁失败

### 2. 获取锁成功, 则方法结束

### 3. 锁获取失败则将当前线程加入等待队列
#### 3.1 首先接当前线程放入队列尾部
```java
// AbstractQueuedSynchronizer
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    // Try the fast path of enq; backup to full enq on failure
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    enq(node);
    return node;
}
```
先基于`tail`属性直接设置, 如果队列为空就会失败, 此时会执行`enq()`方法确保入队列成功.
```java
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) { // Must initialize
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```
注意一点, 如果队列为空, 会执行`compareAndSetHead(new Node())`, 在队列头部放一个空节点, 然后再将当前线程放入队列尾部. 如果队列不为空则通过CAS方式, 利用`unsafe.compareAndSwapObject()`方法直接修改`tail`变量内存地址里保存的引用.
思考: 在将节点放入队列时, 执行步骤是: 1.设置节点的`prev`属性指向队列尾部元素; 2.CAS修改队列的`tail`属性为当前节点; 3.修改原队列尾部元素的`next`属性为当前节点. 为什么是这个顺序? 为什么不CAS成功后, 再一起设置相关`prev`和`next`属性?

#### 3.2 尝试让该线程重新获取锁
因为在将节点放入队列的过程中, 其他线程可能已经执行完成, 所以AQS会先进行自旋操作, 尝试让当前线程重新获取锁. 如果没有获取到锁, 还需要判断是否应该挂起, 判断的依据是它的前驱节点的waitStatus来确定的.
```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```


1. 尝试获取, 分三种情况:
    1.1 锁未被持有
    1.2 锁被当前线程持有
    1.3 锁被其他线程持有, 返回false
2. 获取成功, 方法结束
3. 获取失败, 做如下操作: 
    3.1 添加排他节点到队列尾部
    3.2 阻塞当前线程
    3.3 中断当前线程
    
针对1.1, 如果此时有其他排队的线程的话, 公平锁则返回false, 因为FIFO, 肯定是让排队的线程先获取; 如果是非公平锁 或者 没有其他排队的线程, 则调用`compareAndSetState`

3.1首先尝试将节点放入队尾, 失败后无限次重试. 