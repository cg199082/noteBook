# threadLocal 原理分析
threadLocal<T>类相当于线程访问其线程特有对象的代理（Proxy），各个线程通过这个对象可以创建并访问各自线程的特有对象，参数T指定了线程特有对象的类型。**一个线程可以使用不同的threadLocal对象实例来创建并访问其不同的线程特有对象**。多个线程使用同一个threadLocal实例所访问的的实例是类型T的不同实例（即各个线程各自特有的对象实例）。  
这种代理模式的结构图如下：  

![](./source/threadLocal_001.png)

# threadLocal实现机制
在java平台中每个线程内部都会维护着一个类似hashMap的对象threadLocalMap,每个threadLocalMap内部会包含若干个条目Entry（一个键值对key-value）。Entry的key是一个threadLcoal实例，Value是一个线程特有对象。因此Entry相当于为其属主线程建立了一个threadLocal实例与一个线程特有对象之间的对应关系。对于Entry对于threadLocal的引用是一个弱引用（weak reference），因此他不会阻止被引用的threadLocal实例被垃圾回收，如果一个threadLocal实例灭有对其可达的强引用时，这个实例可以被回收。但是，**由于Entry对于线程特有对象的引用是强引用，因此如果无效条目本身有对他的可达强引用，所以无效条目会阻止其引用的线程特有对象被垃圾回收。**  
线程对象对threadLocal和线程特有对象的引用关系如下图：  

![](./source/threadLocal_002.png)

在Tomcat环境下，web引用自定义的类有类加载器webAppClassLoader负责加载，java标准类有类加载StandardClassLoader负责加载。每个类都会持有对加载该类的类加载器的强引用，并且类加载器本身也会持有其加载过的所有类的强引用。另外每个对象都会持有其相应类的强引用。  那么就会出现一种内存泄漏的情况如下图：  

![](./source/threadLocal_003.png)

常用的解决方案是：通过Filter来规避泄漏，在请求结束后通过threadLocal.remove()强制删除这种强引用关系。