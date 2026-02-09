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

## 3	AbstractServer 抽象类

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

