### BIO & NIO & AIO

### BIO

- 每个连接都创建一个线程对应

##### 缺点

- 线程多，浪费cpu资源与内存资源



### NIO

- nio是面向缓冲区的，这使得它可以不必时刻等待着io通道中的数据流入，做到了非阻塞
- nio由三大核心部分组成：Channel、Buffer、Selector；



#### 什么是Channel，Channel的基本使用

- Channel其实与IO的流很像，只不过它是面向缓冲区的，并且是双向的
- 新建一个`ServerSocketChannelImpl`其本质是在底层操作系统创建了一个fd(即[文件描述符](https://www.linuxprobe.com/linux-file-descriptor.html))，相当于建立了一个用于网络通信的通道，通过这个通道我们可以和外部网络进行通信；

##### 基本使用

- 服务端

  1. 通过ServerSocketChannel的open方法取得实例，用于监听连接
  2. 通过bind方法绑定端口，监听这个端口的连接
  3. 通过configureBlocking方法设置为非阻塞运行或阻塞运行
  4. 通过accept监听连接，监听到连接后会返回一个channel对象，这个Channel对象与客户端的Channel对象并无不同

  由于ServerSocketChannel可以是非阻塞的，而且每个连接实际上使用的是channel，这就给一个线程建立多个连接提供了实现的思路，只需要用一个集合将这些个连接放在一起，然后使用一个线程不停的遍历这个集合，并对有数据的Channel进行操作，这就是最开始的select

- 客户端

  1. 通过`open`方法创建`SocketChannel`，
  2. 然后利用`connect`方法来和服务端发起建立连接，还支持了一些判断连接建立情况的方法；
  3. `read`和`write`支持最基本的读写操作

##### Channel 的实现类

nio中对Channel的实现主要有以下类：

1. 网络I/O设备：
   DatagramChannel:读写UDP通信的数据，对应DatagramSocket类
   SocketChannel:读写TCP通信的数据，对应Socket类
   ServerSocketChannel:监听新的TCP连接，并且会创建一个可读写的SocketChannel，对应ServerSocket类
2. 本地I/O设备：
   FileChannel:读写本地文件的数据,不支持Selector控制，对应File类；实际上用Channel去做本地文件的读取是一件比较愚蠢的事情，因为nio的优势本来就不在这儿，还不如使用bio去读取

- 这些实现都只是声明了可操作的接口，因为具体的实现在不同操作系统上是不同的

#### Buffer的基本使用

1. 创建缓冲区，将文件写入缓冲区
2. 调用filp()方法将缓冲区改成读取模式
3. 从Buffer中读取数据
4. 调用clear()方法或compact()方法将缓冲区清空并改为写模式
   1. clear方法会清空整个缓冲区；compact会清空已经读过的数据

#### Selector

- Selector叫做多路复用器

- Selector的思路是解决两个问题，其实现类在windows上是WindowsSelectorImpl,但是在unix上就是EpollSelectorImpl了，所以这个多路复用就是使用的epoll的方式解决了这两个问题
  - 监听的线程不停空轮询，浪费cpu性能	-->  解决方案：监听线程休眠，有事件产生后让内核通过回调唤醒线程
  - 每有一个事件产生就要遍历完整个fd集合，浪费性能  -->  解决方案：将产生事件的fd单独放在一个集合中，只需要轮询这个集合就可以了

##### Selector的基本使用

1. 通过Selector的open方法取的Selector实例
2. 将ServerSocketChannel绑定到Selector上，并设置唤醒事件为SelectionKey.OP_ACCEPT，表示这个Channel只对连接事件感兴趣
3. 然后死循环调用Selector的select方法，并对各种事件进行相应的处理

#### Channel详解

ServerSocketChannel

```java
/*
	加锁，取得一个ServerSocketChannelImpl对象，其本质是在操作系统创建了一个fd用于传输数据，实际上所有的基本流都是在操作系统层创建了一个fd
*/
public static ServerSocketChannel open() throws IOException {
    return SelectorProvider.provider().openServerSocketChannel();
}

/*
	就是绑定端口
*/
public ServerSocketChannel bind(SocketAddress var1, int var2) throws IOException {
    synchronized(this.lock) {
        if (!this.isOpen()) {
            throw new ClosedChannelException();
        } else if (this.isBound()) {
            throw new AlreadyBoundException();
        } else {
            InetSocketAddress var4 = var1 == null ? new InetSocketAddress(0) : Net.checkAddress(var1);
            SecurityManager var5 = System.getSecurityManager();
            if (var5 != null) {
                var5.checkListen(var4.getPort());
            }

            NetHooks.beforeTcpBind(this.fd, var4.getAddress(), var4.getPort());
            Net.bind(this.fd, var4.getAddress(), var4.getPort());
            Net.listen(this.fd, var2 < 1 ? 50 : var2);
            synchronized(this.stateLock) {
                this.localAddress = Net.localAddress(this.fd);
            }

            return this;
        }
    }
}

/*
	ServerSocketChannel的重点就是用于监听TCP连接，所以accept也是最核心的方法
	使用accept方法来获取tcp连接，并返回这个连接对应的Channel
*/
public SocketChannel accept() throws IOException {
    synchronized(this.lock) {
        if (!this.isOpen()) {
            throw new ClosedChannelException();
        } else if (!this.isBound()) {
            throw new NotYetBoundException();
        } else {
            SocketChannelImpl var2 = null;
            int var3 = 0;
            FileDescriptor var4 = new FileDescriptor();
            InetSocketAddress[] var5 = new InetSocketAddress[1];

            InetSocketAddress var6;
            try {
                this.begin();
                if (!this.isOpen()) {
                    var6 = null;
                    return var6;
                }

                this.thread = NativeThread.current();

                do {
                    var3 = this.accept(this.fd, var4, var5);
                } while(var3 == -3 && this.isOpen());
            } finally {
                this.thread = 0L;
                this.end(var3 > 0);

                assert IOStatus.check(var3);

            }

            if (var3 < 1) {
                return null;
            } else {
                IOUtil.configureBlocking(var4, true);
                var6 = var5[0];
                var2 = new SocketChannelImpl(this.provider(), var4, var6);
                SecurityManager var7 = System.getSecurityManager();
                if (var7 != null) {
                    try {
                        var7.checkAccept(var6.getAddress().getHostAddress(), var6.getPort());
                    } catch (SecurityException var13) {
                        var2.close();
                        throw var13;
                    }
                }

                return var2;
            }
        }
    }
}
```





#### Buffer详解

#### Buffer

- Buffer的主要成员变量

```java
	private int mark = -1;		//标记，可以通过reset()方法回到标记位置以重复读
    private int position = 0;		//下一个要读入数据的索引
    private int limit;		//表示缓存区可操作数据的大小，即limit后的数据不能读写。写入时，limit等于buffer的容量（即capacity的大小）；读取时,limit等于缓存区数据量
    private int capacity;		//缓存区容量
```

- 以上成员一定遵守：0 <= mark <= position <= limit <= capacity

- 常用API

```java
/*
	将limit设置为当前写到了的位置，将position设置为0,这样就是从第0位置开始读到当前写到的位置（当前position的位置）
*/
public final Buffer flip() {
        limit = position;
        position = 0;
        mark = -1;
        return this;
}

/*
	计算出可读取的剩余长度
*/
public final int remaining() {
    return limit - position;
}
/*
	将当前读取的下标重置到mark标记的位置，这样就可以从mark的位置再次读取，配合mark()使用
*/
public final Buffer reset() {
    int m = mark;
    if (m < 0)
        throw new InvalidMarkException();
    position = m;
    return this;
}

/*
	reset是用来从标记位置继续操作，而rewind则是从缓存区的最开始位置开始操作
*/
public final Buffer rewind() {
    position = 0;
    mark = -1;
    return this;
}

/*
	这个方法用来清空缓冲区，但是从代码可以看到仅仅是重置了所有的参数，并没有实际的清空缓存区的内容
*/
public final Buffer clear() {
    position = 0;
    limit = capacity;
    mark = -1;
    return this;
}

```

