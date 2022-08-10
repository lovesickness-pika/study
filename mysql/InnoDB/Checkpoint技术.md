### 

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
- checkpoint_age > sync_water_mark很少出现，除非设置的重做日志文件太小或在进行类似 LOAD DATA的 BULK INSERT操作，此时进行Sync_Flush操作，刷新足够的页使得checkpoint_age < async_water_mark



- InnoDB1.2.x以前会阻塞用户线程，mysql5.6以后这部分操作也放入了单独的Page Cleaner Thread线程中

##### Dirty page too much

- 脏页数量太多，导致Innodb强制进行Checkpoint，可以使用参数 innodb_max_dirty_pages_pct 控制