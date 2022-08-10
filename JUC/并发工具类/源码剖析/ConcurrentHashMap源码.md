## ConcurrentHashMap源码剖析



### 一些重要的变量与常量

- 以下变量先过一遍就行，具体作用放具体场景理解

```java
/*与hash值有关*/
//表示此节点已转移到扩容后的新数组
static final int MOVED     = -1; // hash for forwarding nodes
//此节点为树节点
static final int TREEBIN   = -2; // hash for roots of trees
static final int RESERVED  = -3; // hash for transient reservations
//用来将hash值转化为正数
static final int HASH_BITS = 0x7fffffff; // usable bits of normal node hash


private transient volatile int sizeCtl;		//最重要的变量，用来表示一些状态和记录一些信息，放下面仔细讲

private transient volatile int transferIndex;		//扩容时的下标，与扩容一起讲

private transient volatile int cellsBusy;		//表示CounterCell[]正处于特殊状态，其实就是乐观锁
```

##### sizectl

**过一遍就行，具体作用具体场景理解**

- sizectl是一个用volatile修饰的int类型变量，起到记录扩容阈值、记录数组长度、记录扩容线程数、表示初始化状态的作用
  - sizectl为正数时，值等于ConcurrentHashMap的扩容阈值
  - sizectl为0时，表示尚未初始化
  - sizectl为-1时，表示正在初始化
  - sizectl<-1时，高16位去首位0用来**表示**（并非值就等于这个，但是可以通过这个数计算出）数组容量，低16位-1等于扩容的线程数





**看源码，切记，先梳理脉络，再抓细节**





### ConcurrentHashMap的初始化



- ConcurrentHashMap使用懒加载机制，具体开辟空间会在第一次开辟空间的时候

#### initTable方法

- ConcurrentHashMap使用initTable方法进行初始化，由于可能出现多个线程同时初始化的问题，所以使用sizeCtl=-1来表示线程正在进行初始化，所有初始化线程都使用CAS修改sizeCtl的值，修改成功的先进行一次DCL验证，然后初始化数组；其它线程会调yield方法让出时间片

```java
private final Node<K,V>[] initTable() {
    
    //判断是否要进行初始化
    while ((tab = table) == null || tab.length == 0) {
        //sizectl<0,代表有其它线程正在初始化或正在扩容，当前线程让出时间片
        if ((sc = sizeCtl) < 0)
            Thread.yield(); // lost initialization race; just spin
        
        //cas将sizectl从0修改为-1，修改成功则进入扩容
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                //双重验证
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    
                    //sizectl设置为table长度的0.75
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

- 为什么要进行双重验证？	与DCL单例一样，这里是防止在**((sc = sizeCtl) < 0)**到**(U.compareAndSwapInt(this, SIZECTL, sc, -1))**这个过程期间，其它线程已经将table完成了初始化



### ConcurrentHashMap插入值

ConcurrentHashMap实际插入值的方式其实与hashmap差不多，不过因为要保证线程安全，所以需要先做很多准备工作：

1. ConcurrentHashMap在计算hash值的时候多与一个常量HASH_BITS进行了按位与，这个数实际上是int类型正数的最大值，二进制中首位为0，剩余位全为一，这样保证了计算出来的hash值一定为正数，避免与ConcurrentHashMap内置的`MOVED`、`TREEBIN`、`RESERVED`这3个负数hash冲突。
2. 根据hash值计算的下标若为null，要使用CAS插入节点
3. 若发现该下标对应的节点正在扩容，要调用helpTransfer方法帮助扩容线程进行扩容
4. 在对应下标插入值时，要先取得该节点的对象锁
5. 插入值完成后，如果节点数增加了，先判断是否要进行树化，然后需要进行总节点数量的统计，但如果仅仅使用一个变量来表示节点总数，那么多线程修改这个值也会出现严重的阻塞问题，这里对节点总数的计算采用了Fork/Join框架



#### putval方法

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    
    //ConcurrentHashMap的key与value都不允许为null
    if (key == null || value == null) throw new NullPointerException();
    
    //计算hash值
    int hash = spread(key.hashCode());
    
    //用于记录插入节点的长度，以便转化红黑树
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        
        //要初始化则进行初始化
        if (tab == null || (n = tab.length) == 0)
            ...
            
        //hash值对应下标为null就直接插入值
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            ...
                break;                   
        }
        //发现正在扩容则帮助扩容
        else if ((fh = f.hash) == MOVED)
            ...
            
        //没有以上情况则代表此节点是有值的，则进行链表（或红黑树）比较与插入
        else {
           ...
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }

	//之前没有返回，则代表元素数增加了，进行统计
    addCount(1L, binCount);
    return null;
}
```





##### spread方法

```java
static final int spread(int h) {
    return (h ^ (h >>> 16)) & HASH_BITS;
}
```

- 与hashmap一样，将key的hashcode与高16位做异或运算，使散列更均匀；但还添加了另外一个操作：&HASH_BITS；HASH_BITS是int类型的整数最大值，2进制除了符号位为零，其余全为1，与它做与运算会使符号位为0，剩下不变；其目的是**使得hash值为正数**，因为负数hash值有特殊意义，文章开头有提出



