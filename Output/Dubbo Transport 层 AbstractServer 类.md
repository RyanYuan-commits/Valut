---
type: permanent
banner:
---
---

**关键词**: Dubbo, Remoting

---

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


---