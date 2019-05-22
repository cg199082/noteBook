# SQL性能优化

## 善用explain进行分析
![](./source/sql_001.jpg)

* type：连接类型，一个性能高的sql语句至少要达到range级别。拒绝出现all级别
* key：使用到的索引名，如果没有用到索引值为null。可以采用强制索引。
* key_len：索引长度
* rows：扫描行数。是个预估值。
* extra：详细说明，注意常用的不太友好的值有：using filesort,using temporary