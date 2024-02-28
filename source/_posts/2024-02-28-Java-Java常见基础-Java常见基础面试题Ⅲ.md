---
title: Java 常见基础面试题总结 Ⅲ
date: 2024-02-28 21:34:32
tags: 
  - Java
categories: 
  - Interview
password: zzy   
message: 仅管理员可见
---

# 异常

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E5%B8%B8%E8%A7%81%E5%9F%BA%E7%A1%80-15.png)

## Exception 和 Error

 在 Java 中，所有的异常都有一个共同的祖先 `java.lang` 包中的 `Throwable` 类。`Throwable` 类有两个重要的子类: 

> `Exception` 

- 程序本身可以处理的异常，可以通过 `catch` 来进行捕获。 
- `Exception` 又可以分为 Checked Exception（受检查异常，必须处理）和 Unchecked Exception（不受检查异常，可以不处理）

> `Error`

* 程序无法处理的错误 ，不建议通过 `catch` 来进行捕获
* 例如 Java 虚拟机运行错误（`Virtual MachineError`）、虚拟机内存不够错误(`OutOfMemoryError`)、  类定义错误（`NoClassDefFoundError`）等
* 这些错误发生时，Java 虚拟机（JVM）一般会选择线程终止。 

## Checked Exception 和 Unchecked Exception

> **Checked Exception** 

受检查异常 ，Java 代码在编译过程中，如果受检查异常没有被 `catch`或者`throws` 关键字处理的话，就没办法通过编译。除 `RuntimeException`及其子类外，其他的`Exception`类及其子类都属于受检查异常 。 

常见的受检查异常有：

- IO 相关的异常
- `ClassNotFoundException`
- `SQLException`

> **Unchecked Exception** 

不受检查异常 ，Java 代码在编译过程中 ，我们即使不处理不受检查异常也可以正常通过编译。 

 `RuntimeException` 及其子类都统称为非受检查异常 ：

* `NullPointerException`(空指针错误)
* `ArrayIndexOutOfBoundsException`（数组越界错误）
* `ArithmeticException`（算术错误）
* `ClassCastException`（类型转换错误）
* `IllegalArgumentException`(参数错误比如方法入参类型错误)
* `NumberFormatException`（字符串转换为数字格式错误，`IllegalArgumentException`的子类）
* `SecurityException` （安全错误比如权限不够）
* `UnsupportedOperationException`(不支持的操作错误比如重复创建同一用户)

## Throwable 类常用方法

| 方法                         | 说明                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| String getMessage()          | 返回异常发生时的简要描述                                     |
| String toString()            | 返回异常发生时的详细信息                                     |
| String getLocalizedMessage() | 返回异常对象的本地化信息。使用 `Throwable` 的子类覆盖这个方法，可以生成本地化信息。如果子类没有覆盖该方法，则该方法返回的信息与 `getMessage()`返回的结果相同 |
| void printStackTrace()       | 在控制台上打印 `Throwable` 对象封装的异常信息                |

## try-catch-finally

* `try`块：用于捕获异常。其后可接零个或多个 `catch` 块，如果没有 `catch` 块，则必须跟一个 `finally` 块。
* `catch`块：用于处理 try 捕获到的异常。
* `finally` 块：无论是否捕获或处理异常，`finally` 块里的语句都会被执行。当在 `try` 块或 `catch` 块中遇到 `return` 语句时，`finally` 语句块将在方法返回之前被执行。

```java
try {
    System.out.println("Try to do something");
    throw new RuntimeException("RuntimeException");
} catch (Exception e) {
    System.out.println("Catch Exception -> " + e.getMessage());
} finally {
    System.out.println("Finally");
}
// 
Try to do something
Catch Exception -> RuntimeException
Finally
```



> **注意：不要在 finally 语句块中使用 return!** 

当 try 语句和 finally 语句中都有 return 语句时，try 语句块中的 return 语句会被忽略。

这是因为 try 语句中的 return 返回值会先被暂存在一个本地变量中，当执行到 finally 语句中的 return 之后，这个本地变量的值就变为了 finally 语句中的 return 返回值。

## finally 中的代码一定会执行吗？

 不一定的！在某些情况下，finally 中的代码不会被执行。 

 就比如说 finally 之前虚拟机被终止运行的话，finally 中的代码就不会被执行。 

另外，在以下 2 种特殊情况下，`finally` 块的代码也不会被执行：

1. 程序所在的线程死亡。
2. 关闭 CPU。

```java
try {
    System.out.println("Try to do something");
    throw new RuntimeException("RuntimeException");
} catch (Exception e) {
    System.out.println("Catch Exception -> " + e.getMessage());
    // 终止当前正在运行的Java虚拟机
    System.exit(1);
} finally {
    System.out.println("Finally");
}

// outputs
Try to do something
Catch Exception -> RuntimeException
```

##  try-with-resources

在Java中，`try-with-resources` 是一种语法糖，用于自动关闭实现了 `AutoCloseable` 或 `Closeable` 接口的资源。这包括文件 I/O、数据库连接、网络连接等需要显式关闭的资源。

1. **资源的声明和初始化：** 在 `try` 关键字后面的括号内，你可以声明和初始化需要使用的资源。这些资源必须是实现了 `AutoCloseable` 或 `Closeable` 接口的类的实例。
2. **资源的顺序：** 资源的声明顺序决定了关闭的顺序，即从上到下的顺序关闭。这是因为资源的关闭是按照声明的逆序进行的。
3. **异常处理：** `try` 块内的代码可能抛出异常，而这些异常会在 `catch` 块中被捕获和处理。在 `catch` 块执行完毕后，系统会自动关闭在 `try` 块头部声明的资源。

`try-with-resources` 语句的一般形式如下：

```java
try (ResourceType1 resource1 = new ResourceType1();
     ResourceType2 resource2 = new ResourceType2();
     // ... more resources ...
) {
    // 使用资源的代码块
} catch (ExceptionType e) {
    // 异常处理代码块
}
```

 在 `try` 块结束时，不论是否发生异常，都会自动关闭这些资源。这样可以避免手动处理资源关闭的繁琐工作，并且可以提高代码的可读性。 

* **适用范围（资源的定义）：** 任何实现 `java.lang.AutoCloseable`或者 `java.io.Closeable` 的对象

* **关闭资源和 finally 块的执行顺序：** 在 `try-with-resources` 语句中，任何 catch 或 finally 块在声明的资源关闭后运行

 Java 中类似于`InputStream`、`OutputStream`、`Scanner`、`PrintWriter`等的资源都需要我们调用`close()`方法来手动关闭

一般情况下我们都是通过`try-catch-finally`语句来实现这个需求，如下：

```java
//读取文本文件的内容
Scanner scanner = null;
try {
    scanner = new Scanner(new File("D://read.txt"));
    while (scanner.hasNext()) {
        System.out.println(scanner.nextLine());
    }
} catch (FileNotFoundException e) {
    e.printStackTrace();
} finally {
    if (scanner != null) {
        scanner.close();
    }
}
```

 使用 Java 7 之后的 `try-with-resources` 语句改造上面的代码：

```java
try (Scanner scanner = new Scanner(new File("test.txt"))) {
    while (scanner.hasNext()) {
        System.out.println(scanner.nextLine());
    }
} catch (FileNotFoundException fnfe) {
    fnfe.printStackTrace();
}
```

 当然多个资源需要关闭的时候，使用 `try-with-resources` 实现起来也非常简单，如果你还是用`try-catch-finally`可能会带来很多问题。 

 通过使用分号分隔，可以在`try-with-resources`块中声明多个资源。 

```java
try (BufferedInputStream bin = new BufferedInputStream(new FileInputStream(new File("test.txt")));
     BufferedOutputStream bout = new BufferedOutputStream(new FileOutputStream(new File("out.txt")))) {
    int b;
    while ((b = bin.read()) != -1) {
        bout.write(b);
    }
}
catch (IOException e) {
    e.printStackTrace();
}
```

## 异常使用注意事项

- 不要把异常定义为静态变量，因为这样会导致异常栈信息错乱。每次手动抛出异常，我们都需要手动 new 一个异常对象抛出。

*  抛出的异常信息一定要有意义 

- 建议抛出更加具体的异常比如字符串转换为数字格式错误的时候应该抛出`NumberFormatException`而不是其父类`IllegalArgumentException`。

- 使用日志打印异常之后就不要再抛出异常了（两者不要同时存在一段代码逻辑中）。

#  泛型

 **Java 泛型（Generics）** 是 JDK 5 中引入的一个新特性。 编译器可以对泛型参数进行检测，并且通过泛型参数可以指定传入的对象类型。

比如 `ArrayList<Person> persons = new ArrayList<Person>()` 这行代码就指明了该   `ArrayList` 对象只能传入  `Person` 对象，如果传入其他类型的对象就会报错。 

```java
ArrayList<E> extends AbstractList<E>
```

 并且，原生 `List` 返回类型是 `Object` ，需要手动转换类型才能使用，使用泛型后编译器自动转换 

> **泛型类**： 

```java
public class Generic<T>{

    private T key;

    public Generic(T key) {
        this.key = key;
    }

    public T getKey(){
        return key;
    }
}

// 实例化泛型类
Generic<Integer> genericInteger = new Generic<Integer>(123456);
```

> **泛型接口**： 

```java
public interface Generator<T> {
    public T method();
}

// 实现泛型接口，不指定类型：
class GeneratorImpl<T> implements Generator<T>{
    @Override
    public T method() {
        return null;
    }
}

// 实现泛型接口，指定类型：
class GeneratorImpl<T> implements Generator<String>{
    @Override
    public String method() {
        return "hello";
    }
}
```

> **泛型方法：** 

```java
// 静态泛型方法;
public static < E > void printArray( E[] inputArray ) {
     for ( E element : inputArray ){
        System.out.printf( "%s ", element );
     }
     System.out.println();
}

public <T> void show(T t){}
// 创建不同类型数组：Integer, Double 和 Character
Integer[] intArray = { 1, 2, 3 };
String[] stringArray = { "Hello", "World" };
printArray( intArray  );
printArray( stringArray  );
```

* 在 java 中泛型只是一个占位符，必须在传递类型后才能使用。
* `public static < E > void printArray( E[] inputArray )` 一般被称为静态泛型方法，类在实例化时才能真正的传递类型参数，由于静态方法的加载先于类的实例化
* 也就是说类中的泛型还没有传递真正的类型参数，静态的方法的加载就已经完成了，所以静态泛型方法是没有办法使用类上声明的泛型的。 只能使用自己声明的 `<E>` 

> 项目使用泛型

- 自定义接口通用返回结果 `CommonResult<T>` 通过参数 `T` 可根据具体的返回类型动态指定结果的数据类型

- 定义 `Excel` 处理类 `ExcelUtil` 用于动态指定 `Excel` 导出的数据类型

* 构建集合工具类（参考 `Collections` 中的 `sort`, `binarySearch` 方法） 

# 反射

> 反射

反射赋予了我们在运行时分析类以及执行类中方法的能力。通过反射你可以获取任意一个类的所有属性和方法，你还可以调用这些方法和属性。 

> 反射的优缺点

* 反射可以让我们的代码更加灵活、为各种框架提供开箱即用的功能提供了便利。 

* 反射让我们在运行时有了分析操作类的能力的同时，也增加了安全问题，比如可以无视泛型参数的安全检查（泛型参数的安全检查发生在编译时）。 

* 另外，反射的性能也要稍差点，不过，对于框架来说实际是影响不大的。 

[Java Reflection: Why is it so slow?](https://stackoverflow.com/questions/1392351/java-reflection-why-is-it-so-slow)

>  反射的应用场景

* Spring/Spring Boot、MyBatis 等等框架中都大量使用了反射机制。 
* **这些框架中也大量使用了动态代理，而动态代理的实现也依赖反射。** 
* 像 Java 中的一大利器 **注解** 的实现也用到了反射。 

>  为什么你使用 Spring 的时候 ，一个`@Component`注解就声明了一个类为 Spring Bean 呢？为什么你通过一个 `@Value`注解就读取到配置文件中的值呢？究竟是怎么起作用的呢？ 

 基于反射分析类，然后获取到类/属性/方法/方法的参数上的注解。你获取到注解之后，就可以做进一步的处理 

# 4. 注解

 `Annotation` （注解） 是 Java5 开始引入的新特性，主要用于修饰类、方法或者变量，提供某些信息供程序在编译或者运行时使用， 注解本质是一个继承了`Annotation` 的特殊接口

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {

}

public interface Override extends Annotation{

}
```

> 注解的解析方法

 注解只有被解析之后才会生效，常见的解析方法有两种： 

**编译期直接扫描**：编译器在编译 Java 代码的时候扫描对应的注解并处理，比如某个方法使用`@Override` 注解，编译器在编译的时候就会检测当前的方法是否重写了父类对应的方法。

**运行期通过反射处理**：像框架中自带的注解(比如 Spring 框架的 `@Value`、`@Component`)都是通过反射来进行处理的。

# SPI

> SPI

* SPI 即 Service Provider Interface ，即**专门提供给服务提供者或者扩展框架功能的开发者去使用的一个接口**。

* SPI 将服务接口和具体的服务实现分离开来，将服务调用方和服务实现者解耦，能够提升程序的扩展性、可维护性。修改或者替换服务实现并不需要修改调用方。

* 很多框架都使用了 Java 的 SPI 机制，比如：Spring 框架、数据库加载驱动、日志接口、以及 Dubbo 的扩展实现等等。

> SPI 和 API

![](../../../../../images/复习总纲/2024-01-1 轮复习/1 突击八股文和面经/JavaGuide/Java/复习总纲-2024-01-1 轮复习-1 突击八股文和面经-JavaGuide-Java-19.png)

*  一般模块之间都是通过接口进行通讯，那我们在服务调用方和服务实现方（也称服务提供者）之间引入一个“接口”。 
*  API（接口和实现存在于实现方 ）： 实现方提供接口和实现，我们可以通过调用实现方的接口从而拥有实现方给我们提供的能力

*  SPI（当接口存在于调用方这边时）：**由接口调用方确定接口规则，然后由不同的厂商去根据这个规则对这个接口进行实现，从而提供服务** 

> SPI 的优缺点

通过 SPI 机制能够大大地提高接口设计的灵活性，但是 SPI 机制也存在一些缺点 

* 需要遍历加载所有的实现类，不能做到按需加载，这样效率还是相对较低的 
* 当多个 `ServiceLoader` 同时 `load` 时，会有并发问题。

# 序列化和反序列化

- **序列化**：将数据结构或对象转换成二进制字节流的过程
- **反序列化**：将在序列化过程中所生成的二进制字节流转换成数据结构或者对象的过程

对于 Java 这种面向对象编程语言来说，我们序列化的都是对象（Object）也就是实例化后的类(Class)，但是在 C++这种半面向对象的语言中，struct(结构体)定义的是数据结构类型，而 class 对应的是对象类型。

> 序列化和反序列化常见应用场景： 

* 对象在进行**网络传输**（比如远程方法调用 RPC ）之前需要先被序列化，接收到序列化的对象之后需要再进行反序列化；
* 将对象**存储到文件**之前需要进行序列化，将对象从文件中读取出来需要进行反序列化；
* 将对象**存储到数据库**（如 Redis）之前需要用到序列化，将对象从缓存数据库中读取出来需要反序列化；
* 将对象**存储到内存**之前需要进行序列化，从内存中读取出来之后需要进行反序列化。

**序列化的主要目的是通过网络传输对象或者说是将对象存储到文件系统、数据库、内存中。** 

![](../../../../../images/复习总纲/2024-01-1 轮复习/1 突击八股文和面经/JavaGuide/Java/复习总纲-2024-01-1 轮复习-1 突击八股文和面经-JavaGuide-Java-20.png)

##  序列化协议对应于 TCP/IP 4 层模型的哪一层？

网络通信的双方必须要采用和遵守相同的协议。TCP/IP 四层模型是下面这样的，序列化协议属于哪一层呢？

1. **应用层**
2. 传输层
3. 网络层
4. 网络接口层

![](../../../../../images/复习总纲/2024-01-1 轮复习/1 突击八股文和面经/JavaGuide/Java/复习总纲-2024-01-1 轮复习-1 突击八股文和面经-JavaGuide-Java-21.png)

如上图所示，OSI 七层协议模型中，表示层做的事情主要就是对应用层的用户数据进行处理转换为二进制流。反过来的话，就是将二进制流转换成应用层的用户数据。这不就对应的是序列化和反序列化么？

因为，OSI 七层协议模型中的应用层、表示层和会话层对应的都是 TCP/IP 四层模型中的应用层，所以**序列化协议属于 TCP/IP 协议应用层的一部分**

## 如果有些字段不想进行序列化怎么办？

> 对于不想进行序列化的变量，使用 `transient` 关键字修饰。

- 阻止实例中那些用此关键字修饰的的变量序列化；
- 当对象被反序列化时，被 `transient` 修饰的变量值不会被持久化和恢复。 

> 注意

- `transient` 只能修饰变量，不能修饰类和方法。

*  `transient` 修饰的变量，在反序列化后变量值将会被置成类型的默认值。例如，如果是修饰 `int` 类型，那么反序列后结果就是 `0` 

- `static` 变量因为不属于任何对象(Object)，所以无论有没有 `transient` 关键字修饰，均不会被序列化。

## 常见序列化协议

 JDK 自带的序列化方式一般不会用 ，因为序列化效率低并且存在安全问题。

比较常用的序列化协议有 Hessian、Kryo、Protobuf、ProtoStuff，这些都是基于二进制的序列化协议。 

 像 JSON 和 XML 这种属于文本类序列化方式。虽然可读性比较好，但是性能较差，一般不会选择。 

> 为什么不推荐使用 JDK 自带的序列化？

**不支持跨语言调用** : 如果调用的是其他语言开发的服务的时候就不支持了。

**性能差**：相比于其他序列化框架性能更低，主要原因是序列化之后的字节数组体积较大，导致传输成本加大。

**存在安全问题**：序列化和反序列化本身并不存在问题。但当输入的反序列化的数据可被用户控制，那么攻击者即可通过构造恶意输入，让反序列化产生非预期的对象，在此过程中执行构造的任意代码

 [应用安全：JAVA 反序列化漏洞之殇](https://cryin.github.io/blog/secure-development-java-deserialization-vulnerability/) 

# I/O

## Java IO 流

-  IO 即 `Input/Output`，输入和输出。
-  数据输入到计算机内存的过程即输入，反之输出到外部存储（比如数据库，文件，远程主机）的过程即输出。
-  **IO 流在 Java 中分为输入流和输出流，而根据数据的处理方式又分为字节流和字符流**。 

> Java IO 流的 4 个抽象类基类

- `InputStream`/`Reader`：所有的输入流的基类，前者是字节输入流，后者是字符输入流。
- `OutputStream`/`Writer`：所有输出流的基类，前者是字节输出流，后者是字符输出流。

## I/O 流为什么要分为字节流和字符流呢?

>  **不管是文件读写还是网络发送接收，信息的最小存储单元都是字节**
>
>  **那为什么 I/O 流操作要分为字节流操作和字符流操作呢？** 

I/O（输入/输出）流操作在Java中分为字节流和字符流，这是为了处理不同类型的数据。虽然信息的最小存储单元是字节，但字符流和字节流的区分主要是为了方便处理文本数据和二进制数据。 

**字符编码：** 

- 文本数据在计算机内部以字节形式存储，而不同的字符集和编码方式通过不同的字节序列表示相同的字符。
- **字符流提供字符编码的支持**，使得在读写文本数据时可以方便地进行字符集的转换。

**缓冲处理：** 

- **字符流通常会提供缓冲功能**，允许程序以较大块的数据进行读写，以提高性能
- 而字节流通常没有这种缓冲处理，每次只能读写一个字节。



