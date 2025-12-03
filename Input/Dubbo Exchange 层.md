---
source: https://o.alldu.cn/docs/dubbo%E6%BA%90%E7%A0%81%E8%A7%A3%E8%AF%BB%E4%B8%8E%E5%AE%9E%E6%88%98-%E5%AE%8C/21--exchange-%E5%B1%82%E5%89%96%E6%9E%90%E5%BD%BB%E5%BA%95%E6%90%9E%E6%87%82-request-response-%E6%A8%A1%E5%9E%8B%E4%B8%8A/
created: 2025-11-21
type: input
---
# 阅读笔记

Exchange 层是 Dubbo Remoting 层的最顶层, Dubbo 将信息的 **交换行为** 抽象成 Exchange 层, Exchange 层以 Request 和 Response 为核心, 针对 Channel, ChannelHandler, CLient, RemotingServer 等接口进行了实现.

[[Dubbo Remoting 层核心类与接口]]

## 1	请求与返回类

### 1.1	Request 核心字段

```java
public class Request {

	// 用于生成请求的自增ID，当递增到Long.MAX_VALUE之后，会溢出到Long.MIN_VALUE，我们可以继续使用该负数作为消息ID
	private static final AtomicLong INVOKE_ID = new AtomicLong(0);
	
	private final long mId; // 请求的ID
	
	private String mVersion; // 请求版本号
	
	// 请求的双向标识，如果该字段设置为true，则Server端在收到请求后，
	// 需要给Client返回一个响应
	private boolean mTwoWay = true; 
	
	// 事件标识，例如心跳请求、只读请求等，都会带有这个标识
	private boolean mEvent = false; 
	
	// 请求发送到Server之后，由Decoder将二进制数据解码成Request对象，
	// 如果解码环节遇到异常，则会设置该标识，然后交由其他ChannelHandler根据
	// 该标识做进一步处理
	private boolean mBroken = false;
	
	// 请求体，可以是任何Java类型的对象,也可以是null
	private Object mData; 

}
```

### 1.2	Response 核心字段

```java
public class Response {
	
	// 响应ID，与相应请求的ID一致
	private long mId = 0;
	
	// 当前协议的版本号，与请求消息的版本号一致
	private String mVersion;
	
	// 响应状态码，有OK、CLIENT_TIMEOUT、SERVER_TIMEOUT等10多个可选值
	private byte mStatus = OK; 
	
	private boolean mEvent = false;
	
	private String mErrorMsg; // 可读的错误响应消息
	
	private Object mResult; // 响应体

}
```

## 2	ExchangeChannel

### 2.1	ExchangeChannel 接口

```java
public interface ExchangeChannel extends Channel {

	// ...... 废弃接口 ......

    CompletableFuture<Object> request(Object request, ExecutorService executor) throws RemotingException;

    CompletableFuture<Object> request(Object request, int timeout, ExecutorService executor) throws RemotingException;

    ExchangeHandler getExchangeHandler();

    @Override
    void close(int timeout);
	
}
```

ExchangeChannel 继承了 Channel 接口, 并在此基础上, 抽象了 Exchange 层的网络连接; 在 Exchange 层完成了同步和异步的转换, request 方法的两个重载返回值均为 CompletableFuture. 

### 2.2	HeaderExchangeChannel

```java
@Override  
public void send(Object message, boolean sent) throws RemotingException {  
    if (closed) {  
        throw new RemotingException(  
                this.getLocalAddress(),  
                null,  
                "Failed to send message " + message + ", cause: The channel " + this + " is closed!");  
    }  

	// 只处理 Request, Response 和 String
    if (message instanceof Request || message instanceof Response || message instanceof String) {  
        channel.send(message, sent);  
    } else {  
        Request request = new Request();  
        request.setVersion(Version.getProtocolVersion());  
        request.setTwoWay(false);  
        request.setData(message);  
        channel.send(request, sent);  
    }  
}
```

HeaderExchangeChannel 本质上是一个 Channel 的装饰器, 关键方法是委托给这个 Channel 对象完成的.

```java
@Override  
public CompletableFuture<Object> request(Object request, int timeout, ExecutorService executor)  
        throws RemotingException {  
    if (closed) {  
        throw new RemotingException(  
                this.getLocalAddress(),  
                null,  
                "Failed to send request " + request + ", cause: The channel " + this + " is closed!");  
    }  

	// 构建 Request 对象
    Request req;  
    if (request instanceof Request) {  
        req = (Request) request;  
    } else {  
        // create request.  
        req = new Request();  
        req.setVersion(Version.getProtocolVersion());  
        req.setTwoWay(true);  
        req.setData(request);  
    }  

	// 返回值是一个 DefaultFuture	
    DefaultFuture future = DefaultFuture.newFuture(channel, req, timeout, executor);  
    try {  
        channel.send(req);  
    } catch (RemotingException e) {  
        future.cancel();  
        throw e;  
    }  
    return future;  
}
```

request 方法返回的是一个 DefaultFuture 对象, 其作用和 Netty 的 ChannelFuture 类似, 可以获取某个操作的完成状态, 等收到响应后, DefaultFuture 才算完成.

## 3	DefaultFuture

### 3.1	核心集合

DefaultFuture 继承了 JDK 中的 CompletableFuture, 在其中维护了两个 static Map:

- CHNNELS: 负责请求与 Channel 之间的关联关系, 其中 Key 为请求 ID, Value 为发送请求的 Channel;
	
- FUTURES: 负责请求与 DefaultFuture 之间的关联关系, Key 为请求 ID, Value 为请求对应的 Default Future.

### 3.2	关键字段

- request (Request 类型) 和 id (Long 类型): 对应请求以及请求的 ID. 
	
- channel (Channel 类型): 发送请求的 Channel. 
	
- timeout (int 类型): 整个请求-响应交互完成的超时时间. 
	
- start (long 类型): 该 DefaultFuture 的创建时间. 
	
- sent (volatile long 类型): 请求发送的时间. 
	
- timeoutCheckTask (Timeout 类型): 该定时任务到期时，表示对端响应超时. 
	
- executor (ExecutorService 类型): 请求关联的线程池. 

通过 DefaultFuture.newFuture 创建 Default 实例时, 会初始化上述字段, 并创建请求响应的超时任务.

### 3.3	请求与响应流程

在 HeaderExchangeChannel 方法中完成 DefaultFuture 创建后, 会将请求通过底层的 Dubbo Channel 发送出去, 发送过程中会触发 HeaderExchangeHandler 的 sent 方法, 在这个方法中会更新 sent 字段, 记录发送时间戳, 后续如果出现超时, 会在提示信息中展示该时间戳.

过一段时间后, Consumer 端回收到对端的完整响应, 会根据线程模型来决定是否将后续的 received 方法提交给业务线程池执行 (也就是 DefaultFuture 中的 executor);

当响应传递到 HeaderExchangeHandler 后, 会调用 handleResponse 方法处理, 在这个方法中调用了 DefaultFuture 的 received 方法, 这个方法会找到与这次请求相关联的 DefaultFuture 对象, 然后调用其 doReceived 方法, 将 DefaultFuture 设置为完成状态.

### 3.4	超时响应检测

在创建 DefaultFuture 时, 会调用 timeoutCheck 方法创建 TimeoutCheckTask 定时任务, 并添加到时间轮中:

```java
private static void timeoutCheck(DefaultFuture future) {  
    TimeoutCheckTask task = new TimeoutCheckTask(future.getId());  
    future.timeoutCheckTask = TIME_OUT_TIMER.get().newTimeout(task, future.getTimeout(), TimeUnit.MILLISECONDS);  
}
```

TIME_OUT_TIMER 是一个用于存放超时检查任务的时间轮, 是 DefaultFuture 的 static 属性;

TimeoutCheckTask 是 DefaultFuture 的内部类, 实现了 TimerTask 接口, 可以提交到时间轮中执行, 当响应超时后, 会调用 DefaultFuture 的 received 方法.

```java
private void notifyTimeout(DefaultFuture future) {  
    // create exception response.  
    Response timeoutResponse = new Response(future.getId());  
    // set timeout status.  
    timeoutResponse.setStatus(future.isSent() ? Response.SERVER_TIMEOUT : Response.CLIENT_TIMEOUT);  
    timeoutResponse.setErrorMessage(future.getTimeoutMessage(true));  
    // handle response.  
    DefaultFuture.received(future.getChannel(), timeoutResponse, true);  
}
```

## 4	HeaderExchangeHandler

### 4.1	与上层交互的关键类

HeaderExchangeHandler 是 Exchange 层与上层交互的关键类之一: 

- 上层将自己的实现逻辑封装到 ExchangeHandler 的实现中;
	
- 使用 HeaderExchangeHandler 包装 ExchangeHandler 实例;
	
- HeaderExchangeHandler 本身会作为一个 ChannelHandler 作为传递给 Transport 层的参数.

HeaderExchangeHandler 的 connected, disconnected, sent, received, caught 方法都会转发给上层提供的 ExchangeHandler 来处理.

### 4.2	received 方法

```java
@Override
public void received(Channel channel, Object message) throws RemotingException {
	final ExchangeChannel exchangeChannel = HeaderExchangeChannel.getOrAddChannel(channel);
	if (message instanceof Request) {
		// handle request.
		Request request = (Request) message;
		if (request.isEvent()) {
			handlerEvent(channel, request);
		} else {
			if (request.isTwoWay()) {
				handleRequest(exchangeChannel, request);
			} else {
				handler.received(exchangeChannel, request.getData());
			}
		}
	} else if (message instanceof Response) {
		handleResponse(channel, (Response) message);
	} else if (message instanceof String) {
		if (isClientSide(channel)) {
			Exception e = new Exception("Dubbo client can not supported string message: " + message
					+ " in channel: " + channel + ", url: " + channel.getUrl());
			logger.error(TRANSPORT_UNSUPPORTED_MESSAGE, "", "", e.getMessage(), e);
		} else {
			String echo = handler.telnet(channel, (String) message);
			if (StringUtils.isNotEmpty(echo)) {
				channel.send(echo);
			}
		}
	} else {
		handler.received(exchangeChannel, message);
	}
}
# ```

received 方法对收到的消息做了分类处理:

- 只读请求会由 handlerEvent 来处理, 它会在 Channel 上设置 READONLY 标志, 后续上层调用会读取这个标志;
	
- 双向请求由 handleRequest 方法处理, 将正常解码的请求交给上层 ExchangeHandler 处理, 并添加回调, 即在处理完成后填充结果和响应码, 将其发送到对端;
	
- 单向请求则直接委托给上层的 ExchangeHandler 处理, 不需要关注处理结果;
	
- Response 则会通过 handleResponse 处理, 根据返回结果来改变 DefaultFuture 的状态;
	
- 对于 String 类消息, 会根据服务角色不同分类, 具体与 Dubbo 支持 telnet 有关.

 
---

# 我的思考

这个观点如何与我已知的知识产生联系? 它让我想到了什么?