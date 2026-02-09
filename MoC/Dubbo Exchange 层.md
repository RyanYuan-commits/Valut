---
type: MOC
banner: Assets/Banner/pexels-faikackmerd-1025469.jpg
---
Exchange 层是 Remoting 层的最顶层, 是对信息的**交换行为**的抽象, Exchange 层以 [[Dubbo Exchange 层 Request 类字段解析|Request]] 和 [[Dubbo Exchange 层 Response 类字段解析|Response]] 为核心, 针对 Channel, ChannelHandler, CLient, RemotingServer 等接口进行了实现.