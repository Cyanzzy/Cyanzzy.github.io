---
title: JVM 内存与垃圾回收篇-7-StringTable
date: 2023-08-15 09:39:35
tags: 
  - JVM
categories: 
  - Language
---
# String的基本特性

* String：字符串，使用一对`""`引起来表示

  ```java
  String s1 = "cyan"; // 字面量定义
  String s2 = new String("cyan")
  ```

* String 声明为final的，不可被继承

* String 实现了Serializable接口：表示字符串是支持序列化的

* String 实现了Comparable接口：表示string可以比较大小

* String 在jdk8及以前内部定义了final char[ ] value用于存储字符串数据。jdk9时改为byte[]

  ![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-jvm-20230802-92.jpg)

* String：代表不可变的字符序列，即不可变性。

  * 当对字符串重新赋值时，需要重写指定内存区域赋值，不能使用原有的value进行赋值
  * 当对现有的字符串进行连接操作时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值
  * 当调用String的`replace ()`方法修改指定字符或字符串时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值

* 通过字面量的方式（区别于new）给一个字符串赋值，此时的字符串值声明在字符串常量池中 

> 字符串常量池中是不会存储相同内容的字符串的 

* String的String Pool是一个固定大小的Hashtable，默认值大小长度是1009。如果放进string Pool的String非常多，就会造成Hash冲突严重，从而导致链表会很长，而链表长了后直接会造成的影响就是当调String.intern时性能会大幅下降
* 使用`-XX:StringTablesize`可设置stringTable的长度
* 在jdk6中StringTable是固定的，就是1009的长度，所以如果常量池中的字符串过多就会导致效率下降很快。StringTablesize设置没有要求
* 在jdk7中StringTable的长度默认值是60013，1009是可设置的最小值，StringTablesize设置没有要求
* jdk8开始，设置StringTable的长度，1009是可设置的最小值

# String的内存分配

* 在Java语言中有8种基本数据类型和一种比较特殊的类型String。这些类型为了使它们在运行过程中速度更快、更节省内存，都提供了一种常量池。
* 常量池类似一个Java系统级别提供的缓存。8种基本数据类型的常量池都是系统协调的，String类型的常量池比较特殊 
  * 直接使用双引号声明出来的String对象会直接存储在常量池中 
  * 不是用双引号声明的String对象，可以使用String提供的intern()方法 

* Java 6及以前，字符串常量池存放在永久代。
* Java 7 中将字符串常量池的位置调整到Java堆内，所有的字符串都保存在堆（Heap）中，和其他普通对象一样，可以在进行调优应用时仅需要调整堆大小
* Java8元空间，字符串常量在堆

> StringTable调整原因

* permSize默认比较小
* 永久代垃圾回收频率低

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-jvm-20230802-70.jpg)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-jvm-20230802-71.jpg)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-jvm-20230802-72.jpg)



# 字符串拼接操作

* 常量与常量的拼接结果在常量池，原理是编译期优化
* 常量池中不会存在相同内容的常量
* 只要其中有一个是变量，结果就在堆中。变量拼接的原理是StringBuilder
* 如果拼接的结果调用`intern()`方法，则主动将常量池中还没有的字符串对象放入池中，并返回此对象地址 



![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-jvm-20230802-93.jpg)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-jvm-20230802-94.jpg)

# intern() 的使用

* Jdk1.6中，将这个字符串对象尝试放入串池
  * 如果串池中有，则并不会放入。返回已有的串池中的对象的地址
  * 如果没有，会**把此对象复制一份**，放入串池，并返回串池中的对象地址
* Jdk1.7起，将这个字符串对象尝试放入串池 
  * 如果串池中有，则并不会放入。**返回已有的串池中的对象的地址**
  * 如果没有，则会把**对象的引用地址复制一份**，放入串池，并返回串池中的引用地址

## 理解 intern()

* 如果**不是**用双引号声明的String对象，可以使用String提供的`intern`方法:`intern`**方法会从字符串常量池中查询当前字符串是否存在，若不存在就会将当前字符串放入常量池中。**

  ```java
  String myInfo = new String("I love China" ).intern () ;
  ```

* 如果在任意字符串上调用`String.intern`方法，那么其返回结果所指向的那个类实例，**必须和直接以常量形式出现的字符串实例完全相同**。
* Interned String就是确保字符串在内存里只有一份拷贝，这样可以节约内存空间，加快字符串操作任务的执行速度。这个值会被存放在字符串内部池（String Intern Pool）



> 保证变量s指向的是字符串常量池中的数据

* 字面量定义

  ```java
  String s = "cyan";
  ```

* 调用`intern()`

  ```java
  String s =  new String("cyan").intern();
  // or
  String s =  new StringBuilder("cyan").toString().intern();
  ```

## new String() 创建了几个对象

> `new String("ab")`会创建几个对象？（字符串常量池中会生成"ab"） 

1. `new` 关键字在堆空间创建的
2. 字符串常量池中的对象。（字节码指令`ldc`）

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-jvm-20230802-95.jpg)



> `new String("a") + new String("b")`会创建几个对象？ （字符串常量池中不会生成"ab"）

1. `new StringBuilder()`
2. `new String("a")`
3. 常量池中的`"a"`
4. `new String("b")`
5. 常量池中的`"b"`
6. 深入分析`StringBuilder().toString()`对应字节码`new String("ab")`，强调一下，**`toString()`的调用，在字符串常量池中没有生成"ab"**

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-jvm-20230802-96.jpg)

> intern()的使用：jdk6 vs jdk7/8

调用下面`s3.intern()`时，由于`String s3 = new String("1") + new String("1")`生成的`“11”`并不在字符串常量池中，对于jdk6和7处理方法不同，详见[intern()的使用]()

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-jvm-20230802-97.jpg)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-jvm-20230802-98.jpg)

> 拓展练习

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-jvm-20230802-99.jpg)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-jvm-20230802-100.jpg)

> 强化练习1

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-jvm-20230802-102.jpg)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-jvm-20230802-101.jpg)



> 强化练习2

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-jvm-20230802-103.jpg)

# StringTable 的垃圾回收

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-jvm-20230802-104.jpg)



# G1中的String去重操作

* 当垃圾收集器工作时会访问堆上存活的对象。对每一个访问的对象都会检查是否是候选的要去重的string对象
* 如果是，把这个对象的一个引用插入到队列中等待后续的处理。一个去重的线程在后台运行，处理这个队列。处理队列的一个元素意味着从队列删除这个元素，然后尝试去重它引用的String对象
* 使用一个hashtable来记录所有的被String对象使用的不重复的char数组。当去重的时候，会查这个hashtable，来看堆上是否已经存在一个一模一样的char数组。
* 如果存在，String对象会被调整引用那个数组，释放对原来的数组的引用，最
  终会被垃圾收集器回收掉。
* 如果查找失败，char数组会被插入到hashtable，这样以后的时候就可以共
  享这个数组了。

| 命令行                                    | 说明                                           |
| ----------------------------------------- | ---------------------------------------------- |
| UseStringDeduplication(bool)              | 开启String去重，默认是不开启的，需要手动开启   |
| PrintStringDeduplicationStatistics (bool) | 打印详细的去重统计信息                         |
| StringDeduplicationAgeThreshold (uintx)   | 达到这个年龄的String对象被认为是去重的候选对象 |

