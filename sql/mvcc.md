# SQL学习笔记
## InnoDB的mvcc
mvcc可以理解为java中的多版本
为实现对应的功能他为每行记录都实现了三个隐藏字段

* DB_TRX_ID：6字节事务id
* DB_ROLL_PTR：7字节的回滚指针
* 默认的隐藏ID

对应的实际存储结构如下图：

![](./source/mvcc_001.gif)

## InnoDB的事务相关概念
为了支持事务，innoDB引入了以下概念：

* redo log
> 保存执行的sql语句到一个指定的log文件，当mysql执行recovery时重新执行redo log记录中的SQL语句即可。被保存在一个独立的文件中即innoDB的log文件中

* undo log
> 与redo log相反undo log是为了回滚而用的，具体的内容就是copy事务前的数据内容行到undo buffer,在适合的时间把undo buffer中的内容刷新到新的磁盘中，不存在单独的磁盘文件，存在于innodb数据文件中（表空间）中。

* 共享锁/排他锁
> 共享锁针对读，排他锁针对写。如果某个事务更新某行（排他锁），其他事务无论读写本行都必须等待；如果某个事务读取某行（共享锁），其他读事务无需等待，写事务必须等待。

未完待续....


参考文章：[innoDB中mvcc](http://www.360doc.com/content/14/0821/09/12904276_403505950.shtml)