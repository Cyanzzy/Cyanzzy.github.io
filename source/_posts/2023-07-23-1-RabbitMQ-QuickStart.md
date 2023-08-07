---
title: 1 RabbitMQ-QuickStart
date: 2023-07-23 16:58:30
tags: 
  - MQ
categories: 
  - Technology
swiper_index: 
---

# 消息队列

## MQ的概念

### 基本介绍

* MQ本质是个队列，FIFO 先入先出，只不过队列中存放的内容是 message 而已
* 是一种跨进程的通信机制，用于上下游传递消息。
* 在互联网架构中，MQ 是一种常见的**上下游“逻辑解耦+物理解耦”的消息通信服务**。
* 使用 MQ 之后，消息发送上游只需要依赖 MQ，不用依赖其他服务。 

### 使用原因

> **流量消峰** 

订单系统为例，如果订单系统最多能处理一万次订单，这个处理能力应付正常时段的下单时绰绰有余，正常时段我们下单一秒后就能返回结果。

但是在高峰期，如果有两万次下单操作系统是处理不了的，只能限制订单超过一万后不允许用户下单。

**使用消息队列做缓冲**，可以取消这个限制，把一秒内下的订单分散成一段时间来处理，这时有些用户可能在下单十几秒后才能收到下单成功的操作，但是比不能下单的体验要好。  

> 应用解耦 

电商应用为例，应用中有订单系统、库存系统、物流系统、支付系统。用户创建订单后，如果耦合调用库存系统、物流系统、支付系统，任何一个子系统出了故障，都会造成下单操作异常。

**当转变成基于消息队列的方式后**，系统间调用的问题会减少很多，比如物流系统因为发生故障，需要几分钟来修复。

在这几分钟的时间里，**物流系统要处理的内存被缓存在消息队列中**，用户的下单操作可以正常完成。当物流系统恢复后，继续处理订单信息即可，用户感受不到物流系统的故障，提升系统的可用性。 

> 异步处理   

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-08.png)

### MQ分类

> ActiveMQ 

**优点**：

* 单机吞吐量万级，时效性 ms 级，可用性高，基于主从架构实现高可用性，消息可靠性较低的概率丢失数据 

**缺点**：

* 官方社区现在对 ActiveMQ 5.x 维护越来越少，高吞吐量场景较少使用。  

> Kafka 

**优点**: 性能卓越，单机写入 TPS 约在百万条/秒

* 最大的优点，就是吞吐量高。
* 时效性 ms 级可用性非常高，kafka 是分布式的，一个数据多个副本，少数机器宕机，不会丢失数据，不会导致不可用，消费者采 用 Pull 方式获取消息，消息有序，通过控制能够保证所有消息被消费且仅被消费一次
* 有优秀的第三方 Kafka Web 管理界面 Kafka-Manager
* 在日志领域比较成熟，被多家公司和多个开源项目使用；

**缺点**：

* Kafka 单机超过 64 个队列/分区，Load 会发生明显的飙高现象，队列越多，load 越高，发送消息响应时间变长，使用短轮询方式，实时性取决于轮询间隔时间，消费失败不支持重试；
* 支持消息顺序， 但是一台代理宕机后，就会产生消息乱序，社区更新较慢； 

* **功能支持**： 
  * 功能较为简单，主要支持简单的 MQ 功能，在**大数据领域**的实时计算以及日志采集被大规模使用 

> RocketMQ 

**优点**：

* 单机吞吐量十万级，可用性非常高，分布式架构，消息可以做到 0 丢失
* MQ 功能较为完善，还是分 布式的，扩展性好，支持10 亿级别的消息堆积，不会因为堆积导致性能下降

**缺点**：

* 支持的客户端语言不多，目前是 java 及 c++
* 社区活跃度一般，没有在 MQ 核心中去实现 JMS 等接口，有些系统要迁移需要修改大量代码 

> RabbitMQ 

**优点**：

* 由于 erlang 语言的高并发特性，性能较好
* 吞吐量到万级，MQ 功能比较完备，健壮、稳定、易用、跨平台、支持多种语言 如：Python、Ruby、.NET、Java、JMS、C、PHP、ActionScript、XMPP、STOMP 等，支持 AJAX 文档齐全
* 开源提供的管理界面非常棒，用起来很好用,社区活跃度高
* 更新频率相当高 

### 如何选择

> Kafka

 Kafka 主要特点是基于 Pull 的模式来处理消息消费，追求高吞吐量，一开始的目的就是用于日志收集和传输，适合产生大量数据的互联网服务的数据收集业务。

> RocketMQ 

天生为金融互联网领域而生，对于可靠性要求很高的场景，尤其是电商里面的订单扣款，以及业务削峰，在大量交易涌入时，后端可能无法及时处理的情况。RoketMQ 在稳定性上可能更值得信赖

> RabbitMQ 

结合 erlang 语言本身的并发优势，性能好时效性微秒级，社区活跃度也比较高，管理界面用起来十分方便

## RabbitMQ

> RabbitMQ概念

RabbitMQ是由erlang语言开发，基于AMQP（Advanced Message Queue 高级消息队列协议）协议实现的消息队列，它是一种应用程序之间的通信方法，消息队列在分布式系统开发中应用非常广泛 

> 核心概念

**生产者** 

产生数据发送消息的程序是生产者 

**交换机** 

* 一方面它接收来自生产者的消息，另一方面它将消息推送到队列中
* 交换机必须确切知道**如何处理它接收到的消息**，是将这些消息推送到特定队列还是推送到多个队列，亦或者是把消息丢弃 

**消费者** 

消费者大多时候是一个等待接收消息的程序。请注意生产者，消费者和消息中间件很多时候并不在同一机器上。同一个应用程序既可以是生产者又是可以是消费者。 

**队列**

* 队列是 RabbitMQ 内部使用的一种数据结构，尽管消息流经 RabbitMQ 和应用程序，但它们只能存储在队列中
* 队列仅受主机的内存和磁盘限制的约束，本质上是一个大的消息缓冲区
* 许多生产者可 以将消息发送到一个队列，许多消费者可以尝试从一个队列接收数据。这就是我们使用队列的方式 

### RabbitMQ核心

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-09.png)

### 工作原理

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-10.png)

> **Connection**：

publisher／consumer 和 broker 之间的 **TCP 连接** 

> **Channel**：

* 如果每一次访问 RabbitMQ 都建立一个 Connection，在消息量大的时候建立 TCP Connection 的开销将是巨大的，效率也较低
* Channel 是在 connection 内部建立的逻辑连接，如果应用程序支持多线程，通常每个 thread 创建单独的 channel 进行通讯，AMQP method 包含了 channel id 帮助客 户端和 message broker 识别 channel，所以 channel 之间是完全隔离的
* Channel 作为轻量级的 Connection 极大减少了操作系统建立 TCP connection 的开销  

> **Broker**：

接收和分发消息的应用，RabbitMQ Server 就是 Message Broker 

> **Virtual host**：

* 出于多租户和安全因素设计的，把 AMQP 的基本组件划分到一个虚拟的分组中，类似于网络中的 namespace 概念
* 当多个不同的用户使用同一个 RabbitMQ server 提供的服务时，可以划分出 多个 vhost，每个用户在自己的 vhost 创建 exchange／queue 等 》

> **Exchange**：

* message 到达 broker 的第一站，根据分发规则，匹配查询表中的 routing key，分发 消息到 queue 中去。
* 常用的类型有：direct (point-to-point), topic (publish-subscribe) and fanout (multicast) 

> **Queue**：

消息最终被送到这里等待 consumer 取走 

> **Binding**：

exchange 和 queue 之间的虚拟连接，binding 中可以包含 routing key，Binding 信息被保 存到 exchange 中的查询表中，用于 message 的分发依据 



# Hello World

##  依赖

```xml
  <!--指定 jdk 编译版本-->
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <dependencies>
        <!--rabbitmq 依赖客户端-->
        <dependency>
            <groupId>com.rabbitmq</groupId>
            <artifactId>amqp-client</artifactId>
            <version>5.8.0</version>
        </dependency>
        <!--操作文件流的一个依赖-->
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.6</version>
        </dependency>
    </dependencies>
```

## 消息生产者

```java
public class Producer {

    public static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws TimeoutException, IOException {
        // 创建工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 工厂Ip 连接队列
        factory.setHost("127.0.0.1");
        // 用户名
        factory.setUsername("guest");
        // 密码
        factory.setPassword("guest");

        // 创建连接
        Connection connection = factory.newConnection();
        // 获取信道
        Channel channel = connection.createChannel();

        /**
         * Declare a queue
         * 
         * @param queue the name of the queue
         * @param durable true if we are declaring a durable queue (the queue will survive a server restart)
         * @param exclusive true if we are declaring an exclusive queue (restricted to this connection)
         * @param autoDelete true if we are declaring an autodelete queue (server will delete it when no longer in use)
         * @param arguments other properties (construction arguments) for the queue
         * @return a declaration-confirm method to indicate the queue was successfully declared
         * */
        channel.queueDeclare(QUEUE_NAME, false, false, false,null);

        // 发消息
        String message = "Hello World";

        /**
         * Publish a message.
         *
         * @param exchange the exchange to publish the message to
         * @param routingKey the routing key
         * @param props other properties for the message - routing headers etc
         * @param body the message body
         * @throws java.io.IOException if an error is encountered
         */
        channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
        
        System.out.println("Send Success!");

    }
}
```

## 消息消费者

```java
public class Consumer {

    private static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {
        // 创建工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 相关配置
        factory.setHost("127.0.0.1");
        factory.setUsername("guest");
        factory.setPassword("guest");

        // 创建连接
        Connection connection = factory.newConnection();
        // 创建信道
        Channel channel = connection.createChannel();

        // 声明 接受消息的回调
        DeliverCallback deliverCallback = (consumerTag, message) -> {
            System.out.println(new String(message.getBody()));
        };
        // 声明 取消消息的回调
        CancelCallback cancelCallback = consumerTag -> {
            System.out.println("消息消费中断~");
        };

        /**
         * Start a non-nolocal, non-exclusive consumer, with
         * a server-generated consumerTag.
         *
         * @param queue the name of the queue
         * @param autoAck true if the server should consider messages
         * acknowledged once delivered; false if the server should expect
         * explicit acknowledgements
         * @param deliverCallback callback when a message is delivered
         * @param cancelCallback callback when the consumer is cancelled
         * @return the consumerTag generated by the server
         */
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);
    }
}
```

# Work Queues

工作队列的主要思想是避免立即执行资源密集型任务，而不得不等待它完成。 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-12.png)

**注意事项 一个消息只能被处理一次，不可以被处理多次**

## 轮询分发消息

> 抽取工具类

```java
public class RabbitMQUtils {

    public static Channel getChannel() throws Exception {
        // 创建工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 相关配置
        factory.setHost("127.0.0.1");
        factory.setUsername("guest");
        factory.setPassword("guest");

        // 创建连接
        Connection connection = factory.newConnection();
        // 创建信道
        Channel channel = connection.createChannel();

        return channel;
    }
}
```

> 工作线程

```java
public class Worker {

    // 队列名称
    private final static String QUEUE_NAME = "hello";

    // 接收消息
    public static void main(String[] argv) throws Exception {

        Channel channel = RabbitMQUtils.getChannel();

        // 接收消息
        DeliverCallback deliverCallback = (consumerTag, message) -> {
            System.out.println("deliverCallback：" + new String(message.getBody()));
        };

        // 取消消息接收
        CancelCallback cancelCallback = consumerTag -> {
            System.out.println("cancelCallback");
        };

        channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);
    }
}
```

> 生产者

```java
public class NewTask {

    // 队列名称
    private final static String QUEUE_NAME = "hello";

    // 接收消息
    public static void main(String[] argv) throws Exception {

        // 创建信道
        Channel channel = RabbitMQUtils.getChannel();

        // 声明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false,null);

        Scanner scanner = new Scanner(System.in);

        while (scanner.hasNext()) {
            String message = scanner.next();

            channel.basicPublish("", QUEUE_NAME, null, message.getBytes());

            System.out.println("Send Success!" + message);
        }

    }
}
```

## 消息应答

> 消息应答

**为了保证消息在发送过程中不丢失**，rabbitmq 引入消息应答机制，消息应答就是：消费者在接收到消息并且处理该消息之后，告诉 rabbitmq 它已经处理了，rabbitmq 可以把该消息删除 

### 自动应答（不推荐）

消息发送后**立即被认为已经传送成功**

这种模式需要在高吞吐量和数据传输安全性方面做权衡，因为这种模式如果消息在接收到之前，消费者那边出现连接或者 channel 关闭，那么消息就丢失了

当然另一方面这种模式消费者那边可以传递过载的消息，没有对传递的消息数量进行限制， 当然这样有可能使得消费者这边由于接收太多还来不及处理的消息，导致这些消息的积压，最终使得内存耗尽，最终这些消费者线程被操作系统杀死

所以这种模式**仅适用在消费者可以高效并以某种速率能够处理这些消息的情况下使用**。

### 消息应答方法（手动应答）

* `Channel.basicAck` **(用于肯定确认)** RabbitMQ 已知道该消息并且成功的处理消息，可以将其丢弃了
* `Channel.basicNack`**(用于否定确认)**
* `Channel.basicReject` **(用于否定确认)** 与 Channel.basicNack 相比少一个参数不处理该消息直接拒绝，可以将其丢弃

> 何为Mutiple

手动应答的好处是可以批量应答并且减少网络拥堵  

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-13.png)

参数`mutiple`

* `true` 表示应答channel上未应答的消息

  * 比如说 channel 上有传送 tag 的消息 5,6,7,8 当前 tag 是 8 那么此时 5-8 的这些**还未应答的消息都会被确认收到消息应答** 、

    ![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-14.png)

* `false` 表示不应答channel上未应答的消息

  * 只会应答 tag=8 的消息 5,6,7 这三个消息依然不会被确认收到消息应答 

    ![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-15.png)

> 消息自动重新入队

如果**消费者**由于某些原因**失去连接**(其通道已关闭，连接已关闭或 TCP 连接丢失)，导致消息未发送ACK 确认，**RabbitMQ** 将了解到消息未完全处理，并将**对其重新排队**。如果此时其他消费者可以处理，它将很快将其重新分发给另一个消费者。这样，即使某个消费者偶尔死亡，也可以确保不会丢失任何消息。 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-16.png)

> 消息手动应答案例

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-17.png)

**生产者**

```java
public class Producer {

    // 队列名称
    private final static String QUEUE_NAME = "hello";

    // 发送消息
    public static void main(String[] argv) throws Exception {

        // 创建信道
        Channel channel = RabbitMQUtils.getChannel();

        // 声明队列
        channel.queueDeclare(QUEUE_NAME, false, false,false, null);

        Scanner scanner = new Scanner(System.in);

        while (scanner.hasNext()) {
            String message = scanner.next();

            channel.basicPublish("", QUEUE_NAME, null, message.getBytes("UTF-8"));

            System.out.println("Send Success: " + message);
        }

    }

}
```

**消费者1**

```java
public class Consumer1 {


    // 队列名称
    private final static String QUEUE_NAME = "hello";

    // 接收消息
    public static void main(String[] argv) throws Exception {

        // 创建信道
        Channel channel = RabbitMQUtils.getChannel();

        System.out.println("Consumer1 Receive Message Cost less time");

        DeliverCallback deliverCallback = (consumerTag, message) -> {

            SleepUtils.sleep(1);
            System.out.println("Consumer1--deliverCallback：" + new String(message.getBody(), "UTF-8"));

            /**
             * Acknowledge one or several received messages.
             *
             * @param deliveryTag the tag from the received
             * @param multiple true to acknowledge all messages up to and
             * including the supplied delivery tag; false to acknowledge just
             * the supplied delivery tag.
             */
            channel.basicAck(message.getEnvelope().getDeliveryTag(), false);
        };

        CancelCallback cancelCallback = consumerTag -> {
            System.out.println(consumerTag + "Consumer1--cancelCallback");
        };

        // 手动应答
        boolean autoAck = false;
        channel.basicConsume(QUEUE_NAME, autoAck, deliverCallback, cancelCallback);

    }

}
```

**消费者2**

```java
public class Consumer2 {

    // 队列名称
    private final static String QUEUE_NAME = "hello";

    // 接收消息
    public static void main(String[] argv) throws Exception {

        // 创建信道
        Channel channel = RabbitMQUtils.getChannel();

        System.out.println("Consumer2 Receive Message Cost more time");

        DeliverCallback deliverCallback = (consumerTag, message) -> {

            SleepUtils.sleep(30);
            System.out.println("Consumer2--deliverCallback：" + new String(message.getBody(), "UTF-8"));

            /**
             * Acknowledge one or several received messages.
             *
             * @param deliveryTag the tag from the received
             * @param multiple true to acknowledge all messages up to and
             * including the supplied delivery tag; false to acknowledge just
             * the supplied delivery tag.
             */
            channel.basicAck(message.getEnvelope().getDeliveryTag(), false);
        };

        CancelCallback cancelCallback = consumerTag -> {
            System.out.println(consumerTag + "Consumer2--cancelCallback");
        };

        // 手动应答
        boolean autoAck = false;
        channel.basicConsume(QUEUE_NAME, autoAck, deliverCallback, cancelCallback);

    }

}
```

##  队列持久化

### 持久化概念

**默认情况下** RabbitMQ 退出或由于某种原因崩溃时，它忽视队列和消息，除非告知它不要这样做。确保消息不会丢失：需要将队列和消息都标记为**持久化**。 

### 队列持久化

* 若创建的队列都是非持久化的，rabbitmq 如果重启，该队列就会被删除掉

* 如果要队列实现持久化需要在声明队列的时候把 **durable** 参数设置为持久化 

```java
// MQ持久化
boolean durable = true;
channel.queueDeclare(ACK_QUEUE_NAME, durable, false, false, null);
```

**注意**   如果**之前声明的队列不是持久化**的，需要把**原先队列**先删除，或者重新创建一个持久化的队列，不然就会出现错误 

> 控制台中 持久化与非持久化队列的 UI 显示区、 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-18.png)

### 消息持久化

**让消息实现持久化需要在消息生产者修改代码**，`MessageProperties.PERSISTENT_TEXT_PLAIN` 添加这个属性。 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-19.png)

将消息标记为持久化**并不能完全保证**不会丢失消息。尽管它告诉 RabbitMQ 将消息保存到磁盘，但是这里依然存在当消息刚准备存储在磁盘的时候但是还没有存储完，消息还在缓存的一个间隔点。**此时并没有真正写入磁盘**。持久性保证并不强 

### 不公平分发 

>  场景模拟

在某种场景下，有两个消费者在处理任务，其中有个消费者 1 处理任务的速度非常快，而另外一个消费者 2 处理速度却很慢，这个时候我们还是采用轮询分发的话就会到这处理速度快的这个消费者很大一部分时间处于空闲状态，而处理慢的那个消费者一直在干活，但是 RabbitMQ 并不知道这种情况它依然很公平的进行分发。  

> 解决方案

 为了避免这种情况，可以**在消费者**方设置参数 `channel.basicQos(1); `

```java
int prefetchCount = 1;
channel.basicQos(prefetchCount);
```

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-20.png)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-21.png)

如果**这个任务我还没有处理完或者我还没有应答你**，你先别分配给我，我目前只能处理一个任务，然后 rabbitmq 就会把该任务分配给不忙的空闲消费者，当然如果所有的消费者都没有完成手上任务，队列还在不停的添加新任务，队列有可能就会遇到队列被撑满的情况，这个时候就只能添加新的 worker 或者改变其他存储任务的策略。  

### 预取值

> 消息异步发送

本身消息的发送就是异步发送的，所以在任何时候，channel 上肯定不止只有一个消息，另外来自消费者的**手动确认本质上也是异步的**。

因此这里就**存在一个未确认的消息缓冲区**，因此希望能限制此缓冲区的大小，**以避免缓冲区里面无限制的未确认消息问题**。

> 预取值

通过使用 `basic.qos` 方法设置**“预取计数”值**来完成该需求。该值**定义通道上允许的未确认消息的最大数量**。一旦数量达到配置的数量， RabbitMQ 将停止在通道上传递更多消息，除非至少有一个未处理的消息被确认 。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-22.png)

# 发布确认

## 发布确认原理

生产者将信道设置成 `confirm` 模式，一旦信道进入 `confirm` 模式，**所有在该信道上面发布的消息都将会被指派一个唯一的 ID(从 1 开始)**，一旦消息被投递到所有匹配的队列之后，`broker` **就会发送一个确认给生产者**(包含消息的唯一 ID)，这就使得生产者知道消息已经正确到达目的队列 

如果消息和队列是可持久化的，那么**确认消息会在将消息写入磁盘之后发出**，`broker` 回传给生产者的确认消息中 `delivery-tag` 域包含了确认消息的序列号，此外 `broker` 也可以设置 **basic.ack** 的 `multiple` 域，表示到这个序列号之前的所有消息都已经得到了处理。 

> confirm模式优点

`confirm` 模式是异步的，一旦发布一条消息，生产者应用程序就可以在等信道返回确认的同时继续发送下一条消息，当消息最终得到确认之后，生产者应用便可以通过回调方法来处理该确认消息，如果 RabbitMQ 因为自身内部错误导致消息丢失，就会发送一条 `nack` 消 息，生产者应用程序同样可以在回调方法中处理该 nack 消息。 

##  发布确认的策略

### 开启发布确认的方法

发布确认**默认是没有开启的**，如果要开启需要调用方法 `confirmSelect`，每当你要想使用发布确认，都需要在 channel 上调用该方法  

```java
Channel channel = connection.createChannel();
channel.confirmSelect();
```

### 单个发布

它是一种**同步确认发布**的方式，也就是发布一个消息之后只有**它被确认发布**，后续的消息才能继续发布

`waitForConfirmsOrDie(long)`这个方法只有在消息被确认的时候才返回，如果在指定时间范围内这个消息没有被确认那么它将抛出异常。  

> 缺点

* **发布速度特别的慢**，因为如果没有确认发布的消息就会阻塞所有后续消息的发布

* 最多提供每秒不超过数百条发布消息的吞吐量 

```java
public static void publishMessageIndividually() throws Exception {
     try (Channel channel = RabbitMqUtils.getChannel()) {
         // 声明队列
         String queueName = UUID.randomUUID().toString();
         channel.queueDeclare(queueName, false, false, false, null);
         
         // 开启发布确认
         channel.confirmSelect();
         // 开始时间
         long begin = System.currentTimeMillis();
         for (int i = 0; i < MESSAGE_COUNT; i++) {
             String message = i + "";
             channel.basicPublish("", queueName, null, message.getBytes());
             // 服务端返回 false 或超时时间内未返回，生产者可以消息重发
             boolean flag = channel.waitForConfirms();
             if(flag){
             	System.out.println("消息发送成功");
             }
         }
         // 结束时间
         long end = System.currentTimeMillis();
         System.out.println("发布" + MESSAGE_COUNT + "个单独确认消息,耗时" + (end - begin) +
        "ms");
     }
}
```

### 批量发布

与单个等待确认消息相比，先发布一批消息然后一起确认可以极大地提高吞吐量

> 缺点

当发生故障导致发布出现问题时，不知道是哪个消息出现问题，必须将整个批处理保存在内存中，以记录重要的信息而后重新发布消息。当然这种方案仍然是**同步**的，也一样阻塞消息的发布。 

```java
public static void publishMessageBatch() throws Exception {
     try (Channel channel = RabbitMqUtils.getChannel()) {
         // 声明队列
         String queueName = UUID.randomUUID().toString();
         channel.queueDeclare(queueName, false, false, false, null);
         // 开启发布确认
         channel.confirmSelect();
         // 批量确认消息大小
         int batchSize = 100;
         // 未确认消息个数
         int outstandingMessageCount = 0;
         // 开始时间
         long begin = System.currentTimeMillis();
         // 批量发布
         for (int i = 0; i < MESSAGE_COUNT; i++) {
             String message = i + "";
             channel.basicPublish("", queueName, null, message.getBytes());
             outstandingMessageCount++;
             if (outstandingMessageCount == batchSize) {
                 // 发布确认
                 channel.waitForConfirms();
                 outstandingMessageCount = 0;
             }
         }
         //为了确保还有剩余没有确认消息 再次确认
         if (outstandingMessageCount > 0) {
         channel.waitForConfirms();
         }
         long end = System.currentTimeMillis();
         System.out.println("发布" + MESSAGE_COUNT + "个批量确认消息,耗时" + (end - begin) +
        "ms");
     }
}
```

### 异步发布

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-24.png)

```java
public static void publishMessageAsync() throws Exception {
    try (Channel channel = RabbitMQUtils.getChannel()) {
        // 声明队列
        String queueName = UUID.randomUUID().toString();
        channel.queueDeclare(queueName, false, false, false, null);
        // 开启发布确认
        channel.confirmSelect();

        /**
         * 线程安全有序的一个哈希表，适用于高并发的情况
         *
         * 1.轻松的将序号与消息进行关联
         * 2.轻松批量删除条目 只要给到序列号
         * 3.支持并发访问
         */
        ConcurrentSkipListMap<Long, String> outstandingConfirms = new ConcurrentSkipListMap<>();

        /**
         * 确认收到消息的一个回调
         *
         * 1.消息序列号
         * 2.true 可以确认小于等于当前序列号的消息
         * 3.false 确认当前序列号消息
         */
        ConfirmCallback ackCallback = (sequenceNumber, multiple) -> {
            if (multiple) {
                // 返回的是小于等于当前序列号的未确认消息 是一个 map
                ConcurrentNavigableMap<Long, String> confirmed =
                        outstandingConfirms.headMap(sequenceNumber, true);
                // 清除该部分未确认消息
                confirmed.clear();
            } else {
                // 只清除当前序列号的消息
                outstandingConfirms.remove(sequenceNumber);
            }
        };

        ConfirmCallback nackCallback = (sequenceNumber, multiple) -> {
            String message = outstandingConfirms.get(sequenceNumber);
            System.out.println("发布的消息" + message + "未被确认，序列号" + sequenceNumber);
        };

        /**
         * 添加一个异步确认的监听器
         *
         * 1.确认收到消息的回调
         * 2.未收到消息的回调
         */
        channel.addConfirmListener(ackCallback, null);

        long begin = System.currentTimeMillis();

        for (int i = 0; i < MESSAGE_COUNT; i++) {
            String message = "消息" + i;
            /**
             * channel.getNextPublishSeqNo()获取下一个消息的序列号
             * 通过序列号与消息体进行一个关联
             * 全部都是未确认的消息体
             */
            outstandingConfirms.put(channel.getNextPublishSeqNo(), message);
            channel.basicPublish("", queueName, null, message.getBytes());
        }

        long end = System.currentTimeMillis();
        System.out.println("发布" + MESSAGE_COUNT + "个异步确认消息,耗时" + (end - begin) + "ms");
    }
}
```

>  如何处理异步未确认消息  

把未确认的消息放到一个基于内存的能被发布线程访问的队列， 比如说用 ConcurrentLinkedQueue 这个队列在 confirm callbacks 与发布线程之间进行消息的传递。  

### 三种发布对比

> 单独发布消息 

同步等待确认，简单，但吞吐量非常有限

> 批量发布消息 

批量同步等待确认，简单，合理的吞吐量，一旦出现问题但很难推断出是那条 消息出现了问题

> 异步发布消息

最佳性能和资源使用，在出现错误的情况下可以很好地控制，但是实现起来稍微难些 

# 交换机

## 交换机

> RabbitMQ 消息传递模型的核心思想 

生产者生产的消息从不会直接发送到队列。实际上，通常生产者甚至都不知道这些消息传递传递到了哪些队列中。相反，**生产者只能将消息发送到交换机**(exchange)

> 交换机工作的内容

* 一方面它接收来自生产者的消息
* 另一方面将它们推入队列

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-25.png)

> 交换机类型

* 直接(direct)
* 主题(topic) 
* 标题(headers) 
* 扇出(fanout) 

> 无名交换机

```java
channel.basicPublish("", "hello", null, message.getBytes());
```

第一个参数代表交换机的名称，上述空字符串表示**默认或无名称交换机**

## 临时队列

> 创建临时队列的方式

```java
String queueName = channel.queueDeclare().getQueue(); 
```

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-28.png)

##  绑定

> binding

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-26.png)

## Fanout

> Fanout

它是将接收到的所有消息广播到它知道的所有队列中。 

> 官方案例

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-29.png)

Logs 和临时队列的绑定关系如下图 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-30.png)

**消费者**

ReceiveLogs01 将接收到的消息打印在控制台 

```java
public class ReceiveLogs01 {
    // 交换机名称
    private static final String EXCHANGE_NAME = "logs";

    public static void main(String[] argv) throws Exception {
        // 创建信道
        Channel channel = RabbitMQUtils.getChannel();

        // 声明交换机
        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");

        /**
         * 生成一个临时的队列 队列的名称是随机的
         * 当消费者断开和该队列的连接时 队列自动删除
         */
        String queueName = channel.queueDeclare().getQueue();

        // 把该临时队列绑定exchange 其中 routingkey(也称之为 binding key)为空字符串
        channel.queueBind(queueName, EXCHANGE_NAME, "");

        System.out.println("等待接收消息,把接收到的消息打印在屏幕.....");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println("控制台打印接收到的消息" + message);
        };

        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> {
        });
    }
}
```

ReceiveLogs02 将接收到的消息存储在磁盘 

```java
public class ReceiveLogs02 {
    // 交换机名称
    private static final String EXCHANGE_NAME = "logs";

    public static void main(String[] argv) throws Exception {
        // 创建信道
        Channel channel = RabbitMQUtils.getChannel();

        // 声明交换机
        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");

        /**
         * 生成一个临时的队列 队列的名称是随机的
         * 当消费者断开和该队列的连接时 队列自动删除
         */
        String queueName = channel.queueDeclare().getQueue();

        // 把该临时队列绑定 exchange 其中 routingkey(也称之为 binding key)为空字符串
        channel.queueBind(queueName, EXCHANGE_NAME, "");

        System.out.println("等待接收消息,把接收到的消息写到文件.....");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            File file = new File("C:\\work\\rabbitmq_info.txt");
            FileUtils.writeStringToFile(file, message, "UTF-8");
            System.out.println("数据写入文件成功");
        };

        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> {
        });
    }
}
```

**生产者**

EmitLog 发送消息给两个消费者接收 

```java
public class EmitLog {
    
    // 交换机名称
    private static final String EXCHANGE_NAME = "logs";

    public static void main(String[] argv) throws Exception {
        
        try (Channel channel = RabbitMQUtils.getChannel()) {
            /**
             * 声明一个 exchange
             * 1.exchange 的名称
             * 2.exchange 的类型
             */
            channel.exchangeDeclare(EXCHANGE_NAME, "fanout");

            Scanner sc = new Scanner(System.in);

            System.out.println("请输入信息");

            while (sc.hasNext()) {
                String message = sc.nextLine();
                channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes("UTF-8"));
                System.out.println("生产者发出消息" + message);
            }
        }
    }
}
```

## Direct

> Direct exchange  

direct类型的工作方式是**消息只去到它绑定的 routingKey 队列中** 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-31.png)

在这种绑定情况下，生产者发布消息到 exchange 上，绑定键为 orange 的消息会被发布到队列 Q1。绑定键为 black和green 和的消息会被发布到队列 Q2，其他消息类型的消息将被丢弃。 

> 多重绑定  

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-32.png)

当然如果 exchange 的绑定类型是 direct，但是它绑定的多个队列的 key **如果都相同**，在这种情 况下虽然绑定类型是 direct 但是它表现的就和 fanout 有点类似了，就跟广播差不多 

> 官方案例

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-33.png)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-34.png)

```java
public class ReceiveLogsDirect01 {
    // 交换机名称
    private static final String EXCHANGE_NAME = "direct_logs";

    public static void main(String[] argv) throws Exception {
        // 创建信道
        Channel channel = RabbitMQUtils.getChannel();

        // 声明交换机
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);

        String queueName = "disk";
        channel.queueDeclare(queueName, false, false, false, null);

        channel.queueBind(queueName, EXCHANGE_NAME, "error");

        System.out.println("等待接收消息.....");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            message = "接收绑定键:" + delivery.getEnvelope().getRoutingKey() + ",消息:" + message;
            File file = new File("C:\\work\\rabbitmq_info.txt");
            FileUtils.writeStringToFile(file, message, "UTF-8");
            System.out.println("错误日志已经接收");
        };
        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> {
        });
    }
}
```

```java
public class ReceiveLogsDirect02 {

    // 交换机名称
    private static final String EXCHANGE_NAME = "direct_logs";

    public static void main(String[] argv) throws Exception {
        // 创建你信道
        Channel channel = RabbitMQUtils.getChannel();

        // 交换机
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);

        String queueName = "console";

        channel.queueDeclare(queueName, false, false, false, null);
        channel.queueBind(queueName, EXCHANGE_NAME, "info");
        channel.queueBind(queueName, EXCHANGE_NAME, "warning");

        System.out.println("等待接收消息.....");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" 接收绑定键 :" + delivery.getEnvelope().getRoutingKey() + ", 消息:" + message);
        };
        
        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> {
        });
    }
}
```

```java
public class EmitLogDirect {

    // 交换机名称
    private static final String EXCHANGE_NAME = "direct_logs";

    public static void main(String[] argv) throws Exception {

        try (Channel channel = RabbitMQUtils.getChannel()) {

            // 声明交换机
            channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);

            // 创建多个 bindingKey
            Map<String, String> bindingKeyMap = new HashMap<>();

            bindingKeyMap.put("info", "普通 info 信息");
            bindingKeyMap.put("warning", "警告 warning 信息");
            bindingKeyMap.put("error", "错误 error 信息");
            // debug 没有消费这接收这个消息 所以丢失
            bindingKeyMap.put("debug", "调试 debug 信息");

            for (Map.Entry<String, String> bindingKeyEntry : bindingKeyMap.entrySet()) {
                String bindingKey = bindingKeyEntry.getKey();
                String message = bindingKeyEntry.getValue();
                channel.basicPublish(EXCHANGE_NAME, bindingKey, null,
                        message.getBytes("UTF-8"));
                System.out.println("生产者发出消息:" + message);
            }
        }
    }
}
```

##  Topics

> Topic要求

发送到类型是 topic 交换机的消息的 routing_key 不能随意写，必须满足一定的要求 

* 它必须是一个单词列表，以**点号分隔**开
* 单词可以是任意单词
* 单词列表最多不能超过 255 个字节 
* *(星号)可以代替**一个**单词
* #(井号)可以替代**零个或多个**单词   

> 官方案例

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-35.png)

**绑定关系**

* Q1 绑定的是中间带 orange ，共 3 个单词的字符串 

* Q2 绑定的是最后一个单词是 rabbit ，共 3 个单词 和  第一个单词是 lazy 的多个单词

**数据接收情况**

* quick.orange.rabbit 被队列 Q1、Q2 接收到 

* lazy.orange.elephant 被队列 Q1、Q2 接收到 

* quick.orange.fox 被队列 Q1 接收到 
* lazy.brown.fox 被队列 Q2 接收到
* lazy.pink.rabbit 虽然满足两个绑定但只被队列 Q2 **接收一次** 
* quick.brown.fox **不匹配任何绑定**不会被任何队列接收到**会被丢弃** 
* quick.orange.male.rabbit 是四个单词不匹配任何绑定会被丢弃
* lazy.orange.male.rabbit 是四个单词但匹配 Q2  

> <font color="red"><b>注意</b></font>

* 当一个队列绑定键是`#`,那么这个队列**将接收所有数据**，就有点像 fanout 
* 如果队列绑定键当中**没有**`#`和`*`出现，那么该队列绑定类型就是 direct 

> 官方案例

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-36.png)

```java
public class EmitLogTopic {

    // 队列名称
    private static final String EXCHANGE_NAME = "topic_logs";

    public static void main(String[] argv) throws Exception {

        try (Channel channel = RabbitMQUtils.getChannel()) {

            // 声明交换机
            channel.exchangeDeclare(EXCHANGE_NAME, "topic");

            /**
             * Q1-->绑定的是
             * 中间带 orange 带 3 个单词的字符串(*.orange.*)
             * Q2-->绑定的是
             * 最后一个单词是 rabbit 的 3 个单词(*.*.rabbit)
             * 第一个单词是 lazy 的多个单词(lazy.#)
             *
             */
            Map<String, String> bindingKeyMap = new HashMap<>();

            bindingKeyMap.put("quick.orange.rabbit", "被队列 Q1Q2 接收到");
            bindingKeyMap.put("lazy.orange.elephant", "被队列 Q1Q2 接收到");
            bindingKeyMap.put("quick.orange.fox", "被队列 Q1 接收到");
            bindingKeyMap.put("lazy.brown.fox", "被队列 Q2 接收到");
            bindingKeyMap.put("lazy.pink.rabbit", "虽然满足两个绑定但只被队列 Q2 接收一次");
            bindingKeyMap.put("quick.brown.fox", "不匹配任何绑定不会被任何队列接收到会被丢弃");
            bindingKeyMap.put("quick.orange.male.rabbit", "是四个单词不匹配任何绑定会被丢弃");
            bindingKeyMap.put("lazy.orange.male.rabbit", "是四个单词但匹配 Q2");

            for (Map.Entry<String, String> bindingKeyEntry : bindingKeyMap.entrySet()) {

                String bindingKey = bindingKeyEntry.getKey();
                String message = bindingKeyEntry.getValue();
                channel.basicPublish(EXCHANGE_NAME, bindingKey, null,
                        message.getBytes("UTF-8"));
                System.out.println("生产者发出消息" + message);
            }
        }
    }
}
```

```java
public class ReceiveLogsTopic01 {

    // 队列名称
    private static final String EXCHANGE_NAME = "topic_logs";

    public static void main(String[] argv) throws Exception {

        // 创建信道
        Channel channel = RabbitMQUtils.getChannel();

        // 声明交换机
        channel.exchangeDeclare(EXCHANGE_NAME, "topic");

        //声明 Q1 队列与绑定关系
        String queueName = "Q1";

        // 声明队列
        channel.queueDeclare(queueName, false, false, false, null);

        // 绑定队列
        channel.queueBind(queueName, EXCHANGE_NAME, "*.orange.*");

        System.out.println("等待接收消息.....");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" 接收队列 :" + queueName + " 绑定键:" + delivery.getEnvelope().getRoutingKey() + ", 消息:" + message);
        };
        
        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> {
        });
    }
}
```

```java
public class ReceiveLogsTopic02 {

    // 队列名称
    private static final String EXCHANGE_NAME = "topic_logs";

    public static void main(String[] argv) throws Exception {

        // 创建信道
        Channel channel = RabbitMQUtils.getChannel();

        // 声明交换机
        channel.exchangeDeclare(EXCHANGE_NAME, "topic");

        // 声明 Q2 队列与绑定关系
        String queueName = "Q2";

        // 声明队列
        channel.queueDeclare(queueName, false, false, false, null);

        // 绑定队列
        channel.queueBind(queueName, EXCHANGE_NAME, "*.*.rabbit");
        channel.queueBind(queueName, EXCHANGE_NAME, "lazy.#");

        System.out.println("等待接收消息.....");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" 接收队列 :" + queueName + " 绑定键:" + delivery.getEnvelope().getRoutingKey() + ",消息:" + message);
        };
        
        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> {
        });
    }
}
```
