## ZAB 协议是什么

Zookeeper 集群是一个基于主从复制的高可用集群架构，每个服务器担任如下三个角色中的一种：

1. Leader: 一个 Zookeeper 集群同一时间只能有一个实际工作的 Leader，它会发起并维护与各 Follower 和 Observer 间的心跳。所有的写都必须通过 Leader 完成再由 Leader 广播到其他服务器。
1. Follower: 一个 Zookeeper 集群同一时间可以有多个 Follower，它会响应 Leader 的心跳。Follower 也可以直接处理和返回客户端的读请求，同时会将写请求转发给 Leader 进行处理。
1. Observer: 和 Follower 类似，但没有投票权。

但主从复制的架构，往往存在主节点单点故障的风险。同时，如何通过主节点将数据有效的同步到从节点，这也是一个问题。

ZAB 协议(Zookeeper Atomic Broadcast)是一个分布式最终一致性协议。它主要就是为了解决这两个问题，它具有以下两个主要的作用：

1. 原子广播
1. 崩溃恢复

## 原子广播

1. Leader 生成提议并广播给 Followers。
1. Followers 收到提议后，先将事务写入到本地 **事务日志** ，然后反馈 ACK。
1. Leader 收到一半以上的 ACK 后，广播  commit 消息，同时也会在本地执行 commit 并向连接客户端返回 “成功”。
1. Followers 收到 commit 消息后，才会将事务操作应用到 **内存中** 。

## 崩溃恢复

### 几个概念

- logicClock：每个服务器会维护一个自增的整数，名为 logicClock，它表示这是该服务器发起的第多少轮投票
- state：当前服务器的状态
- self_id：当前服务器的 myid
- self_zxid：当前服务器上所保存的数据的最大 zxid
- vote_id：被推举的服务器的 myid
- vote_zxid：被推举的服务器上所保存的数据的最大 zxid

### 故障投票流程

#### 自增选举轮次

所有有效的投票都必须在同一轮次中。每个服务器在开始新一轮投票时，会先对自己维护的 logicClock 进行自增操作。

#### 初始化投票

新一轮投票前，会将自己的投票箱清空。投票箱存储收到的选票，包括自己发送的投票。例：服务器 2 投票给服务器 3，服务器 3 投票给服务器 1，则服务器 1 的投票箱为 (2, 3), (3, 1), (1, 1)。票箱中只会记录每个投票者的最后一票。

#### 发送初始化选票

每个服务器最开始都是通过广播把票投给自己。

#### 接收外部投票

服务器会尝试从其他服务器获取投票，并记入自己的投票箱中。

#### 判断选举轮次

收到外部投票后，首先会根据投票信息中所包含的 logicClock 来进行不同处理：

- 外部投票 logicClock 大于自己的 logicClock，说明落后其他服务器投票，立即清空自己投票箱并将自己 logicClock 更新为收到的 logicClock，然后进行下一步的选票 PK。
- 外部投票 logicClock 小于自己的 logicClock，则直接忽略继续处理下个投票。
- 外部投票 logicClock 等于自己的 logicClock，进行选票 PK。

#### 选票 PK

选票 PK 是基于 (self_id, self_zxid) 与 (vote_id, vote_zxid) 的对比：

- 对比二者的 vote_zxid, 若外部投票的 vote_zxid 较大，则将自己的票中的 vote_zxid 与 vote_myid 更新为收到的票中的 vote_zxid 与 vote_myid 并广播出去，另外将收到的票及自己更新后的票放入到自己的票箱中，如果已经存在直接覆盖。
- 如果二者的 vote_zxid 一致，则比较二者的 vote_myid，若外部投票的 vote_myid 比较大，则将自己票中的 vote_myid 更新为收到票中的 vote_myid 并广播出去，另外将收到的票及自己更新后的票放入自己的票箱中。

#### 统计选票

如果已经可以确定超过半数的服务器认可了自己的投票（可能是更新后的投票），则终止投票。否则继续接收其他服务器的投票。

#### 更新服务器状态

投票终止后，服务器开始更新自身状态。若过半的票投给自己，则将自己的服务器状态更新为 LEADING，否则将自己的状态更新为 FOLLOWING。

#### 数据同步

以上是 ZAB 协议下一条事务的流程，和 2PC（两阶段提交）很类似，不过 2PC 需要得到所有的 ACK，而 ZAB 只需要半数以上的 ACK 即 commit。因为 ZAB 协议为了在发生网络分区的情况下，仍然服务可用。因此可以看到， ZAB 协议是侧重 CAP 理论中的 CP。

但在特殊情况，在 Leader 服务器收到一半以上的 ACK 后，就会向各个 Follower 广播 commit 命令，但如果各个 Follower 在收到 commit 命令前 Leader 挂了，会导致剩下的服务器 **并没有都执行** 这条消息。

另外，如果 Leader 中某条消息已经存在于自己的事务日志中，如果重启后重新被应用可能会导致其他服务器没有相同的数据。

如何解决这个问题呢？ZAB 遵循以下两个原则：

1. commit 过的数据不丢失
1. 未 commit 过的数据对客户端不可见

##### commit 过的数据不丢失

Leader 发送 commit 消息，说明半数以上的 Follower 已经 ACK，所以在此时宕机，Follower 中 zxid 最大的会被选为 Leader，此服务器上的内存中不存在该事务，但事务日志中存在该事务，因此在被选举后，该事务日志会被加载到新 Leader 的内存并通过提议的方式发送给 Follower。

##### 未 commit 过的数据对客户端不可见

首先，Leader 宕机后重连，此时已经拥有新的 Leader。新 Leader 的 zxid 肯定大于宕机的服务器（zxid 为 64 位，高 32 位为 epoch，每次选举出的 Leader 都会 +1，因此宕机服务器重启后肯定小于新的 Leader），宕机的服务器不会再被选择为 Leader，因此这些事务日志不会被广播到其他服务器，而发现和 Leader 的事务不一致时，会自动抛弃这些事务。

## 几种领导选举的场景

### 集群启动领导选举

初始投票给自己。集群刚启动时，所有服务器的 logicClock 都为 1，zxid 都为 0。各服务器初始化后，都投票给自己。

因为 logicClock 相同，则通常会比较接受到的 vote_zxid，因此也都相同。最后会比较 vote_myid，因此最终会是 id 最大的服务器称为 Leader。

### Follower 重启选举

Follower 重启，或者发生网络分区后找不到 Leader，会进入 LOOKING 状态并发起新的一轮投票。

但此时其实存在 Leader，因此 Leader 在收到该服务器的投票信息后，会返回自己状态是 LEADING 给该服务器。其他服务器也会发送 FOLLOWING 状态给该服务器，该服务器判断后确定超过半数的选票指向了 Leader，则进入 FOLLOWING 变为 Follower。

### Leader 重启选举

Leader 宕机后，其他 Follower 会进行选举，然后任命新的 Leader，因此在该服务器再次加入时，会同上得到 Leader 和其他 Follower 的反馈而进入 FOLLOWING 变为 Follower。


## 参考文章

1. [实例详解 ZooKeeper ZAB 协议、分布式锁与领导选举](https://dbaplus.cn/news-141-1875-1.html)
1. [zookeeper的zab协议工作原理之 崩溃恢复模式 - 云+社区 - 腾讯云](https://cloud.tencent.com/developer/article/1129115)
1. [浅析Zookeeper的一致性原理 - 知乎](https://zhuanlan.zhihu.com/p/25594630)
