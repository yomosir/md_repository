# netty整合springboot

整合netty和springboot

## 唯一id插件

如果要进行分库分表，数据库中的主键必须唯一，所以不能使用自增，必须使用varchar类型ID

## 代码展示

服务启动类必须单例，所以使用单例模式进行编写服务启动

```java
public class WSServer{
    private EvenLoopGroup mainGroup;
    private EvenLoopGroup subGroup;
    private ServerBootstrap server;
    private ChannelFuture future;
    
    private static class SingletionWSServer{
        static final WSServer instance = new WSServer();
    }
    
    public static WSServer getInstance(){
        return SingletionWSServer.instance;
    }
    
    private WSServer(){
        mainGroup = new NioEventLoopGroup();
        subGroup = new NioEventLoopGroup();
        server = new ServerBootstrap();
        server.group(mainGroup,subGroup)
            .channel(NioServerSocketChannel.class)
            .childHandler(null);
        this.start();
    }
    
    private void start(){
        this.future = server.bind(port);
    }
}
```



启动

```java
@Component
public class NettyBooter implements ApplicationListener<ContextRefreshedEvent>{
    
    @Override
	public void onApplicationEvent(ContextRefreshedEvent event) {
		if (event.getApplicationContext().getParent() == null) {
			try {
				WSServer.getInstance().start();
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
	}
}
```