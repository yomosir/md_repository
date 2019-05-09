# netty webscoket

~~there are some tips of using netty to implement a websocket server~~
使用netty构建websocket服务端的相关注意事项；使用的版:netty4.1.6Final

## 关于handler的实现

关于handler实现问题，在实现netty的handler时，有两个可选择的父类，分别是`ChannelInboundHandlerAdapter` 和 `SimpleChannelInboundHandler`.

- 实现`ChannelInboundHandlerAdapter`
  
1. 实现`ChannelInboundHandlerAdapter`需要注意，这个类其本身并由做什么处理，需要overwrite其一些方法，如`channelRead0(),channelActive(),channelInactive()`等方法，对事件进行处理
2. 这个类是不会自动释放发送过来的消息的，所以其接收到的数据是可以传输到下一个handler的（可以配置多个handler，在配置时，先配置的在实际handler链中是在后面的）。在这儿需要注意每次read数据之后要释放数据.

```java
try{
    ...
}finally{
    ReferenceCountUtil.release(msg);
}
```

- 实现`SimpleChannelInboundHandler`

1. `SimpleChannelInboundHandler`是会自动去释放数据，因此不需要去释放数据。
2. `SimpleChannelInboundHandler`在继承时可以添加泛型，在接收到数据时可以直接通过管道中配置的解码将其解码为泛型类。

- 在进行handler实现时，会涉及发送数据给客户端，使用`ctx.writeAndFlush()`方法，此时需要注意发送的数据必须是netty自身定义的websocketFrame类，如`TextWebSocketFrame,BinaryWebSocketFrame`.一般在进行网络传输数据时都会使用二进制的字节流，所以可以将protobuf的数据流转换成`BinaryWebSocketFrame`方式，发送给客户端，这样无论客户端是什么语言，都可以使用protobuf去解析成对应的类。

## HeartBeat相关

- 连接有读操作，写操作，读写操作，如果一段时间没有进行数据的读写，那么就需要心跳包使得长连接的知道客户端还存在，保持长连接。

- 与此同时，如果客户端一段时间没有发心跳，则可以断开该客户端，减少服务端的资源浪费。

### Netty心跳

Netty可以使用userEventTriggered实现对用户事件的出发，通过这个方法来实现心跳机制

**IdelStateEvent**

- 在出现空闲时间，会触发此Event，我可以通过获取到此类型的Event（读空闲，写空闲，读写空闲）

-  可以在加载pipeline时使用`IdleStateHandler(readIdleTime,writeIdleTime,allIdleTime,TimeUnit)`配置，读空闲，写空闲，读写空闲的间隔时间
- 当超过设置的时间就会触发对应的Event，用这种方式来设置心跳的策略