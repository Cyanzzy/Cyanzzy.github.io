---
title: Java 常见基础面试题总结 Ⅱ
date: 2024-02-28 21:33:59
tags: 
  - Java
categories: 
  - Interview
password: zzy   
message: 仅管理员可见
---

# 面向对象基础

## 面向对象和面向过程

- 面向过程把解决问题的过程拆成一个个方法，通过一个个方法的执行解决问题。
- 面向对象会先抽象出对象，然后用对象执行方法的方式解决问题。

 另外，面向对象开发的程序一般更易维护、易复用、易扩展。 

> 面向过程性能比面向对象高吗？

- 面向过程性能比面向对象高。 
- 类调用时需要实例化，开销比较大，比较消耗资源，所以当性能是最重要的考量因素时，比如单片机、嵌入式开发、Linux/Unix等一般采用面向过程开发。 
- 这个并不是根本原因，面向过程也需要分配内存，计算内存偏移量，Java性能差的主要原因并不是因为它是面向对象语言，而是**Java是半编译语言，最终的执行代码并不是可以直接被CPU执行的二进制机械码**。 

- 而面向过程语言大多都是直接编译成机械码在电脑上执行，并且其它一些面向过程的脚本语言性能也并不一定比Java好。 

## 对象实体与对象引用不同

- new 创建对象实例（**对象实例在堆内存中**），对象引用指向对象实例（**对象引用存放在栈内存中**）。

- 一个对象引用可以指向 0 个或 1 个对象，一个对象可以有 n 个引用指向它

## 对象的相等和引用相等的区别

- 对象的相等一般比较的是**内存中存放的内容**是否相等。
- 引用相等一般比较的是他们**指向的内存地址**是否相等。

```java
String str1 = "hello";
String str2 = new String("hello");
String str3 = "hello";
// 使用 == 比较字符串的引用相等
System.out.println(str1 == str2); // false
System.out.println(str1 == str3); // true
// 使用 equals 方法比较字符串的相等
System.out.println(str1.equals(str2)); // true
System.out.println(str1.equals(str3)); // true
```

- `str1` 和 `str2` 不相等，而 `str1` 和 `str3` 相等。这是因为 `==` 运算符比较的是字符串的引用是否相等。
- `str1`、 `str2`、`str3` 三者的内容都相等。这是因为`equals` 方法比较的是字符串的内容，即使这些字符串的对象引用不同，只要它们的内容相等，就认为它们是相等的。

## 类没有声明构造方法，程序能正确执行吗?

* 构造方法是一种特殊的方法，主要作用是完成对象的初始化工作。如果一个类没有声明构造方法，也可以执行。
* 一个类即使没有声明构造方法也**会有默认的无参构造方法**。如果我们自己添加了类的构造方法（无论是否有参），Java 就不会添加默认的无参数的构造方法。
* 在创建对象时后面要加一个括号（因为要调用无参的构造方法）。如果重载有参的构造方法，建议把无参的构造方法也写出来（无论是否用到） 

```java
// 结论：没有显式声明构造参数，但是可以调用默认无参构造函数创建对象
public class Animal {

    private int age;

    private String sex;

    // 没有显式声明构造方法
    
    public static void main(String[] args) {
        Animal a = new Animal();
        // Animal@4554617c
        // 说明有默认无参构造方法
        System.out.println(a);
    }
}
```

```	java
// 虽然显式声明有参构造函数，但未显式声明无参构造函数，调用无参构造函数创建对象编译失败
public class Animal {

    private int age;

    private String sex;

    // 没有显式声明无参构造方法
    
    public Animal(int age, String sex) {
        this.age = age;
        this.sex = sex;
    }

    public static void main(String[] args) {
        // 编译出错
        /*
        Error:(20, 20) java: 无法将类 Animal中的构造器 Animal应用到给定类型;
  		需要: int,java.lang.String
  		找到: 没有参数
  		原因: 实际参数列表和形式参数列表长度不同
        */
        Animal a = new Animal();
     //   Animal b = new Animal(1,"female");

        System.out.println(a);
    //    System.out.println(b);
    }
}
```

```java
// 成功案例
public class Animal {

    private int age;

    private String sex;

    public Animal() {
    }

    public Animal(int age, String sex) {
        this.age = age;
        this.sex = sex;
    }

    public static void main(String[] args) {
        Animal a = new Animal();
        Animal b = new Animal(1,"female");

        // Animal@4554617c
        System.out.println(a);
        // Animal@74a14482
        System.out.println(b);
    }
}
```



## 构造方法特点 && 是否可被 override

> 构造方法特点如下：

- 名字与类名相同 
- 没有返回值，但不能用 void 声明构造函数 
- 生成类的对象时自动执行，无需调用 

构造方法**不能**被 override（重写），但是可以 overload（重载）

## 面向对象三大特征

> 封装

- 封装是指将数据和对数据的操作封装在一个类中，通过访问修饰符来控制对数据的访问权限。 
- 封装可以隐藏类的内部实现细节，只暴露必要的接口给外部使用，提高了代码的安全性和可维护性。  

> 继承

-  继承是指一个类可以继承另一个类的属性和方法 
-  通过继承，子类可以重用父类的代码，并且可以在不修改父类的情况下扩展或修改父类的功能。 
-  继承实现了代码的重用和层次化的组织。 

* 子类拥有父类对象所有的属性和方法（包括私有属性和私有方法），但是父类中的私有属性和方法子类是无法访问，**只是拥有**。子类可以拥有自己属性和方法，即子类可以对父类进行扩展。


> 多态

-  多态是指同一个方法在不同的对象上有不同的行为。 
-  多态通过方法的重写和方法的重载来实现。 
-  方法的重写是指子类可以重写父类的方法，以实现自己的特定行为； 
-  方法的重载是指在一个类中可以定义多个同名但参数列表不同的方法，根据调用时传入的参数类型和个数来确定具体调用哪个方法。 
-  多态提高了代码的灵活性和可扩展性。 

## 接口和抽象类

> 面试版本

**共同点**：

- 两者都不能被实例化
- 两者都可以包含抽象方法
- 两者都可以有默认实现的方法（Java 8 可以用 `default` 关键字在接口中定义默认方法）

**区别**：

- 抽象类可以有构造函数，而接口不可以 
- 一个类只能继承一个抽象类，而一个类可以实现多个接口。
- 抽象类包含抽象方法和具体方法。
- 在Java 8之前，接口不能包含默认方法和静态方法。然而，在Java 8中，接口可以包含默认方法和静态方法。

> 理解版本

 **继承方式：** 

- **抽象类：**一个类只能继承一个抽象类。可以有构造方法，可以有成员变量，可以有普通方法，可以有抽象方法。
- **接口：** 一个类可以实现多个接口。 不允许拥有构造方法，不可以包含成员变量（除了常量），可以有默认方法（Java 8引入）和静态方法。

**构造方法：**

- **抽象类：** 可以有构造方法，用于初始化抽象类的成员变量等。
- **接口：** 不允许有构造方法。接口的成员变量默认是常量，因此它们必须被初始化，而接口没有实例化的过程。 

**成员变量：**

- **抽象类：** 可以包含实例变量。（  实例变量在类中以成员变量的形式声明，但是在方法体外部。它们不使用 `static` 关键字，因此每个对象都有自己的一组实例变量。 ）

  ```	java
  public class MyClass {
      // 实例变量
      int instanceVar1;
      String instanceVar2;
  }
  ```

- **接口：** 可以包含常量（被隐式地指定为 `public static final`）。

**方法：**

- **抽象类：** 可以包含抽象方法和具体方法。
- **接口：** 可以包含抽象方法、默认方法（使用 `default` 关键字，Java 8引入）、静态方法（使用 `static` 关键字，Java 8引入）。

**多继承：**

- **抽象类：** 一个类只能继承一个抽象类，支持单继承。
- **接口：** 一个类可以实现多个接口，支持多继承。 

**用途：**

- **抽象类：** 适用于那些有一些通用的方法，但是其中的一些方法在子类中的实现可能不同的情况。
- **接口：** 用于定义一组相关的抽象方法，不包含具体实现，可以被多个不同类实现，实现了相同接口的类可以用相同的方式进行处理。

## 深拷贝、浅拷贝、引用拷贝

> 图示说明

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B8%B8%E8%A7%81%E5%9F%BA%E7%A1%80-08.png)

> 浅拷贝

浅拷贝会在堆上创建一个新的对象（区别于引用拷贝的一点），**如果原对象内部的属性是引用类型的话，浅拷贝会直接复制内部对象的引用地址**，也就是说拷贝对象和原对象共用同一个内部对象。 

栗子：实现 `Cloneable` 接口，并重写 `clone()` 方法， 直接调用的是父类 `Object` 的 `clone()` 方法。 

```java
public class Address implements Cloneable{
    private String name;
    // 省略构造函数、Getter&Setter方法
    @Override
    public Address clone() {
        try {
            return (Address) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}

public class Person implements Cloneable {
    private Address address;
    // 省略构造函数、Getter&Setter方法
    @Override
    public Person clone() {
        try {
            Person person = (Person) super.clone();
            return person;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}

// 测试
Person person1 = new Person(new Address("武汉"));
Person person1Copy = person1.clone();
// true
System.out.println(person1.getAddress() == person1Copy.getAddress());
```

 从输出结构就可以看出， `person1` 的克隆对象和 `person1` 使用的仍然是同一个 `Address` 对象 

> 深拷贝

深拷贝会完全复制整个对象，包括这个对象所包含的内部对象。	

栗子：修改 `Person` 类的 `clone()` 方法，连带着要把 `Person` 对象内部的 `Address` 对象一起复制。 

```java
@Override
public Person clone() {
    try {
        Person person = (Person) super.clone();
        person.setAddress(person.getAddress().clone());
        return person;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}

// 测试
Person person1 = new Person(new Address("武汉"));
Person person1Copy = person1.clone();
// false
System.out.println(person1.getAddress() == person1Copy.getAddress());
```

 从输出结构就可以看出，显然 `person1` 的克隆对象和 `person1` 包含的 `Address` 对象已经是不同的 

> 引用拷贝

 引用拷贝就是两个不同的引用指向同一个对象 



#  Object

## Object 类的常见方法

 Object 类是一个特殊的类，是所有类的父类。它主要提供了以下 11 个方法： 

```java
/**
 * native 方法，用于返回当前运行时对象的 Class 对象，使用了 final 关键字修饰，故不允许子类重写。
 */
public final native Class<?> getClass()
/**
 * native 方法，用于返回对象的哈希码，主要使用在哈希表中，比如 JDK 中的HashMap。
 */
public native int hashCode()
/**
 * 用于比较 2 个对象的内存地址是否相等，String 类对该方法进行了重写以用于比较字符串的值是否相等。
 */
public boolean equals(Object obj)
/**
 * native 方法，用于创建并返回当前对象的一份拷贝。
 */
protected native Object clone() throws CloneNotSupportedException
/**
 * 返回类的名字实例的哈希码的 16 进制的字符串。建议 Object 所有的子类都重写这个方法。
 */
public String toString()
/**
 * native 方法，并且不能重写。唤醒一个在此对象监视器上等待的线程(监视器相当于就是锁的概念)。如果有多个线程在等待只会任意唤醒一个。
 */
public final native void notify()
/**
 * native 方法，并且不能重写。跟 notify 一样，唯一的区别就是会唤醒在此对象监视器上等待的所有线程，而不是一个线程。
 */
public final native void notifyAll()
/**
 * native方法，并且不能重写。暂停线程的执行。注意：sleep 方法没有释放锁，而 wait 方法释放了锁 ，timeout 是等待时间。
 */
public final native void wait(long timeout) throws InterruptedException
/**
 * 多了 nanos 参数，这个参数表示额外时间（以纳秒为单位，范围是 0-999999）。 所以超时的时间还需要加上 nanos 纳秒。。
 */
public final void wait(long timeout, int nanos) throws InterruptedException
/**
 * 跟之前的2个wait方法一样，只不过该方法一直等待，没有超时时间这个概念
 */
public final void wait() throws InterruptedException
/**
 * 实例被垃圾回收器回收的时候触发的操作
 */
protected void finalize() throws Throwable { } // 调用时机相对不确定
```

## == 和 equals() 的区别

> ==

- 对于基本数据类型来说，`==` 比较的是值。
- 对于引用数据类型来说，`==` 比较的是对象的内存地址

因为 Java 只有值传递，所以对于 == 来说，不管是比较基本数据类型，还是引用数据类型的变量，**其本质比较的都是值**，只是引用类型变量存的值是对象的地址。 

> equals

- **`equals()`** 不能用于判断基本数据类型的变量，**只能用来判断两个对象是否相等**。

- equals 方法存在于  Object  类中，而 Object  类是所有类的直接或间接父类，因此所有的类都有 equals 方法。

 `Object` 类 `equals()` 方法： 

```java
public boolean equals(Object obj) {
     return (this == obj);
}
```

`equals()` 方法存在两种使用情况：

- **类没有重写 `equals()`方法**：通过`equals()`比较该类的两个对象时，等价于通过`==`比较这两个对象，使用的默认是 `Object`类`equals()`方法。
- **类重写 `equals()`方法**：一般我们都重写 `equals()`方法来比较两个对象中的属性是否相等；若它们的属性相等，则返回 true（即，认为这两个对象相等）

```java
String a = new String("ab"); // a 为一个引用
String b = new String("ab"); // b为另一个引用,对象的内容一样
String aa = "ab"; // 放在常量池中
String bb = "ab"; // 从常量池中查找
System.out.println(aa == bb);// true
System.out.println(a == b);// false
System.out.println(a.equals(b));// true
System.out.println(42 == 42.0);// true
```

`String` 中的 `equals` 方法是被重写过的，因为 `Object` 的 `equals` 方法是比较的对象的内存地址，而 `String` 的 `equals` 方法比较的是对象的值。

当创建 `String` 类型的对象时，虚拟机会在常量池中查找有没有已经存在的值和要创建的值相同的对象，如果有就把它赋给当前引用。如果没有就在常量池中重新创建一个 `String` 对象。

 `String`类`equals()`方法： 

```java
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

## hashCode()

 `hashCode()` 的作用是**获取哈希码**（`int` 整数），也称为散列码。这个哈希码的作用是**确定该对象在哈希表中的索引位置**。 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B8%B8%E8%A7%81%E5%9F%BA%E7%A1%80-09.png)

 散列表存储的是键值对(key-value)，它的特点是：**能根据“键”快速的检索出对应的“值”。这其中就利用到了散列码！（可以快速找到所需要的对象）** 

`hashCode()` 定义在 JDK 的  Object  类中，这就意味着 Java 中的任何类都包含有  函数。

- 另外需要注意的是：  Object  的   hashCode()  方法是**本地方法**，也就是用 C 语言或 C++ 实现的。
- 该方法在 **Oracle OpenJDK8** 中默认是 "使用线程局部状态来实现 Marsaglia's xor-shift 随机数生 成", 并不是 "地址" 或者 "地址转换而来"
- 不同 JDK/VM 可能不同在 **Oracle OpenJDK8** 中有六种生成方式中第五种是返回地址），通过添加 VM 参数: -XX:hashCode=4 启用第五种 

```java
public native int hashCode();
```



## 为什么要有 hashCode？

> 以“`HashSet` 如何检查重复”为例子来说明为什么要有 `hashCode`？

* 当你把对象加入 `HashSet` 时，  `HashSet` 会先计算对象的 `hashCode` 值来判断对象加入的位置，  同时也会与其他已经加入的对象的 `hashCode` 值作比较，如果没有相符的 `hashCode`，`HashSet` 会假设对象没有重复出现。  
* 但是如果发现有相同 `hashCode` 值的对象，这时会调用 `equals()` 方法来检查 `hashCode` 相等的对象是否真的相同。  
* 如果两者相同，`HashSet` 就不会让其加入操作成功。如果不同的话，就会重新散列到其他位置。这样我们就大大减少了 `equals` 的次数，相应就大大提高了执行速度。 

* 其实 `hashCode()` 和 `equals()`都是用于比较两个对象是否相等。 

> **那为什么 JDK 还要同时提供这两个方法呢？** 

 因为在一些容器（比如 `HashMap`、`HashSet`）中，有了 `hashCode()` 之后，判断元素是否在对应容器中的效率会更高（参考添加元素进`HashSet`的过程）！ 

 我们在前面也提到了添加元素进`HashSet`的过程，如果 `HashSet` 在对比的时候，同样的 `hashCode` 有多个对 象，它会继续使用 `equals()` 来判断是否真的相同。也就是说 `hashCode` 帮助我们大大缩小了查找成本。 

> **那为什么不只提供 `hashCode()` 方法呢？** 

因为两个对象的`hashCode` 值相等并不代表两个对象就相等。 （**哈希碰撞**） 

>  **那为什么两个对象有相同的 `hashCode` 值，它们也不一定是相等的？** 

 因为 `hashCode()` 所使用的哈希算法也许刚好会让多个对象传回相同的哈希值  （所谓哈希碰撞也就是指的是不同的对象得到相同的 `hashCode` )。 

> 总结

- 如果两个对象的`hashCode` 值相等，那这两个对象不一定相等（哈希碰撞）。

- 如果两个对象的`hashCode` 值相等并且`equals()`方法也返回 `true`，我们才认为这两个对象相等。

- 如果两个对象的`hashCode` 值不相等，我们就可以直接认为这两个对象不相等。

## 为什么重写 equals() 时必须重写 hashCode() 方法？

* 因为两个相等的对象的 `hashCode` 值必须是相等。也就是说如果 `equals` 方法判断两个对象是相等的，那这两个对象的 `hashCode` 值也要相等。

* 如果重写 `equals()` 时没有重写 `hashCode()` 方法的话就可能会导致 `equals` 方法判断是相等的两个对象，`hashCode` 值却不相等。

> **思考**：重写 `equals()` 时没有重写 `hashCode()` 方法的话，使用 `HashMap` 可能会出现什么问题。 

1. **不一致的哈希码：** 如果两个对象在`equals()`方法中被认为是相等的，但它们的`hashCode()`不同，它们将被放置在`HashMap`的不同位置，违反了哈希表的基本原则。
2. **无法正常查找：** 当你尝试在`HashMap`中使用一个与之前放入的键相等的键进行查找时，由于哈希码不同，`HashMap`可能无法正确地定位到该键，导致查找失败。

> **总结**：

- `equals` 方法判断两个对象是相等的，那这两个对象的 `hashCode` 值也要相等。
- 两个对象有相同的 `hashCode` 值，他们也不一定是相等的（哈希碰撞）。



# String

## String、StringBuffer、StringBuilder 区别

> 可变性

* `String` 是不可变的，`String` 类中使用 `final` 关键字修饰字符数组来保存字符串

* `StringBuilder` 与 `StringBuffer` 都继承自 `AbstractStringBuilder` 类，在 `AbstractStringBuilder` 中也是使用字符数组保存字符串，不过没有使用 `final` 和 `private` 关键字修饰 

  最关键的是这个 `AbstractStringBuilder` 类还提供了很多修改字符串的方法比如 `append` 方法。 （请看）

```java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    char[] value;
    public AbstractStringBuilder append(String str) {
        if (str == null)
            return appendNull();
        int len = str.length();
        ensureCapacityInternal(count + len);
        str.getChars(0, len, value, count);
        count += len;
        return this;
    }
  	//...
}
```



> 线程安全

*  `String` 中的对象是不可变的，线程安全                                                                           
*  `StringBuffer` 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。
*  `StringBuilder` 并没有对方法进行加同步锁，所以是非线程安全的。 

> 性能

*  每次对 `String` 类型进行改变的时候，都会生成一个新的 `String` 对象，然后将指针指向新的 `String` 对象。 

*  `StringBuffer` 每次都会对 `StringBuffer` 对象本身进行操作，而不是生成新的对象并改变对象引用。 

*  相同情况下使用 `StringBuilder` 相比使用 `StringBuffer` 仅能获得 10%~15% 左右的性能提升，但却要冒多线程不安全的风险。 

> 总结

1. 操作少量的数据: 适用 `String`
2. 单线程操作字符串缓冲区下操作大量数据： 适用 `StringBuilder`
3. 多线程操作字符串缓冲区下操作大量数据：适用 `StringBuffer`

## String 不可变的原因

 `String` 类中使用 `final` 关键字修饰字符数组来保存字符串，所以`String` 对象是不可变的。 

```java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
    private final char value[];
	//...
}
```

> 注意：

- 被 `final` 关键字修饰的类不能被继承
- 被 `final` 关键字修饰的方法不能被重写
- 被 `final` 关键字修饰的变量是基本数据类型则值不能改变
- 被 `final` 关键字修饰的变量是引用类型则不能再指向其他对象

但 `final` 关键字修饰的数组保存字符串并不是 `String` 不可变的根本原因，**这个数组保存的字符串是可变的**（`final` 修饰引用类型变量的情况）。 

> **`String` 真正不可变有下面几点原因：** 

1. **保存字符串的数组被** `final` **修饰且为私有的，并且**`String` **类没有提供/暴露修改这个字符串的方法。**
1. `String` **类被** `final` **修饰导致其不能被继承，进而避免了子类破坏** `String` **不可变**。

 相关阅读：[如何理解 String 类型值的不可变？ - 知乎提问](https://www.zhihu.com/question/20618891/answer/114125846) 

> Java 9 的 String

 在 Java 9 之后，`String`、`StringBuilder` 与 `StringBuffer` 的实现改用 `byte` 数组存储字符串。 

````java
public final class String implements java.io.Serializable,Comparable<String>, CharSequence {
    // @Stable 注解表示变量最多被修改一次，称为“稳定的”。
    @Stable
    private final byte[] value;
}

abstract class AbstractStringBuilder implements Appendable, CharSequence {
    byte[] value;

}
````

 **Java 9 为何要将 `String` 的底层实现由 `char[]` 改成了 `byte[]` ?** 

-  新版的 String 其实支持两个编码方案：Latin-1 和 UTF-16。 
-  如果字符串中包含的汉字没有超过 Latin-1 可表示范围内的字符，那就会使用 Latin-1 作为编码方案。 
-  Latin-1 编码方案下，`byte` 占一个字节(8 位)，`char` 占用 2 个字节（16），`byte` 相较 `char` 节省一半的内存空间。 
-  JDK 官方就说了绝大部分字符串对象只包含 Latin-1 可表示的字符。 
-  如果字符串中包含的汉字超过 Latin-1 可表示范围内的字符，`byte` 和 `char` 所占用的空间是一样的。 

## 字符串拼接用“+”  OR "StringBuilder"

> 字符串对象通过 `+` 的字符串拼接方式，实际上是通过 `StringBuilder` 调用 `append()` 方法实现的，拼接完成之后调用 `toString()` 得到一个 `String` 对象 。 

```java
String str1 = "he";
String str2 = "llo";
String str3 = "world";
String str4 = str1 + str2 + str3;
```

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B8%B8%E8%A7%81%E5%9F%BA%E7%A1%80-10.png)

> 在循环内使用 `+` 进行字符串的拼接，存在比较明显的缺陷：**编译器不会创建单个 `StringBuilder` 以复用，会导致创建过多的 `StringBuilder` 对象**。 

```java
String[] arr = {"he", "llo", "world"};
String s = "";
for (int i = 0; i < arr.length; i++) {
    s += arr[i];
}
System.out.println(s);

```

 `StringBuilder` 对象是在循环内部被创建的，这意味着每循环一次就会创建一个 `StringBuilder` 对象。 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B8%B8%E8%A7%81%E5%9F%BA%E7%A1%80-11.png)

 如果直接使用 `StringBuilder` 对象进行字符串拼接的话，就不会存在这个问题了。 

```java
String[] arr = {"he", "llo", "world"};
StringBuilder s = new StringBuilder();
for (String value : arr) {
    s.append(value);
}
System.out.println(s);

```

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B8%B8%E8%A7%81%E5%9F%BA%E7%A1%80-12.png)

 不过，使用 `+` 进行字符串拼接会产生大量的临时对象的问题在 JDK9 中得到了解决。  在 JDK9 当中，字符串相加 `+` 改为了用动态方法 `makeConcatWithConstants()` 来实现，而不是大量的 `StringBuilder` 了 

[ 还在无脑用StringBuilder？来重温一下字符串拼接吧](https://juejin.cn/post/7182872058743750715)

## String#equals() 和 Object#equals()

-  `String` 中的 `equals` 方法是被重写过的，比较的是 String 字符串的值是否相等
-  `Object` 的 `equals` 方法是比较的对象的内存地址 

## 字符串常量池 

 **字符串常量池** 是 JVM 为了提升性能和减少内存消耗针对字符串（String 类）专门开辟的一块区域，**主要目的是为了避免字符串的重复创建。** 

```java
// 在堆中创建字符串对象”ab“
// 将字符串对象”ab“的引用保存在字符串常量池中
String aa = "ab";
// 直接返回字符串常量池中字符串对象”ab“的引用
String bb = "ab";
System.out.println(aa==bb);// true

```



##  String s1 = new String("abc");（重点）

 答：会创建 1 或 2 个字符串对象。 

> 如果字符串常量池中**不存在**字符串对象 `“abc”`的引用，那么它会在堆上创建**两个**字符串对象，其中一个字符串对象的引用会被保存在字符串常量池中。 

```java
String s1 = new String("abc");
```

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B8%B8%E8%A7%81%E5%9F%BA%E7%A1%80-13.png)

 `ldc` 命令用于判断字符串常量池中是否保存了对应的字符串对象的引用

- 如果保存了的话直接返回
- 如果没有保存的话，会在堆中创建对应的字符串对象并将该字符串对象的引用保存到字符串常量池中 

> 如果字符串常量池中**已存在**字符串对象 `“abc”` 的引用，则只会在堆中创建 **1 个**字符串对象“abc”。 

```java
// 字符串常量池中已存在字符串对象“abc”的引用
String s1 = "abc";
// 下面这段代码只会在堆中创建 1 个字符串对象“abc”
String s2 = new String("abc");

```

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B8%B8%E8%A7%81%E5%9F%BA%E7%A1%80-14.png)

- 位置 7 的 `ldc` 命令不会在堆中创建新的字符串对象“abc”，  因为位置 0 已经执行了一次 `ldc` 命令 ，已经在堆中创建过一次字符串对象“abc”了
- 位置 7 执行 `ldc` 命令会直接返回字符串常量池中字符串对象“abc”对应的引用 

## String#intern 方法（重点）

`String.intern()` 是一个 native 方法，其作用是将**指定的字符串对象的引用保存在字符串常量池中**

- 如果字符串常量池中**已经保存了**对应的字符串对象的引用，就直接返回该引用。
- 如果字符串常量池中**没有保存**对应的字符串对象的引用，那就在常量池中创建一个指向该字符串对象的引用并返回。

```java
// 在堆中创建字符串对象”Java“
// 将字符串对象”Java“的引用保存在字符串常量池中
String s1 = "Java";
// 直接返回字符串常量池中字符串对象”Java“对应的引用
String s2 = s1.intern();
// 会在堆中在单独创建一个字符串对象
String s3 = new String("Java");
// 直接返回字符串常量池中字符串对象”Java“对应的引用
String s4 = s3.intern();
// s1 和 s2 指向的是堆中的同一个对象
System.out.println(s1 == s2); // true
// s3 和 s4 指向的是堆中不同的对象
System.out.println(s3 == s4); // false
// s1 和 s4 指向的是堆中的同一个对象
System.out.println(s1 == s4); //true

```

## String 类型的变量和常量做 `+` 运算时发生了什么？

 先来看字符串不加 `final` 关键字拼接的情况（JDK1.8）： 

```	java
String str1 = "str";
String str2 = "ing";
String str3 = "str" + "ing";
String str4 = str1 + str2;
String str5 = "string";
System.out.println(str3 == str4);//false
System.out.println(str3 == str5);//true
System.out.println(str4 == str5);//false

```

 **注意**：比较 String 字符串的值是否相等，可以使用 `equals()` 方法。  `String` 中的 `equals` 方法是被重写过的。  `Object` 的 `equals` 方法是比较的对象的内存地址，而 `String` 的 `equals` 方法比较的是字符串的值是否相等 



> **对于编译期可以确定值的字符串，也就是常量字符串 ，jvm 会将其存入字符串常量池。**
>
> **并且，字符串常量拼接得到的字符串常量在编译阶段就已经被存放字符串常量池，这个得益于编译器的优化。** 

 在编译过程中，Javac 编译器（下文中统称为编译器）会进行一个叫做 **常量折叠（Constant Folding）** 的代码优化。

 常量折叠会把**常量表达式**的值求出来作为常量嵌在最终生成的代码中，这是 Javac 编译器会对源代码做的极少量优化措施之一（代码优化几乎都在即时编译器中进行）

 对于 `String str3 = "str" + "ing";` 编译器会给你优化成 `String str3 = "string";` 。 

 并不是所有的常量都会进行折叠，只有编译器在程序**编译期**就可以确定值的常量才可以： 

- 基本数据类型（`byte`、`boolean`、`short`、`char`、`int`、`float`、`long`、`double`）以及字符串常量。
- `final` 修饰的基本数据类型和字符串变量

- 字符串通过 `+` 拼接得到的字符串、基本数据类型之间算数运算（加减乘除）、基本数据类型的位运算（<<、>>、>>> ）

> **引用的值在程序编译期是无法确定的，编译器无法对其进行优化。** 

 对象引用和 `+` 的字符串拼接方式，实际上是通过 `StringBuilder` 调用 `append()` 方法实现的，拼接完成之后调用 `toString()` 得到一个 `String` 对象 。 

```java
// String str4 = str1 + str2;
String str4 = new StringBuilder().append(str1).append(str2).toString();

```

字符串使用 `final` 关键字声明之后，可以让编译器当做常量来处理。 

```java
final String str1 = "str";
final String str2 = "ing";
// 下面两个表达式其实是等价的
String c = "str" + "ing";// 常量池中的对象
String d = str1 + str2; // 常量池中的对象
System.out.println(c == d);// true

```

 被 `final` 关键字修饰之后的 `String` 会被编译器当做常量来处理，编译器在程序编译期就可以确定它的值，其效果就相当于访问常量。 

>  如果编译器在运行时才能知道其确切值的话，就无法对其优化。 

 `str2` 在运行时才能确定其值 ：

```java
final String str1 = "str";
final String str2 = getStr();
String c = "str" + "ing";// 常量池中的对象
String d = str1 + str2; // 在堆上创建的新的对象
System.out.println(c == d);// false
public static String getStr() {
      return "ing";
}
```



