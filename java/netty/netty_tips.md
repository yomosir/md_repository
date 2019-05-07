# netty framework tips

- nio的类库和api使用起来比较麻烦，还必须对多线程等方面有很好的了解。
- nio本身还有bug。netty本身的api比较简单。
- 定制性很强。

- 提供了三种线程模型

## reactor 线程模型

### 单线程模型：所有的IO操作都由同一个NIO线程处理

- 单个线程用于接收所有的链接，只能满足小型的场景

- 缺点： 性能上不能满足特别大的负载，当负荷过大，整体的处理速度就会下降，然后客户端连接时就会超时，这个客户端就会重试，频繁的重试会更加的加重服务器的负载，这样就会恶性循环

### 多线程模型： 有一组NIO线程处理IO操作

将原先的reactor单线程拆分成为了两块：

- *一个是一个单线程* ： 处理客户端链接，将客户端丢给后面的线程池，处理客户端连接请求
- *另一个是一个线程池* ：处理读写请求

### 主从线程模型：一组线程池接受请求，一组线程池处理IO

- 主线程池 ：当服务端接收到客户端请求，然后就会将这些连接注册到从（IO）线程池中，当链路建立完成后，剩下的工作就会由从线程池去操作
- 从线程池 ：编解码，读写



## 编写简单的netty服务的过程

- 构建一对主从线程组

- 定义服务器启动类

- 为服务设置Channel

- 设置处理从线程池的助手类初始化器(相当于拦截器)

- 监听启动和关闭服务器

  ### 编写代码，api介绍

  - 定义一对线程组（使用 **EvenLoopGroup** 类）
    - **bossGroup** 主线程组，用于接收客户端的连接，但是不做任何处理，跟老板一样，不做事
    - **workerGroup** 从线程组，老板线程组会把任务丢给他，让手下线程组去做任务

  - 启动类
    - **ServerBootstrap**
      - group() 方法 ：需要针对线程模型设置group，将两个线程组当做参数传入group方法中
      - channel()方法： 设置通道类型NioServerSocketChannel.class
      - childHandler()：针对从线程组做一个相应的操作。（子处理器）
      - 启动netty服务

  - ```ChannelFuture future =  serverBootStrap.bind(port).sync(); ``//绑定端口后，等待启动完成，同时启动方式为同步

  - ```future.channel().closeFuture().sync()```用于监听关闭的channel，设置为同步方式

  - 优雅的关闭 ： ```bossGroup.shutdownGracefully()    /   workerGroup.shutdownGracefully()```

    
  ####   设置channel初始化器

  每一个channel由多个handler共同组成的管道（**pipeline**）， **handler**可以视为一个个助手类
  
  *pipline可视为一个大的拦截器，然后handler是一个个小的拦截器。* 
  
  - 需要继承`ChannelInitializer<SocketChannel>`,实现需要实现的方法
  
    - 通过SocketChannel获取对应的管道： `ChannelPipline pipline = channel.pipline();`
  
    - addAfter,addBefore,addLast三种方法，一般使用addLast()，常用的handler：
  
      - `HttpServerCodec`时netty自己提供的助手类，当请求到服务端时我们需要进行解码，响应到客户端做编码
      - `ChunkedWriteHandler` 是对写大数据流的支持
      - `HttpObjectAggregator(length)`是一个聚合器，length是消息的最大长度，对HttpMessage进行聚合，聚合成FullHttpRequest和FullHttpResponse，几乎在netty中的变成都会使用到此handler；length可以为：1024*64 **以上三个是对http协议的支持**
  - `WebsocketServerProtocolHandler(path)`是服务器针对websocket处理的协议，用户指定给客户端连接访问的路由：path；
        - 本handler会帮助处理一些繁重复杂的事，会帮助处理握手动作：handshaking（close，ping，pong）
    - 对于websocket来讲，都是以frames传输的，不同的数据类型对应的frames也不同，如（TextWebsocketFrame，BinaryWebsocketFrame等）
      - 如果使用protobuf可以使用protobuf的handler
      - 可以添加自定义的handler
      
  ##### 写自定义handler
      
  - 可以继承`SimpleChannelInboundHandler<HttpObject>`,重写相应的方法
      - `ByteBuf`是一个缓冲区，我们可以将我们的数据拷贝到缓冲区中，使用`Unpooled.copiedBuffer()`将数据拷贝到缓冲区中
      - HTTP_1_1会默认开启keepAlive
      
      **连接建立的生命周期**：
      
      - 助手类添加
      - channel注册
  - channel活跃（Active）
      - channel读取完毕
      - channel不活跃
      - channel移除
      - 助手类移除
      
      