---
title: Java 常见基础面试题总结 Ⅰ
date: 2024-02-28 18:47:38
tags: 
  - Java
categories: 
  - Interview
password: zzy   
message: 仅管理员可见
---

# 基础常识
 
## Java 语言特点

1. 面向对象特性，包括封装，继承，多态
2. JVM 实现平台无关性，即跨平台
3. 多线程和网络编程特性
4. 编译与解释并存特性
5. Java 安全可靠特性，包括多重安全防护机制、异常处理和自动内存管理机制

## Java SE V.S. Java EE

> Java SE，即 Java 平台标准版

- 具有应用程序开发和运行的核心类库以及虚拟机等核心组件
- 用于构建桌面应用程序或简单的服务器应用程序 

> Java EE，即 Java 平台企业版

- 在 Java SE 基础上包含支持企业级应用程序开发和部署的标准和规范（比如 Servlet、JSP、EJB、JDBC、JPA、JTA、JavaMail、JMS）
- 用于构建分布式、可移植、健壮、可伸缩和安全的服务端 Java 应用程序（比如 Web 应用程序）

> 面试版本

- Java SE 是 Java 的基础版本，Java EE 是 Java 的高级版本
- Java SE 更适合开发桌面应用程序或简单的服务器应用程序，Java EE 更适合开发复杂的企业级应用程序或 Web 应用程序

## JVM V.S. JDK V.S. JRE

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B8%B8%E8%A7%81%E5%9F%BA%E7%A1%80-01.png)

> JVM

JVM 是 Java 程序的运行环境，它负责将 Java 源代码编译为字节码，并在运行时负责执行这些字节码。JVM 提供内存管理、垃圾回收、线程管理等运行时支持，使得 Java 程序具有跨平台性

**JVM 并不是只有一种！只要满足 JVM 规范，每个公司、组织或者个人都可以开发自己的专属 JVM** 。除我们平时最常用的 HotSpot VM 外，还有 J9 VM、Zing VM、JRockit VM 等 JVM 。 

> JDK

JDK 包含 JRE，同时还包括 javadoc（文档注释工具）、jdb（调试器）、jconsole（基于 JMX 的可视化监控⼯具）、javap（反编译工具）等工具，可以用于 Java 应用程序的开发和调试 

> JRE

JRE 是 Java 运行时环境，仅包含 Java 应用程序的运行时环境和必要的类库 

## 字节码


Java 源代码被编译成一种称为字节码（Bytecode）的中间代码。字节码是一种与平台无关的、由虚拟机执行的二进制代码。

* 通过字节码，Java 一定程度上解决传统解释型语言执行效率低的问题，同时又保留解释型语言可移植的特点

* 由于字节码并不针对某种特定的机器，Java 程序无须重新编译便可在多种不同操作系统的计算机上运行

> JDK、JRE、JVM、JIT 关系图

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B8%B8%E8%A7%81%E5%9F%BA%E7%A1%80-02.png)

> Java 程序运行过程

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B8%B8%E8%A7%81%E5%9F%BA%E7%A1%80-03.png)

JVM 先加载字节码文件，然后通过解释器执行，该方式执行速度稍慢。JIT（Just-In-Time）编译器是Java虚拟机（JVM）的一部分，用于将Java字节码转换为本地机器码，以提高程序的执行速度。
JIT 编译器的工作方式是在程序运行的过程中，将热点代码（经常被执行的代码）编译成本地机器码，从而避免每次都解释执行字节码。
![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B8%B8%E8%A7%81%E5%9F%BA%E7%A1%80-04.png)

## 编译与解释并存

> 高级编程语言程序执行方式

- **编译型**：编译型语言会通过编译器将源代码一次性翻译成可被该平台执行的机器码。一般情况下，编译语言的**执行速度比较快，开发效率比较低**。常见的编译性语言有 C、C++、Go、Rust 等
- **解释型**：解释型语言会通过解释器一句一句的将代码解释为机器代码后再执行。解释型语言**开发效率比较快，执行速度比较慢**。常见的解释性语言有 Python、JavaScript、PHP 等

> Java 编译与解释并存

- Java 语言既具有编译型语言的特征，也具有解释型语言的特征

- **编译过程：**Java 代码首先通过编译器（javac）编译成字节码，而字节码是一种中间代码，不是针对特定平台的机器代码。这个编译过程发生在开发阶段。

- **解释执行：** 字节码由 JVM 解释执行，解释器逐行解释字节码，并在运行时将其翻译成特定平台的机器代码执行，保证 Java 可以在任何支持 JVM 的平台上运行，实现跨平台性

JIT 编译器与解释器并存。初始阶段，解释器执行字节码，同时收集性能数据。当某些代码段被多次执行，JIT 编译器将热点代码进行即时编译，生成本地机器代码。以后再次执行这些代码时，直接执行本地机器代码，提高了性能。

## Java 和 C++ 的区别

Java 和 C++ 都是面向对象的语言，都支持封装、继承和多态 ：

* Java 不提供指针来直接访问内存，程序内存更加安全
* Java 仅支持单继承，C++ 支持多重继承。虽然 Java 的类不可以多继承，但可以通过接口实现多继承
* Java 有自动内存管理垃圾回收机制，无需手动释放无用内存
* C ++ 同时支持方法重载和操作符重载，但是 Java 只支持方法重载

## Java 9：AOT 

-  JDK 9 引入新的编译模式 **AOT(Ahead of Time Compilation)**  
-  与 JIT 不同的是，AOT 编译模式会在程序执行前就将其编译成机器码，属于静态编译（C、 C++，Rust，Go 等语言就是静态编译）
-  AOT 避免 JIT 预热等各方面的开销，可以提高 Java 程序的启动速度，避免预热时间长
-  并且 AOT 还能减少内存占用和增强 Java 程序的安全性（AOT 编译后的代码不容易被反编译和修改），特别适合云原生场景

> JIT 与 AOT

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B8%B8%E8%A7%81%E5%9F%BA%E7%A1%80-05.png)

* AOT 优势在于启动时间、内存占用和打包体积
* JIT 优势在于具备更高的极限处理能力，可以降低请求的最大延迟

> **既然 AOT 这么多优点，那为什么不全部使用这种编译方式呢？** 

- 只能说 AOT 更适合当下的云原生场景，对微服务架构的支持也比较友好。
- 除此之外，AOT 编译无法支持 Java 的一些动态特性，如反射、动态代理、动态加载、JNI（Java Native Interface）等。但很多框架和库（如 Spring、CGLIB）都用到了这些特性。如果只使用 AOT 编译，那就没办法使用这些框架和库了，或者说需要针对性地去做适配和优化。

比如，CGLIB 动态代理使用 ASM 技术，大致原理是运行时直接在内存中生成并加载修改后的字节码文件。如果全部使用 AOT 提前编译，就不能使用 ASM 技术了。为支持类似的动态特性，因此选择使用 JIT 

# 基本语法

## 注释形式

1. **单行注释**：用于解释方法内单行代码
2. **多行注释**：用于解释一段代码
3. **文档注释**：用于生成 Java 开发文档

## 标识符和关键字的区别

- 标识符是程序中用于标识变量、函数、类、方法或其他用户自定义实体的名称
- 关键字是编程语言中具有特殊含义的预定义词汇。这些词汇被用于表示语法结构、控制流程、声明变量等。关键字不能用作标识符，因为它们已经在语言中有了特殊用途


> 标识符可以是由**字母、数字、下划线**（_）**和美元符号**（$）组成的字符序列，但必须遵循一些规则

1. 可以包含字母、数字、下划线(_)和美元符号($)。
2. 标识符不能以数字开头。
3. 不能使用Java关键字作为标识符。例如，你不能将关键字 `int` 作为标识符。
4. 标识符是区分大小写的。`myVariable` 和 `myvariable` 被视为不同的标识符。
5. Java建议使用驼峰命名法，即在多个单词之间使用大写字母来提高可读性。例如，`myVariableName`。

## Java 语言关键字

| 分类                 | 关键字   |            |          |              |            |           |        |
| :------------------- | -------- | ---------- | -------- | ------------ | ---------- | --------- | ------ |
| 访问控制             | private  | protected  | public   |              |            |           |        |
| 类，方法和变量修饰符 | abstract | class      | extends  | final        | implements | interface | native |
|                      | new      | static     | strictfp | synchronized | transient  | volatile  | enum   |
| 程序控制             | break    | continue   | return   | do           | while      | if        | else   |
|                      | for      | instanceof | switch   | case         | default    | assert    |        |
| 错误处理             | try      | catch      | throw    | throws       | finally    |           |        |
| 包相关               | import   | package    |          |              |            |           |        |
| 基本类型             | boolean  | byte       | char     | double       | float      | int       | long   |
|                      | short    |            |          |              |            |           |        |
| 变量引用             | super    | this       | void     |              |            |           |        |
| 保留字               | goto     | const      |          |              |            |           |        |

> default 说明

- 在程序控制中，当在 `switch` 中匹配不到任何情况时，可以使用 `default` 来编写默认匹配的情况
- 在类、方法和变量修饰符中，Java 8 开始引入默认方法，可以使用 `default` 关键字来定义一个方法的默认实现
- 在访问控制中，如果方法前没有任何修饰符，这个方法就具有默认（包级别）的访问控制。默认访问控制意味着该方法只能被同一包中的其他类访问，而对于其他包中的类来说是不可见的


##  移位运算符

```java
// HashMap（JDK1.8） 中的 hash 方法的源码就用到了移位运算符：
static final int hash(Object key) {
    int h;
    // key.hashCode()：返回散列值也就是hashcode
    // ^：按位异或
    // >>>:无符号右移，忽略符号位，空位都以0补齐
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

* `<<`：左移运算符，向左移若干位，高位丢弃，低位补零。`x << 1`，相当于 x 乘以 $2^1$(不溢出的情况下)。

* `>>` ：带符号右移，向右移若干位，高位补符号位，低位丢弃。正数高位补 0，负数高位补 1。`x >> 1`，相当于 x 除以  $2^1$ 

* `>>>` ：无符号右移，忽略符号位，空位都以 0 补齐。

由于 `double`，`float` 在二进制中的表现比较特殊，因此不能来进行移位操作。

移位操作符实际上支持的类型只有`int`和`long`，编译器在对`short`、`byte`、`char`类型进行移位前，都会将其转换为`int`类型再操作。

> **如果移位的位数超过数值所占有的位数会怎样？**

当 int 类型左移/右移位数大于等于 32 位操作时，会先求余（**%**）后再进行左移/右移操作，也就是说左移/右移 32 位相当于不进行移位操作（32%32=0），左移/右移 42 位相当于左移/右移 10 位（42%32=10）。

当 long 类型进行左移/右移操作时，由于 long 对应的二进制是 64 位，因此求余操作的基数也变成了 64。

也就是说：`x<<42`等同于`x<<10`，`x>>42`等同于`x>>10`，`x >>>42`等同于`x >>> 10`

## continue、break 和 return 的区别

* `continue`：指跳出当前的这一次循环，继续下一次循环

* `break`：指跳出整个循环体，继续执行循环下面的语句

`return` 用于跳出所在方法，结束该方法的运行。return 一般有两种用法：

1. `return;`：直接使用 return 结束方法执行，用于没有返回值函数的方法
2. `return value;`：return 一个特定值，用于有返回值函数的方法

# 基本数据类型

## Java 基本数据类型

- 数字类型：`byte`、`short`、`int`、`long`、`float`、`double`
- 字符类型：`char`
- 布尔型：`boolean`。

| **基本类型** | **位数** | 默认值  | 取值范围                          |
| ------------ | -------- | ------- | --------------------------------- |
| byte         | 8        | 0       | -2^7 ~ 2^7 - 1                  |
| short        | 16       | 0       | -2^15 ~ 2^15 - 1                |
| int          | 32       | 0       | -2^31 ~ 2^31 - 1                |
| long         | 64       | 0L      | -2^63 ~ 2^63 - 1                |
| char         | 16       | 'u0000' | 0 ~ 65535（2^16 - 1）            |
| float        | 32       | 0f      | 1.4E-45 ~ 3.4028235E38            |
| double       | 64       | 0d      | 4.9E-324 ~ 1.7976931348623157E308 |
| boolean      | 1        | false   | true、false                       |

## 基本类型和包装类型的区别

**用途**：

- 除定义一些常量和局部变量外，在其他地方比如方法参数、对象属性中很少会使用基本类型来定义变量
- 包装类型可用于泛型，而基本类型不可以 

**存储方式**：

- 基本数据类型的局部变量存放在 Java 虚拟机栈中的**局部变量表**中，基本数据类型的成员变量（未被 `static` 修饰 ）存放在 Java 虚拟机的**堆**中。
- 包装类型属于对象类型，我们知道几乎所有对象实例都存在于堆中。 

**占用空间**：

- 相比于包装类型（对象类型）， 基本数据类型占用的空间往往非常小。

**默认值**：

- 成员变量包装类型不赋值就是 `null` 
- 基本类型有默认值且不是 `null`。

**比较方式**：

- 对于基本数据类型来说，`==` 比较的是值。
- 对于包装数据类型来说，`==` 比较的是对象的内存地址。
- 所有整型包装类对象之间值的比较，全部使用 `equals()` 方法。

> **为什么说是几乎所有对象实例都存在于堆中呢？** 

HotSpot 虚拟机引入 JIT 优化后，会对对象进行逃逸分析，如果发现某一个对象并没有逃逸到方法外部，那么就可能通过标量替换来实现栈上分配，而避免堆上分配内存

>  **基本数据类型存放在栈中是一个常见的误区！** 

基本数据类型的成员变量如果没有被 `static` 修饰的话，就存放在堆中。 

（不建议这么使用，应该要使用基本数据类型对应的包装类型）

```java
class BasicTypeVar{
  private int x;
}
```

## 包装类型的缓存机制

Java 基本数据类型的包装类型的大部分都用到了缓存机制来提升性能。 **如果超出对应范围仍然会去创建新的对象，缓存的范围区间的大小只是在性能和资源之间的权衡。**

-  `Byte`，`Short`，`Integer`，`Long` 4 种包装类默认创建数值 **[-128，127]** 的相应类型的缓存数据
-  `Character` 创建了数值在 **[0,127]** 范围的缓存数据
-  `Boolean` 直接返回 `True` or `False`。 

**Integer 缓存源码：** 

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static {
        // high value may be configured by property
        int h = 127;
    }
}
```

**Character 缓存源码:** 

```java
public static Character valueOf(char c) {
    if (c <= 127) { // must cache
      return CharacterCache.cache[(int)c];
    }
    return new Character(c);
}

private static class CharacterCache {
    private CharacterCache(){}
    static final Character cache[] = new Character[127 + 1];
    static {
        for (int i = 0; i < cache.length; i++)
            cache[i] = new Character((char)i);
    }

}
```

**Boolean 缓存源码：** 

```java
public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}
```

**两种浮点数类型的包装类 `Float` 和`Double` 并没有实现缓存机制。**

```java
Integer i1 = 33;
Integer i2 = 33;
System.out.println(i1 == i2);// 输出 true

Float i11 = 333f;
Float i22 = 333f;
System.out.println(i11 == i22);// 输出 false

Double i3 = 1.2;
Double i4 = 1.2;
System.out.println(i3 == i4);// 输出 false
```

> 下面的代码的输出结果是 `true` 还是 `false` 呢？ 

```java
Integer i1 = 40;
Integer i2 = new Integer(40);
System.out.println(i1==i2); // false
```

`Integer i1=40` 这一行代码会发生装箱，也就是说这行代码等价于 ` Integer i1=Integer.valueOf(40) ` 。因此， `i1`直接使用的是缓存中的对象。而 ` Integer i2 = new Integer(40) `会直接创建新的对象。

 **所有整型包装类对象之间值的比较，全部使用 equals 方法比较**。 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B8%B8%E8%A7%81%E5%9F%BA%E7%A1%80-06.png)

## 自动装箱与拆箱

- **装箱**：将基本类型用它们对应的引用类型包装起来；
- **拆箱**：将包装类型转换为基本数据类型；

```java
Integer i = 10;  //装箱
int n = i;   //拆箱
```
![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B8%B8%E8%A7%81%E5%9F%BA%E7%A1%80-07.png)

 从字节码中，我们发现装箱其实就是调用了 包装类的`valueOf()`方法，拆箱其实就是调用了 `xxxValue()`方法。 

- `Integer i = 10` 等价于 `Integer i = Integer.valueOf(10)`
- `int n = i` 等价于 `int n = i.intValue()`;

 注意：**如果频繁拆装箱的话，也会严重影响系统的性能。我们应该尽量避免不必要的拆装箱操作。** 

## 浮点数运算精度丢失问题

```java
float a = 2.0f - 1.9f;
float b = 1.8f - 1.7f;
System.out.println(a);// 0.100000024
System.out.println(b);// 0.099999905
System.out.println(a == b);// false
```

> 浮点数运算的精度丢失问题原因

计算机在表示一个数字时，宽度是有限的，无限循环的小数存储在计算机时，只能被截断，所以就会导致小数精度发生损失的情况 

```java
// 0.2 转换为二进制数的过程为，不断乘以 2，直到不存在小数为止，
// 在这个计算过程中，得到的整数部分从上到下排列就是二进制的结果。
0.2 * 2 = 0.4 -> 0
0.4 * 2 = 0.8 -> 0
0.8 * 2 = 1.6 -> 1
0.6 * 2 = 1.2 -> 1
0.2 * 2 = 0.4 -> 0（发生循环）
...

```

> 解决浮点数运算的精度丢失问题

 `BigDecimal` 可以实现对浮点数的运算，不会造成精度丢失。通常情况下，大部分需要浮点数精确运算结果的业务场景（比如涉及到钱的场景）都是通过 `BigDecimal` 来做的。 

```java
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("0.9");
BigDecimal c = new BigDecimal("0.8");

BigDecimal x = a.subtract(b);
BigDecimal y = b.subtract(c);

System.out.println(x); /* 0.1 */
System.out.println(y); /* 0.1 */
System.out.println(Objects.equals(x, y)); /* true */
```

## 表示超过 long 整型的数据

基本数值类型都有一个表达范围，如果超过这个范围就会有数值溢出的风险。

在 Java 中，64 位 long 整型是最大的整数类型。

```java
long l = Long.MAX_VALUE;
System.out.println(l + 1); // -9223372036854775808
System.out.println(l + 1 == Long.MIN_VALUE); // true
```

- `BigInteger` 内部使用 `int[]` 数组来存储任意大小的整形数据。

- 相对于常规整数类型的运算来说，`BigInteger` 运算的效率会相对较低

# 变量

## 成员变量与局部变量的区别

**语法形式**：

* 成员变量是属于类的，而局部变量是在代码块或方法中定义的变量 
* 成员变量可以被 `public`，`private`，`static` 等修饰符所修饰，而局部变量不能被访问控制修饰符及 `static` 所修饰
* 成员变量和局部变量都能被 `final` 所修饰。

**存储方式**：

* 如果成员变量是使用 `static` 修饰的，那么该成员变量是属于类的
* 如果没有使用 `static` 修饰，那么该成员变量是属于实例的
* 对象存在于堆内存，局部变量则存在于栈内存。

**生存时间**：

* 成员变量是对象的一部分，它随着对象的创建而存在
* 局部变量随着方法的调用而自动生成，随着方法的调用结束而消亡 

**默认值**：

* 成员变量如果没有被赋初始值，则会自动以类型的默认值而赋值（一种情况例外：被 `final` 修饰的成员变量也必须显式地赋值）
* 局部变量则不会自动赋值 

> **为什么成员变量有默认值？** 

* 如果没有默认值，变量存储的是内存地址对应的任意随机值，程序读取该值运行会出现意外
* 默认值有两种设置方式：手动和自动，没有手动赋值一定要自动赋值。成员变量在运行时可借助反射等方法手动赋值，而局部变量不行。
* 对于编译器（javac）来说，局部变量没赋值很好判断，可以直接报错。而成员变量可能是运行时赋值，无法判断，误报“没默认值”又会影响用户体验，所以采用自动赋默认值。

```java
public class VariableExample {

    // 成员变量
    private String name;
    private int age;

    // 方法中的局部变量
    public void method() {
        int num1 = 10; // 栈中分配的局部变量
        String str = "Hello, world!"; // 栈中分配的局部变量
        System.out.println(num1);
        System.out.println(str);
    }

    // 带参数的方法中的局部变量
    public void method2(int num2) {
        int sum = num2 + 10; // 栈中分配的局部变量
        System.out.println(sum);
    }

    // 构造方法中的局部变量
    public VariableExample(String name, int age) {
        this.name = name; // 对成员变量进行赋值
        this.age = age; // 对成员变量进行赋值
        int num3 = 20; // 栈中分配的局部变量
        String str2 = "Hello, " + this.name + "!"; // 栈中分配的局部变量
        System.out.println(num3);
        System.out.println(str2);
    }
}
```

## 静态变量

* 静态变量也就是被 `static` 关键字修饰的变量，它可以被类的所有实例共享。
* 即使创建多个对象，静态变量只会被分配一次内存
* 静态变量通过类名来访问 （如果被 `private`关键字修饰就无法这样访问了）。 

```java
public class StaticVariableExample {
    // 静态变量
    public static int staticVar = 0;
}
```

 通常情况下，静态变量会被 `final` 关键字修饰成为常量。 

```java
public class ConstantVariableExample {
    // 常量
    public static final int constantVar = 0;
}
```

## 字符型常量和字符串常量的区别

**形式** :

-  字符常量是单引号引起的一个字符
-  字符串常量是双引号引起的 0 个或若干个字符。

**含义** : 

- 字符常量相当于一个整型值( ASCII 值)，可以参加表达式运算
- 字符串常量代表一个地址值（该字符串在内存中存放位置）。

**占内存大小**：

- 字符常量只占 2 个字节（⚠️ `char` 在 Java 中占两个字节）
- 字符串常量占若干个字节。



```java
public class StringExample {
    // 字符型常量
    public static final char LETTER_A = 'A';

    // 字符串常量
    public static final String GREETING_MESSAGE = "Hello, world!";
    public static void main(String[] args) {
        System.out.println("字符型常量占用的字节数为："+Character.BYTES);
        System.out.println("字符串常量占用的字节数为："+GREETING_MESSAGE.getBytes().length);
    }
}

字符型常量占用的字节数为：2
字符串常量占用的字节数为：13
```

# 方法

## 方法的返回值和方法类型

>  按照方法的返回值和参数类型将方法分为 

* **无参数无返回值的方法** 
* **有参数无返回值的方法** 
* **有返回值无参数的方法** 

* **有返回值有参数的方法** 

## 静态方法不能调用非静态成员 

* 静态方法是属于类的，在类加载时就会分配内存，可以通过类名直接访问
* 非静态成员属于实例对象，只有在对象实例化后才存在，需要通过类的实例对象去访问
* 在类的非静态成员不存在时静态方法就已经存在了，此时调用在内存中还不存在的非静态成员，属于非法操作

## 静态方法和实例方法的不同

> **调用方式** 

- 在外部调用静态方法时，可以使用 `类名.方法名` 的方式，也可以使用 `对象.方法名` 的方式
- 而实例方法只有使用 `对象.方法名` 的方式
- **调用静态方法可以无需创建对象** 。 

不过，需要注意的是一般不建议使用 `对象.方法名` 的方式来调用静态方法。这种方式非常容易造成混淆，静态方法不属于类的某个对象而是属于这个类。

因此，一般建议使用 `类名.方法名` 的方式来调用静态方法。

```java
public class Person {
    public void method() {
      //......
    }

    public static void staicMethod(){
      //......
    }
    public static void main(String[] args) {
        Person person = new Person();
        // 调用实例方法
        person.method();
        // 调用静态方法
        Person.staicMethod()
    }
}
```

> **访问类成员是否存在限制** 

- 静态方法在访问本类的成员时，只允许访问静态成员（即静态成员变量和静态方法），不允许访问实例成员（即实例成员变量和实例方法）
- 而实例方法不存在这个限制 

## 重载和重写

> 重载

- 重载发生在同一个类中（或者父类和子类之间），方法名必须相同，参数类型不同、个数不同、顺序不同
- 方法返回值和访问修饰符可以不同。 

**重载就是同一个类中多个同名方法根据不同的传参来执行不同的逻辑处理。**

> 重写

 重写发生在运行期，是子类对父类的允许访问的方法的实现过程进行重新编写。 

* 方法名、参数列表必须相同，子类方法返回值类型应比父类方法返回值类型更小或相等，抛出的异常范围小于等于父类，访问修饰符范围大于等于父类。

* 如果父类方法访问修饰符为 `private/final/static` 则子类就不能重写该方法，但是被 `static` 修饰的方法能够被再次声明。

* 构造方法无法被重写

 **重写就是子类对父类方法的重新改造，外部样子不能改变，内部逻辑可以改变。** 

| **区别点** | 重载方法 | **重写方法**                                                 |
| ---------- | -------- | ------------------------------------------------------------ |
| 发生范围   | 同一个类 | 子类                                                         |
| 参数列表   | 必须修改 | 一定不能修改                                                 |
| 返回类型   | 可修改   | 子类方法返回值类型应比父类方法返回值类型更小或相等           |
| 异常       | 可修改   | 子类方法声明抛出的异常类应比父类方法声明抛出的异常类更小或相等； |
| 访问修饰符 | 可修改   | 一定不能做更严格的限制（可以降低限制）                       |
| 发生阶段   | 编译期   | 运行期                                                       |

⭐️ 关于 **重写的返回值类型**

- 如果方法的返回类型是 void 和基本数据类型，则返回值重写时不可修改。
- 但是如果方法的返回值是引用类型，重写时是可以返回该引用类型的子类的。 

```java
public class Hero {
    public String name() {
        return "超级英雄";
    }
}
public class SuperMan extends Hero{
    @Override
    public String name() {
        return "超人";
    }
    public Hero hero() {
        return new Hero();
    }
}

public class SuperSuperMan extends SuperMan {
    public String name() {
        return "超级超级英雄";
    }

    @Override
    public SuperMan hero() {
        return new SuperMan();
    }
}
```

## 可变长参数

* Java5 开始，Java 支持定义可变长参数，允许在调用方法时传入不定长度的参数

* 可变参数只能作为函数的最后一个参数，但其前面可以有也可以没有任何其他参数 

* Java 的可变参数编译后实际会被转换成一个数组（看编译后生成的 `class`文件就可以看出来）

````java
public static void method(String arg1, String... args) {
   //......
}
````

> **遇到方法重载的情况怎么办呢？会优先匹配固定参数还是可变参数的方法呢？**

会优先匹配固定参数的方法，因为固定参数的方法匹配度更高。

 ```java
public class VariableLengthArgument {

    public static void printVariable(String... args) {
        for (String s : args) {
            System.out.println(s);
        }
    }

    public static void printVariable(String arg1, String arg2) {
        System.out.println(arg1 + arg2);
    }

    public static void main(String[] args) {
        printVariable("a", "b");
        printVariable("a", "b", "c", "d");
    }
}
ab
a
b
c
d
 ```



 另外，Java 的可变参数编译后实际会被转换成一个数组，我们看编译后生成的 `class`文件就可以看出来了 

```java
public class VariableLengthArgument {

    public static void printVariable(String... args) {
        String[] var1 = args;
        int var2 = args.length;

        for(int var3 = 0; var3 < var2; ++var3) {
            String s = var1[var3];
            System.out.println(s);
        }

    }
    // ......
}
```

