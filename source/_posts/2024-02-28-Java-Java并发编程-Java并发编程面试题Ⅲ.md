---
title: Java 并发编程常见面试题总结 Ⅲ
date: 2024-02-28 22:14:48
tags: 
  - Java
categories: 
  - Interview
password: zzy   
message: 仅管理员可见
---

#   ThreadLocal

> 场景分析

 通常情况下，我们创建的变量是可以被任何一个线程访问并修改的。**如果想实现每一个线程都有自己的专属本地变量该如何解决呢？**  

> 解决方案

 `ThreadLocal` 类 用于创建线程局部变量。线程局部变量是一种特殊类型的变量，每个线程都有自己的一份拷贝，互不干扰。 

* 如果创建  `ThreadLocal`  变量，那么访问这个变量的每个线程都会有这个变量的本地副本
* 可以使用 `get()` 和 `set()` 方法来获取值或将其值更改为当前线程所存的副本的值，从而避免线程安全问题 

##  使用 ThreadLocal

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-21.png)

```java
public class ThreadLocalExample {

    // 创建一个ThreadLocal对象
    private static ThreadLocal<String> threadLocal = new ThreadLocal<>();

    public static void main(String[] args) {
        // 在主线程设置线程局部变量的值
        threadLocal.set("Main Thread Value");

        // 创建并启动一个新线程
        Thread newThread = new Thread(() -> {
            // 在新线程中获取线程局部变量的值
            String value = threadLocal.get();
            System.out.println("New Thread Value: " + value);
        });

        newThread.start();
        
        // 在主线程获取线程局部变量的值
        String mainThreadValue = threadLocal.get();
        System.out.println("Main Thread Value: " + mainThreadValue);
    }
}
```

上面栗子中主线程设置了一个线程局部变量的值，然后创建了一个新线程，在新线程中获取了相同的线程局部变量的值。由于每个线程都有自己的拷贝，所以它们互不影响。

需要注意的是，在使用完`ThreadLocal`后，最好在不再需要的时候调用`remove`方法，以防止内存泄漏。

```java
// 清除线程局部变量
threadLocal.remove();
```

 `ThreadLocal`在多线程环境中经常用于保存线程相关的上下文信息，例如数据库连接、用户身份信息等。 

## ThreadLocal 原理

```JAVA
public class Thread implements Runnable {
    //......
    //与此线程有关的ThreadLocal值。由ThreadLocal类维护
    ThreadLocal.ThreadLocalMap threadLocals = null;

    //与此线程有关的InheritableThreadLocal值。由InheritableThreadLocal类维护
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
    //......
}
```

* `Thread` 类中有一个 `threadLocals` 和 一个 `inheritableThreadLocals` 变量，它们都是 `ThreadLocalMap` 类型的变量 

* 可以把 `ThreadLocalMap` 理解为`ThreadLocal` 类实现的定制化的 `HashMap`。 

* 默认情况下这两个变量都是 null，只有当前线程调用  `ThreadLocal` 类的 `set`或`get`方法时才创建，实际上调用这两个方法时，我们调用的是`ThreadLocalMap`类对应的 `get()`、`set()`方法。 

>  `ThreadLocal`类的`set()`方法 

```	java
public void set(T value) {
    // 获取当前请求的线程
    Thread t = Thread.currentThread();
    // 取出 Thread 类内部的 threadLocals 变量(哈希表结构)
    ThreadLocalMap map = getMap(t);
    if (map != null)
        // 将需要存储的值放入到这个哈希表中
        map.set(this, value);
    else
        createMap(t, value);
}
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

*  最终的变量是放在当前线程的 `ThreadLocalMap` 中，并不是存在 `ThreadLocal` 上，`ThreadLocal` 只是封装了`ThreadLocalMap`并传递了变量值 

*  `ThrealLocal` 类中可以通过 `Thread.currentThread()` 获取到当前线程对象后，直接通过 `getMap(Thread t)`可以访问到该线程的`ThreadLocalMap`对象。 

*  每个`Thread`中都具备一个`ThreadLocalMap`，而`ThreadLocalMap`以`ThreadLocal`为 key ，以 `Object` 对象为 value 的键值对。

```java
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    //......
}
```

 如果在同一个线程中声明两个 `ThreadLocal` 对象的话， `Thread`内部都是使用仅有的那个`ThreadLocalMap` 存放数据的，`ThreadLocalMap`的 key 就是 `ThreadLocal`对象，value 就是 `ThreadLocal` 对象调用`set`方法设置的值。 

> `ThreadLocal` 数据结构：

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-22.png)

> `ThreadLocalMap`是`ThreadLocal`的静态内部类

```java
 static class ThreadLocalMap {

     ... 
        /**
         * Construct a new map initially containing (firstKey, firstValue).
         * ThreadLocalMaps are constructed lazily, so we only create
         * one when we have at least one entry to put in it.
         */
        ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
```

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-23.png)

## ThreadLocal 内存泄露问题

*  `ThreadLocalMap` 中使用的 key 为 `ThreadLocal` 的弱引用，而 value 是强引用。 
*  如果 `ThreadLocal` 没有被外部强引用的情况下，垃圾回收时 `key` 会被清理掉，而 `value` 不会被清理掉。 这样一来，`ThreadLocalMap` 中就会出现 key 为 **null** 的 Entry。  
*  如果不做任何措施，`value` 永远无法被 GC 回收，此时就可能会产生内存泄露。
*  `ThreadLocalMap` 实现中已经考虑了这种情况，在调用 `set()`、`get()`、`remove()` 方法时会清理掉 key 为 **null** 的记录。  
*  使用完 `ThreadLocal`方法后最好手动调用`remove()`方法 

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```



#  线程池

## 线程池

> WHAT 线程池

线程池就是管理一系列线程的资源池。当有任务要处理时，直接从线程池中获取线程来处理，处理完之后线程并不会立即被销毁，而是等待下一个任务。 

>  WHY 线程池

 池化技术的思想主要是为了减少每次获取资源的消耗，提高对资源的利用率，**线程池**提供一种限制和管理资源的方式：

- **降低资源消耗**。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
- **提高响应速度**。当任务到达时，任务可以不需要等到线程创建就能立即执行。
- **提高线程的可管理性**。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

> 创建线程池

**方式一：通过`ThreadPoolExecutor`构造函数来创建（推荐）。** 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-24.png)

**方式二：通过 `Executor` 框架的工具类 `Executors` 来创建。** 

 可以创建多种类型的 `ThreadPoolExecutor` 

`FixedThreadPool`：

* 该方法返回一个固定线程数量的线程池，线程数量始终不变。
* 当有一个新的任务提交时，线程池中若有空闲线程，则立即执行。若没有，则新的任务会被暂存在一个任务队列中，待有线程空闲时，便处理在任务队列中的任务。

`SingleThreadExecutor`： 

- 该方法返回一个只有一个线程的线程池。
- 若多余一个任务被提交到该线程池，任务会被保存在一个任务队列中，待线程空闲，按**先入先出**的顺序执行队列中的任务。

`CachedThreadPool`： 

* 该方法返回一个可根据实际情况调整线程数量的线程池。初始大小为 0。
* 当有新任务提交时，如果当前线程池中没有线程可用，它会创建一个新的线程来处理该任务。
* 如果在一段时间内（默认为 60 秒）没有新任务提交，核心线程会超时并被销毁，从而缩小线程池的大小。

`ScheduledThreadPool`：

*  该方法返回一个用来在给定的延迟后运行任务或者定期执行任务的线程池 

> 对应 `Executors` 工具类中的方法如图所示： 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-25.png)

## 不推荐使用内置线程池

>  在《阿里巴巴 Java 开发手册》“并发处理”这一章节，**明确指出线程资源必须通过线程池提供，不允许在应用中自行显式创建线程**。 
>
>  另外，《阿里巴巴 Java 开发手册》中**强制线程池不允许使用 `Executors` 去创建，而是通过 `ThreadPoolExecutor` 构造函数的方式**，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险 

*  使用线程池可以减少在创建和销毁线程上所消耗的时间以及系统资源开销，解决资源不足的问题。
*  如果不使用线程池，有可能会造成系统创建大量同类线程而导致消耗完内存或者“过度切换”的问题。 

> `Executors` 返回线程池对象的弊端：

*  `FixedThreadPool` 和 `SingleThreadExecutor`：使用的是无界的  `LinkedBlockingQueue`，任务队列最大长度为 `Integer.MAX_VALUE`，可能堆积大量的请求，从而导致 OOM。 

*  `CachedThreadPool`：使用的是同步队列 `SynchronousQueue`，允许创建的线程数量为  `Integer.MAX_VALUE` ，如果任务数量过多且执行速度较慢，可能会创建大量的线程，从而导致 OOM。 

*  `ScheduledThreadPool` 和 `SingleThreadScheduledExecutor` ：使用的无界的延迟阻塞队列  `DelayedWorkQueue`，任务队列最大长度为 `Integer.MAX_VALUE`，可能堆积大量的请求，从而导致 OOM。 

````java
// 无界队列 LinkedBlockingQueue
public static ExecutorService newFixedThreadPool(int nThreads) {

    return new ThreadPoolExecutor(nThreads, nThreads,0L, TimeUnit.MILLISECONDS,new LinkedBlockingQueue<Runnable>());
}

// 无界队列 LinkedBlockingQueue
public static ExecutorService newSingleThreadExecutor() {

    return new FinalizableDelegatedExecutorService (new ThreadPoolExecutor(1, 1,0L, TimeUnit.MILLISECONDS,new LinkedBlockingQueue<Runnable>()));
}

// 同步队列 SynchronousQueue，没有容量，最大线程数是 Integer.MAX_VALUE`
public static ExecutorService newCachedThreadPool() {

    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,60L, TimeUnit.SECONDS,new SynchronousQueue<Runnable>());
}

// DelayedWorkQueue（延迟阻塞队列）
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
          new DelayedWorkQueue());
}
````

## 线程池常见参数

```java
// 用给定的初始参数创建一个新的ThreadPoolExecutor。
public ThreadPoolExecutor(int corePoolSize, // 线程池的核心线程数量
                          int maximumPoolSize, // 线程池的最大线程数
                          long keepAliveTime, // 当线程数大于核心线程数时，多余的空闲线程存活的最长时间
                          TimeUnit unit, // 时间单位
                          BlockingQueue<Runnable> workQueue, // 任务队列，用来储存等待执行任务的队列
                          ThreadFactory threadFactory, // 线程工厂，用来创建线程，一般默认即可
                          RejectedExecutionHandler handler // 拒绝策略，当提交的任务过多而不能及时处理时，我们可以定制策略来处理任务
                           ) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

> `ThreadPoolExecutor` 3 个最重要的参数： 

| 参数              | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| `corePoolSize`    | 任务队列未达到队列容量时，最大可以同时运行的线程数量。       |
| `maximumPoolSize` | 任务队列中存放的任务达到队列容量时，当前可以同时运行的线程数量变为最大线程数。 |
| `workQueue`       | 新任务来时会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中 |

> `ThreadPoolExecutor`其他常见参数 ：

| 参数            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| `keepAliveTime` | 线程池中的线程数量大于 `corePoolSize` 时，如果没有新的任务提交，多余的空闲线程不会立即销毁，而是会等待，直到等待时间超过 `keepAliveTime`才会被回收销毁，线程池回收线程时，会对核心线程和非核心线程一视同仁，直到线程池中线程的数量等于 `corePoolSize` ，回收过程才会停止。 |
| `unit`          | `keepAliveTime` 参数的时间单位。                             |
| `threadFactory` | executor 创建新线程的时候会用到                              |
| `handler`       | 饱和策略                                                     |

>  加深你对线程池中各个参数的相互关系的理解（图片来源：《Java 性能调优实战》） 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-26.png)

## 线程池的饱和策略

 如果当前同时运行的线程数量达到最大线程数量并且队列也已经被放满了任务时，`ThreadPoolTaskExecutor` 定义一些策略: 

| 策略                                     | 说明                                                         |
| ---------------------------------------- | ------------------------------------------------------------ |
| `ThreadPoolExecutor.AbortPolicy`         | 抛出异常 `RejectedExecutionException`来拒绝新任务的处理。    |
| `ThreadPoolExecutor.CallerRunsPolicy`    | 当线程池无法接受新任务，并且工作队列已满时，该拒绝策略会让提交任务的线程（调用线程）来直接执行被拒绝的任务，而不是抛弃任务或抛出异常。 |
| `ThreadPoolExecutor.DiscardPolicy`       | 不处理新任务，直接丢弃掉                                     |
| `ThreadPoolExecutor.DiscardOldestPolicy` | 此策略将丢弃最早的未处理的任务请求。                         |

- Spring 通过 `ThreadPoolTaskExecutor` 或者直接通过  `ThreadPoolExecutor` 的构造函数创建线程池的时候，当不指定饱和策略来配置线程池时**默认使用** `AbortPolicy` 

- 在这种饱和策略下，如果队列满了，`ThreadPoolExecutor` 将抛出 `RejectedExecutionException` 异常来拒绝新来的任务  

- 如果不想丢弃任务，可以使用 `CallerRunsPolicy`，将任务回退给调用者，使用调用者的线程来执行任务 

```java
public static class CallerRunsPolicy implements RejectedExecutionHandler {

    public CallerRunsPolicy() { }

    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (!e.isShutdown()) {
            // 直接主线程执行，而不是线程池中的线程执行
            r.run();
        }
    }
}
```

```java
import java.util.concurrent.*;

public class CallerRunsPolicyExample {

    public static void main(String[] args) {
        // 创建一个线程池，设置最大线程数为2，工作队列大小为2，使用CallerRunsPolicy拒绝策略
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
                2, // corePoolSize
                2, // maximumPoolSize
                0L, // keepAliveTime
                TimeUnit.MILLISECONDS, // unit
                new LinkedBlockingQueue<>(2), // workQueue
                Executors.defaultThreadFactory(), // threadFactory
                new ThreadPoolExecutor.CallerRunsPolicy()); // handler

        // 提交5个任务给线程池
        for (int i = 0; i < 5; i++) {
            Task task = new Task("Task " + i);
            executor.submit(task);
        }

        // 关闭线程池
        executor.shutdown();
    }

    static class Task implements Runnable {
        private String name;

        public Task(String name) {
            this.name = name;
        }

        @Override
        public void run() {
            System.out.println("Executing task: " + name + " in thread " + Thread.currentThread().getName());
            try {
                // 模拟任务执行时间
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```



## 线程池常用的阻塞队列

 新任务来时会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中，不同的线程池会选用不同的阻塞队列 

>  `LinkedBlockingQueue`（无界队列） 

*  使用方：`FixedThreadPool` 和 `SingleThreadExector` 
*  在这两个内置线程池中`LinkedBlockingQueue`属于容量为 `Integer.MAX_VALUE` 的无界队列
*  `FixedThreadPool` 最多只能创建核心线程数的线程（核心线程数和最大线程数相等） 
*  `SingleThreadExector` 只能创建一个线程（核心线程数和最大线程数都是 1） 
*  二者的任务队列永远不会被放满，从而导致 OOM

>  `SynchronousQueue`（同步队列） 

* 使用方：` CachedThreadPool` 
* `SynchronousQueue` 没有容量，不存储元素，目的是保证对于提交的任务，如果有空闲线程，则使用空闲线程来处理；否则新建一个线程来处理任务。 
* `CachedThreadPool` 的最大线程数是 `Integer.MAX_VALUE` ，线程数是可以无限扩展的，可能会创建大量线程，从而导致 OOM。 

>  `DelayedWorkQueue`（延迟阻塞队列） 

*  使用方：`ScheduledThreadPool` 和 `SingleThreadScheduledExecutor` 
*  `DelayedWorkQueue` 的内部元素并不是按照放入的时间排序，而是按照延迟的时间长短对任务进行排序，内部采用的是 “堆” 的数据结构，可以保证每次出队的任务都是当前队列中执行时间最靠前的。 

*  `DelayedWorkQueue`  添加元素满后会自动扩容原来容量的 1/2，即永远不会阻塞，最大扩容可达 `Integer.MAX_VALUE`，所以最多只能创建核心线程数的线程。 

## 线程池处理任务的流程

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-27.png)

1. 如果当前运行的线程数小于核心线程数，那么就会创建线程来执行任务 

2. 如果当前运行的线程数大于等于核心线程数，但是小于最大线程数，那么就把该任务放入任务队列里等待执行 

3. 如果任务队列已满，但是当前运行的线程数小于最大线程数，就创建线程来执行任务 

4. 如果当前运行的线程数已经等于最大线程数，新建线程将会使当前运行的线程超出最大线程数，那么按照任务拒绝策略处理该任务

## 命名线程池

*  默认情况下创建的线程名字类似 `pool-1-thread-n` 这样的，没有业务含义，不利于我们定位问题。 
*  初始化线程池时显示命名（设置线程池名称前缀）有利于定位问题。

>  **1、利用 guava 的 `ThreadFactoryBuilder`** 

```java
ThreadFactory threadFactory = new ThreadFactoryBuilder()
                        .setNameFormat(threadNamePrefix + "-%d")
                        .setDaemon(true).build();
ExecutorService threadPool = new ThreadPoolExecutor(corePoolSize, maximumPoolSize, keepAliveTime, TimeUnit.MINUTES, workQueue, threadFactory);
```

> **2、自己实现 `ThreadFactory`。** 

```java
public final class NamingThreadFactory implements ThreadFactory {

    private final AtomicInteger threadNum = new AtomicInteger();
    private final ThreadFactory delegate;
    private final String name;

    /**
     * 创建一个带名字的线程池生产工厂
     */
    public NamingThreadFactory(ThreadFactory delegate, String name) {
        this.delegate = delegate;
        this.name = name; // TODO consider uniquifying this
    }

    @Override
    public Thread newThread(Runnable r) {
        Thread t = delegate.newThread(r);
        t.setName(name + " [#" + threadNum.incrementAndGet() + "]");
        return t;
    }

}
```

## 设定线程池的大小

> 设置的线程池数量太小

如果同一时间有大量任务/请求需要处理，可能会导致大量的请求/任务在任务队列中排队等待执行，甚至会出现任务队列满了之后任务/请求无法处理的情况，或者大量任务堆积在任务队列导致 OOM。

> 设置线程数量太大

大量线程可能会同时在争取 CPU 资源，这样会导致大量的上下文切换，从而增加线程的执行时间，影响了整体执行效率。

> 推荐设置方案

**CPU 密集型任务(N+1)：**  

*   这种任务消耗的主要是 CPU 资源，可以将线程数设置为 N（CPU 核心数）+1。 
*   比 CPU 核心数多出来的一个线程是为了防止线程偶发的缺页中断，或者其它原因导致的任务暂停而带来的影响。 
*   一旦任务暂停，CPU 就会处于空闲状态，而在这种情况下多出来的一个线程就可以充分利用 CPU 的空闲时间。 

**I/O 密集型任务(2N)：** 

*  这种任务应用起来，系统会用大部分的时间来处理 I/O 交互，而线程在处理 I/O 的时间段内不会占用 CPU 来处理，这时就可以将 CPU 交出给其它线程使用。 
*  因此在 I/O 密集型任务的应用中，我们可以多配置一些线程，具体的计算方法是 2N。 

> **如何判断是 CPU 密集任务还是 IO 密集任务？** 

*  CPU 密集型是利用 CPU 计算能力的任务，比如你在内存中对大量数据进行排序。 
*  但凡涉及到网络读取，文件读取这类都是 IO 密集型，这类任务的特点是 CPU 计算耗费时间相比于等待 IO 操作完成的时间来说很少，大部分时间都花在了等待 IO 操作完成上。 

## 动态修改线程池的参数

美团技术团队在[《Java 线程池实现原理及其在美团业务中的实践》](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)这篇文章中介绍到对线程池参数实现可自定义配置的思路和方法。 

> 美团技术团队的思路是主要对线程池的核心参数实现自定义可配置。这三个核心参数是： 

| 参数              | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| `corePoolSize`    | 核心线程数线程数定义了最小可以同时运行的线程数量             |
| `maximumPoolSize` | 当队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数。 |
| `workQueue`       | 当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。 |

**如何支持参数动态配置？** 且看 `ThreadPoolExecutor` 提供的下面这些方法。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-28.png)

格外需要注意的是corePoolSize， 程序运行期间的时候，我们调用 setCorePoolSize（）这个方法的话，线程池会首先判断当前工作线程数是否大于corePoolSize，如果大于的话就会回收工作线程

另外，你也看到了上面并没有动态指定队列长度的方法，美团的方式是自定义了一个叫做 `ResizableCapacityLinkedBlockIngQueue` 的队列（主要就是把 `LinkedBlockingQueue` 的 capacity 字段的 final 关键字修饰给去掉了，让它变为可变的）。

> 深挖技术

- [《Java 线程池实现原理及其在美团业务中的实践》](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)

- [如何设置线程池参数？美团给出了一个让面试官虎躯一震的回答。](https://mp.weixin.qq.com/s?__biz=Mzg3NjU3NTkwMQ==&mid=2247505103&idx=1&sn=a041dbec689cec4f1bbc99220baa7219&source=41#wechat_redirect)

> 如果我们的项目也想要实现这种效果的话，可以借助现成的开源项目： 

*  **[Hippo4j](https://github.com/opengoofy/hippo4j)** ： 异步线程池框架，支持线程池动态变更&监控&报警，无需修改代码轻松引入。支持多种使用模式，轻松引入，致力于提高系统运行保障能力。 
*  **[Dynamic TP](https://github.com/dromara/dynamic-tp)** ： 轻量级动态线程池，内置监控告警功能，集成三方中间件线程池管理，基于主流配置中心（已支持 Nacos、Apollo，Zookeeper、Consul、Etcd，可通过 SPI 自定义实现）。 

## 设计能够根据任务的优先级来执行的线程池

> **不同的线程池会选用不同的阻塞队列作为任务队列**，
>
> 比如`FixedThreadPool` 使用的是  `LinkedBlockingQueue`（无界队列），由于队列永远不会被放满 ，因此`FixedThreadPool`最多只能创建核心线程数的线程。 

 假如我们需要实现一个优先级任务线程池的话，那可以考虑使用 `PriorityBlockingQueue` （优先级阻塞队列）作为任务队列  （`ThreadPoolExecutor` 的构造函数有一个 `workQueue` 参数可以传入任务队列）。 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-29.png)

 `PriorityBlockingQueue` 是一个支持优先级的无界阻塞队列，可以看作是线程安全的。 `PriorityQueue`，两者底层都是使用小顶堆形式的二叉堆，即值最小的元素优先出队。不过  `PriorityQueue` 不支持阻塞操作。 

 要想让 `PriorityBlockingQueue` 实现对任务的排序，传入其中的任务必须是具备排序能力的：

* 方式1： 提交到线程池的任务实现 `Comparable` 接口，并重写 `compareTo` 方法来指定任务之间的优先级比较规则 

* 方式2：创建 `PriorityBlockingQueue` 时传入一个 `Comparator` 对象来指定任务之间的排序规则(推荐)。

>  不过，这存在一些风险和问题，比如 

- `PriorityBlockingQueue` 是无界的，可能堆积大量的请求，从而导致 OOM。

- 可能会导致饥饿问题，即低优先级的任务长时间得不到执行。

- 由于需要对队列中的元素进行排序操作以及保证线程安全（并发控制采用的是可重入锁 `ReentrantLock`），因此会降低性能。

> 解决方案

* 对于 OOM 这个问题的解决比较简单粗暴，就是继承`PriorityBlockingQueue` 并重写一下 `offer` 方法(入队)的逻辑，当插入的元素数量超过指定值就返回 false 。 

* 饥饿问题这个可以通过优化设计来解决（比较麻烦），比如等待时间过长的任务会被移除并重新添加到队列中，但是优先级会被提升 

* 对于性能方面的影响，是没办法避免的，毕竟需要对任务进行排序操作。并且，对于大部分业务场景来说，这点性能影响是可以接受的 

#  Future

## Future 类

 在 Java 8 中，`Future` 接口是用于表示异步计算结果的一种方式。它的设计目的是为了在执行异步任务时能够获取任务的执行结果，以及能够在任务完成后执行一些操作，主要用在一些需要执行耗时任务的场景，避免程序一直原地等待耗时任务执行完成，执行效率太低。 

如果有一个任务提交给 `Future` 来处理，任务执行期间我自己可以去做任何想做的事情。 在这期间我还可以取消任务以及获取任务的执行状态。一段时间后就可以从 `Future` 直接取出任务执行结果。 

> 说明

 当我们执行某一耗时的任务时，可以将这个耗时任务交给一个子线程去异步执行，同时我们可以干点其他事情，不用傻傻等待耗时任务执行完成。

等我们的事情干完后，我们再通过 `Future` 类获取到耗时任务的执行结果。这样一来，程序的执行效率就明显提高了。 

> 在 Java 中，`Future` 类只是一个泛型接口，位于 `java.util.concurrent` 包下，其中定义了 5 个方法，主要包括下面这 4 个功能： 

- 取消任务；
- 判断任务是否被取消;
- 判断任务是否已经执行完成;
- 获取任务执行结果。

```java
// V 代表了Future执行的任务返回值的类型
public interface Future<V> {
    // 取消任务执行
    // 成功取消返回 true，否则返回 false
    boolean cancel(boolean mayInterruptIfRunning);
    // 判断任务是否被取消
    boolean isCancelled();
    // 判断任务是否已经执行完成
    boolean isDone();
    // 获取任务执行结果
    V get() throws InterruptedException, ExecutionException;
    // 指定时间内没有返回计算结果就抛出 TimeOutException 异常
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutExceptio

}
```

> 案例演示

```java
import java.util.concurrent.*;

public class FutureExample {

    public static void main(String[] args) {
        // 创建一个线程池
        ExecutorService executor = Executors.newFixedThreadPool(1);

        // 提交异步任务
        Future<Integer> future = executor.submit(() -> {
            // 模拟耗时任务
            Thread.sleep(2000);
            return 42;
        });

        // 执行其他操作...

        try {
            // 等待任务完成，并获取结果
            Integer result = future.get();
            System.out.println("异步任务的结果是: " + result);
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }

        // 关闭线程池
        executor.shutdown();
    }
}
```



## Callable 和 Future 关系

 `FutureTask` 提供了 `Future` 接口的基本实现，常用来封装 `Callable` 和 `Runnable`，具有取消任务、查看任务是否执行完成以及获取任务执行结果的方法。 

> `ExecutorService.submit()` 方法返回的其实就是 `Future` 的实现类 `FutureTask` 。 

```java
<T> Future<T> submit(Callable<T> task);
Future<?> submit(Runnable task);
```

> `FutureTask` 不光实现了 `Future`接口，还实现了`Runnable` 接口，因此可以作为任务直接被线程执行 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-30.png)

 `FutureTask` 有两个构造函数，可传入 `Callable` 或者 `Runnable` 对象。实际上，传入 `Runnable` 对象也会在方法内部转换为`Callable` 对象。 

```java
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;
}

public FutureTask(Runnable runnable, V result) {
    // 通过适配器RunnableAdapter来将Runnable对象runnable转换成Callable对象
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;
}

```

 `FutureTask`相当于对`Callable` 进行了封装，管理着任务执行的情况，存储了 `Callable` 的 `call` 方法的任务执行结果 

> `Callable` 

-  `Callable` 接口是 Java 5 引入的一个与 `Runnable` 相似的接口，用于表示可以由线程执行的任务，并且可以返回一个结果
-  相比之下，`Runnable` 接口的 `run` 方法不返回任何结果，而 `Callable` 接口的 `call` 方法可以返回一个泛型类型的结果。 

```java
// A task that returns a result and may throw an exception.
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}

```

```java
@FunctionalInterface
public interface Runnable {
    /**
     * When an object implementing interface <code>Runnable</code> is used
     * to create a thread, starting the thread causes the object's
     * <code>run</code> method to be called in that separately executing
     * thread.
     * <p>
     * The general contract of the method <code>run</code> is that it may
     * take any action whatsoever.
     *
     * @see     java.lang.Thread#run()
     */
    public abstract void run();
}

```

在 Java 8 中，`Callable` 主要用于和 `ExecutorService` 配合使用，通过 `submit` 方法提交 `Callable` 对象，返回一个 `Future` 对象，可以通过 `Future` 对象获取任务的执行结果。`Future` 对象是一个异步计算的结果的句柄，可以在将来的某个时刻获取任务的执行结果。 

```java
import java.util.concurrent.*;

public class CallableExample {

    public static void main(String[] args) {
        // 创建一个线程池
        ExecutorService executor = Executors.newSingleThreadExecutor();

        // 提交 Callable 任务
        Future<Integer> future = executor.submit(new MyCallable());

        // 执行其他操作...

        try {
            // 获取 Callable 任务的执行结果
            Integer result = future.get();
            System.out.println("Callable 任务的结果是: " + result);
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }

        // 关闭线程池
        executor.shutdown();
    }

    // 自定义的 Callable 类
    static class MyCallable implements Callable<Integer> {
        @Override
        public Integer call() throws Exception {
            // 模拟耗时任务
            Thread.sleep(2000);
            return 42;
        }
    }
}

```

 通过 `executor.submit(new MyCallable())` 提交了一个 `Callable` 任务，返回一个 `Future` 对象。后续通过 `future.get()` 方法等待任务完成，并获取任务的执行结果。需要注意的是，`get` 方法会阻塞当前线程，直到任务完成。 

> **Callable 接口和 FutureTask 类**

1. 自定义类实现 Callable 接口
2. 在自定义类中重写 **call()** 方法
3. 创建自定义类的对象
4. 创建 FutureTask 类的对象，把自定义类的对象作为构造方法的参数
5. 创建 Thread 类的对象，把 FutureTask 对象作为构造方法的参数
6. FutrueTask 中的 **get()** 方法可以获取线程执行完后的返回值，如果在线程开启前调用该方法，那么该程序会一直停留在该代码处

```java
public Thread(Runnable target);
public class FutureTask<V> implements RunnableFuture<V>;
public interface RunnableFuture<V> extends Runnable, Future<V>;
public FutureTask(Callable<V> callable);

```



```java
public class Test {
    public static void main(String[] args){
        // 创建线程1
        MyCallable myCallable = new MyCallable();
        FutureTask<String> futureTask = new FutureTask<>(myCallable);
        Thread thread = new Thread(futureTask);
        // 启动线程
        thread.start();
        // 获取执行完的结果
        System.out.println(futureTask.get());
    }
}

class MyCallable implements Callable<String> {
    @Override
    public String call() throws Exception {
        for(int i = 0;i < 100;i++){
            System.out.println(i);
        }
        // 返回值表示线程执行完返回的结果
        return "该线程执行完啦";
    }
}

```

## CompletableFuture 类

> Future 不足

 `Future` 在实际使用过程中存在一些局限性比如不支持异步任务的编排组合、获取计算结果的 `get()` 方法为阻塞调用。  `Future`接口对任务的完成状态只能被查询，而不能主动获取任务的执行结果。

为了解决这个问题，Java 8 引入了`CompletableFuture`类，它实现了`Future`接口，并提供了更丰富的方法来处理异步任务的结果。`CompletableFuture`允许你以函数式的方式处理异步任务的完成事件，使得编写异步代码更加灵活和直观。 

> 解决方案

 Java 8 引入`CompletableFuture` 类可以解决`Future` 的这些缺陷。 `CompletableFuture` 除提供更为好用和强大的 `Future` 特性之外，还提供了函数式编程、异步任务编排组合  （可以将多个异步任务串联起来，组成一个完整的链式调用）等能力。 

```java
public class CompletableFuture<T> implements Future<T>, CompletionStage<T> {
}

```

 可以看到，`CompletableFuture` 同时实现了 `Future` 和 `CompletionStage` 接口。 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-31.png)

-  `CompletionStage` 接口描述了一个异步计算的阶段。很多计算可以分成多个阶段或步骤，此时可以通过它将所有步骤组合起来，形成异步计算的流水线。 

-  `CompletionStage` 接口中的方法比较多，`CompletableFuture` 的函数式能力就是这个接口赋予的。从这个接口的方法参数你就可以发现其大量使用了 Java8 引入的函数式编程。 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-32.png)



 **异步执行任务：** 

```java
CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
    // 异步执行的任务
    System.out.println("Async task executed by thread: " + Thread.currentThread().getName());
});

// 等待异步任务完成
future.join();

```

 **异步执行有返回值的任务：** 

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    // 异步执行的任务，返回一个结果
    System.out.println("Async task executed by thread: " + Thread.currentThread().getName());
    return 42;
});

// 等待异步任务完成并获取结果
Integer result = future.join();

```

 **组合多个 CompletableFuture：** 

```java
CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(() -> 2);
CompletableFuture<Integer> future2 = CompletableFuture.supplyAsync(() -> 3);

// 将两个 CompletableFuture 的结果进行合并
CompletableFuture<Integer> combinedFuture = future1.thenCombine(future2, (result1, result2) -> result1 + result2);

// 等待合并后的 CompletableFuture 完成并获取结果
Integer combinedResult = combinedFuture.join();

```

 **处理异常：** 

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    // 异步执行的任务，可能抛出异常
    throw new RuntimeException("Task failed");
});

// 处理异常
CompletableFuture<Integer> result = future.exceptionally(ex -> {
    System.out.println("Exception occurred: " + ex.getMessage());
    return 0; // 默认返回值
});

```

 **异步执行任务链：** 

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> 2)
        .thenApplyAsync(result -> result * 3)
        .thenApplyAsync(result -> result + 1);

// 等待任务链完成并获取最终结果
Integer finalResult = future.join();

```

 **等待多个 CompletableFuture 完成：** 

```java
CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(() -> 2);
CompletableFuture<Integer> future2 = CompletableFuture.supplyAsync(() -> 3);

// 等待所有 CompletableFuture 完成
CompletableFuture<Void> allOf = CompletableFuture.allOf(future1, future2);

// 等待任一 CompletableFuture 完成
CompletableFuture<Object> anyOf = CompletableFuture.anyOf(future1, future2);

```

 **自定义线程池：** 

```	java
ExecutorService executor = Executors.newFixedThreadPool(5);

CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> 2, executor);

// 在使用完 CompletableFuture 后，需要手动关闭线程池
executor.shutdown();

```

> 更多请参考：

[CompletableFuture](https://cyanzzy.github.io/2023/08/12/JUC-2-CompletableFuture/)

# AQS

## AQS

 AQS 是 Java 中的一个抽象基类，用于实现同步器（Synchronizer）。它是Java并发包中一些重要的同步工具的基础，如ReentrantLock、Semaphore、CountDownLatch等。AQS提供了一种底层机制，帮助开发者实现自定义的同步器，从而能够更灵活地管理多线程之间的竞争和协作。 

- `AbstractQueuedSynchronizer` (AQS) 是 Java 5 引入的一个用于实现同步器的抽象基类，位于`java.util.concurrent.locks` 包下面。
- 它为实现锁、信号量、倒计数器等同步器提供框架，是许多并发工具的基础，如 `ReentrantLock`、`Semaphore`、`CountDownLatch` 等。 

- **AQS 就是一个抽象类，主要用来构建锁和同步器。** 

```java
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer implements java.io.Serializable {
}

```

> AQS 主要特点

**状态管理：** AQS 内部维护一个整数状态，通过 `getState()` 和 `setState(int newState)` 方法进行读取和设置。状态的具体含义由使用 AQS 的同步器自行定义。

```java
// The synchronization state.
private volatile int state;

```

**等待队列：** AQS 使用一个双向链表作为等待队列，用于存储等待线程。每个节点表示一个等待线程，并保存了线程的引用。

**独占锁和共享锁：** AQS 支持独占锁（exclusive lock）和共享锁（shared lock）两种模式。独占锁适用于只能被一个线程持有的锁，而共享锁可以被多个线程同时持有。

**Condition 条件对象：** AQS 提供了 `Condition` 接口，可以通过 `newCondition()` 方法创建与 AQS 关联的条件对象，用于在特定条件下等待或唤醒线程。

**acquire 和 release 操作：** AQS 提供了 `acquire` 和 `release` 方法，用于实现同步器的独占模式。`acquire` 用于获取锁，`release` 用于释放锁。

**tryAcquire 和 tryRelease 操作：** AQS 还提供了 `tryAcquire` 和 `tryRelease` 方法，允许实现者以非阻塞的方式尝试获取或释放锁，通常用于实现可轮询的锁请求。

**模板方法设计模式：** AQS 采用了模板方法设计模式，将同步器的核心逻辑封装在模板方法中，而具体的同步器需要实现抽象方法来定义自己的状态管理和控制流程。

>  **同步器的概念**： 

 同步器是一种用于控制多线程访问共享资源的机制，它通常包含两个重要的方法：`acquire`（获取锁）和`release`（释放锁）。这两个方法分别用于在多线程之间协调资源的访问，确保线程安全。 

>  锁和同步器关系 

-  锁：面向锁使用者，定义程序员和锁交互的API，屏蔽实现细节 
-  同步器：统一规范简化锁的实现，将其抽象出来，屏蔽了同步状态管理、同步队列的管理和维护、阻塞线程排队和通知、唤醒机制等，它是一切锁和同步组件实现的公共基础部分。 

>  **AQS的基本结构**： 

 AQS的核心思想是维护一个等待队列（Wait Queue）和一个状态变量（State）。等待队列中的线程按照先进先出（FIFO）的顺序等待获取锁或资源。状态变量表示同步器的状态，不同的同步器可以定义不同的状态语义。 

>  **AQS的主要方法**： 

| 方法               | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| `acquire(int arg)` | 该方法是独占模式的获取资源操作，调用 `tryAcquire` 方法尝试获取资源，如果失败则通过 `acquireQueued` 方法将线程加入等待队列并自旋等待或阻塞，直到获取到锁或资源 |
| `release(int arg)` | 该方法是独占模式的释放资源操作，调用 `tryRelease` 方法尝试释放资源，如果成功则唤醒等待队列中的下一个线程。 |
| `tryAcquire`       | 尝试以独占模式获取。该方法应查询对象的状态是否允许以独占模式获取，如果允许，则获取该对象。若成功获取则返回 `true`，否则返回 `false`。 |
| `tryRelease`       | 该方法用于尝试释放独占资源，如果成功释放则返回 `true`，否则返回 `false`。 |
| `getState`         | 获取锁的标志state值                                          |
| `setState`         | 设置锁的标志state值                                          |
| `hasQueuedThreads` | 检查是否有线程在等待队列中等待获取锁或资源                   |

>  **AQS的应用**： 

- **ReentrantLock**：Java中可重入锁的实现就是基于AQS的，它允许同一个线程多次获取同一个锁。
- **Semaphore**：Semaphore也是基于AQS的，它用于控制同时访问某个资源的线程数量。
- **CountDownLatch**：CountDownLatch是用于线程之间的协调，它允许一个或多个线程等待一组操作完成。
- **CyclicBarrier**：CyclicBarrier也是用于线程之间的协调，它允许一组线程互相等待达到某个同步点后再继续执行。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-33.png)

## AQS 原理

> AQS 核心思想

* 如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。
* 如果共享资源被占用，就需要一定的阻塞等待唤醒机制来保证锁分配，该机制主要用的是CLH队列的变体实现的，将暂时获取不到锁的线程加入到队列中。它将要请求共享资源的线程及自身的等待状态封装成队列的结点对象（Node），通过CAS、自旋以及LockSupport.park()的方式，维护state变量的状态，使并发达到同步的效果。

> CLH

Craig,Landin,and Hagersten 队列是一个**虚拟的双向队列**（虚拟的双向队列即不存在队列实例，**仅存在结点之间的关联关系**）

* AQS 是将每条请求共享资源的线程封装成一个 CLH 锁队列的一个结点（Node）来实现锁的分配
* 在 CLH 同步队列中，一个节点表示一个线程，它保存着线程的引用（thread）、 当前节点在队列中的状态（waitStatus）、前驱节点（prev）、后继节点（next）。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-34.png)

>  AQS 核心原理图 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-35.png)

 AQS通过使用CAS（Compare-And-Swap）操作来实现对状态变量的原子操作，从而保证线程安全。等待队列则是使用链表数据结构来管理等待线程。 

 `AQS`中 维护了一个`volatile int state`（代表共享资源）和一个`FIFO`线程等待队列（多线程争用资源被阻塞时会进入此队列）。 

* AQS 使用 **int 成员变量 `state` 表示同步状态**
* 通过内置的 **线程等待队列** 来完成获取资源线程的排队工作。 

 `state` 变量由 `volatile` 修饰，用于展示当前临界资源的获锁情况。 

````java
// 共享变量，使用volatile修饰保证线程可见性
private volatile int state;

````

 另外，状态信息 `state` 可以通过 `protected` 类型的 `getState()`、`setState()`和`compareAndSetState()` 进行操作。并且，这几个方法都是  `final` 修饰的，在子类中无法被重写。 

```java
// 返回同步状态的当前值
protected final int getState() {
     return state;
}
 // 设置同步状态的值
protected final void setState(int newState) {
     state = newState;
}
// 原子地（CAS操作）将同步状态值设置为给定值update如果当前同步状态的值等于expect（期望值）
protected final boolean compareAndSetState(int expect, int update) {
      return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}

```

> 以 `ReentrantLock` 为例

**推荐文章**：

- [AQS 源码分析--以ReentrantLock说明](https://cyanzzy.github.io/2023/09/06/JUC-11-AQS/#%E4%BB%A5ReentrantLock%E8%AF%B4%E6%98%8E)
- [美团：从ReentrantLock的实现看AQS的原理及应用](https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html)
- [从 ReentrantLock 的实现看 AQS 的原理及应用 - 美团技术团队](https://javaguide.cn/java/concurrent/reentrantlock.html)

可重入的互斥锁 `ReentrantLock` 为例，它的内部维护了  `state` 变量，用来表示锁的占用状态：

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-36.png)

1. `state` 初始值为 0，表示未锁定状态。
2. 当线程 A 调用 `lock()` 方法时，会尝试通过 `tryAcquire()` 方法独占该锁，并让 `state` 的值加 1
3. 如果成功，那么线程 A 就获取锁。如果失败，那么线程 A 就会被加入等待队列（CLH 队列），直到其他线程释放该锁
4. 假设线程 A 获取锁成功：当然，释放锁前，A 线程自己是可以重复获取此锁的（`state` 会累加）
5. 此后，其他线程再 `tryAcquire()` 时就会失败，直到 A 线程 `unlock()` 到 `state=`0（即释放锁）为止，其它线程才有机会获取该锁。

注意：获取多少次就要释放多少次，这样才能保证 state 是能回到零态的。 

>  以 `CountDownLatch` 以例 

1. 任务分为 N 个子线程去执行，`state` 也初始化为 N（注意 N 要与线程个数一致）。

2. 该 N 个子线程并行执行，每个子线程执行完  `countDown()` 一次，该方法会尝试使用 CAS 操作，让 `state` 的值减少 1。

3. 当所有子线程都执行完毕后（即 `state` 的值变为 0），`CountDownLatch` 会调用 `unpark()` 方法，唤醒主线程。此时主线程就可以从 `await()` 方法返回，继续执行后续操作

## Semaphore

> Semaphore

- `Semaphore` 是 Java 中的一个同步工具类，用于控制对某个共享资源的访问权限
- 它基于计数器的方式，维护了一个许可（permit）的数量，线程在访问共享资源之前需要先获得许可，每个线程访问共享资源会消耗一个许可，释放资源时会释放一个许可。如果许可数量为0，那么其他线程需要等待直到有可用的许可。 

- `synchronized` 和 `ReentrantLock` 都是一次只允许一个线程访问某个资源，而`Semaphore`(信号量)可以用来控制同时访问特定资源的线程数量 

> 栗子：

`Semaphore` 被初始化为2，表示同时最多允许两个线程访问共享资源。`Worker` 类模拟了多个线程同时访问资源的情况。每个线程在执行时，首先尝试通过 `acquire` 方法获取一个许可，然后执行一些工作，最后通过 `release` 方法释放许可。这样就可以保证同时最多有两个线程访问共享资源。

 ```java
import java.util.concurrent.Semaphore;

public class SemaphoreExample {

    public static void main(String[] args) {
        
        // 创建一个Semaphore实例，设置许可的数量为2
        Semaphore semaphore = new Semaphore(2);

        // 创建并启动5个线程
        for (int i = 1; i <= 5; i++) {
            Thread thread = new Thread(new Worker(semaphore, "Thread-" + i));
            thread.start();
        }
    }

    static class Worker implements Runnable {
        private Semaphore semaphore;
        private String name;

        public Worker(Semaphore semaphore, String name) {
            this.semaphore = semaphore;
            this.name = name;
        }

        @Override
        public void run() {
            try {
                System.out.println(name + " is trying to acquire a permit.");
                // 请求许可
                semaphore.acquire();
                System.out.println(name + " has acquired a permit.");

                // 模拟工作
                Thread.sleep(2000);

                System.out.println(name + " is releasing the permit.");
                // 释放许可
                semaphore.release();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

 ```

> `Semaphore` 有两种模式：

- **公平模式：** 调用 `acquire()` 方法的顺序就是获取许可证的顺序，遵循 FIFO；
- **非公平模式：** 抢占式的

> `Semaphore` 对应的两个构造方法如下： 

```java
public Semaphore(int permits) {
  	sync = new NonfairSync(permits);
}

public Semaphore(int permits, boolean fair) {
  	sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}

```

 这两个构造方法，都必须提供许可的数量，第二个构造方法可以指定是公平模式还是非公平模式，默认非公平模式。 

> 应用场景

 `Semaphore` 是一个非常有用的工具，特别适用于需要控制对资源访问的并发场景，如连接池管理、限流（仅限于单机模式） 等 

实际项目中推荐使用 [Redis +Lua 来做限流 ](https://blog.csdn.net/mingpingyao/article/details/129587679)

1. 首先，在Redis中创建一个键用于限流，例如`limit_key`。
2. 在Lua脚本中，使用`KEYS`获取限流键，使用`ARGV`获取限流大小。
3. 使用`redis.call('get', key)`获取当前请求数量，如果不存在则默认为0。
4. 判断当前请求数量是否超过限流大小，如果超过则返回false，表示限流。
5. 如果未超过限流大小，则将请求数量加1，并设置过期时间为2秒。
6. 返回true，表示请求通过限流。

```lua
local key = KEYS[1] -- 限流KEY（一秒一个）
local limit = tonumber(ARGV[1]) -- 限流大小
local current = tonumber(redis.call('get', key) or "0")

if current + 1 > limit then
    return false
else
    redis.call("INCRBY", key, "1")
    redis.call("expire", key, "2")
end

return true

```

 使用Redis的`EVAL`命令执行Lua脚本，将限流键和限流大小作为参数传递给脚本。如果返回true，则表示请求通过限流；如果返回false，则表示请求被限流。 

```bash
EVAL "$(cat limit.lua)" 1 limit_key 100

```



## Semaphore 原理

> `Semaphore` 是共享锁的一种实现，它默认构造 AQS 的 `state` 值为 `permits`， `permits` 的值为许可证的数量，只有拿到许可证的线程才能执行 

```java
Sync(int permits) { setState(permits);}

```

> acquire

*  调用`semaphore.acquire()` ，线程尝试获取许可证
*  如果 `state >= 0` 的话，则表示可以获取成功。如果获取成功的话，使用 CAS 操作去修改  `state` 的值 `state=state-1`。  
*  如果 `state<0` 的话，则表示许可证数量不足。此时会创建一个 Node 节点加入阻塞队列，挂起当前线程。 

```java
// 获取1个许可证
public void acquire() throws InterruptedException {
 	 sync.acquireSharedInterruptibly(1);
}

// 共享模式下获取许可证，获取成功则返回，失败则加入阻塞队列，挂起线程
public final void acquireSharedInterruptibly(int arg)
    throws InterruptedException {
    if (Thread.interrupted())
      throw new InterruptedException();
        // 尝试获取许可证，arg为获取许可证个数，当可用许可证数减当前获取的许可证数结果小于0,则创建一个节点加入阻塞队列，挂起当前线程。
    if (tryAcquireShared(arg) < 0)
      doAcquireSharedInterruptibly(arg);
}

```

> release

* 调用`semaphore.release();`  ，线程尝试释放许可证，并使用 CAS 操作去修改  `state` 的值  `state=state+1`。
* 释放许可证成功之后，同时会唤醒同步队列中的一个线程。被唤醒的线程会重新尝试去修 改 `state` 的值   `state=state-1` ，如果 `state>=0` 则获取令牌成功，否则重新进入阻塞队列，挂起线程。 

```java
// 释放一个许可证
public void release() {
  	sync.releaseShared(1);
}

// 释放共享锁，同时会唤醒同步队列中的一个线程。
public final boolean releaseShared(int arg) {
    // 释放共享锁
    if (tryReleaseShared(arg)) {
      // 唤醒同步队列中的一个线程
      doReleaseShared();
      return true;
    }
    return false;
}

```

## CountDownLatch

 `CountDownLatch` 是 Java 中的一个同步工具类，它可以用于确保某些线程在其他线程完成后再执行。它的核心思想是，创建一个计数器，初始值为某个数值，当计数器的值减为零时，等待在计数器上的线程会被唤醒。 

* `CountDownLatch` 允许 `count` 个线程阻塞在一个地方，直至所有线程的任务都执行完毕， 用于在多个线程之间协调工作。它通常用于等待一组线程完成某个任务，然后再执行其他任务。 
* `CountDownLatch` 是一次性的，计数器的值只能在构造方法中初始化一次，之后没有任何机制再次对其设置值，当 `CountDownLatch` 使用完毕后，它不能再次被使用 

> 栗子

 `CountDownLatch` 的初始计数为3。三个线程通过 `countDown` 方法将计数器减1，主线程通过 `await` 方法等待计数器减为0。当所有线程完成工作后，主线程会被唤醒，然后继续执行。 

```java
import java.util.concurrent.CountDownLatch;

public class CountDownLatchExample {

    public static void main(String[] args) {
        // 创建一个CountDownLatch实例，初始计数为3
        CountDownLatch latch = new CountDownLatch(3);

        // 创建并启动3个线程
        for (int i = 1; i <= 3; i++) {
            Thread thread = new Thread(new Worker(latch, "Thread-" + i));
            thread.start();
        }

        try {
            // 主线程等待计数器为0
            latch.await();
            System.out.println("All threads have completed their work.");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    static class Worker implements Runnable {
        private CountDownLatch latch;
        private String name;

        public Worker(CountDownLatch latch, String name) {
            this.latch = latch;
            this.name = name;
        }

        @Override
        public void run() {
            System.out.println(name + " is doing some work.");
            // 模拟工作
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(name + " has completed its work.");
            
            // 计数器减1
            latch.countDown();
        }
    }
}

```

> 适用场景

 `CountDownLatch` 的主要用途是等待一组线程完成某个任务，然后再继续执行其他任务。这对于协调多个线程并行执行的场景非常有用。 

## CountDownLatch 原理

> `CountDownLatch` 是共享锁的一种实现，它默认构造 AQS 的 `state` 值为 `count`。 

```java
Sync(int count) {    setState(count);}

```

> countDown

 当线程使用 `countDown()` 方法时，其实使用了`tryReleaseShared`方法以 CAS 的操作来减少 `state`，直至 `state` 为 0  

```java
public void countDown() {    
	sync.releaseShared(1);
}

public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}

```

> await

* 当调用 `await()` 方法的时候，如果 `state` 不为 0，说明任务尚未执行完，`await()` 方法就会一直阻塞， `await()` 方法后的语句不会被执行。 

* 直到`count` 个线程调用了`countDown()`使 state 值被减为 0，或者调用`await()`的线程被中断，该线程才会从阻塞中被唤醒，`await()` 方法之后的语句得到执行。 

```java
public void await() throws InterruptedException {   			
    sync.acquireSharedInterruptibly(1);
}

public final void acquireSharedInterruptibly(int arg)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (tryAcquireShared(arg) < 0)
        doAcquireSharedInterruptibly(arg);
}

```



## CyclicBarrier

 `CyclicBarrier` 是 Java 中的同步辅助类，用于在多个线程之间形成屏障（barrier），当所有线程都达到屏障时，屏障会打开，所有线程才能继续执行 ，CyclicBarrier 是一种线程协作的机制。它的特点是可以循环使用，即一次屏障完成后，可以重新使用。 

与 `CountDownLatch` 不同，`CyclicBarrier` 可以被循环使用，一旦被触发，它将会重置计数器，允许线程再次使用它。 

*  `CountDownLatch` 的实现是基于 AQS 的， 
*  `CycliBarrier` 是基于 `ReentrantLock`(`ReentrantLock` 也属于 AQS 同步器)和 `Condition` 的。 

> 栗子

`CyclicBarrier` 的参与者数量被设定为3。三个线程在执行 `run` 方法时，都会调用 `barrier.await()` 方法等待其他线程到达屏障。当所有线程都到达时，屏障会打开，所有线程会继续执行。`CyclicBarrier` 是循环使用的，当所有线程都通过后，屏障会被重置，可以继续使用。 

```java
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierExample {

    public static void main(String[] args) {
        // 创建一个CyclicBarrier实例，设定屏障的参与者数量为3
        CyclicBarrier barrier = new CyclicBarrier(3);

        // 创建并启动3个线程
        for (int i = 1; i <= 3; i++) {
            Thread thread = new Thread(new Worker(barrier, "Thread-" + i));
            thread.start();
        }
    }

    static class Worker implements Runnable {
        private CyclicBarrier barrier;
        private String name;

        public Worker(CyclicBarrier barrier, String name) {
            this.barrier = barrier;
            this.name = name;
        }

        @Override
        public void run() {
            System.out.println(name + " is doing some work.");

            try {
                // 等待所有线程到达屏障
                barrier.await();
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }

            System.out.println(name + " has completed its work.");
        }
    }
}

```

> 适用场景

 `CyclicBarrier` 主要用于多个线程之间的同步，特别适用于任务分解成多个阶段，每个阶段可以并行执行，但需要等待所有阶段都完成后再继续执行的场景。 

## CyclicBarrier 原理

*  `CyclicBarrier` 内部通过一个 `count` 变量作为计数器
*  `count` 的初始值  为 `parties` 属性的初始化值
*  每当一个线程到了栅栏这里，那么就将计数器减 1
*  如果 count 值为 0 了 ，表示这是这一代最后一个线程到达栅栏，就尝试执行我们构造方法中输入的任务。 

````java
//每次拦截的线程数
private final int parties;
//计数器
private int count;

````

> 构造方法

`CyclicBarrier` 默认的构造方法是 `CyclicBarrier(int parties)`，其参数表示屏障拦截的线程数量，  每个线程调用 `await()` 方法告诉 `CyclicBarrier` 我已经到达了屏障，然后当前线程被阻塞。 

````java
public CyclicBarrier(int parties) {
    this(parties, null);
}

public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}

````

 其中 `parties` 就代表了有拦截的线程的数量，当拦截的线程数量达到这个值的时候就打开栅栏，让所有线程通过。 

> await

当调用 `CyclicBarrier` 对象调用 `await()` 方法时，实际上调用的是 `dowait(false, 0L)`方法。  `await()` 方法就像栅栏一样将线程挡住，当拦住的线程数量达到 `parties` 的值时，栅栏才会打开，线程才得以通过执行。 

```java
public int await() throws InterruptedException, BrokenBarrierException {
  try {
    	return dowait(false, 0L);
  } catch (TimeoutException toe) {
   	 throw new Error(toe); // cannot happen
  }
}

```

 `dowait(false, 0L)`方法源码分析如下： 

````java
// 当线程数量或者请求数量达到 count 时 await 之后的方法才会被执行。上面的示例中 count 的值就为 5。
private int count;
/**
 * Main barrier code, covering the various policies.
 */
private int dowait(boolean timed, long nanos)
    throws InterruptedException, BrokenBarrierException,
           TimeoutException {
    final ReentrantLock lock = this.lock;
    // 锁住
    lock.lock();
    try {
        final Generation g = generation;

        if (g.broken)
            throw new BrokenBarrierException();

        // 如果线程中断了，抛出异常
        if (Thread.interrupted()) {
            breakBarrier();
            throw new InterruptedException();
        }
        // cout减1
        int index = --count;
        // 当 count 数量减为 0 之后说明最后一个线程已经到达栅栏了，也就是达到了可以执行await 方法之后的条件
        if (index == 0) {  // tripped
            boolean ranAction = false;
            try {
                final Runnable command = barrierCommand;
                if (command != null)
                    command.run();
                ranAction = true;
                // 将 count 重置为 parties 属性的初始化值
                // 唤醒之前等待的线程
                // 下一波执行开始
                nextGeneration();
                return 0;
            } finally {
                if (!ranAction)
                    breakBarrier();
            }
        }

        // loop until tripped, broken, interrupted, or timed out
        for (;;) {
            try {
                if (!timed)
                    trip.await();
                else if (nanos > 0L)
                    nanos = trip.awaitNanos(nanos);
            } catch (InterruptedException ie) {
                if (g == generation && ! g.broken) {
                    breakBarrier();
                    throw ie;
                } else {
                    // We're about to finish waiting even if we had not
                    // been interrupted, so this interrupt is deemed to
                    // "belong" to subsequent execution.
                    Thread.currentThread().interrupt();
                }
            }

            if (g.broken)
                throw new BrokenBarrierException();

            if (g != generation)
                return index;

            if (timed && nanos <= 0L) {
                breakBarrier();
                throw new TimeoutException();
            }
        }
    } finally {
        lock.unlock();
    }
}

````

## AQS 源码分析

参考文章：https://cyanzzy.github.io/2023/09/06/JUC-11-AQS/

