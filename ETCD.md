# ETCD

### Raft

##### **Raft算法概述**

>  **Raft是一个用于管理日志一致性的协议**。它将分布式一致性分解为多个子问题：**Leader选举（Leader election）、日志复制（Log replication）、安全性（Safety）、日志压缩（Log compaction）等**。同时，Raft算法使用了更强的假设来减少了需要考虑的状态，使之变的易于理解和实现。**Raft将系统中的角色分为领导者（Leader）、跟从者（Follower）和候选者**（Candidate）：

- [ ] Leader：**接受客户端请求，并向Follower同步请求日志，当日志同步到大多数节点上后告诉Follower提交日志**。

- [ ] Follower：**接受并持久化Leader同步的日志，在Leader告之日志可以提交之后，提交日志**。
- [ ] Candidate：**Leader选举过程中的临时角色**



![20190722215348483](D:\图\学习\20190722215348483.jpg)

 Raft要求系统在任意时刻最多只有一个Leader，正常工作期间只有Leader和Followers。Raft算法将时间分为一个个的**任期（term）**，每一个term的开始都是Leader选举。在成功选举Leader之后，Leader会在整个term内管理整个集群。如果Leader选举失败，该term就会因为没有Leader而结束。




##### **Raft分为哪几个部分**

- [ ] leader选举
- [ ] 日志复制
- [ ] 日志压缩
- [ ] 成员变更

##### **Term**

- [ ] **Raft 算法将时间划分成为任意不同长度的任期（term）**。任期用连续的数字进行表示。**每一个任期的开始都是一次选举（election），一个或多个候选人会试图成为领导人**。如果一个候选人赢得了选举，它就会在该任期的剩余时间担任领导人。在某些情况下，选票会被瓜分，有可能没有选出领导人，那么，将会开始另一个任期，并且立刻开始下一次选举。**Raft 算法保证在给定的一个任期最多只有一个领导人**

##### **RPC**

- [ ] Raft 算法中服务器节点之间通信使用**远程过程调用（RPC）**，并且基本的一致性算法只需要两种类型的 RPC，为了在服务器之间传输快照增加了第三种 RPC

RPC有三种

- [ ] **RequestVote RPC**：**候选人在选举期间发起**。
- [ ] **AppendEntries RPC**：**领导人发起的一种心跳机制，复制日志也在该命令中完成**。
- [ ] **InstallSnapshot RPC**: 领导者使用该RPC来**发送快照给太落后的追随者**。



##### **Leader选举的过程**

- [ ] **Raft 使用心跳（heartbeat）触发Leader选举**。当服务器启动时，初始化为Follower。**Leader**向所有**Followers**周期性发送**heartbeat**。**如果Follower在选举超时时间内没有收到Leader的heartbeat，就会等待一段随机的时间后发起一次Leader选举**
- [ ] **每一个follower都有一个时钟，是一个随机的值，表示的是follower等待成为leader的时间，谁的时钟先跑完，则发起leader选举**
- [ ] **Follower将其当前term加一然后转换为Candidate。它首先给自己投票并且给集群中的其他服务器发送 RequestVote RPC**。结果有以下三种情况：
  * 赢得了多数的选票，成功选举为Leader；
  * 收到了Leader的消息，表示有其它服务器已经抢先当选了Leader；
  * 没有服务器赢得多数的选票，Leader选举失败，等待选举时间**超时后发起下一次选举**。



##### **Raft中任何节点都可以发起选举吗**

Raft发起选举的情况有如下几种：

- [ ] `刚启动时`，所有节点都是follower，这个时候发起选举，选出一个leader
- [ ] `当leader挂掉后`，时钟最先跑完的follower发起重新选举操作，选出一个新的leader
- [ ] `成员变更`的时候会发起选举操作



##### **Leader选举的限制**

- [ ] 能被选举成为Leader的节点，一定包含了所有已经提交的日志条目
  * 这个保证是在RequestVoteRPC阶段做的，candidate在发送RequestVoteRPC时，会带上自己的last log entry的term_id和index，follower在接收到RequestVoteRPC消息时，如果发现自己的日志比RPC中的更新，就拒绝投票

##### Raft数据一致性如何实现

>  主要是`通过日志复制实现数据一致性`，leader将请求指令作为一条新的日志条目添加到日志中，然后发起RPC 给所有的follower，进行日志复制，进而同步数据。

##### Raft的日志有什么特点

> 日志由`有序编号（log index）的日志条目组成`，每个日志条目包含它被创建时的`任期号（term）`和`用于状态机执行的命令`

##### 日志复制的过程

##### Raft日志压缩是怎么实现的

- [ ] Raft采用对整个系统进行snapshot来解决，snapshot之前的日志都可以丢弃

