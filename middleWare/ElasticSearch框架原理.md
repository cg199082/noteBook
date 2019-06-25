# ElasticSearch框架内部原理
## 1. 如何实现快速索引
ElasticSearch是通过Luncene的倒排索引技术实现比关系型数据库更快的过滤效果的。特别是它对多条件的过滤支持的非常好，比如年龄在18到30之间，性别是女性这样的组合查询。为什么倒排索引的速度比B-tree的速度快呢？  
笼统的说b-tree索引是为了写入优化的索引结构。当我们不需要支持快速更新的时候，可以用预先排序等方式换取更小的存储空间，更快的检索速度等好处，其代价是更新慢。要进一步深入的话，还是要看一下Luncene的倒排索引是怎样构成的。 

![](./source/elasticSearch_001.jpg)  

这里有好几个概念。下面结合具体的实例进行讲解，假如有如下数据：

 ![](./source/elasticSearch_001.png)  
 
 上图没一行是一个Document。每一个Document都有一个Docid。那么给这些Document建立倒排索引的就是：  
 * 年龄  
 * 性别  
 可以看出倒排索引是Per Field的，每一个字段都有一个自己的倒排索引。18，20这些叫Term，而[1,3]就是Posting List。Posting List就是一个int的数组，存储着所有符合某个Term的文档的Id集合。那么什么是Term Dictionary和Term Index？  
 加入有多个Term如下：  
 **Carla,Sara,Elin,Ada,Patty,Kate,Selena**  
 如果按这样的顺序排列，找出某个特定的Term一定很慢，因为Term没有排序，需要全部过滤一遍才能找到特定的Term。排序之后就变为：  
 **Ada,Carla,Elin,Kate,Patty,Sara,Selena**  
 这样就可以通过二分查找的方式，比全部遍历更快的找到目标。这个就是Term Dictionary。有了Term Dictionary之后，可以在LogN次磁盘查找找到目标。但是磁盘的随机读操作任然是非常昂贵的（一次random dictionary大约需要10S的时间）。所以尽量减少磁盘的读操作，有必要把一些数据缓存到内存中。但是整个term dictionary本身有太大了，无法完整的放到内存中。于是就有了term index。term index有点像一本字典的大的章节表。比如：  
 * A开头的term................XXX页
 * C开头的term................XXX页
 * E开头的term................XXX页
如果所有的term都是英文字符开头的话，可能这个term index就真的是26个英文字符表构成的了。但是实际上，term未必都是英文字符，term可以使任意byte数组。而且26个英文字符也未必每一个字符都有均等的term，比如X字符开头的term可能一个都没有，而S开头的term有特别多。实际的term index是如下的一个trie树：  

 ![](./source/elasticSearch_002.png)  

 例子是一个包含 "A", "to", "tea", "ted", "ten", "i", "in", 和 "inn" 的 trie 树。这棵树不会包含所有的term，它包含的term是一些前缀。通过term index可以快速的定位到term dictionary的某个offset，然后从这个位置再往后顺序查找。再加上一些压缩技术（搜索Luncene Finite State Transducers）term index的尺寸可以只有所有term的尺寸的几十分之一，使得用内存缓存整个term index成为可能。整体上来说就是这样的效果：  

![](./source/elasticSearch_002.jpg)

因此就可以很清楚的明白为什么ElasticSearch比mysql快的原因。Mysql只有term Dictionary这一层，是以b-tree的排序方式存储在磁盘上的。检索一个term需要若干次的random access的磁盘操作。而Luncene在term dictionary的基础上添加了term index来加速检索，term index以树的形式缓存在内存中。从term index查询到对应的term dictionary的block位置后，再去磁盘上找term，大大减少了磁盘的random access次数。  
额外值一提的两点是：term index在内存中是以**FST**（Finite State Transducers）的形式保存的。其特点是非常节省内存。**term dictionary在磁盘上是以分block的方式保存的**，一个block内部利用公共前缀压缩，比如都是Ab开头的单词就可以把Ab省去。这样term dictionary可以比b-tree更节省磁盘空间。  

## 如何联合索引查询
age = 18 and gender = ‘女’  
首先过滤条件age = 18的过程是从term index找到18的term dictionary的大概位置，然后再从term dictionary里精确地找到这个18的term，然后得到posting list或者一个指向posting list位置的指针。然后在查询gender = ‘女’的posting list。最后把这两个post listing 进行与操作。  
