---
source: https://o.alldu.cn/docs/dubbo%E6%BA%90%E7%A0%81%E8%A7%A3%E8%AF%BB%E4%B8%8E%E5%AE%9E%E6%88%98-%E5%AE%8C/21--exchange-%E5%B1%82%E5%89%96%E6%9E%90%E5%BD%BB%E5%BA%95%E6%90%9E%E6%87%82-request-response-%E6%A8%A1%E5%9E%8B%E4%B8%8A/
created: 2025-11-21
type: input
---
### 2.2	关键字段

- request (Request 类型) 和 id (Long 类型): 对应请求以及请求的 ID. 
	
- channel (Channel 类型): 发送请求的 Channel. 
	
- timeout (int 类型): 
	
- start (long 类型): 该 DefaultFuture 的创建时间. 
	
- sent (volatile long 类型): 请求发送的时间. 
	
- timeoutCheckTask (Timeout 类型): 该定时任务到期时，表示对端响应超时. 
	
- executor (ExecutorService 类型): 请求关联的线程池. 

通过 DefaultFuture.newFuture 创建 Default 实例时, 会初始化上述字段, 并创建请求响应的超时任务.

### 2.3	请求与响应流程

在 HeaderExchangeChannel 方法中完成 DefaultFuture 创建后, 会将请求通过底层的 Dubbo Channel 发送出去, 发送过程中会触发 HeaderExchangeHandler 的 sent 方法, 在这个方法中会更新 sent 字段, 记录发送时间戳, 后续如果出现超时, 会在提示信息中展示该时间戳.

过一段时间后, Consumer 端回收到对端的完整响应, 会根据线程模型来决定是否将后续的 received 方法提交给业务线程池执行 (也就是 DefaultFuture 中的 executor);

当响应传递到 HeaderExchangeHandler 后, 会调用 handleResponse 方法处理, 在这个方法中调用了 DefaultFuture 的 received 方法, 这个方法会找到与这次请求相关联的 DefaultFuture 对象, 然后调用其 doReceived 方法, 将 DefaultFuture 设置为完成状态.

### 2.4	超时响应检测

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

## 3	HeaderExchangeHandler

### 3.1	与上层交互的关键类

HeaderExchangeHandler 是 Exchange 层与上层交互的关键类之一: 

- 上层将自己的实现逻辑封装到 ExchangeHandler 的实现中;
	
- 使用 HeaderExchangeHandler 包装 ExchangeHandler 实例;
	
- HeaderExchangeHandler 本身会作为一个 ChannelHandler 作为传递给 Transport 层的参数.

HeaderExchangeHandler 的 connected, disconnected, sent, received, caught 方法都会转发给上层提供的 ExchangeHandler 来处理.

### 3.2	received 方法

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
```

received 方法对收到的消息做了分类处理:

- 只读请求会由 handlerEvent 来处理, 它会在 Channel 上设置 READONLY 标志, 后续上层调用会读取这个标志;
	
- 双向请求由 handleRequest 方法处理, 将正常解码的请求交给上层 ExchangeHandler 处理, 并添加回调, 即在处理完成后填充结果和响应码, 将其发送到对端;
	
- 单向请求则直接委托给上层的 ExchangeHandler 处理, 不需要关注处理结果;
	
- Response 则会通过 handleResponse 处理, 根据返回结果来改变 DefaultFuture 的状态;
	
- 对于 String 类消息, 会根据服务角色不同分类, 具体与 Dubbo 支持 telnet 有关.

 
---