### redis中的数据库



```c
struct redisServer{
    //...
    redisDb *db;		//redis的数据库
    
    int dbnum;		//数据库数量，默认16
    //...
}
```

- redis将所有的数据库都存放在服务器状态server.h/redisServer结构的db数组中

- dbnum属性的值由服务器配置的database选项决定，默认16，也就是16个数据库

```c
typedef struct redisDb {
    dict *dict;                 //当前数据库的键空间，保存着当前数据库的所有键值对
    dict *expires;              //所有的过期时间
    dict *blocking_keys;        /* Keys with clients waiting for data (BLPOP)*/
    dict *ready_keys;           /* Blocked keys that received a PUSH */
    dict *watched_keys;         /* WATCHED keys for MULTI/EXEC CAS */
    int id;                     //当前数据库的id
    long long avg_ttl;          /* Average TTL, just for stats */
    unsigned long expires_cursor; /* Cursor of the active expire cycle. */
    list *defrag_later;         /* List of key names to attempt to defrag one by one, gradually. */
} redisDb;
```

- redis中的数据库以字典数据结构存放键值对，所以很多针对数据库的操作都是通过键空间进行的处理

##### redis对键空间进行的维护操作

- 在读取某个键之后，会根据键是否存在更新服务器的键空间命中hit与不命中miss次数，这两个值可以在INFO stats命令的keyspace_hits属性和keyspace_misses属性查看
- 读取某个键之后，会更新键的LRU时间，这个值用于计算键的闲置时间，使用命令*object idletime*查看
- 读取某个键之前，会判断该键是否过期
- 如果某个客户端使用*watch*命令监视了某个键，对这个键进行写操作时，会将其标记为脏（dirty），从而让事务程序注意到这个键已经被修改
- 服务器每对一个键进行写操作，会将dirty计数器的值+1，用于间隔性持久化操作以及复制操作
- 如果服务器开启了数据库通知功能，那么在对键进行修改后，服务器会按配置发送相应的数据库通知

#### redis客户端切换数据库

- redis客户端的redisClient结构的db属性记录了客户端当前的目标数据库，这个指针指向redisServer的db数组中的一个元素，默认指向第0个，当发生数据库切换时（执行**select**命令），就是改变指针的指向
- 在执行危险操作时（flushdb命令等），应该显式的切换到要执行命令的数据库，以防操作失误