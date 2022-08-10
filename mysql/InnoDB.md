### 

- 查看参数：

```
show variables like '参数名';		#可以使用%匹配
```

- 查看innodb 引擎状态（注意：这个状态是指过去一段时间的状态）：

```
show engine innodb status;
```

## 内存区

- innodb的内存区有：缓存池（innodb_buffer_pool）、重做日志缓存池（redo log_buffer）、额外缓存池（innodb_additional_mem_pool_size）

### 缓存池

- 简单来说，缓存池是一块内存区域
- 缓冲池设计的目的是为了协调cpu速度与磁盘读写速度的差距

#### 缓存池管理

- 可以通过表**INNODB_BUFFER_POOL_STATUS**来观察缓存池的运行状态

LRU list用来管理缓存池中页的可用性，Flush list用来管理将脏页刷新回磁盘

##### LRU list

- innodb中缓存池的页大小默认为16k，使用LRU算法管理缓存池中的页，并对该算法进行了些许优化：添加了一个midpoint位置，新加入的页不是直接放到LRU首端，而是放在midpoint位置，midpoint位置由参数innod_old_blocks_pct控制；在innodb中，将midpoint前面的列表称为new列表，将后面的列表称为old列表；
- innodb中使用参数innodb_old_blocks_time来决定读取到mid位置的数据多久后会加入到LRU列表的首端；当操作扫描等可能将热点数据扫出列表的操作时，可以将此参数调大
- innodb从1.0.x版本开始支持压缩页，由于页面大小不同，LRU列表有了一些变化：对于16k的页，仍使用LRU列表，对于小于16k的列表，使用unzip_LRU列表；需要注意的是，**查看innodb引擎状态时，LRU中的列包含了unzip_URL的列**，即总页数为LRU而非LRU+unzip_LRU
- 可以通过表**INNODB_BUFFER_PAGE_LRU**来观察每个LRU列表的每个页的具体信息

##### LRU算法

- 最少最近使用（Lasted Recent Used），innodb将最频繁使用的页放在LRU队列的前端，将最少使用的页放在尾端；当缓存池无法存取新页时，会释放尾端页。通常数据库都是使用LRU算法进行缓存池管理

##### Flush list

- 当LRU列表中的页被修改后，称改页为**脏页**：即缓存区中页与磁盘中的页产生了不一致
- 当产生脏页后，数据库会通过**CHECKPOINT机制**将脏页刷新回磁盘，而**Flush列表中的页则为脏页**
- 可以通过表**INNODB_BUFFER_PAGE_LRU + OLDEST_MODIFICATION条件**来观察每个LRU列表的每个页的具体信息

##### 缓存池参数

- 缓存池大小：innodb_buffer_pool_size
- 缓存池个数：innode_buffer_pool_instances，默认为1
- 新插入页在LRU列表的位置：innodb_old_blocks_pct，默认为5/8
- mid位置数据刷新到前端的时间：innodb_old_blocks_time，默认1000

### 重做日志缓冲

##### 重做日志缓冲参数

- 重做日志缓冲大小：innodb_log_buffer_size，默认为8MB
- Check point选择：innodb_fast_shutdown，默认为1

##### 重做日志缓冲池中的内容刷新到重做日志

- Master Thread每秒进行一次刷新
- 每次事务提交时
- 重做日志缓冲池剩余空间小于1/2时

### CHECKPOINT技术

##### 概述

- 页操作都是先在缓冲池中完成的，缓存区的页修改后就变成了脏页，但每次都刷新到磁盘会使得性能非常差（一个页中有很多数据，每次发生修改都进行刷新，会导致页不停的在被刷新），且如果刷新时发生宕机，数据将不能恢复，所以目前数据库基本上都使用了**Write Ahead Log策略**，即先写重做日志，再修改页
- Write Ahead Log策略可以通过重做日志恢复数据，但不可能只通过重做日志来完成数据的持久化，所以有了**Checkpoint（检查点）技术**来解决一些问题：
  - 缩短数据库的恢复时间
  - 缓冲池不够用时，将脏页刷新到磁盘
  - 重做日志不可用时，刷新脏页
- InnoDB使用**LSN**标记版本，LSN（Log Sequence Number）：8字节，在每个页中都有LSN，重做日志和Checkpoint也都有LSN
- checkpoint做的事情实际上就是在适当的时间将适当的地方的适当的数量的脏页刷新到磁盘，使得缓冲池有足够可用的页
- InnoDB内部有两种Checkpoint：**Sharp checkpoint、Fuzzy Checkpoint**，通过参数innodb_fast_shutdown选择，默认为Sharp checkpoint

#### Sharp Checkpoint

- 将所有脏页刷新到磁盘
- innodb只在数据库关闭时发生Sharp Checkpoint

#### Fuzzy Checkpoint

- InnoDB使用Fuzzy Checkpoint进行页的刷新，即只刷新一部分脏页
- 可以使用innodb状态观察刷新页

InnoDB会在以下情况下进行Fuzzy Checkpoint：

- Master Thread Checkpoint
- FLUSH_LRU_LISTCheckpoint
- Async/Sync Flush Checkpoint
- Dirty Page too much Checkpoint

##### Master Thread Checkpoint

- Master Thread 以每秒或没十秒的速度从脏页列表刷新一定比例的页到磁盘
- 此过程是异步的

##### FLUSH_LRU_LIST Checkpoint

- InnoDB为了保证LRU有足够的空闲页供使用，会移除LRU尾端的页，如果移除的页中有脏页，就需要进行Checkpoint
- mysql5.6以前，Innodb需要保证有100个空闲页可用，即少于100个空闲页就会移除LRU尾部的页，且这个检查LRU是否有足够的空间的操作发生在用户查询线程中，这会阻塞用户的查询操作；mysql5.6以后，可以通过参数 innodb_lru_scan_depth 修改LRU中可用页的数量，并且这个检查放在了专门的 Page Cleaner中

##### Async/Sync Flush Checkpoint

- 这个检查点是为了保证重做日志循环使用的可用性

- 若重做日志不可用，需要强制将一些页刷新回磁盘，此时的脏页是从脏页列表选取的

Async/sync Flush Checkpoint 过程：

1. 将已经写入到重做日志的LSN记为 redo lsn
2. 将已经刷新回磁盘最新页的LSN记为 checkpoint_lsn
3. 定义：
   1. checkpoint_age = redo_lsn - checkpoint_lsn
   2. async_water_mark = 75% * total_redo_log_file_size
   3. sync_water_mark = 90% * total_redo_log_file_size

- 当 checkpoint_age < async_water_mark，不需要刷新
- 当 async_water_mark < checkpoint_age < sync_water_mark，进行Async Flush，刷新足够的页使得checkpoint_age < async_water_mark
-  checkpoint_age > sync_water_mark很少出现，除非设置的重做日志文件太小或在进行类似 LOAD DATA的 BULK INSERT操作，此时进行Sync_Flush操作，刷新足够的页使得checkpoint_age < async_water_mark



- InnoDB1.2.x以前会阻塞用户线程，mysql5.6以后这部分操作也放入了单独的Page Cleaner Thread线程中

##### Dirty page too much

- 脏页数量太多，导致Innodb强制进行Checkpoint，可以使用参数 innodb_max_dirty_pages_pct 控制

### Master Thread的工作方式

