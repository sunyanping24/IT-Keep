<!-- TOC -->

- [zuul 1.x 和 zuul 2.x 和区别](#zuul-1x-和-zuul-2x-和区别)
- [openfeign、ribbon、hystrix三者的作用和联系](#openfeignribbonhystrix三者的作用和联系)
  - [三者的作用](#三者的作用)
  - [三者之间的联系](#三者之间的联系)

<!-- /TOC -->

# zuul 1.x 和 zuul 2.x 和区别
参考这篇文章即可：[https://blog.csdn.net/yang75108/article/details/86991401](https://blog.csdn.net/yang75108/article/details/86991401)

大概总结一下：    
1. zuul 1.x 使用的是同步编程模型，zuul实际上就是一个servlet；zuul 2.x 使用的是异步编程模型，使用netty框架来实现的。
2. 过滤器不一样：用Inbound Filters代替Pre-routing Filters，用Endpoint Filter代替Routing Filter，用Outbound Filters代替Post-routing Filters。
3. zuul 1.x 由于是同步模型，所以适用于计算密集型场景，而zuul 2.x 适用于I/O密集型场景。
4. 推荐在生产上适用zuul 1.x：（1）zuul 1.x开源时间比较长，更加稳定；（2）zuul 1.x使用的同步模型，开发调试运维简单；（3）zuul 2.x是为了满足Netflix日千亿级流量才开发，使用了异步模型，目前对于大部分企业来说日上亿级数据都是比较遥远，所以zuul 1.x基本完全可以满足需求。（4）zuul 1.x同步编程，方便使用链路监控工具埋点。

# openfeign、ribbon、hystrix三者的作用和联系

## 三者的作用   
- **openfeign:**
Feign是使用Java编写的HttpClient绑定器，在Spring Cloud微服务架构中用于实现服务之间的声明式调用。Feign可以定义请求到其他服务的接口，实现微服务之间的调用，不用自己再写http请求。

- **ribbon:**   
Ribbon是一个基于HTTP和TCP的客户端负载均衡工具，它基于Netflix Ribbon实现。通过Spring Cloud的封装，可以让我们轻松地将面向服务的REST模板请求自动转换成客户端负载均衡的服务调用。ribbon既然做的负载均衡，那么实际上的http请求的发起也就是ribbon发起的。

Ribbon内置了几种常见的负载均衡策略：
1. **轮询策略（RoundRobinRule）**  

2. **权重轮询策略（WeightedResponseTimeRule）**：根据每个 provider 的响应时间分配一个权重，响应时间越长，权重越小，被选中的可能性越低。     
**实现原理**：一开始为轮询策略，并开启一个计时器，每 30 秒收集一次每个 provider 的平均响应时间，当信息足够时，给每个 provider 附上一个权重，并按权重随机选择 provider，高权越重的 provider 会被高概率选中。

3. **随机策略（RandomRule）**

4. **最少并发数策略（BestAvailableRule）**：选择正在请求中的并发数最小的 provider，除非这个 provider 在熔断中。

5. **重试策略（RetryRule）**：其实就是轮询策略的增强版，轮询策略服务不可用时不做处理，重试策略服务不可用时会重新尝试集群中的其他节点。

6. **可用性敏感策略（AvailabilityFilteringRule）**：过滤性能差的 provider   
（1）第一种：过滤掉在 Eureka 中处于一直连接失败的 provider。    
（2）第二种：过滤掉高并发（繁忙）的 provider。

7. **区域敏感性策略（ZoneAvoidanceRule）**      
（1）以一个区域为单位考察可用性，对于不可用的区域整个丢弃，从剩下区域中选可用的 provider。          
（2）如果这个 ip 区域内有一个或多个实例不可达或响应变慢，都会降低该 ip 区域内其他 ip 被选中的权 重。

- **hystrix:**    
Hystrix作为熔断流量控制，旨在通过熔断机制控制服务和第三方库的节点,从而对延迟和故障提供更强大的容错能力。


## 三者之间的联系
![](http://sunyanping.gitee.io/it-keep/ASSET/从一个请求看Feign-Hystrix-Ribbon三者之间的联系.png)
![](http://sunyanping.gitee.io/it-keep/ASSET/从一个请求看Feign-Hystrix-Ribbon三者之间的联系2.png)

从上面这2个图基本能够看出这三者之间的联系，这三者实际上在实现时之间就是存在紧密结合的，在实际的使用中，主要关注的就是调用超时在这三者之间的关系，因为他们都和调用超时存在关系。

**总结这三者的超时时间关系：**      
hystrix的熔断时间配置通过yml配置没法生效，可以通过配置类的方法来修改，feign的超时时间可以通过代码或者yml配置，ribbon的超时时间可以通过yml来配置。
feign和ribbon的超时时间只能二选一，只要feign的超时时间配置了，就以feign的为准，hystrix的超时时间要大于feign/riboon的connectTimeout+readTimeout的和

**参考原文：**    
hystrix ,feign,ribbon的超时时间配置，以及原理分析：[https://www.cnblogs.com/yangxiaohui227/p/13031370.html](https://www.cnblogs.com/yangxiaohui227/p/13031370.html)
