---
title: 企业开发进阶-7-Distributed-Microservices
date: 2023-07-23 19:55:16
tags: 
  - Distributed Microservices
categories: 
  - Technology
swiper_index: 
---

在了解分布式微服务之前，请先了解[Dubbo](https://cyanzzy.github.io/2023/07/23/%E4%BC%81%E4%B8%9A%E5%BC%80%E5%8F%91%E8%BF%9B%E9%98%B6-3-Dubbo/)的核心思想。

> SpringCloud 全家桶

[1 微服务入门](https://cyanzzy.github.io/2023/07/23/1-%E5%BE%AE%E6%9C%8D%E5%8A%A1%E5%85%A5%E9%97%A8/)

[2 服务注册与发现 Eureka、Zookeeper、Consul](https://cyanzzy.github.io/2023/07/23/2-%E6%9C%8D%E5%8A%A1%E6%B3%A8%E5%86%8C%E4%B8%8E%E5%8F%91%E7%8E%B0-Eureka%E3%80%81Zookeeper%E3%80%81Consul/)

[3 服务调用 Ribbon、OpenFeign](https://cyanzzy.github.io/2023/07/23/3-%E6%9C%8D%E5%8A%A1%E8%B0%83%E7%94%A8-Ribbon%E3%80%81OpenFeign/)

[4 服务熔断与降级 Hystrix](https://cyanzzy.github.io/2023/07/23/4-%E6%9C%8D%E5%8A%A1%E7%86%94%E6%96%AD%E4%B8%8E%E9%99%8D%E7%BA%A7-Hystrix/)

[5 服务网关 Gateway](https://cyanzzy.github.io/2023/07/23/5-%E6%9C%8D%E5%8A%A1%E7%BD%91%E5%85%B3-Gateway/)

[6 服务配置 Config](https://cyanzzy.github.io/2023/07/23/6-%E6%9C%8D%E5%8A%A1%E9%85%8D%E7%BD%AE-Config/)

[7 消息总线 Bus](https://cyanzzy.github.io/2023/07/23/7-%E6%B6%88%E6%81%AF%E6%80%BB%E7%BA%BF-Bus/)

[8 消息驱动 Stream](https://cyanzzy.github.io/2023/07/23/8-%E6%B6%88%E6%81%AF%E9%A9%B1%E5%8A%A8-Stream/)

[9 分布式请求链路追踪 Sleuth](https://cyanzzy.github.io/2023/07/23/9-%E5%88%86%E5%B8%83%E5%BC%8F%E8%AF%B7%E6%B1%82%E9%93%BE%E8%B7%AF%E8%BF%BD%E8%B8%AA-Sleuth/)

> SpringCloudAlibaba 全家桶

[1 服务注册和配置中心 Nacos](https://cyanzzy.github.io/2023/07/23/1-%E6%9C%8D%E5%8A%A1%E6%B3%A8%E5%86%8C%E5%92%8C%E9%85%8D%E7%BD%AE%E4%B8%AD%E5%BF%83-Nacos/)

[2 服务熔断和限流 Sentinel](https://cyanzzy.github.io/2023/07/23/2-%E6%9C%8D%E5%8A%A1%E7%86%94%E6%96%AD%E5%92%8C%E9%99%90%E6%B5%81-Sentinel/)

[3 分布式事务处理 Seata](https://cyanzzy.github.io/2023/07/23/3-%E5%88%86%E5%B8%83%E5%BC%8F%E4%BA%8B%E5%8A%A1%E5%A4%84%E7%90%86-Seata/)