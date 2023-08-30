---
title: 8 RabbitMQ-PublishConfirms
date: 2023-07-23 18:17:30
tags: 
  - MQ
categories: 
  - Technology
password: zzy   
message: 亲，能不能输入密码啊？
---

[Publisher confirms](https://www.rabbitmq.com/confirms.html#publisher-confirms)  是 RabbitMQ 实现可靠发布的扩展。当在信道上启用 publisher confirms时，客户端发布的消息将由代理异步确认，这意味着它们已在服务器端得到处理。

在本教程中，我们将使用 publisher confirms 来确保发布的消息已安全到达代理。我们将介绍使用发布者确认的几种策略，并解释它们的优缺点。

# Enabling Publisher Confirms on a Channel

 Publisher confirms 是对 AMQP 0.9.1 协议的 RabbitMQ 扩展，因此默认情况下不会启用。 Publisher confirms 是通过 `confirmSelect `方法在通道级别启用的：

```java
Channel channel = connection.createChannel();
channel.confirmSelect();
```

必须在预期使用 publisher confirms 的每个通道上调用此方法。确认功能只应启用一次，而不是发布的每条信息。

# Strategy #1: Publishing Messages Individually

让我们从最简单的确认发布方法开始，即**发布一条信息并同步等待确认**：

```java
while (thereAreMessagesToPublish()) {
    byte[] body = ...;
    BasicProperties properties = ...;
    channel.basicPublish(exchange, queue, properties, body);
    // uses a 5 second timeout
    channel.waitForConfirmsOrDie(5_000);
}
```

在前面的示例中，我们像往常一样发布一条信息，然后使用 `Channel#waitForConfirmsOrDie(long)` 方法等待确认。

一旦信息得到确认，该方法就会返回。如果消息在超时时间内未得到确认，或者消息被黑屏（意味着由于某种原因，代理无法处理该消息），该方法将抛出一个异常。异常处理通常包括记录错误信息和/或重试发送报文。

它会大大降低发布速度，因为一条消息的确认会阻止所有后续消息的发布。这种方法的吞吐量不会超过每秒几百条已发布信息 

>   Publisher Confirms是异步的吗？

We mentioned at the beginning that the broker confirms published messages asynchronously but in the first example the code waits synchronously until the message is confirmed. The client actually receives confirms asynchronously and unblocks the call to `waitForConfirmsOrDie accordingly`. Think of `waitForConfirmsOrDie` as a synchronous helper which relies on asynchronous notifications under the hood.

# Strategy #2: Publishing Messages in Batches

为了改进前面的示例，我们可以发布一批信息，并等待整批信息得到确认。下面的示例使用了 100 封邮件：

```java
int batchSize = 100;
int outstandingMessageCount = 0;
while (thereAreMessagesToPublish()) {
    byte[] body = ...;
    BasicProperties properties = ...;
    channel.basicPublish(exchange, queue, properties, body);
    outstandingMessageCount++;
    if (outstandingMessageCount == batchSize) {
        channel.waitForConfirmsOrDie(5_000);
        outstandingMessageCount = 0;
    }
}
if (outstandingMessageCount > 0) {
    channel.waitForConfirmsOrDie(5_000);
}
```

与等待单条消息的确认相比，等待一批消息的确认可大幅提高吞吐量（远程 RabbitMQ 节点的确认次数可达 20-30 次）。

如果发生故障，我们无法确切知道出错的原因，因此可能需要在内存中保留整批消息，以便记录有意义的信息或重新发布消息。而且这种解决方案**仍然是同步的**，因此会阻止消息的发布。

# Strategy #3: Handling Publisher Confirms Asynchronously

 broker 会异步确认已发布的消息，用户只需在客户端注册一个回调，就能收到这些确认通知：

```java
Channel channel = connection.createChannel();
channel.confirmSelect();
channel.addConfirmListener((sequenceNumber, multiple) -> {
    // code when message is confirmed
}, (sequenceNumber, multiple) -> {
    // code when message is nack-ed
})
```

有 2 个回调：一个用于已确认的消息，另一个用于 nack-ed 消息（可被代理视为丢失的消息）。每个回调都有 2 个参数：

* `sequenceNumber`：用于识别 confirmed或nack-ed message 的消息的编号
* `multiple`：是一个布尔值。如果为假，则只 confirmed/nack-ed 一条信息；如果为真，则 confirmed/nack-ed 序列号较低或相等的所有信息。

`sequenceNumber`可在发布前通过` Channel#getNextPublishSeqNo()`获取：

```java
int sequenceNumber = channel.getNextPublishSeqNo());
ch.basicPublish(exchange, queue, properties, body);
```

将消息与`sequenceNumber`相关联的一个简单方法是使用  map 。假设我们要发布字符串，因为它们很容易变成字节数组进行发布。

下面是一个代码示例，使用map将发布`sequenceNumber`与消息的字符串正文相关联：

```java
ConcurrentNavigableMap<Long, String> outstandingConfirms = new ConcurrentSkipListMap<>();
// ... code for confirm callbacks will come later
String body = "...";
outstandingConfirms.put(channel.getNextPublishSeqNo(), body);
channel.basicPublish(exchange, queue, properties, body.getBytes());
```

 The publishing code now tracks outbound messages with a map. We need to clean this map when confirms arrive and do something like logging a warning when messages are nack-ed: 

```java
ConcurrentNavigableMap<Long, String> outstandingConfirms = new ConcurrentSkipListMap<>();
ConfirmCallback cleanOutstandingConfirms = (sequenceNumber, multiple) -> {
    if (multiple) {
        ConcurrentNavigableMap<Long, String> confirmed = outstandingConfirms.headMap(
          sequenceNumber, true
        );
        confirmed.clear();
    } else {
        outstandingConfirms.remove(sequenceNumber);
    }
};

channel.addConfirmListener(cleanOutstandingConfirms, (sequenceNumber, multiple) -> {
    String body = outstandingConfirms.get(sequenceNumber);
    System.err.format(
      "Message with body %s has been nack-ed. Sequence number: %d, multiple: %b%n",
      body, sequenceNumber, multiple
    );
    cleanOutstandingConfirms.handle(sequenceNumber, multiple);
});
// ... publishing code
```

总之，异步处理发布者确认通常需要以下步骤

1. 提供一种将发布`sequenceNumber`与消息相关联的方法。

2. 在信道上注册一个确认监听器，以便在发布者的 acks/nack 到达时收到通知，执行相应的操作，如记录或重新发布被 nack 的信息。在这一步中，`sequenceNumber`与消息的关联机制可能也需要进行一些清理
3. 在发布信息前跟踪发布`sequenceNumber`

> 小结

* 单独发布信息，同步等待确认：简单，但吞吐量非常有限。
* 批量发布信息，同步等待确认：简单，吞吐量合理，但出错时很难推理。
* 异步处理：性能最佳，资源利用率最高，出错时控制得当，但要正确实施可能会有困难。

# Putting It All Together

```java
public class PublisherConfirms {

    static final int MESSAGE_COUNT = 50_000;

    static Connection createConnection() throws Exception {
        ConnectionFactory cf = new ConnectionFactory();
        cf.setHost("localhost");
        cf.setUsername("guest");
        cf.setPassword("guest");
        return cf.newConnection();
    }

    public static void main(String[] args) throws Exception {
        publishMessagesIndividually();
        publishMessagesInBatch();
        handlePublishConfirmsAsynchronously();
    }

    static void publishMessagesIndividually() throws Exception {
        try (Connection connection = createConnection()) {
            Channel ch = connection.createChannel();

            String queue = UUID.randomUUID().toString();
            ch.queueDeclare(queue, false, false, true, null);

            ch.confirmSelect();
            long start = System.nanoTime();
            for (int i = 0; i < MESSAGE_COUNT; i++) {
                String body = String.valueOf(i);
                ch.basicPublish("", queue, null, body.getBytes());
                ch.waitForConfirmsOrDie(5_000);
            }
            long end = System.nanoTime();
            System.out.format("Published %,d messages individually in %,d ms%n", MESSAGE_COUNT, Duration.ofNanos(end - start).toMillis());
        }
    }

    static void publishMessagesInBatch() throws Exception {
        try (Connection connection = createConnection()) {
            Channel ch = connection.createChannel();

            String queue = UUID.randomUUID().toString();
            ch.queueDeclare(queue, false, false, true, null);

            ch.confirmSelect();

            int batchSize = 100;
            int outstandingMessageCount = 0;

            long start = System.nanoTime();
            for (int i = 0; i < MESSAGE_COUNT; i++) {
                String body = String.valueOf(i);
                ch.basicPublish("", queue, null, body.getBytes());
                outstandingMessageCount++;

                if (outstandingMessageCount == batchSize) {
                    ch.waitForConfirmsOrDie(5_000);
                    outstandingMessageCount = 0;
                }
            }

            if (outstandingMessageCount > 0) {
                ch.waitForConfirmsOrDie(5_000);
            }
            long end = System.nanoTime();
            System.out.format("Published %,d messages in batch in %,d ms%n", MESSAGE_COUNT, Duration.ofNanos(end - start).toMillis());
        }
    }

    static void handlePublishConfirmsAsynchronously() throws Exception {
        try (Connection connection = createConnection()) {
            Channel ch = connection.createChannel();

            String queue = UUID.randomUUID().toString();
            ch.queueDeclare(queue, false, false, true, null);

            ch.confirmSelect();

            ConcurrentNavigableMap<Long, String> outstandingConfirms = new ConcurrentSkipListMap<>();

            ConfirmCallback cleanOutstandingConfirms = (sequenceNumber, multiple) -> {
                if (multiple) {
                    ConcurrentNavigableMap<Long, String> confirmed = outstandingConfirms.headMap(
                            sequenceNumber, true
                    );
                    confirmed.clear();
                } else {
                    outstandingConfirms.remove(sequenceNumber);
                }
            };

            ch.addConfirmListener(cleanOutstandingConfirms, (sequenceNumber, multiple) -> {
                String body = outstandingConfirms.get(sequenceNumber);
                System.err.format(
                        "Message with body %s has been nack-ed. Sequence number: %d, multiple: %b%n",
                        body, sequenceNumber, multiple
                );
                cleanOutstandingConfirms.handle(sequenceNumber, multiple);
            });

            long start = System.nanoTime();
            for (int i = 0; i < MESSAGE_COUNT; i++) {
                String body = String.valueOf(i);
                outstandingConfirms.put(ch.getNextPublishSeqNo(), body);
                ch.basicPublish("", queue, null, body.getBytes());
            }

            if (!waitUntil(Duration.ofSeconds(60), () -> outstandingConfirms.isEmpty())) {
                throw new IllegalStateException("All messages could not be confirmed in 60 seconds");
            }

            long end = System.nanoTime();
            System.out.format("Published %,d messages and handled confirms asynchronously in %,d ms%n", MESSAGE_COUNT, Duration.ofNanos(end - start).toMillis());
        }
    }

    static boolean waitUntil(Duration timeout, BooleanSupplier condition) throws InterruptedException {
        int waited = 0;
        while (!condition.getAsBoolean() && waited < timeout.toMillis()) {
            Thread.sleep(100L);
            waited += 100;
        }
        return condition.getAsBoolean();
    }

}
```

