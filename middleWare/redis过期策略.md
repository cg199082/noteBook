# redis过期策略
redis的过期策略有以下三种：
* 被动删除
* 主动删除
* 当前已用内存超过maxMemory限定时，触发主动清理策略

## 被动删除
当读写一个已经过期的key时，会触发惰性删除策略直接删除掉这个过期的key

## 主动删除
redis后台有一个时间事件，每过一段时间定时处理相关的业务（清理失效客户端，进行数据备份，同步集群数据，测试集群连接情况等）包括删除过期的key。  
redis会周期性的随机性的测试一批设置了过期时间的Key并进行清理。测试到的已过期的Key将会被删除。定性的方式是，Redis每秒做10次如下操作：
* 随机测试100个设置了过期时间的Key
* 删除所有发现已过期的Key
* 若删除的Key超过25个则重复步骤1

redis这样的设置是为了保证不会因为每次清理Key暂用时间过长而导致影响服务响应请求的性能

## maxMemory
当前已用内存超过maxMemory限定时，触发主动清理策略:
* volatile-lru:只对设置了过期时间的Key进行LRU(默认值)
* allKey-lru:删除lru算法的Key
* volatile-random：随即删除即将过期的Key
* Allkeys-random：随机删除
* volatile-ttl：删除即将过期的
* noeviction：永不过期

当使用内存超过maxMemory时，对于所有的读写请求都会出发清理内存操作。而且该操作是阻塞的，直到清理出足够的内存空间为止。

参考文档：
https://www.cnblogs.com/chenpingzhao/p/5022467.html
https://www.toutiao.com/i6691142944201638403/