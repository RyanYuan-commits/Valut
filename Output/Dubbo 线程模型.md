---
created: 2025-11-30
type: permanent
draw: "[[Draw Place of Dubbo 线程模型]]"
---
# 🌐 核心观点



---

# 🔖 详细解释

## 1	线程模型概述

Dubbo 使用 Netty 作为默认的网络通信框架, Provider 方使用两级线程池, boss 线程池负责处理连接建立事件, worker 线程池负责处理其他事件, 这两级线程池中的线程一般被称为 IO 线程. 

Dubbo 中的 IO 线程是非常珍贵的资源, 这种珍贵体现在其重要性和数量上:

- 从重要性角度来说, IO 线程直接负责连接建立, 数据读写; 一但 IO 线程耗尽, Netty 的事件循环就被卡住了, 具体体现为导致新连接无法建立, 已有连接无法读取或写出等状况;
	
- 从数量角度来说, IO 线程的数量是有限的, Dubbo 默认的 boss 线程数为 1, worker 线程数最大为 32.

所以, 在 IO 线程上不应该处理耗时过久的任务, 基于这个原则, Dubbo 设计了业务线程池来隔离业务逻辑和 IO 处理逻辑, 通过 Dispatcher 来处理从 IO 线程到业务线程的派发, 整体的线程模型大致为:

![[Dubbo 线程模型.png|700]]

Diaptcher.dispatch 方法返回一个 ChannelHandler 实例, 这个 ChannelHandler 最终会作为 Dubbo 请求处理流中的一个节点, 完成线程池切换工作.

## 2	ChannelHandler 与 Message

Dubbo 设计了 ChannelHandler 接口来处理通信过程中发生的各种事件, 当事件发生后, Dubbo 会调用 ChannelHandler 对应方法来处理, 具体有:

```java
// org.apache.dubbo.remoting.ChannelHandler
@SPI(scope = ExtensionScope.FRAMEWORK)
public interface ChannelHandler {

	// 连接建立
    void connected(Channel channel) throws RemotingException;

	// 连接断开
    void disconnected(Channel channel) throws RemotingException;

	// 消息发送
    void sent(Channel channel, Object message) throws RemotingException;

	// 消息接收
    void received(Channel channel, Object message) throws RemotingException;

	// 异常补货
    void caught(Channel channel, Throwable exception) throws RemotingException;
	
}
```

其中 received 事件根据消息的类型不同又分为:

- message instance of Request: 接收到请求消息
	
- message instance of Response: 接收到响应消息
	
- message instance of Event: 接收到 Event 消息

事件会在 Dubbo 的各个 ChannelHandler 中传递, 当抵达 Dispatcher ChannelHandler 后, 会执行线程切换, 上面提到, Dubbo 中的线程派发作用于请求处理流程, 所以 sent 事件没有相应的处理逻辑.

## 3	Dispatcher 接口与分发逻辑

### 3.1	接口定义

Dispatcher 是 Dubbo Remoting 层的重要接口, 负责消息处理的线程分发逻辑, 是一个 SPI 拓展点:

```java
// org.apache.dubbo.remoting.Dispatcher
@SPI(value = AllDispatcher.NAME, scope = ExtensionScope.FRAMEWORK)
public interface Dispatcher {
    @Adaptive({Constants.DISPATCHER_KEY, "dispather", "channel.handler"})
    ChannelHandler dispatch(ChannelHandler handler, URL url);
}
```

### 3.2	拓展与继承关系

```java
Dispatcher (com.alibaba.dubbo.remoting)
	ExecutionDispatcher (org.apache.dubbo.remoting.transport.dispatcher.execution)
	DirectDispatcher (org.apache.dubbo.remoting.transport.dispatcher.direct)
	MessageOnlyDispatcher (org.apache.dubbo.remoting.transport.dispatcher.message)
	AllDispatcher (org.apache.dubbo.remoting.transport.dispatcher.all)
```

Dubbo 提供了四种派发策略, 分别对应 Dispatcher 接口的一个实现类:

- All: 所有的消息都会被派发到线程池, 包括请求, 响应, 连接事件, 断开事件, 心跳等;
	
- Direct: 所有的消息在 IO 线程池上直接执行;
	
- Message: 只有请求和响应消息派发到线程池, 其他连接断开事件, 心跳等消息, 直接在 IO 线程上执行;
	
- Execution: 只有请求消息派发到线程池, 不含响应消息, 其他消息在线程池上直接执行;

### 3.3	源码案例分析

以 Dubbo 默认的派发策略实现类 AllDispatcher 为例, 理解派发策略是如何实现的:

```java
// org.apache.dubbo.remoting.transport.dispatcher.all.AllDispatcher
public class AllDispatcher implements Dispatcher {  

    public static final String NAME = "all";  
  
    @Override  
    public ChannelHandler dispatch(ChannelHandler handler, URL url) {  
        return new AllChannelHandler(handler, url);  
    }  
	
}
```

dispatch 方法返回一个 AllChannelHandler 实例, 先关注一下其继承关系和构造方法:

```java
AllChannelHandler(org.apache.dubbo.remoting.transport.dispatcher.all)
	WrappedChannelHandler (org.apache.dubbo.remoting.transport.dispatcher)
	    ChannelHandlerDelegate (org.apache.dubbo.remoting.transport)
	        ChannelHandler (org.apache.dubbo.remoting)

public class AllChannelHandler extends WrappedChannelHandler {
	public AllChannelHandler(ChannelHandler handler, URL url) {  
	    super(handler, url);  
	}
}

public interface ChannelHandlerDelegate extends ChannelHandler {  
    ChannelHandler getHandler();  
}
```

- ChannelHandlerDelegate 可以理解为一个标记接口, 标记其对于 ChannelHandler 的实现是委托给 getHandler 方法返回的 ChannelHandler 执行的;
	
- WrappedChannelHandler: 实现了 ChannelHandlerDelegate 的委托逻辑, 并且提供了事件派发和线程模型管理的能力, 在 WrappedChannelHandler 的实现中, 所有的方法都是在 **当前线程** 执行的.

---

AllChannelHandler 的入参是 handler 和 url, handler 提供给 WrappedChannelHandler 作为被委托对象, url 提供给 WrappedChannelHandler 用于确定线程池策略等.

```java
// org.apache.dubbo.remoting.transport.dispatcher.all.AllChannelHandler#connected
@Override
public void connected(Channel channel) throws RemotingException {
	ExecutorService executor = getSharedExecutorService();
	try {
		executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.CONNECTED));
	} catch (Throwable t) {
		throw new ExecutionException(
				"connect event", channel, getClass() + " error when process connected event .", t);
	}
}

// org.apache.dubbo.remoting.transport.dispatcher.all.AllChannelHandler#disconnected
@Override
public void disconnected(Channel channel) throws RemotingException {
	ExecutorService executor = getSharedExecutorService();
	try {
		executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.DISCONNECTED));
	} catch (Throwable t) {
		throw new ExecutionException(
				"disconnect event", channel, getClass() + " error when process disconnected event .", t);
	}
}

// org.apache.dubbo.remoting.transport.dispatcher.all.AllChannelHandler#received
@Override
public void received(Channel channel, Object message) throws RemotingException {
	ExecutorService executor = getPreferredExecutorService(message);
	try {
		executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.RECEIVED, message));
	} catch (Throwable t) {
		if (message instanceof Request && t instanceof RejectedExecutionException) {
			sendFeedback(channel, (Request) message, t);
			return;
		}
		throw new ExecutionException(message, channel, getClass() + " error when process received event .", t);
	}
}

// org.apache.dubbo.remoting.transport.dispatcher.all.AllChannelHandler#caught
@Override
public void caught(Channel channel, Throwable exception) throws RemotingException {
	ExecutorService executor = getSharedExecutorService();
	try {
		executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.CAUGHT, exception));
	} catch (Throwable t) {
		throw new ExecutionException("caught event", channel, getClass() + " error when process caught event .", t);
	}
}
```

AllDispatcher 实现的 connected, disconnected, received, caught 方法的逻辑均为通过 getSharedExecutorService 方法获取业务线程池, 然后将收到的消息封转成 ChannelEventRunnable 后, 派发给业务线程池执行;  

ChannelEventRunnable 会根据入参中的 ChannelState 来调用 handler 入参的对应方法, 一般情况下, 入参中的 handler 为 DecodeHandler 实例, 这也是一个 ChannelHandlerDelegate, 内部真正负责处理事件的是 HeaderExchangeHandler. 

---

根据 AllDispatcher, 猜测 MessageOnlyDispatcher 提供的 ChannelHandler 应该只重写覆盖了 received 方法, 其他处理逻辑交给父类执行, 也就是在当前线程 (IO 线程) 执行.

```java
// org.apache.dubbo.remoting.transport.dispatcher.message.MessageOnlyChannelHandler
public class MessageOnlyChannelHandler extends WrappedChannelHandler {
    public MessageOnlyChannelHandler(ChannelHandler handler, URL url) {
        super(handler, url);
    }

    @Override
    public void received(Channel channel, Object message) throws RemotingException {
        ExecutorService executor = getPreferredExecutorService(message);
        try {
            executor.execute(new ChannelEventRunnable(channel, handler, ChannelState.RECEIVED, message));
        } catch (Throwable t) {
            if (message instanceof Request && t instanceof RejectedExecutionException) {
                sendFeedback(channel, (Request) message, t);
                return;
            }
            throw new ExecutionException(message, channel, getClass() + " error when process received event .", t);
        }
    }
}
```

## 4	从 ChannelHandler 的角度看线程模型

上面的内容分析了当请求到达 Dispatcher 接口时执行的分发逻辑, 但是在这之前还有一系列操作; 本部分从 ChannelHandler 出发, 理清每个 ChannelHandler 是在哪个线程执行的.

### 4.1	Netty 初始化流程

 ```java
 // org.apache.dubbo.remoting.transport.netty4.NettyServer#initServerBootstrap
protected void initServerBootstrap(NettyServerHandler nettyServerHandler) {
	boolean keepalive = getUrl().getParameter(KEEP_ALIVE_KEY, Boolean.FALSE);
	bootstrap
			.group(bossGroup, workerGroup)
			.channel(NettyEventLoopFactory.serverSocketChannelClass())
			.option(ChannelOption.SO_REUSEADDR, Boolean.TRUE)
			.childOption(ChannelOption.TCP_NODELAY, Boolean.TRUE)
			.childOption(ChannelOption.SO_KEEPALIVE, keepalive)
			.childOption(ChannelOption.ALLOCATOR, PooledByteBufAllocator.DEFAULT)
			.childHandler(new ChannelInitializer<SocketChannel>() {
				@Override
				protected void initChannel(SocketChannel ch) throws Exception {
					int closeTimeout = UrlUtils.getCloseTimeout(getUrl());
					NettyCodecAdapter adapter = new NettyCodecAdapter(getCodec(), getUrl(), NettyServer.this);
					ch.pipeline().addLast("negotiation", new SslServerTlsHandler(getUrl()));
					ch.pipeline()
							.addLast("decoder", adapter.getDecoder())
							.addLast("encoder", adapter.getEncoder())
							.addLast("server-idle-handler", new IdleStateHandler(0, 0, closeTimeout, MILLISECONDS))
							.addLast("handler", nettyServerHandler);
				}
			});
} 
 ```

通过上述方法构造的 NettyServer 的结构大致为:

![[Dubbo NettyServer.png|850]]

可以得知 internalEncoder, internalDecoder 以及 IdleStateHandler 一定是在 IO 线程中执行的, 还需要分析一下 NettyServerHandler 的执行.

### 4.2	NettyServerHandler

NettyServerHandler 一般由 createNettyServerHandler 方法创建, 入参是 url 和 NettyServer 自身;

```java
// org.apache.dubbo.remoting.transport.netty4.NettyServer#createNettyServerHandler
protected NettyServerHandler createNettyServerHandler() {
	return new NettyServerHandler(getUrl(), this);
}
```

NettyServerHandler 中的部分方法是交给 NettyServer 执行的, 而 NettyServer 也维护了一个 ChannelHandler 来处理消息, NettyServer 中的 ChannelHandler 比较复杂, 大概是这样的:

- 最外部是一个 MultiMessageHandler, 是一个 ChannelHandlerDelegate;
	
- MultiMessageHandler 内部的 ChannelHandler 为 HeartbeatHandler, 同样是一个 ChannelHandlerDelegate;
	
- 最内部是 Dispatcher 接口提供的 ChannelHandler, 会通过 Dubbo SPI 获取 Dispatcher 动态拓展点, 然后调用其 dispatch 方法.

上述分析参照代码:

```java
// org.apache.dubbo.remoting.transport.netty4.NettyServer#NettyServer
public NettyServer(URL url, ChannelHandler handler) throws RemotingException {
	super(url, ChannelHandlers.wrap(handler, url));
}

// org.apache.dubbo.remoting.transport.dispatcher.ChannelHandlers#wrap 
public static ChannelHandler wrap(ChannelHandler handler, URL url) {  
    return ChannelHandlers.getInstance().wrapInternal(handler, url);  
}

// org.apache.dubbo.remoting.transport.dispatcher.ChannelHandlers#wrapInternal
protected ChannelHandler wrapInternal(ChannelHandler handler, URL url) {
	return new MultiMessageHandler(new HeartbeatHandler(url.getOrDefaultFrameworkModel()
			.getExtensionLoader(Dispatcher.class)
			.getAdaptiveExtension()
			.dispatch(handler, url)));
}
```

wrapInternal 方法的入参就是上面在 Dispatcher 部分提到的 DecodeHandler, 可以得出结论: Dispatcher 中使用的 ChannelHandler 来自 NettyServer.

由于 Dubbo 的派发逻辑是在 Dispatcher 提供的 ChannelHandler 执行的, 可以得知 NettyServerHandler, MultiMessageHandler, HeartbeatHandler 的逻辑均是在 IO 线程中执行的.

### 4.3	总结

通过上面的分析可以得出以下结论:

- 必定在 IO 线程内执行的 ChannelHandler 实现依次有 InternalEncoder, InternalDecoder, IdleStateHandler, NettyServerHandler, MultiMessageHandler, HeartbeatHandler, Dispatcher 返回的 Handler;
	
- 剩余需要执行的 Handler 有: DecodeHandler, HeaderExchangeHandler, DubboProtocol$requestHandler; 执行这个链路的线程会根据派发策略的不同而变化, 比如在 All 派发策略下, 上面的链路是由业务线程执行的.

## 5	Dubbo 线程池模型

上面的内容分析了 Dubbo 在何时将消息派发给任务线程池, Dubbo 根据不同的需求场景设计了不同的线程池, 具体来说是定义了 ThreadPool 接口, 来提供获取线程池的方法:

```java
@SPI(value = "fixed", scope = ExtensionScope.FRAMEWORK)
public interface ThreadPool {

    @Adaptive({THREADPOOL_KEY})
    Executor getExecutor(URL url);
	
}
```

Dubbo 业务线程池在客户端或服务端启动时调用 DefaultExecutorRepository#createExecutorIfAbsent 方法创建, 最终会调用到 ThreadPool.getExecutor 方法. 创建出来的线程池会存储在 DefaultExecutorRepository.data 中:

```java
// org.apache.dubbo.common.threadpool.manager.DefaultExecutorRepository#data
private final ConcurrentMap<String, ConcurrentMap<String, ExecutorService>> data = new ConcurrentHashMap<>();
```

Dubbo 中 Consumer 端共用一个线程池, Provider 端根据服务端口划分了多个线程池. 当消息传送到 Dispatcher.dispatch 方法返回的 Handler 时, 会将后续逻辑放到 WrappedChannelHandler#getPreferredExecutorService 方法得到的线程池中继续执行.

由此, 就可以得到 Dubbo 老的线程池模型. 大致为: 

![[Dubbo 老线程池模型.png|800]]

重点关注一下 Consumer 部分:

- 业务线程发出请求, 拿到一个 Future 实例;
	
- 业务线程调用 future.get 方法阻塞等待结果返回;
	
- 当结果返回后, 交给 Consumer 端线程池进行序列化等处理, 并调用 future.set 方法将最终结果放入 future 中;
	
- 业务侧线程池拿到结果返回.

这样导致即使是同步的请求方式, 也就是业务线程空闲阻塞的情况下, Dubbo 每次返回都需要从 Consumer 线程池中获取线程来执行复杂的序列化等步骤, 导致资源浪费, 并且 Consumer 端的线程池为 CachedThreadPool 对创建线程数没有限制, 最终导致了在高并发切服务端偶发性变慢 / 集中返回响应后, 瞬间创建大量消费者线程, 可能会导致 OOM, 具体详见:  [Need a limited Threadpool in consumer side #2013](https://github.com/apache/dubbo/issues/2013)

![[Dubbo 线程池模型 new.png|700]]

为了修复这个问题, Dubbo 将线程池模型修改成了如上的形式, 将反序列化等步骤交给阻塞的业务线程执行, 也就是在派发时将消息派发给业务线程.

具体到实现上, 当 Dubbo 发现某个请求是同步请求, 会将 Future 绑定的线程池设置为 ThreadlessExecutor:

```java
org.apache.dubbo.rpc.protocol.dubbo.DubboInvoker#doInvoke
	org.apache.dubbo.rpc.protocol.AbstractInvoker#getCallbackExecutor
	org.apache.dubbo.remoting.exchange.ExchangeChannel#request(java.lang.Object, int, java.util.concurrent.ExecutorService)

// org.apache.dubbo.rpc.protocol.AbstractInvoker#getCallbackExecutor
protected ExecutorService getCallbackExecutor(URL url, Invocation inv) {  
    if (InvokeMode.SYNC == RpcUtils.getInvokeMode(getUrl(), inv)) {  
	    //  同步使用 ThreadlessExecutor
        return new ThreadlessExecutor();  
    }  
    return ExecutorRepository.getInstance(url.getOrDefaultApplicationModel())  
            .getExecutor(url);  
}
```

业务线程调用 DubboInvoker.doInvoker 后, 会调用 ThreadlessExecutor#waitAndDrain 方法, 超时等待结果返回, 当结果返回后, 如果等待未超时, 则将后续步骤交给业务线程处理.

---

# 📚 参考内容

- 消费端线程模型, 提供端线程模型 - 官方文档: https://cn.dubbo.apache.org/zh-cn/overview/mannual/java-sdk/tasks/framework/threading-model/