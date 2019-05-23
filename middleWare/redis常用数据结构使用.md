# redis常用数据结构使用
基于redis的抽奖活动经常是针对一些秒杀类的活动

## 1. String
按照： XXXX:XXXX:XXXX的格式拼接key
由于redis是单线程模型，天然是线程安全的，这使得INCR/DECR命令非常适合实现高并发场景下的精确控制：* 库存控制
>设置库存：SET inv:remain "100"
扣减库存+余量校验：DECR inv:remain

* 自增序列的实现
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
一般用于存储奖池奖品记录，抽奖时采用rpop的方式弹出奖品

## 4. set
没用过

## 5. sortSet
用于存储排名信息，一般采用分数位的小数位来解决投资额度相同的问题。
涉及根据投资时间来生成排序因子的功能，代码如下：

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