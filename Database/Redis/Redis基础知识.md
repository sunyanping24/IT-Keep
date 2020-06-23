<!-- TOC -->

- [缓存穿透、缓存击穿、缓存雪崩](#缓存穿透缓存击穿缓存雪崩)
- [Redis的发布订阅模式](#redis的发布订阅模式)
- [Redis为什么这么快](#redis为什么这么快)
- [Redis的集群模式](#redis的集群模式)
- [Redis的持久化机制](#redis的持久化机制)
  - [RDB 持久化(快照持久化)](#rdb-持久化快照持久化)
  - [AOF 持久化](#aof-持久化)
  - [Redis 4.0 对持久化机制的优化](#redis-40-对持久化机制的优化)
- [Redis hotKey的检测](#redis-hotkey的检测)
- [Redis的数据结构及常用命令](#redis的数据结构及常用命令)
  - [String 字符串](#string-字符串)
  - [List 列表](#list-列表)
  - [Hash 哈希](#hash-哈希)
  - [SET 集合](#set-集合)
  - [ZSET 有序集合](#zset-有序集合)
- [Redis key的过期时间](#redis-key的过期时间)
  - [key的过期策略](#key的过期策略)
- [Redis性能监控](#redis性能监控)
  - [几个常用的性能指标整理](#几个常用的性能指标整理)
  - [性能测试工具](#性能测试工具)

<!-- /TOC -->

# 缓存穿透、缓存击穿、缓存雪崩
在使用缓存的时候这几个就是比较常见的概念了  
**缓存穿透**： 需要请求的数据刚好没有做缓存这一层，而该数据却需要被频繁的查询，那就会造成数据库的压力，严重的情况下可能会导致数据库崩溃。这种情况出现的原因一般比如恶性攻击，故意使用一些可能程序未考虑的参数情况等。  
**缓存击穿**： 需要请求的数据已经做了缓存这一层，但是在需要对该数据进行查询的时候，该数据过期了，导致瞬间大量的请求转向数据库，导致数据库崩溃。  
**缓存雪崩**： 缓存数据大面积失效，大量的请求转向数据库，导致数据库崩溃。
  
缓存雪崩的问题处理方案：  
1. 设计程序时首先需要考虑到这种问题的发生的可能性。合理的设置数据的过期时间。并且在除了这一层缓存外还需要设计限流的方案，以防该情况的发生，导致数据库崩掉。比如使用hystrix限流。  
2. 如果考虑到缓存服务的稳定性导致的缓存数据失效的话，那就得保证集群的可用性。
![缓存雪崩处理方案](http://sunyanping.gitee.io/it-keep/ASSET/缓存雪崩处理方案.jpg)

缓存穿透的问题处理方案：  
1. 保证程序的设计是合理的，有大并发需要做缓存的地方都已经做了缓存。  
2. 做好请求参数的过滤处理，防止有恶行的攻击行为。直接在请求到缓存之前就已经被拒绝。

# Redis的发布订阅模式

项目中的整合可以参考：[自己的keep-server](https://gitee.com/sunyanping/keep-server/tree/master/keep-spring-data/keep-spring-data-redis)

虽然Redis也提供了发布订阅消息的功能，但是者毕竟不是Redis的主要功能，如何和Kafka、Rabbitmq这些专门的消息中间件相比的话，Redis的发布订阅功能就非常单一了，使用的场景在实际的生产中可能范围更少。
1. 发布/订阅和数据库没关系，比如连接是0的数据库发布消息，连接是1的数据只要订阅了该主题的消息就可以收到消息。
2. 主要适用一些吞吐不高、持久性要求不高、可以接受消息缺失、数据量不大的场景。比如系统刚好使用了redis，并且redis发布订阅可以满足要求，那就可以不用在引入其他的中间件增加系统的复杂度。

# Redis为什么这么快
- 完全基于内存，绝大部分请求是纯粹的内存操作，非常快速；
- 数据结构简单，对数据操作也简单，Redis 中的数据结构是专门进行设计的；
- 采用单线程，避免了不必要的上下文切换和竞争条件，也不存在多进程或者多线程导致的切换而消耗 CPU，不用去考虑各种锁的问题，不存在加锁释放锁操作，没有因为可能出现死锁而导致的性能消耗；
- 使用多路I/O复用模型，非阻塞 IO。只有单个线程，通过跟踪每个I/O流的状态，来管理多个I/O流；
- 我们的 redis-client 在操作的时候，会产生具有不同事件类型的 socket。在服务端，有一段 I/0 多路复用程序，将其置入队列之中。然后，文件事件分派器，依次去队列中取，转发到不同的事件处理器中。

# Redis的集群模式
- 主从复制模式
- 哨兵（Sentinel）模式
- Cluster 集群模式

# Redis的持久化机制
Redis是内存型数据库，数据是存储在内存中的，如果数据不持久化在磁盘中，当Redis服务出现宕机、服务器故障重启问题时内存中的数据就会丢失。为了防止这种情况的发生，Redis提供了持久化功能。包含**RDB**和**AOF**两种持久化方式，默认开启的是RDB的方式。

## RDB 持久化(快照持久化)
RDB 持久化：将某个时间点的所有数据都存放到硬盘上。

可以将快照复制到其它服务器从而创建具有相同数据的服务器副本。如果系统发生故障，将会丢失最后一次创建快照之后的数据。如果数据量很大，保存快照的时间会很长。

快照持久化是Redis默认采用的持久化方式，在redis.conf配置文件中默认有此下配置：
```
save 900 1  # 在900秒(15分钟)之后，如果至少有1个key发生变化，Redis就会自动触发BGSAVE命令创建快照。
save 300 10  #在300秒(5分钟)之后，如果至少有10个key发生变化，Redis就会自动触发BGSAVE命令创建快照。          
save 60 10000  #在60秒(1分钟)之后，如果至少有10000个key发生变化，Redis就会自动触发BGSAVE命令创建快照。
```

## AOF 持久化
AOF 持久化：将写命令添加到 AOF 文件（Append Only File）的末尾。

与快照持久化相比，AOF持久化 的实时性更好，因此已成为主流的持久化方案。默认情况下Redis没有开启AOF（append only file）方式的持久化，可以通过appendonly参数开启：`appendonly yes`

开启AOF持久化后每执行一条会更改Redis中的数据的命令，Redis就会将该命令写入硬盘中的AOF文件。AOF文件的保存位置和RDB文件的位置相同，都是通过dir参数设置的，默认的文件名是appendonly.aof。

使用 AOF 持久化需要设置同步选项，从而确定写命令同步到磁盘文件上的时机。这是因为对文件进行写入并不会马上将内容同步到磁盘上，而是先存储到缓冲区，然后由操作系统决定什么时候同步到磁盘。在Redis的配置文件中存在三种同步方式
```
appendfsync always  # 每个写命令都同步，这样会严重降低Redis的速度
appendfsync everysec # 每秒同步一次
appendfsync no  # 让操作系统来决定何时同步
```

## Redis 4.0 对持久化机制的优化
Redis 4.0 开始支持 RDB 和 AOF 的混合持久化（默认关闭，可以通过配置项 aof-use-rdb-preamble 开启）。

如果把混合持久化打开，AOF 重写的时候就直接把 RDB 的内容写到 AOF 文件开头。这样做的好处是可以结合 RDB 和 AOF 的优点, 快速加载同时避免丢失过多的数据。当然缺点也是有的， AOF 里面的 RDB 部分就是压缩格式不再是 AOF 格式，可读性较差。

# Redis hotKey的检测
redis 4.0.3提供了redis-cli的热点key发现功能，执行redis-cli时加上–hotkeys选项即可，示例如下：
```
# Scanning the entire keyspace to find hot keys as well as
# average sizes per key type.  You can use -i 0.1 to sleep 0.1 sec
# per 100 SCAN commands (not usually needed).
 
[00.00%] Hot key 'counter:000000000002' found so far with counter 87
[00.00%] Hot key 'key:000000000001' found so far with counter 254
[00.00%] Hot key 'mylist' found so far with counter 107
[00.00%] Hot key 'key:000000000000' found so far with counter 254
[45.45%] Hot key 'counter:000000000001' found so far with counter 87
[45.45%] Hot key 'key:000000000002' found so far with counter 254
[45.45%] Hot key 'myset' found so far with counter 64
[45.45%] Hot key 'counter:000000000000' found so far with counter 93
 
-------- summary -------
 
Sampled 22 keys in the keyspace!
hot key found with counter: 254    keyname: key:000000000001
hot key found with counter: 254    keyname: key:000000000000
hot key found with counter: 254    keyname: key:000000000002
hot key found with counter: 107    keyname: mylist
hot key found with counter: 93    keyname: counter:000000000000
hot key found with counter: 87    keyname: counter:000000000002
hot key found with counter: 87    keyname: counter:000000000001
hot key found with counter: 64    keyname: myset
```

# Redis的数据结构及常用命令

- String 字符串     
String数据结构是简单的key-value类型，value其实不仅可以是String，也可以是数字。 常规key-value缓存应用。可以使用常规的统计。

- List 列表     
使用Lists结构，我们可以轻松地实现最新消息排行等功能。List的另一个应用就是消息队列，可以利用List的PUSH操作，将任务存在List中，然后工作线程再用POP操作将任务取出进行执行。Redis还提供了操作List中某一段的api，你可以直接查询，删除List中某一段的元素。 
Redis的list是每个子元素都是String类型的双向链表，可以通过push和pop操作从列表的头部或者尾部添加或者删除元素，这样List即可以作为栈，也可以作为队列。

- Hash 哈希       
Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。 
存储部分变更的数据，如用户信息等。

- SET 集合      
set就是一个集合，集合的概念就是一堆不重复值的组合。利用Redis提供的set数据结构，可以存储一些集合性的数据。set中的元素是没有顺序的。 


- ZSET 集合     
和set相比，sorted set增加了一个权重参数score，使得集合中的元素能够按score进行有序排列，比如一个存储全班同学成绩的sorted set，其集合value可以是同学的学号，而score就可以是其考试得分，这样在数据插入集合的时候，就已经进行了天然的排序。可以用sorted set来做带权重的队列，比如普通消息的score为1，重要消息的score为2，然后工作线程可以选择按score的倒序来获取工作任务。让重要的任务优先执行。


*这里总结的主要是使用redis-cli客户端访问数据的一些命令*
```
redis-cli -h 127.0.0.1 -p 6379
```

Redis常用命令: [http://redisdoc.com/index.html](http://redisdoc.com/index.html)
## String 字符串

```
SET <key> <value>   // 添加/更新key值为value
GET <key>   // 获取key的value
SET <key> <value> [EX seconds] [PX milliseconds] [NX|XX]    // EX-设置键过期时间(s)，PX-设置键过期时间(ms)，NX-只有键存在时才操作，XX-只有键不存在时才操作
MSET <key> <value> [key value ...]  // 批量设置
MGET <key> [<key>...]     // 批量获取
GETSET <key> <value>    // 将给定key的值设为value，并返回旧值
INCR <key>    // 将key存储的数字加1。值必须是数字类型，否则会报错；如果key不存在，则会初始化key的值为0
DECR <key>    // 将key存储的数字减1
INCRBY <key>  <increment>   // 将key的值增加指定的增量
DECRBY <key>  <decrement>   // 将key的值减去指定的增量
APPEND <key> <value>    // 如果 key 已经存在并且是一个字符串， APPEND 命令将 value 追加到 key 原来的值的末尾。
STRLEN <key>    // 返回 key 所储存的字符串值的长度。
```

## List 列表
List是一个字符串列表，Left/Right都可以插入元素。如果key不存在，创建列表；如果key存在，列表中添加内容；如果列表内容全部移除，则key自动删除。无论是头尾插入元素都非常快，中间插入元素效率略低。

```
LPUSH <key> <value ...>   // 先进后出，在列表头部插入元素
RPUSH <key> <value ...>     // 先进先出，在列表尾部插入元素
LRANGE <key> <index1> <index2>    // 根据索引范围获取列表中的元素，从头部到尾部0~n，从尾部到头部 -1 ~ -n，比如：LRANGE list01 0 -1可以取出所有的列表元素
LPOP <key>    // 移除并返回key列表的头元素
RPOP <key>    // 移除并返回key列表的尾部元素
LINDEX <key> <index>   // 根据索引取出元素，从头部到尾部0~n，从尾部到头部 -1 ~ -n
LLEN <key>    // 链表的长度
LREM <key> <n> <value>    // 删除key列表中n个value，从头向尾
LTRIM <key> <index>   // 根据索引删除指定元素
RPOPLPUSH <key1> <key2>   // key1列表的尾部元素出栈，并添加到key2列表的头部
LSET <key> <index> <value>    // 替换key列表index位置的value
```

## Hash 哈希
![Redis的Hash表](http://sunyanping.gitee.io/it-keep/ASSET/Redis的Hash表.png)
Redis hash 是一个 string 类型的 field 和 value 的映射表，hash 特别适合用于存储对象。

Hash的存储方式：    
键值key
字段1 字段值
字段2 字段值

Hash存储可以直接看作对象的存储，key为对象名，对象里有很多的属性(字段)。

```
HSET <key> <field> <value>    // key的hash表中添加域field并设置值value;若key不存在则创建;若field存在则更新
HGET <key> <field>    // 获取key的hash表中域field的value
HDEL <key> [<field> ...]    // 删除key的hash表中一个或多个域field
HEXISTS <key> <field>   // 判断key的hash表中field是否存在
HGETALL <key>   // 获取key的hash表中所有的域和值
HKEYS <key>   // 获取hash表中的所有域
HLEN <key>    // 获取hash表中域的数量
HMSET <key> <field1> <value1> <field2> <value2> ...   // 添加多个域和值到key的hash表中
HMGET <key> <field1> <field2> ...   // 获取hash表中多个域的值
HSETNX <key> <field> <value>  // 添加域和值到hash表中，当且仅当域 field 不存在
HVALS <key>   // 获取hash表key中所有的域的值
```

## SET 集合
包含字符串的无序集合，并且被包含的每个字符串都是独一无二、各不相同的	
```
SADD <key> <value> [value2]   // 向集合key中添加元素
SCARD <key>   // 获取集合元素的数量
SDIFF <key1> <key2>   // 获取key1中除去包含的key2中的元素的其他元素
SDIFFSTORE <targetKeySet> <key1> <key2>   // 将上一个命令得到的结果存到指定的集合中，若集合已存在则覆盖
SINTER <key1> [<key2>]    // 返回集合的交集
SINTERSTORE <targetKeySet> <key1> <key2>  // 将上一个命令得到的结果存到指定的集合中，若集合已存在则覆盖
SISMEMBER <key> <value>   // 判断集合中是否存在value
SMEMBERS <key>    // 返回集合中所有的元素
SMOVE <key1> <key2> <value> // 将元素value从key1集合移到key2集合
```

## ZSET 有序集合
有序集合和集合一样也是string类型元素的集合,且不允许重复的成员。
不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。
有序集合的成员是唯一的,但分数(score)却可以重复。
 
```
ZADD <key1> <value> [value...]    // 向key集合中添加一个或多个元素
ZCARD <key>   // 获取集合中元素的数量
ZCOUNT <key> min max    // 计算在有序集合中指定区间分数的成员数
ZINCRBY key <increment> <value> // 有序集合中对指定成员的分数加上增量 increment
ZINTERSTORE <key1> <numkeys> <key> [key …]    // 计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中,numkeys代表后面有几个key
ZLEXCOUNT <key> min max     // 在有序集合中计算指定字典区间内成员数量
ZRANGE <key> <index1> <index2>    // 获取key集合中下标指定范围内的元素
ZRANGEBYSCORE <key> min max   // 获取分数区间内key集合中的元素
ZREM <key> <value>    // 删除集合key中value元素，存在便返回1，不存在则返回0
```

# Redis key的过期时间
key的过期时间可以通过命令`ttl key`来进行查看，一般创建key时不指定ttl，则ttl的值是-1，代表的是永远不会过期。

```
ttl <key>     // 查看key的有限期
expire <key>  <time>    // 设置key的过期时间
persist <key>   // 清除key的过期时间
```

## key的过期策略

**Redis使用懒惰删除+定时删除的方式处理过期的key。**     

1. 懒惰删除

所谓懒惰删除就是在客户端访问该key的时候，redis会对key的过期时间进行检查，如果过期了就立即删除。

这种方式看似很完美，在访问的时候检查key的过期时间，不会占用太多的额外CPU资源。但是如果一个key已经过期了，如果长时间没有被访问，那么这个key就会一直存留在内存之中，严重消耗了内存资源。

2. 定时删除

定期删除的原理是，Redis会将所有设置了过期时间的key放入一个字典中，然后每隔一段时间从字典中随机一些key检查过期时间并删除已过期的key。

Redis默认每秒进行10次过期扫描：
（1）从过期字典中随机20个key
（2）删除这20个key中已过期的
（3）如果超过25%的key过期，则重复第一步

同时，为了保证不出现循环过度的情况，Redis还设置了扫描的时间上限，默认不会超过25ms。

# Redis性能监控

## 几个常用的性能指标整理
Redis的性能指标的数据可以通过Redis自带的**redis-cli**来监控，`./redis-cli info`命令即可展示所有的redis相关的信息，能够从这些数据中分析得出想要的性能数据。

1. **性能指标**     
- **latency**：Redis响应一个请求的时间
- **instantaneous_ops_per_sec**：平均每秒处理请求总数
- **hi rate(calculated)**：缓存命中率（计算出来的）

2. **内存指标**     
- **used_memory**：已使用内存
- **mem_fragmentation_ratio**：内存碎片率
- **evicted_keys**：由于最大内存限制被移除的key的数量
- **blocked_clients**：由于BLPOP,BRPOP,or BRPOPLPUSH而备阻塞的客户端

3. **基本活动指标**   
- **connected_clients**：客户端连接数
- **conected_laves**：slave数量
- **master_last_io_seconds_ago**：最近一次主从交互之后的秒数
- **keyspace**：数据库中的key值总数

4. **持久性指标**   
- **rdb_last_save_time**：最后一次持久化保存磁盘的时间戳
- **rdb_changes_sice_last_save**：自最后一次持久化以来数据库的更改数

5. **错误指标**     
- **rejected_connections**：由于达到maxclient限制而被拒绝的连接数
- **keyspace_misses**：key值查找失败(没有命中)次数
- **master_link_down_since_seconds**：主从断开的持续时间（以秒为单位)

## 性能测试工具
使用redis自带的性能测试工具：`./redis-benchmark -c 100 -n 5000`（100个连接，5000次请求对应的性能）

