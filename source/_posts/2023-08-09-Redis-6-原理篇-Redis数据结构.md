---
title: Redis-6-原理篇-Redis数据结构
date: 2023-08-09 10:45:44
tags: 
  - Redis
categories: 
  - Technology
swiper_index: 
---

#  SDS

> Why SDS

字符串是Redis中最常用的一种数据结构。不过Redis没有直接使用C语言中的字符串，因为C语言字符串存在很多问题：

* 获取字符串长度的需要通过运算
* 非二进制安全
* 不可修改

> SDS

Redis构建了一种新的字符串结构，称为**简单动态字符串**（Simple Dynamic String），简称SDS。

Redis是C语言实现的，其中SDS是一个结构体，源码如下：

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-175.png)

> SDS底层结构

例如，一个包含字符串“name”的sds结构如下：

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-176.png)

SDS之所以叫做动态字符串，是因为它具备动态扩容的能力，例如一个内容为“hi”的SDS：

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-177.png)

假如我们要给SDS追加一段字符串“,Amy”，这里首先会申请新内存空间：

* 如果新字符串小于1M，则新空间为扩展后字符串长度的两倍+1；

* 如果新字符串大于1M，则新空间为扩展后字符串长度+1M+1。称为内存预分配。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-178.png)

> SDS优点

* 获取字符串长度时间复杂度 $O(1)$
* 支持动态 扩容
* 减少内存分配次数
* 二进制安全



# intset

IntSet是Redis中set集合的一种实现方式，基于整数数组来实现，并且具备长度可变、有序等特征。

> intset结构 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-179.png)

其中的encoding包含三种模式，表示存储的整数大小不同：

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-180.png)

为了方便查找，Redis会将intset中所有的整数按照升序依次保存在contents数组中，结构如图：

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-181.png)



现在数组中每个数字都在`int16_t`的范围内，因此采用的编码方式是`INTSET_ENC_INT16`，每部分占用的字节大小为：

* `encoding`：4字节
* `length`：4字节
* `contents`：2字节 * 3  = 6字节

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-182.png)

> 如果向该其中添加一个数字：50000，这个数字超出了`int16_t`的范围，intset会自动升级编码方式到合适的大小

* 升级编码为`INTSET_ENC_INT32`, 每个整数占4字节，并按照新的编码方式及元素个数扩容数组
* 倒序依次将数组中的元素拷贝到扩容后的正确位置
* 将待添加的元素放入数组末尾
* 最后，将inset的`encoding`属性改为`INTSET_ENC_INT32`，将`length`属性改为4

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-183.png)

> 源码如下 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-184.png)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-185.png)

> 总结：

Intset可以看做是特殊的整数数组，具备一些特点：

* Redis会确保Intset中的元素唯一、有序
* 具备类型升级机制，可以节省内存空间
* 底层采用二分查找方式来查询

# Dict

Redis是一个键值型（Key-Value Pair）的数据库，可以根据键实现快速的增删改查。而键与值的映射关系正是通过Dict来实现的。Dict由三部分组成，分别是：哈希表（DictHashTable）、哈希节点（DictEntry）、字典（Dict）

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-186.png)

当向Dict添加键值对时，Redis首先根据key计算出hash值（h），然后利用 `h & sizemask`来计算元素应该存储到数组中的哪个索引位置。我们存储k1=v1，假设k1的哈希值h =1，则1&3 =1，因此k1=v1要存储到数组角标`1`位置。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-187.png)

> Dict由三部分组成，分别是：哈希表（DictHashTable）、哈希节点（DictEntry）、字典（Dict）

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-188.png)

![1653985586543](E:/work/job/尚硅谷资源/Redis-笔记资料/04-原理篇/讲义/原理篇.assets/1653985586543.png)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-189.png)

> **Dict的扩容**

Dict中的HashTable就是数组结合单向链表的实现，当集合中元素较多时，必然导致哈希冲突增多，链表过长，则查询效率会大大降低。
Dict在每次新增键值对时都会检查负载因子（`LoadFactor = used/size`） ，满足以下两种情况时会触发哈希表扩容：

* 哈希表的`LoadFactor >= 1`，并且服务器没有执行 BGSAVE 或者 BGREWRITEAOF 等后台进程；
* 哈希表的`LoadFactor > 5` ；

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-190.png)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-191.png)

> **Dict的rehash**

不管是扩容还是收缩，必定会创建新的哈希表，导致哈希表的`size`和`sizemask`变化，而key的查询与`sizemask`有关。**因此必须对哈希表中的每一个key重新计算索引，插入新的哈希表，这个过程称为rehash**。

过程如下：

* 计算新hash表的`realeSize`，值取决于当前要做的是扩容还是收缩：
  * 如果是扩容，则新`size`为第一个大于等于`dict.ht[0].used + 1`的 $2^n$
  * 如果是收缩，则新`size`为第一个大于等于`dict.ht[0].used`的 $2^n$（不得小于4）

* 按照新的`realeSize`申请内存空间，创建dictht，并赋值给`dict.ht[1]`
* 设置`dict.rehashidx = 0`，标示开始rehash
* 将`dict.ht[0]`中的每一个dictEntry都rehash到`dict.ht[1]`
* 将`dict.ht[1]`赋值给`dict.ht[0]`，给`dict.ht[1]`初始化为空哈希表，释放原来的`dict.ht[0]`的内存
* 将`rehashidx`赋值为-1，代表rehash结束
* 在rehash过程中，新增操作，则直接写入`ht[1]`，查询、修改和删除则会在`dict.ht[0]`和`dict.ht[1]`依次查找并执行。这样可以确保ht[0]的数据只减不增，随着rehash最终为空

整个过程可以描述成：

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-192.png)

> 总结：

Dict的结构：

* 类似java的HashTable，底层是数组加链表来解决哈希冲突
* Dict包含两个哈希表，`ht[0]`平常用，`ht[1]`用来rehash

Dict的伸缩：

* 当LoadFactor大于5或者LoadFactor大于1并且没有子进程任务时，Dict扩容
* 当LoadFactor小于0.1时，Dict收缩
* 扩容大小为第一个大于等于`used + 1`的 $2^n$
* 收缩大小为第一个大于等于`used `的 $2^n$
* Dict采用渐进式rehash，每次访问Dict时执行一次rehash
* rehash时`ht[0]`只减不增，新增操作只在`ht[1]`执行，其它操作在两个哈希表

# ZipList

ZipList 是一种特殊的“双端链表” ，由一系列特殊编码的连续内存块组成。可以在任意一端进行压入/弹出操作, 并且该操作的时间复杂度为 $O(1)$ 。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-193.png)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-194.png)

| **属性** | **类型** | **长度** | **用途**                                                     |
| -------- | -------- | -------- | ------------------------------------------------------------ |
| zlbytes  | uint32_t | 4 字节   | 记录整个压缩列表占用的内存字节数                             |
| zltail   | uint32_t | 4 字节   | 记录压缩列表表尾节点距离压缩列表的起始地址有多少字节，通过这个偏移量，可以确定表尾节点的地址。 |
| zllen    | uint16_t | 2 字节   | 记录了压缩列表包含的节点数量。 最大值为UINT16_MAX （65534），如果超过这个值，此处会记录为65535，但节点的真实数量需要遍历整个压缩列表才能计算得出。 |
| entry    | 列表节点 | 不定     | 压缩列表包含的各个节点，节点的长度由节点保存的内容决定。     |
| zlend    | uint8_t  | 1 字节   | 特殊值 0xFF （十进制 255 ），用于标记压缩列表的末端。        |

> **ZipListEntry**

ZipList 中的Entry并不像普通链表那样记录前后节点的指针，因为记录两个指针要占用16个字节，浪费内存。而是采用了下面的结构：

![1653986055253](E:/work/job/尚硅谷资源/Redis-笔记资料/04-原理篇/讲义/原理篇.assets/1653986055253.png)

* previous_entry_length：前一节点的长度，占1个或5个字节。
  * 如果前一节点的长度小于254字节，则采用1个字节来保存这个长度值
  * 如果前一节点的长度大于254字节，则采用5个字节来保存这个长度值，第一个字节为0xfe，后四个字节才是真实长度数据

* encoding：编码属性，记录content的数据类型（字符串还是整数）以及长度，占用1个、2个或5个字节
* contents：负责保存节点的数据，可以是字符串或整数

ZipList中所有存储长度的数值均采用小端字节序，即低位字节在前，高位字节在后。例如：数值0x1234，采用小端字节序后实际存储值为：0x3412

> **Encoding编码**

ZipListEntry中的encoding编码分为字符串和整数两种：

**字符串**：如果encoding是以“00”、“01”或者“10”开头，则证明content是字符串

| **编码**                                         | **编码长度** | **字符串大小**      |
| ------------------------------------------------ | ------------ | ------------------- |
| `|00pppppp\|`                                    | 1 bytes      | <= 63 bytes         |
| `|01pppppp|qqqqqqqq|`                            | 2 bytes      | <= 16383 bytes      |
| `|10000000|qqqqqqqq|rrrrrrrr|ssssssss|tttttttt|` | 5 bytes      | <= 4294967295 bytes |

例如，我们要保存字符串：“ab”和 “bc”

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-195.png)

**整数**：如果encoding是以“11”开始，则证明content是整数，且encoding固定只占用1个字节

| **编码**   | **编码长度** | **整数类型**                                               |
| ---------- | ------------ | ---------------------------------------------------------- |
| `11000000` | 1            | int16_t（2 bytes）                                         |
| `11010000` | 1            | int32_t（4 bytes）                                         |
| `11100000` | 1            | int64_t（8 bytes）                                         |
| `11110000` | 1            | 24位有符整数(3 bytes)                                      |
| `11111110` | 1            | 8位有符整数(1 bytes)                                       |
| `1111xxxx` | 1            | 直接在xxxx位置保存数值，范围从0001~1101，减1后结果为实际值 |

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-196.png)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-197.png)



> ZipList的连锁更新问题

ZipList的每个Entry都包含`previous_entry_length`来记录上一个节点的大小，长度是1个或5个字节：

* 如果前一节点的长度小于254字节，则采用1个字节来保存这个长度值
* 如果前一节点的长度大于等于254字节，则采用5个字节来保存这个长度值，第一个字节为0xfe，后四个字节才是真实长度数据

现在，假设有N个连续的、长度为250~253字节之间的entry，因此entry的`previous_entry_length`属性用1个字节即可表示，如图所示：

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-198.png)

ZipList这种特殊情况下产生的连续多次空间扩展操作称之为连锁更新（Cascade Update）。新增、删除都可能导致连锁更新的发生。

> **ZipList特性总结：**

* 压缩列表的可以看做一种连续内存空间的"双向链表"
* 列表的节点之间不是通过指针连接，而是记录上一节点和本节点长度来寻址，内存占用较低
* 如果列表数据过多，导致链表过长，可能影响查询性能
* 增或删较大数据时有可能发生连续更新问题

# QuickList

> 存在问题

* ZipList虽然节省内存，但申请内存必须是连续空间，如果内存占用较多，申请内存效率很低。怎么办？

  为了缓解这个问题，我们必须限制ZipList的长度和entry大小。

* 但是我们要存储大量数据，超出了ZipList最佳的上限该怎么办？

  可以创建多个ZipList来分片存储数据。

* 数据拆分后比较分散，不方便管理和查找，这多个ZipList如何建立联系？

  Redis在3.2版本引入了新的数据结构QuickList，它是一个双端链表，只不过链表中的每个节点都是一个ZipList。

> QuickList

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-199.png)

为了避免QuickList中的每个ZipList中entry过多，Redis提供了一个配置项：`list-max-ziplist-size`来限制。
如果值为正，则代表ZipList的允许的entry个数的最大值
如果值为负，则代表ZipList的最大内存大小，分5种情况：

* -1：每个ZipList的内存占用不能超过4kb
* -2：每个ZipList的内存占用不能超过8kb
* -3：每个ZipList的内存占用不能超过16kb
* -4：每个ZipList的内存占用不能超过32kb
* -5：每个ZipList的内存占用不能超过64kb

其默认值为 -2：

> 以下是QuickList的和QuickListNode的结构源码：

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-200.png)

接下来用一段流程图来描述当前的这个结构

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-201.png)



> QuickList的特点总结

* 是一个节点为ZipList的双端链表
* 节点采用ZipList，解决了传统链表的内存占用问题
* 控制了ZipList大小，解决连续内存空间申请效率问题
* 中间节点可以压缩，进一步节省了内存

# SkipList

SkipList（跳表）首先是链表，但与传统链表相比有几点差异：

* 元素按照升序排列存储
* 节点可能包含多个指针，指针跨度不同

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-202.png)

> 跳表结构定义

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-203.png)



![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-204.png)

> SkipList的特点总结

* 跳跃表是一个双向链表，每个节点都包含score和ele值
* 节点按照score值排序，score值一样则按照ele字典排序
* 每个节点都可以包含多层指针，层数是1到32之间的随机数
* 不同层指针到下一个节点的跨度不同，层级越高，跨度越大
* 增删改查效率与红黑树基本一致，实现却更简单

# RedisObject

Redis中的任意数据类型的键和值都会被封装为一个RedisObject，也叫做Redis对象 

> redisObject：

* 从Redis的使用者的角度，⼀个Redis节点包含多个database（非cluster模式下默认是16个，cluster模式下只能是1个），而一个database维护了从key space到object space的映射关系。这个映射关系的key是string类型，⽽value可以是多种数据类型
* 从Redis内部实现的⾓度，database内的这个映射关系是用⼀个dict来维护的。dict的key固定用动态字符串sds来表达。而value则比较复杂，为了在同⼀个dict内能够存储不同类型的value，这就需要⼀个通⽤的数据结构，这个通用的数据结构就是robj，全名是redisObject。

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-205.png)

> Redis的编码方式

Redis中会根据存储的数据类型不同，选择不同的编码方式，共包含11种不同类型：

| **编号** | **编码方式**            | **说明**               |
| -------- | ----------------------- | ---------------------- |
| 0        | OBJ_ENCODING_RAW        | raw编码动态字符串      |
| 1        | OBJ_ENCODING_INT        | long类型的整数的字符串 |
| 2        | OBJ_ENCODING_HT         | hash表（字典dict）     |
| 3        | OBJ_ENCODING_ZIPMAP     | 已废弃                 |
| 4        | OBJ_ENCODING_LINKEDLIST | 双端链表               |
| 5        | OBJ_ENCODING_ZIPLIST    | 压缩列表               |
| 6        | OBJ_ENCODING_INTSET     | 整数集合               |
| 7        | OBJ_ENCODING_SKIPLIST   | 跳表                   |
| 8        | OBJ_ENCODING_EMBSTR     | embstr的动态字符串     |
| 9        | OBJ_ENCODING_QUICKLIST  | 快速列表               |
| 10       | OBJ_ENCODING_STREAM     | Stream流               |

> 五种数据结构

Redis中会根据存储的数据类型不同，选择不同的编码方式。每种数据类型的使用的编码方式如下：

| **数据类型** | **编码方式**                                       |
| ------------ | -------------------------------------------------- |
| OBJ_STRING   | int、embstr、raw                                   |
| OBJ_LIST     | LinkedList和ZipList(3.2以前)、QuickList（3.2以后） |
| OBJ_SET      | intset、HT                                         |
| OBJ_ZSET     | ZipList、HT、SkipList                              |
| OBJ_HASH     | ZipList、HT                                        |

# String

* String是Redis中最常见的数据存储类型，其基本编码方式是RAW，基于简单动态字符串（SDS）实现，存储上限为512mb。

* 如果存储的SDS长度小于44字节，则会采用EMBSTR编码，此时object head与SDS是一段连续空间。申请内存时只需要调用一次内存分配函数效率更高。

* 底层实现⽅式：动态字符串sds 或者 long。String的内部存储结构⼀般是sds（Simple Dynamic String，可以动态扩展内存），但是如果⼀个String类型的value的值是数字，那么Redis内部会把它转成long类型来存储，从⽽减少内存的使用。

  ![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-206.png)

  如果存储的字符串是整数值，并且大小在LONG_MAX范围内，则会采用INT编码：直接将数据保存在RedisObject的ptr指针位置（刚好8字节），不再需要SDS 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-207.png)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-208.png)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-209.png)

> String在Redis中是⽤⼀个robj来表示的 

用来表示String的robj可能编码成3种内部表⽰，其中前两种编码使⽤的是sds来存储，最后⼀种OBJ_ENCODING_INT编码直接把string存成了long型 

* OBJ_ENCODING_RAW
* OBJ_ENCODING_EMBSTR
* OBJ_ENCODING_INT。

在对string进行incr, decr等操作的时候，如果它内部是OBJ_ENCODING_INT编码，那么可以直接行加减操作；

如果它内部是OBJ_ENCODING_RAW或OBJ_ENCODING_EMBSTR编码，那么Redis会先试图把sds存储的字符串转成long型，如果能转成功，再进行加减操作。

对⼀个内部表示成long型的string执行append, setbit, getrange这些命令，针对的仍然是string的值（即⼗进制表示的字符串），而不是针对内部表⽰的long型进⾏操作。

# List

Redis的List结构类似一个双端链表，可以从首、尾操作列表中的元素：

* 在3.2版本之前，Redis采用ZipList和LinkedList来实现List，当元素数量小于512并且元素大小小于64字节时采用ZipList编码，超过则采用LinkedList编码 

* 在3.2版本之后，Redis统一采用QuickList来实现List 

# Set

Set是Redis中的单列集合，满足下列特点：

* 不保证有序性
* 保证元素唯一
* 求交集、并集、差集

Set是Redis中的集合，不一定确保元素有序，可以满足元素唯一、查询效率要求极高。为了查询效率和唯一性，set采用HT编码（Dict）。Dict中的key用来存储元素，value统一为null。当存储的所有数据都是整数，并且元素数量不超过set-max-intset-entries时，Set会采用IntSet编码，以节省内存

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-210.png)

> 结构如下

​	![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-211.png)

# ZSET

ZSet也就是SortedSet，其中每一个元素都需要指定一个score值和member值：

* 可以根据score值排序后
* member必须唯一
* 可以根据member查询分数

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-212.png)

因此，zset底层数据结构必须满足键值存储、键必须唯一、可排序这几个需求

* SkipList：可以排序，并且可以同时存储score和ele值（member）
* HT（Dict）：可以键值存储，并且可以根据key找value

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-213.png)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-214.png)

当元素数量不多时，HT和SkipList的优势不明显，而且更耗内存。因此zset还会采用ZipList结构来节省内存，不过需要同时满足两个条件：

* 元素数量小于zset_max_ziplist_entries，默认值128
* 每个元素都小于zset_max_ziplist_value字节，默认值64

ziplist本身没有排序功能，而且没有键值对的概念，因此需要有zset通过编码实现：

* ZipList是连续内存，因此score和element是紧挨在一起的两个entry， element在前，score在后
* score越小越接近队首，score越大越接近队尾，按照score值升序排列

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-215.png)

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/04-redis-20230512-216.png)



#  Hash

> Hash结构与Redis中的Zset非常类似：

* 都是键值存储
* 都需求根据键获取值
* 键必须唯一

> 区别如下：

* zset的键是member，值是score；hash的键和值都是任意值
* zset要根据score排序；hash则无需排序

> 底层实现方式：压缩列表ziplist 或者 字典dict

当Hash中数据项比较少的情况下，Hash底层才⽤压缩列表ziplist进⾏存储数据，随着数据的增加，底层的ziplist就可能会转成dict，具体配置如下：

```properties
hash-max-ziplist-entries 512

hash-max-ziplist-value 64
```

当满足上面两个条件其中之⼀的时候，Redis就使⽤dict字典来实现hash。

> Redis的hash之所以这样设计，是因为当ziplist变得很⼤的时候，它有如下几个缺点：

* 每次插⼊或修改引发的realloc操作会有更⼤的概率造成内存拷贝，从而降低性能。
* ⼀旦发生内存拷贝，内存拷贝的成本也相应增加，因为要拷贝更⼤的⼀块数据。
* 当ziplist数据项过多的时候，在它上⾯查找指定的数据项就会性能变得很低，因为ziplist上的查找需要进行遍历。

总之，ziplist本来就设计为各个数据项挨在⼀起组成连续的内存空间，这种结构并不擅长做修改操作。⼀旦数据发⽣改动，就会引发内存realloc，可能导致内存拷贝。





