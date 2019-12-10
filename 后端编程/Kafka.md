# Kafka常用命令
```
./bin/kafka-server-start.sh config/server.properties &    // 后台运行的方式启动kafka
./bin/kafka-topics.sh --zookeeper 192.168.121.128:2181 --partitions 1 --replication-factor 1 --create --topic mytopic   // 创建Topic
./bin/kafka-console-producer.sh --broker-list 192.168.121.128:9092 --topic mytopic  // 生产者客户端
./bin/kafka-console-consumer.sh --bootstrap-server 192.168.121.128:9092  --from-beginning --topic mytopic // 消费者控制台
./bin/kafka-topics.sh --list --zookeeper 192.168.121.128:2181   // 查看所有的topic
./bin/kafka-consumer-groups.sh --bootstrap-server 192.168.121.128:9092 --list group // 查看所有的消费者组
./bin/kafka-consumer-groups.sh --bootstrap-server 192.168.121.128:9092 --describe --group qmscustview // 查看某个消费者组的信息
./bin/kafka-topics.sh -zookeeper 192.168.121.128:2181 -describe -topic mytopic    // 查看主题的信息
./bin/kafka-topics.sh --zookeeper 192.168.121.128:2181 --delete --topic mytopic   // 删除topic
./bin/kafka-topics.sh --zookeeper 192.168.121.128:2181 --alter --topic mytopic --partitions 2   // 修改topic的partition数量
./bin/kafka-console-producer.sh --broker-list 192.168.121.128:9092 --topic mytopic < <filepath>   // 将指定文件的内容作为消息发布到指定的Topic
./bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list 192.168.121.128:9092 --topic mytopic --time -1    // 查看Topic中的总数量
```

## 删除topic

1. step1:  `./bin/kafka-topics.sh --zookeeper 192.168.121.128:2181 --delete --topic mytopic` 执行命令删除topic  
2. step2: 删除数据目录。kafka的topic数据就是在配置文件中`log.dirs`指定的目录存储，如果想要赶紧的删除topic及其数据，在删除topic之后，还需要到指定的目录删除数据。  
3. 在zookeeper中查看kafka相关的数据，使用zkCli客户端连接到zookeeper  
- 未被标记为`marked for deletion`: 执行`ls /brokers/topics`查看所有的topic，找到要删除的topic，执行`rmr /brokers/topics/<topic_name>`删除  
- 已被标记为`marked for deletion`：执行`ls /admin/delete_topics`查看所有的topic，找到要删除的topic，执行`rmr /admin/delete_topics/<topic_name>`删除
```
ls /brokers/topics    // 查看所有的topic
get /brokers/topics/<topic_name>    // 查看topic的信息
rmr /brokers/topics/<topic_name>    // 删除topic的数据
```

# kafka

## 4种API：  
- `Producer API`: 发布消息  
- `Consumer API`: 消费消息  
- `Streams API`: 流处理器，可以将消息流处理为另外的输出流  
- `Connector API`: 可以将kafka的Topic连接到一个已有的程序或者数据库，比如连接到数据库进行变化数据捕捉。
API中提供了SourceConnectors、SinkConnectors可以根据自己的业务自行开发Connector

## kafka消息顺序问题的思考  

kafka的设计就是为了满足**大数据、高吞吐**的场景。往往使用增加partition的数量来提高他得吞吐能力，这也是kafka的关键的特性。
但是在Topic级别是不能保证分区的消息的顺序的，只能保证1个分区内的消息的顺序。如果要保证不同分区的数据的顺序，通常的一种做法：
发布消息时，将需要保证先后顺序的消息发布到同一partition。如果需要对所有的分区的所有数据都要严格的保证消息的顺序，那在kafka的
层面是不能解决的，除非partition的数量是1。那遇到这种情况的时候就要思考选择kafka是否是符合自己当前的业务场景的。也就是面临的问题：
（1）对顺序要求不严格的场景消息中间件的选择（2） 对顺序要求严格的场景消息中间件的选择

