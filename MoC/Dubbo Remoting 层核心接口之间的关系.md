---
type: MOC
banner: Assets/Banner/pexels-8kspain-21564213.jpg
---
[[Dubbo Remoting 层 Endpoint 接口|Endpoint]] 代表网络中的某一个端点, 它是**无状态**的, 回答了 "去哪里通信?" 的问题;

与 [[Dubbo Remoting 层 Endpoint 接口|Endpoint]] 不同, [[Dubbo Remoting 层 Channel 接口|Channel]] 是有状态的, 当两个端点之间建立起了连接, 双方会持有对方的一个 [[Dubbo Remoting 层 Channel 接口|Channel]], 是传输数据的动态管道, [[Dubbo Remoting 层 Channel 接口|Channel]] 伴随连接一起诞生;

[[Dubbo Remoting 层 ChannelHandler 接口|ChannelHandler]] 与 [[Dubbo Remoting 层 Channel 接口|Channel]] 绑定, 处理具体的业务逻辑以及协议逻辑, 回答了 "通信中可以做什么?" 的问题.