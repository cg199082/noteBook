# zookeeper选举机制分析

zookeeper的选举机制涉及到zookeeper中的以下基础功能：
* Znode的数据类型（主要用到临时节点，顺序节点）
* Zookeeper的监视机制

## Znode的数据类型
Znode被分为持久节点（Persistent），顺序节点（sequential）和临时节点（ephemeral）
* 持久节点：即使在创建该节点的客户端断开连接后，持久节点仍然存在。默认情况下除非另有说明，否则所有的Znode都是持久的。
* 临时节点：客户端活跃的时候，临时节点就是有效的。当客户端与Zookerper集合断开连接时，临时节点会自动删除。因此，只有临时节点不允许有子节点。如果临时节点被删除，则下一个合适的节点将填充其位置。临时节点在Leader选举中起着重要的作用。
* 顺序节点：顺序节点可以是持久的或临时的。当一个新的Znode被创建为顺序节点时，Zookeeper通过将10位的序列号附加到原始名称来设置Znode的路径。
> 例如，如果将具体的路径/myapp的Znode创建为顺序节点，则Zookeeper会将路径更改为/myapp0000000001，并将下一个序列号设置为0000000002。如果两个序列号同时创建的，那么Zookeeper不会对每一个Znode使用相同的数字。顺序节点在锁定和同步中起到重要的作用。

## Watches监视
监视是一个简单的机制，使客户端收到关于Zookeeper集合中的更改的通知。客户端可以在读取特定的Znode时设置Watches。Watches会向注册的客户端发送该Node的任何更改通知。  
Znode的更改是指：与该Znode相关的数据的修改或者该Znode子项的更改。只触发一次Watches。如果客户端想要再次通知，则必须通过另一个读取操作来设置Watches。当连接会话过期时，客户端将于服务器端断开连接，相关的Watches也将被删除。

## Zookeeper选举
假设一个集群中有N个节点，leader选举的过程如下：
* 所有节点申请创建相同路径的顺序，临时节点/app/leader_election/guid_
* Zookeeper集合附加十位序列号到路径上，创建的Znode将是如下格式：  
/app/leader_election/guid_0000000001，/app/leader_election/guid_0000000002
* 对于给定的实例，在Znode中创建的最小数字的节点将成为leader，而其余的节点将成为follower
* 每一个follower节点监视上一个具有最小数字的Znode。
> 例如：创建/app/leader_election/guid_0000000008的节点将监视/app/leader_election/guid_0000000007的节点
* 如果leader关闭，则相应的节点就会自动被删除
* 下一个在线follower节点将通过监视器获得关于leader节点被移除的通知
* 下一个在线的follower节点将检查是否存在其他具有最小数值的Znode。如果没有，那么他将承担leader的角色。否则，它找到的具有最小数值的Znode节点将成为leader节点
* 类似的，所有其他的follower节点选举创建具有最小数值的Znode节点作为leader节点。

参考文档：  
zookeeper基础：https://www.w3cschool.cn/zookeeper/zookeeper_fundamentals.html  
zookeeper的选举机制：https://www.w3cschool.cn/zookeeper/zookeeper_leader_election.html  
zookeeper的选举客户端调用实现：http://ifeve.com/zookeeper-leader/