## Lock接口

| 上锁 | 可中断上锁        | try上锁 | 解锁   |
| ---- | ----------------- | ------- | ------ |
| lock | lockInterruptibly | tryLock | unlock |







## ReentrantLock

实现Lock接口

#### 结构

基于AQS实现

有一个抽象静态内部类Sync，继承AQS

有两个静态内部类FairSync和NonfairSync，继承Sync，重写AQS的tryAcquire()方法，实现公平锁与非公平锁



#### 公平锁原理

每个线程调用tryAcquire()获取同步状态时

会先调用hasQueuedPredecessors()方法判断当前同步队列中有没有比自己先进入等待状态的线程

有他只能进入等待队列排队，直到被唤醒

没有就直接CAS设置state变量



#### 非公平锁原理

非公平锁的tryAcquire()方法获取同步状态时，少了公平锁hasQueuedPredecessors()的判断

Q：那么非公平锁不就和AQS同步队列FIFO性质冲突了吗？

A：有以下非公平情景：

> t1线程释放锁，唤醒队头t2线程，调用tryAcquire()尝试获取同步状态
>
> 这时来了t3线程，也调用tryAcquire()尝试获取同步状态
>
> 注意，此时t2和t3调用的是同一段代码，即NonfairSync重写AQS的tryAcquire()方法
>
> 就会导致两个线程同时尝试修改同步状态
>
> 若t2还没成功获取锁时，t3成功把锁抢了，t2只能接着在队列等待，就导致了不公平
>
> 所以这里的不公平针对的是同步队列中被唤醒的线程与未加入同步队列第一次tryAcquire()的线程
>
> 若t3抢锁失败，还是会到同步队列中排队，所以对于同步队列内部的线程还是公平的

ReentrantLock(boolean fair)带布尔值的构造函数可用来指定是否用公平锁

*公平锁为了保证公平需要进行大量的线程切换 `不是很理解`，非公平锁会造成线程饥饿，但却保证了更高的吞吐量* 



#### 锁重入原理

ReentrantLock的FairSync和NonfairSync重写AQS的tryAcquire()方法中有这段逻辑

```java
else if (current == getExclusiveOwnerThread()) {
    int nextc = c + acquires;
    if (nextc < 0)
        throw new Error("Maximum lock count exceeded");
    setState(nextc);
    return true;
}
```

current == getExclusiveOwnerThread()  判断独占锁是否被当前线程持有

是就把state变量加1，同一个线程多次获得锁不用阻塞等待，即可重入



#### tryLock原理

- tryLock() -- 尝试锁，锁失败**不会入队尾**等待，直接返回false

  不指定等待时间，直接调用NonfairSync的nonfairTryAcquire()方法，非公平操作

- tryLock(long timeout, TimeUnit unit) -- 指定等待时间，尝试锁，锁失败**入队尾**带超时等待

  调用抽象静态内部类Sync的tryAcquireNanos()方法

  接下来在整体上就是走了AQS的acquire()逻辑（看上面流程图），只不过多了等待时间判断

  在tryAcquire()失败加入同步队列之前，会根据当前时间 + 指定的等待时间，算出等待超时时间

  加入同步队列之后，就进入死循环操作

  若前一个节点不是头结点，会判断当前是否超出等待时间，是就返回加锁失败，否就调用LockSupport.parkNanos()方法阻塞，这里会指定最大阻塞时间

  若前一个节点是头结点，死循环自旋tryAcquire()操作，也会判断当前是否超出等待时间

  带超时判断获取同步状态代码：

  ```java
  if (nanosTimeout <= 0L)
      return false;
  final long deadline = System.nanoTime() + nanosTimeout;
  final Node node = addWaiter(Node.EXCLUSIVE);
  try {
      for (;;) {
          final Node p = node.predecessor();
          if (p == head && tryAcquire(arg)) {
              setHead(node);
              p.next = null; // help GC
              return true;
          }
          nanosTimeout = deadline - System.nanoTime();
          if (nanosTimeout <= 0L) {
              cancelAcquire(node);
              return false;
          }
          if (shouldParkAfterFailedAcquire(p, node) &&
              nanosTimeout > SPIN_FOR_TIMEOUT_THRESHOLD)
              LockSupport.parkNanos(this, nanosTimeout);
          if (Thread.interrupted())
              throw new InterruptedException();
      }
  } catch (Throwable t) {
      cancelAcquire(node);
      throw t;
  }
  ```

  

#### lockInterruptibly原理

调用AQS的acquireInterruptibly()方法

接下来在整体上就是走了AQS的acquire()逻辑（看上面流程图）

tryAcquire()失败就加入同步队列，就进入死循环操作

当前一个节点不是头结点，阻塞被唤醒之后，若判断到当前线程被中断，直接抛出中断异常（即响应中断）

带响应中断的获取同步状态代码：

```java
for (;;) {
    final Node p = node.predecessor();
    if (p == head && tryAcquire(arg)) {
        setHead(node);
        p.next = null; // help GC
        return;
    }
    if (shouldParkAfterFailedAcquire(p, node) &&
        parkAndCheckInterrupt())
        throw new InterruptedException();  //抛中断异常，响应中断
}
```

