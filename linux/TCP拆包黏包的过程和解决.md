# TCP拆包和黏包的过程和解决

## 粘包和拆包的表现形式
假设客户端向服务器端发送了两个数据包，用packet1和packet2表示，那么服务器端收到数据时可分为下列三种情况：  
### 第一种：接收端正常收到两个数据包，没有发生粘包和拆包的情况

![](./source/tcpPacket_001.jpg)

### 第二种：接收方只收到一个包，因为TCP是不会出现丢包的，所以这个包包含了发送端发送的两个包的数据，即出现粘包情况

![](./source/tcpPacket_002.jpg)

这种情况下由于接收端不知道两个数据包的界限，所以对于接收端来说很难处理

### 第三种：这种情况有两种表现形式，如下图：

![](./source/tcpPacket_003.jpg)  
![](./source/tcpPacket_004.jpg)

接收端收到两个数据包，但是这两个数据包要么是不完整的，要么就是多出来一块，这种情况即发生了拆包和粘包。这两种情况如果不加特殊处理，对于接收端同样是不好处理的。

## 粘包和拆包的解决方法
通过以上分析，可以很清楚的知道粘包和拆包出现的原因，解决问题点关键在于如何给每个数据包添加边界信息，常用的手段偶以下几种：  
1. 发送端给每个数据包添加包首部，首部中包含数据包的长度，这样接收端在收到数据后，通过读取包首部的长度字段，便知道每个数据包的实际长度了。
2. 发送端将每个数据包封装成固定的长度（不够的可以通过补零填充），这样这样接收端每次从接收缓存区中读出固定长度的数据就自然而然的把每个数据包拆开了。
3. 可以在数据包之间设置边界，如回车换行符号，这样接收端通过这个边界就可以将不同的数据包拆分开来。

## netty的解决方案
netty封装了JDK的NIO，是一个异步事件驱动的应用框架，用于快速开发可维护的高性能服务器和客户端。同时也内置了上述三种解决粘包和拆包问题的方案

### 方案一的Netty实现方式：
发送端继承MessageToByteEncoder<T>，重写其encode函数，来自定义编码器。  
```
public class SocketEncoder extends MessageToByteEncoder<Packet> {
 @Override
 protected void encode(ChannelHandlerContext channelHandlerContext, NetPacket msg, ByteBuf byteBuf) throws Exception {
 byte body[] = msg.getBody();
 int packetLen = body.length;
 // 先设置包长度，然后写入二进制数据
 byteBuf.writeInt(packetLen);
 byteBuf.writeBytes(body);
 }
}
```

接收端继承ByteToMessageDecoder，重写其decode函数，用来自定义解码器。
```
public class SocketDecoder extends ByteToMessageDecoder {
 @Override
 void decode(ChannelHandlerContext channelHandlerContext, ByteBuf byteBuf, List<Object> list) throws Exception {
 int bufLen = byteBuf.readableBytes();
 // 解决粘包问题（不够一个包头的长度）
 // 4字节是报文中使用了一个int表示了报文长度
 if (bufLen < 4) {
 return;
 }
 // 标记一下当前的readIndex的位置
 byteBuf.markReaderIndex();
 int packetLength = byteBuf.readInt();
 // 读到的消息体长度如果小于我们传送过来的消息长度，则resetReaderIndex。重置读索引,继续接收
 if (byteBuf.readableBytes() < packetLength) {
 // 配合markReaderIndex使用的。把readIndex重置到mark的地方
 byteBuf.resetReaderIndex();
 return;
 }
 NetPacket netPacket = new NetPacket();
 netPacket.setPacketLen(packetLength);
 // 传送过来数据的长度，满足我们的要求了
 byte body[] = new byte[packetLength];
 byteBuf.readBytes(body);
 netPacket.setBody(body);
 list.add(netPacket);
 }
}
```


参考文档：  
TCP拆包和黏包的过程和解决：https://blog.csdn.net/xmh594603296/article/details/86649537  
TCP粘拆包详解与Netty代码示例：https://www.toutiao.com/i6715713088542212622/