# java性能调优

参考问文档： 
一次性搞清楚线上CPU100%，频繁FullGC排查套路：https://www.toutiao.com/i6706006742905405963/  
线上FullGC频繁的排查：https://juejin.im/post/5b4819fce51d45198c017d58  
JVM频繁Full GC的情况及应对策略：https://blog.csdn.net/endlu/article/details/51144918
关于施用full gc频繁的分析及解决：https://my.oschina.net/goldwave/blog/168516
# 概述
本文章主要包括以下内容：  
1. 线上FullGC频繁的排查
2. java常见线上问题定位


# 线上FullGC频繁的排查
## Full GC的原因
Full GC触发的常见原因：
1. 程序执行了system.gc()        //可通过-XX:+ DisableExplicitGC来关闭
2. 执行了jmap -history：live pid命令        //该命令会触发full gc
3. 在执行minor gc的时候进行了一系列检查
```
在执行minor GC的时候，jvm会检查老年代中最大的连续可用空间是否大于当前新生代所有对象的总大小  
如果大于，则直接进行minor GC（这个时候执行minor GC是没有任何风险的）  
如果小于，jvm会检查是否开启了空间分配担保机制，如果没有则直接改为执行Full GC  
如果开启了，则jvm会检查老年代中最大连续可用空间是否大于历次剑圣到老年代中的平均大小，如果小于则执行Full GC  
如果大于则会执行minor GC，如果minor GC执行失败则会执行Full GC  
```
4. 使用了大对象     //大对象会直接进入老年代
5. 在程序中长时间持有了对象的引用       //对象年龄进入指定阈值也会进入老年代

对于我们来说，可以初步排除第1，2两种情况，最有可能的是4，5两种情况。