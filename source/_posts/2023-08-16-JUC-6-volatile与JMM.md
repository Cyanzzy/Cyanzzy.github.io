---
title: JUC-6-volatile与JMM
date: 2023-08-16 16:33:27
tags: 
  - Java
categories: 
  - Language
---


# volatile

> 被`volatile`修饰的变量特点

* 可见性
* 有序性

> `volatile`内存语义

* 当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值立即刷新回主内存中 

* 当读一个volatile变量时，JMM会把该线程对应的本地内存设置为无效，重新回到主内存中读取最新共享变量
* volatile`写`的内存语义是直接刷新到主内存中，`读`的内存语义是直接从主内存中读取。

# 内存屏障

内存屏障（内存栅栏，屏障指令等），是一类同步屏障指令，是CPU或编译器在对内存随机访问的操作中的一个同步点，使得此点之前的所有读写操作都执行后才可以开始执行此点之后的操作，避免代码重排序。内存屏障其实就是一种JVM指令，Java内存模型的重排规则会要求Java编译器在生成JVM指令时插入特定的内存屏障指令，通过这些内存屏障指令，volatie实现了Java内存模型中的可见性和有序性（禁重排），但volatile无法保证原子性。

**内存屏障之前**的所有**写操作**都要回写到主内存，
**内存屏障之后**的所有**读操作**都能获得内存屏障之前的所有写操作的最新结果（实现了可见性）。
写屏障（Store Memory Barrier）：告诉处理器在写屏障之前将所有存储在缓存（store bufferes）中的数据同步到主内存。当看到Store屏障指令，就必须把该指令之前所有写入指令执行完毕才能继续往下执行。
读屏障（Load Memory Barrier）：处理器在读屏障之后的读操作，都在读屏障之后执行。在Load屏障指令之后就能够保证后面的读取数据指令一定能够读取到最新的数据。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-46.jpg)

因此重排序时不允许把内存屏障之后的指令重排序到内存屏障之前。对一个volatile变量的写，先行发生于任意后续对这个volatile变量的读（写后读）

> 读屏障

在读指令之前插入读屏障，让工作内存或CPU高速缓存当中的缓存数据失效，重新回到主内存中获取最新数据

> 写屏障

在写指令之后插入写屏障，强制把写缓冲区的数据刷回到主内存中

> 四大屏障

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-48.jpg)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-47.jpg)

> 保证有序性

通过内存屏障禁重排

* 对于编译器的重排序，JMM会根据重排序的规则，禁止特定类型的编译器重排序。
* 对于处理器的重排序，Java编译器在生成指令序列的适当位置，插入内存屏障指令，来禁止特定类型的处理器排序。

> happens-before --- volatile变量规则

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-49.jpg)

> JMM 将内存屏障插入策略分为4种规则

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-47.jpg)

**读屏障**

* 在每个volatile读操作的后面插入一个LoadLoad屏障
* 在每个volatile读操作的后面插入一个LoadStore屏障

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-50.jpg)

**写屏障**

* 在每个volatile 写操作的前面插入一个StoreStore屏障
* 在每个volatile 写操作的后面插入一个StoreLoad屏障

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-51.jpg)

# volatile 特性

## 保证可见性

> 可见性

保证不同线程对某个变量完成操作后结果及时可见，即该共享变量一旦改变所有线程立即可见

> 案例演示

* 不加volatile，没有可见性，程序无法停止
* 加了volatile，保证可见性，程序可以停止

```java
private static volatile boolean flag = true;

public static void main(String[] args) {
    new Thread(() -> {
        System.out.println(Thread.currentThread().getName() + "\t ---进来了");
        while (flag) {

        }
        System.out.println(Thread.currentThread().getName() + "\t ---flag被设置为false，程序停止");
    }, "t1").start();

    try {
        TimeUnit.SECONDS.sleep(2);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    flag = false;

    System.out.println(Thread.currentThread().getName() + "\t 修改完成， flag = " + flag);
}

t1	 ---进来了
main	 修改完成， flag = false
t1	 ---flag被设置为false，程序停止
```

> 未使用`volatile`修饰flag时，t1为何看不到被主线程main修改为false的flag的值

**可能原因**

* 主线程修改了flag之后没有将其刷新到主内存，所以t1线程看不到。
* 主线程将flag刷新到了主内存，但是t1一直读取的是自己工作内存中flag的值，没有去主内存中更新获取flag最新的值。

**解决方案**
使用volatile修饰共享变量，被volatile修改的变量有以下特点:

* 线程中读取的时候，每次读取都会去主内存中读取共享变量最新的值，然后将其复制到工作内存
* 线程中修改了工作内存中变量的副本，修改之后会立即刷新到主内存 

> volatile变量的读写过程

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-52.jpg)

| 指令   | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| read   | 作用于主内存，将变量的值从主内存传输到工作内存，主内存到工作内存 |
| load   | 作用于工作内存，即数据加载，将read从主内存传输的变量值放入工作内存变量副本中 |
| use    | 作用于工作内存，将工作内存变量副本的值传递给执行引擎，每当JVM遇到需要该变量的字节码指令时会执行该操作 |
| assign | 作用于工作内存，将从执行引擎接收到的值赋值给工作内存变量，每当JVM遇到一个给变量赋值字节码指令时会执行该操作 |
| store  | 作用于工作内存，将赋值完毕的工作变量的值写回给主内存         |
| write  | 作用于主内存，将store传输过来的变量值赋值给主内存中的变量    |

**由于上述6条只能保证单条指令的原子性，针对多条指令的组合性原子保证，没有大面积加锁，所以，JVM提供了另外两个原子指令**

| 指令   | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| lock   | 作用于主内存，将一个变量标记为一个线程独占的状态，只是写时候加锁，就只是锁了写变量的过程。 |
| unlock | 作用于主内存，把一个处于锁定状态的变量释放，然后才能被其他线程占用 |

## 没有原子性

volatile变量的复合操作不具有原子性

> 读取赋值一个普通变量的情况

当线程1对主内存对象发起read操作到write操作第一套流程的时间里，线程2随时都有可能对这个主内存对象发起第二套操作

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-53.jpg)

> 不保证原子性

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-54.jpg)

对于volatle变量具备可见性，JVM只是保证从主内存加载到线程工作内存的值是最新的，也仅是数据加载时是最新的。但是多线程环境下，"数据计算"和"数据赋值"操作可能多次出现，若数据在加载之后，若主内存volatile修饰变量发生修改之后，
线程工作内存中的操作将会作废去读主内存最新值，操作出现写丢失问题。**即各线程私有内存和主内存公共内存中变量不同步**，进而导致数据不一致。由此可见volatile解决的是变量读时的可见性问题，**但无法保证原子性，对于多线程修改主内存共享变量的场景必须使用加锁问步**

> 从`i++`字节码角度说明

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-55.jpg)

原子性指的是一个操作是**不可中断**的，即使是在多线程环境下，一个操作一旦开始就不会被其他线程影响。

```java
public void add() {
    i++; // 不具备原子性，该操作是先读取值，然后写回一个新值，相当于原来的值加上1，分3步完成
}
```

如果第二个线程在第一个线程读取旧值和写回新值期间读取i的域值，那么第二个线程就会与第一个线程一起看到同一个值；并执行相同值的加1操作，这也就造成了线程安全失败，因此对于add方法必须使用synchronized修饰以便保证线程安全 

> 结论

volatile变量不适合参与到依赖当前值的运算

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-56.jpg)

## 指令禁重排

> 重排序

重排序是指编译器和处理器为了优化程序性能而对指令序列进行重新排序的一种手段，有时候会改变程序语句的先后顺序。如果不存在数据依赖关系，可以重排序；如果存在数据依赖关系，禁止重排序；但重排后的指令绝对不能改变原有的串行语义 

> 重排序的分类和执行流程

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-57.jpg)

* 编译器优化的重排序：编译器在不改变单线程串行语义的前提下，可以重新调整指令的执行顺序
* 指令级并行的重排序：处理器使用指令级并行技术来讲多条指令重叠执行，若不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序
* 内存系统的重排序：由于处理器使用缓存和读/写缓冲区，这使得加载和存储操作看上去可能是乱序执行

> 数据依赖性

若两个操作访问同一变量，且这两个操作中有一个为写操作，此时两操作间就存在数据依赖性。

> volatile 禁止指令重排行为

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-58.jpg)

> 四大屏障插入情况

* 在每一个volatile写操作前面插入一个StoreStore屏障：StoreStore屏障可以保证在volatile写之前，其前面的所有普通写操作都已经刷新到主内存
* 在每一个volatile写操作后面插入一个StoreLoad屏障：StoreLoad屏障的作用是避免volatile写与后面可能有的volatile读/写操作重排序
* 在每一个volatile读操作后面插入一个LoadLoad屏障：LoadLoad屏障用来禁止处理器把上面的volatile读与下面的普通读重排序
* 在每一个volatile读操作后面插入一个LoadStore屏障：LoadStore屏障用来禁止处理器把上面的volatile读与下面的普通写重排序


![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-59.jpg)



# 正确使用volatile

> 单一赋值可以，但是含复合运算赋值不可以（i++之类）

```java
// 单一赋值
volatile int a = 10;
volatile boolean flag = false;
```



> 状态标志，判断业务是否结束﹒

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-60.jpg)



> 开销较低的读，写锁策略

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-61.jpg)

> DCL双端锁的发布

**存在问题**

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-62.jpg)

**单线程**

单线程环境下，上述问题代码会执行如下操作保证获取到已完成初始化的实例

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-63.jpg)

**多线程**

多线程环境下，上述问题代码会执行如下操作，由于重排序导致2,3乱序，导致其他线程得到的是null而不是完成的初始化对象

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-64.jpg)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-65.jpg)

**解决方案**

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-66.jpg)

# 总结

> **volatile 可见性**

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-67.jpg)

> **volatile 没有原子性**

> **volatile 禁止指令重排**

* 写指令

  ![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-68.jpg)

* 读指令

  ![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-69.jpg)

> Java代码中volatile与系统底层加入的内存屏障的关系

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-70.jpg)

> **WHAT 内存屏障**

内存屏障也叫内存栅栏或栅栏指令，是一种屏障指令，它使得CPU或编译器对屏障指令的前和后所发出的内存操作执行一个排序的约束 

> **WHY 内存屏障**

* 阻止屏障两边的指令重排序
* 写数据时加入屏障，强制将线程私有工作内存的数据刷回主物理内存
* 读数据时加入屏障，线程私有工作内存的数据失效，重新到主物理内存中获取最新数据

>  **内存屏障四大指令**

* 在每一个volatile写操作前面插一个StoreStore屏障
* 在每一个volatile写操作后面插入一个StoreLoad屏障
* 在每一个volatile读操作后面插入一个LoadLoad屏障
* 在每一个volatile读操作后面插入一个LoadStore屏障

> 小总结

* volatile 写之前的操作，都禁止重排序到volatile之后
* volatile读之后的操作，都禁止重排序到volatile 之前
* volatile 写之后volatile读，禁止重排序
