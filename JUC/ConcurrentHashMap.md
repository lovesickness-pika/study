### ConcurrentHashMap



##### sizectl

- sizectl是一个volatile类型对象，在ConcurrentHashMap用来保存节点数，扩容线程数，特殊状态，扩容阈值；
- sizectl为负数时，高16位保存容量，低十六位-1代表扩容线程数，-1表示正在初始化，0为默认值
- sizectl为正数时，表示扩容阈值



#### 构造方法

- ConcurrentHashMap采用懒加载的方式构造对象，在第一次插入值时才开辟空间



##### ConcurrentHashMap怎么保证线程安全

- 在1.7版本中，CHM使用分段锁来保证线程安全，在1.8版本中，对锁的粒度进行了细化，使用Synchronized+CAS保证线程的安全性，维护了一个sizeCtl的全局volatile属性表示多种状态，并在多个地方使用到了DCL验证，同时添加了TreeBin对象保存红黑树的root节点，这样在进行红黑树的旋转时，不会因为根节点发生了变化而导致并发问题



#### ConcurrentHashMap初始化

- ConcurrentHashMap使用initTable方法进行初始化，由于可能出现多个线程同时初始化的问题，所以使用sizeCtl=-1来表示线程正在进行初始化，所有初始化线程都使用CAS修改sizeCtl的值，修改成功的先进行一次DCL验证，然后初始化数组；其它线程会调yield方法让出时间片





#### ConcurrentHashMap插入值

- ConcurrentHashMap实际插入值的方式其实与hashmap差不多，不过因为要保证线程安全，所以需要先做很多准备工作：
  1. ConcurrentHashMap在计算hash值的时候多与一个常量HASH_BITS进行了按位与，这个数实际上是int类型正数的最大值，二进制中首位为0，剩余位全为一，这样保证了计算出来的hash值一定为正数，避免与ConcurrentHashMap内置的`MOVED`、`TREEBIN`、`RESERVED`这3个负数hash冲突。
  2. 根据hash值计算的下标若为null，要使用CAS插入节点
  3. 若发现该下标对应的节点正在扩容，要调用helpTransfer方法帮助扩容线程进行扩容
  4. 在对应下标插入值时，要先取得该节点的对象锁
  5. 插入值完成后，如果节点数增加了，先判断是否要进行树化，然后需要进行总节点数量的统计，但如果仅仅使用一个变量来表示节点总数，那么多线程修改这个值也会出现严重的阻塞问题，这里对节点总数的计算采用了Fork/Join框架



##### put过程

1. 调用put方法，put方法中回调putval方法
2. 在putval方法中调用spread方法取得hash值（spread方法不仅使散列更均匀，同时也避免了hashcode值为负数，负数hash值在ConcurrentHashMap中有特殊含义）
3. 进入死循环：
   1. 判断表有没有初始化，没初始化调用initTable方法初始化
   2. 初始化了则判断该下标处有没有节点，没有则cas插入节点
   3. 如果下标处已有节点，判断表是否在扩容，是则调用helpTransfer方法帮助扩容
   4. 表不在扩容则竞争该下标节点的监视器锁
   5. 竞争成功后再一次进行判断，防止该节点在竞争过程中被修改了
   6. 未被修改进入添加节点：
      1. 通过hash是否>=0判断是否为链表节点；是则循环遍历链表并比较各节点，直到替换值或在尾部添加节点；退出循环
      2. 判断头节点是否为TreeBin类对象，是则通过putTreeVal插入节点
      3. 通过binCount判断是否要进行树化；再判断此次插入值有没有插入新的节点，如果没有则直接返回旧节点而不会调用后面的addCount方法；然后退出死循环
4. 调用addCount方法记录节点数
5. 返回null



#### ConcurrentHashMap插入树节点

- ConcurrentHashMap插入树节点与hashmap基本一致，但是在ConcurrentHashMap中引入了TreeBin对象，这个对象包含了红黑树的root节点，这保证了在获取TreeBin的对象锁后，不会因为红黑树的旋转而出现并发问题



#### ConcurrentHashMap扩容

- 在addCount方法中计算节点数时，会与sizectl比较判断是否要进行扩容，然后CAS修改sizectl的值进行扩容
- 扩容分为4步：分配任务、领取任务、转移数据、结尾处理
  - 分配任务：根据CPU核心数和原数组的大小指定每次分配任务的多少，如果cpu核心数为1，那么就将整个扩容任务分配给一个线程就好了，否则每次分配（原数组长度>>>3)/cpu核心数，但每次分配至少16个节点
  - 领取任务，线程CAS修改下标













### 源码

##### initable

- 初始化ConcurrentHashMap

```java
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        //sizeCtl<0表示有其它线程正在对chm进行初始化，当前线程让出cpu
        if ((sc = sizeCtl) < 0)
            Thread.yield(); // lost initialization race; just spin
        //cas修改sizectl为-1，修改成功则由当前线程进行初始化
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
                    //如果有设置初始化容量则数组大小为初始容量，否则为默认容量
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;		
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2);		//扩容阈值为table长度的0.75倍
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

##### spread

```java
static final int spread(int h) {
    //HASH_BITS的值为0x7fffffff，与这个值做与运算以保证返回的hash值为正数（负数的hash值有其它特殊作用）
    return (h ^ (h >>> 16)) & HASH_BITS;
}
```



##### putVal

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    //chm的key与value都不能为null
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());		//取得hash值,用于定位在数组的下标位置
    int binCount = 0;		//遍历节点的数量
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        //table为空，进行初始化
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        //根据（n-1）&hash这个取余运算取得该key在table对应的位置i，如果这个位置为空，则cas将节点设置在table[i]上，然后返回（tabAt方法会保证从table[i]取值的可见性）
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        //如果当前节点正在扩容(hash值为MOVED表示正在转移元素，也就是扩容)，则当前线程辅助扩容线程进行扩容
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        //定位到的下标i处已有节点，获取table[i]的锁，进行链表的遍历
        else {
            V oldVal = null;
            synchronized (f) {
                //防止在获取锁的过程中有其它线程将table[i]处的值修改为其它值
                if (tabAt(tab, i) == f) {
                    //f的hash值>=0代表当前节点是普通的链表节点（Node），则进行链表的遍历
                    if (fh >= 0) {
                        binCount = 1;
                        //遍历链表，如果链表的某个节点的hash值相等且key为同一个对象（根据等值比较和equals比较判断是否为同一个），则用value替换旧节点，否则将节点添加到链表的尾部
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    //当前节点是树节点则进行树的遍历
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                              value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            //bincount不为零则表示进行过链表或红黑树的遍历
            if (binCount != 0) {
                //如果节点数大于等于8，转化为红黑树
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                //如果旧值不为零，表示替换了某个节点，节点数没有增加，直接返回旧值
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    //节点数增加
    addCount(1L, binCount);
    return null;
}
```

##### replaceNode	

```java
final V replaceNode(Object key, V value, Object cv) {
    int hash = spread(key.hashCode());
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0 ||
            (f = tabAt(tab, i = (n - 1) & hash)) == null)
            break;
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            boolean validated = false;
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        validated = true;
                        for (Node<K,V> e = f, pred = null;;) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                V ev = e.val;
                                if (cv == null || cv == ev ||
                                    (ev != null && cv.equals(ev))) {
                                    oldVal = ev;
                                    if (value != null)
                                        e.val = value;
                                    else if (pred != null)
                                        pred.next = e.next;
                                    else
                                        setTabAt(tab, i, e.next);
                                }
                                break;
                            }
                            pred = e;
                            if ((e = e.next) == null)
                                break;
                        }
                    }
                    else if (f instanceof TreeBin) {
                        validated = true;
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        TreeNode<K,V> r, p;
                        if ((r = t.root) != null &&
                            (p = r.findTreeNode(hash, key, null)) != null) {
                            V pv = p.val;
                            if (cv == null || cv == pv ||
                                (pv != null && cv.equals(pv))) {
                                oldVal = pv;
                                if (value != null)
                                    p.val = value;
                                else if (t.removeTreeNode(p))
                                    setTabAt(tab, i, untreeify(t.first));
                            }
                        }
                    }
                }
            }
            if (validated) {
                if (oldVal != null) {
                    if (value == null)
                        addCount(-1L, -1);
                    return oldVal;
                }
                break;
            }
        }
    }
    return null;
}
```



##### addCount

```java
/**
*添加计数，如果表太小且尚未扩容，则进行扩容。如果已经扩容，则在工作可用时帮助转移元素。在转移元素后检查是否需要再次进行扩容，因为扩容过程中添加元素可能会触发再次扩容
*@param x 增加的节点数
*@check 如果check<0，不检查是否需要扩容，如果check<=1,只在无竞争的情况下检查，竞争的情况下会调用fullAddCount方法计数后直接返回
*/
private final void addCount(long x, int check) {
    	//as用于存放节点数量，用多个CounterCell存放节点数量，这样就可以缓解多个线程同时修改baseCount的问题
        CounterCell[] as; long b, s;
    	//如果counterCells数组不为空或者 counterCells数组为空但是cas修改baseCount计数器失败
        if ((as = counterCells) != null ||
            !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
            CounterCell a; long v; int m;
            boolean uncontended = true;
            //如果出现以下情况，使用fullAddCount进行计数
            //1.如果counterCells数组为空或者长度为零
            //2.如果as[ThreadLocalRandom.getProbe() & m]为空（ThreadLocalRandom是一个线程安全的随机数生成器）
            //3.cas修改as[ThreadLocalRandom.getProbe() & m]失败（表示产生了竞争）
            if (as == null || (m = as.length - 1) < 0 ||
                (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
                !(uncontended =
                  U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
                fullAddCount(x, uncontended);
                return;
            }
            //check <= 1，表示不需要进行sumCount，减少竞争
            if (check <= 1)
                return;
            s = sumCount();
        }
    	//check >= 0,检查是否需要进行扩容
        if (check >= 0) {
            Node<K,V>[] tab, nt; int n, sc;
            //当满足以下条件时，表示需要进行扩容
            //1.节点数大于等于扩容阈值
            //2.当前节点数小于最大容量
            while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
                   (n = tab.length) < MAXIMUM_CAPACITY) {
                //rs的值为 0000 0000 0000 0000 1000 0000 000x xxxx，
                //xxxxx表示n前面零的个数，比如16前面零的个数为32-5=28，则xxxxx为1 1100
                int rs = resizeStamp(n);
                //sc小于零表示数组修改容量，判断当前线程是否需要辅助进行元素的转移
                if (sc < 0) {
                    //出现以下情况表示不需要辅助扩容
                    //1.sc >>> 16 != rs，rs是当前线程计算的值，
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                        sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                        transferIndex <= 0)
                        break;
                    if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                        transfer(tab, nt);
                }
                //cas修改sizectl的值为(rs<<16) + 2，由于rs的从低向高数第16位为1，右移16位后为符号位，所以这个数一定为负数，加2后低16位为2，表示有2-1 = 1个线程在进行扩容操作
                else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                             (rs << RESIZE_STAMP_SHIFT) + 2))
                    transfer(tab, null);
                s = sumCount();
            }
        }
    }
```

##### transfer

```java
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
    int n = tab.length, stride;
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
        stride = MIN_TRANSFER_STRIDE; // subdivide range
    if (nextTab == null) {            // initiating
        try {
            @SuppressWarnings("unchecked")
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
            nextTab = nt;
        } catch (Throwable ex) {      // try to cope with OOME
            sizeCtl = Integer.MAX_VALUE;
            return;
        }
        nextTable = nextTab;
        transferIndex = n;
    }
    int nextn = nextTab.length;
    ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
    boolean advance = true;
    boolean finishing = false; // to ensure sweep before committing nextTab
    for (int i = 0, bound = 0;;) {
        Node<K,V> f; int fh;
        while (advance) {
            int nextIndex, nextBound;
            if (--i >= bound || finishing)
                advance = false;
            else if ((nextIndex = transferIndex) <= 0) {
                i = -1;
                advance = false;
            }
            else if (U.compareAndSwapInt
                     (this, TRANSFERINDEX, nextIndex,
                      nextBound = (nextIndex > stride ?
                                   nextIndex - stride : 0))) {
                bound = nextBound;
                i = nextIndex - 1;
                advance = false;
            }
        }
        if (i < 0 || i >= n || i + n >= nextn) {
            int sc;
            if (finishing) {
                nextTable = null;
                table = nextTab;
                sizeCtl = (n << 1) - (n >>> 1);
                return;
            }
            if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                    return;
                finishing = advance = true;
                i = n; // recheck before commit
            }
        }
        else if ((f = tabAt(tab, i)) == null)
            advance = casTabAt(tab, i, null, fwd);
        else if ((fh = f.hash) == MOVED)
            advance = true; // already processed
        else {
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    Node<K,V> ln, hn;
                    if (fh >= 0) {
                        int runBit = fh & n;
                        Node<K,V> lastRun = f;
                        for (Node<K,V> p = f.next; p != null; p = p.next) {
                            int b = p.hash & n;
                            if (b != runBit) {
                                runBit = b;
                                lastRun = p;
                            }
                        }
                        if (runBit == 0) {
                            ln = lastRun;
                            hn = null;
                        }
                        else {
                            hn = lastRun;
                            ln = null;
                        }
                        for (Node<K,V> p = f; p != lastRun; p = p.next) {
                            int ph = p.hash; K pk = p.key; V pv = p.val;
                            if ((ph & n) == 0)
                                ln = new Node<K,V>(ph, pk, pv, ln);
                            else
                                hn = new Node<K,V>(ph, pk, pv, hn);
                        }
                        setTabAt(nextTab, i, ln);
                        setTabAt(nextTab, i + n, hn);
                        setTabAt(tab, i, fwd);
                        advance = true;
                    }
                    else if (f instanceof TreeBin) {
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        TreeNode<K,V> lo = null, loTail = null;
                        TreeNode<K,V> hi = null, hiTail = null;
                        int lc = 0, hc = 0;
                        for (Node<K,V> e = t.first; e != null; e = e.next) {
                            int h = e.hash;
                            TreeNode<K,V> p = new TreeNode<K,V>
                                (h, e.key, e.val, null, null);
                            if ((h & n) == 0) {
                                if ((p.prev = loTail) == null)
                                    lo = p;
                                else
                                    loTail.next = p;
                                loTail = p;
                                ++lc;
                            }
                            else {
                                if ((p.prev = hiTail) == null)
                                    hi = p;
                                else
                                    hiTail.next = p;
                                hiTail = p;
                                ++hc;
                            }
                        }
                        ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                        (hc != 0) ? new TreeBin<K,V>(lo) : t;
                        hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                        (lc != 0) ? new TreeBin<K,V>(hi) : t;
                        setTabAt(nextTab, i, ln);
                        setTabAt(nextTab, i + n, hn);
                        setTabAt(tab, i, fwd);
                        advance = true;
                    }
                }
            }
        }
    }
}
```



