---
source: https://kaiwu.lagou.com/course/courseInfo.htm?courseId=393#/detail/pc?id=4261
type: input
banner: 附件/Banner/pexels-ken-cheung-3355734-5574638.jpg
---
zookeeper 是一个针对分布式系统的可靠的、可拓展的协调服务，它通常作为统一命名服务、统一配置管理、注册中心（分布式集群管理）、分布式锁服务、leader 选举服务等角色出现。本身也以集群的方式部署，有 client、leader、follower、observer 四种节点类型。

---

zookeeper 以树形结构存储数据，其中的节点称为 znode，每个 znode 可以有自己的子节点，类似文件系统结构。

znode 节点类型有如下四种：

- 持久节点。 持久节点创建后，会一直存在，不会因创建该节点的 client 会话失效而删除。
- 持久顺序节点。 持久顺序节点的基本特性与持久节点一致，创建节点的过程中，zookeeper 会在其名字后自动追加一个单调增长的数字后缀，作为新的节点名。
- 临时节点。 创建临时节点的 zooKeeper client 会话失效之后，其创建的临时节点会被 zookeeper 集群自动删除。与持久节点的另一点区别是，临时节点下面不能再创建子节点。
- 临时顺序节点。 基本特性与临时节点一致，创建节点的过程中，zookeeper 会在其名字后自动追加一个单调增长的数字后缀，作为新的节点名。

---

每个 znode 会有一个 stat 结构，记录这个节点的元数据包括版本号、ACL、时间戳、数据长度等。

---

除了可以通过 zookeeper client 来对 znode 进行操作以外，还可以通过 zookeeper 的 watcher 机制，来实现对 znode 节点本身以及其子节点的监控。

zookeeper watcher 有如下的特点：

- **主动推送**： watcher 被触发时，由 zookeeper 集群主动将更新推送给客户端，而不需要客户端轮询；
- **一次性**：数据变化时，watcher 只会被触发一次。如果客户端想得到后续更新的通知，必须要在 watcher 被触发后重新注册一个 watcher；
- **可见性**：客户端不会在收到 watcher 消息之前读到新数据，也就是推送消息优先于更新数据；
- **顺序性**：如果多个更新触发了多个 watcher ，那 watcher 被触发的顺序与更新顺序一致。

---

zookeeper 官方提供的客户端存在较多使用问题，一般选用第三方开源的客户端，常见的有 zkclient 和 apache curator；其中 zkclient 还存在一些问题，如文档不全，重试机制难用等，故一般选择 apache curator 作为 zookeeper 客户端。

apache curator 是 apache 基金会提供的一套 java zookeeper 客户端实现，提供了一套易用性和可读性非常强的 fluent 风格的 api。

apache curator 提供了如下的 jar 包：

- curator-framework：zookeeper api 的高层封装，简化 zookeeper 的客户端编程，添加了 zookeeper 连接管理、重试机制、重复注册 watcher 等功能；
- curator-recipes：zookeeper 的典型应用场景的实现，如 leader 选举、分布式锁、barrier 分布式队列实现等，基于 curator-framework 包；
- curator-client：zookeeper client 的封装，用于取代原生的 zookeeper 客户端；
- curator-x-discovery：基于 curator-framework 构建的服务发现功能；
- curator-x-discoveryserver：和 curator-x-discovery 一起使用的 restful server；
- curator-examples：展示 curator 的使用案例。



