### 哈希表如何解决哈希冲突



#### Hash函数

1. 直接定址法
2. 取余法
3. 平方取中法
4. 折叠法

#### 开放定址法

开放地址法中的删除操作不能直接删除，只能设置删除标记，因为可能会中断查找路径

- 线性探测，以增量序列线性探测
- 平方探测，以增量序列1，-1,4，-4且q<=tableSize/2试探下一个地址
- 随机探测，使用伪随机数列进行再探测



#### 链地址法

将hash冲突的节点以链表的方式放在这个下标上

#### 再哈希法

通过其它的hash函数进行再hash

#### 公共溢出区

将产生冲突的节点放到公共溢出区中，每次在hash表中没找到的数据都要遍历这个公共溢出区

