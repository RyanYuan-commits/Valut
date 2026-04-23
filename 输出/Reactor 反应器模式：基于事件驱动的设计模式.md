---
type: permanent
banner: 附件/Banner/pexels-erike-fusiki-58866350-8150291.jpg
---
## 1	Reactor 模式中 IO 时间的处理流程

- **通道注册**：IO 事件源于 channel，一个 IO 事件一定属于某个通道；如果要查询 channel 的事件，需要将 channel 注册到 selector 上；
- **查询事件**：在 Reactor 模式中，一个线程会负责一个选择器，不断的轮询，查询选择器中发生的 IO 事件；
- **事件分发**：如果线程查询到 IO 事件，分发给与 IO 事件有绑定关系的 handler 处理；
- **事件处理**：由 handler 业务处理器负责。

## 2	Netty 的 Reactor 模型

netty 对经典的 reactor 模式进行了细微的调整：

netty 对 nio 的 selector 组件和线程实例进行了封装，设计了自己的 reactor 角色，称为 event loop；同时，其拓展了 nio 的 channel，设计了 netty channel；通道注册指的是将 netty channel 注册到 event loop 上，对应到底层就是 nio channel 注册到 nio selector 上；



