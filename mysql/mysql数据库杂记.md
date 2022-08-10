#### undo log

- undo log是用来完成事务回滚操作的，在每次进行写操作时，undo log中总会存放一条反向的逻辑语句，当需要回滚的时候，就通过反向补偿的方式将数据回滚到写操作之前的状态；这里是通过逻辑操作将数据库进行恢复的，所以数据结构和页本身是可能与之前不一样，只是存储的数据一样；undo log存放在表的共享空间中；并且MVCC的实现也是通过undo log实现的，当读取到某行已经被其它事务占用后，就会通过undo log读取之前的版本，以实现非锁定读；undo log的产生也会伴随着redo log，因为undo log本身也是需要持久化保护的





#### 二阶段提交

- 为什么必须有“两阶段提交”呢？这是为了让两份日志之间的逻辑一致

1. 先写 redo log 后写 binlog。假设在 redo log 写完，binlog 还没有写完的时候，MySQL 进程异常重启。
   由于我们前面说过的，redo log 写完之后，系统即使崩溃，仍然能够把数据恢复回来，所以恢复后这一行 c 的值是 1。
   但是由于 binlog 没写完就 crash 了，这时候 binlog 里面就没有记录这个语句。因此，之后备份日志的时候，
   存起来的 binlog 里面就没有这条语句。然后你会发现，如果需要用这个 binlog 来恢复临时库的话，
   由于这个语句的 binlog 丢失，这个临时库就会少了这一次更新，恢复出来的这一行 c 的值就是 0，与原库的值不同。

2. 先写 binlog 后写 redo log。如果在 binlog 写完之后 crash，由于 redo log 还没写，崩溃恢复以后这个事务无效，
   所以这一行 c 的值是 0。但是 binlog 里面已经记录了“把 c 从 0 改成 1”这个日志。所以，
   在之后用 binlog 来恢复的时候就多了一个事务出来，恢复出来的这一行 c 的值就是 1，与原库的值不同。



#### Group Commit



#### Binary Log Group Commit





Centos7 完全卸载MySQL8.0
1、查看mysql安装了哪些东西

rpm -qa |grep -i mysql
1
执行结果如下：

2、开始卸载

依次执行 yum remove 包名
例如：yum remove mysql-community-client-8.0.22-1.el7.x86_64
1
2
3、查看是否卸载完成，执行步骤（1）的命令
4、查找mysql相关目录

find / -name mysql
1
执行结果如下，具体根据自己的目录而定

5、删除相关目录

依次删除上面得到的目录：
例如：rm -rf /var/libmysql
1
2
6、删除配置文件：

rm -rf /etc/my.cnf
1
7、删除/var/log/mysqld.log（如果不删除这个文件，会导致新安装的mysql无法生存新密码，导致无法登陆）

rm -rf /var/log/mysqld.log
————————————————
版权声明：本文为CSDN博主「@hhr」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/weixin_36332085/article/details/110428594
