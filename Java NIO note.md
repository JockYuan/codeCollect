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


### Selector
Selector允许单线程处理多个Channel. 如果应用打个多个连接(通道), 每个连接的流量都很低, 使用Selector会很合适
单个线程中使用一个Selector处理3个channel的图示:
![](http://ifeve.com/wp-content/uploads/2013/06/overview-selectors.png)

使用Selector, 得向Selector注册Channel, 然后调用select()方法, 这个方法会一直阻塞到某个注册的通道有事件就绪. 一旦这个方法返回, 线程就可以处理这些事件.
