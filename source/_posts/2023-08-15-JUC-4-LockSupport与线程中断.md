---
title: JUC-4-LockSupport与线程中断
date: 2023-08-15 17:53:58
tags: 
  - Java
categories: 
  - Language
password: zzy   
message: 亲，能不能输入密码啊？
---

# 线程中断机制

## 中断机制

一个线程不应由其他线程来强制中断或停止，而应该由线程自己自行停止，因此`Thread.stop/suspend/resume`被废弃。Java提供了一种用于停止线程的协商机制--中断，它只是一种协作协商机制，Java没有给中断增加任何语法。
若要中断一个线程，你需要手动调用该线程的`interrupt`方法，该方法也仅仅是将线程对象的中断标识设成`true`；接着需要自己写代码检测当前线程的标识位，如果为`true`，表示别的线程请求这条线程中断。
每个线程对象中都有一个中断标识位，用于表示线程是否被中断，该标识位为`true`表示中断，为`false`表示未中断。该方法可以在别的线程中调用，也可以在自己的线程中调用。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-26.jpg)

## 中断API

| 方法                                  | 说明                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| `public void interrupt()`             | 实例方法，仅仅设置线程的中断状态为`true`，发起一个协商而不会立即停止一个线程 |
| `public static boolean interrupted()` | 静态方法，判断线程是否被中断并清除当前中断状态；1. 返回当前线程的中断状态，测试当前线程是否已被中断；2. 将当前线程的状态状态清零，并重新设为`flase`，清除线程的中断状态。如果连续两次调用此方法，则第二次调用将返回`false`，因为连续调用两次的结果可能不一样 |
| `public boolean isInterrupted()`      | 实例方法，判断当前线程是否被中断（检查中断标志位）           |

`public static boolean interrupted()`

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-28.jpg)

## 疑难解答

### 如何停止中断运行中的线程？

> 使用`volatile`变量实现

```java
public class VolatileDemo {

    private static volatile boolean isStop = false;

    public static void main(String[] args) {
        new Thread(() -> {
            while (true) {
                if (isStop) {
                    System.out.println(Thread.currentThread().getName() + "\t isStop 改为true， 程序停止");
                    break;
                }
                System.out.println("t1************** hello volatile");
            }
        }, "t1").start();

        try {
            TimeUnit.MILLISECONDS.sleep(20);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        new Thread(() -> {
            isStop = true;
        }, "t2").start();

    }
}

t1************** hello volatile
t1************** hello volatile
t1************** hello volatile
t1************** hello volatile
t1************** hello volatile
t1************** hello volatile
t1************** hello volatile
t1************** hello volatile
t1************** hello volatile
t1************** hello volatile
t1************** hello volatile
t1************** hello volatile
t1	 isStop 改为true， 程序停止
```

> 使用`AtomicBoolean`

```java
public class AtomicBooleanDemo {

    private static AtomicBoolean atomicBoolean = new AtomicBoolean(false);

    public static void main(String[] args) {
        new Thread(() -> {
            while (true) {
                if (atomicBoolean.get()) {
                    System.out.println(Thread.currentThread().getName() + "\t atomicBoolean 改为true， 程序停止");
                    break;
                }
                System.out.println("t1************** hello atomicBoolean");
            }
        }, "t1").start();

        try {
            TimeUnit.MILLISECONDS.sleep(20);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        new Thread(() -> {
            atomicBoolean.set(true);
        }, "t2").start();

    }
}
```

> 使用`Thread`类自带的中断api实例方法实现

```java
public class InterruptDemo {

    public static void main(String[] args) {

        Thread t1 = new Thread(() -> {
            while (true) {
                if (Thread.currentThread().isInterrupted()) {
                    System.out.println(Thread.currentThread().getName() + "\t isInterrupted 改为true， 程序停止");
                    break;
                }
                System.out.println("t1************** hello interrupt api");
            }
        }, "t1");
        t1.start();

        try {
            TimeUnit.MILLISECONDS.sleep(20);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        new Thread(t1::interrupt, "t2").start();
    }
}
```



在需要中断的线程中不断监听中断状态，一旦发生中断，就执行相应的中断处理业务逻辑stop线程

当对一个线程调用`interrupt()`时 

* 如果线程处于正常活动状态，那么会将该线程的中断标志设置为true，仅此而已。被设置中断标志的线程将继续正常运行，不受影响。所以，interrupt()并不能真正的中断线程，需要被调用的线程自己进行配合才行。
* 如果线程处于被阻塞状态（例如处于sleep, wait, join等状态），在别的线程中调用当前线程对象的interrupt方法，那么线程将立即退出被阻塞状态，并抛出一个InterruptedException异常。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-27.jpg)

### 当前线程的中断标识为`true`，是不是线程就立刻停止？

中断只是一种协商机制，修改中断标识位仅此而已，不是立刻stop打断

### 谈谈你对静态方法`Thread.interrupted`的理解？

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-29.jpg)

## 总结

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-30.jpg)

# LockSupport

LockSupport用于创建锁和其他同步类的基本线程阻塞原语，其方法`park()`是阻塞线程和`unpark()`是解除阻塞线程

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-31.jpg)

* LockSupport是用来创建锁和其他同步类的基本线程阻塞原语。
* LockSupport类使用了一种名为Permit(许可）的概念来做到阻塞和唤醒线程的功能，每个线程都有一个许可(permit)
* 但与 Semaphore不同的是，许可的累加上限是`1` 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-36.jpg)

permit许可证默认没有不能放行，所以一开始调`pak()`方法当前线程就会阻塞，直到别的线程给当前线程的发放permit，`park`方法才会被咳醒。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-37.jpg)

调用`unpark(thread)`方法后，就会将thread线程的许可证permit发放，会自动唤醒park线程，即之前阻塞中的`LockSupport.park()`方法会立即返回。

# 线程等待唤醒机制

## 线程等待唤醒方法

> 使用Object中的`wait()`方法使线程等待，使用Object中的`notify()`方法唤醒线程

**正常情况**

```java
public static void main(String[] args) {
    Object objectLock = new Object();

    new Thread(()->{
        synchronized (objectLock) {
            System.out.println(Thread.currentThread().getName() + "\t ********* 进来了");
            try {
                objectLock.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "\t ****** 被唤醒");
        }
    }, "t1").start();

    try {
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    new Thread(()->{
        synchronized (objectLock) {
            objectLock.notify();
            System.out.println(Thread.currentThread().getName() + "\t ********* 发出通知");
        }
    }, "t2").start();
}

t1	 ********* 进来了
t2	 ********* 发出通知
t1	 ****** 被唤醒
```

**异常情况1：wait()和notify()两个都去掉同步代码块**

```java
public static void main(String[] args) {
    Object objectLock = new Object();

    new Thread(()->{
//            synchronized (objectLock) {
            System.out.println(Thread.currentThread().getName() + "\t ********* 进来了");
            try {
                objectLock.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "\t ****** 被唤醒");
//            }
    }, "t1").start();

    try {
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    new Thread(()->{
//            synchronized (objectLock) {
            objectLock.notify();
            System.out.println(Thread.currentThread().getName() + "\t ********* 发出通知");
//            }
    }, "t2").start();
}
```

程序异常

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-32.jpg)



**异常情况2：将notify()放在wait()方法前**

程序无法执行，无法唤醒

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-33.jpg)

**总结**

* `wait()`和`notify()`必须在同步方法块或方法里面并且成对出现使用
* 使用顺序：先`wait()`然后`notify()`



> 使用JUC中Condition的`await()`方法使线程等待，使用`signal()`方法唤醒线程

**正常情况**

```java
public static void main(String[] args) {
    Lock lock = new ReentrantLock();
    Condition condition = lock.newCondition();

    new Thread(()-> {

        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t ********* 进来了");
            condition.await();
            System.out.println(Thread.currentThread().getName() + "\t ****** 被唤醒");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }, "t1").start();

    try {
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    new Thread(()->{
        try {
            lock.lock();
            condition.signal();
            System.out.println(Thread.currentThread().getName() + "\t ********* 发出通知");
        } finally {
            lock.unlock();
        }
    }, "t2").start();
}

t1	 ********* 进来了
t2	 ********* 发出通知
t1	 ****** 被唤醒
```

**异常情况1：去掉lock/unlock**

出现异常

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-34.jpg)

**异常情况2：将signal()放在await()方法前**

程序无法运行

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-35.jpg)



**结论**

* 在`lock()`和`unlock()`对里面才能正确调用condition中线程等待和唤醒的方法

* 使用顺序：先`await()`然后`signal()`

> 上述两个对象Object和Condition使用的限制

* 线程先要获得并持有锁，必须在锁块（synchronized或lock）中
* 必须要先等待后唤醒，线程才能够被唤醒
  心

> 使用LockSupport类可以阻塞当前线程以及唤醒指定被阻塞的线程

```java
public static void main(String[] args) {
    Thread t1 = new Thread(() -> {

        System.out.println(Thread.currentThread().getName() + "\t ********* 进来了");
        LockSupport.park();
        System.out.println(Thread.currentThread().getName() + "\t ****** 被唤醒");

    }, "t1");
    t1.start();

    try {
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    new Thread(() -> {
        LockSupport.unpark(t1);
        System.out.println(Thread.currentThread().getName() + "\t ********* 发出通知");
    }, "t2").start();
}
```

如果与上述一样先唤醒后等待（异常情况），LockSupport照样支持，但要牢记成双成对

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-38.jpg)

## 总结

* LockSupport是用来创建锁和其他同步类的基本线程阻塞原语
* LockSupport是一个线程阻塞工具类，所有的方法都是静态方法，可以让线程在任意位置阻塞，阻寨之后也有对应的唤醒方法。归根结底，LockSupport调用的Unsafe中的native代码
* LockSupport提供park()和unpark()方法实现阻塞线程和解除线程阻塞的过程
* LockSupport和每个使用它的线程都有一个许可(（permit）关联。
* 每个线程都有一个相关的permit, permit**最多只有一个**，重复调用unpark也不会积累凭证。

* 当调用park方法时
  * 如果有凭证，则会直接消耗掉这个凭证然后正常退出
  * 如果无凭证，就必须阻塞等待凭证可用 

* 而unpark则相反，它会增加一个凭证，但凭证最多只能有1个，累加无效 

> 为什么LockSupport可以突破wait/notify的原有调用顺序

因为unpark获得了一个凭证，之后再调用park方法，就可以名正言顺的凭证消费，故不会阻塞。先发放了凭证后续可以畅通无阻、

> 为什么唤醒两次后阻塞两次，但最终结果还会阻塞线程?

因为凭证的数量最多为1，连续调用两次unpark和调用一次unpark效果一样，只会增加一个凭证；而调用两次park却需要消费两个凭证，证不够，不能放行。