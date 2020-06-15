# zuul 1.x 和 zuul 2.x 和区别
参考这篇文章即可：[https://blog.csdn.net/yang75108/article/details/86991401](https://blog.csdn.net/yang75108/article/details/86991401)

大概总结一下：    
1. zuul 1.x 使用的是同步编程模型，zuul实际上就是一个servlet；zuul 2.x 使用的是异步编程模型，使用netty框架来实现的。
2. 过滤器不一样：用Inbound Filters代替Pre-routing Filters，用Endpoint Filter代替Routing Filter，用Outbound Filters代替Post-routing Filters。
3. zuul 1.x 由于是同步模型，所以适用于计算密集型场景，而zuul 2.x 适用于I/O密集型场景。
4. 推荐在生产上适用zuul 1.x：（1）zuul 1.x开源时间比较长，更加稳定；（2）zuul 1.x使用的同步模型，开发调试运维简单；（3）zuul 2.x是为了满足Netflix日千亿级流量才开发，使用了异步模型，目前对于大部分企业来说日上亿级数据都是比较遥远，所以zuul 1.x基本完全可以满足需求。（4）zuul 1.x同步编程，方便使用链路监控工具埋点。
