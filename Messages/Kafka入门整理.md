<!-- TOC -->

- [Kafka的设计动机](#kafka的设计动机)
- [一张图理解Kafka](#一张图理解kafka)
  - [**Kafka提供的4个核心的API:**](#kafka提供的4个核心的api)
  - [Kafka的几个重要概念（我一开始学习Kafka是完全是懵逼状态的，这几个概念一定要搞清楚）](#kafka的几个重要概念我一开始学习kafka是完全是懵逼状态的这几个概念一定要搞清楚)
    - [Producer(生产者)和Consumer(消费者)](#producer生产者和consumer消费者)
    - [Broker(代理)和Cluster(集群)](#broker代理和cluster集群)
    - [Topic(主题)和Partition(分区)](#topic主题和partition分区)
    - [日志（commit log）和偏移量(offset)](#日志commit-log和偏移量offset)
    - [消费组（Consumer Group）](#消费组consumer-group)
- [Kafka的设计思想](#kafka的设计思想)
  - [生产者（Producer）](#生产者producer)
  - [消费者（Consumer）](#消费者consumer)
  - [消息的持久化](#消息的持久化)
  - [Kafka底层存储结构](#kafka底层存储结构)
- [Zookeeper 在 Kafka 中的作用](#zookeeper-在-kafka-中的作用)

<!-- /TOC -->

# Kafka的设计动机
往往在系统中选用一种技术的时候，要想最快的知道要不要选择这项技术，其中比较直接有效的一种判断方法就是：**了解这项技术的设计动机**。
因为设计动机是最能直接有效的反应这项技术能够解决的问题，那么就刚刚好，我们就寻找到了使用哪种技术方案的答案。

**设计的 Kafka 能够作为一个统一的平台来处理大公司可能拥有的所有实时数据馈送** 这样就必须得满足以下几个特点：  
- 必须具有高吞吐量来支持高容量事件流
- 需要能够正常处理大量的数据积压，以便能够支持来自离线系统的周期性数据加载。
- 必须处理低延迟分发，来处理更传统的消息传递用例
- 希望支持对这些馈送进行分区，分布式，以及实时处理来创建新的分发馈送等特性
- 在数据流被推送到其他数据系统进行服务的情况下，要求系统在出现机器故障时必须能够保证容错

为支持这些使用场景导致设计了一些独特的元素，使得 Kafka 相比传统的消息系统更像是数据库日志。

# 一张图理解Kafka
![Kafka是一个分布式流处理平台](/ASSET/一张图理解Kafka整体框架.jpg)
这张图是Kafka官网首页上的图，这个图基本上就将Kafka的4大核心功能和应用就描述的很清楚了。
> Apache Kafka® 是 一个分布式流处理平台.

首先Kafka是一个流式处理平台，我们平常接触到的使用流式处理数据的比如Java8中引入的Stream API，使用过这些API的开发者们应该能够感受到Kafka的流处理是什么样子的吧。

另外，分布式系统的共同的特点：集群、可用性、数据一致性、容错性等。

## **Kafka提供的4个核心的API:**  
- **Producer API:** 允许一个应用程序发布一串流式的数据到一个或者多个Kafka topic。
- **Consumer API:**  允许一个应用程序订阅一个或多个 topic ，并且对发布给他们的流式数据进行处理。
- **Connector API:**  允许一个应用程序作为一个流处理器，消费一个或者多个topic产生的输入流，然后生产一个输出流到一个或多个topic中去，在输入输出流中进行有效的转换。
- **Stream API:** 许构建并运行可重用的生产者或者消费者，将Kafka topics连接到已存在的应用程序或者数据系统。比如，连接到一个关系型数据库，捕捉表（table）的所有变更内容。

## Kafka的几个重要概念（我一开始学习Kafka是完全是懵逼状态的，这几个概念一定要搞清楚）
**Kafka拓扑结构**
![Kafka的拓扑结构](/ASSET/Kafka的拓扑结构.jpg)

**Kafka的几个重要概念图示如下：**
![Kafka概念图示](/ASSET/Kafka概念图示.png)

### Producer(生产者)和Consumer(消费者)
`Producer`和`Consumer`作为Kafka的两个重要的客户端。生产者是来向Kafka的Topic中发送消息，消费者是从Kafka的Topic中订阅消息。
图中只是画了数据的单向流向，在实际的开发实践中生产者还可以作为消费者，消费者还可以作为生产者。

`Kafka Stream`和`Kafka Connector`是这两者的高级使用，底层实际上还是使用的Producer和Consumer的发布和订阅，只是Stream和Connector将其又封装了一层而已。

### Broker(代理)和Cluster(集群)
Kafka作为分布式的发布订阅平台，大多数的情况下会使用集群的方式来部署使用，当然单节点部署也是没有问题的，只是可用性就远远不及集群，所以有条件的环境下生产环境尽量使用集群的方式来进行部署。集群环境就是又多台主机或者云主机组合而成的，而集群中的每一节点（主机）称之为Broker。

### Topic(主题)和Partition(分区)
Kafka将消息按照主题（Topic）分类，每个主题都对应一个消息队列。由于Kafka设计的初衷中重要的一个目标：*处理海量数据的发布/订阅*，这样就需要面对海量数据的时候，提高系统的吞吐量，如果一个主题只有一个消息队列，那肯定是会遇到性能瓶颈的，所以在设计中设计了分区（Partition）的概念，一个Topic可以存在多个Partition，当数据量增大时，可以对应的增加分区的数量，消息发布时通过负载均衡发送到不同的分区，不同的分区可以使用不同的消费者来处理数据，这样就可以提高消息的发布订阅效率，提高了系统的吞吐。所以Partition是Topic的消息队列的扩展。

**Kafka中的Partition是一个并行的概念**

### 日志（commit log）和偏移量(offset)
对于每一个topic， Kafka集群都会维持一个分区日志，如下所示：
![Kafka的Topic的分区日志](/ASSET/Kafka的Topic的分区日志.png)
上面的Partition讲过就是Topic的分区，每个分区会又单独的日志记录。每个分区都是有序且顺序不可变的记录集，并且不断地追加到结构化的commit log文件。分区中的每一个记录都会分配一个id号来表示顺序，我们称之为offset，offset用来唯一的标识分区中每一条记录。

### 消费组（Consumer Group）
传统的消息系统有两个模块: 队列 和 发布-订阅。 在队列中，消费者池从server读取数据，每条记录被池子中的一个消费者消费; 在发布订阅中，记录被广播到所有的消费者。两者均有优缺点。队列的优点在于它允许你将处理数据的过程分给多个消费者实例，使你可以扩展处理过程。 不好的是，队列不是多订阅者模式的—一旦一个进程读取了数据，数据就会被丢弃。 而发布-订阅系统允许你广播数据到多个进程，但是无法进行扩展处理，因为每条消息都会发送给所有的订阅者。

消费组在Kafka有两层概念。在队列中，消费组允许你将处理过程分发给一系列进程(消费组中的成员)。 在发布订阅中，Kafka允许你将消息广播给多个消费组。

**总结**： 消费组就是多个消费者组成的一个组，（1）消息发布者可以将消息发布到队列，多个不同的消费组都可以订阅消费；（2）每个组中只允许有其中一个消费者消费消息

# Kafka的设计思想

## 生产者（Producer）
1. 负载均衡
生产者直接发送数据到主分区的服务器上，不需要经过任何中间路由。 为了让生产者实现这个功能，所有的 kafka 服务器节点都能响应这样的元数据请求： 哪些服务器是活着的，主题的哪些分区是主分区，分配在哪个服务器上，这样生产者就能适当地直接发送它的请求到服务器上。

客户端控制消息发送数据到哪个分区，这个可以实现随机的负载均衡方式,或者使用一些特定语义的分区函数。 我们有提供特定分区的接口让用于根据指定的键值进行hash分区(当然也有选项可以重写分区函数)，例如，如果使用用户ID作为key，则用户相关的所有数据都会被分发到同一个分区上。 这允许消费者在消费数据时做一些特定的本地化处理。这样的分区风格经常被设计用于一些本地处理比较敏感的消费者。

2. 异步发送（批量发送）  
批处理是提升性能的一个主要驱动，为了允许批量处理，kafka 生产者会尝试在内存中汇总数据，并用一次请求批次提交信息。 批处理，不仅仅可以配置指定的消息数量，也可以指定等待特定的延迟时间(如64k 或10ms)，这允许汇总更多的数据后再发送，在服务器端也会减少更多的IO操作。 该缓冲是可配置的，并给出了一个机制，通过权衡少量额外的延迟时间获取更好的吞吐量。

## 消费者（Consumer）
Kafka consumer通过向 broker 发出一个“fetch”请求来获取它想要消费的 partition。consumer 的每个请求都在 log 中指定了对应的 offset，并接收从该位置开始的一大块数据。
当需要提高系统的吞吐时可以适当的增加Topic的Partition，同时消费组的消费者需要适当的增加，在条件允许的情况下，最佳的配置是**Partition数量=消费者组内消费者的数量**

Topic的分区和消费者的关系如下图：
![Kafka的Topic和消费者组的关系](/ASSET/Kafka的Topic和消费者组的关系.png)

## 消息的持久化
Kafka 对消息的存储和缓存严重依赖于文件系统。人们对于“磁盘速度慢”的普遍印象，使得人们对于持久化的架构能够提供强有力的性能产生怀疑。事实上，磁盘的速度比人们预期的要慢的多，也快得多，这取决于人们使用磁盘的方式。而且设计合理的磁盘结构通常可以和网络一样快。

**相比于维护尽可能多的 in-memory cache，并且在空间不足的时候匆忙将数据 flush 到文件系统，我们把这个过程倒过来。所有数据一开始就被写入到文件系统的持久化日志中，而不用在 cache 空间不足的时候 flush 到磁盘。实际上，这表明数据被转移到了内核的 pagecache 中。**

对于使用这样的进行消息持久化的方式，大概两点原因：  
1. 现代磁盘的性能优化，顺序写的速度甚至可以和内存性能相媲美。与其加入内存来共同维护和持久化数据到磁盘，提高性能的同时，也提高了系统的复杂度，还不如直接将数据写入磁盘。
2. 因为设计的Partition，commit log是append的方式，这样不仅保证了顺序性，并且还能保证磁盘的顺序写。所以使用磁盘写性能不会影响太大。

对于Kafka消息的持久化为什么选择直接将数据写入文件系统的持久化日志可以参考官网中文文档的解释：[参考文章](http://kafka.apachecn.org/documentation.html#design_filesystem)

## Kafka底层存储结构

默认情况下，kafka将数据存储在`/tmp/kafka-logs/`的目录下，每个Topic按照Partition的数量生成多个不同的目录，如`mytopic-0`, `mytopic-1`等。
而每个分区的数据又是分段（Segment）进行存储的，以该文件第一条数据的offset来命名。每个Segment文件的大小是可以设置的，默认是超过7天或者超过500M,
则会将消息保存到新的Segment文件中。
```
|-- mytopic-0
  |-- 00000000000000000000.index
  |-- 00000000000000000000.log
  |-- 00000000000000000000.timeindex
  |-- 00000000000000015467.index
  |-- 00000000000000015467.log
  |-- 00000000000000015467.timeindex
  |-- 00000000000000000050.snapshot
  |-- leader-epoch-checkpoint
|-- mytopic-1
```

1. `leader-epoch-checkpoint`文件:   
  对应整个log目录

2. `*.log`文件  
    实际存储data的Segment文件，文件名是Segment的baseOffset（起始offset）。通过配置文件中的log.segment.bytes选项，
可以定义当文件多大时roll out一个出新的Segment。

3. `*.index`文件  
    文件名是Segment的baseOffset，存储了messageRelativeOffset->filePosition的key-value list，通过配置，
可以定义data中隔多少bytes就打一次index（bytes都是完整的记录包，所以不一定是准确的间隔），默认是4096 bytes。kafka partition中的offset都称为messageOffset, 
而index文件中的messageRelativeOffset，则是通过 (messasgeOffset - Segment的baseOffset) 得到的，而filePosition就是message存储在log文件中的字节偏移量。

4. `*.timeindex`文件   
文件名是Segment的baseOffset，存储了每个记录包中最大的timestamp -> 每个记录包中最大的messageRelativeOffset（存储时也是减去Segment的baseOffset）的key-value list。

5. `*.snapshot`文件  
  记录了producer的事务信息。

以以上的存储举例。`00.log`存储的消息是`0~15466`,`15467.log`存储的消息是`15467~下一个`。

**Partition中的每条Message由offset来表示它在这个partition中的偏移量，这个offset不是该Message在partition数据文件中的实际存储位置，而是逻辑上一个值，它唯一确定了partition中的一条Message。因此，可以认为offset是partition中Message的id。partition中的每条Message包含了以下三个属性：offset / MessageSize / data。**

![Kafka底层存储结构0](/ASSET/Kafka底层存储结构0.png)

index和log的关系及内部数据信息：
![Kafka底层存储结构](/ASSET/Kafka底层存储结构.png)

**index、log这些文件总是成对存在的，index文件作为log数据查找的索引文件，来提高查询数据的效率。index文件中的每条数据有2个值, offset:0（消息偏移量/ID） position: 0（对应消息存储的物理位置），但是offset的值并不是连续的，这是因为 Kafka 采取稀疏索引存储的方式，每隔一定字节的数据建立一条索引，它减少了索引文件大小，使得能够把 index 映射到内存，降低了查询时的磁盘 IO 开销，同时也并没有给查询带来太多的时间消耗。**

**log文件中offset的值是顺序加1的，position的值也不是顺序加1的。这是因为position的位置是根据消息的字节大小，计算出在磁盘的位置。根据这种规律，可以知道，虽然position不是顺序加1，但是position的值始终是随着offset增大而增大**

获取消息过程举例：比如，按照上图中的数据，需要查找offset为7的消息  
1. 使用二分法查找到从哪个个LogSegment中查找
2. 打开该段中的index文件，找到小于或者等于7的offset的数据，那就找到`<4，476>`这条数据，然后使用476找到log中位置是476的消息，并对比offset是否为7，不是的话就顺序向后找，直到找到offset是7的消息。

# Zookeeper 在 Kafka 中的作用
1. Broker注册

Broker是分布式部署并且相互之间相互独立，但是需要有一个注册系统能够将整个集群中的Broker管理起来，此时就使用到了Zookeeper。在Zookeeper上会有一个专门用来进行Broker服务器列表记录的节点：**/brokers/ids**

每个Broker在启动时，都会到Zookeeper上进行注册，即到/brokers/ids下创建属于自己的节点，如/brokers/ids/[0...N]。

Kafka使用了全局唯一的数字来指代每个Broker服务器，不同的Broker必须使用不同的Broker ID进行注册，创建完节点后，每个Broker就会将自己的IP地址和端口信息记录到该节点中去。其中，Broker创建的节点类型是临时节点，一旦Broker宕机，则对应的临时节点也会被自动删除。

2. Topic注册

在Kafka中，同一个Topic的消息会被分成多个分区并将其分布在多个Broker上，这些分区信息及与Broker的对应关系也都是由Zookeeper在维护，由专门的节点来记录，如：**/borkers/topics**

Kafka中每个Topic都会以/brokers/topics/[topic]的形式被记录，如/brokers/topics/login和/brokers/topics/search等。Broker服务器启动后，会到对应Topic节点（/brokers/topics）上注册自己的Broker ID并写入针对该Topic的分区总数，如/brokers/topics/login/3->2，这个节点表示Broker ID为3的一个Broker服务器，对于"login"这个Topic的消息，提供了2个分区进行消息存储，同样，这个分区节点也是临时节点。

3. 生产者负载均衡

由于同一个Topic消息会被分区并将其分布在多个Broker上，因此，生产者需要将消息合理地发送到这些分布式的Broker上，那么如何实现生产者的负载均衡，Kafka支持传统的四层负载均衡，也支持Zookeeper方式实现负载均衡。

(1) 四层负载均衡，根据生产者的IP地址和端口来为其确定一个相关联的Broker。通常，一个生产者只会对应单个Broker，然后该生产者产生的消息都发往该Broker。这种方式逻辑简单，每个生产者不需要同其他系统建立额外的TCP连接，只需要和Broker维护单个TCP连接即可。但是，其无法做到真正的负载均衡，因为实际系统中的每个生产者产生的消息量及每个Broker的消息存储量都是不一样的，如果有些生产者产生的消息远多于其他生产者的话，那么会导致不同的Broker接收到的消息总数差异巨大，同时，生产者也无法实时感知到Broker的新增和删除。

(2) 使用Zookeeper进行负载均衡，由于每个Broker启动时，都会完成Broker注册过程，生产者会通过该节点的变化来动态地感知到Broker服务器列表的变更，这样就可以实现动态的负载均衡机制。

4. 消费者负载均衡

与生产者类似，Kafka中的消费者同样需要进行负载均衡来实现多个消费者合理地从对应的Broker服务器上接收消息，每个消费者分组包含若干消费者，每条消息都只会发送给分组中的一个消费者，不同的消费者分组消费自己特定的Topic下面的消息，互不干扰。

5. 分区 与 消费者 的关系

消费组 (Consumer Group)：consumer group 下有多个 Consumer（消费者）。   

对于每个消费者组 (Consumer Group)，Kafka都会为其分配一个全局唯一的Group ID，Group 内部的所有消费者共享该 ID。订阅的topic下的每个分区只能分配给某个 group 下的一个consumer(当然该分区还可以被分配给其他group)。同时，Kafka为每个消费者分配一个Consumer ID，通常采用"Hostname:UUID"形式表示。

在Kafka中，规定了每个消息分区 只能被同组的一个消费者进行消费，因此，需要在 Zookeeper 上记录 消息分区 与 Consumer 之间的关系，每个消费者一旦确定了对一个消息分区的消费权力，需要将其Consumer ID 写入到 Zookeeper 对应消息分区的临时节点上，例如：/consumers/[group_id]/owners/[topic]/[broker_id-partition_id]

其中，[broker_id-partition_id]就是一个 消息分区 的标识，节点内容就是该 消息分区 上 消费者的Consumer ID。

6. 消息 消费进度Offset 记录

在消费者对指定消息分区进行消息消费的过程中，需要定时地将分区消息的消费进度Offset记录到Zookeeper上，以便在该消费者进行重启或者其他消费者重新接管该消息分区的消息消费后，能够从之前的进度开始继续进行消息消费。Offset在Zookeeper中由一个专门节点进行记录，其节点路径为: /consumers/[group_id]/offsets/[topic]/[broker_id-partition_id]

节点内容就是Offset的值。

7. 消费者注册

消费者服务器在初始化启动时加入消费者分组的步骤如下

**注册到消费者分组。** 每个消费者服务器启动时，都会到Zookeeper的指定节点下创建一个属于自己的消费者节点，例如/consumers/[group_id]/ids/[consumer_id]，完成节点创建后，消费者就会将自己订阅的Topic信息写入该临时节点。

**对 消费者分组 中的 消费者 的变化注册监听。** 每个 消费者 都需要关注所属 消费者分组 中其他消费者服务器的变化情况，即对/consumers/[group_id]/ids节点注册子节点变化的Watcher监听，一旦发现消费者新增或减少，就触发消费者的负载均衡。

**对Broker服务器变化注册监听。** 消费者需要对/broker/ids/[0-N]中的节点进行监听，如果发现Broker服务器列表发生变化，那么就根据具体情况来决定是否需要进行消费者负载均衡。

**进行消费者负载均衡。** 为了让同一个Topic下不同分区的消息尽量均衡地被多个 消费者 消费而进行 消费者 与 消息 分区分配的过程，通常，对于一个消费者分组，如果组内的消费者服务器发生变更或Broker服务器发生变更，会发出消费者负载均衡。