- mysql4.1以前是大小写敏感的

### SQL语句

#### DML语句

- 去重：DISTINCT

- 显示返回结果数：LIMIT offset，returnNum；只有一个参数m返回前m行， 两个参数m，n表示从第m行数据开始，返回n行；**mysql起始行从第0行开始**

- 排序：ORDER BY；当有多个参数时，从左到右进行排序；使用ASC进行顺序排序，使用DESC进行逆序排序

- 使用全文索引：

  ```mysql
  select * from table tableName where match(完整的索引列) against ('匹配值');
  ```

  

### 权限管理

#### mysql用户

- 创建用户

```mysql
create user 'username'@'ip_address' identified 'password';	
create user 'username'@'ip_address' identified with 'mysql_native_password' by 'password';
```

- 删除用户

```mysql
drop user username
```

- 添加权限

```mysql
grant replication slave on test.* to 'copy'@'从机ip';
flush privileges;
```



- 查看用户：mysql库下的user表

##### 连接远程mysql



### 存储引擎

- 数据实际上还是要存储到磁盘上的，而存储引擎就是：数据库管理系统如何存储数据、如何为存储的数据建立索引与如何更新、查询数据等技术的实现方法。
- InnoDB与MyISAM

|          | MyISAM                                                       | InnoDB                                                       |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 存储结构 | 每张表由三个文件存储：.frm存储表结构、.MYD(my data)存储数据文件、.MYI(my index)存储索引文件 | innodb有两种数据存储方式，一个是共享表空间，一个是独占表空间；innodb有一个变量叫做innodb_file_per_table，5.6以后默认值为1，此时使用的是独占表空间 |
| 应用场景 | 高性能的查询，不需要使用事务，适合读多并发少的场景（因为使用的是表锁） | 处理大容量数据，事务的使用                                   |
| 锁       | 支持表锁                                                     | 支持表锁、行锁                                               |

### 索引--innodb存储引擎

- 索引存在的意义是加快数据访问
- mysql的Innodb默认的是BTree（底层实际是**B+Tree**）+自适应hash，可以通过**show index from tablename**查看某个表中索引使用的数据结构
- 为什么使用B+树？
  - 索引存在的目的是为了加快数据访问速度，通过索引可以更快的定位到列的地址，但索引本身也是放在磁盘中的，也需要通过io读取到内存，影响io有两个方面：io次数与每次io访问的数据量；有序线性表显然不行，因为需要将整个表读取到内存，数据量太大；所有的二叉树都存在无限加深的问题，io次数太多；B树在节点处也保存了行数据，占据了一定的空间，并且不利于进行范围查询，所以使用B+树，在节点处只保存索引信息，在叶子节点处保存行信息

##### 索引覆盖

如果一个索引包含所需要的查询的所有字段的值，就叫做索引覆盖。即无需**回表**；

可以通过执行计划的type是否为index查看

##### 联合索引的索引失效问题



##### 聚簇索引与非聚簇索引

- 聚簇索引根据每张表的主键构造一棵B+树，并在叶子节点中存储行数据，每张表仅一个聚簇索引；其它辅助索引的叶子节点存储的是主键值；
- InnoDB根据主键聚集数据，如果表未定义主键，会根据非空的唯一索引构建聚簇索引树，否则会隐式的定义一个自增的6个字节的长整型字段作为主键
- 非聚簇索引中索引与数据文件是分离的，每个索引树的叶子节点存放的都是行记录的地址

### 锁

- mysql查看锁

```mysq
show engine innodb status;
```

- mysql设置输出锁信息

```mysql
set global innodb_status_output_locks=1;
```

- lock mode IX：意向锁
- lock mode X：临键锁

##### mysql锁的分类

- 根据对数据操作的类型分为：共享锁和排他锁
- 根据对数据操作的粒度分为：表锁和行锁

#### 共享锁和排他锁（写锁与读锁）--行级锁定

##### 读锁（S锁）

- 事务T1持有行r的S锁，当事务T2请求行r的S锁时，会立刻得到行r的S锁，结果事务T1和T2都持有行r的读锁

##### 写锁（X锁）

- 事务T1持有行r的T锁，当事务T2请求行r的X锁或S锁时，会被阻塞，等待T1释放X锁或者超出等待时间操作失败

#### 意向锁--表级锁定

- InnoDB支持多粒度锁定，允许行锁和表锁共存，在这种多粒度级别下，引入了意向锁。
- 如果没有意向锁：在行锁与表锁可能共存的的机制下，当事务T请求行锁时只需要判断当前表是否被其它事务持有表锁，然后再获取锁即可；但若事务T请求当前表的表锁时，需要确认每行数据的行锁，数据量越多，难度越大；
- 意向锁存在的意义：如果另一个任务试图在该表级别上应用共享或排它锁，则受到由第一个任务控制的表级别意向锁的阻塞。第二个任务在锁定该表前不必检查各个页或行锁，而只需检查表上的意向锁。；--**意向锁的作用就在于标志当前表有某行被持有锁**

- 意向锁分为 **意向共享锁（IS锁）**与**意向排他锁（IX锁）**

- 获取时机：事务在请求行r的S锁/X锁之前，会先请求该表的IS锁/IX锁
- IS锁的作用：当事务T1请求表t的**表级X锁**时，若其它事务持有IS锁，该请求被阻塞
- IX锁的作用：当事务T1请求表t的**表级X锁或表级S锁**时，若其它事务持有IX锁，该请求被阻塞

意向锁与**表级**锁的互斥性：

- 意向锁之间是兼容的

- |                   | 意向共享锁（IS锁） | 意向排他锁（IX锁） |
  | ----------------- | ------------------ | ------------------ |
  | 表级共享锁（S锁） | 兼容               | 互斥               |
  | 表级排他锁（X锁） | 互斥               | 互斥               |

### 事务

- ACID：Atomic、Consistency、Isolation、Durability
  - Atomic：原子性，事务中的操作要不全做，要不全不做；依赖undo log

##### 事务隔离级别

- 可以通过查看全局变量 transaction_isolation查看当前事务隔离级别

- 读未提交、读已提交、可重复读（默认）、串行化

  - RU：允许事务读取到其它事务已修改但还未提交的数据

  - RC：只允许事务读取到其它事务提交后的数据

  - RR：确保事务在多次从同一字段中读取的值是相同的

  - S：确保事务多次读取的返回值相同

| 隔离级别         | 脏读   | 不可重复读 | 幻读   |
| ---------------- | ------ | ---------- | ------ |
| Read Uncommitted | 可能   | 可能       | 可能   |
| Read Commited    | 不可能 | 可能       | 可能   |
| Repeatable Read  | 不可能 | 不可能     | 可能   |
| Serializable     | 不可能 | 不可能     | 不可能 |

- 脏读：事务读到另一个事务已经修改但还未提交的数据；
- 不可重复读：事务在先后两次读取同一个数据时取得的结果不一样
- 幻读：事务先后查询数据库查询结果的条数不同

##### undolog

- 回滚日志

- mysql使用事务前，要先关闭自动提交：**set autocommit=0**

##### 当前读

读取到的是数据的最新版本

- 所有的数据操作语句（增删改）以及加锁的查询语句（select ... lock in share mod;	select ... for update;）都是当前读

##### 快照读

- 查询语句是快照读；事务会在第一次进行快照读时产生read view

读取到的是数据的历史版本

**read view**

readview：事务在进行快照读时产生的读视图

- trx_list:系统活跃的事务id
- up_limit_id：列表中事务最小的事务id（低水位）
- low_limit_id：系统最大的事务id（高水位）
- creator_trx_id：表示生成该ReadView的事务的事务id

用ReadView判断哪个版本的数据可读的过程：

- 如果被访问版本的trx_id属性值小于ReadView中的min_trx_id值，表明生成该版本的事务在当前事务生成ReadView前已经提交，所以该版本可以被当前事务访问。
- 如果被访问版本的trx_id属性值与ReadView中的creator_trx_id值相同，意味着当前事务在访问它自己修改过的记录，所以该版本可以被当前事务访问。
- 如果被访问版本的trx_id属性值大于或等于ReadView中的max_trx_id值，表明生成该版本的事务在当前事务生成ReadView后才开启，所以该版本不可以被当前事务访问。
- 如果被访问版本的trx_id属性值在ReadView的min_trx_id和max_trx_id之间，那就需要判断一下trx_id属性值是不是在m_ids列表中，如果在，说明创建ReadView时生成该版本的事务还是活跃的，该版本不可以被访问；如果不在，说明创建ReadView时生成该版本的事务已经被提交，该版本可以被访问。

![image-20210712103719229](C:\Users\皮卡丘\AppData\Roaming\Typora\typora-user-images\image-20210712103719229.png)

**read view 产生的时机与事务隔离级别相关**：

- RU级别在每次进行快照读前生成新的read view；RC

- 性能分析：set profiling=1;	sql语句；	show profiles;	show profile for query 1;



mysql调优，从以下几个方面思考：

- 表结构 

- 执行计划
- 索引的使用
- 性能的监控
- sql语句的调整
- 参数的调整

事务的相关操作：

- ```mys
  select * from information_schema.innodb_trx;	#查看所有事务
  select * from information_schema.innodb_trx where trx_mysql_thread_id = connection_id();	#查看当前事务id
  ```

- 

#### MVCC

- Multi-Version Concurrency Control，多版本并发控制

- 用于提高并发过程中对数据库的读写效率

##### 前置知识

- 当前读：读取到的数据一定是最新的

- 快照读：读取到的数据不定是最新的（mysql会存储数据的历史版本，用于回滚）

- 每一行记录都有一些隐藏字段：

  | 字段名      | 说明                                                 |
  | ----------- | ---------------------------------------------------- |
  | DB_TRX_ID   | 创建或最后一次修改该数据的事务id                     |
  | DB_ROW_ID   | 当前表不存在主键和唯一键时被InnoDB自动创建的隐藏主键 |
  | DB_ROOL_PTR | 回滚指针，配合undolog使用                            |





### 日志



#### 二进制日志：Binlog

MySQL binlog记录所有数据库表结构变更以及表数据变更的二进制日志

##### 应用场景

- mysql主从复制：MySQL Replication在Master端开启binlog，Master把它的二进制日志传递给slaves来达到master-slave数据一致的目的
- 数据恢复

##### binlog写入

- 写入时机：binlog在**事务提交前**写入

- 事务执行过程中，会先将要保存的日志写到 binlog cache， 事务提交前，再将binlog cache中的write到page cache中，然后根据**sync_binlog**的值决定是否fsync到磁盘

  - **每个线程有自己 binlog cache （ binlog cache 是每个线程自己维护的），但是共用同一份 binlog 文件，保证了一个事务的binlog不能被拆开。**
  - write  和 fsync 的时机，是由参数 sync_binlog 控制的：

  1. sync_binlog=0 的时候，表示每次提交事务都只 write ，不 fsync （不建议这样设置）
  2. sync_binlog=1 的时候，表示每次提交事务都会执行 fsync ；
  3. sync_binlog=N(N>1) 的时候，表示每次提交事务都 write ，但累积 N 个事务后才 fsync 

  ![img](https://img-blog.csdnimg.cn/20190424162231963.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hhbnBlaXl1MTk5NQ==,size_16,color_FFFFFF,t_70)

记录在二进制日志中的事件的格式取决于二进制记录格式。支持三种格式类型：

- STATEMENT（5.7.7以后默认）：基于SQL语句的复制（statement-based replication, SBR）
- ROW（5.7.7以前默认）：基于行的复制（row-based replication, RBR）
- MIXED：混合模式复制（mixed-based replication, MBR）：一般语句使用statement，一些函数，statement无法完成主从复制，则采用row；

![img](https://img-blog.csdnimg.cn/20190326103256132.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ppU2h1aVNhblFpYW5MaQ==,size_16,color_FFFFFF,t_70)

binlog的使用：

- 查看binlog模式：selece @@binlog_format;
- 查看mysql是否开启binlog同步功能：show variables like 'log_bin'
- 启动binlog（该变量在mysql中是只读的，需要去配置文件中启动）：

```properties
[mysqld]  # 这一行必须有，否则会出错
#log_bin
log-bin = mysql-bin #开启binlog
binlog-format = ROW #选择row模式
server_id = 12345 #配置mysql replication需要定义，不能和canal的slaveId重复
```

- binlog相关命令：

```mysq
show binary logs   #获取binlog文件日志列表
show master status  # 查看当前正在写入的binlog文件
show master logs  # 查看master上的binlog文件
show binlog events  #查看第一个binlog文件内容
show binlog events 'mysql-bin.000002'  # 查看指定binlog文件内容， 如：查看mysql-bin.000002文件内容
```



#### 事务日志：undo log

##### 写入时机

- 在操作任何数据前，数据都将被备份

#### 事务日志：redo log

##### 写入时机

- 操作任何数据后，备份新的数据

##### mysql事务的二阶段提交

- redo log 有 prepare和commit两种状态

![img](https://static001.geekbang.org/resource/image/2e/be/2e5bff4910ec189fe1ee6e2ecc7b4bbe.png)

- 为什么必须有“两阶段提交”呢？这是为了让两份日志之间的逻辑一致

1. 先写 redo log 后写 binlog。假设在 redo log 写完，binlog 还没有写完的时候，MySQL 进程异常重启。
   由于我们前面说过的，redo log 写完之后，系统即使崩溃，仍然能够把数据恢复回来，所以恢复后这一行 c 的值是 1。
   但是由于 binlog 没写完就 crash 了，这时候 binlog 里面就没有记录这个语句。因此，之后备份日志的时候，
   存起来的 binlog 里面就没有这条语句。然后你会发现，如果需要用这个 binlog 来恢复临时库的话，
   由于这个语句的 binlog 丢失，这个临时库就会少了这一次更新，恢复出来的这一行 c 的值就是 0，与原库的值不同。

2. 先写 binlog 后写 redo log。如果在 binlog 写完之后 crash，由于 redo log 还没写，崩溃恢复以后这个事务无效，
   所以这一行 c 的值是 0。但是 binlog 里面已经记录了“把 c 从 0 改成 1”这个日志。所以，
   在之后用 binlog 来恢复的时候就多了一个事务出来，恢复出来的这一行 c 的值就是 1，与原库的值不同。

