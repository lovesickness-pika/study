#### ReentrantReadWriteLock

- 读写锁，对资源访问实施共享锁和互斥锁
- 关于锁的获取：写写互斥，写读互斥，读写互斥，读读共享；即，在A线程获取写锁后，B线程不可再获取写锁与读锁；A线程获取读锁后，B线程不可再获取写锁，但可以获取读锁；
- 支持锁降级，不支持锁升级；即，A线程获取写锁后，可以先获取读锁，再释放写锁；但是，A获取读锁后，想要获取写锁，必须先释放读锁；
- 为什么支持锁降级而不支持锁升级呢？主要是为了保证数据的可见性，如果当前线程不获取读锁而直接释放写锁，假设此刻另一个线程获取了写锁并修改了数据，那么当前线程是无法感知线程T对数据的更新，并且避免了先释放写锁之后再去获取读锁带来的不必要的锁竞争；假如只有一个线程t1，当t1已经获取`读锁`之后，再次获取`写锁`，因为`写锁`在加锁时判断到**当前锁已经被加过读锁**，`读写互斥`，所以`写锁`会等待`读锁`释放之后再加锁。但是因为`读锁`是被**当前线程**持有的，所以这个等待会无限的等待下去，最后就成了死锁。

##### Sync

- 应AQS框架推荐，ReentrantReadWriteLock通过内部类Sync实现了AQS，在sync中，因为AQS只维护了一个int类型的成员state来表示锁的状态，但是读写锁确有两种锁，那么就需要通过这一个成员来实现对两种锁状态的维护：读写锁中使用state的高16位代表读锁被获取的次数，用state的低16位代表写锁的获取次数，以下是sync的具体实现：

```
abstract static class Sync extends AbstractQueuedSynchronizer {
    static final int SHARED_SHIFT   = 16;		//代表写锁使用的位数
    static final int SHARED_UNIT    = (1 << SHARED_SHIFT);		//这个常量用于对写锁持有数加一
    static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;		//表示可申请读锁的最大数量
    //这个常量用于计算写锁数，它的前32-SHARED_SHIFT位为0，后SHARED_SHIFT位为1，用state进行与运算可以得出写锁的持有数
    static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;
    
    //由于在读写锁中，可能有多个线程同时持有多个读锁，那么不能像ReentrantLock一样直接以state记录线程持有锁数，而是通过将这个数据记录在各个线程的线程本地变量中，key即为readHolds
    //记录线程获取的读锁数
    private transient ThreadLocalHoldCounter readHolds;

	//避免多次从线程本地变量取值而对值进行缓存
    private transient HoldCounter cachedHoldCounter;

    private transient Thread firstReader = null;
    private transient int firstReaderHoldCount;
```



- tryAcquireShared
  - 虽然此方法名为tryAcquireShared，但是由于在此方法的最后一句显示的调用了fullTryAcquireShared，所以只要当前线程满足获取读锁的条件（其它线程未持有写锁，不满足readerShouldBlock，读锁数未达上限）,通过此方法一定能取得读锁

```
protected final int tryAcquireShared(int unused) {

    Thread current = Thread.currentThread();		//取得当前线程
    int c = getState();		//获取锁状态
    if (exclusiveCount(c) != 0 &&		//判断是否已经被持有写锁
        getExclusiveOwnerThread() != current)		//此读写锁已经被持有写锁，判断持有者是否是当前线程；如果没有这个判断，可能会导致死锁：当前线程持有写锁，但是在获取读锁时进入了阻塞，写锁未被释放
        return -1;		//其他线程持有写锁，或者读锁失败，返回-1
    int r = sharedCount(c);				//取得读锁被持有数
    if (!readerShouldBlock() &&		//防止写锁长时间阻塞
        r < MAX_COUNT &&		//判断锁数是否超过最大限额
        compareAndSetState(c, c + SHARED_UNIT)) {		//尝试以CAS方式添加一个读锁
    	//读锁添加成功
    	//判断读锁被持有数是否为零，是则用firstReader记录第一个读锁获取者为当前线程，用firstReaderHoldCount记录持有读锁数
        if (r == 0) {		
            firstReader = current;
            firstReaderHoldCount = 1;
        //当前线程是第一个读锁的获取者，第一个读锁获取者的读锁获取数加一
        } else if (firstReader == current) {
            firstReaderHoldCount++;
        } else {
            HoldCounter rh = cachedHoldCounter;
            //如果最后一个获取读锁的线程为null或者当前线程不是这个线程，从当前线程中取得readHolds并赋给cachedHoldCounter
            if (rh == null || rh.tid != getThreadId(current))
                cachedHoldCounter = rh = readHolds.get();
            //如果最后一个获取读锁的线程为当前线程且读锁获取数为0，将当前线程获取读锁数存到线程本地变量中，方便在整个请求上下文（请求锁、释放锁等过程中）使用持有锁次数信息。
            else if (rh.count == 0)
                readHolds.set(rh);
            //当前线程获取读锁数加一
            rh.count++;
        }
        return 1;
    }
    return fullTryAcquireShared(current);
}
```

- fullTryAcquireShared

```
final int fullTryAcquireShared(Thread current) {
    HoldCounter rh = null;
    for (;;) {
        int c = getState();
        if (exclusiveCount(c) != 0) {
            if (getExclusiveOwnerThread() != current)
                return -1;
        } else if (readerShouldBlock()) {
            // Make sure we're not acquiring read lock reentrantly
            if (firstReader == current) {
                // assert firstReaderHoldCount > 0;
            } else {
                if (rh == null) {
                    rh = cachedHoldCounter;
                    if (rh == null || rh.tid != getThreadId(current)) {
                        rh = readHolds.get();
                        if (rh.count == 0)
                            readHolds.remove();
                    }
                }
                if (rh.count == 0)
                    return -1;
            }
        }
        if (sharedCount(c) == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        if (compareAndSetState(c, c + SHARED_UNIT)) {
            if (sharedCount(c) == 0) {
                firstReader = current;
                firstReaderHoldCount = 1;
            } else if (firstReader == current) {
                firstReaderHoldCount++;
            } else {
                if (rh == null)
                    rh = cachedHoldCounter;
                if (rh == null || rh.tid != getThreadId(current))
                    rh = readHolds.get();
                else if (rh.count == 0)
                    readHolds.set(rh);
                rh.count++;
                cachedHoldCounter = rh; // cache for release
            }
            return 1;
        }
    }
}
```

- readerShouldBlock

  - readerShouldBolck在公平锁中的实现

  在公平锁中，readerShouldBlock实际上是调用了AQS提供的hasQueuedPredecessors，通过判断阻塞队列中头节点之后是否还有其他线程的节点来实现公平竞争

  ```
  public final boolean hasQueuedPredecessors() {
      //当等待队列中还有节点，且第一个节点不为空且不是当前节点时，返回true继续尝试获取锁，否则返回false
      Node t = tail; // Read fields in reverse initialization order
      Node h = head;
      Node s;
      return h != t &&
          ((s = h.next) == null || s.thread != Thread.currentThread());
  }
  ```

  - readerShouldBlock在非公平锁中的实现

  在非公平锁中，readerShouldBlock显示的调用了AQS提供的apparentlyFirstQueuedIsExclusive方法，通过判断阻塞队列中头节点的下一个节点是否是请求独占式访问的节点（即写节点）来选择是否要继续获取锁，以防止写锁的长时间阻塞：由于读写互斥，线程想要获取写锁必须要等待其它读锁全部释放，但因为读锁又是共享的，如果一直有线程通过非公平的方式获取到读锁，那么写锁的获取将永无出头之日

  ```
  //判断在阻塞队列中，第一个阻塞节点是否为一个有效的独占式请求节点
  final boolean apparentlyFirstQueuedIsExclusive() {
      Node h, s;
      return (h = head) != null &&		//判断头节点是否为空
          (s = h.next)  != null &&		//判断阻塞队列的第一个节点是否为空
          !s.isShared()         &&		//判断该节点是否为独占式请求
          s.thread != null;		//判断该节点的线程是否为空
  }
  ```

