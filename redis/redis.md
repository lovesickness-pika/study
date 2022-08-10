## redis

##### 前置知识

- 计算机中，数据是存在磁盘中的，想要从磁盘中取数据，有两个问题：寻址，带宽，磁盘的寻址速度为ms级别，带宽一般为几百M或一两G，而内存的寻址速度为ns级别，磁盘比内存在寻址上慢了10w倍
- 磁盘中有磁道和扇区，一扇区为512byte，但如果以一扇区为最小存取单位，那么索引的代价会变大，所以操作系统访问磁盘的时候，有一个4k对齐，即每次都以4k为基本单位





- redis单进程单线程处理对用户数据操作，但redis内部并非只有一个线程
- redis保证每连接内的顺序性，即每个客户进程的所有线程的顺序性





#### redis常用命令

##### 通用命令

```shell
auth password		#密码验证

ping		#连接检查

quit		#断开连接

save		#RDB持久化

bgsave		#fork子进程进行RDB持久化


```

##### String命令

```shell
set key value  #创建一个string对象

mset key1 value1 key2 value2...  #创建多个字符串对象

get key		#获取string

mget key1 key2...		#获取多个字符串对象

del key value  #删除一个对象

append key value  #给string追加字符串
```



##### List命令

```shell
lpush key value1 value2...		#从队列头部插入值

rpush key value1 value2...		#从队列尾部插入值

lpop key [count]		#取得队列头部的值，可以用count指定数量

rpop key		#取得队列尾部的值

lindex key		#取得指定下标的值（从0开始）

lrem key count value  #删除list中count个value（从左到右）

lrange key start stop		#取从start下标到stop下标的值，闭区间

lpushx和rpush会先判断集合存不存在
```



##### Set命令

```shell
sadd key value1 value2...		#添加数据到set集合

spop key count		#从set集合中取得count个数

scard key		#获取set集合的元素数量

smembers key		#获取set集合的所有元素

sinter key1 key2...		#获取set集合的交集

sinterstroe destination key1 key2...	#获取set集合的交集并存在新的集合中

sunion key1 key1...		#获取set集合的并集

sunionstore key1 key2...	#获取set集合的并集并存放在新的集合中

sismember key member		#判断一个值是否存在于set集合中

srandmember key count	#随机的从set集合中获取count个数
```



##### Hash命令

```shell
hset key field1 value1 field2 value2...		#创建一个hash对象并设置一些字段

hget key field		#获取hash对象的字段的对应的值

hmget key field1 field2...		#获取hash对象的多个字段对应的值

hlen key		#获取hash对象的字段数

hkeys key 		#获取hash对象的所有字段
```



##### ZSet命令

#####  



#### redis常用参数

##### redis的conf文件常用参数

```properties
protected-mode yes/no		#保护模式，打开外网就访问不到了,默认yes

port	#端口号，连接时使用ip加端口号连接，ip通过ifconfig查看

daemonize	yes/no 			#默认情况redis不是在后台运行的，设置daemonize可以使redis后台启动

pidfile	 **/**/redis-6379.pid	#pidfile当 Redis 在后台运行的时候， Redis 默认会把 pid 文件放在/var/run/redis.pid，你可以配置到其他地址。当运行多个 redis 服务时，需要指定不同的 pid 文件和端口

bind	127.0.0.1		#bind指定 Redis 只接收来自于该 IP 地址的请求

timeout		#设置客户端连接时的超时时间

loglevellog 	#等级分为 4 级， debug, verbose, notice, 和 warning。生产环境下一般开启 notice

logfile		#配置 log 文件地址，默认使用标准输出，即打印在命令行终端的窗口上

databases	#设置数据库的个数，可以使用 SELECT 命令来切换数据库。默认使用的数据库是 

save n m			#Redis 进行数据库镜像的频率,n秒内发生了m个key的变化则进行持久化

rdbcompression	#在进行镜像备份时，是否进行压缩

dbfilename		#镜像备份文件的文件名,默认为dump.rdb	
```





##### 什么是redis

- redis是一款存储kv对的nosql数据库





#### redis的持久化





