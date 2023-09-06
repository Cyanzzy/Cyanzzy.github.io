---
title: JUC-10-Synchronized与锁升级
date: 2023-09-06 14:01:12
tags: 
  - Java
categories: 
  - Language
password: zzy   
message: 亲，能不能输入密码啊？
---

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-106.jpg)

# Synchronized性能变化

> Java5前，只有synchronized重量级锁，如果竞争激烈，性能会下降

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-107.jpg)

Java早期版本中，synchronized属于重量级锁，效率低下，因为监视器锁（monitor）是依赖底层的操作系统的MutexLock（系统互斥量）实现的，挂起线程和恢复线程都需要转入内核态去完成，阻塞或唤醒一个Java线程需要操作系统切换CPU状态来完成，这种状态切换需要耗费处理器时间。

Java的线程是映射到操作系统原生线程之上的，如果要阻塞或唤醒一个线程就需要操作系统介入，需要在户态与核心态之间切换，这种切换会消耗大量的系统资源。为了减少获得锁和释放锁所带来的性能消耗引入了轻量级锁和偏向锁。



> WHY 每个对象都可以成为一个锁

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-108.jpg)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-109.jpg)

Monitor是一个同步工具，常被描述为一个Java对象，它依赖于底层操作系统的MutexLock实现，操作系统实现线程之间的切换需要从用户态到内核态的转换是成本非常高的

**Monitor**

JVM中的同步基于进入和退出管程（Monitor）对象实现，每个对象实例都会有一个Monitor，Monitor可以和对象一起创建、销毁。Monitor由ObjectMonitor实现，而ObjectMonitor是由C++的ObjectMonitor.hpp文件实现：

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-110.jpg)

**MutexLock**

Monitor是在JVM底层实现，本质是依赖于底层操作系统的MutexLock实现，操作系统实现线程之间的切换需要从用户态到内核态的转换，状态转换需要耗费很多的处理器时间成本非常高。

**Monitor与Java对象以及线程如何关联**

* 如果Java对象被某个线程锁住，则该Java对象的Mark Word字段中LockWord指向monitor的起始地址
* Monitor的Owner字段会存放拥有相关联对象锁的线程id

MutexLock的切换需要从用户态转换到核心态中，因此状态转换需要耗费很多的处理器时间

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-111.jpg)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-112.jpg)

> Java6后为减少获取锁和释放锁带来的性能消耗引入轻量级锁和偏向锁

# Synchronized锁升级

synchronized用的锁是存在Java对象头里的Mark Word中
锁升级功能主要依赖MarkWord中锁标志位和释放偏向锁标志位

> 64 位标记图

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-112.jpg)

> 锁指向

* 偏向锁：MarkWord存储的是偏向的线程ID
* 轻量：锁MarkWord存储的是指向线程栈中Lock Record的指针
* 重量锁：MarkWord存储的是指向堆中的monitor对象的指针

## 无锁

初始状态，一个对象被实例化后，若还未被任何线程竞争锁，那么它为`无锁状态`（001）

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-113.jpg)

> 调用`hashCode()`前

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-114.jpg)

> 调用`hashCode()`后

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-115.jpg)



## 偏锁

> 偏向锁

单线程进程，当线程A第一次竞争到锁时，通过操作修改Mark Word中的偏向线程ID、偏向模式。如果不存在其他线程竞争，那么持有偏向锁的线程将永远不需要进行同步。

> 主要作用

当一段同步代码一直被同一个线程多次访问，由于只有一个线程那么该线程在后续访问时便会自动获得锁

> 研究发现

多线程的情况下，锁不仅不存在多线程竞争，还存在锁由同一个线程多次获得的情况，偏向锁就是在这种情况下出现的，它的出现是为解决只有在一个线程执行同步时提高性能。

偏向锁会偏向于第一个访问锁的线程，如果在接下来的运行过程中，该锁没有被其他的线程访问，则持有偏向锁的线程将永远不需要触发同步

> 说明

在实际应用运行过程中发现，“锁总是同一个线程持有，很少发生竞争”，即锁总是被第一个占用他的线程拥有，这个线程就是锁的偏向线程。

那么只需要在锁第一次被拥有时记录下偏向线程ID。这样偏向线程就一直持有着锁，后续该线程进入和退出这段加同步锁的代码块时，不需要再次加锁和释放锁。而是直接会去检查锁的MarkWord里面是不是放的自己的线程ID

* 如果相等表示偏向锁偏向于当前线程，不需再尝试获得锁，直到竞争发生才释放锁。以后每次同步，检查锁的偏向线程ID与当前线程ID是否一致，如果一致直接进入同步。无需每次加锁解锁都去CAS更新对象头
* 如果不等表示发生竞争，锁已经不总是偏向于同一个线程，这时会尝试使用CAS来替换MarkWord里面的线程ID为新线程的ID
  * 竞争成功表示之前的线程不存在了，MarkWord里面的线程ID为新线程的ID，锁不会升级，仍然为偏向锁
  * 竞争失败，这时可能需要升级变为轻量级锁，才能保证线程间公平竞争锁

注意：偏向锁只有遇到其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁，线程不会主动释放偏向锁

> 技术实现

synchronized方法被一个线程抢到了锁时，那这个方法所在的对象就会在其所在的Mark Word中将偏向锁修改状态位，同时还会有占用前54位来存储线程指针作为标识。若该线程再次访问同一个synchronized方法时，该线程只需去对象头的Mark Word中去判断一下是否有偏向锁指向本身的ID，无需再进入 Monitor去竞争对象。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-116.jpg)

> 栗子

偏向锁的操作不涉及用户到内核转换，不必要直接升级为最高级，以一个account对象的“对象头”为例：

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-117.jpg)

假如有个线程执行到synchronized代码块时，JVM使用CAS操作把线程指针ID记录到Mark Word，并修改标偏向标示，标示当前线程就获得该锁。锁对象变成偏向锁（通过CAS修改对象头里的锁标志位）。执行完同步代码块后，线程并不会主动释放偏向锁

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-118.jpg)

此时线程获得锁并可以执行同步代码块，当该线程第二次到达同步代码块时会判断此时持有锁的线程是否还是自己，JVM通过account对象的Mark Word判断到当前线程ID还在，说明还持有着这个对象的锁，就可以继续进入临界区工作。由于之前未释议锁，这里就不需要重新加锁。如果自始至终使用锁的线程只有一个，很明显偏向锁几乎没有额外开销，性能极高。

> 偏锁JVM参数

| JVM参数                                                 | 说明                                             |
| ------------------------------------------------------- | ------------------------------------------------ |
| `-XX:+UseBiasedLocking -XX:BiasedLockingStartupDelay=0` | 开启偏向锁                                       |
| `-XX:UseBiasedLocking`                                  | 关闭偏向锁（关闭后程序默认直接进入轻量级锁状态） |

> 一切默认情况

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-119.jpg)

**参数系统默认开启**

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-120.jpg)

**关闭延时参数，启用该功能**

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-121.jpg)

> 偏向锁的撤销

* 当有另外线程逐步来竞争锁时就不能再使用偏向锁，而要升级为轻量级锁
* 竞争线程尝试CAS更新对象头失败，会等待到全局安全点（此时不会执行任何代码）撤销偏向锁

**偏锁的撤销：**

偏向锁使用一种等到**竞争出现才释放锁**的机制，只有当其他线程竞争锁时，持有偏向锁的原来线程才会被撤销。撤销需要等待全局安全点，同时检查持有偏向锁的线程是否还在执行：

* 第一个线程正在执行synchronized方法（处于同步块），它还没有执行完，其它线程来抢夺，该偏向锁会被取消掉并出现锁升级。此时轻量级锁由原持有偏向锁的线程持有，继续执行其同步代码，而正在竞争的线程会进入自旋等待获得该轻量级锁
* 第一个线程执行完成synchronized方法（退出同步块），则将对象头设置成无锁状态并撤销偏向锁，重新偏向

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-122.jpg)

> 总体步骤流程（红线流程部分为偏向锁获取和撤销流程）

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-123.jpg)

## 轻锁

> 轻量级锁

多线程竞争，但是任意时刻最多只有一个线程竞争，即不存在锁竞争太过激烈的情况，也就没有线程阻塞。

> 主要作用

有线程来参与锁的竞争，但是获取锁的冲突时间极短，本质是CAS自旋锁

> 轻量锁的获取

轻量级锁是为在线程近乎交替执行同步块时提高性能 

* 主要目的：在没有多线程竞争的前提下，通过CAS减少重量级锁使用操作系统互斥量产生的性能消耗（即先自旋，不行才升级阻塞）

* 升级时机：当关闭偏向锁功能或多线程竞争偏向锁会导致偏向锁升级为轻量级锁

假如线程A已经拿到锁，这时线程B又来抢该对象的锁，由于该对象的锁已经被线程A拿到，当前该锁则已是偏向锁

而线程B在争抢时发现对象头Mark Word中的线程ID不是线程B自己的线程ID（而是线程A的），那线程B就会进行CAS操作希望能获得锁

**此时线程B操作中有两种情况：**
**如果锁获取成功**，直接替换Mak Word中的线程ID为B自己的ID（A→B），重新偏向于其他线程（即将偏向锁交给其他线程，相当于当前线程"被"释放锁），该锁会保持偏向锁状态，A线程结束，B线程上位 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-124.jpg)

**如果锁获取失败**，则偏向锁升级为轻量级锁（设置偏向锁标识为0，并设置锁标志位为00），此时轻量级锁由原持有偏向锁的线程持有，继续执行其同步代码，而正在竞争的线程B会进入自旋等待获得该轻量级锁 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-125.jpg)

> 轻量级锁的加锁

JVM会为每个线程在当前线程的栈帧中创建用于存储锁记录的空间（Displaced Mark Word）。若一个线程获得锁时发现是轻量级锁，会把锁的MarkWord复制到自己的Displaced Mark Word里面。然后线程尝试用CAS将锁的MarkWord替换为指向锁记录的指针。如果成功，当前线程获得锁，如果失败，表示Mark Word已经被替换成其他线程的锁记录，说明在与其它线程竞争锁，当前线程就尝试使用自旋来获取锁。

自旋CAS：不断尝试去获取锁，能不升级就不往上捅，尽量不要阻塞

> 轻量级锁的释放

在释放锁时，当前线程会使用CAS操作将Displaced Mark Word的内容复制回锁的Mark Word里面。如果没有发生竞争，那么这个复制的操作会成功。如果有其他线程因为自旋多次导致轻量级锁升级成重量级锁，那么CAS操作会失败，此时会释放锁并唤醒被阻塞的线程。

> 轻量级锁演示

| VM参数                  | 说明                                   |
| ----------------------- | -------------------------------------- |
| `-XX:-UseBiasedLocking` | 如果关闭偏向锁，则可以直接进入轻量级锁 |

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-126.jpg)

> 步骤流程图

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-123.jpg)

> 自旋达到一定次数和程度

* Java6前默认启用，默认情况下自旋次数为10，使用`-XX:PreBlockSpin=10`修改。或者自旋线程数超过CPU核数一半

* Java6后，如果线程自旋成功，那么下次自旋的最大次数会增加；如果很少自旋成功，那么下次会减少自旋次数甚至不自旋（自适应意味自旋次数不是固定不变，而是根据同一个锁上次自旋的时间核拥有锁线程的状态来决定）

> 轻量级锁和偏向锁的区别

* 争夺轻量级锁失败时自选尝试抢占锁
* 轻量级锁每次退出同步块都需要释放锁，而偏向锁是在竞争发生时才释放锁

## 重锁

> 重锁

有大量的线程参与锁的竞争吗，冲突性很高

> 锁标志位

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-127.jpg)

> 重量级锁原理

Java中`synchronized`锁基于进入和退出Monitor对象实现，在编译时会将同步块的开始位置插入`monitor enter`指令，在结束位置插入`monitor exit`指令。
当线程执行到`monitor enter`指令时会尝试获取对象所对应的Monitor所有权，如果获取到锁则会在Monitor的`owner`中存放当前线程的id，这样它将处于锁定状态，除非退出同步块，否则其他线程无法获取到此Monitor。

> 栗子

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-128.jpg)

## 锁升级后HashCode去向

> 锁升级为轻量级或重量级锁后，Mark Word中保存的分别是线程栈帧里的锁记录指针和重量级锁指针，已经没有位置再保存哈希码，GC年龄了，那么这些信息被移动到哪里去了呢？

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-129.jpg)

**在无锁状态下**，Mark Word中可以存储对象的`identity hash code`值。当对象的`hashCode()`方法第一次被调
用时，JVM会生成对应的`identity hash code`值并将该值存储到Mark Word中。

**对于偏向锁**，在线程获取偏向锁时会用Thread ID和epoch值覆盖`identity hash code`所在位置。如果一个对象的`hashCode()`方法已经被调用过一次后，该对象不能被设置偏向锁（因为如果可以的设置的话，那Mark Word中的`identity hash code`必然会被偏向线程ld给覆盖，这就会造成同一个对象前后两次调用`hashCode()`方法得到的结果不一致）

**升级为轻量级锁时**，JVM会在当前线程的栈帧中创建一个锁记录（Lock Record）空间，用于存储锁对象的Mark Word拷贝，该拷贝中可以包含`identity hash code`，所以轻量级锁可以和`identity hash code`共存，哈希码和GC年龄自然保存在此，释放锁后会将这些信息写回到对象头。

**升级为重量级锁后**，Mark Word保存的重量级锁指针，代表重量级锁的`ObjectMonitor`类里有字段记录非加锁状态下的Mark Word，锁释放后也会将信息写回到对象头。

## 各种锁优缺点

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-130.jpg)

> 偏向锁

适用于单线程适用的情况，不存在锁竞争时进入同步方法/代码块则使用偏向锁

> 轻量级锁

适用于竞争较不激烈的情况，存在竞争时升级为轻量级锁，轻量级锁采用的是自旋锁，如果同步方法/代码块执行时间很短，采用轻量级锁虽然会占用cpu资源但是相对比使用重量级锁还是更高效

> 重量级锁

适用于竞争激烈的情况，如果同步方法/代码块执行时间很长，那么使用轻量级锁自旋带来的性能消耗就比使用重量级锁更严重，这时就需要升级为重量级锁

# JIT编译器对锁的优化

>  JIT

Just In Time Compiler

> 锁消除和锁粗化

Java 中的锁消除（Lock Elimination）和锁粗化（Lock Coarsening）都是与多线程编程相关的优化技术，它们旨在提高程序的性能和降低同步操作的开销 

 **锁消除（Lock Elimination）**： 

 锁消除是一种编译器或运行时系统的优化技术，用于消除不必要的同步操作。在多线程编程中，为了保护共享资源或临界区，程序员通常使用锁（如 synchronized 关键字或 ReentrantLock 类）来确保只有一个线程可以访问该资源。然而，在某些情况下，编译器或运行时系统可以通过分析代码来确定某些锁操作是不必要的，然后将它们消除掉。 

锁消除通常发生在以下情况下：

- 编译器或运行时系统可以证明某个锁只能被单个线程访问，因此其他线程不可能进入临界区。这可以通过静态分析或运行时分析得出。
- 编译器或运行时系统可以证明某个锁在某些情况下根本不会被访问，因此可以安全地删除相关的同步操作。

锁消除有助于减少不必要的同步开销，提高程序性能，但需要注意，它必须是安全的，不会导致数据竞争或错误。

**锁粗化（Lock Coarsening）**：

锁粗化是另一种多线程编程中的性能优化技术，它的目标是减少同步操作的次数，从而降低锁的竞争开销。通常情况下，线程在执行临界区代码时会获得锁，然后在离开临界区时释放锁。如果在短时间内多次进入临界区，就会频繁地获得和释放锁，这会导致锁的竞争，降低程序性能。

锁粗化的思想是将多个连续的临界区合并成一个大的临界区，从而减少锁的获取和释放次数。这样可以降低锁的竞争，提高程序的性能。

锁粗化的实现通常需要编译器或运行时系统的支持。它可以通过检测连续的临界区代码，然后将它们合并成一个大的临界区来实现。

总之，锁消除和锁粗化都是用于优化多线程程序性能的技术。锁消除通过消除不必要的同步操作来减少开销，而锁粗化通过合并连续的临界区来减少锁的竞争开销。







