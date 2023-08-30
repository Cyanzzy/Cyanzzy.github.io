---
title: 3 服务调用 Ribbon、OpenFeign
date: 2023-07-23 20:50:31
tags: 
  - Distributed Microservices
categories: 
  - Technology
password: zzy   
message: 亲，能不能输入密码啊？
---

# 负载均衡服务调用 Ribbon

##  Ribbon介绍

[Spring Cloud Ribbon](https://github.com/Netflix/ribbon/wiki/Getting-Started)是基于Netflix Ribbon实现的一套客户端 **负载均衡**的工具，Ribbon主要提供**客户端的软件负载均衡算法和服务调用**Ribbon客户端组件提供一系列完善的配置项（如连接超时、重试）

在配置文件中列出`Load Balancer`所有的机器，Ribbon自动帮你基于某种规则，连接这些机器



> Ribbon功能

* 负载均衡：将用户的请求平摊分配到多个服务器上，从而达到高可用（常见的负载均衡软件有Nginx、LVS、硬件F5）
  * 集中式LB：在服务的消费方和提供方之间使用独立的LB设施，由该设施负责把访问请求通过某种策略转发至服务的提供方
  * 进程内LB：将LB逻辑集成到消费方，消费方从服务注册中心获知哪些地址可用，然后自己再从这些地址中选择出一个合适的服务器
  * **Ribbon属于进程内LB**，它只是一个类库，集成于消费方进程，消费方通过它获取到服务提供方的地址
* 负载均衡+RestTemplate调用

> 补充：
>
> Ribbon本地负载均衡客户端VS. Nginx服务端负载均衡
>
> * Nginx是服务器负载均衡，客户端所有请求都会交给Nginx，然后有Nginx实现转发请求。负载均衡是由服务端实现的
> * Ribbon本地负载均衡，在调用微服务接口时，会在注册中心获取注册信息服务列表后缓存到JVM本地，从而在本地实现RPC远程服务调用技术

## Ribbon的负载均衡和Rest调用

Ribbon其实就是软负载均衡的客户端组件，可以和其他所需请求的客户端结合使用

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-springcloud-20230723-28.png)

1. 先选择EurekaServer，它优先选择在同一个区域内负载较少的server

2. 再根据用户指定的策略，从server取到的服务注册列表中选择一个地址

   其中Ribbon提供多种策略（轮询、随机和根据响应时间加权等）

> **RestTemplate**

| 方法             | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| **getForObject** | 返回对象为响应体中数据转换成的对象，基本可以理解为Json       |
| **getForEntity** | 返回对象为ResponseEntity对象，包含响应中的一些重要信息（如响应头、响应状态码、响应体） |

## Ribbon核心组件IRule

### Ribbon默认自带的负载规则

IRule：根据特定算法从服务列表中选取一个要访问的服务

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-springcloud-20230723-29.png)

| 所属包名                                | 说明                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| com.netflix.loadbalancer.RoundRobinRule | 轮询                                                         |
| com.netflix.loadbalancer.RandomRule     | 随机                                                         |
| com.netflix.loadbalancer.RetryRule      | 先按照RoundRobinRule的策略获取服务。如果获取服务失败，则在指定时间内会进行重试，获取可用的服务 |
| WeightedResponseTimeRule                | 对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择 |
| BestAvailableRule                       | 先过滤由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务 |
| AvailabilityFilteringRule               | 先过滤掉故障实例，再选择并发较小的实例                       |
| ZoneAvoidanceRule                       | 默认规则，复合判断server所在区域的性能和server的可用性选择服务器 |

### Ribbon负载规则替换

**1.修改cloud-consumer-order-80**

**2.配置细节**

**官网文档明确给出警告：该自定义配置类不能放在`@ComponentScan`所扫描的当前包以及子包下，否则自定义的配置类会被所有的Ribbon客户端共享，从而达不到特殊化定制目的**

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-springcloud-20230723-30.png)

**3.新建包com.cyan.myrule**

**4.新建类添加规则**

```java
@Configuration
public class MyselfRule {

    @Bean
    public IRule myRule() {
        return new RandomRule();
    }
}
```

**5.主启动添加`RibbonClient`**

```java
@SpringBootApplication
@EnableEurekaClient
@RibbonClient(name = "CLOUD-PAYMENT-SERVICE", configuration = MyselfRule.class)
public class OrderMain80 {

    public static void main(String[] args) {
        SpringApplication.run(OrderMain80.class, args);
    }
}
```

##  Ribbon均衡算法

### Ribbon默认负载轮询算法原理

> **负载均衡算法：**
>
> **rest接口第几次请求数 % 服务器集群总数量 = 实际调用服务器位置下标​**
>
> 每次服务重启动后rest接口计数从1开始

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-springcloud-20230723-31.png)

### 手写本地负载均衡器

**1.7001/7002集群启动**

**2.8001/8002微服务改造**

```java
@GetMapping("/lb")
public String getPaymentLB() {
    return serverPort;
}
```

**3.80订单微服务改造**

> ApplicationContextConfig去掉`LoadBalanced`

```java
@Configuration
public class ApplicationContextConfig {

    @Bean
//    @LoadBalanced // 赋予RestTemplate负载均衡能力
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

> LoadBalancer接口

```java
public interface LoadBalancer {

    ServiceInstance instances(List<ServiceInstance> serviceInstances);
}
```

> MyLB

```java
@Component
public class MyLB implements LoadBalancer{

    private AtomicInteger atomicInteger = new AtomicInteger(0);

    public final int getAndIncrement() {
        int current;
        int next;

        do {
            current = this.atomicInteger.get();
            next = current >= Integer.MAX_VALUE ? 0 : current + 1;

        } while (!this.atomicInteger.compareAndSet(current, next));
        System.out.println("(第几次访问)next = " + next);
        return next;
    }

    @Override
    public ServiceInstance instances(List<ServiceInstance> serviceInstances) {

        int index = getAndIncrement() % serviceInstances.size();
        return serviceInstances.get(index);
    }
}
```

> OrderController

```java
 @GetMapping("payment/lb")
public String getPaymentLB() {
    List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");

    if (instances == null || instances.size() <= 0) {
        return null;
    }

    ServiceInstance serviceInstance = loadBalancer.instances(instances);
    URI uri = serviceInstance.getUri();

    return restTemplate.getForObject(uri + "/payment/lb", String.class);
}
```

> 测试

http://localhost/consumer/payment/lb

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-springcloud-20230723-32.png)

#  服务接口调用OpenFeign

##  OpenFeign介绍

[Feign](https://spring.io/projects/spring-cloud-openfeign)是一个声明式WebService客户端，使用Feign能让编写的WebService客户端更加简单。它的使用方法是**定义一个服务接口然后添加注解**。

Feign也支持可拔插式的编码器和解码器。SpringCloud对Feign进行封装，使其支持SpringMVC标准注解和HttpMessageConverters。Feign可以与Eureka和Ribbon组合使用可以支持负载均衡

> Feign作用

Feign旨在使Java Http客户端编写更容易。前面使用Riibbon+RestTempate时，利用RestTemplate对http请求的封装处理，形成模板化的**调用方法**。但实际开发中，**由于对服务依赖的调用可能不止一处**，**往往一个接口会被多处调用，通常都会针对每个微服务自行封装一些客户端来包装这些依赖服务的调用。**

所以Feign在此基础上进一步封装，由它帮我们定义和实现依赖服务接口的定义。在Feign的实现下，仅需要创建一个接口并使用注解的方式来配置它（以前是Dao接口标注Mapper注解，现在一个微服务接口上面标注Feign即可），**即可完成对服务提供方的接口绑定**，简化使用SpringCloud Ribbon时，自动封装服务调用客户端的开发量

> Feign集成了Ribbon

利用Ribbon维护Payment的服务列表信息，并且通过轮询实现客户端的负载均衡，而与Ribbon不同的是，**通过Feign只需要定义服务绑定接口且以声明式的方法**，优雅而简单的实现服务的调用

> Feign与OpenFeign的区别

| Feign                                                        | OpenFeign                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Feign是SpringCloud组件中的一个轻量级RESTful的HTTP服务客户端Feign内置Ribbon，用来做客户端负载均衡，去调用服务注册中心的服务。Feign的使用方式：使用Feign的注解定义接口，调用该接口，就可以调用服务注册中心的服务 | OpenFeign是SpringCloud在Feign的基础上支持SpringMVC注解。OpenFeign的`@FeignClient`可以解析SpringMVC的@RequestMapping注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务 |
| spring-cloud-starter-feign                                   | spring-cloud-starter-open-feign                              |

##  OpenFeign服务调用

> 接口+注解：
>
> 微服务调用接口+@FeignClient

**1.新建cloud-consumer-feign-order-80**

<font color = "red">**Feign在消费端使用**</font>

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-springcloud-20230723-33.png)

**2.引入pom**

* spring-cloud-starter-openfeign
* spring-cloud-starter-netflix-eureka-client
* spring-boot-starter-web
* spring-boot-starter-actuator
* spring-boot-starter-test

**3.配置YML**

```yml
server:
  port: 80

eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/
```

**4.添加主启动类**

```java
@SpringBootApplication
@EnableFeignClients
public class OrderFeignMain80 {

    public static void main(String[] args) {
        SpringApplication.run(OrderFeignMain80.class, args);
    }
}
```

**6.处理业务**

> 业务逻辑接口+@FeignClient配置调用provider服务

> 新建PaymentFeignService接口并新增注解@FeignClient

```java
@FeignClient(value = "CLOUD-PAYMENT-SERVICE")
public interface PaymentFeignService {

    @GetMapping("/payment/get/{id}")
    CommonResult getPaymentById(@PathVariable("id") Long id);

}
```

> 控制层Controller

```java
@Slf4j
@RestController
@RequestMapping("/consumer")
public class OrderFeignController {

    @Resource
    private PaymentFeignService paymentFeignService;

    @GetMapping("/payment/get/{id}")
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id) {
        return paymentFeignService.getPaymentById(id);
    }

}
```

**7.测试**

1. 先启动eureka集群7001 7002

2. 再启动两个微服务8001 8002

3. 启动OpenFeign

4. 访问http://localhost/consumer/payment/get/1

   **Feign自带负载均衡配置项**

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-springcloud-20230723-34.png)

## OpenFeign超时控制

默认Feign客户端只等待一秒钟，但是服务端处理需要超过一秒钟，导致Feign客户端不想再等待，直接返回报错，为了避免该情况，我们需要设置Feign客户端的**超时控制**

```yml
# 设置Feign客户端超时时间
ribbon:
  # 建立连接后从服务器读取可用资源所用的时间
  ReadTimeOut: 5000
  # 建立连接所用时间
  ConnectTimeOut: 5000
```

## OpenFeign日志增强

> 日志打印功能

Feign提供日志打印功能，可以通过配置来调整日志级别，从而了解Feign中Http请求的细节 (**对Feign接口的调用情况进行监控和输出**)

> 日志级别

| 日志级别 | 说明                                                        |
| -------- | ----------------------------------------------------------- |
| NONE     | 默认，不显示任何日志                                        |
| BASIC    | 仅记录请求方法、URL、响应状态码以及执行时间                 |
| HEADERS  | 除了BASIC中定义的信息之外，还有请求和响应的头信息           |
| FULL     | 除了HEADERS中定义的信息之外，还有请求和响应的正文以及元数据 |

> 配置

```java
@Configuration
public class FeignConfig {

    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```

> 开启日志的Feign客户端

```yml
logging:
  level:
    # feign日志级别 以及监控监控
    com.cyan.springcloud.service.Payment: debug
```

