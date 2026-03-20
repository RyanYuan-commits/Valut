---
type: permanent
banner: 附件/Banner/pexels-walidphotoz-1509582.jpg
---
---

**关键词**: Raft

---

## 1	写请求流程

写操作需要 leader 节点统一收口，如果 follower 节点接收到了写请求，会告知客户端 leader 节点的节点 id，让客户端重新将写请求发送给 leader 处理；

leader 接收写请求后，会将其抽象为一个预写日志，追加到预写日志数组的结尾；随后，leader 节点会广播向集群中所有节点发送同步这笔日志的请求，称之为第一阶段的 proposal；

follower 将日志同步到本机的预写日志数组后，会给 leader 回复一个“同步成功”的 ack；

leader 发现这笔请求对应的预写日志已经被集群中的[[Raft 算法通过多数派原则来保障系统的一致性和可用性|多数派]]（包括自身）完成同步时，会”提交“该日志，并向客户端回复写请求成功。

- 倘若某个节点**拒绝**了同步请求，但回复了相同的任期，leader 会递归发送前一条日志给该节点，直到其接受同步请求为止；
- 倘若一个节点**超时**未给 leader 回复，leader 会重发这笔同步日志的请求。

## 2	日志同步请求

### 2.1	请求与响应字段

```json
{
  // 领导者的任期号，用于检测领导者是否过期
  "term": 5,
  
  // 领导者的节点 ID，follower 需要知道向谁回复以及转发写请求
  "leaderID": "node-1",
  
  // 领导者已提交的最高日志索引，follower 据此更新自己的 commitIndex
  "leaderCommit": 10,
  
  // 新日志条目之前那条日志的索引，用于日志一致性检查
  "prevLogIndex": 7,
  
  // 新日志条目之前那条日志的任期号，与 prevLogIndex 一起校验日志连续性
  "prevLogTerm": 4,
  
  // 需要同步给 follower 的日志条目数组（可能包含多条，用于批量追赶）
  "log": [
    {
      "index": 8,
      "term": 5,
      "command": "SET x 100"
    },
    {
      "index": 9,
      "term": 5,
      "command": "SET y 200"
    },
    {
      "index": 10,
      "term": 5,
      "command": "DEL z"
    }
  ]
}
```

```json
{
	// 节点当前任期
	"term": 5,

	// 同步日志的请求是否成功
	"success": true
}
```

### 2.2	针对请求终点的分类讨论

请求终点为 leader

- 如果发现自己的任期大于对方的任期，拒绝，并回复自己的任期；
- 如果发现请求的任期大于自己的任期，自动退位为 follower，按照 follower 的模式处理该请求

请求终点为 follower

- 发现请求的任期滞后，拒绝并返回自己的任期；
- 如果发现自己有多余的日志，删掉以保持和当前任期的 leader 的同步；
- 倘若 follower 存在日志滞后，则拒绝请求，让 leader 重发更早的日志，直到补齐所有缺失。
- 
请求终点为 candidate

- 发现自己的任期滞后，退化为 follower 处理；
- 如果 leader 任期小于自己，拒绝，并回复自己的最新任期。


---