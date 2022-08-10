### ConcurrentSkipListMap







#### 术语

- base层：最下面的层（第0层），节点类型为Node。
- index层：第1层或以上，节点类型为Index。
- node节点：base层的节点。
- index节点：index层的节点。
- headIndex节点：index层的头节点。
- marker：不存储数据，只是为了删除时使用。
- 被标记为删除：Node的value为置为null。简称为“被标记”。