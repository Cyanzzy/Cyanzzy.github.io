---
title: Java8-1-Lambda Expression
date: 2023-03-17 21:04:06
tags: 
  - Java
categories: 
  - Language
swiper_index: 
---
> 面向对象编程是对数据进行抽象；函数式编程是对行为进行抽象。
>
> **核心思想**: 使用不可变值和函数，函数对一个值进行处理，映射成另一个值。

lambda表达式仅能放入如下代码:

* 预定义使用了 @Functional 注释的函数式接口，自带一个抽象函数的方法
* SAM(Single Abstract Method 单个抽象方法)类型。

这些称为lambda表达式的目标类型，可以用作返回类型，或lambda目标代码的参数。

若一个方法接收Runnable、Comparable或者 Callable 接口，都有**单个抽象方法**，可以传入lambda表达式。

类似的，如果**一个方法接受声明于 java.util.function 包内的接口**，例如 Predicate、Function、Consumer 或 Supplier，那么可以向其传lambda表达式。

# 基本格式

```java
{params}->{code}
```

> 案例1

```java
new Thread(new Runnable() {
    pubic void run() {
        // code
    }
}).start();

new Thread(
    ()->{
        // code
    }
).start();
```

> 案例2

```java
// 匿名内部类
Runnable r = new Runnable() {
    public void run() {
        System.out.println("Hello");
    }
}

// Lambda 表达式
Runnale r = () -> System.out.println("Hello");
```

```java
// 使用匿名内部类作为参数传递
TreeSet<String> ts = new TreeSet<>(new Comparator<String>() {
    public int compare(String o1, String o2) {
        return Integer.compare(o1.length(), o2.length());
    }
});

// Lambda 表达式作为参数传递
TreeSet<String> ts = new TreeSet<> (
	(o1, o2) -> Integer.compare(o1.length(), o2.length())
);
```

> 案例3

 现有方法定义如下，其中`IntBinaryOperator`是一个接口， `IntBinaryOperator`是一个函数式接口并且自带一个抽象方法，因此可以使用lambda表达式。 

 ```java
public static int calculateNum(IntBinaryOperator operator) {
    int a = 1;
    int b = 2;
    return operator.applyAsInt(a, b);
}
 ```

```java
// 匿名内部类实现
int var = calculateNum(new IntBinaryOperator() {
    public int applyAsInt(int left, int right) {
        return left + right;
    } 
});

// Lambda表达式实现
int var = calculateNum((int left, int right)->{
   return left + right;
});

// 进一步省略
int var = calculateNum((left, right)->left + right);
```

> 案例4

现有方法定义如下，`IntPredicate`是一个接口 ， `IntPredicate`是一个函数式接口并且自带一个抽象方法，因此可以使用lambda表达式。 

```java
public static void printNum(IntPredicate predicate) {
    int[] arr = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    for (int i : arr) {
        if (predicate.test(i)) {
            System.out.println(i);
        }
    }
}
```

```java
// 匿名内部类实现
printNum(new IntPredicate() {
   public boolean test(int value) {
       return value % 2 == 0;
   } 
});

// Lambda表达式实现
printNum((int value)->{
    return value % 2 == 0;
});
```

> 案例5

```java
public static <R> R typeConver(Function<String, R> function) {
    String str="12345";
    R result = function.apply(str);
    return result;
}
```

```java
// 匿名内部类实现
Integer res = typeConver(new Function<String, Integer> {
    public Integer apply(String s) {
      return Integer.valueOf(s);  
    }
});

// Lambda表达式实现
Integer res = typeConver((String s)->{
      return Integer.valueOf(s);  
});
```

> 案例6

```java
public static void foreachArr(IntConsumer consumer) {
    int[] arr = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    for (int i : arr) {
        consumer.accept(i);
    }
}
```

```java
// 匿名内部类实现
foreachArr(new IntConsumer() {
     public void accept(int value) {
         System.out.println(value);
     }
});

// Lambda表达式实现
foreachArr((int value)->{
    System.out.println(value);
});
```

# 省略规则

- 参数类型可以省略
- 方法体只有一句代码时大括号return和唯一一句代码的分号可以省略
- 方法只有一个参数时小括号可以省略