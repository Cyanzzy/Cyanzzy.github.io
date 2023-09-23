---
title: 1 RabbitMQ-QuickStart
date: 2023-07-23 16:58:30
tags: 
  - MQ
categories: 
  - Technology
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



## Springboot 整合

> 确认机制

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-58.png)

> 配置文件

在配置文件当中需要添加

```properties
spring.rabbitmq.publisher-confirm-type=correlated 
```

* `NONE ` ：禁用发布确认模式，是默认值
* `CORRELATED`：发布消息成功到交换器后会触发回调方法
* `SIMPLE `： 
  1. 和 CORRELATED 值一样会触发回调方法
  2. 在发布消息成功后使用 rabbitTemplate 调用 waitForConfirms 或 waitForConfirmsOrDie 方法 等待 broker 节点返回发送结果，根据返回结果来判定下一步的逻辑  **要注意的点是 waitForConfirmsOrDie 方法如果返回 false 则会关闭 channel，则接下来无法发送消息到 broker**

```properties
spring.rabbitmq.host=xxxxx
spring.rabbitmq.port=5672
spring.rabbitmq.username=xxxx
spring.rabbitmq.password=xxx
spring.rabbitmq.publisher-confirm-type=correlated
```

> 配置类

```java
@Configuration
public class ConfirmConfig {
    public static final String CONFIRM_EXCHANGE_NAME = "confirm.exchange";
    public static final String CONFIRM_QUEUE_NAME = "confirm.queue";

    // 声明业务 Exchange
    @Bean("confirmExchange")
    public DirectExchange confirmExchange(){
    	return new DirectExchange(CONFIRM_EXCHANGE_NAME);
    }
    
    // 声明确认队列
    @Bean("confirmQueue")
    public Queue confirmQueue(){
    	return QueueBuilder.durable(CONFIRM_QUEUE_NAME).build();
    }
    
    // 声明确认队列绑定关系
    @Bean
    public Binding queueBinding(@Qualifier("confirmQueue") Queue queue,
    @Qualifier("confirmExchange") DirectExchange exchange){
    	return BindingBuilder.bind(queue).to(exchange).with("key1");
    }
}
```

> 消息生产者

```java
@RestController
@RequestMapping("/confirm")
@Slf4j
public class Producer {
public static final String CONFIRM_EXCHANGE_NAME = "confirm.exchange";
    @Autowired
    private RabbitTemplate rabbitTemplate;
    @Autowired
    private MyCallBack myCallBack;
    
    // 依赖注入 rabbitTemplate 之后再设置它的回调对象
    @PostConstruct
    public void init(){
    rabbitTemplate.setConfirmCallback(myCallBack);
    }
    
    @GetMapping("sendMessage/{message}")
    public void sendMessage(@PathVariable String message){
     // 指定消息 id 为 1
    CorrelationData correlationData1=new CorrelationData("1");
    String routingKey = "key1";
     	rabbitTemplate.convertAndSend(CONFIRM_EXCHANGE_NAME,routingKey,message+routingKey,correlationData1);
    CorrelationData correlationData2=new CorrelationData("2");
    routingKey="key2";
    rabbitTemplate.convertAndSend(CONFIRM_EXCHANGE_NAME,routingKey,message+routingKey,correlationData2);
    log.info("发送消息内容:{}",message);
    }
}
```

> 回调接口

```java
@Component
@Slf4j
public class MyCallBack implements RabbitTemplate.ConfirmCallback {
    /**
    * 交换机不管是否收到消息的一个回调方法
    * CorrelationData
    * 消息相关数据
    * ack
    * 交换机是否收到消息
    */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        String id=correlationData!=null?correlationData.getId():"";
        if(ack){
        	log.info("交换机已经收到 id 为:{}的消息",id);
        } else {
        	log.info("交换机还未收到 id 为:{}消息,由于原因:{}",id,cause);
        }
    }
}
```

> 消息消费者

```java
@Component
@Slf4j
public class ConfirmConsumer {
    public static final String CONFIRM_QUEUE_NAME = "confirm.queue";
    
    @RabbitListener(queues =CONFIRM_QUEUE_NAME)
    public void receiveMsg(Message message){
        String msg=new String(message.getBody());
        log.info("接受到队列 confirm.queue 消息:{}",msg);
    }
}
```

> 结果分析

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-59.png)

发送两条消息，第一条消息的 RoutingKey 为 "key1"，第二条消息的 RoutingKey 为 "key2"，两条消息都成功被交换机接收，也收到了交换机的确认回调，但消费者只收到了一条消息，因为 第二条消息的 RoutingKey 与队列的 BindingKey 不一致，也没有其它队列能接收这个消息，所有第二条 消息被直接丢弃了 

##  回退消息

>  Mandatory 参数  

在**仅开启生产者确认机制**的情况下，交换机接收到消息后，会直接给消息生产者发送确认消息，如果发现该消息**不可路由**，那么消息会被**直接丢弃**，此时生产者是不知道消息被丢弃的

但通过设置 mandatory 参数可以在**当消息传递过程中不可达目的地时将消息返回给生产者**。 

> 消息生产者

```java
@Slf4j
@Component
public class MessageProducer implements RabbitTemplate.ConfirmCallback ,
RabbitTemplate.ReturnCallback {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
	// rabbitTemplate 注入之后就设置该值
    @PostConstruct
    private void init() {
        rabbitTemplate.setConfirmCallback(this);
        /**
        * true：
        * 交换机无法将消息进行路由时，会将该消息返回给生产者
        * false：
        * 如果发现消息无法进行路由，则直接丢弃
        */
        rabbitTemplate.setMandatory(true);
        //设置回退消息交给谁处理
        rabbitTemplate.setReturnCallback(this);
    }
    
    @GetMapping("sendMessage")
    public void sendMessage(String message){
        //让消息绑定一个 id 值
        CorrelationData correlationData1 = new CorrelationData(UUID.randomUUID().toString());
        rabbitTemplate.convertAndSend("confirm.exchange","key1",message+"key1",correlationData1)
        ;
        log.info("发送消息 id 为:{}内容为{}",correlationData1.getId(),message+"key1");
        CorrelationData correlationData2 = new CorrelationData(UUID.randomUUID().toString());

        rabbitTemplate.convertAndSend("confirm.exchange","key2",message+"key2",correlationData2);
        log.info("发送消息 id 为:{}内容为{}",correlationData2.getId(),message+"key2");
    }
    
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
    	String id = correlationData != null ? correlationData.getId() : "";
        if (ack) {
        	log.info("交换机收到消息确认成功, id:{}", id);
        } else {
        	log.error("消息 id:{}未成功投递到交换机,原因是:{}", id, cause);
        }
    }
    
    @Override
    public void returnedMessage(Message message, int replyCode, String replyText, String
    exchange, String routingKey) {
        log.info("消息:{}被服务器退回，退回原因:{}, 交换机是:{}, 路由 key:{}",
        new String(message.getBody()),replyText, exchange, routingKey);
    }
}
```

> 回调接口

```java
@Component
@Slf4j
public class MyCallBack implements
RabbitTemplate.ConfirmCallback,RabbitTemplate.ReturnCallback {
    /**
    * 交换机不管是否收到消息的一个回调方法
    * CorrelationData
    * 消息相关数据
    * ack
    * 交换机是否收到消息
    */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        String id=correlationData!=null?correlationData.getId():"";
        if(ack){
        	log.info("交换机已经收到 id 为:{}的消息",id);
        }else{
        	log.info("交换机还未收到 id 为:{}消息,由于原因:{}",id,cause);
        }
    }
    
	// 当消息无法路由的时候的回调方法
    @Override
    public void returnedMessage(Message message, int replyCode, String replyText, String
    exchange, String routingKey) {
    	log.error(" 消 息 {}, 被交换机 {} 退回，退回原因 :{}, 路 由 key:{}",new
    	String(message.getBody()),exchange,replyText,routingKey);
    }
}
```

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-60.png)

##  备份交换机

在RabbitMQ中，我们并不知道该如何处理这些无法路由的消息，最多打个日志，然后触发报警，再来手动处理。而通过日志来处理这些无法路由的消息很不优雅，特别是所在的服务器有多台机器的时候。所以这里就可以使用**备份交换机**来把这些**无法路由的消息全部放到备份交换机的备份队列里面**。 

> 架构

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-61.png)

> 配置类

```java
@Configuration
public class ConfirmConfig {
    public static final String CONFIRM_EXCHANGE_NAME = "confirm.exchange";
    public static final String CONFIRM_QUEUE_NAME = "confirm.queue";
    public static final String BACKUP_EXCHANGE_NAME = "backup.exchange";
    public static final String BACKUP_QUEUE_NAME = "backup.queue";
    public static final String WARNING_QUEUE_NAME = "warning.queue";
    
    // 声明确认队列
    @Bean("confirmQueue")
    public Queue confirmQueue(){
    	return QueueBuilder.durable(CONFIRM_QUEUE_NAME).build();
    }
    
    // 声明确认队列绑定关系
    @Bean
    public Binding queueBinding(@Qualifier("confirmQueue") Queue queue,
    @Qualifier("confirmExchange") DirectExchange exchange){
    	return BindingBuilder.bind(queue).to(exchange).with("key1");
    }
    
    // 声明备份 Exchange
    @Bean("backupExchange")
    public FanoutExchange backupExchange(){
    	return new FanoutExchange(BACKUP_EXCHANGE_NAME);
    }
    // 声明确认 Exchange 交换机的备份交换机
    @Bean("confirmExchange")
    public DirectExchange confirmExchange(){
        ExchangeBuilder exchangeBuilder =
        ExchangeBuilder.directExchange(CONFIRM_EXCHANGE_NAME)
        .durable(true)
        // 设置该交换机的备份交换机
        .withArgument("alternate-exchange", BACKUP_EXCHANGE_NAME);
        return (DirectExchange)exchangeBuilder.build();
    }
    
    // 声明警告队列
    @Bean("warningQueue")
    public Queue warningQueue(){
    	return QueueBuilder.durable(WARNING_QUEUE_NAME).build();
    }
    // 声明报警队列绑定关系
    @Bean
    public Binding warningBinding(@Qualifier("warningQueue") Queue queue,
    @Qualifier("backupExchange") FanoutExchange
    backupExchange){
    	return BindingBuilder.bind(queue).to(backupExchange);
    }
    
    // 声明备份队列
    @Bean("backQueue")
    public Queue backQueue(){
    	return QueueBuilder.durable(BACKUP_QUEUE_NAME).build();
    }
    
    // 声明备份队列绑定关系
    @Bean
    public Binding backupBinding(@Qualifier("backQueue") Queue queue,
    @Qualifier("backupExchange") FanoutExchange backupExchange){
    	return BindingBuilder.bind(queue).to(backupExchange);
    }
}
```

> 报警消费者 用独立的消费者来进行监测和报警。  

```java
@Component
@Slf4j
public class WarningConsumer {
    public static final String WARNING_QUEUE_NAME = "warning.queue";
    
    @RabbitListener(queues = WARNING_QUEUE_NAME)
    public void receiveWarningMsg(Message message) {
        String msg = new String(message.getBody());
        log.error("报警发现不可路由消息：{}", msg);
    }
}
```

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-62.png)



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

#  死信队列

##  死信概念

死信是无法被消费的消息，一般来说producer 将消息投递到 queue 里，consumer 从 queue 取出消息进行消费，但某些时候由于特定的原因导致 queue 中的某些消息无法被消费，这样的消息没有后续的处理就成为死信。

应用：为了防止订单业务的消息数据丢失，需要使用 RabbitMQ 的死信队列机制，当消息消费发生异常时，将消息投入死信队列中

##  死信来源

* 消息 TTL 过期 

* 队列达到最大长度（队列满，无法再添加数据到 mq 中）

* 消息被拒绝（basic.reject 或 basic.nack）并且 requeue=false. 

##  案例演示

> 架构

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-38.png)

### 消息 TTL 过期

> **生产者** 

```java
public class Producer {

    // 交换机名称
    private static final String NORMAL_EXCHANGE = "normal_exchange";

    public static void main(String[] argv) throws Exception {

        try (Channel channel = RabbitMQUtils.getChannel()) {
            // 声明交换机
            channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);

            // 设置消息的 TTL 时间
            AMQP.BasicProperties properties = new AMQP.BasicProperties()
                    .builder()
                    .expiration("10000")
                    .build();

            // 演示队列个数限制
            for (int i = 1; i < 11; i++) {
                String message = "info" + i;
                channel.basicPublish(NORMAL_EXCHANGE, "zhangsan", properties, message.getBytes());
                System.out.println("生产者发送消息:" + message);
            }
        }
    }
}
```

> **消费者 C1** (启动之后关闭该消费者 模拟其**接收不到消息**) 

```java
public class Consumer01 {
    // 普通交换机名称
    private static final String NORMAL_EXCHANGE = "normal_exchange";

    // 死信交换机名称
    private static final String DEAD_EXCHANGE = "dead_exchange";

    public static void main(String[] argv) throws Exception {
        Channel channel = RabbitMQUtils.getChannel();

        // 声明死信和普通交换机 类型为 direct
        channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
        channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);

        // 声明死信队列
        String deadQueue = "dead-queue";
        channel.queueDeclare(deadQueue, false, false, false, null);

        // 死信队列绑定死信交换机与 routing key
        channel.queueBind(deadQueue, DEAD_EXCHANGE, "lisi");

        // 正常队列绑定死信队列信息
        Map<String, Object> params = new HashMap<>();
        // 正常队列设置死信交换机 参数 key 是固定值
        params.put("x-dead-letter-exchange", DEAD_EXCHANGE);
        // 正常队列设置死信 routing-key 参数 key 是固定值
        params.put("x-dead-letter-routing-key", "lisi");

        // 声明正常队列
        String normalQueue = "normal-queue";
        channel.queueDeclare(normalQueue, false, false, false, params);
        channel.queueBind(normalQueue, NORMAL_EXCHANGE, "zhangsan");

        System.out.println("等待接收消息.....");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println("Consumer01 接收到消息" + message);
        };

        channel.basicConsume(normalQueue, true, deliverCallback, consumerTag -> {
        });
    }
}
```

**生产者未发送消息**

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-39.png)

**生产者发送10条消息（此时正常消息队列有10条未消费消息）**

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-40.png)

**时间过去10秒（正常队列里的消息由于没有消费，消息进入死信队列）**

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-41.png)

> **消费者 C2**  (以上步骤完成后 启动 C2 消费者 它消费死信队列里面的消息) 

```java
public class Consumer02 {

    // 交换机名称
    private static final String DEAD_EXCHANGE = "dead_exchange";

    public static void main(String[] argv) throws Exception {

        // 创建信道
        Channel channel = RabbitMQUtils.getChannel();

        // 声明交换机
        channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);

        // 声明队列
        String deadQueue = "dead-queue";
        channel.queueDeclare(deadQueue, false, false, false, null);
        // 绑定队列
        channel.queueBind(deadQueue, DEAD_EXCHANGE, "lisi");

        System.out.println("等待接收死信队列消息.....");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println("Consumer02 接收死信队列的消息" + message);
        };

        channel.basicConsume(deadQueue, true, deliverCallback, consumerTag -> {
        });
    }
}
```

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-42.png)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-43.png)

### 队列达到最大长度

> **消息生产者代码去掉 TTL 属性**

```java
public class Producer {

    // 交换机名称
    private static final String NORMAL_EXCHANGE = "normal_exchange";

    public static void main(String[] argv) throws Exception {

        try (Channel channel = RabbitMQUtils.getChannel()) {

            // 声明交换机
            channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);

            // 该信息是用作演示队列个数限制
            for (int i = 1; i < 11; i++) {
                String message = "info" + i;
                channel.basicPublish(NORMAL_EXCHANGE, "zhangsan", null, message.getBytes());
                System.out.println("生产者发送消息: " + message);
            }
        }
    }
}
```

> **C1 消费者修改以下代码**(启动之后关闭该消费者 模拟其**接收不到消息**) 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-44.png)

<font color="red">**注意此时需要把原先队列删除 因为参数改变** </font>

> **C2 消费者代码不变**(启动 C2 消费者)  

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-45.png)

### 消息被拒

> **消息生产者**代码同上生产者一致 

> **C1 消费者代码**(启动之后关闭该消费者 模拟其接收不到消息)  

```java
public class Consumer01 {

    // 普通交换机名称
    private static final String NORMAL_EXCHANGE = "normal_exchange";

    // 死信交换机名称
    private static final String DEAD_EXCHANGE = "dead_exchange";

    public static void main(String[] argv) throws Exception {

        // 创建信道
        Channel channel = RabbitMQUtils.getChannel();

        // 声明死信和普通交换机 类型为 direct
        channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
        channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);

        // 声明死信队列
        String deadQueue = "dead-queue";
        channel.queueDeclare(deadQueue, false, false, false, null);

        // 死信队列绑定死信交换机与 routingkey
        channel.queueBind(deadQueue, DEAD_EXCHANGE, "lisi");

        // 正常队列绑定死信队列信息
        Map<String, Object> params = new HashMap<>();
        // 正常队列设置死信交换机 参数 key 是固定值
        params.put("x-dead-letter-exchange", DEAD_EXCHANGE);
        // 正常队列设置死信 routing-key 参数 key 是固定值
        params.put("x-dead-letter-routing-key", "lisi");

        // 声明正常队列
        String normalQueue = "normal-queue";
        channel.queueDeclare(normalQueue, false, false, false, params);
        channel.queueBind(normalQueue, NORMAL_EXCHANGE, "zhangsan");

        System.out.println("等待接收消息.....");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            if (message.equals("info5")) {
                System.out.println("Consumer01 接收到消息" + message + "并拒绝签收该消息");
                // requeue 设置为 false 代表拒绝重新入队 该队列如果配置了死信交换机将发送到死信队列中
                channel.basicReject(delivery.getEnvelope().getDeliveryTag(), false);
            } else {
                System.out.println("Consumer01 接收到消息" + message);
                channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
            }
        };
        boolean autoAck = false;
        channel.basicConsume(normalQueue, autoAck, deliverCallback, consumerTag -> {
        });
    }
}
```

**生产者发送消息后**

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-46.png)

> C2 消费者代码不变 

**启动消费者 1 然后再启动消费者 2**  

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-47.png)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-48.png)







#   延迟队列

## 延迟队列介绍

> 概念

延时队列就是用来存放需要**在指定时间被处理**的元素的队列

> 使用场景

* 订单在十分钟之内未支付则自动取消
* 新创建的店铺，如果在十天内都没有上传过商品，则自动发送消息提醒
* 用户注册成功后，如果三天内没有登陆则进行短信提醒
* 用户发起退款，如果三天内没有得到处理则通知相关运营人员 
* 预定会议后，需要在预定的时间点前十分钟通知各个与会人员参加会议 

> 使用场景特点

需要在某个事件发生之后或者之前的**指定时间点完成某一项任务** 

> 使用原因

对于**数据量比较大，并且时效性较强的场景**

如：“订单十 分钟内未支付则关闭“，短期内未支付的订单数据可能会有很多，活动期间甚至会达到百万甚至千万 级别，对这么庞大的数据量**仍旧使用轮询的方式是不可取的**，很可能在一秒内无法完成所有订单的检查，同时会给数据库带来很大压力，无法满足业务要求而且性能低下

> TTL概念

* TTL 是 RabbitMQ 中一个消息或者队列的属性，表明一条消息或者该队列中的所有消息的**最大存活时间** ，TTL单位是毫秒

* 如果一条消息设置了 TTL 属性或者进入了设置 TTL 属性的队列，那么这条消息如果在 TTL 设置的时间内没有被消费，则会成为"死信"
* 如果同时配置了队列的 TTL 和消息的 TTL，那么较小的那个值将会被使用 

> 消息设置TTL

 针对每条消息设置 TTL 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-49.png)

> 队列设置TTL

 创建队列的时候设置队列的“x-message-ttl”属性 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-50.png)

> **注意**

* 消息设置TTL： 消息即使过期，不一定会被马上丢弃，因为消息是否过期是在即将投递到消费者 之前判定的，如果当前队列有严重的消息积压情况，则已过期的消息也许还能存活较长时间
* 队列设置TTL：旦消息过期，就会被队列丢弃(如果配置死信队列被丢到死信队列中) 
* 如果不设置 TTL，表示消息永远不会过期
* 如果将 TTL 设置为 0，则表示除非此时可以直接投递该消息到消费者，否则该消息将会被丢弃 

## 案例演示

> 引入依赖

```xml
<!--RabbitMQ 依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>

 <!--RabbitMQ 测试依赖-->
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit-test</artifactId>
    <scope>test</scope>
</dependency>
```

```xml
<!--swagger-->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>

<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```

> 修改配置文件

```properties
spring.rabbitmq.host=xxxx
spring.rabbitmq.port=5672
spring.rabbitmq.username=xxxx
spring.rabbitmq.password=xxxx
```

> 添加Swagger配置类

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket webApiConfig(){
        return new Docket(DocumentationType.SWAGGER_2)
        .groupName("webApi")
        .apiInfo(webApiInfo())
        .select()
          .build();
    }
    private ApiInfo webApiInfo(){
        return new ApiInfoBuilder()
        .title("rabbitmq 接口文档")
        .description("本文档描述了 rabbitmq 微服务接口定义")
        .version("1.0")
        .contact(new Contact("enjoy6288", "http://atguigu.com",
        "1551388580@qq.com"))
        .build();
    }
}
```

> 架构图

创建队列 QA 和 QB，队列 TTL 分别设置为 10S 和 40S，然后创建一个交换机 X 和死信交换机 Y，类型都是 direct，创建死信队列 QD 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-51.png)

> 配置类

```java
@Configuration
public class TtlQueueConfig {
    // 交换机X
    public static final String X_EXCHANGE = "X";
    // 队列QA
    public static final String QUEUE_A = "QA";
    // 队列QB
    public static final String QUEUE_B = "QB";
    // 死信交换机Y
    public static final String Y_DEAD_LETTER_EXCHANGE = "Y";
    // 死信队列QD
    public static final String DEAD_LETTER_QUEUE = "QD";
    
	// 声明 xExchange
    @Bean("xExchange")
    public DirectExchange xExchange(){
    	return new DirectExchange(X_EXCHANGE);
    } 
     
    // 声明 yExchange
    @Bean("yExchange")
    public DirectExchange yExchange(){
    	return new DirectExchange(Y_DEAD_LETTER_EXCHANGE);
    }
    
    // 声明队列 QA TTL 为 10s 并绑定到对应的死信交换机Y
    @Bean("queueA")
    public Queue queueA(){
        Map<String, Object> args = new HashMap<>(3);
    	// 声明当前队列绑定的死信交换机
    	args.put("x-dead-letter-exchange", Y_DEAD_LETTER_EXCHANGE);
    	// 声明当前队列的死信路由 key
    	args.put("x-dead-letter-routing-key", "YD");
    	// 声明队列的 TTL
    	args.put("x-message-ttl", 10000);
    	return QueueBuilder.durable(QUEUE_A).withArguments(args).build();
    }
    
    // 声明队列 A 绑定 X 交换机
    @Bean
    public Binding queueaBindingX(@Qualifier("queueA") Queue queueA,
        @Qualifier("xExchange") DirectExchange xExchange){
        return BindingBuilder.bind(queueA).to(xExchange).with("XA");
    }
    
    // 声明队列 QB TTL 为 40s 并绑定到对应的死信交换机
    @Bean("queueB")
    public Queue queueB(){
        Map<String, Object> args = new HashMap<>(3);
        // 声明当前队列绑定的死信交换机
        args.put("x-dead-letter-exchange", Y_DEAD_LETTER_EXCHANGE);
        // 声明当前队列的死信路由 key
        args.put("x-dead-letter-routing-key", "YD");
        // 声明队列的 TTL
        args.put("x-message-ttl", 40000);
        return QueueBuilder.durable(QUEUE_B).withArguments(args).build();
    }
    
    // 声明队列 QB 绑定 X 交换机
    @Bean
    public Binding queuebBindingX(@Qualifier("queueB") Queue queue1B,
    @Qualifier("xExchange") DirectExchange xExchange){
    	return BindingBuilder.bind(queue1B).to(xExchange).with("XB");
    }
    
    // 声明死信队列 QD
    @Bean("queueD")
    public Queue queueD(){
    	return new Queue(DEAD_LETTER_QUEUE);
    }
    
    // 声明死信队列 QD 绑定关系
    @Bean
    public Binding deadLetterBindingQAD(@Qualifier("queueD") Queue queueD,
    @Qualifier("yExchange") DirectExchange yExchange){
    	return BindingBuilder.bind(queueD).to(yExchange).with("YD");
    }
}

```

> 消息生产者

```java
@Slf4j
@RequestMapping("ttl")
@RestController
public class SendMsgController {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    @GetMapping("sendMsg/{message}")
    public void sendMsg(@PathVariable String message) {
        log.info("当前时间：{},发送一条信息给两个 TTL 队列:{}", new Date(), message);
        rabbitTemplate.convertAndSend("X", "XA", "消息来自 ttl 为 10S 的队列: " + message);
        rabbitTemplate.convertAndSend("X", "XB", "消息来自 ttl 为 40S 的队列: " + message);
    }
}
```

> 消息消费者

```java
@Slf4j
@Component
public class DeadLetterQueueConsumer {
    @RabbitListener(queues = "QD")
    public void receiveD(Message message, Channel channel) throws IOException {
        String msg = new String(message.getBody());
        log.info("当前时间：{},收到死信队列信息{}", new Date().toString(), msg);
    }
}
```

 发起一个请求 http://localhost:8080/ttl/sendMsg/嘻嘻嘻 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-52.png)

第一条消息在 10S 后变成了死信消息，然后被消费者消费掉

第二条消息在 40S 之后变成了死信消息， 然后被消费掉 

##  	延时队列优化

> 架构图

新增了一个队列 QC，该队列不设置 TTL 时间 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-53.png)

> 配置类

```java
@Component
public class MsgTtlQueueConfig {
    // 死信交换机Y
    public static final String Y_DEAD_LETTER_EXCHANGE = "Y";
    // 队列C
    public static final String QUEUE_C = "QC";
    
    // 声明队列 C 死信交换机
    @Bean("queueC")
    public Queue queueB(){
        Map<String, Object> args = new HashMap<>(3);
        // 声明当前队列绑定的死信交换机
        args.put("x-dead-letter-exchange", Y_DEAD_LETTER_EXCHANGE);
        // 声明当前队列的死信路由 key
        args.put("x-dead-letter-routing-key", "YD");
        // 没有声明 TTL 属性
        return QueueBuilder.durable(QUEUE_C).withArguments(args).build();
    }
    
    // 声明队列 B 绑定 X 交换机
    @Bean
    public Binding queuecBindingX(@Qualifier("queueC") Queue queueC,
    @Qualifier("xExchange") DirectExchange xExchange){
    	return BindingBuilder.bind(queueC).to(xExchange).with("XC");
    }
}
```

> 消息生产者

```java
@GetMapping("sendExpirationMsg/{message}/{ttlTime}")
public void sendMsg(@PathVariable String message,@PathVariable String ttlTime) {
    rabbitTemplate.convertAndSend("X", "XC", message, correlationData ->{
        correlationData.getMessageProperties().setExpiration(ttlTime);
        return correlationData;
    });
    log.info("当前时间：{},发送一条时长{}毫秒 TTL 信息给队列 C:{}", new Date(),ttlTime, message);
}
```

 发起请求

http://localhost:8080/ttl/sendExpirationMsg/你好 1/20000 

http://localhost:8080/ttl/sendExpirationMsg/你好 2/2000 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-54.png)

如果使用在消息属性上设置 TTL 的方式，消息可能并不会按时“死亡“，因为 RabbitMQ 只会检查第一个消息是否过期，如果过期则丢到死信队列， 如果第一个消息的延时时长很长，而第二个消息的延时时长很短，第二个消息并不会优先得到执行。 

##  Rabbitmq 插件实现延迟队列

[rabbitmq_delayed_message_exchange 插件](https://www.rabbitmq.com/community-plugins.html)

> 安装步骤

1. 下载插件， 放置到 RabbitMQ 的插件目录

2. 进入 RabbitMQ 的安装目录下的 plgins 目录，执行下面命令让该插件生效，然后重启 RabbitMQ 

   ```bash
   /usr/lib/rabbitmq/lib/rabbitmq_server-3.8.8/plugins
   rabbitmq-plugins enable rabbitmq_delayed_message_exchange
   ```

   ![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-55.png)

> 架构图

新增队列 delayed.queue，一个自定义交换机 delayed.exchange 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-56.png)

> 配置类

在自定义的交换机中，这是一种新的交换类型，该类型消息**支持延迟投递机制**，消息传递后并不会立即投递到目标队列中，而是存储在 mnesia(一个分布式数据系统)表中，当达到投递时间时，才投递到目标队列中 

```java
@Configuration
public class DelayedQueueConfig {
    public static final String DELAYED_QUEUE_NAME = "delayed.queue";
    public static final String DELAYED_EXCHANGE_NAME = "delayed.exchange";
    public static final String DELAYED_ROUTING_KEY = "delayed.routingkey";
    
    @Bean
    public Queue delayedQueue() {
    	return new Queue(DELAYED_QUEUE_NAME);
    }
    
    // 自定义交换机 我们在这里定义的是一个延迟交换机
    @Bean
    public CustomExchange delayedExchange() {
        Map<String, Object> args = new HashMap<>();
        // 自定义交换机的类型
        args.put("x-delayed-type", "direct");
        return new CustomExchange(DELAYED_EXCHANGE_NAME, "x-delayed-message", true, false,
        args);
    }
    
    @Bean
    public Binding bindingDelayedQueue(@Qualifier("delayedQueue") Queue queue,
    @Qualifier("delayedExchange") CustomExchange
    delayedExchange) {
    	return BindingBuilder.bind(queue).to(delayedExchange).with(DELAYED_ROUTING_KEY).noargs();
    }
}
```

> 消息生产者

```java
public static final String DELAYED_EXCHANGE_NAME = "delayed.exchange";
public static final String DELAYED_ROUTING_KEY = "delayed.routingkey";

@GetMapping("sendDelayMsg/{message}/{delayTime}")
public void sendMsg(@PathVariable String message,@PathVariable Integer delayTime) {
    rabbitTemplate.convertAndSend(DELAYED_EXCHANGE_NAME, DELAYED_ROUTING_KEY, message,
    correlationData ->{
        correlationData.getMessageProperties().setDelay(delayTime);
        return correlationData;
     });
    log.info(" 当 前 时 间 ： {}, 发送一条延迟 {} 毫秒的信息给队列 delayed.queue:{}", new
    Date(),delayTime, message);
}

```

> 消息消费者

```java
public static final String DELAYED_QUEUE_NAME = "delayed.queue";

@RabbitListener(queues = DELAYED_QUEUE_NAME)
public void receiveDelayedQueue(Message message){
    String msg = new String(message.getBody());
    log.info("当前时间：{},收到延时队列的消息：{}", new Date().toString(), msg);
}
```

发起请求： 

http://localhost:8080/ttl/sendDelayMsg/come on baby1/20000 

http://localhost:8080/ttl/sendDelayMsg/come on baby2/2000 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-57.png)

第二个消息被先消费掉  

##  总结

*  延时队列在需要延时处理的场景下非常有用，使用 RabbitMQ 来实现延时队列可以很好的利用 RabbitMQ 的特性
*  如：消息可靠发送、消息可靠投递、死信队列来保障消息至少被消费一次以及未被正 确处理的消息不会被丢弃
*  通过 RabbitMQ 集群的特性，可以很好的解决单点故障问题，不会因为 单个节点挂掉导致延时队列不可用或者消息丢失。 

# 幂等性 

> 概念

用户对于同一操作发起的一次请求或者多次请求的结果是一致的，不会因为多次点击而产生了副作用。 

用户购买商品后支付，支付扣款成功，但是返回结果的时候网络异常， 此时钱已经扣了，用户再次点击按钮，此时会进行第二次扣款，  返回结果成功，用户查询余额发现多扣钱 了，流水记录也变成了两条 

> 消息重复消费

消费者在消费 MQ 中的消息时，MQ 已把消息发送给消费者，消费者在给 MQ 返回 ack 时**网络中断**， 故 **MQ 未收到确认信息**，该条消息**会重新发给其他的消费者**，或者在网络重连后再次发送给该消费者，但实际上该消费者已成功消费了该条消息，造成消费者消费了重复的消息。 

> 解决方案

MQ 消费者的幂等性的解决**一般使用全局 ID 或者写个唯一标识**

比如时间戳或者 UUID 或者订单消费者消费 MQ 中的消息也可利用 MQ 的该 id 来判断，或者可按自己的规则生成一个全局唯一 id，每次消费消息时用该 id 先判断该消息是否已消费过。 

>  消费端的幂等性保障  

在海量订单生成的业务高峰期，生产端有可能就会重复发生了消息，这时候消费端就要实现幂等性， 这就意味着我们的**消息永远不会被消费多次**，即使我们收到了一样的消息。 

业界主流的幂等性有两种操作

* a. 唯一 ID+指纹码机制，利用数据库主键去重
  * 指纹码: 一些规则或者时间戳加别的服务给到的**唯一信息码**，它并不一定是我们系统生成的，基本都是由我们的业务规则拼接而来，但是一定要保证唯一性
  * 然后就利用查询语句进行判断这个 id 是否存在数据库中。优势就是实现简单就一个拼接，然后查询判断是否重复
  * 劣势就是在高并发时，如果是单个数 据库就会有写入性能瓶颈当然也可以采用分库分表提升性能  
* b. 利用 redis 的原子性去实现  
  * 利用 redis 执行 `setnx` 命令，天然具有幂等性。从而实现不重复消费 

# 优先级队列

> 添加方式
>
> 队列需要设置为优先级队列，消息需要设置消息的优先级，消费者需要等待消息已经发送到队列中才去消费，这样才有机会对消息进行排序 

* 控制台页面添加 

  ![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-63.png)

  

* 队列中代码添加优先级 

  ```java
  Map<String, Object> params = new HashMap();
  params.put("x-max-priority", 10);
  channel.queueDeclare("hello", true, false, false, params);
  ```

  ![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-64.png)

* 消息中代码添加优先级 

  ```java
  AMQP.BasicProperties properties = new
  AMQP.BasicProperties().builder().priority(5).build()
  ```

> 实现

**消息生产者**

```java
public class Producer {
	private static final String QUEUE_NAME="hello";
    public static void main(String[] args) throws Exception {
        try (Channel channel = RabbitMqUtils.getChannel();) {
        // 给消息赋予一个 priority 属性
        AMQP.BasicProperties properties = new
        AMQP.BasicProperties().builder().priority(5).build();
        for (int i = 1; i <11; i++) {
            String message = "info"+i;
            if(i==5){
            	channel.basicPublish("", QUEUE_NAME, properties, message.getBytes());
            }else{
            	channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
            }
            System.out.println("发送消息完成:" + message);
            }
        }
    }
}
```

**消息消费者**

```java
public class Consumer {
    private static final String QUEUE_NAME="hello";
   
    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        // 设置队列的最大优先级 最大可以设置到 255 官网推荐 1-10 如果设置太高比较吃内存和 CPU
        Map<String, Object> params = new HashMap();
        params.put("x-max-priority", 10);
        channel.queueDeclare(QUEUE_NAME, true, false, false, params);
        System.out.println("消费者启动等待消费......");
        DeliverCallback deliverCallback=(consumerTag, delivery)->{
            String receivedMessage = new String(delivery.getBody());
            System.out.println("接收到消息:"+receivedMessage);
        };
        channel.basicConsume(QUEUE_NAME,true,deliverCallback,(consumerTag)->{
        	System.out.println("消费者无法消费消息时调用，如队列被删除");
        });
    }
}
```

# 惰性队列

惰性队列会尽可能的将消息存入磁盘中，而在消费者消费到相应的消息时才会被加载到内存中，它的一个重要的设计目标**是支持更多的消息存储**。当消费者由于各种各样的原因(比如消费者下线、宕机亦或者是由于维护而关闭等)而致使**长时间内不能消费消息造成堆积**时，惰性队列就很有必要 

> 惰性队列两种模式： default 和 lazy 

*  默认的为 default 模式，在 3.6.0 之前的版本无需做任何变更 

*  lazy 模式即为惰性队列的模式
   * 可以通过调用 channel.queueDeclare 方法的时候在参数中设置，也可以通过 Policy 的方式设置
*  如果一个队列同时使用这两种方式设置的话，那么 Policy 的方式具备更高的优先级。 如果要通过声明的方式改变已有队列的模式的话，那么只能先删除队列，然后再重新声明一个新的。  

在队列声明的时候可以通过`x-queue-mode`参数来设置队列的模式，取值为“default”和“lazy” 

```java
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-queue-mode", "lazy");
channel.queueDeclare("myqueue", false, false, false, args);
```

> 内存开销对比  

 在发送 1 百万条消息，每条消息大概占 1KB 的情况下，普通队列占用内存是 1.2GB，而惰性队列仅仅 占用 1.5MB 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-65.png)



