---
title: JUC-12-StampedLock
date: 2023-09-06 14:02:44
tags: 
  - Java
categories: 
  - Language
---


# ReentrantReadWriteLock

读写锁定义：一个资源能够被多个读线程访问，或者被一个写线程访问，但是不能同时存在读写线程。

它只允许`读读`共存，而`读写`和`写写`依然互斥，大多实际场景`读读`线程间并不存在互斥关系，只有`读写`线程或`写写`线程间的操作需要互斥。

一个ReentrantReadWriteLock同时只能存在`一个写锁`但是可以存在`多个读锁`，但不能同时存在写锁和读锁，即一个资源可以被多个读操作访问―或一个写操作访问，但两者不能同时进行。

> 特点

* 可重入
* 读写兼顾

> 锁降级

ReentrantReadWriteLock锁降级：将写入锁降级为读锁，锁的严苛程度变强叫做升级，反之叫做降级。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-151.jpg)

* 如果同一个线程持有写锁，在没有释放写锁的情况下，它还可以继续获得读锁。这就是写锁的降级，降级成为读锁
* 将写锁降级为读锁：遵循获取写锁、获取读锁再释放写锁的次序，写锁能够降级为读锁
* 如果释放了写锁，那么就完全转换为读锁。

* 读锁升级到写锁是不可能的
* 如果有线程在读，写线程需要等待读线程释放锁后才能获取锁
* 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-152.jpg)

> 写锁和读锁互斥

写锁和读锁是互斥的（是指线程间的互斥，当前线程可以获取到写锁又获取到读锁，但是获取到了读锁不能继续获取写锁），这是因为读写锁要保持写操作的可见性，如果允许读锁在被获取的情况下对写锁的获取，那么正在运行的其他读线程无法感知到当前写线程的操作。

# StampedLock

StampedLock（邮戳锁）是Java8新增的一个读写锁，是对读写锁ReentrantReadWriteLock的优化

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-153.jpg)

stamp代表锁的状态。当stamp返回零时表示线程获取锁失败，并且当释放锁或者转换锁时都要传入最初获取的stamp值



> 锁饥饿

ReentrantReadWriteLock实现读写分离，但是一旦读操作比较多的时候，想要获取写锁就变得比较困难

* 使用“公平”策略一定程度缓解该问题

  ```java
  new ReentrantReadWriteLock(true);
  ```

* 使用“公平”策略是以牺牲系统吞吐量为代价的

> StampedLock

ReentrantReadWriteLock读锁被占用时其他线程尝试获取写锁的时候会被阻塞，但StampedLock采取乐观获取锁后，其他线程尝试获取写锁时不会被阻塞

* 所有获取锁的方法都返回一个邮戳（Stamp），Stamp为零表示获取失败，其余都表示成功
* 所有释放锁的方法都需要一个邮戳（Stamp），Stamp必须是和成功获取锁时得到的Stamp一致
* StampedLock是**不可重入**的，如果一个线程已经持有写锁，再去获取写锁的话就会造成死锁 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-154.jpg)

> StampedLock三种访问模式

1. Reading（读模式悲观）:功能和ReentrantReadWriteLock的读锁类似
2. Writing（写模式）︰功能和ReentrantReadWriteLbck的写锁类似
3. Optimistic reading（乐观读模式）：无锁机制，类似于数据库中的乐观锁，支持读写并发，很乐观认为读取时没人修改，假如被修改再实现升级为悲观读模式

> 栗子1（传统读写）

```java
public class StampedLocckDemo {

    static int number = 88;

    static StampedLock stampedLock = new StampedLock();

    public void write() {
        long stamp = stampedLock.writeLock();
        System.out.println(Thread.currentThread().getName() + " 写线程准备修改");

        try {
            number = number + 12;
        } finally {
            stampedLock.unlockWrite(stamp);
        }
        System.out.println(Thread.currentThread().getName() + " 写线程结束修改");
    }

    public void read() {

        long stamp = stampedLock.readLock();
        System.out.println(Thread.currentThread().getName() + " 进入读锁，4s后继续...");

        for (int i = 0; i < 4; i++) {
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " 正在读取中...");
        }

        try {

            int result = number;
            System.out.println(Thread.currentThread().getName() + " 获取成员变量result值：" + result);
            System.out.println(Thread.currentThread().getName() + " 写线程准备修改");
            System.out.println("写线程没有修改成功读锁时无法介入");
        } finally {
            stampedLock.unlockRead(stamp);
        }

    }

    public static void main(String[] args) {

        // 传统读写
        StampedLocckDemo resource = new StampedLocckDemo();

        new Thread(()->{
            resource.read();
        }, "readThread").start();

        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        new Thread(()->{
            System.out.println(Thread.currentThread().getName() + " 进入");
            resource.write();
        }, "writeThread").start();

        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println(Thread.currentThread().getName() + " 获取成员变量result值：" + number);
    }
}

readThread 进入读锁，4s后继续...
readThread 正在读取中...
writeThread 进入
readThread 正在读取中...
readThread 正在读取中...
readThread 正在读取中...
readThread 获取成员变量result值：88
readThread 写线程准备修改
写线程没有修改成功读锁时无法介入
writeThread 写线程准备修改
writeThread 写线程结束修改
main 获取成员变量result值：100
```

> 栗子2（乐观读）

**读过程也允许获取写锁介入**

```java
public void tryOptimisticRead() {
    long stamp = stampedLock.tryOptimisticRead();
    int result = number;

    System.out.println("4s前validate()值（true无修改false有修改）" + stampedLock.validate(stamp));
    for (int i = 0; i < 4; i++) {
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " 正在读取中..." + i + "s后validate()值（true无修改false有修改）" + stampedLock.validate(stamp));
    }
    if (!stampedLock.validate(stamp)) {
        System.out.println("有线程修改过了");
        stamp = stampedLock.readLock();

        try {
            System.out.println("从乐观读变更为悲观读...");
            result = number;
            System.out.println("重新进行悲观读后 result：" + result);
        } finally {
            stampedLock.unlockRead(stamp);
        }
        System.out.println(Thread.currentThread().getName() + " 最终结果为：result = " + result);
    }

}

public static void main(String[] args) {

    StampedLocckDemo resource = new StampedLocckDemo();

    new Thread(()->{
        resource.tryOptimisticRead();
    },"readThread").start();

    try {
        TimeUnit.SECONDS.sleep(2);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    new Thread(()->{
        System.out.println(Thread.currentThread().getName() + "来了...");
        resource.write();
    }, "writeThread").start();
}

4s前validate()值（true无修改false有修改）true
readThread 正在读取中...0s后validate()值（true无修改false有修改）true
writeThread来了...
writeThread 写线程准备修改
writeThread 写线程结束修改
readThread 正在读取中...1s后validate()值（true无修改false有修改）false
readThread 正在读取中...2s后validate()值（true无修改false有修改）false
readThread 正在读取中...3s后validate()值（true无修改false有修改）false
有线程修改过了
从乐观读变更为悲观读...
重新进行悲观读后 result：100
readThread 最终结果为：result = 100
```

> StampedLock 缺点

* StampedLock不支持重入
* StampedLock 的悲观读锁和写锁都不支持条件变量(Condition)
* 使用StampedLock**一定不要调用中断操作**，即不要调用`interrupt()`方法
