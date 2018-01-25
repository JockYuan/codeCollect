# ***Java NIO 系列教程***

### Java NIO:Channels and Buffers (通道和缓冲区)
NIO基于通道和缓冲区(Buffer)进行操作, 数据总是从通道读取到缓冲区中, 或者从缓冲区写入通道

### Java NIO: Non-blocking IO(非阻塞IO)
Java NIO 可以用非阻塞的使用IO, 当线程从通道读取数据到缓冲区时, 线程还是可以进行其他事情.
当数据给写入到缓冲区时, 线程可以继续处理它.

### Java NIO: Selector(选择器)
选择器用于监听多个通道的事件. 如: 连接打开, 数据到达.
单个线程可以监听多个数据通道.


# Java NIO 概述
Java NIO 几个核心部分组成:
- Channels
- Buffers
- Selectors

### Channel和Buffer
所有的IO在NIO中都是从一个Channel开始. Channel有点像流, 数据可以从Channel读取到Buffer中, 也可以从Buffer写到Channel中.

![](http://ifeve.com/wp-content/uploads/2013/06/overview-channels-buffers1.png)

##### Java NIO 中主要Channel的实现
- FileChannel
- DatagkramChannel
- SocketChannel
- ServerSocketChannel
这些通道涵盖了UDP和TCP网络IO, 以及文件IO.
FileChannel 从文件中读写数据
DatagrameChannel 通过UDP读写网络中的数据
SocketChannel 通过TCP读写网络中的数据
ServerSocketChannel 可以监听新进来的TCP连接, 像Web服务器那样. 对每一个新进来的连接都会创建一个SocketChannel.

##### 基本的Channel示例:
```java
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();

ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buf);
while(bytesRead != -1) {
  System.out.println("Read " + bytesRead);
  buf.flip(); // make buffer ready for read

  while(buf.hasRemaining()) {
    System.out.print((char) buf.get());
  }
  buf.clear();
  bytesRead = inChannel.read(buf);
}
aFile.close();
```

##### Java NIO 关键Buffer实现:
- ByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer
- MappedByteBuffer

### Buffer的基本用法
四个步骤:
1. 写入数据到Buffer
2. 调用flip()方法
3. 从Buffer中读取数据
4. 调用clear()方法或者compact()方法

当向Buffer写入数据时, buffer会记录写了多少数据, 一旦要读取数据, 需要通过flip()方法将Buffer从写模式切换到读模式.
在读模式,可以读取之前写入到buffer的所有数据.

读完所有数据后, 需要调用clear()或compact()方法来清空缓冲区.
clear()方法会清空这个缓冲区. compact()方法只会清除已经读过的数据.

任何未读的数据都被移到缓冲区的起始位置, 新写入的数据将放到缓冲区未读数据的后面.

##### 使用Buffer的示例:
```java
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();

//create buffer with capacity of 48 bytes
ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buf);
while(bytesRead != -1) {
  System.out.println("Read " + bytesRead);
  buf.flip(); // make buffer ready for read

  while(buf.hasRemaining()) {
    System.out.print((char) buf.get()); // read 1 byte at a time
  }
  buf.clear();
  bytesRead = inChannel.read(buf);
}
aFile.close();
```
### Buffer的capacity, position和limit
![](http://ifeve.com/wp-content/uploads/2013/06/buffers-modes.png)

#### capacity
buffer的容量, 表示只能往里写capacity个byte, long, char等类型.
一旦Buffer满了, 需要清空数据才能继续写数据.

#### position
当写数据到buffer时, position表示当前的位置. 初始position为0.当一个byte,long等数据写到Buffer后, position会向前移到到下一个数据的Buffer单元. position最大值为capacity-1.

当从buffer中读取数据时, position会被重置为0, 当从Buffer的postion处读取数据后, position会移动到下一个可读位置.

#### limit
在写模式下, Buffer的limit表示你最多能往Buffer里写多少数据, 写模式下, limit等于capacity.
在读模式下, limit表示最多能读到多少数据. 在Buffer从写模式转换成读模式时, limit会被设置为写模式下的position值.

### Buffer的分配
```java
  ByteBuffer buf = ByteBuffer.allocate(48); // 分配48个字节
  CharBuffer buf1 = CharBuffer.allocate(1024); // 分配1024个字符
```
### 向Buffer中写数据
写数据到Buffer两种方式
- 从Channel写到Buffer.
- 通过Buffer的put()方法写到Buffer里.

```java
int bytesRead = inChannel.read(buf); // 从Channel写到Buffer里
buf.put(123); //通过put()写到Buffer里
```

##### flip()方法
flip方法将Buffer从写模式切换到读模式. 调用flip()方法会将position设置为0, 并将limit设置为之前position的值.
position现在用于标记读的位置, limit表示之前写进了多少byte, char等, 现在能读取多少个byte, char等.

##### 从Buffer中读取数据
1. 从Buffer读取到Channel
2. 使用get()方法从Buffer中读取数据
```java
int byteWritter = inChannel.write(buf); // 将buf中内容写入channel中
byte aByte = buf.get(); // 从buff中读取数据
```
##### rewind()方法
将position设置为0, 可以重读Buffer中所有的数据. limit保持不变, 表示能从Buffer中读取多少个元素.

##### clear与compact()方法
clear方法, 将position设置为0, limit设置为capacity值, 即重置了设置, 可以写入数据, 原来的数据即使存在也将无效, 在写入新数据时,将其擦除.

如果Buffer中仍然有未读数据, 此时要写数据, 需要compact方法.
compact()方法将所有未读的数据复制到buffer起始位置, 然后将position设置到最后一个未读元素的后面, limit属性仍然想clear()时, 设置为capacity值, Buffer准备后写数据了, 但是不会覆盖未读数据.

##### mark()与reset方法
make()方法标记一个特定的position. 之后可以通过reset()方法恢复这个position.
```java
buffer.mark();
// call buffer.get() a couple of times // 多次调用get(),获取buffer中的数据.
buffer.reset(); // 恢复mark时, position的位置.
```
##### equals()
比较两个Buffer是否相等:
1. 有相同的类型.
2. Buffer中剩余的byte, char等元素个数相等.
3. Buffer中剩余的byte, char等元素本身也相等.
equals值比较Buffer的一部分, 只比较Buffer中剩余的元素.

##### compareTo()
满足下列条件则认为一个Buffer"小于"另一个Buffer:
1. 第一个不相等的元素小于另一个Buffer中对应的元素.
2. 所有元素相等, 但第一个Buffer比另一个先耗尽(第一个Buffer中的剩余元素个数比另一个少).

### Selector
Selector允许单线程处理多个Channel. 如果应用打个多个连接(通道), 每个连接的流量都很低, 使用Selector会很合适
单个线程中使用一个Selector处理3个channel的图示:
![](http://ifeve.com/wp-content/uploads/2013/06/overview-selectors.png)

使用Selector, 得向Selector注册Channel, 然后调用select()方法, 这个方法会一直阻塞到某个注册的通道有事件就绪. 一旦这个方法返回, 线程就可以处理这些事件.

### Scatter/Gather
Scatter(分散) 从channel中读取数据写入多个Buffer中.
Gather(聚集) 在写操作时, 将多个Buffer的数据写入同一个Channel中.

常用用于需要将传输的数据分开处理的场合.如传输一个有消息头和消息体组成的消息,可能会将消息体和消息头分散到不同的Buffer中.可以方便的处理消息头和消息体.

##### Scatter read
![](http://ifeve.com/wp-content/uploads/2013/06/scatter.png)
```java
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body = ByteBuffer.allocate(1024);
ByteBuffer[] bufferArray = {header, body};
channel.read(bufferArray);
```

read()方法按照buffer在数组中的顺序将channel中的数据写入到buffer中, 当一个buffer写满后, channel才会写入下一个buffer. 适用于固定大小分组的消息

##### Gathering writes
![](http://ifeve.com/wp-content/uploads/2013/06/gather.png)
```java
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body = ByteBuffer.allocate(1024);
// write data iont buffers;
ByteBuffer[] bufferArray = {header, body};
channel.write(bufferArray); // 将buffer数组中的数据写入到channel中.
```
在写入数据时, 只有在buffer中的有效数据才能被写入(即position和limit之间的数据才会被写入). 当header的buffer中只有58个byte的数据时, 这58个数据将会被写入到channel中, Gathering writes可以处理动态消息.

### 通道之间的数据传输
##### transferFrom()
FileChannel的transferFrom()方法可以将数据从源通道传输到FileChannel中
```java
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel fromChannel = fromFile.getChannel();
RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel toChannel = toFile.getChannel();
long position = 0;
long count = fromChannel.size();
toChannel.transferFrom(fromChannel,position,  count);
```

##### transferTot()
transferTo()方法将数据从FileChannel传输到其他的channel中.
```java
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();

long position = 0;
long count = fromChannel.size();

fromChannel.transferTo(position, count, toChannel);
```
关于SocketChannel的transferTo()方法同样存在. SocketChannel会一直传输数据知道目标buffer填满.
