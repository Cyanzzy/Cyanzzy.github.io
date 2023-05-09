---
title: Java8-4-Stream API
date: 2023-03-17 21:14:32
tags: 
  - Java8
categories: 
  - Language
swiper_index: 
---

Stream API 提供了一种高效且易于使用的处理数据的方式， Java8的Stream使用函数式编程模式，可以用来对集合或数组进行链状流式的操作  

# 概述

> Stream

Stream 是数据渠道，用于操作数据源（集合、数组等）所生成的元素序列。  

*  Stream 自己**不会存储元素**。 
*  Stream **不会改变源对象**。相反，他们会返回一个持有结果的新Stream 
*  Stream 操作是延迟执行的。这意味着他们会等到需要结果的时候才执行 

# 操作 

## 创建操作

 一个数据源（如：集合、数组），获取一个流  

> 获取流

| 方法                            | 说明           |
| ------------------------------- | -------------- |
| default Stream stream()         | 返回一个顺序流 |
| default Stream parallelStream() | 返回一个并行流 |

> 数组创建流

 Java8 中的 Arrays 的静态方法 stream() 可以获取数组流 

| 方法                                   | 说明       |
| -------------------------------------- | ---------- |
| static<T>  Stream<T> stream(T[] array) | 返回一个流 |

 数组：`Arrays.stream(数组)`或使用`Stream.of()`创建 

```java
Integer[] arr = {1, 2, 3};
Stream<Integer> stream = Arrays.stream(arr);
Stream<Integer> stream2 = Stream.of(arr);
```

 单列集合：`集合对象.stream()` 

```java
List<Author> authors = getAuthors();
Stream<Author>stream = authors.stream();
```

 双列集合：转换成单列集合再创建 

```java
Map<String, Integer> map = new HashMap<>();
map.put("Cyan", 100);
map.put("Jack", 200);

Stream<Map.Entry<String, Integer>> stream = map.entrySet().stream();
```

>  值创建流 

 使用静态方法 Stream.of(), 通过显示值创建一个流。它可以接收任意数量的参数。 

| 方法                                       | 说明       |
| ------------------------------------------ | ---------- |
| public static<T> Stream<T> of(T... values) | 返回一个流 |

>  函数创建流

 使用静态方法 Stream.iterate() 和 Stream.generate(), 创建无限流 

| 方法                                                         | 说明 |
| ------------------------------------------------------------ | ---- |
| public static<T> Stream<T> iterate(final T seed, final UnaryOperator f) | 迭代 |
| public static<T> Stream<T> generate(Supplier<T> s)           | 生成 |

## 中间操作 

 一个中间操作链，对数据源的数据进行处理 。 多个中间操作可以连接起来形成一个流水线，除非流水线上触发终止操作，否则中间操作不会执行任何的处理！而在终止操作时一次性全部处理，称为“惰性求值” 

> 筛选与切片 

| 方法                | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| filter(Predicate p) | 接收 Lambda ， 从流中**排除**某些元素。                      |
| distinct()          | 筛选，通过流所生成元素的 hashCode() 和 equals() **去除重复**元素 |
| limit(long maxSize) | **截断**流，使其元素不超过给定数量。                         |
| skip(long n)        | **跳过**元素，返回一个扔掉了前 n 个元素的流。若流中元素不足 n 个，则返回一个空流。与 limit(n) 互补 |

### filter

 对流中的元素进行条件过滤，符合过滤条件的才能继续留在流中

```java
// 匿名内部类写法
authors.stream().
       .filter(new Predicate<Author> {
            public boolean test(Author author) {
                return author.getName().length() > 1;}
        })
       .forEach(new Consumer<Author>() {
            public void accept(Author author) {
                System.out.println(author.getName());
            }
        });

// lambda表达式写法
authors.stream()
     .filter(author->author.getName().length>1)
     .forEach(author->System.out.println(author.getName()));
```

### distinct

 流中去重`[distinct是依赖Object的equals方法判断是否是相同对象的，要重写equals方法，lombok中使用@EqualsAndHashCode注解重写]` 

```java
authors.stream()
       .distinct()
       .forEach(author->System.out.println(author.getName()));
```

### limit

 设置流的最大长度，超出的部分将被抛弃 

```java
authors.stream()
       .distinct()
       .sorted()
       .limit(2)
       .forEach(author->System.out.println(author.getName()));
```

### skip

 跳过流中前n个元素，返回剩下的元素 

```java
authors.stream()
       .distinct()
       .sorted.skip(1)
       .forEach(author->System.out.println(author.getName()));
```

> 映射

| 方法                            | 说明                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| map(Function f)                 | **接收一个函数**作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素。 |
| mapToDouble(ToDoubleFunction f) | 接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的 DoubleStream。 |
| mapToInt(ToIntFunction f)       | 接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的 IntStream。 |
| mapToLong(ToLongFunction f)     | 接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的 LongStream。 |
| flatMap(Function f)             | 接收一个函数作为参数，**将流中的每个值都换成另一个流，然后把所有流连接成一个流** |

### map

 把流中的元素进行计算或转换 

```java
// 匿名内部类写法
authors.stream().
       .map(new Function<Author,String>() {
           public String apply(Author author) {
               return author.getName();
           }
       })
       .forEach(new Consumer<String>() {
           public void accept(String s) {
               System.out.println(s);
           }
       });

// lambda表达式写法
authors.stream()
       .map(author->author.getName())
       .forEach(s->System.out.println(s));
```

### flatMap

 `map`只能只能把一个对象转换成另一个对象来作为流中的元素，而`flatMap`可以**把一个对象转换成多个对象作为流中的元素**。 

```java
public class Author implements Comparable<Author> {
    private Long id;
    private String name;
    private Integer age;
    private String intro;
    private List<Book> books;
    
    public int compare(Author o) {
        return this.getAge() - o.getAge();
    }
}

authors.stream()
       .flatMap(new Function<Author, Stream<Book>>() {
           public Stream<?> apply(Author author) {
               return author.getBooks().stream();
           }
       }) 
       .distinct()
       .forEach(new Consumer<Book>() {
           public void accept(Book book) {
               System.out.println(book.getName());
           }
       });

authors.stream()
       .flatMap(author->author.getBooks().stream())
       .distinct()
       .forEach(book->System.out.println(book.getName()));


//  打印所有数据的分类，对分类进行去重
authors.stream()
       .flatMap(author->author.getName().stream())
       .distinct()
       .flatMap(book->Arrays.stream(book.getCategory().split(",")))
       .distinct()
       .forEach(category->System.out.println(category));
```

> 排序

| 方法                    | 说明                               |
| ----------------------- | ---------------------------------- |
| sorted()                | 产生一个新流，其中按自然顺序排序   |
| sorted(Comparator comp) | 产生一个新流，其中按比较器顺序排序 |

### sorted

 1.调用无参`sorted`方法 ，调用空参的`sorted()`方法，需要流中元素实现`Comparable`接口 

```java
// 重写Comparable接口
public class Author implements Comparable<Author> {
    ...
    
    public int compare(Author o) {
        return this.getAge() - o.getAge();
    }
}

authors.stream()
       .distinct()
       .sort()
       .forEach(author->System.out.println(author.getAge()));
```

 2.调用有参`sorted`方法 

```java
authors.stream()
       .distinct()
       .sort((o1,o2)->o1.getAge()-o2.getAge())
       .forEach(author->System.out.println(author.getAge()));
```

## 终止操作

 一个终止操作，执行中间操作链，并产生结果， 终端操作会从流的流水线生成结果。其结果可以是任何不是流的值，例如：List、Integer，甚至是 void 。  

> 查找与匹配

| 方法                   | 说明                                                         |
| ---------------------- | ------------------------------------------------------------ |
| allMatch(Predicate p)  | 检查是否匹配所有元素                                         |
| anyMatch(Predicate p)  | 检查是否至少匹配一个元素                                     |
| noneMatch(Predicate p) | 检查是否没有匹配所有元素                                     |
| findFirst()            | 返回第一个元素                                               |
| findAny()              | 返回当前流中的任意元素                                       |
| count()                | 返回流中元素总数                                             |
| max(Comparator c)      | 返回流中最大值                                               |
| min(Comparator c)      | 返回流中最小值                                               |
| forEach(Consumer c)    | 内部迭代(使用 Collection 接口需要用户去做迭代，称为外部迭代。相反，Stream API 使用内部 迭代——它帮你把迭代做了) |

### forEach

 对流中的元素进行遍历操作，通过传入的参数去指定对遍历到的元素进行具体操作 

```java
authors.stream()
       .map(author->author.getName())
       .distinct()
       .forEach(name->System.out.print(name));
```

### count

 可以获取当前流中元素的个数 

```java
long cnt = authors.stream()
                  .flatMap(author->getBooks().stream())
                  .distinct()
                  .count();
```

### max&min

 统计流中的最值 

```java
Optional<Integer> max=authors.stream()
                             .flatMap(author->author.Books())
                             .map(book->book.getScore())
                             .max((score1-score2)->score1-score2);

Optional<Integer> min=authors.stream()
                             .flatMap(author->author.Books())
                             .map(book->book.getScore())
                             .min((score1-score2)->score1-score2);
```

### AnyMatch

判断是否有任意符合匹配条件的元素，结果为boolean类型

```java
authors.stream()
       .AnyMatch(author->author.getAge()>29);
```

### allMatch

判断是否都符合匹配条件，结果为boolean类型

```java
authors.stream()
       .allMatch(author->author.getAge()>29)
```

### noneMatch

判断留着元素都不符合匹配条件的，结果为boolean类型

```java
authors.stream()
       .noneMatch(author->author.getAge()>29)
```

### findAny

 获取流中任意一个元素[保证不了是否为第一个元素] 

```java
Optional<Author> author = authors.stream()
                                 .filter(author->author.getAge()>52)
                                 .findAny();
author.ifPresent(author->System.out.print(author.getName()));
```

### findFirst

获取流中第一个元素

```java
Optional<Author> first = authors.stream()
                                .sorted((o1,o2)->o1.getAge()-o2.getAge())
                                .findFirst();
first.ifPresent(author->System.out.print(author.getName()));
```

> 规约

| 方法                             | 说明                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| reduce(T iden, BinaryOperator b) | 可以将流中元素反复结合起来，得到一个值。 返回 T              |
| reduce(BinaryOperator b)         | 可以将流中元素反复结合起来，得到一个值。 返回 Optional<T>    |
| collect(Collector c)             | 将流转换为其他形式。接收一个 Collector接口的实现，用于给Stream中元素做汇总的方法 |

Collector 接口中方法的实现决定了如何对流执行收集操作(如收 集到 List、Set、Map)。但是 Collectors 实用类提供了很多静态方法，可以方便地创建常见收集器实例

### reduce归并

对流中的数据按照你制定的计算方式计算出一个结果，即**把stream中的元素组合起来，传入一个初始值，按照计算方式依次拿流中的元素和在初始化值的基础上进行计算，计算结果再和后面元素计算**

> **reduce单个参数的重载形式内部计算逻辑** 

```java
boolean foundAny = false;
T result = null;
for (T element : this stream) {
    if(!foundAny) {
        foundAny = true;
        result = element;
    }
    else 
        result = accumulator.apply(result, element);
}
return foundAny ? Optional.of(result) : Optional.empty();
```

 将流中第一个元素作为变量初始化值 

> **reduce两个参数的重载形式内部计算逻辑** 

```java
// 给定初始值
T result = identity;
for (T element : this stream)
    result = accumulator.apply(result, element)
return result;    
```

其中，identity是通过方法参数传入的初始值，accumulator的apply具体进行什么计算也是通过方法参数确定的

**使用reduce求和**

```java
// 双参数
List<Integer> list = Arrays.asList(1,2,3,4,5);
list.stream().reduce(0, (x1, x2) -> x1 + x2);
list.stream().reduce(0, (x1, x2) -> Integer.sum(x1, x2));
list.stream().reduce(0,Integer::sum);
```

**使用reduce员工工资求和**

```java
// 单参数
data.stream()
    .map(emp->emp.getSalary())
    .reduce((salary1, salary2)-> salary1 + salary2);
```

**使用reduce对年龄求和**

```java
authors.stream()
       .map(author->author.getAge())
       .reduce(0, new BinaryOperator<Integer>(){
           public Integer apply(Integer result, Integer element) {
               return result + element;
           }
       });

Integer sum = authors.stream()
                     .map(author->author
                     .getAge())
                     .reduce((result,element) ->result + element); 
```

**使用reduce求年龄最大值**

```java
authors.stream().map(author->author.getAge())
                .reduce(Integer.MIN_VALUE, new BinaryOperator<Integer>(){
                    public Integer apply(Integer result, Integer element) {
                        return result < element ? element : result;
                    }
                })

Integer max = authors.stream()
                     .map(author->author.getAge())
                     .reduce(Integer.MIN_VALUE,(result,element)->result < element ? element : result);
```

 **使用reduce求年龄最小值** 

```java
// 两个参数的重载实现
authors.stream().map(author->author.getAge())
                .reduce(Integer.MAX_VALUE, new BinaryOperator<Integer>() {
                    public Integer apply(Integer result, Integer element) {
                        return result > element ? element : result;
                    }
                })

authors.stream()
       .map(author->author.getAge())
       .reduce(Integer.MAX_VALUE,(result, element)->result > element ? element : result);

// 单个参数的重载实现
Optional<Integer> min = authors.stream()
                               .map(author->author.getAge())
                               .reduce(new BinaryOperator<Integer>() {
                                    public Integer apply(Integer result, Integer element) {
                                        return result > element ? element : result;
                                    }
                               });
min.ifPresent(age->System.out.println(age))
```

### collect

将流转换成集合

```java
// List
List<String> list = authors.stream()
                           .map(author->author.getName())
                           .collect(Collectors.toList());

// Set
Set<Book> books = authors.stream()
                         .flatMap(author->author.getBooks.stream())
                         .collect(Collectors.toSet());

// Map
authors.stream()
       .collect(Collectors.toMap(new Function<Author,String>(){
           public String apply(Author author) {
               return author.getName();
           }
       }, new Function<Author,List<Book>>(){
           public List<Book> apply(Author author) {
               return author.getBooks();
           }
       }));

authors.stream()
       .distinct()
       .collect(Collectors.toMap(author->author.getName(),author.getBooks()));	
```
> 注意事项

- 没有终结操作，中间操作是不会执行的
- 一旦一个流对象经过终结操作后，该流不能再被使用
- 在流中对数据进行操作，但不能影响到原来集合的元素
