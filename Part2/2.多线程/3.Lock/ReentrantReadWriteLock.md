实现Lock接口

读锁 -> 共享锁

写锁 -> 排它锁

```java
static void read(Lock lock) {
    try {
        lock.lock();
        Thread.sleep(1000);
        System.out.println("read over");
    } catch (InterruptedException e) {
        e.printStackTrace();
    } finally {
        lock.unlock();
    }
}

static void write(Lock lock) {
    try {
        lock.lock();
        Thread.sleep(1000);
        System.out.println("write over");
    } catch (InterruptedException e) {
        e.printStackTrace();
    } finally {
        lock.unlock();
    }
}

public static void main(String[] args) {
    ReentrantLock reentrantLock = new ReentrantLock();
    ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
    Lock readLock = readWriteLock.readLock();
    Lock writeLock = readWriteLock.writeLock();
    for (int i = 0; i < 10; i++) new Thread( () -> read(readLock)).start();
    for (int i = 0; i < 2; i++) new Thread( () -> write(writeLock)).start();
}
```

​      

