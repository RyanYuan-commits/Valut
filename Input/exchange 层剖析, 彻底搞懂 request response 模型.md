---
source: https://o.alldu.cn/docs/dubbo%E6%BA%90%E7%A0%81%E8%A7%A3%E8%AF%BB%E4%B8%8E%E5%AE%9E%E6%88%98-%E5%AE%8C/21--exchange-%E5%B1%82%E5%89%96%E6%9E%90%E5%BD%BB%E5%BA%95%E6%90%9E%E6%87%82-request-response-%E6%A8%A1%E5%9E%8B%E4%B8%8A/
date: 2025-11-21
type: input
---
# 阅读笔记

Exchange 层是 Dubbo Remoting 层的最顶层

Dubbo 将信息的 **交换行为** 抽象成 Exchange 层, Exchange 层以 Request 和 Response 为核心, 针对 Channel, ChannelHandler, CLient, RemotingServer 等接口进行了实现.

## 1	Request & Response

Exchange 的核心对象

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

---

# 我的思考

这个观点如何与我已知的知识产生联系? 它让我想到了什么?