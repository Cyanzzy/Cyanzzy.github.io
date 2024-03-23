---
title: Redis 常见面试题总结 Ⅱ
date: 2024-02-28 23:06:47
tags: 
  - Redis
categories: 
  - Interview
password: zzy   
message: 仅管理员可见
---

#  Redis 事务

Redis 事务是一组命令的有序执行序列，通过事务可以将一系列的命令作为一个单独的执行单元来执行，从而确保这些命令要么全部执行，要么全部不执行。

> 事务提供一种将多个命令组合在一起执行的机制，同时保持原子性 

1. **MULTI：** 用于开启一个事务块。
2. **EXEC：** 用于执行所有在 `MULTI` 和 `EXEC` 之间的命令。
3. **DISCARD：** 用于取消事务，放弃所有已经入队的命令。
4. **WATCH：** 用于监视一个或多个键，如果在事务执行之前这些键被其他命令所改动，事务将被打断。

 **Redis 事务实际开发中使用的非常少，功能比较鸡肋，不要将其和我们平时理解的关系型数据库的事务混淆了。 Redis 事务是不建议在日常开发中使用的。** 

> （不建议）使用 Redis 事务
>
> Redis 官网相关介绍 https://redis.io/topics/transactions 

Redis 可以通过 **`MULTI`，`EXEC`，`DISCARD` 和 `WATCH`** 等命令来实现事务功能：

```sh
> MULTI
OK
> SET PROJECT "JavaGuide"
QUEUED
> GET PROJECT
QUEUED
> EXEC
1) OK
2) "JavaGuide"
```

 [`MULTI`](https://redis.io/commands/multi)  命令后可以输入多个命令，Redis 不会立即执行这些命令，而是将它们放到队列，当调用  [`EXEC`](https://redis.io/commands/exec)  命令后，再执行所有的命令：

1. 开始事务（`MULTI`）；
2. 命令入队(批量操作 Redis 的命令，先进先出（FIFO）的顺序执行)；
3. 执行事务(`EXEC`)。

 [`DISCARD`](https://redis.io/commands/discard)  命令用于取消一个事务，它会清空事务队列中保存的所有命令：

```sh
> MULTI
OK
> SET PROJECT "JavaGuide"
QUEUED
> GET PROJECT
QUEUED
> DISCARD
OK
```

[`WATCH`](https://redis.io/commands/watch)  命令用于监听指定的 Key，当调用 `EXEC` 命令执行事务时，如果一个被 `WATCH` 命令监视的 Key 被 **其他客户端/Session** 修改的话，整个事务都不会被执行：

```sh
# 客户端 1
> SET PROJECT "RustGuide"
OK
> WATCH PROJECT
OK
> MULTI
OK
> SET PROJECT "JavaGuide"
QUEUED

# 客户端 2
# 在客户端 1 执行 EXEC 命令提交事务之前修改 PROJECT 的值
> SET PROJECT "GoGuide"

# 客户端 1
# 修改失败，因为 PROJECT 的值被客户端2修改了
> EXEC
(nil)
> GET PROJECT
"GoGuide"
```

如果 **WATCH** 与 **事务** 在同一个 Session 里，并且被 **WATCH** 监视的 Key 被修改的操作发生在事务内部，这个事务是可以被执行成功的（相关 issue：[WATCH 命令碰到 MULTI 命令时的不同效果](https://github.com/Snailclimb/JavaGuide/issues/1714) ）

 事务内部修改 WATCH 监视的 Key： 

```sh
> SET PROJECT "JavaGuide"
OK
> WATCH PROJECT
OK
> MULTI
OK
> SET PROJECT "JavaGuide1"
QUEUED
> SET PROJECT "JavaGuide2"
QUEUED
> SET PROJECT "JavaGuide3"
QUEUED
> EXEC
1) OK
2) OK
3) OK
127.0.0.1:6379> GET PROJECT
"JavaGuide3"
```

 事务外部修改 WATCH 监视的 Key： 

````sh
> SET PROJECT "JavaGuide"
OK
> WATCH PROJECT
OK
> SET PROJECT "JavaGuide2"
OK
> MULTI
OK
> GET USER
QUEUED
> EXEC
(nil)
````



## Redis 事务不满足原子性

> Redis 的事务和关系型数据库的事务不同，事务具有四大特性： 

1. **原子性（Atomicity）：** 事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用；
2. **隔离性（Isolation）：** 并发访问数据库时，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的；

3. **持久性（Durability）：** 一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。
4. **一致性（Consistency）：** 执行事务前后，数据保持一致，多个事务对同一个数据读取的结果是相同的；

> Redis 事务在运行错误的情况下，除执行过程中出现错误的命令外，其他命令都能正常执行。并且 Redis 事务是不支持回滚（roll back）操作的。**因此，Redis 事务其实是不满足原子性的。** 

Redis 开发者们觉得没必要支持回滚，这样更简单便捷并且性能更好。Redis 开发者觉得即使命令执行错误也应该在开发过程中就被发现而不是生产过程中。 

 **相关 issue** : 

- [issue#452: 关于 Redis 事务不满足原子性的问题](https://github.com/Snailclimb/JavaGuide/issues/452) 

- [Issue#491:关于 Redis 没有事务回滚？](https://github.com/Snailclimb/JavaGuide/issues/491)

## Redis 事务没法保证持久性

> Redis 不同于 Memcached 的很重要一点就是，Redis 支持持久化，而且支持 3 种持久化方式：

- 快照（snapshotting，RDB）
- 只追加文件（append-only file, AOF）
- RDB 和 AOF 的混合持久化（Redis 4.0 新增）

 与 RDB 持久化相比，AOF 持久化的实时性更好。在 Redis 的配置文件中存在三种不同的 AOF 持久化方式（ `fsync`策略），它们分别是： 

```properties
appendfsync always    #每次有数据修改发生时都会调用fsync函数同步AOF文件,fsync完成后线程返回,这样会严重降低Redis的速度
appendfsync everysec  #每秒钟调用fsync函数同步一次AOF文件
appendfsync no        #让操作系统决定何时进行同步，一般为30秒一次
```

*  AOF 持久化的`fsync`策略为 `no`、`everysec` 时都会存在数据丢失的情况 
*  AOF 持久化的`fsync`策略为 `always` 时可以基本是可以满足持久性要求的，但性能太差，实际开发过程中不会使用。 

 **因此，Redis 事务的持久性也是没办法保证的。** 

## 如何解决 Redis 事务的缺陷？

> Redis 从 2.6 版本开始支持执行 Lua 脚本，它的功能和事务非常类似。**利用 Lua 脚本来批量执行多条 Redis 命令**，这些 Redis 命令会被提交到 Redis 服务器一次性执行完成，大幅减小了网络开销。 

一段 Lua 脚本可以视作一条命令执行，一段 Lua 脚本执行过程中不会有其他脚本或 Redis 命令同时执行，保证了操作不会被其他指令插入或打扰。 

如果 Lua 脚本运行时出错并中途结束，出错后的命令是不会被执行的。并且出错前执行的命令是不会被执行的。并且出错前执行的命令是无法被撤销的，无法实现类似关系型数据库执行失败可以回滚的那种原子性效果。

因此， **严格来说的话，通过** **Lua 脚本来批量执行 Redis 命令实际也是不完全满足原子性的。** 

 **如果想要让 Lua 脚本中的命令全部执行，必须保证语句语法和命令都是对的。** 

 另外，Redis 7.0 新增了 [Redis functions](https://redis.io/docs/manual/programmability/functions-intro/)  特性，你可以将 Redis functions 看作是比 Lua 更强大的脚本。 

#  Redis 性能优化 

> 推荐文章

- [你的 Redis 真的变慢了吗？性能优化如何做 - 阿里开发者](https://mp.weixin.qq.com/s/nNEuYw0NlYGhuKKKKoWfcQ)

- [Redis 常见阻塞原因总结 - JavaGuide](https://javaguide.cn/database/redis/redis-common-blocking-problems-summary.html)

## 使用批量操作减少网络传输

> 一个 Redis 命令的执行可以简化为以下 4 步： 

1. 发送命令
2. 命令排队
3. 命令执行
4. 返回结果

 其中，第 1 步和第 4 步耗费时间之和称为 **Round Trip Time (RTT,往返时间)** ，也就是数据在网络上传输的时间。 

 **使用批量操作可以减少网络传输次数，进而有效减小网络开销，大幅减少 RTT。** 

 另外，除了能减少 RTT 之外，发送一次命令的 socket I/O 成本也比较高（涉及上下文切换，存在`read()`和  `write()`系统调用），批量操作还可以减少 socket I/O 成本。

这个在官方对 pipeline 的介绍中有提到：  https://redis.io/docs/manual/pipelining/ 

###  原生批量操作命令

 Redis 中有一些原生支持批量操作的命令，比如： 

- `MGET`（获取一个或多个指定 key 的值）、`MSET`（设置一个或多个指定 key 的值）

- `HMGET`（获取指定哈希表中一个或者多个指定字段的值）、`HMSET`（同时将一个或多个 field-value 对设置到指定哈希表中）

- `SADD`（向指定集合添加一个或多个元素）

> 在 Redis 官方提供的分片集群解决方案 Redis Cluster 下，使用这些原生批量操作命令可能会存在问题

比如说 `MGET` 无法保证所有的 key 都在同一个 **hash slot**（哈希槽）上，`MGET`可能还是需要多次网络传输，原子操作也无法保证。不过，相较于非批量操作，还是可以节省不少网络传输次数。 

> 整个步骤的简化版如下（通常由 Redis 客户端实现，无需我们自己再手动实现）： 

1. 找到 key 对应的所有 hash slot；
2. 分别向对应的 Redis 节点发起 `MGET` 请求获取数据；
3. 等待所有请求执行结束，重新组装结果数据，保持跟入参 key 的顺序一致，然后返回结果。

 如果想要解决这个多次网络传输的问题，比较常用的办法是自己维护 key 与 slot 的关系。不过这样不太灵活，虽然带来了性能提升，但同样让系统复杂性提升。 

Redis Cluster 并没有使用一致性哈希，采用的是 **哈希槽分区** ，每一个键值对都属于一个 **hash slot**（哈希槽） 。当客户端发送命令请求时，需要先根据 key 计算找到的对应的哈希槽，然后再查询哈希槽和节点的映射关系，即可找到目标 Redis 节点。 

### pipeline

对于不支持批量操作的命令，可以利用 **pipeline（流水线)** 将一批 Redis 命令封装成一组，这些 Redis 命令会一次性提交到 Redis 服务器，只需要一次网络传输。

不过需要注意控制一次批量操作的 **元素个数**（例如 500 以内，实际也和元素字节数有关），避免网络传输的数据量过大。 

与`MGET`、`MSET`等原生批量操作命令一样，pipeline 同样在 Redis Cluster 上使用会存在问题。原因类似，无法保证所有的 key 都在同一个 **hash slot**（哈希槽）上。如果想要使用的话，客户端需要自己维护 key 与  slot 的关系。 

> 原生批量操作命令和 pipeline 的是有区别的，使用的时候需要注意： 

- 原生批量操作命令是原子操作，pipeline 是非原子操作

- pipeline 可以打包不同的命令，原生批量操作命令不可以

- 原生批量操作命令是 Redis 服务端支持实现的，而 pipeline 需要服务端和客户端的共同实现 

>   pipeline 和 Redis 事务的对比： 

- 事务是原子操作，pipeline 是非原子操作。两个不同的事务不会同时运行，而 pipeline 可以同时以交错方式执行

- Redis 事务中每个命令都需要发送到服务端，而 Pipeline 只需要发送一次，请求次数更少

> 事务可以看作是一个原子操作，但其实并不满足原子性。
>
> 当提到 Redis 中的原子操作时，主要指的是该操作（比如事务、Lua 脚本）不会被其他操作（比如其他事务、Lua 脚本）打扰，并不能完全保证这个操 作中的所有写命令要么都执行要么都不执行，主要因为 Redis 是不支持回滚操作。 

另外，pipeline 不适用于执行顺序有依赖关系的一批命令。就比如说，你需要将前一个命令的结果给后续的命令使用，pipeline 就没办法满足你的需求了。对于这种需求，我们可以使用 **Lua 脚本** 。

### Lua 脚本

 Lua 脚本同样支持批量操作多条命令。一段 Lua 脚本可以视作一条命令执行，可以看作是 **原子操作** ，一段 Lua 脚本执行过程中不会有其他脚本或 Redis 命令同时执行，保证了操作不会被其他指令插入或打 扰，这是 pipeline 所不具备的。 

 并且 Lua 脚本中支持一些简单的逻辑处理比如使用命令读取值并在 Lua 脚本中进行处理，这同样是 pipeline 所不具备的。 

> Lua 脚本缺陷： 

- 如果 Lua 脚本运行时出错并中途结束，之后的操作不会进行，但是之前已经发生的写操作不会撤销，所以即使使用 Lua 脚本，也不能实现类似数据库回滚的原子性。

- Redis Cluster 下 Lua 脚本的原子操作也无法保证，原因同样是无法保证所有的 key 都在同一个 **hash slot**（哈希槽）上。

## 大量 key 集中过期问题

前面提过，对于过期 key，Redis 采用的是 **定期删除+惰性/懒汉式删除** 策略。 

 定期删除执行过程中，如果突然遇到大量过期 key 的话，客户端请求必须等待定期清理过期 key 任务线程执行完成，因为这个这个定期任务线程是在 Redis **主线程**中执行的。这就导致客户端请求没办法被及时处理，响应速度会比较慢。 

>  解决方案

1. 给 key 设置随机过期时间

2. 开启 lazy-free（惰性删除/延迟释放） 。lazy-free 特性是 Redis 4.0 开始引入的，指的是让 Redis 采用异步方式延迟释放 key 使用的内存，将该操作交给单独的子线程处理，避免阻塞主线程。 

 **建议不管是否开启 lazy-free，我们都尽量给 key 设置随机过期时间** 

## Redis bigkey 

### bigkey

 在 Redis 中，`bigkey` 是指占用较大内存的键。这样的键可能对系统性能产生负面影响，因为大键可能导致网络带宽消耗、内存碎片化和慢查询等问题。

**如果一个 key 对应的 value 所占用的内存比较大，那这个 key 就可以看作是 bigkey**。具体多大才算大呢？有一个不是特别精确的参考标准 

- String 类型的 value 超过 1MB

- 复合类型（List、Hash、Set、Sorted Set 等）的 value 包含的元素超过 5000 个（不过，对于复合类型的 value 来说，不一定包含的元素越多，占用的内存就越多）。  

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/%E6%95%B0%E6%8D%AE%E5%BA%93-Redis-04.png)

### bigkey 是怎么产生的？有什么危害？

> bigkey 产生原因

- 在不合适的场景下使用 Redis 容易造成 Key 的 value 过大，如使用 String 类型的Key存放大体积二进制文件型数据；
- 业务上线前规划设计不足，没有对Key中的成员进行合理的拆分，造成个别Key中的成员数量过多；
- 未定期清理无效数据，造成如HASH类型Key中的成员持续不断地增加；
- 使用LIST类型Key的业务消费侧发生代码故障，造成对应Key的成员只增不减。

> bigkey 危害

- 客户端执行命令的时长变慢
- Redis 内存达到**maxmemory**参数定义的上限引发操作阻塞或重要的 Key 被逐出，甚至引发内存溢出
- 集群架构下，某个数据分片的内存使用率远超其他数据分片，无法使数据分片的内存资源达到均衡。
- 对大 Key 执行读请求，会使Redis实例的带宽使用率被占满，导致自身服务变慢，同时易波及相关的服务。
- 对大 Key 执行删除操作，易造成主库较长时间的阻塞，进而可能引发同步中断或主从切换。

 

> bigkey 除了会消耗更多的内存空间和带宽，还会对性能造成比较大的影响。 

大 key 还会造成阻塞问题：

- **客户端超时阻塞**。由于 Redis 执行命令是单线程处理，然后在操作大 key 时会比较耗时，那么就会阻塞 Redis，从客户端这一视角看，就是很久很久都没有响应。
- **引发网络阻塞**。每次获取大 key 产生的网络流量较大，如果一个 key 的大小是 1 MB，每秒访问量为 1000，那么每秒会产生 1000MB 的流量，这对于普通千兆网卡的服务器来说是灾难性的。
- **阻塞工作线程**。如果使用 del 删除大 key 时，会阻塞工作线程，这样就没办法处理后续的命令。
- **内存分布不均**。集群模型在 slot 分片均匀情况下，会出现数据和查询倾斜情况，部分有大 key 的 Redis 节点占用内存多，QPS 也会比较大。

 大 key 造成的阻塞问题还会进一步影响到主从同步和集群扩容。 

综上，大 key 带来的潜在问题是非常多的，我们应该尽量避免 Redis 中存在 bigkey。



### 如何发现 bigkey？

> **redis客户端工具**

redis-cli 提供了 `--bigkeys` 来查找 bigkey：

- 优点：方便、快速、安全。
- 缺点：分析结果不可定制化，准确性与时效性差。

```properties
# redis-cli -p 6379 --bigkeys

# Scanning the entire keyspace to find biggest keys as well as
# average sizes per key type.  You can use -i 0.1 to sleep 0.1 sec
# per 100 SCAN commands (not usually needed).

[00.00%] Biggest string found so far '"ballcat:oauth:refresh_auth:f6cdb384-9a9d-4f2f-af01-dc3f28057c20"' with 4437 bytes
[00.00%] Biggest list   found so far '"my-list"' with 17 items

-------- summary -------

Sampled 5 keys in the keyspace!
Total key length in bytes is 264 (avg len 52.80)

Biggest   list found '"my-list"' has 17 items
Biggest string found '"ballcat:oauth:refresh_auth:f6cdb384-9a9d-4f2f-af01-dc3f28057c20"' has 4437 bytes

1 lists with 17 items (20.00% of keys, avg size 17.00)
0 hashs with 0 fields (00.00% of keys, avg size 0.00)
4 strings with 4831 bytes (80.00% of keys, avg size 1207.75)
0 streams with 0 entries (00.00% of keys, avg size 0.00)
0 sets with 0 members (00.00% of keys, avg size 0.00)
0 zsets with 0 members (00.00% of keys, avg size 0.00
```

* 该命令会扫描 Redis 中的所有 key ，会对 Redis 的性能有一点 影响。
* 该方式只能找出每种数据结构 top 1 bigkey（占用内存最大的 String 数据类型，包含元素最多的复合数据类型）。
* 然而一个 key 的元素多并不代表占用内存也多，需要我们根据具体的业务情况来进一步判断 
* 在线上执行该命令时，为了降低对 Redis 的影响，需要指定 `-i` 参数控制扫描的频率。`redis-cli -p 6379 -- bigkeys -i 3` 表示扫描过程中每次扫描后休息的时间间隔为 3 秒。 

> **debug object**

 redis 提供一个 `debug object key` 命令： object命令： [https://redis.io/commands/object](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fredis.io%2Fcommands%2Fobject&source=article&objectId=1862794) 

需求： 找出redis中大于10KB以上的key 

为了得到该结果数据，就需要先扫描出所有的key，然后通过循环调用 `debug object key` 得到所有 key 的字节大小。 

> **分析 RDB 文件** 

 通过RDB工具对RDB文件进行扫描，可以查找出存在的bigkey：

- [redis-rdb-tools](https://github.com/sripathikrishnan/redis-rdb-tools) ： Python 语言写的用来分析 Redis 的 RDB 快照文件用的工具 
- [rdb_bigkeys](https://github.com/weiyanwei412/rdb_bigkeys) ： Go 语言写的用来分析 Redis 的 RDB 快照文件用的工具，性能更好。 

 该方法前提是 RDB 文件持久化，RDB 持久化是一种内存快照的形式，按照一定的频次进行快照落盘

这种方案是一种理想化的选择，不会影响redis主机的运行，但在对数据可靠性要求很高的场景，不会选择 RDB 持久化方案，也因此它不具有普遍适用性。 

- 优点：支持定制化分析，对线上服务无影响。
- 缺点：时效性差，RDB文件较大时耗时较长。

### 如何处理 bigkey？

> **对 bigkey 进行拆分**

例如将含有数万成员的一个HASH Key拆分为多个HASH Key，并确保每个Key的成员数量在合理范围。在Redis集群架构中，拆分大Key能对数据分片间的内存平衡起到显著作用。

> **对 bigkey 进行清理** 

 将不适用Redis能力的数据存至其它存储，并在Redis中删除此类数据。 

Redis 4.0+ 可以使用 UNLINK 命令来异步删除一个或多个指定的 key。Redis 4.0 以下可以考虑使用 SCAN 命令结合 DEL 命令来分批次删除。

>  **对过期数据进行定期清理** 

 堆积大量过期数据会造成大Key的产生，例如在HASH数据类型中以增量的形式不断写入大量数据而忽略了数据的时效性。可以通过定时任务的方式对失效数据进行清理。

 在清理 HASH 数据时，建议通过**HSCAN**命令配合**HDEL**命令对失效数据进行清理，避免清理大量数据造成Redis阻塞。  

> **采用合适的数据结构**：

 根据实际需求优化数据结构，避免不必要的内存占用。例如，如果一个列表的元素数量很大，可以考虑是否有必要将其改为有序集合。 

例如，文件二进制数据不使用 String 保存、使用 HyperLogLog 统计页面 UV、Bitmap 保存状态信息（0/1）

> **开启 lazy-free（惰性删除/延迟释放）** ：

 考虑使用懒删除策略，即等到键过期时再删除，而不是立即删除，可以减小删除操作对系统的影响。 

lazy-free 特性是 Redis 4.0 开始引入的，指的是让 Redis 采用异步方式延迟释放 key 使用的内存，将该操作交给单独的子线程处理，避免阻塞主线程。

> 推荐文章

[发现并处理Redis的大Key和热Key](https://help.aliyun.com/zh/redis/user-guide/identify-and-handle-large-keys-and-hotkeys#section-yxd-sx9-atn)

## Redis hotkey 

### hotkey

 "Hotkey" 在 Redis 中指的是被频繁访问的键，通常是因为某个特定键在短时间内接收大量的读或写请求。热键可能会成为系统的瓶颈，因为过度集中在一个键上的请求可能导致性能问题、网络负担增加等。

如果一个 key 的访问次数比较多且明显多于其他 key 的话，那这个 key 就可以看作是 **hotkey（热 Key）**。

例如：在 Redis 实例的每秒处理请求达到 5000 次，而其中某个 key 的每秒访问量就高达 2000 次，那这个 key 就可以  看作是 hotkey 

> 产生热键的原因：

- **热点数据：** 在某些业务场景中，特定的键可能是业务上的热点数据，被大量访问。预期外的访问量陡增，如突然出现的爆款商品、访问量暴涨的热点新闻、直播间某主播搞活动带来的大量刷屏点赞、游戏中某区域发生多个工会之间的战斗涉及大量玩家等。
- **缓存失效（Cache Miss）：** 当一个缓存键过期或者被删除，下一次访问时需要重新从数据源获取数据，可能导致短时间内对同一个键的大量请求。

- **写入压力：** 如果某个键接收到大量写入操作，例如频繁地执行 `SET` 操作，它可能成为热键。

### hotkey 有什么危害？

处理 hotkey 会占用大量的 CPU 和带宽，可能会影响 Redis 实例对其他请求的正常处理。此外，如果突然访问  hotkey 的请求超出了 Redis 的处理能力，**Redis 就会直接宕机**。这种情况下，大量请求将落到后面的数据库 上，可能会**导致数据库崩溃**。 

- 占用大量的CPU资源，影响其他请求并导致整体性能降低。
- 集群架构下，产生访问倾斜，即某个数据分片被大量访问，而其他数据分片处于空闲状态，可能引起该数据分片的连接数被耗尽，新的连接建立请求被拒绝等问题。
- 在抢购或秒杀场景下，可能因商品对应库存Key的请求量过大，超出Redis处理能力造成超卖。
- 热Key的请求压力数量超出Redis的承受能力易造成缓存击穿，即大量请求将被直接指向后端的存储层，导致存储访问量激增甚至宕机，从而影响其他业务。



### 如何发现 hotkey？

> **使用 Redis 自带的 `--hotkeys` 参数来查找**

Redis 4.0.3 版本中新增了 `hotkeys` 参数，该参数能够返回所有 key 的被访问次数。 

使用该方案的前提条件是 Redis Server 的 `maxmemory-policy` 参数设置为 LFU 算法，不然就会出现如下所示的  错误。 

- 优点：方便、快速、安全。
- 缺点：分析结果不可定制化，准确性与时效性差。

```sh
# redis-cli -p 6379 --hotkeys

# Scanning the entire keyspace to find hot keys as well as
# average sizes per key type.  You can use -i 0.1 to sleep 0.1 sec
# per 100 SCAN commands (not usually needed).

Error: ERR An LFU maxmemory policy is not selected, access frequency not tracked. Please note that when switching between policies at runtime LRU and LFU data will take some time to adjust.
```

 Redis 中有两种 LFU 算法： 

| LFU                                       | 说明                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| **volatile-lfu（least frequently used）** | 从已设置过期时间的数据集（`server.db[i].expires`）中挑选最不经常使用的数据淘汰。 |
| **allkeys-lfu（least frequently used）**  | 当内存不足以容纳新写入数据时，在键空间中，移除最不经常使用的 key。 |

 以下是配置文件 `redis.conf` 中的示例： 

```properties
# 使用 volatile-lfu 策略
maxmemory-policy volatile-lfu

# 或者使用 allkeys-lfu 策略
maxmemory-policy allkeys-lfu
```

 需要注意的是，`hotkeys` 参数命令也会增加 Redis 实例的 CPU 和内存消耗（全局扫描），因此需要谨慎使用。 

> **使用`MONITOR` 命令** 

 `MONITOR` 命令是 Redis 提供的一种实时查看 Redis 的所有操作的方式，可以用于临时监控 Redis 实例的操作情况，包括读写、删除等操作。 

 Redis的**MONITOR**命令能够忠实地打印Redis中的所有请求，包括时间信息、Client信息、命令以及Key信息。 

在发生紧急情况时，可以通过短暂执行**MONITOR**命令并将返回信息输入至文件，在关闭**MONITOR**命令后，对文件中请求进行归类分析，找出这段时间中的热Key。

- 优点：方便、安全。
- 缺点：会占用CPU、内存、网络资源，时效性与准确性较差。

 由于该命令对 Redis 性能的影响比较大，因此禁止长时间开启 `MONITOR`（生产环境中建议谨慎使用该命令）。 

```sh
# redis-cli
127.0.0.1:6379> MONITOR
OK
1683638260.637378 [0 172.17.0.1:61516] "ping"
1683638267.144236 [0 172.17.0.1:61518] "smembers" "mySet"
1683638268.941863 [0 172.17.0.1:61518] "smembers" "mySet"
1683638269.551671 [0 172.17.0.1:61518] "smembers" "mySet"
1683638270.646256 [0 172.17.0.1:61516] "ping"
1683638270.849551 [0 172.17.0.1:61518] "smembers" "mySet"
1683638271.926945 [0 172.17.0.1:61518] "smembers" "mySet"
1683638274.276599 [0 172.17.0.1:61518] "smembers" "mySet2"
1683638276.327234 [0 172.17.0.1:61518] "smembers" "mySet"
```

 在发生紧急情况时，我们可以选择在合适的时机短暂执行 `MONITOR` 命令并将输出重定向至文件，在关闭 `MONITOR` 命令后通过对文件中请求进行归类分析即可找出这段时间中的 hotkey。 

>  **通过业务层定位热Key** 

 通过在业务层增加相应的代码对Redis的访问进行记录并异步汇总分析。 

可以根据业务情况来预估一些 hotkey，比如参与秒杀活动的商品数据等。不过，我们无法预估所有 hotkey 的出现，比如突发的热点新闻事件等。

在业务代码中添加相应的逻辑对 key 的访问情况进行记录分析。不过，这种方式会让业务代码的复杂性增加，一般也不会采用。

- 优点：可准确并及时地定位热Key。
- 缺点：业务代码复杂度的增加，同时可能会降低一些性能。

### 如何解决 hotkey？

> 在Redis集群架构中对热Key进行复制

- 在Redis集群架构中，由于热Key的迁移粒度问题，无法将请求分散至其他数据分片，导致单个数据分片的压力无法下降。
- 此时，可以将对应热Key进行复制并迁移至其他数据分片，例如将热Key foo复制出3个内容完全一样的Key并名为foo2、foo3、foo4，将这三个Key迁移到其他数据分片来解决单个数据分片的热Key压力。

- 该方案的缺点在于需要联动修改代码，同时带来了数据一致性的挑战（由原来更新一个Key演变为需要更新多个Key），仅建议该方案用来解决临时棘手的问题。 

> 使用读写分离架构

- 如果热Key的产生来自于读请求，可以将实例改造成读写分离架构来降低每个数据分片的读请求压力，甚至可以不断地增加从节点。主节点处理写请求，从节点处理读请求。
- 但是读写分离架构在增加业务代码复杂度的同时，也会增加Redis集群架构复杂度。
- 不仅要为多个从节点提供转发层（如Proxy，LVS等）来实现负载均衡，还要考虑从节点数量显著增加后带来故障率增加的问题。Redis集群架构变更会为监控、运维、故障处理带来了更大的挑战。

- 读写分离架构同样存在缺点，在请求量极大的场景下，读写分离架构会产生不可避免的延迟，此时会有读取到脏数据的问题。因此，在读、写压力都较大且对数据一致性要求很高的场景下，读写分离架构并不是最优方案。 

> 推荐文章

[发现并处理Redis的大Key和热Key](https://help.aliyun.com/zh/redis/user-guide/identify-and-handle-large-keys-and-hotkeys#section-yxd-sx9-atn)

## 慢查询命令

### 为什么会有慢查询命令

> 一个 Redis 命令的执行可以简化为以下 4 步： 

1. 发送命令
2. 命令排队
3. 命令执行
4. 返回结果

Redis 慢查询统计的是命令执行这一步骤的耗时，慢查询命令也就是那些命令执行时间较长的命令。 

> Redis 中的大部分命令都是 O(1)时间复杂度，但也有少部分 O(n) 时间复杂度的命令，例如： 

- `KEYS *`：会返回所有符合规则的 key。

- `HGETALL`：会返回一个 Hash 中所有的键值对。
- `LRANGE`：会返回 List 中指定范围内的元素。
- `SMEMBERS`：返回 Set 中的所有元素。

- `SINTER`/`SUNION`/`SDIFF`：计算多个 Set 的交集/并集/差集。

 **由于这些命令时间复杂度是 O(n)，有时候也会全表扫描，随着 n 的增大，执行耗时也会越长。不过， 这些命令  并不是一定不能使用，但是需要明确 N 的值。另外，有遍历的需求可以使用 `HSCAN`、`SSCAN`、`ZSCAN` 代替。** 

> 除这些 O(n) 时间复杂度的命令可能会导致慢查询之外， 还有一些时间复杂度可能在 O(N) 以上的命令，例如： 

-  `ZRANGE`/`ZREVRANGE`：返回指定 Sorted Set 中指定排名范围内的所有元素。时间复杂度为 O(log(n)+m)，n  为所有元素的数量， m 为返回的元素数量，当 m 和 n 相当大时，O(n) 的时间复杂度更小。 
-  `ZREMRANGEBYRANK`/`ZREMRANGEBYSCORE`：移除 Sorted Set 中指定排名范围/指定 score 范围内的所有元素。时  间复杂度为 O(log(n)+m)，n 为所有元素的数量， m 被删除元素的数量，当 m 和 n 相当大时，O(n) 的时间复杂度更小 

Redis 提供了用于监控慢查询的命令，主要包括 `SLOWLOG` 相关的命令。通过慢查询日志，你可以了解到执行时间较长的命令，有助于识别潜在的性能问题。 

### 如何找到慢查询命令？

> 在 `redis.conf` 文件中，可以使用 `slowlog-log-slower-than` 参数设置耗时命令的阈值， 并使用 `slowlog-max-len` 参数设置耗时命令的最大记录条数。 

 当 Redis 服务器检测到执行时间超过 `slowlog-log-slower-than`阈值的命令时，就会将该命令记录在慢查询日志  (slow log) 中，这点和 MySQL 记录慢查询语句类似。当慢查询日志超过设定的最大记录条数后，Redis 会把  最早的执行命令依次舍弃。 

注意：由于慢查询日志会占用一定内存空间，如果设置最大记录条数过大，可能会导致内存占用过高的问题。 

 `slowlog-log-slower-than`和`slowlog-max-len`的默认配置如下：

```sh
# The following time is expressed in microseconds, so 1000000 is equivalent
# to one second. Note that a negative number disables the slow log, while
# a value of zero forces the logging of every command.
slowlog-log-slower-than 10000

# There is no limit to this length. Just be aware that it will consume memory.
# You can reclaim memory used by the slow log with SLOWLOG RESET.
slowlog-max-len 128
```

> 除修改配置文件之外，可以直接通过 `CONFIG` 命令直接设置： 

```sh
# 命令执行耗时超过 10000 微妙（即10毫秒）就会被记录
CONFIG SET slowlog-log-slower-than 10000
# 只保留最近 128 条耗时命令
CONFIG SET slowlog-max-len 128
```

> 获取慢查询日志的内容很简单，直接使用`SLOWLOG GET` 命令即可。 

```sh
##  获取最新的慢查询日志，可指定返回的条目数量（n）。
redis-cli SLOWLOG GET 10
```



```sh
127.0.0.1:6379> SLOWLOG GET #慢日志查询
 1) 1) (integer) 5
   2) (integer) 1684326682
   3) (integer) 12000
   4) 1) "KEYS"
      2) "*"
   5) "172.17.0.1:61152"
   6) ""
  // ...
```

> 慢查询日志中的每个条目都由以下六个值组成： 

1. 唯一渐进的日志标识符。
2. 处理记录命令的 Unix 时间戳。
3. 执行所需的时间量，以微秒为单位。
4. 组成命令参数的数组。
5. 客户端 IP 地址和端口。
6. 客户端名称。

 `SLOWLOG GET` 命令默认返回最近 10 条的的慢查询命令，你也自己可以指定返回的慢查询命令的数量 `SLOWLOG GET N`。 

> 其他比较常用的慢查询相关的命令： 

```sh
#  获取慢查询日志中的条目数量。
redis-cli SLOWLOG LEN
#  清空慢查询日志。
redis-cli SLOWLOG RESET
```



#  Redis 生产问题 

## 缓存穿透

### 缓存穿透

缓存穿透是指查询一个**一定不存在的数据**，由于缓存是未命中时需要从数据库查询，查不到数据则不写入缓存，这将导致**这个不存在的数据每次请求都要到数据库去查询**，进而给数据库带来压力。 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/%E6%95%B0%E6%8D%AE%E5%BA%93-Redis-05.png)

### 解决方案

 最基本的就是首先做好参数校验，一些不合法的参数请求直接抛出异常信息返回给客户端。比如查询的数据库 id 不能小于 0、传入的邮箱格式不对的时候直接返回错误消息给客户端等等。 

> **缓存空对象** 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-28.png)



当客户端访问不存在的数据时，先请求 redis，但是此时 redis 中没有数据，此时会访问到数据库，但是数据库中也没有数据，这个数据穿透了缓存，直击数据库

 如果缓存和数据库都查不到某个 key 的数据就写到 Redis 中去并设置过期时间，具体命令如下：  `SET key value EX 10086` 。 

**该方式用于解决请求的 key 变化不频繁的情况**，如果黑客恶意攻击，每次构建不同的请求  key，会导致 Redis 中缓存大量无效的 key 。该方案并不能从根本上解决此问题，如果非要用该方式来解决穿透问题，尽量将无效 key 的过期时间设置短点 

* 优点：实现简单，维护方便
* 缺点：额外的内存消耗；可能造成短期的不一致

```java
public Object getObjectInclNullById(Integer id) {
    // 从缓存中获取数据
    Object cacheValue = cache.get(id);
    // 缓存为空
    if (cacheValue == null) {
        // 从数据库中获取
        Object storageValue = storage.get(key);
        // 缓存空对象
        cache.set(key, storageValue);
        // 如果存储数据为空，需要设置一个过期时间(300秒)
        if (storageValue == null) {
            // 必须设置过期时间，否则有被攻击的风险
            cache.expire(key, 60 * 5);
        }
        return storageValue;
    }
    return cacheValue;
}
```

> **布隆过滤器** 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-29.png)



布隆过滤器其实采用的是哈希思想来解决这个问题，通过一个庞大的二进制数组，**去判断当前这个要查询的这个数据是否存在**，如果布隆过滤器判断存在，则放行，这个请求会去访问 redis，哪怕此时 redis 中的数据过期了，但是数据库中一定存在这个数据，在数据库中查询出来这个数据后，再将其放入到 redis 中，假设布隆过滤器判断这个数据不存在，则直接返回

该方式节约内存空间，但存在误判，误判原因在于：布隆过滤器是哈希思想，只要哈希思想，就可能存在哈希冲突

* 优点：内存占用较少，没有多余key
* 缺点：实现复杂，存在误判可能

 

把所有可能存在的请求的值都存放在布隆过滤器中，当用户请求过来，先判断用户发来的请求的值是否存在于布隆过滤器中。如果不存在，直接返回请求参数错误信息给客户端，如果存在才会走下面的流程。 

加入布隆过滤器之后的缓存处理流程图如下：

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/%E6%95%B0%E6%8D%AE%E5%BA%93-Redis-06.png)

但是，需要注意的是布隆过滤器可能会存在误判的情况。总结来说就是：**布隆过滤器说某个元素存在，小概率会误判。布隆过滤器说某个元素不在，那么这个元素一定不在** 



> 为什么会出现误判的情况呢？

 **当一个元素加入布隆过滤器中的时候，会进行哪些操作：** 

1. 使用布隆过滤器中的哈希函数对元素值进行计算，得到哈希值（有几个哈希函数得到几个哈希值）。
2. 根据得到的哈希值，在位数组中把对应下标的值置为 1。

**当我们需要判断一个元素是否存在于布隆过滤器的时候，会进行哪些操作：** 

1. 对给定元素再次进行相同的哈希计算；
2. 得到值之后判断位数组中的每个元素是否都为 1，如果值都为 1，那么说明这个值在布隆过滤器中，如果存在一个值不为 1，说明该元素不在布隆过滤器中。

可能会出现这样一种情况：**不同的字符串可能哈希出来的位置相同。**（可以适当增加位数组大小或者调整哈希函数来降低概率） 



## 缓存击穿

###  缓存击穿

缓存击穿问题也叫 **热点 Key 问题**，就是一个被 **高并发访问** 并且 **缓存重建业务较复杂** 的 key **突然失效了**，无数的请求访问会在瞬间给数据库带来巨大的冲击。

缓存击穿中请求的 key 对应的是 **热点数据** ，该数据 **存在于数据库中，但不存在于缓存中（通常是因为缓存** **中的那份数据已经过期）** 。这就可能会导致瞬时大量的请求直接打到了数据库上，对数据库造成了巨大的压 力，可能直接就被这么多请求弄宕机了。 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/%E6%95%B0%E6%8D%AE%E5%BA%93-Redis-07.png)



假设线程 1 在查询缓存后本来应该去查询数据库，然后把这个数据重新加载到缓存的，此时只要线程 1 走完这个逻辑，其他线程就都能从缓存中加载这些数据，但是 **假设在线程 1 没有走完时，后续的线程 2，线程 3，线程 4 同时过来访问当前这个方法**， 那么这些线程都不能从缓存中查询到数据，那么他们就会同一时刻来访问查询缓存，都没查到，接着同一时间去访问数据库，同时的去执行数据库代码，对数据库访问压力过大

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-32.png)

### 解决方案

> **设置热点数据永不过期或者过期时间比较长**

对于一些非常热点的数据，可以将其设置为永不过期或者过期时间比较长，确保即使缓存失效也能够尽快重新加载，减轻数据库的压力。

>  **使用互斥锁或分布式锁：** 

在缓存失效时，使用互斥锁或分布式锁来阻止多个线程同时访问数据库，只允许一个线程去加载数据，其他线程等待。当第一个线程加载完数据后，其他线程可以直接从缓存中获取。 

假设线程过来，只能一个人一个人的来访问数据库，从而避免对于数据库访问压力过大，但这也会**影响查询的性能**，因为此时会让查询的性能从并行变成了串行，可以采用 tryLock方法 + double check 来解决这样的问题。

假设现在线程 1 过来访问，他查询缓存没有命中，但是此时他获得到了锁的资源，那么线程 1 就会一个人去执行逻辑，假设现在线程 2 过来，线程 2 在执行过程中，并没有获得到锁，那么线程 2 就可以进行到休眠，直到线程 1 把锁释放后，线程 2 获得到锁，然后再来执行逻辑，此时就能够从缓存中拿到数据了。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-33.png)

>  **缓存预热** 

针对热点数据提前预热，将其存入缓存中并设置合理的过期时间。比如秒杀场景下的数据在秒杀结束之前不过期。

> **逻辑过期**

之所以会出现缓存击穿问题，是因为对 key 设置了过期时间，如果不设置过期时间就不会有缓存击穿的问题，但是不设置过期时间会导致数据就一直占用内存。**把过期时间设置在 redis 的 value 中，但这个过期时间并不会直接作用于 redis，而是后续通过逻辑去处理。**

假设线程 1 去查询缓存，然后从 value 中判断出来当前的数据已经过期，此时线程 1 去获得互斥锁，那么其他线程会进行阻塞。**获得锁的线程 1 会开启一个线程 2 去进行重建缓存的逻辑**，直到新开的线程 2 完成这个逻辑后才释放锁， 而线程 1 直接进行返回过期数据

假设现在线程 3 过来访问，由于线程线程 2 持有着锁，所以线程 3 无法获得锁，线程 3 也直接返回过期数据，**只有等到新开的线程 2 重建缓存后，其他线程才能返回正确的数据。**

该方案巧妙在于异步的构建缓存，**缺点在于在构建完缓存之前，返回的都是脏数据**。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-34.png)

> 进行对比

**互斥锁方案：**由于保证了互斥性，所以数据一致，且实现简单，因为仅仅只需要加一把锁而已，也没其他的事情需要操心，所以没有额外的内存消耗，缺点在于有锁就有死锁问题的发生，且只能串行执行性能肯定受到影响

**逻辑过期方案：** 线程读取过程中不需要等待，性能好，有一个额外的线程持有锁去进行重构数据，但是在重构数据完成前，其他的线程只能返回之前的数据，且实现起来麻烦

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-35.png)



### 缓存穿透和缓存击穿区别

- 缓存穿透中，请求的 key 既不存在于缓存中，也不存在于数据库中。 
- 缓存击穿中，请求的 key 对应的是 **热点数据** ，该数据 **存在于数据库中，但不存在于缓存中（通常是因为缓存中的那份数据已经过期）** 。 

## 缓存雪崩

###  缓存雪崩

缓存雪崩是指在 **同一时段大量的缓存 key 同时失效或者 Redis 服务宕机**，导致大量请求到达数据库，带来巨大压力。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-31.png)

### 解决方案

 **针对 Redis 服务不可用的情况：** 

1. 利用 Redis 集群提高服务的可用性，避免单机出现问题整个缓存服务都没办法使用。
2. 给缓存业务添加降级限流策略，避免同时处理大量的请求。

 **针对热点缓存失效的情况：** 

1. 给不同的 Key 的 TTL 添加随机值
2. 给业务添加多级缓存

### 缓存雪崩和缓存击穿区别

缓存雪崩和缓存击穿比较像，但缓存雪崩导致的原因是缓存中的大量或者所有数据失效，缓存击穿导致的原因 主要是某个热点数据不存在与缓存中（通常是因为缓存中的那份数据已经过期）。

## 保证缓存和数据库数据的一致性

 Cache Aside Pattern 中遇到写请求是这样的：**更新 DB，然后直接删除 cache 。** 

> 如果更新数据库成功，而删除缓存失败，有两个解决方案： 

1. **缓存失效时间变短（不推荐，治标不治本）**：我们让缓存数据的过期时间变短，这样的话缓存就会从数据库中加载数据。另外，这种解决办法对于先操作缓存后操作数据库的场景不适用。

2. **增加 cache 更新重试机制（常用）**：如果 cache 服务当前不可用导致缓存删除失败，就隔一段时间进行重试，重试次数可以自己定。如果多次重试还是失败，可以把当前更新失败的 key 存入队列  中，等缓存服务可用之后，再将缓存中对应的 key 删除即可。 

> 文章推荐

[缓存和数据库一致性问题，看这篇就够了 - 水滴与银弹](https://mp.weixin.qq.com/s?__biz=MzIyOTYxNDI5OA==&mid=2247487312&idx=1&sn=fa19566f5729d6598155b5c676eee62d&chksm=e8beb8e5dfc931f3e35655da9da0b61c79f2843101c130cf38996446975014f958a6481aacf1&scene=178&cur_album_id=1699766580538032128#rd) 



