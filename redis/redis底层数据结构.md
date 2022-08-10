### redis底层数据结构

- 可以使用命令 object encoding 查看某个redis对象的底层数据结构



### SDS 简单动态字符串 

```c
struct sdsstr{
    int len;		//buf数组已使用的长度
    int free;		//buf未使用的长度
    char buf[]		//字节数组，用于保存字符串
}
```

- 一个SDS简单动态字符串有三个属性，长度，未使用空间，以及字符数组

##### SDS与C字符串

- C字符串使用'\0'代表字符串结尾，每次获取长度和每次判断都是到\0结束，所以C字符串的空间总是长度加一，因此每次修改都需要重新分配空间



##### SDS的优势

- 判断长度的时间复杂度为O(1)，直接取得len的值即可
- 杜绝了缓存池溢出，每次修改字符串前要先进行扩容
- 减少内存重分配次数，通过预分配空间实现，未使用的空间的大小存放在free字段中
- 二进制安全，不以\0结尾，不对字符有任何转义
- 兼容部分C字符串函数，也使用\0结尾



### 双向链表 list

```c
struct listNode{
    //前置节点
    struct listNode *prev;
    //后置节点
    struct listNode *next;
    //节点的值,可以为不同类型的对象
    void *value;
}listNode;

struct list{
    //表头节点
    listNode *head;
    //表尾节点
    listNode *tail;
    //节点数量
    unsigned long len;
    //节点值复制函数
    void *(*dup)(void *ptr);
    //节点值释放函数
    void (*free)(void *ptr);
    //节点值比较函数
    int (*match)(void *ptr, void *key);
}
```

- 链表可用于实现Redis的各种功能：列表键、发布与订阅、慢查询、监视器

#### redis的链表的特性

- 双端
- 无环
- 带表头指针和表尾指针
- 带链表长度计数器



### 字典

- 字典源码路径：dict.h

```c
typedef struct dictType {
    uint64_t (*hashFunction)(const void *key);
    void *(*keyDup)(void *privdata, const void *key);
    void *(*valDup)(void *privdata, const void *obj);
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);
    void (*keyDestructor)(void *privdata, void *key);
    void (*valDestructor)(void *privdata, void *obj);
    int (*expandAllowed)(size_t moreMem, double usedRatio);
} dictType;

typedef struct dict {
    dictType *type;
    void *privdata;
    dictht ht[2];
    long rehashidx; /* rehashing not in progress if rehashidx == -1 */
    int16_t pauserehash; /* If >0 rehashing is paused (<0 indicates coding error) */
} dict;

```



- redis的字典使用hash表作为底层实现，使用链地址解决hash冲突
- MurmurHash2算法用来计算键的哈希值，特点是运算性能高，碰撞率低。
- 保存了节点数
- 保存了哈希表的大小

##### 字典的扩容

- redis 的字典实际上使用了两个hash表，当负载因子为1时进行rehash重新计算在扩容后的hash表的位置（有特殊情况，正发生BGSAVE或BGREWRITEAOF时到负载因子为5才进行扩容）
- redis使用渐进式hash，渐进式的rehash将rehash键值对所需的计算工作均摊到对字典的每个添加、删除、查找和更新操作上。



#### 跳跃表

- 跳表源码路径：server.h

```c
typedef struct zskiplistNode {
    sds ele;
    double score;
    struct zskiplistNode *backward;
    struct zskiplistLevel {
        struct zskiplistNode *forward;
        unsigned long span;
    } level[];
} zskiplistNode;

typedef struct zskiplist {
    struct zskiplistNode *header, *tail;
    unsigned long length;
    int level;
} zskiplist;

```



- 跳跃表是一种有序数据结构，它通过在每个节点中维持多个指向其他节点的指针，从而达到快速访问节点的目的。
- Redis只在两个地方用到了跳跃表，一个是实现有序集合键，另一个是在集群结点中用作内部数据结构，除此之外，跳跃表没有其他用途。



#### 整数集合

- 整数集合是Redis用于保存整数值的集合抽象数据结构，它可以保存类型为 int16_t 、int32_t 或者 int64_t 的整数值，并且保证集合中不出现重复值。
- 整数集合可以升级不可以降级



#### 压缩列表

- 

