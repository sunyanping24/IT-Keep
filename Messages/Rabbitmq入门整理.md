<!-- TOC -->

- [Rabbitmq介绍](#rabbitmq介绍)
- [Rabbitmq概念](#rabbitmq概念)
- [交换机类型](#交换机类型)
  - [direct模式](#direct模式)
  - [fanout模式](#fanout模式)
  - [topic模式](#topic模式)
  - [headers模式](#headers模式)
- [RPC机制](#rpc机制)

<!-- /TOC -->

# Rabbitmq介绍
AMQP，即Advanced Message Queuing Protocol，高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。消息中间件主要用于组件之间的解耦，消息的发送者无需知道消息使用者的存在，反之亦然。AMQP的主要特征是面向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全。

RabbitMQ是实现AMQP（高级消息队列协议）的消息中间件的一种，最初**起源于金融系统**，用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。RabbitMQ主要是为了实现系统之间的双向解耦而实现的。当生产者大量产生数据时，消费者无法快速消费，那么需要一个中间层。保存这个数据。

服务器端用Erlang语言编写，支持多种客户端，用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。

**总结：**  
1. 应用层的一个消息队列，主要用于消息的发送、组件之间的解耦；
2. 服务端使用Erlang语言开发，这就意味着Rabbitmq是需要运行在Erlang的环境下，就犹如Java和Jdk的关系；
3. 支持多种客户端，那就可以广泛的应用到各种的语言环境下，开发会比较方便。但是若需堆Rabbitmq进行扩展就需要Erlang开发者，者可能是一个比较困难的问题。
4. 起源于金融系统，按理来说安全性、消息的可靠性都是比较高的。
5. 是一个分布式的消息中间件架构，那就可用性强、容易扩展，可以用于分布式的系统中，按理来说性能表现也会比较好。

# Rabbitmq概念
Rabbitmq作为消息队列的内部结构基本如下图：   
![RabbitMQ内部结构](/ASSET/RabbitMQ内部结构.png)

1. **Message：消息**      
消息是不具名的，它由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key（路由键）、priority（相对于其他消息的优先权）、delivery-mode（指出该消息可能需要持久性存储）等。

2. **Publisher：生产者**    
消息的生产者，也是一个向交换器发布消息的客户端应用程序。

3. **Exchange： 交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列**  
在RabbitMQ中消息直接发送给队列这种事情永远不会发生。实际的情况是，生产者将消息发送到Exchange（交换器，下图中的X），由Exchange将消息路由到一个或多个Queue中（或者丢弃）。   
![RabbitMQ交换机](/ASSET/RabbitMQ交换机.png)

**Routing Key**    
生产者在将消息发送给Exchange的时候，一般会指定一个routing key，来指定这个消息的路由规则，而这个routing key需要与Exchange Type及binding key联合使用才能最终生效。 在Exchange Type与binding key固定的情况下（在正常使用时一般这些内容都是固定配置好的），我们的生产者就可以在发送消息给Exchange时，通过指定routing key来决定消息流向哪里。 RabbitMQ为routing key设定的长度限制为255 bytes。

4. **Binding: 绑定**      
RabbitMQ中通过Binding将Exchange与Queue关联起来，这样RabbitMQ就知道如何正确地将消息路由到指定的Queue了。    
![RabbitMQ绑定](/ASSET/RabbitMQ绑定.png)

在绑定（Binding）Exchange与Queue的同时，一般会指定一个binding key；消费者将消息发送给Exchange时，一般会指定一个routing key；当binding key与routing key相匹配时，消息将会被路由到对应的Queue中。 在绑定多个Queue到同一个Exchange的时候，这些Binding允许使用相同的binding key。 binding key 并不是在所有情况下都生效，它依赖于Exchange Type，比如fanout类型的Exchange就会无视binding key，而是将消息路由到所有绑定到该Exchange的Queue。

5. **Queue: 消息队列**     
用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。

6. **Connection: 网络连接，比如一个TCP连接**    

7. **Channel: 信道**    
多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的TCP连接内地虚拟连接，AMQP 命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成。因为对于操作系统来说建立和销毁 TCP 都是非常昂贵的开销，所以引入了信道的概念，以复用一条 TCP 连接。

8. **Consumer：消费者** 表示一个从消息队列中取得消息的客户端应用程序。 

9. **Virtual Host: 虚拟主机**  
表示一批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个 vhost 本质上就是一个 mini 版的 RabbitMQ 服务器，拥有自己的队列、交换器、绑定和权限机制。vhost 是 AMQP 概念的基础，必须在连接时指定，RabbitMQ 默认的 vhost 是 / 。

10. **Broker**   表示消息队列服务器实体 

# 交换机类型
## direct模式
direct类型的Exchange路由规则也很简单，它会把消息路由到那些binding key与routing key完全匹配的Queue中。
![RabbitMQ交换机direct类型](/ASSET/RabbitMQ交换机direct类型.png)

以上图的配置为例，我们以routingKey=”error”发送消息到Exchange，则消息会路由到Queue1（amqp.gen-S9b…，这是由RabbitMQ自动生成的Queue名称）和Queue2（amqp.gen-Agl…）；如果我们以routingKey=”info”或routingKey=”warning”来发送消息，则消息只会路由到Queue2。如果我们以其他routingKey发送消息，则消息不会路由到这两个Queue中。

## fanout模式
fanout类型的Exchange路由规则非常简单，它会把所有发送到该Exchange的消息路由到所有与它绑定的Queue中。**无论Queue使用哪种routingKey绑定到交换机上的，只要是通过该交换机分发的消息都会广播给这些queue**

![RabbitMQ交换机fanout类型](/ASSET/RabbitMQ交换机fanout类型.png)

上图中，生产者（P）发送到Exchange（X）的所有消息都会路由到图中的两个Queue，并最终被两个消费者（C1与C2）消费。

## topic模式
topic类型的Exchange在匹配规则上进行了扩展，它与direct类型的Exchage相似，也是将消息路由到binding key与routing key相匹配的Queue中，但这里的匹配规则有些不同，它约定：

- routing key为一个句点号“. ”分隔的字符串（我们将被句点号“. ”分隔开的每一段独立的字符串称为一个单词），如“stock.usd.nyse”、“nyse.vmw”、“quick.orange.rabbit”
- binding key与routing key一样也是句点号“. ”分隔的字符串
- binding key中可以存在两种特殊字符“*”与“#”，用于做模糊匹配，其中“*”用于匹配一个单词，“#”用于匹配多个单词（可以是零个）

![RabbitMQ交换机topic类型](/ASSET/RabbitMQ交换机topic类型.png)

以上图中的配置为例，routingKey=”quick.orange.rabbit”的消息会同时路由到Q1与Q2，routingKey=”lazy.orange.fox”的消息会路由到Q1与Q2，routingKey=”lazy.brown.fox”的消息会路由到Q2，routingKey=”lazy.pink.rabbit”的消息会路由到Q2（只会投递给Q2一次，虽然这个routingKey与Q2的两个bindingKey都匹配）；routingKey=”quick.brown.fox”、routingKey=”orange”、routingKey=”quick.orange.male.rabbit”的消息将会被丢弃，因为它们没有匹配任何bindingKey。

**注意：比如routingKey=topic， ` * / topic / topic.# / #.topic.#` 都可以匹配到**

## headers模式
headers类型的Exchange不依赖于routing key与binding key的匹配规则来路由消息，而是根据发送的消息内容中的headers属性进行匹配。 在绑定Queue与Exchange时指定一组键值对；当消息发送到Exchange时，RabbitMQ会取到该消息的headers（也是一个键值对的形式），对比其中的键值对是否完全匹配Queue与Exchange绑定时指定的键值对；如果完全匹配则消息会路由到该Queue，否则不会路由到该Queue。

**这种方式和direct的方式其实差别不大，但是性能非常的差，所以实际使用种很少用到，所以建议尽量不要使用这种类型的交换机。**

# RPC机制
MQ本身是基于异步的消息处理，前面的示例中所有的生产者（P）将消息发送到RabbitMQ后不会知道消费者（C）处理成功或者失败（甚至连有没有消费者来处理这条消息都不知道）。 但实际的应用场景中，我们很可能需要一些同步处理，需要同步等待服务端将我的消息处理完成后再进行下一步处理。这相当于RPC（Remote Procedure Call，远程过程调用）。在RabbitMQ中也支持RPC。

![RabbitMQ的RPC机制](/ASSET/RabbitMQ的RPC机制.png)

**RabbitMQ 中实现RPC 的机制是：**

- 客户端发送请求（消息）时，在消息的属性（MessageProperties ，在AMQP 协议中定义了14中properties ，这些属性会随着消息一起发送）中设置两个值replyTo （一个Queue 名称，用于告诉服务器处理完成后将通知我的消息发送到这个Queue 中）和correlationId （此次请求的标识号，服务器处理完成后需要将此属性返还，客户端将根据这个id了解哪条请求被成功执行了或执行失败）
- 服务器端收到消息并处理
- 服务器端处理完消息后，将生成一条应答消息到replyTo 指定的Queue ，同时带上correlationId 属性
- 客户端之前已订阅replyTo 指定的Queue ，从中收到服务器的应答消息后，根据其中的correlationId 属性分析哪条请求被执行了，根据执行结果进行后续业务处理

