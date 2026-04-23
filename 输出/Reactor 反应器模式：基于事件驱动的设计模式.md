---
type: permanent
banner: 附件/Banner/pexels-erike-fusiki-58866350-8150291.jpg
---
在 Reactor 模式出现之前，传统的服务器架构面临一个致命瓶颈：**资源诅咒**。当客户端连接数成千上万时，系统会因为维护过多的空闲线程而消耗大量内存，且 CPU 大部分时间都在进行低效的“线程上下文切换”，而不是处理真正的业务逻辑。Reactor 模式的诞生，就是为了解决 “如何用最少的线程，最高效地管理海量的并发连接” 这一问题。它将 “**连接接入**” 与 “**业务处理**” 解耦，让服务器从繁琐的等待中解放出来。

## 1	核心构成

==Reactor==：核心中枢，负责监听和分发事件（连接请求、读数据、写数据）；

==Handlers==：执行实际的业务逻辑（解码、计算、编码）。

==Acceptor==：Reactor 的子集，专门负责处理新的连接请求并建立连接。

## 2	运作机制

**通道注册**：IO 事件源于 channel，一个 IO 事件一定属于某个通道；如果要查询 channel 的事件，需要将 channel 注册到 selector 上；

**查询事件**：在 Reactor 模式中，一个线程会负责一个选择器，不断的轮询，查询选择器中发生的 IO 事件；

**事件分发**：如果线程查询到 IO 事件，分发给与 IO 事件有绑定关系的 handler 处理；

**事件处理**：由 handler 业务处理器负责。

## 3	Netty 的 Reactor 模型

netty 对经典的 reactor 模式进行了细微的调整：

netty 对 nio 的 selector 组件和线程实例进行了封装，设计了自己的 reactor 角色，称为 event loop；同时，其拓展了 nio 的 channel，设计了 netty channel；通道注册指的是将 netty channel 注册到 event loop 上，对应到底层就是 nio channel 注册到 nio selector 上；



