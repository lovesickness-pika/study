### Master Thread

##### 概述

- Master Thread是一个核心的线程，负责将缓冲池中的数据异步刷新到磁盘，保持数据的一致性

#### Master Thread 工作机制

- Master Thread具有**最高的线程优先级别**
- Master Thread内部由多个循环组成：
  - 主循环（loop）
  - 后台循环（background loop）
  - 刷新循环（flush loop）
  - 暂停循环（suspend loop）
- loop主要分两种操作：十秒一次和每秒一次的操作
- Master Thread中所有刷新脏页的操作都会放到单独的Page Cleaner Thread中进行

##### loop每秒一次

1. 刷新日志缓冲到磁盘
2. 合并插入缓冲。合并插入缓冲并非每秒都会进行，innodb引擎会根据前一秒的io情况进行判断：前一秒内io次数小于5次，执行合并插入缓冲
3. 刷新脏页：innodb通过判断当前缓冲池中脏页的比例是否超过参数*innodb_max_dirty_pages_pct*（默认值75%）,如果超过了这个阈值则进行磁盘同步
4. 如果没有用户活动，切换到background loop

##### loop十秒一次

1. 刷新脏页。innodb引擎会根据前一秒的io情况进行判断：前10s内io次数小于200，刷新innodb_io_capacity个脏页到磁盘
2. 合并插入缓冲,数量为innodb_io_capacity的5%
3. 刷新日志缓冲到磁盘
4. 执行full purge，删除无用的undo页
5. 刷新脏页。innodb存储引擎判断当前脏页的比例，如果超过70%，刷新innodb_io_capacity个脏页，否则刷新10个脏页

##### background loop

1. 删除无用的undo页
2. 合并innodb_io_capacity的5%个插入缓冲
3. 跳回主循环
4. 跳转到flush loop

##### flush loop

1. 不断刷新innodb_io_capacity个脏页
2. 切换到suspend loop

##### suspend loop

1. 将Master Thread挂起，等待事件发生。

### IO Thread

##### 概述

- 在InnoDB存储引擎中大量的使用了AIO来处理IO请求，这样可以极大的提高数据库的性能；IO Thread的工作是负责这些IO请求的回调



### Purge Thread

##### 概述

- 在事务提交后对undolog 的回收工作



### Page Cleaner Thread

##### 概述

- 将脏页的刷新使用单独线程完成，减轻Master Thread的负担
- 阻塞用户查询线程