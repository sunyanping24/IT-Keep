<!-- TOC -->

- [Redis的发布订阅模式](#redis的发布订阅模式)

<!-- /TOC -->

# Redis的发布订阅模式

项目中的整合可以参考：[自己的keep-server](https://gitee.com/sunyanping/keep-server/tree/master/keep-spring-data/keep-spring-data-redis)

虽然Redis也提供了发布订阅消息的功能，但是者毕竟不是Redis的主要功能，如何和Kafka、Rabbitmq这些专门的消息中间件相比的话，Redis的发布订阅功能就非常单一了，使用的场景在实际的生产中可能范围更少。
1. 发布/订阅和数据库没关系，比如连接是0的数据库发布消息，连接是1的数据只要订阅了该主题的消息就可以收到消息。
2. 主要适用一些吞吐不高、持久性要求不高、可以接受消息缺失、数据量不大的场景。比如系统刚好使用了redis，并且redis发布订阅可以满足要求，那就可以不用在引入其他的中间件增加系统的复杂度。
