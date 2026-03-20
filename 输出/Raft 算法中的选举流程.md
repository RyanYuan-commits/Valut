---
type: permanent
banner: 附件/Banner/pexels-vadimsadovski-33522604-7049988.jpg
---
---

**关键词**: Raft、选举、心跳、竞选

---

## 1	心跳 & 提交同步请求

leader 需要周期性的向 follower 节点发送心跳请求，证明自己还健在，同时同步日志提交的进度，心跳请求是单向无阻塞的，leader 无需等待回复。

```json
{
	"term": 5,
	"leaderID": 1,
	"leaderCommit": 10
}
```

接收到心跳请求的一方如果发现请求的任期比自己当前的任期要小，直接无视，反正，则转化为 follower 节点，重置 leader 心跳检测计时器，并且查看 leaderCommit，查看是否有日志可以被提交。

## 2	竞选拉票流程

当 follower 在一段时间内没有收到 leader 节点的同步请求后，会转变为 candidate 参与竞选；candidate 首先会在当前 term 的基础上加一，作为新的任期标识，然后将自己的一票投给自己，随后广播向其他 follower 节点争取选票。

如果在拉票请求超时前得到了多数派认可，则晋升为 leader，反之则退回 follower；而如果在超时前没有完成竞选，则将当前任期加一，再次参与竞选，为了避免陷入死循环，两次竞选期间需要进行一次随机时间的暂停。

---

```json
{
  // candidate 发起竞选的任期号，如果选举成功，集群将采用此任期
  "term": 7,

  // candidate 的节点唯一标识，follower 记录此 ID 表示自己已将票投给该候选人
  "candidateID": "node-3",

  // candidate 本地日志中最新条目的索引，用于 follower 判断 candidate 的日志是否足够新
  "lastLogIndex": 15,

  // candidate 本地日志中最新条目的任期号，与 lastLogIndex 一起决定日志的完整性和新旧程度
  "lastLogTerm": 6
}
```

```json
{
	// 节点当前的任期号
	"term": 7,
	
	// 是否投了赞成票
	"granted": true
}
```

---

leader 节点或者 candidate 节点收到拉票请求时，若发现对方的任期小于自己的任期，会拒绝然后回复自己的任期，而如果对方的任期更大，则退位为 follower 处理；

如果处理终点为 follower，且当前其没有给其他 candidate 投票，follower 会通过请求中的 lastLogIndex + lastLogTerm 来检查对方的日志是否**滞后**于自己，如果没有，则投出赞成票。

---