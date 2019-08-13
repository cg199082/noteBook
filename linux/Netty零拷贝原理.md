# Netty零拷贝原理分析

## Netty高性能的原因
Netty作为异步事件驱动的框架，其高性能的原因主要在于其**高效的IO模型**和**线程处理模型**，前者决定了如何收发，后者决定了如何处理数据。主要体现在如下几个方面：  
1. 基于IO多路复用器模型  
基于NIO模型，采用异步IO操作，使用无锁化的串行设计理念
2. 零拷贝  
使用NIO的Buffer
3. 基于内存池的缓冲区重用机制
4. 提供对ProtoBuf等高性能序列化协议支持
5. 可以对TCP进行更加灵活的支持

## Netty的零拷贝
操作系统层面的零拷贝是指避免在用户态和内核态之间来回的拷贝数据的技术。Netty中的零拷贝事实上是对操作系统零拷贝的一种封装调用，只不过是通过java的方式灵活的封装，优化调用操作系统相关的API过程。  
Netty的零拷贝主要做了如下封装操作：  
1. Netty的接收和发送的ByteBuffer使用的是直接内存进行Socket读写，不需要进行字节缓冲区的二次拷贝。如果使用JVM的堆内存的话，发送和接收过程就会多出两次用户态内存缓冲区的拷贝操作。
2. Netty的文件传输调用的是FileRegion包装的TransferTo方法，可以直接将文件缓冲区的数据发送到目标Channel，避免了通过循环Write方式导致的内存拷贝操作。
3. Netty提供了用于封装java原生buffer的ByteBuf类，以及对ByteBuf进行各种操作的ComPositeByteBuf类  
ByteBuf支持slice操作，可以将ByteBuf分解成多个共享同一个存储区域的ByteBuffer，避免了内存拷贝  
CompositeByteBuf提供了Wrap操作，可以将java原生的byte[]数组，ByteBuffer等包装成ByteBuf对象，从而避免拷贝操作  
CompositeByteBuf提供了多种构造方法可以将多个ByteBuf合并为一个逻辑上的ByteBuf，避免了各个ByteBuf之间的拷贝  

### 直接内存映射
关于直接内存映射可以参考以下文档NIO效率高的原理之零拷贝与直接内存映射：https://www.toutiao.com/i6722656646830490126/?group_id=6722656646830490126

### 通过FileRegion实现零拷贝
FileRegion底层调用的是java原生NIO的FileChannel的TransferTo函数。其对应的实现是openjdk\jdk\src\share\classes\sun\nio\ch\FileChannelImpl.java ，其实现逻辑是先尝试调用sendFile，如果系统不支持，对于信任的Channel类型通过调用Mmap进行传递，否则走标准的read/Write系统调用。（window系统不支持sendFile，所以window默认是走Mmap方式的，虽然是Mmap方式相比于标准的read/write还是快很多的）。  
从源码来看只有源为FileChanel才支持transfer这种高效的复制方式，其他如SocketChanel都不支持Transfer模式。当然目的Channel没有这种限制，所以一般可以做FileChannel——>FileChannel和FileChannel——》SocketChannel的transfer。  

```
public long transferTo(long position, long count,
                           WritableByteChannel target)
        throws IOException
    {
        ensureOpen();
        if (!target.isOpen())
            throw new ClosedChannelException();
        if (!readable)
            throw new NonReadableChannelException();
        if (target instanceof FileChannelImpl &&
            !((FileChannelImpl)target).writable)
            throw new NonWritableChannelException();
        if ((position < 0) || (count < 0))
            throw new IllegalArgumentException();
        long sz = size();
        if (position > sz)
            return 0;
        int icount = (int)Math.min(count, Integer.MAX_VALUE);
        if ((sz - position) < icount)
            icount = (int)(sz - position);

        long n;

        // Attempt a direct transfer, if the kernel supports it
        if ((n = transferToDirectly(position, icount, target)) >= 0)
            return n;

        // Attempt a mapped transfer, but only to trusted channel types
        if ((n = transferToTrustedChannel(position, icount, target)) >= 0)
            return n;

        // Slow path for untrusted targets
        return transferToArbitraryChannel(position, icount, target);
    }
```

### 通过CompositeByteBuf实现零拷贝
CompositeByteBuf可以把需要合并的多个ByteBuf组合起来，对外提供统一的readIndex和WriteIndex。但在CompositeByteBuf内部，被合并的多个ByteBuf其实都是单独存在的，CompositeByteBuf只是逻辑上的一个整体。CompositeByteBuf内部有一个Component数组，被组合的ByteBuf都是被封装成Component对象后存放在CompositeByteBuf内部的Component数组里面的。

假设有一份协议数据，由头部和消息体构成，并且头部和消息体分别放在两个ByteBuf中，为了后续处理数据需要合并两个ByteBuf。  
* 传统的合并ByteBuf的做法
```
ByteBuf header = ...
ByteBuf body = ...
// 按照原本的做法 将header和body合并为一个ByteBuf
ByteBuf allBuf = Unpooled.buffer(header.readableBytes() + body.readableBytes());
allBuf.writeBytes(header);
allBuf.writeBytes(body);
```

在该过程中将Header和Body都拷贝到了新的AllBuf中，就额外增加了两次拷贝操作。

* CompositeByteBuf的实现方式

(./source/nettyZeroCopy_001.jpg)

CompositeByteBuf的合并ByteBuf的操作减少了两次额外的数据拷贝操作。  

```
ByteBuf header = ...
ByteBuf body = ...
// 新建CompositeByteBuf对象
CompositeByteBuf compositeByteBuf = Unpooled.compositeBuffer();
// 第一个参数是true, 表示当添加新的ByteBuf时, 自动递增 CompositeByteBuf 的 writeIndex。如果不传第一个参数或第一个参数为false，则合并后的compositeByteBuf的writeIndex不移动，即不能从compositeByteBuf中读取到新合并的数据。
compositeByteBuf.addComponents(true,header,body);
```

readIndex和writeIndex结构原理图如下：

(./source/nettyZeroCopy_002.jpg)

当然除了使用除了使用CompositeByteBuf类进行操作外，还可以直接使用Unpooled.WrappedBuffer方法。Unpooled封装了CompositeByteBuf的操作，使用起来更加方便：  

```
ByteBuf header = ...
ByteBuf body = ...
ByteBuf allByteBuf = Unpooled.wrappedBuffer(header, body);
```

### 通过Wrap实现零拷贝
如果将一个byte数组转化为一个ByteBuf对象，传统的做法是将byte数组拷贝到ByteBuf中。  
```
byte[] bytes = ...
ByteBuf byteBuf = Unpooled.buffer();
byteBuf.writeBytes(bytes);
```

显然这种方式额外增加了一个拷贝操作，我们可以直接使用Unpooled的相关方法，包装这个byte数组，生成一个新的ByteBuf对象，而不需要拷贝操作：  
```
byte[] bytes = ...
ByteBuf byteBuf = Unpooled.wrappedBuffer(bytes);
```

通过Unpooled.wrappedBuffer方法将byte数组包装成一个UnpooledHeapByteBuf对象，而在包装的过程中，不会由拷贝的操作，即生成的ByteBuf对象和byte数组公用了同一个存储空间，对byte的操作也就是对ByteBuf对象的操作。  
同时Unpooled类还提供了很多重载WrappedBuffer方法可以将一个或多个Buffer也包装成一个ByteBuf对象，从而实现零拷贝。  
```
public static ByteBuf wrappedBuffer(byte[] array)
public static ByteBuf wrappedBuffer(byte[] array, int offset, int length)
public static ByteBuf wrappedBuffer(ByteBuffer buffer)
public static ByteBuf wrappedBuffer(ByteBuf buffer)
public static ByteBuf wrappedBuffer(byte[]... arrays)
public static ByteBuf wrappedBuffer(ByteBuf... buffers)
public static ByteBuf wrappedBuffer(ByteBuffer... buffers)
public static ByteBuf wrappedBuffer(int maxNumComponents, byte[]... arrays)
public static ByteBuf wrappedBuffer(int maxNumComponents, ByteBuf... buffers)
public static ByteBuf wrappedBuffer(int maxNumComponents, ByteBuffer... buffers)
```

### 通过slice操作实现零拷贝
Slice操作和Wrap操作刚好相反，它可以将一个ByteBuf切片为对个共享同一个存储区域的ByteBufer对象。其实现逻辑图如下：  

(./source/nettyZeroCopy_003.jpg)

ByteBuf提供了两个slice操作方法：  
```
public ByteBuf slice();
public ByteBuf slice(int index, int length);
```

前者等同于buf.slice(buf.readerIndex(), buf.readableBytes())调用，即返回Buf中可读部分的切片。后者比较灵活，可以设置不同的参数获取部分不同区域的切片。由slice方法产生的header和body的过程是没有拷贝操作的，Header和Body对象在内部其实是共享了ByteBuf存储空间的不同部分而已。

参考文档：  
彻底搞懂Netty高性能之零拷贝：https://www.toutiao.com/i6722975256065081859/
NIO效率高的原理之零拷贝与直接内存映射：https://www.toutiao.com/i6722656646830490126/?group_id=6722656646830490126
Java下FileChannel的实现剖析：https://blog.csdn.net/tlxamulet/article/details/80786089
Netty zero copy之CompositeByteBuf读取和写入：https://blog.csdn.net/weixin_38009046/article/details/81985091