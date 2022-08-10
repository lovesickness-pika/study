### 插入缓冲Insert Buffer

##### 概述

- Insert Buffer与数据页一样，都是物理页的一个组成部分，其设计出来的目的是为了解决离散插入的性能问题
- InnoDB开创性的设计了Insert Buffer，对于非聚集索引的插入或更新操作，并非每次都直接插入到索引页，而是先判断插入的非聚集索引是否在缓冲池中，若在，则直接插入；若不在，则先放到一个Insert Buffer对象中（并没有实际插入到非聚集树的叶子节点），然后再以一定的频率进行Insert Buffer和辅助索引叶子节点的merge（合并）操作，这时通常能将多个插入合并到一个操作中（因为在一个索引页中），大大提高了对于非聚集索引的插入性能



##### 插入缓存使用限制

1. 插入缓存只能用于辅助索引。
2. **索引不能是唯一列**。因为在插入缓冲时，不会去查找索引页来验证索引唯一性。如果去查找又会有离散读取的情况发生，导致insert 怒份儿失去了意义--》既然都已经将索引页读入了缓冲区，就不需要进行缓冲了

当满足这两个条件时，InnoDB存储引擎会使用Insert Buffer，这样就能提高插入操作的性能了。但由于Insert Buffer毕竟是存放在内存区，未进行持久化，若应用程序进行了大量的插入操作后宕机，由于Insert Buffer并没有与实际的非聚集索引合并，会导致数据库要进行长时间的恢复。



#### Insert Buffer的内部实现

##### Insert Buffer结构

- insert buffer在内存中的结构为B+树，全局只有一棵Insert Buffer树，存放在共享表空间中，默认为ibdata1；因此，试图通过独立表空间恢复表中数据时，往往会导致CHECK TABLE失败，这是因为表的辅助索引中的数据可能存放在共享表空间中，所以通过ibd文件进行恢复后，还需要进行REPAIR TABLE操作在重建表上的辅助索引
- Insert Buffer这棵B+树中，非叶子节点存放的是查询的search key，其构造为|space|marker|offset|;共占九个字节，
  - space占4个字节，表示待插入记录所在的表空间id,在Innodb中，每个表都有唯一的space id
  - marker占一个字节，用于兼容老版本的Insert Buffer
  - offeset表示页所在的偏移量（通过这个得到页的位置）
- Insert Buffer B+树中，叶子节点存放的是插入记录，其结构为|space|marker|offset|metadata|辅助索引信息|
  - 前三个字段与非叶子节点的作用一样
  - metadata占用4字节，其字段为|IBUF_REC_OFFSET_COUNT|IBUF_REC_OFFSET_TYPE|IBUF_REC_OFFSET_FLAGS|，其中COUNT占用2个字节，用来排序每个记录进入Insert Buffer的顺序；
- 因为启用了Insert Buffer索引后，辅助索引页的记录可能会被插入到Insert Buffer中，为了保证每次merge Insert Buffer成功，还需要一个特殊的页来标记每个辅助索引页（space，page_no）的可用空间，这个页的类型为Insert Buffer Bitmap；每个Insert Buffer Bitmap用来追踪16384个辅助索引页，也就是256个区，每个Insert Buffer Bitmap在16384页中的第二页，每个辅助索引页在Insert Buffer Bitmap中占用4位

##### Insert Buffer实现步骤

- 当一个索引要插入到页，如果这个页不在缓存池中，那么InnoDB存储引擎首先构造一个search key，接下来查询Insert Buffer这颗B+树，在将这条记录插入到Insert Buffer B+树的叶子节点中

#### Merge Insert Buffer

简单来说，Insert Buffer与辅助索引树的合并可能发生在以下几种情况：

	- 辅助索引树被读取到缓冲池中时
	- Insert Buffer Bitmap追踪到该辅助索引页已无可用空间时
	- Master Thread

##### 具体实现

1. 当辅助索引页被读取到缓冲池中时，需要检查Insert Buffer Bitmap页，然后确认该辅助索引页是否有记录存放在Insert Buffer B+树中；若有，则将该记录从Insert Buffer B+树中插入到索引树，这里将该页的多条插入记录合并到了1次
2. 若通过Insert Buffer Bitmap发现经过某次插入操作后该辅助索引树的可用空间会小于1/32，则会强制进行合并操作
3. Master Thread每秒和每10秒都可能进行合并操作

### Change Buffer

##### 概述

- InnoDB从1.0.x版本开始，就引入了Change Buffer，其可以视为Insert Buffer的升级版本；Change Buffer可以对DML都进行缓冲，分别为insert buffer、delete buffer、purge buffer

##### 管理Change Buffer

- InnoDB提供了参数来开启各种buffer，参数可选值为：inserts、deletes、purges、changes、all、none；其中changes表示启用inserts、deletes，all表示全部启用，none表示全不启用；默认值为all，全部启用
- 参数**innodb_change_buffer_max_size**用于控制Change Buffer最大使用内存的数量，默认值25，代表内存池的1/4

#### 查看Change Buffer

![image-20210719201259040](C:\Users\皮卡丘\AppData\Roaming\Typora\typora-user-images\image-20210719201259040.png)

- seg size显示当前Insert Buffer的大小；free list 表示空闲列表的大小；size表示已经合并页的数量
- merge operation中显示合并页的具体信息（增删改各种操作合并的数）
- discarded operation表示当发生页合并时，表已经删除





### 两次写 doublewrite

##### 问题的引出

- **部分写失效**：innodb在写入某个页到表中时，若发生了宕机，这种情况叫做部分写失效
- 部分写失效失效问题无法直接通过重做日志恢复解决，因为此时这个页已经发生损坏，而重做日志是对页进行物理操作，所以要在应用重做日志恢复前，想办法先还原该页，这就是doublewrite

##### 概述

- doublewrite带给innodb的是数据页的可靠性
- doublewrite是页的一个副本，当发生部分写失效时，先用这个副本还原该页

#### doublewrite的组成

- doublewrite由两部分组成，一部分为内存中的doublewrite，大小为2MB，另一部分为物理磁盘上共享表空间的中连续的128页，即2个区

#### doublewrite步骤

- 在对缓冲池的脏页刷新时，并不直接写磁盘，而是通过memcpy函数先将脏页复制到内存中的doublewriter buffer，之后通过doublewrite buffer再分两次，每次1MB顺序的写入共享表空间，并立刻调用fync函数，同步磁盘，避免缓冲写带来的问题。
- 在这个过程中，由于doublewrite页是连续的，因此这个过程是顺序写的，开销并不大，完成doublewrite页的写入后，再将doublewrite buffer中的页写入各个表空间，此时的写入为离散的。



### 自适应hash索引

##### 概述

- InnoDB会监控对表上个索引页的查询，如果观察到建立hash索引可以带来速度提升，则建立hash索引，称之为**自适应哈希索引**（Adaptive Hash Index，AHI）；InnoDB引擎会自动根据访问频率和模式来为某些热点页建立hash索引

### 异步IO

##### 概述

- 为了提高性能，当前的数据库都采用异步IO的方式来处理磁盘操作

