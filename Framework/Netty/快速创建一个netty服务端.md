<!-- TOC -->

- [构建Netty服务端](#构建netty服务端)
- [Bootstrap Netty引导类](#bootstrap-netty引导类)
- [EventLoopGroup](#eventloopgroup)
- [ChannelHandler中的handler的执行顺序](#channelhandler中的handler的执行顺序)
- [`handler` 和 `childHandler` 的区别](#handler-和-childhandler-的区别)
- [如何理解：netty中的I/O操作都是异步的](#如何理解netty中的io操作都是异步的)

<!-- /TOC -->

# 构建Netty服务端

1. 使用Netty服务端引导类启动
```
@SuppressWarnings("all")
@Autowired(required = false)
@Qualifier(value = "workGroup")
private EventLoopGroup workGroup;

@SuppressWarnings("all")
@Autowired(required = false)
@Qualifier(value = "bossGroup")
private EventLoopGroup bossGroup;

@Autowired(required = false)
@Qualifier(value = "executorGroup")
private EventExecutorGroup executorGroup;

@Autowired(required = false)
private SocketResponseHandler responseHandler;

@Autowired(required = false)
private SocketRequestHandler requestHandler;

public void start() throws InterruptedException {

    // 创建服务端启动类
    ServerBootstrap bootstrap = new ServerBootstrap();
    // 设置启动参数
    BaseNettyServerProperties.RecBuf recBuf = nettyServerProperties.getRecbuf();

    bootstrap.group(bossGroup, workGroup)
            // 设置服务绑定段楼
            .localAddress(new InetSocketAddress(nettyServerProperties.getPort()))
            // 指定Channel 根据操作系统设置
            .channel(OSInfo.useNettyEpoll() ? EpollServerSocketChannel.class : NioServerSocketChannel.class)
            // 设置为TCP协议
            .childOption(ChannelOption.TCP_NODELAY, true)
            // 设置接收字节流大小空间
            .childOption(ChannelOption.RCVBUF_ALLOCATOR,
                    new AdaptiveRecvByteBufAllocator(recBuf.getMinSize(), recBuf.getInitial(), recBuf.getMaxSize()))
            //设置最大发送字节
            .childOption(ChannelOption.SO_SNDBUF, recBuf.getMaxSize())
            //设置TCP长连接,一般如果两个小时内没有数据的通信时,TCP会自动发送一个活动探测数据报文
            .childOption(ChannelOption.SO_KEEPALIVE, true)
            //将小的数据包包装成更大的帧进行传送，提高网络的负载
            .childOption(ChannelOption.TCP_NODELAY, false)
            .childHandler(new ChannelInitializer<SocketChannel>() {

                @Override
                protected void initChannel(SocketChannel channel) throws Exception {
                    channel.pipeline()
                            // 字符串节码
                            .addLast(new StringEncoder(Charset.forName("GBK")))
                            // 字符串编码
                            .addLast(new StringDecoder(Charset.forName("GBK")))
                            // 响应处理器
                            .addLast(responseHandler)
                            // 读取数据超时
                            .addLast(new ReadTimeoutHandler(nettyServerProperties.getRecbuf().getTimeout() * 1000, TimeUnit.MILLISECONDS))
                            // 写入数据超时
                            .addLast(new WriteTimeoutHandler(nettyServerProperties.getWriteBuf().getTimeout() * 1000, TimeUnit.MILLISECONDS))
                            // 业务请求处理器
                            .addLast(executorGroup, "reqHandler", requestHandler);

                }
            });
    ChannelFuture future = bootstrap.bind().sync();

    //3、添加绑定端口监听器
    future.addListener(new ChannelFutureListener() {
        @Override
        public void operationComplete(ChannelFuture future) throws Exception {
            if (future.isSuccess()) {
                logger.info("Socket server bound successfully,the port is {}", nettyServerProperties.getPort());
            } else {
                logger.info("Socket server bound failed,the cause is {}", future.cause().getMessage(), future.cause());
            }
        }
    });

    //4、添加关闭通道监听器
    future.channel().closeFuture().sync().addListener(new ChannelFutureListener() {
        @Override
        public void operationComplete(ChannelFuture future) throws Exception {
            if (future.isSuccess()) {
                logger.info("the server channel is closed successfully");
            } else {
                logger.info("the server channel is closed with exception:{}", future.cause().getMessage(), future.cause());
            }
        }
    });
}

public void stop() {
    try {
        //boss线程组关闭监听器
        bossGroup.shutdownGracefully().sync().addListener(new GenericFutureListener<Future<Object>>() {
            @Override
            public void operationComplete(Future<Object> future) throws Exception {
                if (future.isSuccess()) {
                    logger.info("the bossGroup is closed successfully");
                } else {
                    logger.info("the bossGroup is closed with exception:{}", future.cause().getMessage(), future.cause());
                }
            }
        });
        //worker线程组关闭监听器
        workGroup.shutdownGracefully().sync().addListener(new GenericFutureListener<Future<Object>>() {
            @Override
            public void operationComplete(Future<Object> future) throws Exception {
                if (future.isSuccess()) {
                    logger.info("the workGroup is closed successfully");
                } else {
                    logger.info("the workGroup is closed with exception:{}", future.cause().getMessage(), future.cause());
                }
            }
        });
    } catch (Exception e) {
        logger.error("{}", e.getMessage(), e);
    }
}    
```

2. 读数据handler(`ChannelInboundHandler`)
```
@Component
@ChannelHandler.Sharable
public class SocketRequestHandler extends SimpleChannelInboundHandler<String> {
    private static final long serialVersionUID = 5106143657009716158L;

    private static final Logger logger = LoggerFactory.getLogger(SocketRequestHandler.class);

    @Autowired
    private IServiceFacade serviceFacade;

//    @Override
    public String reqHandler(String request) throws Exception {

        return serviceFacade.doHandle(request);

    }

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {
        logger.debug("服务器收到消息{}===>{}", Thread.currentThread().getName(), msg);

        String reqResult = reqHandler(msg.substring(8));
        ctx.writeAndFlush(reqResult);
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ctx.close();
    }
}
```

3. 写数据handler(`ChannelOutboundHandler`)
```
@Component
@ChannelHandler.Sharable
public class SocketResponseHandler extends ChannelOutboundHandlerAdapter {

    private static final Logger logger = LoggerFactory.getLogger(SocketResponseHandler.class);

    @Override
    public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
        ByteBuf buf = Unpooled.copiedBuffer((String)msg, CharsetUtil.UTF_8);
        logger.debug("socket result data :{}",buf.toString(CharsetUtil.UTF_8));
        super.write(ctx, msg, promise);
    }
}
```

# Bootstrap Netty引导类
1. 一个 Netty 应用通常由一个 Bootstrap 开始， 主要作用是配置整个 Netty 程序， 串联各个组件， Netty 中 Bootstrap 类是客户端程序的启动引导类， ServerBootstrap 是服务端启动引导类。
2. 常见的方法：
- public ServerBootstrap group(EventLoopGroup parentGroup, EventLoopGroup childGroup)，该方法用于服务器端，用来设置两个 EventLoop public B group(EventLoopGroup group) ，该方法用于客户端，用来设置一个 EventLoop
- public B channel(Class<? extends C> channelClass)，该方法用来设置一个服务器端的通道实现
- public <T> B option(ChannelOption<T> option, T value)，用来给 ServerChannel 添加配置
- public <T> ServerBootstrap childOption(ChannelOption<T> childOption, T value)，用来给接收到的通道添加配置
- public ServerBootstrap childHandler(ChannelHandler childHandler)，该方法用来设置业务处理类（自定义的 handler）
- public ChannelFuture bind(int inetPort) ，该方法用于服务器端，用来设置占用的端口号 public ChannelFuture connect(String inetHost, int inetPort) ，该方法用于客户端，用来连接服务器端

# EventLoopGroup
1. EventLoopGroup 是一组 EventLoop 的抽象， Netty 为了更好的利用多核 CPU 资源， 一般会有多个 EventLoop同时工作， 每个 EventLoop 维护着一个 Selector 实例。
2. EventLoopGroup 提供 next 接口， 可以从组里面按照一定规则获取其中一个 EventLoop 来处理任务。 在 Netty服 务 器 端 编 程 中 ， 我 们 一 般 都 需 要 提 供 两 个 EventLoopGroup ，例 如 ： BossEventLoopGroup 和WorkerEventLoopGroup。
3. 通常一个服务端口即一个 ServerSocketChannel 对应一个 Selector 和一个 EventLoop 线程。 BossEventLoop 负责接收客户端的连接并将 SocketChannel 交给 WorkerEventLoopGroup 来进行 IO 处理， 如下图所示 
![EventLoopGroup](http://sunyanping24.gitee.io/ASSET/EventLoopGroup.png)

# ChannelHandler中的handler的执行顺序
有时需要在ChannelHandler添加多个业务处理handler，多个handler的执行顺序通常是：ChannelInboundHandler按照注册的先后顺序执行；ChannelOutboundHandler按照注册的先后顺序逆序执行。

# `handler` 和 `childHandler` 的区别
handler在初始化时就会执行，而childHandler会在客户端成功connect后才执行。

# 如何理解：netty中的I/O操作都是异步的

