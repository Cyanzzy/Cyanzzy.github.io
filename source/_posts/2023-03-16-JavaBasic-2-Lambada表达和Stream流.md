---
title: JavaBasic-2-Lambada表达和Stream流
date: 2023-03-16 18:25:38
tags: 
  - Java
categories: 
  - Language
swiper_index: 
---

# Lambda Expression
> 概述

Lambda表达式是函数式编程思想的体现，函数式编程思想尽量忽略面向对象的复杂语法，强调“做什么”而不是以什么形式去做

> 格式

Lambda表达式格式：`(形式参数) -> {代码}`

1. 如果有多个参数，参数之间用逗号隔开，如果没有参数则留空
2. ->代表指向动作
3. 代码指具体实现的内容

> 前提

Lambda表达式使用前提

1. 存在接口
2. **接口中有且仅有一个抽象方法**

> 分类

1. 无参无返回值
2. 有参无返回值
3. 无参有返回值
4. 有参数有返回值

> Lambda表达式省略模式

1. 参数类型可以省略，如果有多个参数，要么不省略要么全部省略
2. 如果参数有且仅有一个，那么参数的括号可以省略
3. 如果代码块中只有一条语句，那么大括号和语句的分号可以省略，return关键字也可以省略

> **Lambda表达式和匿名内部类的区别**

1. 所需类型不同
   * 匿名内部类：可以是接口，可以是抽象类，还可以是具体的类
   * Lambda表达式：**只能是接口**
2. 使用限制不同
   * 如果接口中只有一个抽象方法，则可以使用Lambda表达式也可以使用匿名内部类
   * 接口中有多于一个抽象方法，只能使用匿名内部类，不能使用Lambda表达式
3. 实现原理不同
   * 匿名内部类：编译之后，产生单独的.class字节码文件
   * Lambda表达式：编译之后，没有产生单独的.class字节码文件，对应的字节码会在运行的时候动态生成

# Stream流

> Stream流中的三类方法

1. 获取Stream流的方法：创建一条类似于流水线的流，并且将数据放上流中准备进行处理
2. 中间方法：流水线上的操作，一次操作结束后还能继续进行其他操作
3. 终结方法：一个Stream流**只能有一个终结方法，是流水线上最后的一个操作**

> 获取Stream流的方法

1. 单列集合：可以使用**Collection**接口的默认方法stream()生成流

   `default Stream<E> stream()`

2. 双列集合：间接生成流，可以先通过keySet或entrySet获取一个Set集合，再获取Stream流

3. 数组：Arrays中的静态方法stream()生成

4. 同种类型的多个数据：使用Stream.of(T...value)方法获取，其中参数是**可变参数**

> 常用中间方法

1. `Stream<T> filter(Predicate<? super T> predicate)`，用于按照某规则过滤流中的数据
   * 其中Predicate接口含有抽象方法test(T t)，形参是流中的数据，重写该方法时可以指定过滤规则
   * 当返回值是true时保留数据，当false时过滤掉数据
   * 由于Predicate是只含有一个抽象方法的接口，所以可以使用Lambda表达式，也可以使用匿名内部类
2. `Stream<T> limit(long maxsize)`，将流中前maxsize个数据截取出来
3. `Stream<T> skip(long n)`，跳过n个数据
4. `static Stream<T> concat(Stream a,Stream b)`，Stream中的静态方法，用于合并两个流
5. `Stream<T> distinct()`，去除流中重复的数据，依赖hashCode()方法和equals()方法

> 常用终结方法

1. `void forEach(Consumer<? super T> consumer)`，用于遍历流中的数据
   * 其中Consumer接口含有一个抽象方法accpet(T t)，该方法的形参就是流中的数据，所以可以在重写该方法时设计对流中数据的操作
   * 比如打印输出，由于该接口只有一个抽象方法所以可以使用Lambda表达式
2. `long count()`，返回流中的元素个数

> 注意事项

* Stream流**无法修改集合或者数组等数据源中的数据**

* Stream流中的收集操作

  1. `R collect(Collector collector)`，用于将流中的数据收集一个集合中并返回这个集合，集合的类型由Collector决定，R表示集合类型
  2. **Collectors**工具类
     * 该类可以由静态方法**获取collector对象**
     * `public static <T> Collector toList()`，收集到List集合中
     * `public static <T> Collector toSet()`，收集到Set集合中
     * `public static <T> Collector toMap(Function keyMapper,Function valueMapper)`，收集到Map集合中，注意toMap()方法中的两个参数，可以理解为获取键和值的方式，可以使用Lambda表达式