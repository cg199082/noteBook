# zookeeper的选举机制
参考文档：  
Zookeeper崩溃恢复过程(Leader选举)：https://blog.csdn.net/weixin_30268071/article/details/95118843  
理解zookeeper选举机制：https://cloud.tencent.com/developer/article/1084681

# 崩溃恢复
1. leader选择过程可以保证新的leader是ZXID最大的节点
2. ZAB协议确保丢弃那些只有在leader上被提交的事务，场景leader发出proposal收到ack，但是发出commit前发生崩溃，则新的群组会丢弃这条消息

# leader选举过程
## 服务器涉及到的状态
* looking：系统刚启动或者leader崩溃后的选举状态，认为当前集群中没哟leader，因此进入选举状态
* following：跟随者状态，角色是follower
* leading：领导状态，角色是leader
* observing：观察者状态（只支持查询不参加选举），角色是observer

## 投票过程中传递的数据结构vote
* id：被选举的leader的Sid值
* zxid：被选举的leader的事务id
* electionEpoch：逻辑时钟，用来判断多个投票是否在同一轮的选举周期中。该值在服务器端是一个自增序列，每次进入新一轮额投票后都会对该值进行加1操作
* peerEpoch：被选举的leader的epoch

## QuorumCnxmanager-server socket负责选举
* 每个服务器都会启动一个QuorumCnxmanager-server socket，负责服务器之间的leader选举

* socket内部维护着以下几个队列，每个队列又是按照Sid进行分组的队列集合
1. recvQueue：选票接收队列
2. queueSendMap：待发送的消息，内部按照sid为每台机器分配一个单独的队列，保证相互之间互不影响。
3. sendWorkerMap：负责消息发送，也是按照sid进行分组
4. lastMessageSent：最近发送的消息，按sid进行分组

* 建立连接，集群中机器需要进行两两连接，规则“只允许sid大的机器主动和其他机器进行连接，否则断开连接”来防止重复连接

* 当服务器检测到当前服务器状态变成looking时，就会触发leader选举
1. 如果已存在leader，则发送选票后会被告知leader的信息，直接连接即可，不需要进行后续步骤。
2. 自增选举轮次，在fastLeaderElection实现中有一个logicalclock属性，用来标识当前leader的选举轮次，zk规定所有投票必须在同一轮次，server开始新一轮投票前会进行自增操作
3. 初始化选票（第一次先投票给自己），参照前面的vote数据结构
4. 发送初始化选票
5. zk从receivedQueue中接收外部选票
如果zk发现自己无法获得任何投票，则马上检查是否和其他zk保持了有效连接，没有则建立连接，并再次发送自己当前的内部投票
6. 判断选举轮次（接收到外部的投票）
    * 外部投票轮次大于内部投票：立即更新自己的选举轮次（logicalclock）,并清空已经收到的所有投票，然后使用初始化投票来PK（第7步），根据PK结果来决定是否变更内部投票，最终再将内部投票发送出去
    * 外部投票的轮次小于内部投票，zk直接忽略该投票不做任何处理，返回步骤5
    * 内部投票轮次一样，开始选票PK
7. 选票PK  
比较顺序从先到后依次是：轮次 > ZXID > SID，**三种比较都是外部大于内部，则进行投票变更**
8. 投票变更  
用外部投票的信息覆盖内部投票，变更完成后，再次将这个信息发送出去
9. 选票归档  
无论是否进行了选票变更，都会将刚刚收到的那份投票放入选票集合“recvset”中，recvset内部按照sid存在本轮次收到的所有外部投票
10. 统计选票  
完成选票归档后，就开始统计选票。如果确定已经有超过半数的服务器认可了该内部投票，则终止投票。否则返回步骤5
11. 如果可以终止投票（在等200ms来确定是否有更优的投票），则更新服务器状态。  
首先判断投票选出的leader是否是自己，然后根据情况更新自己的状态为leading/following/observing
12. 选出leader后，所有的learner向leader发送learnerinfo消息，等待超过半熟的learner连接完成后（取它们最大的的epoch当做leader的epoch）
13. leader向learner发送leaderinfo消息，learner从中解析出epoch和zxid，然后向leader反馈一个ackepoch
14. leader接收到learner的ack后就开始进入数据同步环节。