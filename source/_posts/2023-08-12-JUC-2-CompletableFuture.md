---
title: JUC-2-CompletableFuture
date: 2023-08-12 16:54:11
tags: 
  - Java
categories: 
  - Language
---

> Future接口

Future接口（FutureTask实现类）提供一种异步并行计算的功能（获取异步任务的执行结果、取消任务的执行、判断任务是否被取消、判断任务执行是否完毕）

如果主线程需要执行耗时的计算任务，可以通过future把这任务放到异步线程中执行，主线程继续处理其他任务或者先行结束，再通过future获取计算结果。它可以为主线程开一个分支任务，专门为主线程处理耗时和费力的复杂业务

# Future接口常用实现类FutureTask异步任务

* 多线程

* 有返回值

* 异步任务

优点：future + 线程池异步多线程任务配合，能显著提高程序的执行效率

缺点：

* get()阻塞：一旦调用`get()`方法求结果，如果计算没有完成容易导致程序阻塞；
* `isDone轮询`：
  * 轮询的方式会耗费CPU资源，而且不一定能及时得到计算结果
  * 如果想要异步获取结果，通常会以轮询的方式去获取结果，尽量不要阻塞

结论：Future对于结果的获取不是很友好，只能通过阻塞或轮询的方式得到任务的结果

# CompletableFuture对Future的改进

> 回顾Future

* get()方法在Future 计算完成之前会一直处在阻塞状态下

* isDone()方法容易耗费CPU资源
* 对于真正的异步处理希望可以通过传入回调函数，在Future结束时自动调用该回调函数，就不用等待结果。

**阻塞的方式和异步编程的设计理念相违背，而轮询的方式会耗费无谓的CPU资源**。因此，JDK8设计出CompletableFuture。CompletableFuture提供了一种观察者模式类似的机制，可以让任务执行完成后通知监听的一方。

> 类架构说明

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-10.jpg)

> CompletionStage

* CompletionStage代表异步计算过程中的某一个阶段，一个阶段完成以后可能会触发另外一个阶段
* 一个阶段的计算执行可以是一个Function，Consumer或者Runnable
* 一个阶段的执行可能是被单个阶段的完成触发，也可能是由多个阶段一起触发

代表异步计算过程中的某一个阶段，一个阶段完成以后可能会触发另外一个阶段，有些类似Linux系统的管道分隔符传参数。

> CompletableFuture

* 在Java8中，CompletableFuture提供了非常强大的Future的扩展功能，可以帮助我们简化异步编程的复杂性，并且提供了函数式编程的能力，可以通过回调的方式处理计算结巢，也提供了转换和组合CompletableFuture的方法。
* 它可能代表一个明确完成的Future，也有可能代表一个完成阶段（CompletionStage )，它支持在计算完成以后触发一些函数或执行某些动作。
* 它实现了Future和CompletionStage接口

> 核心的四个静态方法创建异步任务

<table>
    <tr>
        <td>方法</td>
        <td>说明</td>
    </tr>
    <tr>
        <td rowspan="2">runAsync 无返回值</td>
        <td>public static CompletableFuture < Void> runAsync(Runnable runnable)</td>
    </tr>
    <tr>
        <td>public static CompletableFuture< Void> runAsync(Runnable runnable,Executor executor)</td>
    </tr>
     <tr>
        <td rowspan="2">supplyAsync 有返回值</td>
        <td>public static < U> CompletableFuture< U> supplyAsync(Supplier< U> supplier)</td>
    </tr>
    <tr>
        <td>public static < U> CompletableFuture< U> supplyAsync(Supplier< U> supplierExecutor executor)</td>
    </tr>
</table>

**上述Executor executor参数说明**

* 没有指定Executor的方法，直接使用默认的``ForkJoinPool.commonPool()作为它的线程池执行异步代码 
* 如果指定线程池，则使用我们自定义的或者特别指定的线程池执行异步代码

**未指定线程池 + 无返回值**

```java
public class CompletableFutureBuildDemo {

    // 
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(() -> {
            System.out.println(Thread.currentThread().getName());
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        System.out.println(completableFuture.get());
    }
}

// outputs
ForkJoinPool.commonPool-worker-9
null
```

**指定线程池 + 无返回值**

```java
public class CompletableFutureBuildDemo {

    public static void main(String[] args) throws ExecutionException, InterruptedException {

        ExecutorService threadPool = Executors.newFixedThreadPool(3);

        CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(() -> {
            System.out.println(Thread.currentThread().getName());
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, threadPool);

        System.out.println(completableFuture.get());
    }
}

pool-1-thread-1
null
```

**未指定线程池 + 有返回值**

```java
public class CompletableFutureBuildDemo {

    public static void main(String[] args) throws ExecutionException, InterruptedException {

        CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName());
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "hello, supplyAsync";
        });
        System.out.println(completableFuture.get());
    }
}

ForkJoinPool.commonPool-worker-9
hello, supplyAsync
```

**指定线程池 + 有返回值**

```java
public class CompletableFutureBuildDemo {

    public static void main(String[] args) throws ExecutionException, InterruptedException {

        ExecutorService threadPool = Executors.newFixedThreadPool(3);

        CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName());
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "hello, supplyAsync";
        }, threadPool);
        System.out.println(completableFuture.get());
    }
}
pool-1-thread-1
hello, supplyAsync
```

> 减少阻塞和轮询

从Java8开始引入了CompletableFuture，它是Future的功能增强版，减少阻塞和轮询可以传入回调对象，当异步任务完成或者发生异常时，自动调用回调对象的回调方法

**future1使用CompletableFuture完成异步任务**

```java
private static void future1() throws InterruptedException, ExecutionException {
    CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(() -> {
        System.out.println(Thread.currentThread().getName() + "----come in");
        int result = ThreadLocalRandom.current().nextInt(10);
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("----After 1 min " + result);
        return result;
    });
    System.out.println(Thread.currentThread().getName() + "finish other task firstly");
    System.out.println(completableFuture.get());
}

mainfinish other task firstly
ForkJoinPool.commonPool-worker-9----come in
----After 1 min 0
0
```

**future2使用whenComplete完成异步任务：主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭**

```java
private static void future2() {
    CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(() -> {
        System.out.println(Thread.currentThread().getName() + "----come in");
        int result = ThreadLocalRandom.current().nextInt(10);
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("----After 1 min " + result);
        return result;
    }).whenComplete((v, e)->{
        if (e == null) {
            System.out.println("计算完成，上一步计算结果是：" + v);
        }
    }).exceptionally(e -> {
        e.printStackTrace();

        System.out.println("Exception: " + e.getCause() + "\t" + e.getMessage());
        return null;
    });
    System.out.println(Thread.currentThread().getName() + "finish other task firstly");

    // 主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭
    try {
        TimeUnit.SECONDS.sleep(3);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}

ForkJoinPool.commonPool-worker-9----come in
mainfinish other task firstly
----After 1 min 6
计算完成，上一步计算结果是：6
```

**future3结合线程池使用whenComplete，无需在main结束前手动制造sleep**

```java
private static void future3() {
    ExecutorService threadPool = Executors.newFixedThreadPool(3);

    try {
        CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName() + "----come in");
            int result = ThreadLocalRandom.current().nextInt(10);
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("----After 1 min " + result);
            return result;
        }, threadPool).whenComplete((v, e)->{
            if (e == null) {
                System.out.println("计算完成，上一步计算结果是：" + v);
            }
        }).exceptionally(e -> {
            e.printStackTrace();

            System.out.println("Exception: " + e.getCause() + "\t" + e.getMessage());
            return null;
        });
        System.out.println(Thread.currentThread().getName() + " finish other task firstly");
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        threadPool.shutdown();
    }
}

pool-1-thread-1----come in
main finish other task firstly
----After 1 min 4
计算完成，上一步计算结果是：4
```

> CompletableFuture优点

* 异步任务结束时，会自动回调某个对象的方法;
* 主线程设置好回调后，不再关心异步任务的执行，异步任务之间可以顺序执行
* 异步任务出错时，会自动回调某个对象的方法;

# 电商比价实战

**[需求]**：

* 同一款产品同时搜索出同款产品在各大电商平台的售价
* 同一款产权同时搜索出本产品在同一个电商平台下各个入驻卖家的售价

**[输出]**：

请输出同款产品的在不同地方的价格清单，返回`List<String>`

**[要求]**：

* 函数编程
* 链式编程
* 流式计算

**[暴力方法]**

```java
public class CompletableFutureMallDemo {

    static List<NetMall> list = Arrays.asList(
            new NetMall("jd"),
            new NetMall("dangdang"),
            new NetMall("taobao")
    );

    /**
     * step by step 一家家搜
     */
    public static List<String> getPrice(List<NetMall> list, String produceName) {
        // 《mysql》 in taobao price is 90.12
        return list.stream()
                   .map(netMall -> String.format(produceName + " in %s price is %.2f",
                                                 netMall.getNetMallName(),
                                                 netMall.calcPrice(produceName)))
                   .collect(Collectors.toList());
    }

    public static void main(String[] args) {

        long startTime = System.currentTimeMillis();

        List<String> lists = getPrice(CompletableFutureMallDemo.list, "mysql");
        for (String ele : lists) {
            System.out.println(ele);
        }
        long endTime = System.currentTimeMillis();


        System.out.println("**********costTime: " + (endTime - startTime) + " ms");
    }

}

@AllArgsConstructor
class NetMall {

    @Getter
    private String netMallName;

    public double calcPrice(String productName) {
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return ThreadLocalRandom.current().nextDouble() * 2 + productName.charAt(0);
    }

}

mysql in jd price is 109.90
mysql in dangdang price is 110.09
mysql in taobao price is 109.20
**********costTime: 3108 ms
```

**[优化性能]**

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-11.jpg)

```text
mysql in jd price is 109.23
mysql in dangdang price is 109.66
mysql in taobao price is 109.11
**********costTime1: 3081 ms

****************************
mysql in jd price is 110.37
mysql in dangdang price is 110.25
mysql in taobao price is 109.79
**********costTime2: 1008 ms
```

# CompletableFuture常用方法

## 获取结果和触发计算

> 获取结果

| 方法                                        | 说明                                                         |
| ------------------------------------------- | ------------------------------------------------------------ |
| `public T get()`                            | 完成后返回结果值（不会抛出异常）                             |
| `public T get(long timeout, TimeUnit unit)` | 完成后返回结果值（不会抛出异常，可以设置超时）               |
| `public T join()`                           | 完成后返回结果值，如果完成异常，则抛出（未选中）异常         |
| `public T getNow(T valueIdAbsent)`          | 没有计算完成时给一个替代的结果；立刻获取结果不阻塞，计算完成，返回结果，未计算完成，返回设置的valueIfAbsent值 |

```java
// Waits if necessary for this future to complete, and then returns its result.
public T get() throws InterruptedException, ExecutionException {
    Object r;
    return reportGet((r = result) == null ? waitingGet(true) : r);
}

// Waits if necessary for at most the given time for this future to complete, and then returns its result, if available.
public T get(long timeout, TimeUnit unit)
    throws InterruptedException, ExecutionException, TimeoutException {
    Object r;
    long nanos = unit.toNanos(timeout);
    return reportGet((r = result) == null ? timedGet(nanos) : r);
}

// Returns the result value when complete, or throws an (unchecked) exception if completed exceptionally. To better conform with the use of common functional forms, if a computation involved in the completion of this CompletableFuture threw an exception, this method throws an (unchecked) {@link CompletionException} with the underlying exception as its cause.
public T join() {
    Object r;
    return reportJoin((r = result) == null ? waitingGet(false) : r);
}


// Returns the result value (or throws any encountered exception) if completed, else returns the given valueIfAbsent.
public T getNow(T valueIfAbsent) {
    Object r;
    return ((r = result) == null) ? valueIfAbsent : reportJoin(r);
}
```



> 主动触发计算

| 方法                               | 说明//                         |
| ---------------------------------- | ------------------------------ |
| `public boolean complete(T value)` | 是否打断get方法，立即返回value |

```java
// If not already completed, sets the value returned by {@link#get()} and related methods to the given value.
public boolean complete(T value) {
    boolean triggered = completeValue(value);
    postComplete();
    return triggered;
}
```

## 对计算结果进行处理

> thenApply

| 方法                                        | 说明                                                         |
| ------------------------------------------- | ------------------------------------------------------------ |
| `public <U> CompletableFuture<U> thenApply` | 返回一个新的 CompletionStage，当该阶段正常完成时，会将该阶段的结果作为所提供函数的参数执行。一旦执行过程中出现异常则会停止 |



```java
// Returns a new CompletionStage that, when this stage completes normally, is executed with this stage's result as the argument to the supplied function.
public <U> CompletableFuture<U> thenApply(
    Function<? super T,? extends U> fn) {
    return uniApplyStage(null, fn);
}
```

```java
public class CompletableFutureAPIDemo {

    public static void main(String[] args) {

        ExecutorService threadPool = Executors.newFixedThreadPool(3);

        CompletableFuture.supplyAsync(() -> {

            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("第一步");
            return 1;
        }, threadPool).thenApply(f -> {
            System.out.println("第二步");
            return f + 2;
        }).thenApply(f -> {
            System.out.println("第三步");
            return f + 3;
        }).whenComplete((v, e) -> {
            if (e == null) {
                System.out.println("计算结果：" + v);
            }
        }).exceptionally(e -> {
            e.printStackTrace();
            System.out.println(e.getMessage());
            return null;
        });

        System.out.println(Thread.currentThread().getName() + "*****主线程先区忙其他任务");

        threadPool.shutdown();
    }
}

main*****主线程先区忙其他任务
第一步
第二步
第三步
计算结果：6
```



> handle

| 方法                                     | 说明                                                         |
| ---------------------------------------- | ------------------------------------------------------------ |
| `public <U> CompletableFuture<U> handle` | 返回一个新的 CompletionStage，当该阶段正常完成或异常完成时，将以该阶段的结果和异常作为参数执行所提供的函数。本阶段完成后，以本阶段的结果和异常为参数调用给定函数，函数结果用于完成返回的阶段。（有异常可以继续往下走，根据带的异常参数可以进一步处理） |

```java
// Returns a new CompletionStage that, when this stage completes either normally or exceptionally, is executed with this stage's result and exception as arguments to the supplied function.
// When this stage is complete, the given function is invoked with the result (or {@code null} if none) and the exception (or{@code null} if none) of this stage as arguments, and the function's result is used to complete the returned stage.
public <U> CompletableFuture<U> handle(
    BiFunction<? super T, Throwable, ? extends U> fn) {
    return uniHandleStage(null, fn);
}
```

## 对计算结果进行消费

接收任务的处理结果，并消费处理，无返回结果

> thenAccept

| 方法                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `public CompletableFuture<Void> thenAccept(Consumer<? super T> action)` | 返回一个新的 CompletionStage，当该阶段正常完成时，将以该阶段的结果为参数执行所提供的操作。 |

```java
// Returns a new CompletionStage that, when this stage completes normally, is executed with this stage's result as the argument to the supplied action.
public CompletableFuture<Void> thenAccept(Consumer<? super T> action) {
    return uniAcceptStage(null, action);
}
```



```java
public class CompletableFutureAPIDemo {

    public static void main(String[] args) {

        CompletableFuture.supplyAsync(() -> {

            System.out.println("第一步");
            return 1;
        }).thenApply(f -> {

            System.out.println("第二步");
            return f + 2;
        }).thenApply(f -> {

            System.out.println("第三步");
            return f + 3;
        }).thenAccept(r -> {
            System.out.println(r);
        });

    }
}
第一步
第二步
第三步
6
```

> 任务之间的顺序执行

| 方法                         | 说明                                              |
| ---------------------------- | ------------------------------------------------- |
| thenRun(Runnable)            | 任务A执行完执行B，并且B不需要A的结果              |
| thenAccept(Consumer  action) | 任务A执行完执行B，B需要A的结果，但是任务B无返回值 |
| thenApply(Function fn)       | 任务A执行完执行B，B需要A的结果，但是任务B有返回值 |

```java
// null
System.out.println(CompletableFuture.supplyAsync(()->"resultA").thenRun(() -> {}).join());
// resultAnull
System.out.println(CompletableFuture.supplyAsync(()->"resultA").thenAccept(r -> System.out.print(r)).join());
// resultAresult B
System.out.println(CompletableFuture.supplyAsync(()->"resultA").thenApply(r -> r + "result B").join());
```

> CompletableFuture和线程池说明

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/06-juc-20230402-12.jpg)

1. 如果没有传入自定义的线程池，都使用默认的线程池ForkJoinPool
2. 如果传入自定义线程池
   * 如果执行第一个任务时传入一个自定义线程池
   * 调用`thenRun`方法执行第二个任务时，则第二个任务和第一个任务是共用同一个线程池
   * 调用`thenRunAsync`方法执行第二个任务时，则第一个任务使用自己传入的线程池，第二个任务使用默认的线程池ForkJoinPool
3. 备注：有可能处理太快，系统优化切换原则，直接使用main线程处理

## 对计算速度进行选用

谁快用谁

> applyToEither

| 方法                                            | 说明                                                         |
| ----------------------------------------------- | ------------------------------------------------------------ |
| `public <U> CompletableFuture<U> applyToEither` | 返回一个新的 CompletionStage，当此阶段或其他给定阶段正常完成时，该阶段将被执行，并将相应的结果作为所提供函数的参数。 |



```java
// Returns a new CompletionStage that, when either this or the other given stage complete normally, is executed with the corresponding result as argument to the supplied function.
public <U> CompletableFuture<U> applyToEither(
    CompletionStage<? extends T> other, Function<? super T, U> fn) {
    return orApplyStage(null, other, fn);
}
```

```java
public class CompletableFutureAPIDemo {

    public static void main(String[] args) {

        CompletableFuture<String> playerA = CompletableFuture.supplyAsync(() -> {
            System.out.println("A come in");
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "playerA";
        });

        CompletableFuture<String> playerB = CompletableFuture.supplyAsync(() -> {
            System.out.println("b come in");
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "playerB";
        });

        CompletableFuture<String> result = playerA.applyToEither(playerB, f -> {
            return f +  " is winner";
        });

        System.out.println(Thread.currentThread().getName() + "\t" + " ----- " + result.join());

    }
}

A come in
b come in
main	 ----- playerA is winner
```

## 对计算结果进行合并

两个CompletionStage任务完成后，最终能把两个任务的结果一起交给`thenCombine`来处理。先完成的先等待其他分支任务

> thenCombine

| 方法                                            | 说明                                                         |
| ----------------------------------------------- | ------------------------------------------------------------ |
| `public <U,V> CompletableFuture<V> thenCombine` | 返回一个新的 CompletionStage，当此阶段和另一个给定阶段都正常完成时，将以这两个结果为参数执行所提供的函数。 |

```java
// Returns a new CompletionStage that, when this and the other given stage both complete normally, is executed with the two results as arguments to the supplied function.
public <U,V> CompletableFuture<V> thenCombine(
    CompletionStage<? extends U> other,
    BiFunction<? super T,? super U,? extends V> fn) {
    return biApplyStage(null, other, fn);
}
```



```java
public class CompletableFutureAPIDemo {

    public static void main(String[] args) {

        CompletableFuture<Integer> completableFuture1 = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName() + "\t *** 启动");
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return 10;
        });

        CompletableFuture<Integer> completableFuture2 = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName() + "\t *** 启动");
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return 20;
        });

        CompletableFuture<Integer> result = completableFuture1.thenCombine(completableFuture2, (x, y) -> {
            System.out.println("**** 开始两个结果合并");
            return x + y;
        });
        System.out.println(result.join());
    }
}

ForkJoinPool.commonPool-worker-9	 *** 启动
ForkJoinPool.commonPool-worker-2	 *** 启动
**** 开始两个结果合并
30
```


