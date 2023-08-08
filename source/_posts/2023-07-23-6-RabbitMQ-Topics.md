---
title: 6 RabbitMQ-Topics
date: 2023-07-23 18:16:59
tags: 
  - MQ
categories: 
  - Technology
swiper_index: 
---

# Topic exchange

发送到 `topic` exchange 的消息不能有任意的 `routing_key`--**它必须是一个用点分隔的单词列表**。这些单词可以是任何内容，但通常会指定与信息相关的一些特征。

> A few valid routing key examples, there can be as many words in the routing key as you like, up to the limit of 255 bytes. 

`stock.usd.nyse`、`nyse.vyw`、`quick.orange.rabbit`。路由密钥的字数不限，最多 255 字节。

`binding key` 的形式也必须相同。 topic exchange 背后的逻辑与直接交换类似--使用特定`routing key`发送的消息将被传送到所有使用匹配`binding key` 绑定的队列。不过，`binding key` 有两种重要的特殊情况：

* `*` (star) can substitute for exactly **one word**.

* `#` (hash) can substitute for **zero or more words.**

![](![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-35.png)

我们要发送的消息都是描述动物的。发送消息时将使用由三个单词（两个点）组成的` routing key `。路由键中的第一个词描述速度，第二个词描述颜色，第三个词描述物种：` <speed>.<colour>.<species> `。

 These bindings can be summarised as: 

* Q1 is interested in all the orange animals.
* Q2 wants to hear everything about rabbits, and everything about lazy animals.

` routing key `设置为`quick.orange.rabbit`的消息将同时送到两个队列。消息 `lazy.orange.elephant `也会同时进入两个队列。另一方面，`quick.orange.fox`只会发送到第一个队列，而 `azy.brown.fox `只会发送到第二个队列。

尽管 `lazy.pink.rabbit `匹配了两个绑定，**但它只会被送到第二个队列一次**。`quick.brown.fox `不匹配任何绑定，因此会被丢弃。

如果我们违反规则，发送包含一个或四个单词的信息，如`orange `或 `quick.orange.new.rabbit`，这些信息与任何绑定都不匹配，因此会丢失。

另一方面，`lazy.orange.new.rabbit `虽然有四个单词，但它将与最后一个绑定匹配，并被传送到第二个队列。

> Topic exchange
>
> 当队列使用 `#`（散列）`binding key `绑定时，无论路由密钥如何，它都将接收所有信息，和fanout exchange一样
>
> 如果绑定中没有使用特殊字符 `*`（星号）和`#`（散列），和 direct  exchange一样

#  EmitLogTopic

```java
public class EmitLogTopic {

    private static final String EXCHANGE_NAME = "topic_logs";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {

            channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);

            String routingKey = getRouting(argv);
            String message = getMessage(argv);

            channel.basicPublish(EXCHANGE_NAME, routingKey, null, message.getBytes("UTF-8"));
            System.out.println(" [x] Sent '" + routingKey + "':'" + message + "'");
        }
    }

    private static String getRouting(String[] strings) {
        if (strings.length < 1)
            return "anonymous.info";
        return strings[0];
    }

    private static String getMessage(String[] strings) {
        if (strings.length < 2)
            return "Hello World!";
        return joinStrings(strings, " ", 1);
    }

    private static String joinStrings(String[] strings, String delimiter, int startIndex) {
        int length = strings.length;
        if (length == 0) return "";
        if (length < startIndex) return "";
        StringBuilder words = new StringBuilder(strings[startIndex]);
        for (int i = startIndex + 1; i < length; i++) {
            words.append(delimiter).append(strings[i]);
        }
        return words.toString();
    }
}
```

# ReceiveLogsTopic

```java
public class ReceiveLogsTopic {

    private static final String EXCHANGE_NAME = "topic_logs";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);
        String queueName = channel.queueDeclare().getQueue();

        if (argv.length < 1) {
            System.err.println("Usage: ReceiveLogsTopic [binding_key]...");
            System.exit(1);
        }

        for (String bindingKey : argv) {
            channel.queueBind(queueName, EXCHANGE_NAME, bindingKey);
        }

        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [x] Received '" + delivery.getEnvelope().getRoutingKey() + "':'" + message + "'");
        };
        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> { });
    }
}
```

