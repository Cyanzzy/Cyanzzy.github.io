---
title: Java 并发编程常见面试题总结 Ⅱ
date: 2024-02-28 22:14:41
tags: 
  - Java
categories: 
  - Interview
password: zzy   
message: 仅管理员可见
---

#  volatile 关键字

##  保证变量的可见性

* 在 Java 中，`volatile` 关键字可以保证变量的可见性

* 如果我们将一个变量使用 `volatile` 修饰，这就指示编译器，这个变量是共享且不稳定的，每次使用它都到主存中进行读取。 

* `volatile` 关键字能保证数据的可见性，但**不能保证数据的原子性**。`synchronized` 关键字两者都能保证。 

>  JMM（Java 内存模型）

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-06.png)

> JMM（Java 内存模型）：强制在主存中进行读取 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-07.png)

## 禁止指令重排序

 **在 Java 中，`volatile` 关键字除了可以保证变量的可见性，还有一个重要的作用就是防止 JVM 的指令重排序。** 

 如果我们将变量声明为 `volatile` ，在对这个变量进行读写操作的时候，会通过插入特定的 **内存屏障** 的方式来禁止指令重排序。 

 在 Java 中，`Unsafe` 类提供了三个开箱即用的内存屏障相关的方法，屏蔽了操作系统底层的差异： 

```java
public native void loadFence();
public native void storeFence();
public native void fullFence();
```

 理论上来说，你通过这个三个方法也可以实现和`volatile`禁止重排序一样的效果，只是会麻烦一些。 

> 下面我以一个常见的面试题为例讲解一下 `volatile` 关键字禁止指令重排序的效果。 
>
> 面试中面试官经常会说：“单例模式了解吗？来给我手写一下！给我解释一下双重检验锁方式实现单例模式的原理呗！” 

 **双重校验锁实现对象单例（线程安全）**： 

```java
public class Singleton {

    private volatile static Singleton uniqueInstance;

    private Singleton() {
    }

    public  static Singleton getUniqueInstance() {
       // 先判断对象是否已经实例过，没有实例化过才进入加锁代码
        if (uniqueInstance == null) {
            // 类对象加锁
            synchronized (Singleton.class) {
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}
```

 `uniqueInstance` 采用 `volatile` 关键字修饰也是很有必要的， `uniqueInstance = new Singleton();` 这段代码其实是分为三步执行： 

1. 为 `uniqueInstance` 分配内存空间

2. 初始化 `uniqueInstance`
3. 将 `uniqueInstance` 指向分配的内存地址

 但是由于 JVM 具有指令重排的特性，执行顺序有可能变成 1->3->2。指令重排在单线程环境下不会出现问题，但是在多线程环境下会导致一个线程获得还没有初始化的实例 

 例如，线程 T1 执行了 1 和 3，此时 T2 调用 `getUniqueInstance`() 后  发现 `uniqueInstance` 不为空，因此返回 `uniqueInstance`，但此时 `uniqueInstance` 还未被初始化。 

## volatile 不能保证变量操作的原子性

 **`volatile` 关键字能保证变量的可见性，但不能保证对变量的操作是原子性的。** 

```java
public class VolatoleAtomicityDemo {
    public volatile static int inc = 0;

    public void increase() {
        inc++;
    }

    public static void main(String[] args) throws InterruptedException {
        ExecutorService threadPool = Executors.newFixedThreadPool(5);
        VolatoleAtomicityDemo volatoleAtomicityDemo = new VolatoleAtomicityDemo();
        for (int i = 0; i < 5; i++) {
            threadPool.execute(() -> {
                for (int j = 0; j < 500; j++) {
                    volatoleAtomicityDemo.increase();
                }
            });
        }
        // 等待1.5秒，保证上面程序执行完成
        Thread.sleep(1500);
        System.out.println(inc);
        threadPool.shutdown();
    }
}
```

 正常情况下，运行上面的代码理应输出 `2500`。但你真正运行了上面的代码之后，你会发现每次输出结果都小于 `2500`。 

 如果 `volatile` 能保证 `inc++` 操作的原子性的话。  每个线程中对 `inc` 变量自增完之后，其他线程可以立即看到修改后的值。  5 个线程分别进行了 500 次操作，那么最终 inc 的值应该是 5*500=2500。 

> 很多人会误认为自增操作 `inc++` 是原子性的，实际上，`inc++` 其实是一个复合操作，包括三步： 

1. 读取 inc 的值。
2. 对 inc 加 1。
3. 将 inc 的值写回内存。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-08.png)

> `volatile` 是无法保证这三个操作是具有原子性的，有可能导致下面这种情况出现： 

1. 线程 1 对 `inc` 进行读取操作之后，还未对其进行修改。线程 2 又读取了 `inc`的值并对其进行修改（+1），再将`inc` 的值写回内存。

2. 线程 2 操作完毕后，线程 1 对 `inc`的值进行修改（+1），再将`inc` 的值写回内存。

 这也就导致两个线程分别对 `inc` 进行了一次自增操作后，`inc` 实际上只增加了 1。 

> 其实，如果想要保证上面的代码运行正确也非常简单，利用 `synchronized`、`Lock`或者`AtomicInteger`都可以。 

 使用 `synchronized` 改进： 

```java
public synchronized void increase() {
    inc++;
}
```

 使用 `AtomicInteger` 改进： 

```java
public AtomicInteger inc = new AtomicInteger();

public void increase() {
    inc.getAndIncrement();
}
```

 使用 `ReentrantLock` 改进： 

```java
Lock lock = new ReentrantLock();
public void increase() {
    lock.lock();
    try {
        inc++;
    } finally {
        lock.unlock();
    }
}
```

#  乐观锁和悲观锁

## 悲观锁

悲观锁总是假设最坏的情况，认为共享资源每次被访问的时候就会出现问题（比如共享数据被修改），所以每次在获取资源操作的时候都会上锁，这样其他线程想拿到这个资源就会阻塞直到锁被上一个持有者释放

高并发的场景下，激烈的锁竞争会造成线程阻塞，大量阻塞线程会导致系统的上下文切换，增加系统的性能开销。并且，悲观锁还可能会存在死锁问题，影响代码的正常运行。

* **共享资源每次只给一个线程使用，其它线程阻塞，用完后再把资源转让给其它线程**

* 像 Java 中`synchronized`和`ReentrantLock`等独占锁就是悲观锁思想的实现 

```java
public void performSynchronisedTask() {
    synchronized (this) {
        // 需要同步的操作
    }
}

private Lock lock = new ReentrantLock();
lock.lock();
try {
   // 需要同步的操作
} finally {
    lock.unlock();
}
```

## 乐观锁

乐观锁总是假设最好的情况，认为共享资源每次被访问的时候不会出现问题，线程可以不停地执行，无需加锁也无需等待，只是在提交修改的时候去验证对应的资源（也就是数据）是否被其它线程修改了

高并发的场景下，乐观锁相比悲观锁来说，不存在锁竞争造成线程阻塞，也不会有死锁的问题，在性能上往往会更胜一筹。但是，如果冲突频繁发生（写占比非常多的情况），会频繁失败和重试，这样同样会非常影响性能，导致 CPU 飙升。

 不过，大量失败重试的问题也是可以解决的，像我们前面提到的 `LongAdder`以空间换时间的方式就解决了这个问题。 

> 实现方式

* 使用版本号机制
* 使用 CAS 算法

 在 Java 中`java.util.concurrent.atomic`包下面的原子变量类（比如`AtomicInteger`、`LongAdder`）就是使用了乐观锁的一种实现方式 **CAS** 实现的。 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-09.png)

```java
// LongAdder 在高并发场景下会比 AtomicInteger 和 AtomicLong 的性能更好
// 代价就是会消耗更多的内存空间（空间换时间）
LongAdder sum = new LongAdder();
sum.increment();
```



### 实现乐观锁

 乐观锁一般会使用版本号机制或 CAS 算法实现，CAS 算法相对来说更多一些，这里需要格外注意。 

> **版本号机制**

*  一般是在数据表中加上一个数据版本号 `version` 字段，表示数据被修改的次数。
*  当数据被修改时，`version` 值会加一。
*  当线程 A 要更新数据值时，在读取数据的同时也会读取  `version` 值，在提交更新时，若刚才读取到的 version 值为当前数据库中的  `version` 值相等时才更新，否则重试更新操作，直到更新成功。 

**小栗子**：假设数据库中帐户信息表中有一个 version 字段，当前值为 1 ；而当前帐户余额字段（ balance ）为 100 。

1. 操作员 A 此时将其读出（ version=1 ），并从其帐户余额中扣除 50（ 100-50 ）。
2. 在操作员 A 操作的过程中，操作员 B 也读入此用户信息（ version=1 ），并从其帐户余额中扣除 20 （ ​100-​20 ）。
3. 操作员 A 完成了修改工作，将数据版本号（ version=1 ），连同帐户扣除后余额（ balance=50 ），提交至数据库更新，此时由于提交数据版本等于数据库记录当前版本，数据被更新，数据库记录 version 更新为 2 
4. 操作员 B 完成了操作，也将版本号（ version=1 ）试图向数据库提交数据（ balance=80 ），但此时比对数据库记录版本时发现，操作员 B 提交的数据版本号为 1 ，数据库记录当前版本为 2 ，不满足 “ 提交版本必须等于当前版本才能执行更新 “ 的乐观锁策略，因此，操作员 B 的提交被驳回。

这样就避免了操作员 B 用基于 version=1 的旧数据修改的结果覆盖操作员 A 的操作结果的可能。

> **CAS 算法**

*  CAS 的全称是 **Compare And Swap（比较与交换）**，用于实现乐观锁
*  CAS 就是**用一个预期值和要更新的变量值进行比较，两值相等才会进行更新**

 CAS 是一个原子操作，底层依赖于一条 CPU 的原子指令。  **原子操作** 即最小不可拆分的操作，也就是说操作一旦开始，就不能被打断，直到操作完成 

CAS 涉及到三个操作数：

- **V**：要更新的变量值（Var）
- **E**：预期值（Expected）
- **N**：拟写入的新值（New）

 **当且仅当 V 的值等于 E 时，CAS 通过原子方式用新值 N 来更新 V 的值。如果不等，说明已经有其它线程更新了 V，则当前线程放弃更新。** 

 **当多个线程同时使用 CAS 操作一个变量时，只有一个会胜出，并成功更新，其余均会失败，但失败的线程并不会被挂起，仅是被告知失败，并且允许再次尝试，当然也允许失败的线程放弃操作。** 

**小栗子：**线程 A 要修改变量 i 的值为 6，i 原值为 1（V = 1，E=1，N=6，假设不存在 ABA 问题）。

1. i 与 1 进行比较，如果相等， 则说明没被其他线程修改，可以被设置为 6 。
2. i 与 1 进行比较，如果不相等，则说明被其他线程修改，当前线程放弃更新，CAS 操作失败。

> Java 语言并没有直接实现 CAS，CAS 相关的实现是通过 C++ 内联汇编的形式实现的（JNI 调用）。因此， CAS 的具体实现和操作系统以及 CPU 都有关系。 

 `sun.misc`包下的`Unsafe`类提供了`compareAndSwapObject`、`compareAndSwapInt`、`compareAndSwapLong`方法  来实现的对`Object`、`int`、`long`类型的 CAS 操作 

```java
/**
	*  CAS
  * @param o         包含要修改field的对象
  * @param offset    对象中某field的偏移量
  * @param expected  期望值
  * @param update    更新值
  * @return          true | false
  */
public final native boolean compareAndSwapObject(Object o, long offset,  Object expected, Object update);

public final native boolean compareAndSwapInt(Object o, long offset, int expected,int update);

public final native boolean compareAndSwapLong(Object o, long offset, long expected, long update);
```



### 乐观锁存在问题

> **ABA 问题** 

ABA 问题是在并发编程中常见的一种问题，特别是在无锁算法和数据结构的情境中。 指的是**一个值在并发操作中经历了两次变化（从 A 到 B，然后再回到 A）的场景，这可能导致意外或不可预测的行为。**

以下是 ABA 问题的简要解释：

1. **初始状态（A）：** 假设有一个共享数据结构，其值最初设为 A。
2. **操作 1（A 到 B）：** 线程 1 读取值 A，执行一些操作，并将值更新为 B。
3. **中间状态（B）：** 此时，另一个线程（线程 2）中断了线程 1。
4. **操作 2（B 到 A）：** 线程 2 读取当前值 B，执行一些操作，并将值再次更新为 A。
5. **最终状态（A）：** 线程 1 恢复执行并读取值，此时值又变回了 A。

从线程 1 的角度来看，它可能会错误地认为数据结构没有发生变化，因为它在操作之前和之后都读取到了值 A。然而，在线程 2 的执行期间，发生了中间的变化（从 A 到 B，然后再回到 A）。

为了解决 ABA 问题，采用了各种技术，例如使用原子比较与交换（CAS）操作，并附加时间戳或版本号等机制。这些机制有助于检测在读取和写入操作之间其他线程对数据结构的修改，从而防止意外的后果。

JDK 1.5 以后的 `AtomicStampedReference` 类就是用来解决 ABA 问题的，其中的 `compareAndSet()` 方法就是**首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。** 

```java
public boolean compareAndSet(V   expectedReference,
                             V   newReference,
                             int expectedStamp,
                             int newStamp) {
    Pair<V> current = pair;
    return
        expectedReference == current.reference &&
        expectedStamp == current.stamp &&
        ((newReference == current.reference &&
          newStamp == current.stamp) ||
         casPair(current, Pair.of(newReference, newStamp)));
}
```

> **循环时间长开销大**

 CAS 经常会用到自旋操作来进行重试，也就是不成功就一直循环执行直到成功。如果长时间不成功，会给 CPU 带来非常大的执行开销。 

**如果** JVM 能支持处理器提供的 pause 指令那么效率会有一定的提升，pause 指令有两个作用：

1. 可以延迟流水线执行指令，使 CPU 不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零。
2. 可以避免在退出循环的时候因内存顺序冲而引起 CPU 流水线被清空，从而提高 CPU 的执行效率

> **只能保证一个共享变量的原子操作**

 CAS 只对单个共享变量有效，当操作涉及跨多个共享变量时 CAS 无效。但是从 JDK 1.5 开始，提供了`AtomicReference`类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行 CAS 操作. 

 所以我们可以使用锁或者利用`AtomicReference`类把多个共享变量合并成一个共享变量来操作 

## 双锁总结

* 悲观锁通常多用于写比较多的情况（**多写场景**，竞争激烈），这样可以避免频繁失败和重试影响性能，悲观锁的开销是固定的。不过，如果乐观锁解决了频繁失败和重试这个问题的话（比如`LongAdder`），也是可以考虑使用乐观锁的，要视实际情况而定。

* 乐观锁通常多用于写比较少的情况（**多读场景**，竞争较少），这样可以避免频繁加锁影响性能。不过，乐观锁主要针对的对象是单个共享变量（参考`java.util.concurrent.atomic`包下面的原子变量类）

#  synchronized 关键字

## synchronized

 `synchronized` 主要解决的是多个线程之间访问资源的同步性，可以保证被它修饰的方法或者代码块在任意时刻只能有一个线程执行。 

>  在 Java 早期版本中

- `synchronized` 属于 **重量级锁**，效率低下。这是因为监视器锁（monitor）是依赖于底层的操作系统的 `Mutex Lock` 来实现的， Java 的线程是映射到操作系统的原生线程之上的。
- 如果要挂起或者唤醒一个线程，都需要操作系统帮忙完成，而操作系统实现线程之间的切换时需要从用户态转换到内核态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高。

>  在 Java 6 之后 

 `synchronized` 引入了大量的优化如自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁等技术来减少锁操作的开销，这些优化让 `synchronized` 锁的效率提升了很多 

>【拓展】锁消除、锁粗化 
>
>参考：https://blog.csdn.net/weixin_45325628/article/details/123690289

 锁消除和锁粗化是 Java 虚拟机在运行时对锁进行优化的两种策略。 锁消除和锁粗化都是为了减少锁的开销，提高程序的性能。但需要注意的是，这些优化策略并不是在所有情况下都适用，需要根据具体的代码和场景进行判断和优化。 

**锁消除**：

-  锁消除是指在编译器优化阶段，通过静态分析判断出某些代码块不可能存在竞争条件，因此可以将其中的锁消除掉，从而减少锁的开销
-  这种优化通常发生在方法内部，当编译器确定某个对象不会被其他线程访问时，就会将该对象的锁消除掉。这样可以提高程序的性能。 

**锁粗化：**

- 锁粗化是指将多个连续的加锁、解锁操作合并为一个更大范围的加锁、解锁操作。
- 当 JVM 检测到对同一个对象连续加锁、解锁操作时，会将这些操作合并为一个更大范围的操作，从而减少加锁、解锁的次数，提高程序的性能。
- 这种优化通常发生在循环或迭代操作中，将循环内的加锁、解锁操作移到循环外部。 

 

> 【拓展】偏向锁、轻量级锁
>
> 参考：https://blog.csdn.net/weixin_45325628/article/details/123690289

 偏向锁和轻量级锁是 Java 中用于提高多线程并发性能的锁优化技术。 

**偏向锁：**

- 偏向锁（Biased Locking）是指当一个线程获取对象锁时，如果该对象没有被其他线程竞争，那么该线程会将对象头中的标记位设置为偏向锁，并将线程ID记录在对象头中
- 这样在后续获取该对象锁时，该线程可以直接获得锁，而无需进行任何同步操作
- 偏向锁的目标是减少无竞争情况下的同步操作，提高程序的性能。

**轻量级锁：**

- 轻量级锁（Lightweight Locking）是指当一个线程尝试获取对象锁时，如果该对象当前没有被锁定，该线程会尝试使用 CAS（Compare And Swap）操作来获取锁
- 如果成功，该线程就获得了锁，并将对象头中的标记位设置为轻量级锁
- 如果失败，说明有其他线程竞争了锁，此时该线程会进入自旋状态等待锁的释放或者升级为重量级锁
- 轻量级锁的目标是减少线程间的竞争，提高程序的性能。

## 使用 synchronized

`synchronized` 关键字的使用方式主要有下面 3 种：

1. 修饰实例方法
2. 修饰静态方法
3. 修饰代码块

> **修饰实例方法** （锁当前对象实例） 

 给当前对象实例加锁，进入同步代码前要获得 **当前对象实例的锁** 。 

```java
synchronized void method() {
    // 业务代码
}
```

>  **修饰静态方法** （锁当前类） 

 给当前类加锁，会作用于类的所有对象实例 ，进入同步代码前要获得 **当前 class 的锁**。  这是因为静态成员不属于任何一个实例对象，归整个类所有，不依赖于类的特定实例，被类的所有实例共享。 

```java
synchronized static void method() {
    //业务代码
}
```

 静态 `synchronized` 方法和非静态 `synchronized` 方法之间的调用互斥么？ 

* 不互斥！
* 如果一个线程 A 调用一个实例对象的非静态 `synchronized` 方法 ，而线程 B 需要调用这个实例对象所属类的静态 `synchronized` 方法，是允许的，不会发生互斥现象，
* 因为访问静态 `synchronized` 方法占用的锁是当前类的锁，而访问非静态 `synchronized` 方法占用的锁是当前实例对象锁。  

> **修饰代码块** （锁指定对象/类） 

对括号里指定的对象/类加锁：

- `synchronized(object)` 表示进入同步代码库前要获得 **给定对象的锁**。
- `synchronized(类.class)` 表示进入同步代码前要获得 **给定 Class 的锁**

```java
synchronized(this) {
    //业务代码
}
```

>  **总结：** 

* `synchronized` 关键字加到 `static` 静态方法和 `synchronized(class)` 代码块上都是是给 Class 类上锁；

* `synchronized` 关键字加到实例方法上是给对象实例上锁；

* 尽量不要使用 `synchronized(String str)` ，因为 JVM 中，字符串常量池具有缓存功能

## 构造方法不能使用 synchronized 关键字修饰

Java 语法规定构造方法**不能被 synchronized 关键词修饰**，

1. 在构造函数中，`this` 事实上没有完全构造好，此时不能用作锁。
2. 构造函数中， `this` 还没有被传递出去，事实上不存在并发问题。

对一个对象进行 `new` 的时候，事实上不可能存在多个线程同时 `new` 同一个对象，那么也就是正常情况下，不存在对 `this` 的并发安全性问题，  所以，在构造函数中，对 `this` 加锁，是完全没有意义的。 

 **有个情况例外**，如果你在构造函数中，提前把 `this` 暴露到某个地方，就有可能存在构造函数中也存在并发安全性问题。例如： 

```java
private static Object that;

// 请不要这样做
public Server() {
    that = this;
}
```

这是个非常不好的习惯，我们在实际应用中应该尽量避免这样做。

有些朋友可能会问，那如果在构造函数中传递某个对象，会不会出现并发安全性问题。
答案当然是会。如果要规避这些问题，那么你在构造函数里，不应该是对 `this` 加锁，而是应该**在构造函数中对所涉及到的有并发安全性问题的对象进行加锁**，或者在构造函数之外对这个对象进行加锁（不常用）。

记住：构造函数本身是并发安全的，只是因为额外添加了不安全的参数，导致了构造函数的不安全。

参考文章：https://xhinliang.win/2018/02/backend/constructor-with-sychronized/

## synchronized 底层原理

### synchronized 同步语句块 

```java
public class SynchronizedDemo {
    public void method() {
        synchronized (this) {
            System.out.println("synchronized 代码块");
        }
    }
}
```

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-10.png)

-  `synchronized` 同步语句块的实现使用的是 `monitorenter` 和 `monitorexit` 指令  ，其中 `monitorenter` 指令指向同步代码块的开始位置，`monitorexit` 指令则指明同步代码块的结束位置。 

-  上面的字节码中包含一个 `monitorenter` 指令以及两个 `monitorexit` 指令，这是为了保证锁在同步代码块代码正常执行以及出现异常的这两种情况下都能被正确释放。 

**当执行 `monitorenter` 指令时，线程试图获取锁也就是获取 对象监视器 `monitor` 的持有权。** 

> 在 Java 虚拟机(HotSpot)中，Monitor 是基于 C++实现的，由[ObjectMonitor](https://github.com/openjdk-mirror/jdk7u-hotspot/blob/50bdefc3afe944ca74c3093e7448d6b889cd20d1/src/share/vm/runtime/objectMonitor.cpp)实现的。  每个对象中都内置了一个 `ObjectMonitor`对象。 
>
> 另外，`wait/notify`等方法也依赖于`monitor`对象，  这就是为什么只有在同步的块或者方法中才能调用`wait/notify`等方法，否则会抛出`java.lang.IllegalMonitorStateException`的异常的原因。 

 **在执行`monitorenter`时，会尝试获取对象的锁，如果锁的计数器为 0 则表示锁可以被获取，获取后将锁计数器设为 1 也就是加 1。** 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-11.png)

 **对象锁的的拥有者线程才可以执行 `monitorexit` 指令来释放锁。在执行 `monitorexit` 指令后，将锁计数器设为 0，表明锁被释放，其他线程可以尝试获取锁。** 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-12.png)

 **如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止。** 

### synchronized 修饰方法 

```java
public class SynchronizedDemo2 {
    public synchronized void method() {
        System.out.println("synchronized 方法");
    }
}
```

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-13.png)

- `synchronized` 修饰的方法并没有 `monitorenter` 指令和 `monitorexit` 指令
- 取得代之的是 `ACC_SYNCHRONIZED` 标识，  该标识指明该方法是一个同步方法
- JVM 通过该 `ACC_SYNCHRONIZED` 访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用。 

- 如果是实例方法，JVM 会尝试获取实例对象的锁；如果是静态方法，JVM 会尝试获取当前 class 的锁。 

### 总结

*  `synchronized` 同步语句块的实现使用的是 `monitorenter` 和 `monitorexit` 指令，  其中 `monitorenter` 指令指向同步代码块的开始位置，`monitorexit` 指令则指明同步代码块的结束位置。 
*  `synchronized` 修饰的方法并没有 `monitorenter` 指令和 `monitorexit` 指令 ，取得代之的确实是 `ACC_SYNCHRONIZED` 标识，该标识指明该方法是一个同步方法。 

 **不过两者的本质都是对对象监视器 monitor 的获取。** 

[有赞技术--Java锁与线程的那些事 ](https://tech.youzan.com/javasuo-yu-xian-cheng-de-na-xie-shi/)

> 进阶一下：学有余力的小伙伴可以抽时间详细研究一下对象监视器 `monitor` 

## JDK 1.6 后的 synchronized 底层优化

JDK1.6 对锁的实现引入了大量的优化，如偏向锁、轻量级锁、自旋锁、适应性自旋锁、锁消除、锁粗化等技术来减少锁操作的开销。

锁主要存在四种状态，依次是：无锁状态、偏向锁状态、轻量级锁状态、重量级锁状态，他们会随着竞争的激烈而逐渐升级。注意锁可以升级不可降级，这种策略是为了提高获得锁和释放锁的效率

 **`synchronized` 锁升级是一个比较复杂的过程** 

> 参考文章

- [【多线程】锁消除、锁粗化、偏向锁、自旋锁、自适应字段锁、轻量级锁、重量级锁](https://blog.csdn.net/weixin_45325628/article/details/123690289)

- [Java6 及以上版本对 synchronized 的优化](https://www.cnblogs.com/wuqinglong/p/9945618.html) 

- [Synchronized与锁升级](https://cyanzzy.github.io/2023/09/06/JUC-10-Synchronized%E4%B8%8E%E9%94%81%E5%8D%87%E7%BA%A7/#%E9%87%8D%E9%94%81)

## synchronized 和 volatile 区别

 `synchronized` 关键字和 `volatile` 关键字是**两个互补的存在**，而不是对立的存在！ 

*  `volatile` 关键字是线程同步的轻量级实现，所以 `volatile`性能肯定比`synchronized`关键字要好 。  但是 `volatile` 关键字只能用于变量而 `synchronized` 关键字可以修饰方法以及代码块 。 

- `volatile` 关键字能保证数据的可见性，但不能保证数据的原子性。`synchronized` 关键字两者都能保证。

* `volatile`关键字主要用于解决变量在多个线程之间的可见性，而 `synchronized` 关键字解决的是多个线程之间访问资源的同步性 

#  ReentrantLock

## ReentrantLock 是什么？

```java
public class ReentrantLock implements Lock, java.io.Serializable {}

```

-  `ReentrantLock` 实现了 `Lock` 接口，是一个可重入且独占式的锁，  和 `synchronized` 关键字类似
-  `ReentrantLock` 更灵活、更强大，增加了轮询、超时、中断、公平锁和非公平锁等高级功能。 

-  `ReentrantLock` 里面有一个内部类 `Sync`，`Sync` 继承 AQS（`AbstractQueuedSynchronizer`），添加锁和释放锁的大部分操作实际上都是在 `Sync` 中实现的。`Sync` 有公平锁 `FairSync` 和非公平锁 `NonfairSync` 两个子类。 
-  `ReentrantLock` 默认使用非公平锁，也可以通过构造器来显式的指定使用公平锁。 

```java
// 传入一个 boolean 值，true 时为公平锁，false 时为非公平锁
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}

```

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-14.png)



## 公平锁和非公平锁

**公平锁** ：

- **锁被释放之后，先申请的线程先得到锁。**
- 性能较差一些，因为公平锁为了保证时间上的绝对顺序，上下文切换更频繁。

**非公平锁**：

- 锁被释放之后，后申请的线程可能会先获取到锁，是随机或者按照其他优先级排序的。
- 性能更好，但可能会导致某些线程永远无法获取到锁。

## synchronized 和 ReentrantLock 区别

> 共同点

 **都用于实现线程的同步和互斥：** 

-  `synchronized` 和 `ReentrantLock` 都用于确保多个线程之间的互斥访问，防止数据不一致或冲突。 

**都支持可重入：**

* **可重入锁** 也叫递归锁，指的是线程可以再次获取自己的内部锁。
* 比如一个线程获得了某个对象的锁，此时这个对象锁还没有释放，当其再次想要获取这个对象的锁的时候还是可以获取的，如果是不可重入锁的话，就会造成死锁。

* JDK 提供的所有现成的 `Lock` 实现类，包括 `synchronized` 关键字锁都是可重入的。 

 在下面的代码中，`method1()` 和 `method2()`都被 `synchronized` 关键字修饰，`method1()`调用了`method2()`。 

```java
public class SynchronizedDemo {
    public synchronized void method1() {
        System.out.println("方法1");
        method2();
    }

    public synchronized void method2() {
        System.out.println("方法2");
    }
}

```

 由于 `synchronized`锁是可重入的，同一个线程在调用`method1()` 时可以直接获得当前对象的锁，执行 `method2()` 的时候可以再次获取这个对象的锁，不会产生死锁问题。

假如 `synchronized`是不可重入锁的话，由于该对象的锁已被当前线程所持有且无法释放，这就导致线程在执行 `method2()`时获取锁失败，会出现死锁问题。 

> 区别

 **机制和语法：** 

-  `synchronized` 是 Java 中的关键字，通过对方法或代码块进行修饰，实现对共享资源的同步。 
-  `ReentrantLock` 是 Java 中的类，属于 `java.util.concurrent.locks` 包，提供了显式的锁对象，通过 `lock()` 和 `unlock()` 方法进行同步控制。 

 **锁的获取和释放方式：** 

-  `synchronized` 的锁是隐式获取的，当线程进入同步代码块或方法时，自动获取锁，退出时自动释放锁。 
-  `ReentrantLock` 的锁是显式获取和释放的，需要在代码中明确地调用 `lock()` 和 `unlock()` 方法。 

 **可中断性：** 

- `synchronized` 不支持线程的中断操作，即使一个线程在等待锁时被中断，它仍然会一直等待获取锁。
- `ReentrantLock` 支持线程的中断操作，当一个线程在等待锁的过程中可以被中断。  

 **公平性：** 

-  `synchronized` 是非公平锁，不考虑等待时间，线程竞争时，不保证先等待的线程先获得锁。 
-  `ReentrantLock` 可以是公平锁（按照线程等待时间来分配锁）或非公平锁，通过构造函数的参数进行设置 

**适用场景：**

-  在一般情况下，推荐使用 `synchronized`，因为它简单、易用，并且在性能上进行了优化。 
-  如果需要更多的灵活性，比如可中断锁、公平锁、超时等，可以考虑使用 `ReentrantLock`。 



> **synchronized 依赖于 JVM 而 ReentrantLock 依赖于 API**

* `synchronized` 是依赖于 JVM 实现的，在 JDK1.6 为   `synchronized` 关键字进行了很多优化，但是这些优化都是在虚拟机层面实现的，并没有直接暴露给我们。

* `ReentrantLock` 是 JDK 层面实现的（也就是 API 层面，需要 lock() 和 unlock() 方法配合 try/finally 语句块来完成） 

> ReentrantLock 比 synchronized 增加了一些高级功能

**等待可中断** ：

- `ReentrantLock`提供了一种能够中断等待锁的线程的机制
- 通过 `lock.lockInterruptibly()` 来实现这个机制，即正在等待的线程可以选择放弃等待，改为处理其他事情。

**可实现公平锁** ：

- `ReentrantLock`可以指定是公平锁还是非公平锁。而`synchronized`只能是非公平锁。
- 公平锁就是先等待的线程先获得锁。
- `ReentrantLock`默认情况是非公平的，可以通过 `ReentrantLock`类的`ReentrantLock(boolean fair)`构造方法来指定是否是公平的。

**可实现选择性通知（锁可以绑定多个条件）**：

-  `synchronized`关键字与`wait()`和`notify()`/`notifyAll()`方法相结合可以实现等待/通知机制
-  `ReentrantLock`类当然也可以实现，但是需要借助于`Condition`接口与`newCondition()`方法。

>  【拓展】 `Condition`接口 

- `Condition` 接口是在 `java.util.concurrent.locks` 包下引入的，用于在锁上等待的条件。它通常与 `ReentrantLock` 结合使用，提供了更灵活的线程同步和协调机制。`Condition` 接口允许线程在等待某个条件满足时暂停执行，然后在条件满足时被唤醒。 

* 比如可以实现多路通知功能也就是在一个`Lock`对象中可以创建多个`Condition`实例（即对象监视器）， 线程对象可以注册在指定的`Condition`中，从而可以有选择性的进行线程通知，在调度线程上更加灵活。
* 在使用`notify()/notifyAll()`方法进行通知时，被通知的线程是由 JVM 选择的，用`ReentrantLock`类结合`Condition`实例可以实现“选择性通知” ，而且是 `Condition` 接口默认提供的。 
* 而`synchronized`关键字就相当于整个 `Lock` 对象中只有一个`Condition`实例，所有的线程都注册在它一个身上。  如果执行`notifyAll()`方法的话就会通知所有处于等待状态的线程，这样会造成很大的效率问题。而`Condition`实例的 `signalAll()`方法，只会唤醒注册在该`Condition`实例中的所有等待线程。 

## 可中断锁和不可中断锁区别

* **可中断锁**：获取锁的过程中可以被中断，不需要一直等到获取锁之后才能进行其他逻辑处理。`ReentrantLock` 就属于是可中断锁。
* **不可中断锁**：一旦线程申请了锁，就只能等到拿到锁以后才能进行其他的逻辑处理。 `synchronized` 就属于是不可中断锁。

 `ReentrantLock` 可以通过 `lockInterruptibly()` 方法实现中断响应，该方法会在等待锁的过程中响应中断，即在等待锁的线程上调用 `interrupt()` 方法时，`lockInterruptibly()` 方法会抛出 `InterruptedException`。 

 下面是一个简单的示例，演示了如何使用 `ReentrantLock` 进行中断响应： 

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class InterruptibleLockExample {
    private Lock lock = new ReentrantLock();

    public void performTask() {
        try {
            // 尝试获取锁，响应中断
            lock.lockInterruptibly();
            try {
                // 执行需要同步的任务
                System.out.println(Thread.currentThread().getName() + " is performing the task");
                Thread.sleep(5000); // 模拟任务执行时间
            } finally {
                lock.unlock(); // 释放锁
            }
        } catch (InterruptedException e) {
            System.out.println(Thread.currentThread().getName() + " was interrupted while waiting for the lock");
            // 可以根据实际需求处理中断异常
        }
    }

    public static void main(String[] args) {
        InterruptibleLockExample example = new InterruptibleLockExample();

        // 创建线程1并启动
        Thread thread1 = new Thread(() -> {
            example.performTask();
        });

        thread1.start();

        // 创建线程2并启动，一定时间后中断线程1
        Thread thread2 = new Thread(() -> {
            try {
                Thread.sleep(1000); // 等待一定时间
                thread1.interrupt(); // 中断线程1
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        thread2.start();
    }
}

```

在上述示例中，线程1在执行任务时，使用了 `lockInterruptibly()` 方法获取锁，当线程2在等待一定时间后中断线程1时，线程1会在获取锁的过程中抛出 `InterruptedException`，并在 `catch` 语句块中处理中断异常。

请注意，`lockInterruptibly()` 方法并不是 `ReentrantLock` 类的唯一方法，`lock()` 方法也可以用于获取锁，但不会响应中断。要使 `lock()` 方法响应中断，可以在获取锁后手动检查中断状态，或者使用 `lockInterruptibly()` 方法。

#  ReentrantReadWriteLock

## ReentrantReadWriteLock 是什么？

 `ReentrantReadWriteLock` 实现了 `ReadWriteLock` ，是一个**可重入的读写锁**，既可以保证多个线程同时读的效率，同时又可以保证有写入操作时的线程安全。 

```java
public class ReentrantReadWriteLock
        implements ReadWriteLock, java.io.Serializable{
}
public interface ReadWriteLock {
    Lock readLock();
    Lock writeLock();
}

```

- 一般锁进行并发控制的规则：读读互斥、读写互斥、写写互斥。
- 读写锁进行并发控制的规则：读读不互斥、读写互斥、写写互斥（只有读读不互斥）

`ReentrantReadWriteLock` 其实是两把锁，一把是  (写锁)，一把是 （读锁） 。读锁是共享锁，写锁是独占锁。读锁可以被同时读，可以同时被多个线程持有，而写锁最多只能同时被一个线程持有。

和 `ReentrantLock` 一样，`ReentrantReadWriteLock` 底层也是基于 AQS 实现的。 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-15.png)

 `ReentrantReadWriteLock` 也支持公平锁和非公平锁，默认使用非公平锁，可以通过构造器来显示的指定。 

```java
// 传入一个 boolean 值，true 时为公平锁，false 时为非公平锁
public ReentrantReadWriteLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
    readerLock = new ReadLock(this);
    writerLock = new WriteLock(this);
}
```

## ReentrantReadWriteLock 适合场景

* 由于 `ReentrantReadWriteLock` 既可以保证多个线程同时读的效率，同时又可以保证有写入操作时的线程安全。
* 因此，在**读多写少**的情况下，使用  `ReentrantReadWriteLock` 能够明显提升系统性能。 



## 共享锁和独占锁

- **共享锁**：一把锁可以被多个线程同时获得。
- **独占锁**：一把锁只能被一个线程获得。

## 线程持有读锁还能获取写锁吗？

* 在线程持有读锁的情况下，该线程不能取得写锁。因为获取写锁的时候，如果发现当前的读锁被占用，就马上获取失败，不管读锁是不是被当前线程持有 

* 在线程持有写锁的情况下，该线程可以继续获取读锁。获取读锁时如果发现写锁被占用，只有写锁没有被当前线程占用的情况才会获取失败

##  读锁为什么不能升级为写锁？

 写锁可以降级为读锁，但是读锁却不能升级为写锁。

* 这是因为**读锁升级为写锁会引起线程的争夺**，毕竟写锁属于是独占锁，这样的话，会影响性能。 

* 另外，还可能会有**死锁问题**发生。（举个例子：假设两个线程的读锁都想升级写锁，则需要对方都释放自己锁，而双方都不释放，就会产生死锁。 ）

#  StampedLock

## StampedLock 是什么？

 `StampedLock` 是 JDK 1.8 引入的性能更好的**读写锁**，**不可重入且不支持条件变量** `Conditon`。 

 不同于一般的 `Lock` 类，`StampedLock` 并不是直接实现 `Lock`或 `ReadWriteLock`接口，而是基于 **CLH 锁** 独立实现的（AQS 也是基于这玩意）。 

```java
public class StampedLock implements java.io.Serializable {
}
```

> `StampedLock` 提供了三种模式的读写控制模式：读锁、写锁和乐观读。 

* **写锁**：独占锁，一把锁只能被一个线程获得。当一个线程获取写锁后，其他请求读锁和写锁的线程必须等待。类似于 `ReentrantReadWriteLock` 的写锁，不过这里的写锁是**不可重入**的。

* **读锁** （悲观读）：共享锁，没有线程获取写锁的情况下，多个线程可以同时持有读锁。如果己经有线程持有写锁，则其他线程请求获取该读锁会被阻塞。类似于 `ReentrantReadWriteLock` 的读锁，不过这里的读锁是**不可重入**的。

* **乐观读**：允许多个线程获取乐观读以及读锁。同时允许一个写线程获取写锁。

> 另外，`StampedLock` 还支持这三种锁**在一定条件下进行相互转换 。** 

```java
long tryConvertToWriteLock(long stamp){}
long tryConvertToReadLock(long stamp){}
long tryConvertToOptimisticRead(long stamp){}
```

> `StampedLock` 在获取锁的时候会返回一个 long 型的数据戳，该数据戳用于稍后的锁释放参数，如果返回的数据戳为 0 则表示锁获取失败。  **当前线程持有了锁再次获取锁还是会返回一个新的数据戳，这也是`StampedLock`不可重入的原因。** 

```java
// 写锁
public long writeLock() {
    long s, next;  // bypass acquireWrite in fully unlocked case only
    return ((((s = state) & ABITS) == 0L &&
             U.compareAndSwapLong(this, STATE, s, next = s + WBIT)) ?
            next : acquireWrite(false, 0L));
}
// 读锁
public long readLock() {
    long s = state, next;  // bypass acquireRead on common uncontended case
    return ((whead == wtail && (s & ABITS) < RFULL &&
             U.compareAndSwapLong(this, STATE, s, next = s + RUNIT)) ?
            next : acquireRead(false, 0L));
}
// 乐观读
public long tryOptimisticRead() {
    long s;
    return (((s = state) & WBIT) == 0L) ? (s & SBITS) : 0L;
}
```

## StampedLock 的性能为什么更好？

*  相比于传统读写锁多出来的**乐观读**是`StampedLock`比 `ReadWriteLock` 性能更好的关键原因 
*  `StampedLock` 的**乐观读**允许一个写线程获取写锁，所以不会导致所有写线程阻塞
*  也就是当读多写少的时候，写线程有机会获取写锁，减少了线程饥饿的问题，吞吐量大大提高。

## StampedLock 适合场景

*  和 `ReentrantReadWriteLock` 一样，`StampedLock` 同样**适合读多写少**的业务场景，可以作为 `ReentrantReadWriteLock`的替代品，性能更好。 

*  不过，需要注意的是`StampedLock`**不可重入，不支持条件变量** `Conditon`，对中断操作支持也不友好（使用不当容易导致 CPU 飙升）。如果你需要用到   `ReentrantLock` 的一些高级性能，就不太建议使用 `StampedLock` 了。 

 另外，`StampedLock` 性能虽好，但使用起来相对比较麻烦，一旦使用不当，就会出现生产问题。强烈建议你  在使用`StampedLock` 之前，看看 [StampedLock 官方文档中的案例](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/StampedLock.html)

### StampedLock 的底层原理 

*  `StampedLock` 不是直接实现 `Lock`或 `ReadWriteLock`接口，而是基于 **CLH 锁** 实现的（AQS 也是基于这玩意）
*  CLH 锁是对自旋锁的一种改良，是一种隐式的链表队列
*  `StampedLock` 通过 CLH 队列进行线程的管理，通过同步状态值 `state` 来表示锁的状态和类型。 

> `StampedLock` 的原理和 AQS 原理比较类似，这里就不详细介绍了，感兴趣的可以看看下面这两篇文章：

- [AQS 详解](https://javaguide.cn/java/concurrent/aqs.html)
- [StampedLock 底层原理分析](https://segmentfault.com/a/1190000015808032)

如果你只是准备面试的话，建议多花点精力搞懂 AQS 原理即可，`StampedLock` 底层原理在面试中遇到的概率非常小。

# 聊聊 Java 的几把 JVM 级锁

##  synchronized

 在 JDK1.6 之前， syncronized 是一把重量级的锁，不过随着 JDK 的升级，也在对它进行不断的优化，如今它变得不那么重了，甚至在某些场景下，它的性能反而优于轻量级锁。在加了 syncronized 关键字的方法、代码块中，一次只允许一个线程进入特定代码段，从而避免多线程同时修改同一数据。 

> 有锁升级过程

- 在 JDK1.5 (含)之前， synchronized 的底层实现是重量级的，所以之前一致称呼它为"重量级锁" 

- 在 JDK1.5 之后，对 synchronized 进行了各种优化，它变得不那么重了，实现原理就是锁升级的过程。 

**Java 对象内存布局：** 

在创建一个对象后，在 JVM 虚拟机( HotSpot )中，对象在 Java 内存中的存储布局 可分为三块 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-16.png)

 **对象头区域此处存储的信息包括两部分：** 

-  对象自身的运行时数据( MarkWord ) ： 存储 hashCode、GC 分代年龄、锁类型标记、偏向锁线程 ID 、 CAS 锁指向线程 LockRecord 的指针等， synconized 锁的机制与这个部分( markwork )密切相关，用 markword 中最低的三位代表锁的状态，其中一位是偏向锁位，另外两位是普通锁位。 

-  对象类型指针( Class Pointer ) ： 对象指向它的类元数据的指针、 JVM 就是通过它来确定是哪个 Class 的实例。 

 **实例数据区域**  

 此处存储的是对象真正有效的信息，比如对象中所有字段的内容 

 **对齐填充区域** 

  JVM 的实现 HostSpot 规定对象的起始地址必须是 8 字节的整数倍 。现在 64 位的 OS 往外读取数据的时候一次性读取 64bit 整数倍的数据，也就是 8 个字节，所以 HotSpot 为了高效读取对象，就做了"对齐"，  如果一个对象实际占的内存大小不是 8byte 的整数倍时，就"补位"到 8byte 的整数倍。所以对齐填充区域的大小不是固定的。 

 当线程进入到 synchronized 处尝试获取该锁时， synchronized 锁升级流程如下： 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-17.png)



> 如上图所示， **synchronized 锁升级的顺序为：偏向锁->轻量级锁->重量级锁**，每一步触发锁升级的情况如下： 

**偏向锁**

在 JDK1.8 中，其实默认是轻量级锁，但如果设定了 `-XX:BiasedLockingStartupDelay = 0` ，那在对一个 Object 做 syncronized 的时候，会立即上一把偏向锁。当处于偏向锁状态时， markwork 会记录当前线程 ID 。



 **升级到轻量级锁** 

-  当下一个线程参与到偏向锁竞争时，会先判断 markword 中保存的线程 ID 是否与这个线程 ID 相等，如果不相等，会立即撤销偏向锁，升级为轻量级锁。 
-  每个线程在自己的线程栈中生成一个 LockRecord ( LR )，然后每个线程通过 CAS (自旋)的操作将锁对象头中的 markwork 设置为指向自己的 LR 的指针，哪个线程设置成功，就意味着获得锁。 

 关于 synchronized 中此时执行的 CAS 操作是通过 native 的调用 HotSpot 中 bytecodeInterpreter.cpp 文件 C++ 代码实现的，有兴趣的可以继续深挖。 

 **升级到重量级锁** 

-  如果锁竞争加剧(如线程自旋次数或者自旋的线程数超过某阈值， JDK1.6 之后，由 JVM 自己控制该规则)，就会升级为重量级锁。 
-  此时就会向操作系统申请资源，线程挂起，进入到操作系统内核态的等待队列中，等待操作系统调度，然后映射回用户态。 
-  在重量级锁中，由于需要做内核态到用户态的转换，而这个过程中需要消耗较多时间，也就是"重"的原因之一。 

> **可重入** 

- synchronized 拥有强制原子性的内部锁机制，是一把可重入锁。 因此，在一个线程使用 synchronized 方法时调用该对象另一个 synchronized 方法，即一个线程得到一个对象锁后再次请求该对象锁，是永远可以拿到锁的。 
- 在 Java 中线程获得对象锁的操作是以线程为单位的，而不是以调用为单位的。 
- synchronized 锁的对象头的 markwork 中会记录该锁的线程持有者和计数器，当一个线程请求成功后， JVM 会记下持有锁的线程，并将计数器计为1。此时其他线程请求该锁，则必须等待。而该持有锁的线程如果再次请求这个锁，就可以再次拿到这个锁，同时计数器会递增。当线程退出一个 synchronized 方法/块时，计数器会递减，如果计数器为 0 则释放该锁锁。 

>  **悲观锁(互斥锁、排他锁)** 

 synchronized 是一把悲观锁(独占锁)，当前线程如果获取到锁，会导致其它所有需要锁该的线程等待，一直等待持有锁的线程释放锁才继续进行锁的争抢。 

##  ReentrantLock

-  ReentrantLock 从字面可以看出是一把可重入锁，这点和 synchronized 一样，但实现原理也与 syncronized 有很大差别，它是基于经典的 AQS(AbstractQueueSyncronized) 实现的 
-  AQS 是基于 volitale 和 CAS 实现的，其中 AQS 中维护一个 valitale 类型的变量 state 来做一个可重入锁的重入次数，加锁和释放锁也是围绕这个变量来进行的。  
-  ReentrantLock 也提供了一些 synchronized 没有的特点，因此比 synchronized 好用。 

>  **AQS模型如下图：** 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-18.png)



> **可重入** 

-  ReentrantLock 和 syncronized 关键字一样，都是可重入锁 
-  RetrantLock 利用 AQS 的的 state 状态来判断资源是否已锁
-  同一线程重入加锁， state 的状态 +1
-  同一线程重入解锁, state 状态 -1 (解锁必须为当前独占线程，否则异常)
-  当 state 为 0 时解锁成功。 

> **需要手动加锁、解锁** 

-  synchronized 关键字是自动进行加锁、解锁的
-  ReentrantLock 需要 lock() 和 unlock() 方法配合 try/finally 语句块来完成，来手动加锁、解锁。 

>  **支持设置锁的超时时间** 

- synchronized 关键字无法设置锁的超时时间，如果一个获得锁的线程内部发生死锁，那么其他线程就会一直进入阻塞状态
- ReentrantLock 提供 tryLock 方法，允许设置线程获取锁的超时时间，如果超时，则跳过，不进行任何操作，避免死锁的发生

> **支持公平/非公平锁** 

-  synchronized 关键字是一种非公平锁，先抢到锁的线程先执行。 
-  ReentrantLock 的构造方法中允许设置 true/false 来实现公平、非公平锁
-  如果设置为 true ，则线程获取锁要遵循"先来后到"的规则，每次都会构造一个线程 Node ，然后到双向链表的"尾巴"后面排队，等待前面的 Node 释放锁资源。 

>  **可中断锁** 

-   ReentrantLock 中的 lockInterruptibly() 方法使得线程可以在被阻塞时响应中断 
-   比如一个线程 t1 通过 lockInterruptibly() 方法获取到一个可重入锁，并执行一个长时间的任务，另一个线程通过 interrupt() 方法就可以立刻打断 t1 线程的执行，来获取t1持有的那个可重入锁。 
-   而通过 ReentrantLock 的 lock() 方法或者 Synchronized 持有锁的线程是不会响应其他线程的 interrupt() 方法的，直到该方法主动释放锁之后才会响应 interrupt() 方法。 

##  ReentrantReadWriteLock

-  ReentrantReadWriteLock (读写锁)其实是两把锁，一把是 WriteLock (写锁)，一把是读锁， ReadLock 
-  读写锁的规则是：读读不互斥、读写互斥、写写互斥 

-  在一些实际的场景中，读操作的频率远远高于写操作，**如果直接用一般的锁进行并发控制的话，就会读读互斥、读写互斥、写写互斥，效率低下**，读写锁的产生就是为了优化这种场景的操作效率。 
-  一般情况下独占锁的效率低来源于高并发下对临界区的激烈竞争导致线程上下文切换。因此当并发不是很高的情况下，读写锁由于需要额外维护读锁的状态，可能还不如独占锁的效率高，因此需要根据实际情况选择使用。  

 ReentrantReadWriteLock 的原理也是基于 AQS 进行实现的，  与 ReentrantLock 的差别在于 ReentrantReadWriteLock 锁拥有共享锁、排他锁属性。读写锁中的加锁、释放锁也是基于 Sync (继承于 AQS )，并且主要使用 AQS 中的 state 和 node 中的 waitState 变量进行实现的。

  实现读写锁与实现普通互斥锁的主要区别在于需要分别记录读锁状态及写锁状态，并且等待队列中需要区别处理两种加锁操作。

 ReentrantReadWriteLock 中将 AQS 中的 int 类型的 state 分为高 16 位与第 16 位分别记录读锁和写锁的状态，如下图所示： 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-19.png)

>  **WriteLock(写锁)是悲观锁(排他锁、互斥锁)** 

 通过计算 state&((1<<16)-1) ，将 state 的高 16 位全部抹去，因此 state 的低位记录着写锁的重入计数。 

 获取写锁源码： 

```java
/**
     * 获取写锁
       Acquires the write lock.
     *  如果此时没有任何线程持有写锁或者读锁，那么当前线程执行CAS操作更新status，
     *  若更新成功，则设置读锁重入次数为1，并立即返回
     * <p>Acquires the write lock if neither the read nor write lock
     * are held by another thread
     * and returns immediately, setting the write lock hold count to
     * one.
     *  如果当前线程已经持有该写锁，那么将写锁持有次数设置为1，并立即返回
     * <p>If the current thread already holds the write lock then the
     * hold count is incremented by one and the method returns
     * immediately.
     *  如果该锁已经被另外一个线程持有，那么停止该线程的CPU调度并进入休眠状态，
     *  直到该写锁被释放，且成功将写锁持有次数设置为1才表示获取写锁成功
     * <p>If the lock is held by another thread then the current
     * thread becomes disabled for thread scheduling purposes and
     * lies dormant until the write lock has been acquired, at which
     * time the write lock hold count is set to one.
     */
    public void lock() {
        sync.acquire(1);
    }
/**
 * 该方法为以独占模式获取锁，忽略中断
 * 如果调用一次该“tryAcquire”方法更新status成功，则直接返回，代表抢锁成功
 * 否则，将会进入同步队列等待，不断执行“tryAcquire”方法尝试CAS更新status状态，直到成功抢到锁
 * 其中“tryAcquire”方法在NonfairSync(公平锁)中和FairSync(非公平锁)中都有各自的实现
 *
 * Acquires in exclusive mode, ignoring interrupts.  Implemented
 * by invoking at least once {@link #tryAcquire},
 * returning on success.  Otherwise the thread is queued, possibly
 * repeatedly blocking and unblocking, invoking {@link
 * #tryAcquire} until success.  This method can be used
 * to implement method {@link Lock#lock}.
 *
 * @param arg the acquire argument.  This value is conveyed to
 *        {@link #tryAcquire} but is otherwise uninterpreted and
 *        can represent anything you like.
 */
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
protected final boolean tryAcquire(int acquires) {
        /*
         * Walkthrough:
         * 1、如果读写锁的计数不为0，且持有锁的线程不是当前线程，则返回false
         * 1. If read count nonzero or write count nonzero
         *    and owner is a different thread, fail.
         * 2、如果持有锁的计数不为0且计数总数超过限定的最大值，也返回false
         * 2. If count would saturate, fail. (This can only
         *    happen if count is already nonzero.)
         * 3、如果该锁是可重入或该线程在队列中的策略是允许它尝试抢锁，那么该线程就能获取锁
         * 3. Otherwise, this thread is eligible for lock if
         *    it is either a reentrant acquire or
         *    queue policy allows it. If so, update state
         *    and set owner.
         */
        Thread current = Thread.currentThread();
        //获取读写锁的状态
        int c = getState();
        //获取该写锁重入的次数
        int w = exclusiveCount(c);
        //如果读写锁状态不为0，说明已经有其他线程获取了读锁或写锁
        if (c != 0) {
            //如果写锁重入次数为0，说明有线程获取到读锁，根据“读写锁互斥”原则，返回false
            //或者如果写锁重入次数不为0，且获取写锁的线程不是当前线程，根据"写锁独占"原则，返回false
            // (Note: if c != 0 and w == 0 then shared count != 0)
            if (w == 0 || current != getExclusiveOwnerThread())
                return false;
           //如果写锁可重入次数超过最大次数（65535），则抛异常
            if (w + exclusiveCount(acquires) > MAX_COUNT)
                throw new Error("Maximum lock count exceeded");
            //到这里说明该线程是重入写锁，更新重入写锁的计数(+1)，返回true
            // Reentrant acquire
            setState(c + acquires);
            return true;
        }
        //如果读写锁状态为0,说明读锁和写锁都没有被获取，会走下面两个分支：
        //如果要阻塞或者执行CAS操作更新读写锁的状态失败，则返回false
        //如果不需要阻塞且CAS操作成功，则当前线程成功拿到锁，设置锁的owner为当前线程，返回true
        if (writerShouldBlock() ||
            !compareAndSetState(c, c + acquires))
            return false;
        setExclusiveOwnerThread(current);
        return true;
    }

```

 释放写锁源码： 

```java
/*
* Note that tryRelease and tryAcquire can be called by
* Conditions. So it is possible that their arguments contain
* both read and write holds that are all released during a
* condition wait and re-established in tryAcquire.
*/
protected final boolean tryRelease(int releases) {
    //若锁的持有者不是当前线程，抛出异常
    if (!isHeldExclusively())
     throw new IllegalMonitorStateException();
    //写锁的可重入计数减掉releases个
    int nextc = getState() - releases;
    //如果写锁重入计数为0了，则说明写锁被释放了
    boolean free = exclusiveCount(nextc) == 0;
    if (free)
    //若写锁被释放，则将锁的持有者设置为null，进行GC
    setExclusiveOwnerThread(null);
    //更新写锁的重入计数
    setState(nextc);
    return free;
}

```

>  **ReadLock(读锁)是共享锁(乐观锁)** 

通过计算 state>>>16 进行无符号补 0 ，右移 16 位，因此 state 的高位记录着写锁的重入计数. 

读锁获取锁的过程比写锁稍微复杂些：

1. 首先判断写锁是否为 0 并且当前线程不占有独占锁，直接返回； 

2. 否则，判断读线程是否需要被阻塞并且读锁数量是否小于最大值并且比较设置状态成功
   - 若当前没有读锁，则设置第一个读线程 firstReader 和 firstReaderHoldCount ； 
   - 若当前线程线程为第一个读线程，则增加 firstReaderHoldCount ；  否则，将设置当前线程对应的 HoldCounter 对象的值，更新成功后会在 firstReaderHoldCount 中 readHolds ( ThreadLocal 类型的)的本线程副本中记录当前线程重入数，

 这是为了实现 JDK1.6 中加入的 getReadHoldCount ()方法的，这个方法能获取当前线程重入共享锁的次数( state 中记录的是多个线程的总重入次数)，加入了这个方法让代码复杂了不少，但是其原理还是很简单的： 

 如果当前只有一个线程的话，还不需要动用 ThreadLocal ，直接往 firstReaderHoldCount 这个成员变量里存重入数，当有第二个线程来的时候，就要动用 ThreadLocal 变量 readHolds 了，每个线程拥有自己的副本，用来保存自己的重入数。 

 获取读锁源码：  

```java
/**
         * 获取读锁
         * Acquires the read lock.
         * 如果写锁未被其他线程持有，执行CAS操作更新status值，获取读锁后立即返回
         * <p>Acquires the read lock if the write lock is not held by
         * another thread and returns immediately.
         *
         * 如果写锁被其他线程持有，那么停止该线程的CPU调度并进入休眠状态，直到该读锁被释放
         * <p>If the write lock is held by another thread then
         * the current thread becomes disabled for thread scheduling
         * purposes and lies dormant until the read lock has been acquired.
         */
        public void lock() {
            sync.acquireShared(1);
        }
   /**
     * 该方法为以共享模式获取读锁，忽略中断
     * 如果调用一次该“tryAcquireShared”方法更新status成功，则直接返回，代表抢锁成功
     * 否则，将会进入同步队列等待，不断执行“tryAcquireShared”方法尝试CAS更新status状态，直到成功抢到锁
     * 其中“tryAcquireShared”方法在NonfairSync(公平锁)中和FairSync(非公平锁)中都有各自的实现
     * (看这注释是不是和写锁很对称)
     * Acquires in shared mode, ignoring interrupts.  Implemented by
     * first invoking at least once {@link #tryAcquireShared},
     * returning on success.  Otherwise the thread is queued, possibly
     * repeatedly blocking and unblocking, invoking {@link
     * #tryAcquireShared} until success.
     *
     * @param arg the acquire argument.  This value is conveyed to
     *        {@link #tryAcquireShared} but is otherwise uninterpreted
     *        and can represent anything you like.
     */
    public final void acquireShared(int arg) {
        if (tryAcquireShared(arg) < 0)
            doAcquireShared(arg);
    }
    protected final int tryAcquireShared(int unused) {
            /*
             * Walkthrough:
             * 1、如果已经有其他线程获取到了写锁，根据“读写互斥”原则，抢锁失败，返回-1
             * 1.If write lock held by another thread, fail.
             * 2、如果该线程本身持有写锁，那么看一下是否要readerShouldBlock，如果不需要阻塞，
             *    则执行CAS操作更新state和重入计数。
             *    这里要注意的是，上面的步骤不检查是否可重入(因为读锁属于共享锁，天生支持可重入)
             * 2. Otherwise, this thread is eligible for
             *    lock wrt state, so ask if it should block
             *    because of queue policy. If not, try
             *    to grant by CASing state and updating count.
             *    Note that step does not check for reentrant
             *    acquires, which is postponed to full version
             *    to avoid having to check hold count in
             *    the more typical non-reentrant case.
             * 3、如果因为CAS更新status失败或者重入计数超过最大值导致步骤2执行失败
             *    那就进入到fullTryAcquireShared方法进行死循环，直到抢锁成功
             * 3. If step 2 fails either because thread
             *    apparently not eligible or CAS fails or count
             *    saturated, chain to version with full retry loop.
             */

            //当前尝试获取读锁的线程
            Thread current = Thread.currentThread();
            //获取该读写锁状态
            int c = getState();
            //如果有线程获取到了写锁 ，且获取写锁的不是当前线程则返回失败
            if (exclusiveCount(c) != 0 &&
                getExclusiveOwnerThread() != current)
                return -1;
            //获取读锁的重入计数
            int r = sharedCount(c);
            //如果读线程不应该被阻塞，且重入计数小于最大值，且CAS执行读锁重入计数+1成功，则执行线程重入的计数加1操作，返回成功
            if (!readerShouldBlock() &&
                r < MAX_COUNT &&
                compareAndSetState(c, c + SHARED_UNIT)) {
                //如果还未有线程获取到读锁，则将firstReader设置为当前线程，firstReaderHoldCount设置为1
                if (r == 0) {
                    firstReader = current;
                    firstReaderHoldCount = 1;
                } else if (firstReader == current) {
                    //如果firstReader是当前线程，则将firstReader的重入计数变量firstReaderHoldCount加1
                    firstReaderHoldCount++;
                } else {
                    //否则说明有至少两个线程共享读锁，获取共享锁重入计数器HoldCounter
                    //从HoldCounter中拿到当前线程的线程变量cachedHoldCounter，将此线程的重入计数count加1
                    HoldCounter rh = cachedHoldCounter;
                    if (rh == null || rh.tid != getThreadId(current))
                        cachedHoldCounter = rh = readHolds.get();
                    else if (rh.count == 0)
                        readHolds.set(rh);
                    rh.count++;
                }
                return 1;
            }
            //如果上面的if条件有一个都不满足，则进入到这个方法里进行死循环重新获取
            return fullTryAcquireShared(current);
        }
        /**
         * 用于处理CAS操作state失败和tryAcquireShared中未执行获取可重入锁动作的full方法(补偿方法？)
         * Full version of acquire for reads, that handles CAS misses
         * and reentrant reads not dealt with in tryAcquireShared.
         */
        final int fullTryAcquireShared(Thread current) {
            /*
             * 此代码与tryAcquireShared中的代码有部分相似的地方，
             * 但总体上更简单，因为不会使tryAcquireShared与重试和延迟读取保持计数之间的复杂判断
             * This code is in part redundant with that in
             * tryAcquireShared but is simpler overall by not
             * complicating tryAcquireShared with interactions between
             * retries and lazily reading hold counts.
             */
            HoldCounter rh = null;
            //死循环
            for (;;) {
                //获取读写锁状态
                int c = getState();
                //如果有线程获取到了写锁
                if (exclusiveCount(c) != 0) {
                    //如果获取写锁的线程不是当前线程，返回失败
                    if (getExclusiveOwnerThread() != current)
                        return -1;
                    // else we hold the exclusive lock; blocking here
                    // would cause deadlock.
                } else if (readerShouldBlock()) {//如果没有线程获取到写锁，且读线程要阻塞
                    // Make sure we're not acquiring read lock reentrantly
                    //如果当前线程为第一个获取到读锁的线程
                    if (firstReader == current) {
                        // assert firstReaderHoldCount > 0;
                    } else { //如果当前线程不是第一个获取到读锁的线程(也就是说至少有有一个线程获取到了读锁)
                        //
                        if (rh == null) {
                            rh = cachedHoldCounter;
                            if (rh == null || rh.tid != getThreadId(current)) {
                                rh = readHolds.get();
                                if (rh.count == 0)
                                    readHolds.remove();
                            }
                        }
                        if (rh.count == 0)
                            return -1;
                    }
                }
                /**
                 *下面是既没有线程获取写锁，当前线程又不需要阻塞的情况
                 */
                //重入次数等于最大重入次数，抛异常
                if (sharedCount(c) == MAX_COUNT)
                    throw new Error("Maximum lock count exceeded");
                //如果执行CAS操作成功将读写锁的重入计数加1，则对当前持有这个共享读锁的线程的重入计数加1，然后返回成功
                if (compareAndSetState(c, c + SHARED_UNIT)) {
                    if (sharedCount(c) == 0) {
                        firstReader = current;
                        firstReaderHoldCount = 1;
                    } else if (firstReader == current) {
                        firstReaderHoldCount++;
                    } else {
                        if (rh == null)
                            rh = cachedHoldCounter;
                        if (rh == null || rh.tid != getThreadId(current))
                            rh = readHolds.get();
                        else if (rh.count == 0)
                            readHolds.set(rh);
                        rh.count++;
                        cachedHoldCounter = rh; // cache for release
                    }
                    return 1;
                }
            }
        }

```

 释放读锁源码： 

````java
/**
  * Releases in shared mode.  Implemented by unblocking one or more
  * threads if {@link #tryReleaseShared} returns true.
  *
  * @param arg the release argument.  This value is conveyed to
  *        {@link #tryReleaseShared} but is otherwise uninterpreted
  *        and can represent anything you like.
  * @return the value returned from {@link #tryReleaseShared}
  */
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {//尝试释放一次共享锁计数
        doReleaseShared();//真正释放锁
        return true;
    }
        return false;
}
/**
 *此方法表示读锁线程释放锁。
 *首先判断当前线程是否为第一个读线程firstReader，
 *若是，则判断第一个读线程占有的资源数firstReaderHoldCount是否为1，
  若是，则设置第一个读线程firstReader为空，否则，将第一个读线程占有的资源数firstReaderHoldCount减1；
  若当前线程不是第一个读线程，
  那么首先会获取缓存计数器（上一个读锁线程对应的计数器 ），
  若计数器为空或者tid不等于当前线程的tid值，则获取当前线程的计数器，
  如果计数器的计数count小于等于1，则移除当前线程对应的计数器，
  如果计数器的计数count小于等于0，则抛出异常，之后再减少计数即可。
  无论何种情况，都会进入死循环，该循环可以确保成功设置状态state
 */
protected final boolean tryReleaseShared(int unused) {
      // 获取当前线程
      Thread current = Thread.currentThread();
      if (firstReader == current) { // 当前线程为第一个读线程
          // assert firstReaderHoldCount > 0;
         if (firstReaderHoldCount == 1) // 读线程占用的资源数为1
              firstReader = null;
          else // 减少占用的资源
              firstReaderHoldCount--;
     } else { // 当前线程不为第一个读线程
         // 获取缓存的计数器
         HoldCounter rh = cachedHoldCounter;
         if (rh == null || rh.tid != getThreadId(current)) // 计数器为空或者计数器的tid不为当前正在运行的线程的tid
             // 获取当前线程对应的计数器
             rh = readHolds.get();
         // 获取计数
         int count = rh.count;
         if (count <= 1) { // 计数小于等于1
             // 移除
             readHolds.remove();
             if (count <= 0) // 计数小于等于0，抛出异常
                 throw unmatchedUnlockException();
         }
         // 减少计数
         --rh.count;
     }
     for (;;) { // 死循环
         // 获取状态
         int c = getState();
         // 获取状态
         int nextc = c - SHARED_UNIT;
         if (compareAndSetState(c, nextc)) // 比较并进行设置
             // Releasing the read lock has no effect on readers,
             // but it may allow waiting writers to proceed if
             // both read and write locks are now free.
             return nextc == 0;
     }
 }
 /**真正释放锁
  * Release action for shared mode -- signals successor and ensures
  * propagation. (Note: For exclusive mode, release just amounts
  * to calling unparkSuccessor of head if it needs signal.)
  */
private void doReleaseShared() {
        /*
         * Ensure that a release propagates, even if there are other
         * in-progress acquires/releases.  This proceeds in the usual
         * way of trying to unparkSuccessor of head if it needs
         * signal. But if it does not, status is set to PROPAGATE to
         * ensure that upon release, propagation continues.
         * Additionally, we must loop in case a new node is added
         * while we are doing this. Also, unlike other uses of
         * unparkSuccessor, we need to know if CAS to reset status
         * fails, if so rechecking.
         */
        for (;;) {
            Node h = head;
            if (h != null && h != tail) {
                int ws = h.waitStatus;
                if (ws == Node.SIGNAL) {
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                        continue;            // loop to recheck cases
                    unparkSuccessor(h);
                }
                else if (ws == 0 &&
                         !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                    continue;                // loop on failed CAS
            }
            if (h == head)                   // loop if head changed
                break;
        }
    }

````

通过分析可以看出：

在线程持有读锁的情况下，该线程不能取得写锁(因为获取写锁的时候，如果发现当前的读锁被占用，就马上获取失败，不管读锁是不是被当前线程持有)。

在线程持有写锁的情况下，该线程可以继续获取读锁（获取读锁时如果发现写锁被占用，只有写锁没有被当前线程占用的情况才会获取失败）。

##  LongAdder

-  在高并发的情况下，我们对一个 Integer 类型的整数直接进行 i++ 的时候，无法保证操作的原子性，会出现线程安全的问题。为此我们会用 juc 下的 AtomicInteger ，它是一个提供原子操作的 Interger 类，内部也是通过 CAS 实现线程安全的 

-  但当大量线程同时去访问时，就会因为大量线程执行 CAS 操作失败而进行空旋转，导致 CPU 资源消耗过多，而且执行效率也不高。 
-  Doug Lea 大神应该也不满意，于是在 JDK1.8 中对 CAS 进行了优化，提供了 LongAdder ，它是基于了 CAS 分段锁的思想实现的。 

>  线程去读写一个 LongAdder 类型的变量时，流程如下： 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B-20.png)



-  LongAdder 也是基于 Unsafe 提供的 CAS 操作 +valitale 去实现的。 
-  在 LongAdder 的父类 Striped64 中维护着一个 base 变量和一个 cell 数组，当多个线程操作一个变量的时候，先会在这个 base 变量上进行 cas 操作，当它发现线程增多的时候，就会使用 cell 数组。 
-  比如当 base 将要更新的时候发现线程增多（也就是调用 casBase 方法更新 base 值失败），那么它会自动使用 cell 数组，每一个线程对应于一个 cell ，在每一个线程中对该 cell 进行 cas 操作，这样就可以将单一 value 的更新压力分担到多个 value 中去，降低单个 value 的 “热度”，同时也减少了大量线程的空转，提高并发效率，分散并发压力。 
-  这种分段锁需要额外维护一个内存空间 cells ，不过在高并发场景下，这点成本几乎可以忽略。分段锁是一种优秀的优化思想， juc 中提供的的 ConcurrentHashMap 也是基于分段锁保证读写操作的线程安全。 



> 参考文章

 读写锁的源码分析，推荐阅读 [聊聊 Java 的几把 JVM 级锁 - 阿里巴巴中间件](https://mp.weixin.qq.com/s/h3VIUyH9L0v14MrQJiiDbw) 这篇文章，写的很不错。 