---
title: Redis 常见面试题总结 Ⅰ
date: 2024-02-28 23:06:02
tags: 
  - Redis
categories: 
  - Interview
password: zzy   
message: 仅管理员可见
---

#  Redis 基础

## Redis 

* [Redis](https://redis.io/) （**RE**mote **DI**ctionary **S**erver）是基于 C 的开源 NoSQL 数据库 ，用于存储 KV 键值对数据

* Redis 的数据保存在内存中并支持持久化，因此读写速度非常快，广泛应用于分布式缓存方向

* 为满足不同的业务场景，Redis 内置多种数据类型实现
* Redis 支持事务、持久化、Lua 脚本、多种开箱即用的集群方案（Redis  Sentinel、Redis Cluster）

## Redis 为什么这么快？

> Redis 内部做了非常多的性能优化

1. Redis **基于内存**，内存的访问速度是磁盘的上千倍；
2. Redis **基于 Reactor 模式**设计开发了一套高效的事件处理模型，主要是单线程事件循环和 IO 多路复用
3. Redis **内置多种优化后**的数据类型/结构实现，性能非常高。

>  [Why is Redis so fast?](https://twitter.com/alexxubyte/status/1498703822528544770) 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/%E6%95%B0%E6%8D%AE%E5%BA%93-Redis-01.png)

1. **基于内存：** Redis 是一个基于内存的数据库，所有数据存储在内存中，这使得读取和写入非常快速。内存的随机访问速度远远高于磁盘的顺序访问速度，从而提高了 Redis 的性能。
2. **单线程模型：** Redis 使用单线程模型，通过事件循环机制处理多个并发请求。这种轻量级的线程模型避免了线程切换和同步的开销，减少了竞争条件，提高了性能。虽然是单线程，但通过非阻塞的 IO 操作和事件驱动的方式，Redis 能够支持并发连接。
3. **非阻塞 I/O：** Redis 使用非阻塞的 I/O 操作，通过 epoll 或 kqueue 等机制监听多个套接字上的事件。这使得 Redis 能够高效地处理大量并发连接，而不会因为等待 I/O 操作而阻塞整个进程。
4. **数据结构的选择：** Redis 提供了多种高效的数据结构，如字符串、哈希表、列表、集合、有序集合等。这些数据结构的实现经过优化，以满足不同的应用场景，提高了数据的存储和访问效率。

总体而言，Redis 通过在内存中存储数据、采用单线程模型、使用非阻塞 I/O 等多种优化手段，以及提供高效的数据结构和网络协议，实现了出色的性能表现。这使得 Redis 成为一个广泛应用于缓存、消息队列等场景的高性能数据库。



## 技术选型：Redis 和 Memcached 区别

分布式缓存使用的比较多的还是 **Memcached** 和 **Redis**。不过，现在基本没有看过还有项目使用 **Memcached** 来做缓存，都是直接用 **Redis**。 

**Redis：**

- **特点：** Redis 是一个开源的内存数据库，支持分布式部署，并提供多种数据结构（字符串、哈希表、列表等）。
- **优势：** 快速、灵活，支持丰富的数据结构和丰富的功能，适用于缓存、会话存储等场景。
- **注意事项：** Redis 虽然提供主从复制和分片机制，但在大规模应用中可能需要配合其他工具来解决一致性和分片的问题。

**Memcached：**

- **特点：** Memcached 是一个高性能的分布式内存缓存系统，以键值对的形式存储数据。
- **优势：** 简单、轻量，适用于对一致性要求不高的场景，例如缓存静态资源。
- **注意事项：** 不适用于需要复杂数据结构或一致性要求较高的场景。****

> **共同点**： 

1. 都是基于内存的数据库，一般都用来当做缓存使用。
2. 都有过期策略。
3. 两者的性能都非常高。

> **区别**： 

-  **Redis 支持更丰富的数据类型**（支持更复杂的应用场景）。Redis 不仅仅支持简单的 k/v 类型的数据，同时还提供 list，set，zset，hash 等数据结构的存储。Memcached 只支持最简单的 k/v 数据类型。 

-  **Redis 支持数据的持久化和灾难恢复机制**，可以将内存中的数据持久在磁盘中，重启时可以再次加载使用，而 Memcached 把数据全部存在内存中，不支持持久化
-  Redis 在服务器内存使用完后，可以将不用的数据放到磁盘上。但 Memcached 在服务器内存使用完后，就会直接报异常。
-  **Redis 支持原生的 cluster 模式 **；Memcached 没有原生的集群模式，需要依靠客户端来实现往集群中分片写入数据 

-  **Redis 使用单线程的多路 IO 复用模型**。 （Redis 6.0 针对网络数据的读写引入了多线程）；Memcached 是多线程，非阻塞 IO 复用的网络模型 

-  **Redis 支持发布订阅模型、Lua 脚本、事务等功能**； Memcached 不支持

-  **Redis 同时使用了惰性删除与定期删除**；Memcached 过期数据的删除策略只用了惰性删除 

## 技术选型：为什么要用 Redis/ 为什么要用缓存？

1. **高性能：** Redis 是基于内存的数据库，所有数据存储在内存中，因此具有快速的读写速度。采用单线程模型和非阻塞 I/O 操作，以及高效的数据结构实现，使得 Redis 能够提供出色的性能。
2. **高并发：**一般像 MySQL 这类的数据库的 QPS 大概都在 1w 左右（4 核 8g） ，但是使用 Redis 缓存之后很容易达到 10w+，甚至最高能达到 30w+（就单机 Redis 的情况，Redis 集群的话会更高）。 
3. **低延迟：** 由于数据存储在内存中，Redis 提供低延迟的读写访问。这使得它非常适用于需要快速响应的应用场景，如实时数据分析、缓存等。
4. **丰富的数据结构：** Redis 提供多种灵活的数据结构，包括字符串、哈希表、列表、集合、有序集合等。这种多样性的数据结构使得 Redis 不仅仅可以作为简单的键值存储，还能够满足复杂数据操作的需求。
5. **持久性选项：** Redis 提供多种持久化选项，包括 RDB 快照和 AOF 日志。这使得可以根据应用场景的不同选择适当的持久化方式，同时保证数据的持久性。
6. **多种应用场景：** Redis 被设计用于解决多种问题，包括缓存、会话存储、实时分析、消息队列等。其灵活性和多功能性使得它在各种应用场景中都能发挥作用。
7. **分布式支持：** Redis 提供分布式部署的支持，可以通过主从复制、分片等方式实现高可用性和横向扩展。这使得 Redis 能够处理大规模和高并发的工作负载。
8. **发布-订阅模式：** Redis 支持发布-订阅（Pub/Sub）模式，使得多个客户端之间能够进行实时的消息传递。这在实现消息队列、实时通信等场景中非常有用。

> QPS（Query Per Second）：服务器每秒可以执行的查询次数 

使用缓存能承受的数据库请求数量是远远大于直接访问数据库的，可以考虑把数据库中的部分数据转移到缓存中去，这样用户的一部分请求会直接到缓存这里而不用经过数据库。进而提高系统整体的并发。 

##  Redis 应用

> Redis 除了做缓存，还能做什么？

- **分布式锁**：通过 Redis 来做分布式锁是一种比较常见的方式。通常基于 Redisson 来实现分布式锁。关于 Redis 实现分布式锁的详细介绍，推荐文章：[分布式锁详解](https://javaguide.cn/distributed-system/distributed-lock.html) 
- **限流**：一般是通过 Redis + Lua 脚本的方式来实现限流。推荐文章：[《我司用了 6 年的 Redis 分布式限流器，可以说是非常厉害了！》](https://mp.weixin.qq.com/s/kyFAWH3mVNJvurQDt4vchA) 
- **消息队列**：Redis 自带的 List 数据结构可以作为一个简单的队列使用。Redis 5.0 中增加的 Stream 类型的数据结构更加适合用来做消息队列。  它比较类似于 Kafka，有主题和消费组的概念，支持消息持久化以及 ACK 机制。 推荐文章：[Redis 消息队列发展历程 - 阿里开发者 - 2022](https://mp.weixin.qq.com/s/gCUT5TcCQRAxYkTJfTRjJw) 
- **延时队列**：Redisson 内置了延时队列（基于 Sorted Set 实现的）。
- **分布式 Session** ：利用 String 或者 Hash 数据类型保存 Session 数据，所有的服务器都可以访问。
- **复杂业务场景**：通过 Redis 以及 Redis 扩展（比如 Redisson）提供的数据结构，可以很方便地完成很多复杂的业务场景比如**通过 Bitmap 统计活跃用户 、通过 Sorted Set 维护排行榜。**  推荐文章：[面试指北](https://www.yuque.com/snailclimb/mf2z3k/hbsnl8)

> 应用场景

- Redis 实现分布式锁
- Redis 实现消息队列
- Redis + Lua 实现限流
- Redis 分布式 Session
- 复杂业务场景：活跃用户、排行榜、秒杀系统

#  Redis 线程模型

*  **对于读写命令来说，Redis 一直是单线程模型**
*  **Redis 4.0 版本后引入多线程**执行一些大键值对的异步删除操作
*  **Redis 6.0 版本后引入多线程**来处理网络请求（提高网络 IO 读写性能）

## Redis 单线程模型

*  **Redis 基于 Reactor 模式设计开发了一套高效的事件处理模型** （Netty 的线程模型也基于 Reactor 模式，  Reactor 模式不愧是高性能 IO 的基石），这套事件处理模型对应的是 Redis 中的文件事件处理器（file event  handler）。
*  由于文件事件处理器（file event handler）是**单线程方式**运行的，所以我们一般都说 Redis 是单线程模型。 

> Reactor 模式

- [Netty](https://blog.csdn.net/ChiYoun/article/details/128315085)

- [Reactor 模式](https://zhuanlan.zhihu.com/p/347779760)

>  《Redis 设计与实现》有一段话是如是介绍文件事件处理器的：
>
>  Redis 基于 Reactor 模式开发了自己的网络事件处理器：这个处理器被称为文件事件处理器（file event handler）。 
>
>  - 文件事件处理器使用 I/O 多路复用（multiplexing）程序来同时监听多个套接字，并根据套接字目前执行的任务来为套接字关联不同的事件处理器。
>  - 当被监听的套接字准备好执行连接应答（accept）、读取（read）、写入（write）、关 闭（close）等操 作时，与操作相对应的文件事件就会产生，这时文件事件处理器就会调用套接字之前关联好的事件处理器  来处理这些事件。 
>
>  **虽然文件事件处理器以单线程方式运行，但通过使用 I/O 多路复用程序来监听多个套接字**，文件事件处理器  既实现了高性能的网络通信模型，又可以很好地与 Redis 服务器中其他同样以单线程方式运行的模块进行对  接，这保持了 Redis 内部单线程设计的简单性。 

Redis 使用 Reactor 模式来处理多个并发连接，确保高性能和可伸缩性。Reactor 模式是一种事件驱动的设计模式，用于处理并发的输入/输出操作，使得单线程能够有效地处理多个客户端连接。

Redis 的 Reactor 模式包括以下关键组件：

**事件驱动：** Redis 使用事件驱动的方式来处理客户端的请求和网络事件，Redis 主进程通过监听套接字（Socket）上的事件，然后触发相应的处理函数来响应这些事件。

**单线程模型：** Redis 采用单线程模型，即主进程（主事件循环）负责监听事件、处理事件和执行命令，该单线程通过异步非阻塞 I/O 操作来同时处理多个连接，而无需为每个连接创建一个新线程。

**事件循环：** Redis 主进程持续运行一个事件循环，不断监听并处理事件。在事件循环中，主要包含以下几个阶段：

- **监听（Listening）：** 主进程监听套接字上的事件，例如新连接的到来或现有连接上有数据可读。
- **分发（Dispatching）：** 一旦有事件发生，主进程将事件分发给相应的事件处理函数。
- **处理（Handling）：** 处理函数执行相应的操作，可能涉及到命令的执行、数据的读写等。
- **等待（Waiting）：** 如果没有新的事件发生，主进程进入等待状态，等待下一个事件的到来。

**非阻塞 I/O：** Redis 使用非阻塞 I/O 操作，通过异步方式处理多个连接。当有新数据到达时，可以立即读取而不必等待，从而充分利用 CPU 时间。

**事件处理函数：** Redis 使用不同的事件处理函数来处理不同类型的事件，例如处理新连接的事件、处理数据读取的事件、处理命令执行的事件等。

**文件事件：** Redis 使用文件事件（File Event）作为事件的基本单元，它包括套接字的可读事件、可写事件等。主进程通过监听文件事件来感知不同类型的事件。

通过上述组件的协同工作，Redis 能够以高效的方式处理大量并发连接，确保了系统的高性能和可伸缩性。这种 Reactor 模式的设计使得 Redis 成为一个在单线程下也能处理高并发的数据库系统。



> **既然是单线程，那怎么监听大量的客户端连接呢？** 

* Redis 通过 **IO 多路复用程序** 来监听来自客户端的大量连接（或者说是监听多个 socket），它会将感兴趣的事件及类型（读、写）注册到内核中并监听每个事件是否发生。 

* **I/O 多路复用技术的使用让 Redis 不需要额外创建多余的线程来监听客户端的大量连接，降低了资源的消耗**（和 NIO 中的 `Selector` 组件很像）。 

> 文件事件处理器（file event handler）主要是包含 4 个部分： 

- 多个 socket（客户端连接）

- IO 多路复用程序（支持多个客户端连接的关键）

- 文件事件分派器（将 socket 关联到相应的事件处理器）

- 事件处理器（连接应答处理器、命令请求处理器、命令回复处理器）

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/%E6%95%B0%E6%8D%AE%E5%BA%93-Redis-02.png)

> 推荐文章

[Redis 事件机制详解](http://remcarpediem.net/article/1aa2da89/) 

## Redis 6.0 前为什么不使用多线程

> 虽然说 Redis 是单线程模型，但是，实际上，**Redis 在 4.0 后的版本中就已经加入了对多线程的支持。** 

* Redis 4.0 增加的多线程主要是**针对一些大键值对的删除操作的命令**，操作这些命令就会使用主线程之外的其他线程来“异步处理”。 

* Redis 4.0 后新增 `UNLINK`（可以看作是 `DEL` 的异步版本）、`FLUSHALL ASYNC`（清空所有数据库的 所有 key，不仅仅是当前 `SELECT` 的数据库）、`FLUSHDB ASYNC`（清空当前 `SELECT` 数据库中的所有 key）等异  步命令。 

 大体上来说，**Redis 6.0 之前主要还是单线程处理。** 

>  **那 Redis6.0 前为什么不使用多线程？** 

- 单线程编程容易并且更容易维护；
- Redis 的性能瓶颈不在 CPU ，主要在内存和网络；
- 多线程就会存在死锁、线程上下文切换等问题，甚至会影响性能。

> 推荐文章

- [为什么 Redis 选择单线程模型？](https://draveness.me/whys-the-design-redis-single-thread/) 

## Redis 6.0 后为何引入了多线程？

> **Redis 6.0 引入多线程主要是为提高网络 IO 读写性能**，因为这个是 Redis 中的一个性能瓶颈（Redis 的瓶颈主要受限于内存和网络）。 

Redis 6.0 引入多线程，但是 Redis 的多线程只是在网络数据的读写这类耗时操作上使用，**执行命令仍然是单线程顺序执行**，因此不需要担心线程安全问题。 

> Redis 6.0 的多线程默认是禁用的，只使用主线程。如需开启需要设置 IO 线程数 > 1，需要修改 redis 配置文件 `redis.conf`： 

```properties
io-threads 4 #设置1的话只会开启主线程，官网建议4核的机器建议设置为2或3个线程，8核的建议设置为6个线程
```

- io-threads 的个数一旦设置，不能通过 config 动态设置。
- 当设置 ssl 后，io-threads 将不工作。

> 开启多线程后，默认只会使用多线程进行 IO 写入 writes，即发送数据给客户端，如果需要开启多线程 IO 读取 reads，同样需要修改 redis 配置文件 `redis.conf` : 

```properties
io-threads-do-reads yes

```

但是官网描述开启多线程读并不能有太大提升，因此一般情况下并不建议开启

>  推荐文章

- [Redis 6.0 新特性-多线程连环 13 问！](https://mp.weixin.qq.com/s/FZu3acwK6zrCBZQ_3HoUgw)

- [Redis 多线程网络模型全面揭秘](https://segmentfault.com/a/1190000039223696) 

## Redis 后台线程 

> 虽然经常说 Redis 是单线程模型（主要逻辑是单线程完成的），但实际还有一些后台线程用于执行一些比较耗时的操作： 

- 通过 `bio_close_file` 后台线程来释放 AOF / RDB 等过程中产生的临时文件资源。

- 通过 `bio_aof_fsync` 后台线程调用 `fsync` 函数将系统内核缓冲区还未同步到到磁盘的数据强制刷到磁盘（ AOF 文件）。

- 通过 `bio_lazy_free`后台线程释放大对象（已删除）占用的内存空间.

 在`bio.h` 文件中有定义（Redis 6.0 版本，源码地址：[https://github.com/redis/redis/blob/6.0/src/bio.h）](https://github.com/redis/redis/blob/6.0/src/bio.h）：) 

```c
#ifndef __BIO_H
#define __BIO_H

/* Exported API */
void bioInit(void);
void bioCreateBackgroundJob(int type, void *arg1, void *arg2, void *arg3);
unsigned long long bioPendingJobsOfType(int type);
unsigned long long bioWaitStepOfType(int type);
time_t bioOlderJobOfType(int type);
void bioKillThreads(void);

/* Background job opcodes */
#define BIO_CLOSE_FILE    0 /* Deferred close(2) syscall. */
#define BIO_AOF_FSYNC     1 /* Deferred AOF fsync. */
#define BIO_LAZY_FREE     2 /* Deferred objects freeing. */
#define BIO_NUM_OPS       3

#endif

```

> 推荐文章

 [Redis 6.0 后台线程有哪些？](https://juejin.cn/post/7102780434739626014) 

#  Redis 内存管理

## Redis 给缓存数据设置过期时间有啥用？

> 一般情况下，设置保存的缓存数据的时候都会设置一个过期时间。为什么呢？ 

 因为内存是有限的，如果缓存中的所有数据都是一直保存的话，分分钟直接 Out of memory。 

> Redis 自带了给缓存数据设置过期时间的功能，比如： 

```sh
127.0.0.1:6379> expire key 60 # 数据在 60s 后过期
(integer) 1
127.0.0.1:6379> setex key 60 value # 数据在 60s 后过期 (setex:[set] + [ex]pire)
OK
127.0.0.1:6379> ttl key # 查看数据还有多久过期
(integer) 56

```

 注意：**Redis 中除了字符串类型有自己独有设置过期时间的命令 `setex` 外，其他方法都需要依靠 `expire` 命令** 

> **过期时间除了有助于缓解内存的消耗，还有什么其他用么？** 

  很多时候，我们的业务场景就是需要某个数据只在某一时间段内存在，比如我们的短信验证码可能只在 1 分钟内有效，用户登录的 Token 可能只在 1 天内有效。 

 如果使用传统的数据库来处理的话，一般都是自己判断过期，这样更麻烦并且性能要差很多 

## Redis 如何判断数据是否过期

* Redis 通过一个叫做过期字典（可以看作是 hash 表）来保存数据过期的时间。
* 过期字典的键指向 Redis 数据库  中的某个 key(键)，过期字典的值是一个 long long 类型的整数，这个整数保存了 key 所指向的数据库键的过期时间（毫秒精度的 UNIX 时间戳）。 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/%E6%95%B0%E6%8D%AE%E5%BA%93-Redis-03.png)

 过期字典是存储在 redisDb 这个结构里的： 

```c
typedef struct redisDb {
    ...

    dict *dict;     //数据库键空间,保存着数据库中所有键值对
    dict *expires   // 过期字典,保存着键的过期时间
    ...
} redisDb;
```

## 过期数据删除策略

 如果假设你设置了一批 key 只能存活 1 分钟，那么 1 分钟后，Redis 是怎么对这批 key 进行删除的呢？ 

> 常用的过期数据的删除策略就两个（重要！自己造缓存轮子的时候需要格外考虑的东西）： 

1. **惰性删除**：只会在取出 key 时才对数据进行过期检查。这样对 CPU 最友好，但是可能会造成太多过期 key 没有被删除。

2. **定期删除**：每隔一段时间抽取一批 key 执行删除过期 key 操作。并且，Redis 底层会通过限制删除操作执行的时长和频率来减少删除操作对 CPU 时间的影响。

 定期删除对内存更加友好，惰性删除对 CPU 更加友好。两者各有千秋，所以 Redis 采用的是 **定期删除+惰性/懒汉式删除** 。 

> 但是，仅仅通过给 key 设置过期时间还是有问题的。因为还是可能存在定期删除和惰性删除漏掉了很多过期  key 的情况。这样就导致大量过期 key 堆积在内存里，然后就 Out of memory 了。 

 怎么解决这个问题呢？答案就是：**Redis 内存淘汰机制。** 

## Redis 内存淘汰机制 

>  相关问题：

**MySQL 里有 2000w 数据，Redis 中只存 20w 的数据，如何保证 Redis 中的数据都是热点数据?** 

| 内存淘汰策略                            | 说明                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| **volatile-lru（least recently used）** | 从**已设置过期时间**的数据集（`server.db[i].expires`）中挑选**最近最少使用的**数据淘汰。 |
| **volatile-ttl**                        | 从**已设置过期时间**的数据集（`server.db[i].expires`）中挑选**将要过期的**数据淘汰。 |
| **volatile-random**                     | 从**已设置过期时间**的数据集（`server.db[i].expires`）中**任意选择**数据淘汰。 |
| **allkeys-lru（least recently used）**  | 当内存不足以容纳新写入数据时，在键空间中，移除**最近最少使用的** key（这个是最常用的）。 |
| **allkeys-random**                      | 从数据集（`server.db[i].dict`）中**任意选择**数据淘汰。      |
| **no-eviction**                         | 不淘汰任何数据，当内存不足时，新增操作会报错，`Redis` **默认内存淘汰策略**； |

 4.0 版本后增加以下两种： 

| 内存淘汰策略                              | 说明                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| **volatile-lfu（least frequently used）** | 从**已设置过期时间**的数据集（`server.db[i].expires`）中挑选**最不经常使用的**数据淘汰。 |
| **allkeys-lfu（least frequently used）**  | 当内存不足以容纳新写入数据时，在键空间中，移除**最不经常使用的** key。 |

> 推荐文章

[Redis 内存淘汰机制与算法](https://juejin.cn/post/7059326976334495774)

[内存淘汰机制和过期数据区别](https://www.xiaolincoding.com/redis/module/strategy.html#%E8%BF%87%E6%9C%9F%E5%88%A0%E9%99%A4%E7%AD%96%E7%95%A5)