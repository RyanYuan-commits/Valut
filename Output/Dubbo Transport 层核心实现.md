---
created: 2025-11-24
type: permanent
banner: Assets/Banner/pexels-walidphotoz-1509582.jpg
---
# 🌐 核心观点

Dubbo Transport 属于 [[Dubbo Remoting 层核心类与接口|Dubbo Remoting]] 层, 抽象 mina 和 netty 为统一接口, 以 Message 为中心;

![[Dubbo Remoting 层.png|900]]

---

# 🔖 详细解释

## 2	AbstractEndpoint 抽象类


## 3	AbstractServer 抽象类

```java
public abstract class AbstractServer extends AbstractEndpoint implements RemotingServer {  
  
    private Set<ExecutorService> executors = new ConcurrentHashSet<>();  
    private InetSocketAddress localAddress;  
    private InetSocketAddress bindAddress;  
    private int accepts;
	
	// ......
}
```

### 3.1	继承关系与关键属性

AbstractServer 继承了 AbstractEndpoint 抽象类, 同时实现了 RemotingServer 接口, 是一个有连接状态的类.

其核心字段有下面几个:

- localAddress: 本地地址, 或者说叫发布地址, 是 Provider 向注册中心注册时使用的地址;
		
- bindAddress: 服务启动时将其套接字绑定到本地的哪个 IP 地址, 一个服务器可能有多个网络接口, 对应多个 IP 地址.
	
- accepts: 该 Server 能接受的最大连接数, 同样从 URL 中获取, 默认值为 0, 表示没有限制;
	
- executorRepository: 负责管理线程池;
	
- executors: 当前 Server 绑定的线程池, 由 executorRepository 创建.

### 3.2	构造方法与启动流程

AbstractServer 的构造方法中根据传入的 URL 初始化上述字段, 然后调用 doOpen 抽象方法完成启动流程.

```java
public AbstractServer(URL url, ChannelHandler handler) throws RemotingException {  
	// 根据传入的 URL 初始化字段
    super(url, handler);  
    executorRepository = ExecutorRepository.getInstance(url.getOrDefaultApplicationModel());  
    localAddress = getUrl().toInetSocketAddress();  
  
    String bindIp = getUrl().getParameter(Constants.BIND_IP_KEY, getUrl().getHost());  
    int bindPort = getUrl().getParameter(Constants.BIND_PORT_KEY, getUrl().getPort());  
    if (url.getParameter(ANYHOST_KEY, false) || NetUtils.isInvalidLocalHost(bindIp)) {  
        bindIp = ANYHOST_VALUE;  
    }  
    bindAddress = new InetSocketAddress(bindIp, bindPort);  
    this.accepts = url.getParameter(ACCEPTS_KEY, DEFAULT_ACCEPTS);  

	// 调用 doOpen 方法完成启动
    try {  
        doOpen();  
        if (logger.isInfoEnabled()) {  
            logger.info("[SERVICE_PUBLISH][METADATA_REGISTER] Start "  
                    + getClass().getSimpleName() + " bind " + getBindAddress() + ", export " + getLocalAddress());  
        }  
    } catch (Throwable t) {  
        throw new RemotingException(  
                url.toInetSocketAddress(),  
                null,  
                "Failed to bind " + getClass().getSimpleName() + " on " + bindAddress + ", cause: "  
                        + t.getMessage(),  
                t);  
    }  

	// 构造线程池
    executors.add(  
            executorRepository.createExecutorIfAbsent(ExecutorUtil.setThreadName(url, SERVER_THREAD_POOL_NAME)));  
}
```

## 4	NettyServer

NettyServer 继承了 AbstractServer 抽象类, 实现了其定义的 doOpen 抽象方法.

```java
@Override
protected void doOpen() throws Throwable {
	NettyHelper.setNettyLoggerFactory();

	// 构建 boxx 和 worker 线程池
	ExecutorService boss = Executors.newCachedThreadPool(new NamedThreadFactory(EVENT_LOOP_BOSS_POOL_NAME, true));
	ExecutorService worker =
			Executors.newCachedThreadPool(new NamedThreadFactory(EVENT_LOOP_WORKER_POOL_NAME, true));
	ChannelFactory channelFactory = new NioServerSocketChannelFactory(
			boss, worker, getUrl().getPositiveParameter(IO_THREADS_KEY, Constants.DEFAULT_IO_THREADS));
	
	// 核心启动类
	bootstrap = new ServerBootstrap(channelFactory);

	// 维护与该 Server 建立链接的 Channel, 是一个 Map, key 是远程地址
	final NettyHandler nettyHandler = new NettyHandler(getUrl(), this);
	channels = nettyHandler.getChannels();

	// 启动类配置
	bootstrap.setOption("child.tcpNoDelay", true);
	bootstrap.setOption("backlog", getUrl().getPositiveParameter(BACKLOG_KEY, Constants.DEFAULT_BACKLOG));
	bootstrap.setOption("reuseAddress", true);
	bootstrap.setPipelineFactory(new ChannelPipelineFactory() {
		@Override
		public ChannelPipeline getPipeline() {
			NettyCodecAdapter adapter = new NettyCodecAdapter(getCodec(), getUrl(), NettyServer.this);
			ChannelPipeline pipeline = Channels.pipeline();
			pipeline.addLast("decoder", adapter.getDecoder());
			pipeline.addLast("encoder", adapter.getEncoder());
			pipeline.addLast("handler", nettyHandler);
			return pipeline;
		}
	});

	// 绑定并获取 Channel 实例
	channel = bootstrap.bind(getBindAddress());
}
```

在这个方法中, Dubbo 完成了 ServerBootstrap 的初始化, Boss 和 Worker EventLoopGroup 的创建, 以及通过 ChannelInitializer 初始化 ChannelHandler 等一系列 Netty 服务启动的标准流程; 最终得到的结构大致为:

![[Dubbo NettyServer|900]]

---

# 📚 参考内容

