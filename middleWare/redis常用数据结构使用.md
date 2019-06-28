# redis常用数据结构使用
本文档是结合redis在抽奖活动中的应用这个具体的场景进行的剖析  

## 1. String
规范： 一般按照XXXX:XXXX:XXXX的格式进行拼接key  
由于redis是单线程模型，天然是线程安全的，这使得INCR/DECR命令非常适合实现高并发场景下的精确控制  
* 应用于库存控制
>设置库存：SET inv:remain "100"  
扣减库存+余量校验：DECR inv:remain

* 应用于自增序列的实现
> 设置序列起始值：SET sequence "10000"  
获取一个序列：INCR sequence  
获取一批序列：INCRBY sequence 100

## 2. hashMap
一般用于分期奖池的分期信息存储和检索
* 通过HGETALL获取一个活动的所有分期奖池
> key：奖池key  
value：奖池的开始结束时间（格式：开始时间字符串_结束时间字符串）
* 匹配合适的奖池时间，根据对应的key获取对应的奖池进行抽奖

## 3. list
一般用于存储秒杀类奖品（一般秒杀类奖品数量不多，但是请求量巨大，放到数据库中秒杀时性能太低），抽奖时采用rpop的方式弹出奖品，全部数据都弹出，即表示秒杀完毕。

## 4. set
没用过

## 5. sortSet
特别适合用于生成，存储排行榜信息（比如一段时间内的投资排行榜），一般用投资额度来作为排序的分值，对应的命令Zrange和ZREVRANGE。  
在实际过程中经常会遇见投资额度相同的情况，一般的解决方案是：合理利用SortSet的分数数据是Double类型这个特点
采用将Double数据的整数部分设置为投资额度，小数位采用排序因子的方式(该排序因子一般根据投资的先后时间来生成)来实现相同投资额度的排序。  
涉及根据投资时间来生成排序因子的代码逻辑如下：

```
/**
 * 计算排序因子
 * 加入时间距离开始时间越近,结果越大.
 * @param activityStart
 * @param orderTime
 * @return
 */
public static double calcRankFactor(LocalDateTime activityStart, LocalDateTime orderTime){
    //已经过了多少毫秒
    long pastMillis = Duration.between(activityStart, orderTime).toMillis();
    //
    return (1 - (pastMillis+1)*100000000d/Long.MAX_VALUE);
}
```



参考文档：
https://www.toutiao.com/i6691159847691354638/