---
title: 5 RabbitMQ-Routing
date: 2023-07-23 18:16:43
tags: 
  - MQ
categories: 
  - Technology
password: zzy   
message: 亲，能不能输入密码啊？
---

# Direct exchange

direct exchange路由算法 -消息会进入binding key 与消息routing key完全匹配的队列。

可以看到Direct exchange X 绑定了两个队列。第一个队列的绑定键为orange，第二个队列有两个绑定键，一个绑定键为black，另一个绑定键为green。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-31.png)

在此情况下，如果发布到交换机的消息routing key 为orange，则会被路由到队列 Q1。routing key 为black或green的消息将进入 Q2，**其他消息都将被丢弃**

# Multiple bindings

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-32.png)

用同一个binding key绑定多个队列是完全合法的。在我们的例子中，我们可以用binding key black 在 X 和 Q1 之间添加一个绑定。在这种情况下，direct exchange 将像fanout exchange 一样，向所有匹配队列广播消息，routing key为 black 的消息将同时发送到 Q1 和 Q2。

# Emitting logs

我们和以往一样继续使用日志Demo，不同的是将fanout换成direct。我们将 the log severity 作为routing key。

和之前一样，我们先声明一个队列

```java
channel.exchangeDeclare(EXCHANGE_NAME, "direct");
```

然后发送消息

```java
channel.basicPublish(EXCHANGE_NAME, severity, null, message.getBytes());
```

# Subscribing

接收信息的方式与上一教程相同， 我们将为感兴趣的每个severity创建一个新的绑定。

```java
String queueName = channel.queueDeclare().getQueue();

for(String severity : argv){
  channel.queueBind(queueName, EXCHANGE_NAME, severity);
}
```

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-37.png)

> 完整代码

```java
public class EmitLogDirect {

    private static final String EXCHANGE_NAME = "direct_logs";

    public static void main(String[] argv) throws Exception {
        
        ConnectionFactory factory = new ConnectionFactory();
        
        factory.setHost("localhost");
        
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            
            channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);

            String severity = getSeverity(argv);
            String message = getMessage(argv);

            channel.basicPublish(EXCHANGE_NAME, severity, null, message.getBytes("UTF-8"));
            System.out.println(" [x] Sent '" + severity + "':'" + message + "'");
        }
    }

    private static String getSeverity(String[] strings) {
        if (strings.length < 1)
            return "info";
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
        if (length <= startIndex) return "";
        StringBuilder words = new StringBuilder(strings[startIndex]);
        for (int i = startIndex + 1; i < length; i++) {
            words.append(delimiter).append(strings[i]);
        }
        return words.toString();
    }
}
```

```java
public class ReceiveLogsDirect {

  private static final String EXCHANGE_NAME = "direct_logs";

  public static void main(String[] argv) throws Exception {
      
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
      
    Connection connection = factory.newConnection();
      
    Channel channel = connection.createChannel();

    channel.exchangeDeclare(EXCHANGE_NAME, "direct");
    String queueName = channel.queueDeclare().getQueue();

    if (argv.length < 1) {
        System.err.println("Usage: ReceiveLogsDirect [info] [warning] [error]");
        System.exit(1);
    }

    for (String severity : argv) {
        channel.queueBind(queueName, EXCHANGE_NAME, severity);
    }
    System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

    DeliverCallback deliverCallback = (consumerTag, delivery) -> {
        String message = new String(delivery.getBody(), "UTF-8");
        System.out.println(" [x] Received '" +
            delivery.getEnvelope().getRoutingKey() + "':'" + message + "'");
    };
    channel.basicConsume(queueName, true, deliverCallback, consumerTag -> { });
  }
}
```

# Spring AMQP

我们将使用 Spring Boot 来引导和配置 Spring AMQP 项目，[代码仓库链接](https://gitee.com/cyanzzy/spring-amqp-learning)，与往常一样，配置类如下

```java
@Configuration
public class RabbitMQConfig {

    @Bean
    public DirectExchange direct() { // DirectExchange
        return new DirectExchange("routing-direct");
    }

    @Bean // 创建临时队列
    public Queue autoDeleteQueue1() {
        return new AnonymousQueue();
    }

    @Bean
    public Binding binding1a(DirectExchange direct, Queue autoDeleteQueue1) {
        return BindingBuilder.bind(autoDeleteQueue1)
                .to(direct)
                .with("orange");
    }

    @Bean
    public Binding binding1b(DirectExchange direct, Queue autoDeleteQueue1) {
        return BindingBuilder.bind(autoDeleteQueue1)
                .to(direct)
                .with("black");
    }

    @Bean // 创建临时队列
    public Queue autoDeleteQueue2() {
        return new AnonymousQueue();
    }

    @Bean
    public Binding binding2a(DirectExchange direct, Queue autoDeleteQueue2) {
        return BindingBuilder.bind(autoDeleteQueue2)
                .to(direct)
                .with("green");
    }

    @Bean
    public Binding binding2b(DirectExchange direct, Queue autoDeleteQueue2) {
        return BindingBuilder.bind(autoDeleteQueue2)
                .to(direct)
                .with("black");
    }

}
```

消息发送方：

```java
@Service
public class RoutingSenderServiceImpl implements RoutingSenderService {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Autowired
    private DirectExchange direct;

    private final String[] keys = {"orange", "black", "green"};

    @Override
    public void send(String message) { // 使用RabbitTemplate向队列发送消息

        for (int i = 0; i < keys.length; i++) {
            String key = keys[i];
            System.out.println("第 " + i + "次循环发消息");

            rabbitTemplate.convertAndSend(direct.getName(), key, message);
            System.out.println(" *********************send() 发送了 ：" + message + " exchange is: " + direct.getName() + "routing key is: " + key);
        }

    }
}
```

消息接收方：

```java
@Component
public class RoutingReceiver {

    @RabbitListener(queues = "#{autoDeleteQueue1.name}")
    public void receive1(String in) {

        System.out.println("receive1: " + in);
    }

    @RabbitListener(queues = "#{autoDeleteQueue2.name}")
    public void receive2(String in) {

        System.out.println("receive2: " + in);
    }
}
```

使用http client 测试，结果如下：

```json
# Routing 测试
GET {{routing_host}}/amqp/send?message= 你好  routing and routing
```

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-rabbitmq-20230723-72.png)

