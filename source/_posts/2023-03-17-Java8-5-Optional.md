---
title: Java8-5-Optional
date: 2023-03-17 22:52:57
tags: 
  - Java
categories: 
  - Language
---

Optional 类 (java.util.Optional) 是一个容器类，**代表一个值存在或不存在**， 原来用 null 表示一个值不存在，现在 Optional 可以更好的表达这个概念。并且可以避免空指针异常。  

# 常用方法

| 方法                     | 说明                                                         |
| ------------------------ | ------------------------------------------------------------ |
| `Optional.of(T t)`         | 创建一个 Optional 实例                                       |
| `Optional.empty()`         | 创建一个空的 Optional 实例                                   |
| `Optional.ofNullable(T t)` | 若 t 不为 null,创建 Optional 实例,否则创建空实例             |
| `isPresent()`              | 判断是否包含值                                               |
| `orElse(T t)`              | 如果调用对象包含值，返回该值，否则返回 t                      |
| `orElseGet(Supplier s)`    | 如果调用对象包含值，返回该值，否则返回 s 获取的值            |
| `map(Function f)`          | 如果有值对其处理，并返回处理后的Optional，否则返回 Optional.empty() |
| `flatMap(Function mapper)` | 与 map 类似，要求返回值必须是Optional                        |

# 使用

## 创建对象

Optional类似包装类，将具体的数据封装 Optional 对象内部，然后使用 Optional 内部封装号的方法操作数据，从而避免空指针异常。 

1.一般使用 Optional 静态方法`ofNullable`把数据封装成 Optional 对象，无论传入的参数是否为 null 都不会出现问题 

```java
Author author = getAuthor();
Optional<Author> authorOptional = Optional.ofNullable(author);
```

2.如果 **确定对象不是空** 可以使用 Optional 的静态方法 `of` 将数据封装成 Optional 对象 

```java
Optional<Author> authorOptional = Optional.of(author);
```

<font color="red">使用 of <b>传入的参数必须不为空</b></font>

3.如果方法的返回值为 Optional 类型，经判断发现某次计算得到的返回值为 null，需要把 null 封装成 Optional 对象返回，可以使用 Optional 的静态方法 `empty` 进行封装 

```java
Optional.empty();
```

## 安全消费值

如果获取到 Optional 对象需要使用数据，使用其 `ifPresent` 方法消费其中的值

该方法判断其内封装的数据是否为空，不空时执行具体的消费代码，具有安全性

```java
Author author = getAuthor();
Optional<Author> authorOptional = Optional.ofNullable(author);
authorOptional.ifPresent(author->System.out.println(author.getName()));
```

## 安全获取值

 期望安全获取值，推荐使用 Optional 提供的以下方法： 

> orElseGet

获取数据并且设置数据为空时的默认值，如果数据不为空就能获取到该数据，如果为空根据传入的参数创建对象作为默认返回值信息

```java
Optional<Author> authorOptional = Optional.ofNullable(getAuthor());
Author author = authorOptional.erElseGet(()->new Author());
```

> orElseThrow 

 获取数据，如果数据不为空就能获取到该数据，如果为空则根据传入的参数来创建异常抛出 

```java
Optional<Author> authorOptional = Optional.ofNullable(getAuthor());
Author author = authorOptional.erElseGet((Supplier<Throwable>) ()-> new RuntimeException("author is null"));
```

## 过滤

使用 `filter` 方法进行数据过滤，原本有数据，但不符合要求，会变成无数据的 Optional 对象 

```java
Optional<Author> authorOptional = Optional.ofNullable(getAuthor());
authorOptional.filter(author->author.getAge()>100)
              .ifPresent(author->System.out.println(author.getName()));
```

## 判断

 使用 `ifPresent` 方法进行是否存在数据的判断，如果为空返回值为 false，如果不为空，返回值为t rue

```java
Optional<Author> authorOptional = Optional.ofNullable(getAuthor());
if (authorOptional.isPresent()) {
    System.out.println(authorOptional.get().getName());
}
```

## 数据转换

 Option 提供 `map` 方法对数据进行转换，并且转换得到的数据还是被 Optional 包装好的，保证使用安全 

```java
Optional<Author> authorOptional = Optional.ofNullable(getAuthor());
Optional<List<Book>> books = authorOptional.map(author->author.getBooks());
books.ifPresent(new Consumer<List<Book>>() {
    public void accept(List<Book> books) {
        books.forEach(book->System.out.println(book.getName()));
    }
});
```

