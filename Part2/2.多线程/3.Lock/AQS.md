学这块要看源码

## 定义

AbstractQueuedSynchronizer  抽象队列同步器，基于模板模式的**锁框架**，供各种子类去实现







## 方法

|      | 获取同步状态  | 可中断获取同步状态         | try获取同步状态            | 释放          | try释放                    |
| ---- | ------------- | -------------------------- | -------------------------- | ------------- | -------------------------- |
| 独占 | acquire       | acquireInterruptibly       | tryAcquire(模板方法)       | release       | tryRelease(模板方法)       |
| 共享 | acquireShared | acquireSharedInterruptibly | tryAcquireShared(模板方法) | releaseShared | tryReleaseShared(模板方法) |







## 组成

- state变量

  > volatile类型，初始值为0，具体作用和用法由子类实现去定义
  >
  > 例如ReentrantLock实现：0解锁状态，每获取一次锁state +1（**可重入**）
  >
  > CountDownLatch实现：设置state初始值为x，每次 -1直到0

- 等待队列头尾节点指针

  >双向链表，每个node保存的是线程
  >双向是因为加入队列添加节点时需要考虑前面节点的状态，如果前面节点正在持有线程，新节点就排在他后面，若前面节点被取消了就应该越过前面的节点
  >
  >`还不是很理解双向的原因`

- Node节点

  - waitStatus -- 等待状态
  - prev -- 前驱节点指针
  - next -- 后继节点指针
  - thread -- 加锁线程变量

- VarHandle（变量句柄）

  > 对于某块地址空间的值，可绕过指向他的变量直接指向这块地址空间，可对里面的值进行原子操作，即对普通属性的原子操作
  >
  > 直接操作二进制码，效率比反射高
  >
  > `深入了解VarHandle和反射`







## acquire

即独占锁获取同步状态流程

> !tryAcquire() -- 尝试获取同步状态，即获取state变量
>
> 若同步状态获取失败，调用addWaiter()方法构造Node，以CAS的形式插到同步队列队尾
>
> 再调用acquireQueued()死循环判断前驱节点是否队头元素，是就继续tryAcquire()，获取同步状态
>
> 若前驱节点非队头元素或获取同步状态失败，则调用LockSupport.park()阻塞当前线程
>
> 直到前驱节点出队将其唤醒，或阻塞线程被中断将其唤醒 `中断唤醒这里不理解`

<img src="D:\Learning\Gunners-Java\Part2\2.多线程\pic\AQS框架原理.jpg" style="zoom:80%; float:left" />

当锁被某个线程持有，修改state值，修改state是CAS操作，加锁线程变量值改成自己

当锁被释放，再修改state值，同时唤醒队头的线程

当线程尝试CAS修改state获取锁时，若获取不了，会把自己挂起，进入等待队列

**进入等待队列的过程也是CAS过程**，防止一个节点后面挂好多个节点，导致线程不安全。入队前先保存tail节点，执行CAS操作时判断当前tail节点跟之前保存的tail节点是否一致

在无竞争的状态下，队头head不会被初始化







## tryAcquire

独占锁获取同步状态，模板方法，AQS的子类有不同的实现







## release

独占锁释放同步状态，LockSupport.unpark(s.thread)唤醒队头的下一个线程







## 自定义同步组件

实现Lock类，声明静态内部类继承AQS

锁面向的是使用用户，而同步器面向的则是线程控制，那么在锁的实现中聚合同步器而不是直接继承AQS就可以很好的隔离二者所关注的事情