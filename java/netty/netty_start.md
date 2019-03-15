# netty starter

netty相关的基础开发使用方式

## simple implements

1. handler应该要继承 `io.netty.channel.ChannelInboundHandlerAdapter`类

```java
    pulic class MyHandler extends ChannelInboundHandlerAdapter{
        //重写 channelRead > 当接收到消息是调用 
        //重写 exceptionCaught > 当出现Netty异常时调用
        ...
    }
```

2. 服务类

```java
    public void run() throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup(); // (1)
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap b = new ServerBootstrap(); // (2)
            b.group(bossGroup, workerGroup)
             .channel(NioServerSocketChannel.class) // (3)
             .childHandler(new ChannelInitializer<SocketChannel>() { // (4)
                 @Override
                 public void initChannel(SocketChannel ch) throws Exception {
                     ch.pipeline().addLast(new DiscardServerHandler());
                 }
             })
             .option(ChannelOption.SO_BACKLOG, 128)          // (5)
             .childOption(ChannelOption.SO_KEEPALIVE, true); // (6)
            // Bind and start to accept incoming connections.
            ChannelFuture f = b.bind(port).sync(); // (7)
            // Wait until the server socket is closed.
            // In this example, this does not happen, but you can do that to gracefully
            // shut down your server.
            f.channel().closeFuture().sync();
        } finally {
            workerGroup.shutdownGracefully();
            bossGroup.shutdownGracefully();
        }
    }
```

- `NioEventLoopGroup` : 多线程事件循环，当使用服务端应用，需要使用两个NioEventLoopGroup,其中boss接受过来的连接，worker处理已接受连接的流量,多少线程会被使用，他们是怎么映射的依赖于`EventLoopGroup`实现，甚至可以通过构造函数去配置
- `ServerBootstrap` : 帮助类去启动server
- 指定`NioServerSocketChannel` 去新建一个新的Channel接收连接
- `ChannelInitializer`用来配置一个新的Channel
- option() 是为`NioServerSocketChannel`接收的连接设置的参数，childOption()是为通过ServerChannel接收的Channels设置的，例如`NioServerSocketChannel`

3. 接收数据的方法

```java
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ByteBuf in = (ByteBuf) msg;//将数据强制转化成需要的类的类型
        try {
            while (in.isReadable()) { // (1)
                System.out.print((char) in.readByte());
                System.out.flush();
            }
        } finally {
            ReferenceCountUtil.release(msg); // (2)
        }
    }
```

- (1)处低效的循环可以使用`System.out.println(in.toString(io.netty.util.CharsetUtil.US_ASCII))`简化
- (2)处可以使用`in.release()`代替

4. 一个可以返回数据的服务端

```java
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ctx.write(msg); // (1)
        ctx.flush(); // (2)
    }
```

- `ChannelHandlerContext`提供了许多可以触发许多I/O事件和操作的操作，这里不需要release，这是因为在你将其写出时，netty帮你release了。

- ctx.write()方法并没有将数据写出，他是将数据缓存起来，然后通过ctx.flush()将数据刷出去，你可以使用ctx.writeAndFlush()代替

5. 一个定时服务

这个服务会忽略所有发过来的信息，然后像客户端发送时间，因此需要覆写channelActive方法

```java
    @Override
    public void channelActive(final ChannelHandlerContext ctx) { // (1)
        final ByteBuf time = ctx.alloc().buffer(4); // (2)
        time.writeInt((int) (System.currentTimeMillis() / 1000L + 2208988800L));
        final ChannelFuture f = ctx.writeAndFlush(time); // (3)
        f.addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture future) {
                assert f == future;
                ctx.close();
            }
        }); // (4)
    }
```

- channelActive ： 当连接建立完成，等待生成流量

- 发送消息需要分配包含消息的新缓存，通过`ctx.alloc()`获取当前的分配空间并分配新的缓存

- `ctx.writeAndFlush()`返回一个ChannelFuture,一个ChannelFuture代表一个还没有发生的I/O操作。这代表，任何被请求过的操作可能还不会发生，一位所有的操作再netty中都是异步的。

```java
//下面这段代码可能会使得在发送消息之前将连接关闭
    Channel ch = ...;
    ch.writeAndFlush(message);
    ch.close();
```

因此需要等ChannelFuture完成后才能关闭连接。只需要为返回的ChannelFuture简单的添加一个`ChannelFutureListener`。创建一个匿名的`ChannelFutureListener`，当操作结束后在进行关闭连接。
