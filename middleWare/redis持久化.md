# redis持久化方式
## redis持久化的两种方式
1. RDB：对redis中的数据进行周期性持久化
2. AOF：对每条写入命令都添加到日志。以append-only的模式写入日志文件中，在redis重启是以回放AOF日志的方式重构数据集。

## RDB优缺点
1. RDB会生成多个数据文件，每个文件代表某一时刻的redis数据，适合做冷备份
2. RDB对redis对外提供读写服务影响很小，保持了redis的高性能（因为redis只需要fork一个子进程，让子进程执行磁盘的IO操作来进行RDB持久化）
3. 重启和恢复redis进程更快

## RDB缺点
1. 不能保证数据实时性（一般每隔5分钟备份一次）
2. 如果数据文件特别大，可能导致对客户端提供的服务暂停数毫秒，甚至数秒。

## AOF优点
1. 更好的保护数据（一般每秒进行一次AOF，通过后台的fsync操作）
2. 以append-only的模式写入，没有磁盘寻址开销，写入性能高。
3. 即使日志文件过大导致rewrite，也不会影响客户端读写。
> rewrite log逻辑：fork一个子进程进行rewrite，父进程继续接受命令，现在的写操作命令会被额外添加到一个aof_rewrite_buf_block缓存中，同时AOF重写并不需要对原有的AOF文件进行任何写入和读取，它保存的事实上是redis数据库中的当前键值对，当rewrite结束后，父进程收到子进程退出信息，把aof_rewrite_buf_block文件添加到rewrite后的文件中，然后切换AOF的文件fd，rewrite完成。

## AOF缺点
1. 日志文件比RDB文件更大
2. 如果配置成实时写入日志，QPS会下降，redis性能会大大降低（一般配置成每秒fsycn一次日志文件）


参考文档：
面试题：redis 的持久化有几种？不同的持久化机制有什么优缺点？：
https://www.toutiao.com/i6689783907300147726/
AOF：https://redisbook.readthedocs.io/en/latest/internal/aof.html