#### IO

##### 流的概念和作用

- 流：代表任何有能力产出数据的数据源对象或者是有能力接受数据的接收端对象<Thinking in Java>

- 流的作用：为数据源和目的地建立一个输送通道

**采用数据流的目的是使数据传输独立于设备**











### IO模式

#### NIO

- java1.4引入
- 面向缓冲区、基于通道
- java.nio.*;

##### NIO的三的核心：Channel，Buffer，Selector



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
	将limit设置为当前写到了的位置，将position设置为0,，这样就是从第0位置开始读到当前写到的位置
*/
public final Buffer flip() {
        limit = position;
        position = 0;
        mark = -1;
        return this;
}
```



#### Channel

- Channel只能与Buffer交互   

#### Selector

