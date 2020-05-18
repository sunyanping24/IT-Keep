<!-- TOC -->

- [构建Netty服务端](#构建netty服务端)

<!-- /TOC -->

# 构建Netty服务端

1. 使用Netty服务端引导类启动
```
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
```

```
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