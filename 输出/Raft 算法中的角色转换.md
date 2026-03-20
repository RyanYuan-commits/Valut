---
type: permanent
banner: 附件/Banner/pexels-picjumbo-com-55570-4457409.jpg
---
---

**关键词**: Raft

---

**leader 转化为 follower**：当 leader 发现系统中出现了更大的任期，会自动“禅让”转化为 follower 节点，leader 发现出现了更大任期的方式有三种：

- 与 follower 同步日志时，从 follower 的返回中获取；
- 接收到新的 leader 节点发送同步请求时；
- 收到了任期更大的 candidate 节点的拉票请求。

**follower 转化为 candidate**：当 follower 在一定时间内没有没有接收到 master 的心跳，会认为 master 已经挂掉，以当前任期加一作为自己的任期，变为 candidate 参与竞选。

**candidate 转化为 follower**：当收到多数派的反对票或者收到了任期更大的 leader 的请求，candidate 会在竞选期间转回 follower。

**candidate 转化为 leader**：在 candidate 竞选时，若多数派投了赞成票，则 candidate 会晋升为 leader。

**candidate 转化为 candidate**：candidate 的竞选有一个时间阈值，若在这个时间内多数派没有达成共识，则在当前任期的基础上加一，开始下一次竞选。

---