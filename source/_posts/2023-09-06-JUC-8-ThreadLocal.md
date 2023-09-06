---
title: JUC-8-ThreadLocal
date: 2023-09-06 13:59:36
tags: 
  - Java
categories: 
  - Language
password: zzy   
message: 亲，能不能输入密码啊？
---



# ThreadLocal 介绍

> WHAT

ThreadLocal提供线程局部变量，这些变量与正常变量不同，每一个线程在访问ThreadLocal实例时都有自己的、独立初始化的变量副本。ThreadLocal实例通常是类中的私有静态字段，使用它的目的是希望将状态（例如，用户ID或事务ID）与线程关联起来。

> WHY

每一个线程都有自己专属的本地变量副本，主要解决让每个线程绑定自己的值，通过使用get()和set()方法获取默认值或将其值更改为当前线程所存的副本的值从而避免了线程安全问题

> Methods

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-85.jpg)

> 栗子

```java
class House {

    int saleCount = 0;

    public synchronized void saleHouse() {
        ++saleCount;
    }

    ThreadLocal<Integer> saleVolume = ThreadLocal.withInitial(() -> 0);

    public void saleVolumeByThreadLocal() {
        saleVolume.set(1 + saleVolume.get());
    }

}

public class ThreadLocalDemo {

    public static void main(String[] args) {

        House house = new House();

        for (int i = 0; i < 5; i++) {

            new Thread(() -> {
                int size = new Random().nextInt(5) + 1;

                for (int j = 0; j < size; j++) {
                    house.saleHouse();
                    house.saleVolumeByThreadLocal();
                }
                System.out.println(Thread.currentThread().getName() + "\t sold " + house.saleVolume.get());
            }, String.valueOf(i)).start();
        }

        try {
            TimeUnit.MILLISECONDS.sleep(300);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println(Thread.currentThread().getName() + "\t total sold " + house.saleCount);
    }
}

3	 sold 1
0	 sold 5
4	 sold 1
2	 sold 5
1	 sold 4
main	 total sold 16
```

**main优化**

```java
public static void main(String[] args) {

    House house = new House();

    for (int i = 0; i < 5; i++) {

        new Thread(() -> {
            int size = new Random().nextInt(5) + 1;

            try {
                for (int j = 0; j < size; j++) {
                    house.saleHouse();
                    house.saleVolumeByThreadLocal();
                }
                System.out.println(Thread.currentThread().getName() + "\t sold " + house.saleVolume.get());
            } finally {
                house.saleVolume.remove();
            }
        }, String.valueOf(i)).start();
    }

    try {
        TimeUnit.MILLISECONDS.sleep(300);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    System.out.println(Thread.currentThread().getName() + "\t total sold " + house.saleCount);
}
```



> 阿里巴巴规范

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-86.jpg)

```java
class MyData {

    ThreadLocal<Integer> threadLocalField = ThreadLocal.withInitial(() -> 0);

    public void add() {
        threadLocalField.set(1 + threadLocalField.get());
    }
}

public class ThreadLocalDemo2 {

    public static void main(String[] args) {
        MyData myData = new MyData();

        ExecutorService threadPool = Executors.newFixedThreadPool(3);

        try {
            for (int i = 0; i < 10; i++) {
                threadPool.submit(() -> {
                    try {
                        Integer beforeInt = myData.threadLocalField.get();
                        myData.add();
                        Integer afterInt = myData.threadLocalField.get();
                        System.out.println(Thread.currentThread().getName() + "beforeInt: " + beforeInt + ", afterInt: " + afterInt);
                    } finally {
                        myData.threadLocalField.remove();
                    }
                });
            }

        } catch (Exception e) {

        } finally {
            threadPool.shutdown();
        }
    }
}
```

* 每个Thread内有自己的实例副本且该副本只由当前线程自己使用
* 既然其它Thread不可访问，那就不存在多线程间共享的问题
* 统一设置初始值，但是每个线程对这个值的修改都是各自线程互相独立的

# 源码分析

> Thread 和 ThreadLocal

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-87.jpg)

> ThreadLocal 和 ThreadLocalMap

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-88.jpg)![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-89.jpg)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-90.jpg)

当为ThreadLocal变量赋值，实际上是以当前threadLocal实例为key，值为value的Entry往该threadLocalMap中存放

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-91.jpg)

JVM内部维护了一个线程版的`Map<ThreadLocal,Value>`，每个线程要用到时，用当前的线程去Map里面获取，通过这样让每个线程都拥有了自己独立的变量，在并发模式下是绝对安全的变量。

# 内存泄漏问题

> 内存泄漏

不再使用的对象或者对象占用的内存无法回收

> ThreadLocalMap

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-92.jpg)

ThreadLocalMap是一个保存ThreadLoSal对象的map：

* 第一层包装是使用`WeakReference<ThreadLocak<?>>`将ThreadLocal对象变成一个弱引用的对象
* 第二层包装是定义Entry来扩展`WeakReference<ThreadLocal<?>>`



![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-93.jpg)

> 强引用

当内存不足，JVM开始垃圾回收，对于强引用的对象，就算是出现OOM也不会对该对象进行回收

* 在Java中最常见的就是强引用，把一个对象赋给一个引用变量，这个引用变量就是一个强引用
* 当一个对象被强引用变量引用时，它处于可达状态，它是不可能被垃圾回收机制回收的，即使该对象以后永远都不会被用到，JVM也不会回收
* 因此强引用是造成Java内存泄漏的主要原因之一

> 软引用

* 用`java.lang.ref.SoftReference`类来实现

* 对于只有软引用的对象来说，当系统内存充足时它不会被回收，当系统内存不足时它被回收。

> 弱引用

* 用`java.lang.ref.WeakReference`类来实现，它比软引用的生存期更短
* 对于只有弱引用的对象来说，只要垃圾回收机制一运行，不管JVM的内存空间是否足够，都会回收该对象占用的内存

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-94.jpg)

> 虚引用

* 虚引用必须和引用队列`ReferenceQueue`联合使用
  用`java.lang.ref.PhantomReference`类来实现，虚引用并不会决定对象的生命周期。如果对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收，它不能单独使用也不能通过它访问对象，虚引用必须和引用队列联合使用
* PhantomReference的get方法总是返回null
  虚引用的主要作用是跟踪对象被垃圾回收的状态。仅是提供确保对象被finalize后做某些事情的通知机制。PhantomReference的get方法总是返回null，因此无法访问对应的引用对象
* 处理监控通知使用
  设置虚引用关联对象的唯一目的是在这个对象被收集器回收时收到一个系统通知或者后续添加进一步的处理，用来实现比finalize机制更灵活的回收操作

> 小结

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-95.jpg)

> 关系

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-96.jpg)

ThreadLocal本身并不存储值，它只是作为`key`来让线程从`ThreadLocalMap`获取`value`，每个Thread对象维护着一个`ThreadLocalMap`的引用，ThreadLocalMap是ThreadLocal的内部类，用Entry来进行存储

* 调用ThreadLocal的set()方法时实际是往ThreadLocalMap设置值，key是ThreadLocal对象，值Value是传递进来的对象
* 调用ThreadLocal的get()方法时实际是往ThreadLocalMap获取值，key是ThreadLocal对象

> 为什么源码使用弱引用？

当方法执行完后栈帧销毁，强引用`tl`也就消失，但此时线程的ThreadLocalMap里某个entry的`key`引用还指向该对象

* 若此`key`引用是强引用则会导致`key`指向的ThreadLocal对象及`v`指向的对象不能被gc回收，造成内存泄漏
* 若此`key`引用是弱引用则大概率会减少内存泄漏的问题，使用弱引用可以使ThreadLocal对象在方法执行完后顺利被回收且Entry的`key`引用指向为null

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-97.jpg)

ThreadLocalMap使用ThreadLocal的弱引用作为`key`，如果ThreadLocal没有外部强引用来引用他，那么系统gc时该ThreadLocal势必会被回收，ThreadLocalMap中就会出现`key`为nul的`Entry`，我们没有办法访问`key`为null的`Entry`的`value`，如果当前线程再迟迟不结束，这些`key`为null的`Entry`的`value`就会一直存在一条强引用链。

虽然弱引用保证`key`指向的ThreadLocal对象能被及时回收，但是`v`指向的value对象是需要ThreadLocalMap只有调用get、set并且key为`null`时才回收整个`Entry`、`value`，因此弱引用不能100%保证内存不泄露。

在不使用ThreadLocal对象后需要手动调用`remove()`方法删除它，尤其是线程池中的线程重复使用意味线程的ThreadLocalMap对象也是重复使用的，如果不手动调用`remove()`方法，那么后面的线程就有可能获取到上个线程遗留下来的value值，造成bug。

> 建议

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-98.jpg)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-86.jpg)

# 总结

* ThreadLocal并不解决线程间共享数据的问题
* ThreadLocal适用于变量在线程间隔离且在方法间共享的场景
* ThreadLocal通过隐式的在不同线程内创建独立实例副本避免实例线程安全的问题
* 每个线程持有只属于自己的专属Map并维护ThreadLocal对象与具体实例的映射，该Map由于只被持有它的线程访问，故不存在线程安全以及锁的问题
* ThreadLocalMap的Entry对ThreadLocal的引用为弱引用，避免ThreadLocal对象无法被回收的问题
* 都会通过expungeStaleEntry，cleanSomeSlots，replaceStaleEntry三方法回收键为`null`的Entry对象的值以及Entry对象本身从而防止内存泄漏，属于安全加固的方法

