---
title: 3 RabbitMQ-WorkQueues
date: 2023-07-23 18:16:07
tags: 
  - MQ
categories: 
  - Technology
swiper_index: 
---

在本篇中，我们将创建一个 *Work Queue*，用于在多个 *Worker* 之间分配耗时的任务。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-11.png)

工作队列背后的主要理念是避免立即执行资源密集型任务而不得不等待其完成。将任务安排在稍后完成。我们将 *task* 封装为消息，并将其发送到队列。在后台运行的 Worker 进程会弹出任务并最终执行作业。当运行多个 Worker 时，任务将在它们之间共享。

# 准备工作

将对上篇文章示例的`Send.java`稍作修改，以此允许使用命令行发送任意消息，该程序是将为工作队列安排任务，将其命名为`NewTask.java`

```java
String message = String.join(" ", argv);

channel.basicPublish("", "hello", null, message.getBytes());
System.out.println(" [x] Sent '" + message + "'");
```

对于上篇文章示例的`Recv.java`也稍做修改，我们令每个dot休眠一秒钟作为工作时间，它将处理已发送的消息并执行任务，将其命名为`Worker.java`

```java
DeliverCallback deliverCallback = (consumerTag, delivery) -> {
  String message = new String(delivery.getBody(), "UTF-8");

  System.out.println(" [x] Received '" + message + "'");
  try {
    doWork(message);
  } finally {
    System.out.println(" [x] Done");
  }
};
boolean autoAck = true; // acknowledgment is covered below
channel.basicConsume(TASK_QUEUE_NAME, autoAck, deliverCallback, consumerTag -> { });
```

模拟执行时间的假任务：

```java
private static void doWork(String task) throws InterruptedException {
    for (char ch: task.toCharArray()) {
        if (ch == '.') Thread.sleep(1000);
    }
}
```

Compile them as in tutorial one (with the jar files in the working directory and the environment variable CP):

```bash
javac -cp $CP NewTask.java Worker.java
```

# 轮询分发

使用任务队列的优势之一是可以轻松并行处理工作。如果积压大量工作，只需添加更多的工作进程，就能轻松实现扩展。

首先，让我们尝试同时运行两个 Worker 实例。它们都会从队列中获取信息，但具体是如何获取的呢？让我们来看看。

你需要打开三个控制台。其中两个将运行 Worker 程序。这两个控制台就是我们的两个消费者 - C1 和 C2。

```bash
# shell 1
java -cp $CP Worker
# => [*] Waiting for messages. To exit press CTRL+C
```

```bash
# shell 2
java -cp $CP Worker
# => [*] Waiting for messages. To exit press CTRL+C
```

在第三项中，我们将发布新任务。启动消费者后，您可以发布一些信息：

```bash
# shell 3
java -cp $CP NewTask First message.
# => [x] Sent 'First message.'
java -cp $CP NewTask Second message..
# => [x] Sent 'Second message..'
java -cp $CP NewTask Third message...
# => [x] Sent 'Third message...'
java -cp $CP NewTask Fourth message....
# => [x] Sent 'Fourth message....'
java -cp $CP NewTask Fifth message.....
# => [x] Sent 'Fifth message.....'
```

 Let's see what is delivered to our workers: 

```bash
java -cp $CP Worker
# => [*] Waiting for messages. To exit press CTRL+C
# => [x] Received 'First message.'
# => [x] Received 'Third message...'
# => [x] Received 'Fifth message.....'
```

```bash
java -cp $CP Worker
# => [*] Waiting for messages. To exit press CTRL+C
# => [x] Received 'Second message..'
# => [x] Received 'Fourth message....'
```

默认情况下，RabbitMQ 会将每条消息按顺序发送给下一个消费者。平均而言，每个消费者将收到相同数量的消息。这种分配消息的方式称为轮循。 

# 消息确认

> 执行一项任务可能需要几秒，如果消费者启动了一项较长的任务，但**在任务完成之前就终止了**，会发生什么情况？

在当前的代码中，一旦 RabbitMQ 将消息传递给消费者，它就会立即将其标记为删除。

**在这种情况下，如果您终止 Worker，它刚刚处理的消息就会丢失。同时，已分派给该特定 Worker 但尚未处理的消息也会丢失。**

但是我们不想有任何损失。如果一个worker宕机，我们将当前任务托付给另一个worker。为了确保消息永不丢失，RabbitMQ 支持消息确认。**消费者**会发送acknowledgement ，**告诉 RabbitMQ 已收到并处理了特定消息，RabbitMQ 可以删除该消息**。

如果消费者死亡（其通道关闭、连接关闭或 TCP 连接丢失）而未发送应答，RabbitMQ 将了解消息未被完全处理，**并将其重新排队**。如果有其他消费者同时在线，它将迅速将消息重新传递给另一个消费者。这样，您就可以确保不会丢失任何消息，即使worker偶尔死机。

消费者确认交付时会有一个超时（默认为 30 分钟）。这有助于检测出从不确认交付的错误（卡住）消费者。您可以按照交付确认超时中的说明增加超时时间。

Manual message acknowledgments 默认为打开。在之前的示例中，我们**通过 `autoAck=true` 标志显式地将其关闭**。现在将该标记设置为 false，并在完成任务后从 Worker 发送适当的确认信息。

```java
channel.basicQos(1); // accept only one unack-ed message at a time (see below)

DeliverCallback deliverCallback = (consumerTag, delivery) -> {
  String message = new String(delivery.getBody(), "UTF-8");

  System.out.println(" [x] Received '" + message + "'");
  try {
    doWork(message);
  } finally {
    System.out.println(" [x] Done");
    channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
  }
};
boolean autoAck = false;
channel.basicConsume(TASK_QUEUE_NAME, autoAck, deliverCallback, consumerTag -> { });
```

Using this code, you can ensure that even if you terminate a worker using CTRL+C while it was processing a message, **nothing is lost**. **Soon after the worker terminates, all unacknowledged messages are redelivered.**

**Acknowledgement must be sent on the same channelthat received the delivery**. Attempts to acknowledge using a different channel will result in a channel-level protocol exception. 

> 注意事项

It's a common mistake to miss the basicAck. It's an easy error, but the consequences are serious. Messages will be redelivered when your client quits (which may look like random redelivery), but RabbitMQ will eat more and more memory as it won't be able to release any unacked messages.

In order to debug this kind of mistake you can use `rabbitmqctl` to print the **messages_unacknowledged field**:

```bash
# linux
sudo rabbitmqctl list_queues name messages_ready messages_unacknowledged
# windows
rabbitmqctl.bat list_queues name messages_ready messages_unacknowledged
```

# 消息持久化

当 RabbitMQ 退出或崩溃时，它会忘记队列和消息，除非您告诉它不要这样做。要确保消息不会丢失，**需要做两件事：我们需要将队列和消息都标记为持久。**

> 队列持久化

首先，我们需要确保队列能在 RabbitMQ 节点重启后继续运行。为此，我们需要将其声明为 *durable*：

```java
boolean durable = true;
channel.queueDeclare("hello", durable, false, false, null);
```

Although this command is correct by itself, it won't work in our present setup. That's because we've already defined a queue called hello which is not durable. RabbitMQ doesn't allow you to redefine an existing queue with different parameters and will return an error to any program that tries to do that. But there is a quick workaround - let's declare a queue with different name, for example task_queue:

```java
boolean durable = true;
channel.queueDeclare("task_queue", durable, false, false, null);
```

> 消息持久化

对 queueDeclare 的更改需要同时应用于生产者和消费者代码。

至此，我们确信即使 RabbitMQ 重新启动，task_queue 队列也不会丢失。现在，我们需要将消息标记为持久消息，方法是将 **MessageProperties**（实现 BasicProperties）设置为 `PERSISTENT_TEXT_PLAIN`。

```java
import com.rabbitmq.client.MessageProperties;

channel.basicPublish("", "task_queue",
            MessageProperties.PERSISTENT_TEXT_PLAIN,
            message.getBytes());
```

> Note on message persistence

将消息标记为持久**并不能完全保证消息不会丢失**。虽然它会告诉 RabbitMQ 将消息保存到磁盘，**但当 RabbitMQ 接受了消息但尚未保存时，仍然会有一个很短的时间窗口**。

此外，RabbitMQ 不会对每条消息都执行 fsync(2) -- 消息可能只是保存到缓存中，而不是真正写入磁盘。

持久性保证并不强，但对于我们简单的任务队列来说绰绰有余。如果需要更强的保证，可以使用publisher confirms

# 公平分发

您可能已经注意到，调度工作仍然不能完全按照我们的要求进行。例如，**在有两个 Worker 的情况下，当所有奇数消息都很重，而偶数消息都很轻时，一个 Worker 将一直处于忙碌状态，而另一个 Worker 几乎不做任何工作**。但是，RabbitMQ 对此一无所知，它仍然会均匀地分派消息。

出现这种情况是因为 RabbitMQ 只是在消息进入队列时分派消息。它不会查看消费者未确认消息的数量。它只是盲目地将每 n 条消息分派给第 n 个消费者。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-23.png)

为了解决这个问题，可以设置如下的 basicQos 方法。这样，RabbitMQ 就不会一次向 Worker 发送多于一条消息。**在处理并确认前一条消息之前，不要向 Worker 发送新消息。相反，它会将消息分派给下一个不忙的 Worker。**

```java
int prefetchCount = 1;
channel.basicQos(prefetchCount);
```



> 本篇教程完整代码如下

```java
public class Worker {
    private static final String TASK_QUEUE_NAME = "task_queue";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        final Connection connection = factory.newConnection();
        final Channel channel = connection.createChannel();

        channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);
        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        channel.basicQos(1);

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");

            System.out.println(" [x] Received '" + message + "'");
            try {
                doWork(message);
            } finally {
                System.out.println(" [x] Done");
                channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
            }
        };
        channel.basicConsume(TASK_QUEUE_NAME, false, deliverCallback, consumerTag -> { });
    }

    private static void doWork(String task) {
        for (char ch : task.toCharArray()) {
            if (ch == '.') {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException _ignored) {
                    Thread.currentThread().interrupt();
                }
            }
        }
    }
}
```

```java
public class NewTask {

    private static final String TASK_QUEUE_NAME = "task_queue";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);

            String message = String.join(" ", argv);

            channel.basicPublish("", TASK_QUEUE_NAME,
                    MessageProperties.PERSISTENT_TEXT_PLAIN,
                    message.getBytes("UTF-8"));
            System.out.println(" [x] Sent '" + message + "'");
        }
    }
}
```

# Spring AMQP

我们将使用 Spring Boot 来引导和配置 Spring AMQP 项目，[代码仓库链接](https://gitee.com/cyanzzy/spring-amqp-learning)，与往常一样，配置类如下

```java
@Configuration
public class RabbitMQConfig {

    @Bean // 创建队列
    public Queue workQueue() { // 创建队列对象
        return new Queue("work-queue");
    }

}
```

消息发送方不变

```java
@Service
public class WorkQueuesSenderServiceImpl implements WorkQueuesSenderService {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Autowired
    private Queue queue; // 注入构造的队列

    @Override
    public void send(String message) { // 使用RabbitTemplate向队列发送消息
        rabbitTemplate.convertAndSend(queue.getName(), message);
        System.out.println(" ********************* [x] Sent '" + message + ", Routing key is " + queue.getName());
    }
}
```

当然也可以使用SpringBootTest测试

```java
@Component
public class WorkQueuesSender {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void send(String message) {
        rabbitTemplate.convertAndSend("work-queues", message);
    }
}
```

与之前不同的是，这里我们使用两个消费者接收消息，但最终只有一个能消费

```java
@Component
public class WorkQueuesReceiver1 {

    @RabbitListener(queues = "work-queue")
    public void receive(String in, Channel channel, Message message) throws IOException, InterruptedException {

        TimeUnit.SECONDS.sleep(1);


        int prefetchCount = 1;
        // 定义通道上允许的未确认消息的最大数量
        channel.basicQos(prefetchCount);

        System.out.println("WorkQueuesReceiver1: " + in);
        // 消息应答
        channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
    }
}
```

```java
@Component
public class WorkQueuesReceiver2 {

    @RabbitListener(queues = "work-queue")
    public void receive(String in, Channel channel, Message message) throws IOException, InterruptedException {

        TimeUnit.SECONDS.sleep(2);


        int prefetchCount = 1;
        // 定义通道上允许的未确认消息的最大数量
        channel.basicQos(prefetchCount);

        System.out.println("WorkQueuesReceiver2: " + in);
        // 消息应答
        channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
    }
}
```

使用http client测试，连续发送两次请求，测试结果如下：

```json
# WorkQueues 测试
GET {{work_queues_host}}/amqp/send?message=workqueues
```

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-68.png)


