---
type: permanent
banner: Assets/Banner/pexels-jeremy-bishop-1260133-2524874.jpg
---
---

**关键词**: Dubbo, Exchange, 接口设计

---

`HeaderExchangeHandler` 是 `Exchange` 层**与上层交互**的关键类, 上层将自己的逻辑封装成一个 `ExchangeHandler` 实例, 然后使用 `HeaderExchangeHandler` 包装.

```java
// org.apache.dubbo.remoting.exchange.support.header.HeaderExchangeHandler#received
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

`received` 方法对收到的消息做了分类处理:

- 只读请求会由 `handlerEvent()` 方法处理, 它会在 `Channel` 上设置 `READONLY` 标志, 后续上层调用会读取这个标志;
	
- 双向请求由 `handleRequest` 方法处理, 将正常解码的请求交给上层 `ExchangeHandler` 处理, 并添加回调, 即在处理完成后填充结果和响应码, 将其发送到对端;
	
- 单向请求则直接委托给上层的 `ExchangeHandler` 处理, 不需要关注处理结果;
	
- `Response` 则会通过 `handleResponse` 处理, 根据返回结果来改变 `DefaultFuture` 的状态;
	
- 对于 `String` 类消息, 会根据服务角色不同分类, 具体与 Dubbo 支持 `telnet` 有关.

---