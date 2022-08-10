### redis的数据对象

- 使用命令 type 可以查看键对应的数据对象的类型



- redis一共有5种常用的数据对象和3种特殊的数据对象：String、List、Hash、Set、ZSet、geospatial、hypeloglog、bitMap



- 每个数据对象都是用 redisObject这个数据结构来表示的，redisObject有几个重要的字段

```c
typeof 占4个字节，表示对象的类型
encoding 占4个字节，表示编码，也就是底层使用的数据结构
*ptr 指向对应的数据结构的指针
```



#### String

##### String类型的数据结构



- String 的底层由3种数据结构组成，分别是int、embstr、raw
- 如果存入的数据是整形，那么就会用int数据结构存储，此时可以进行incr，decr，incrby，decrby对数据进行原子增加或减少操作
- 如果存入的数据是较短的字符串（长度小于等于44位）,使用embstr来进行存储，长于44位后使用raw来进行存储；这两种数据结构都是SDS字符串，不同的是embstr类型的string对象的object与sdshdr是一块连续的内存，这是对raw的一种优化，因为embstr编码只需要分配一次内存且回收时只需要进行一次内存的回收，但是当对embstr进行修改时，编码会被修改为raw，因为重新分配内存

