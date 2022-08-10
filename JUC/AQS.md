#### AQS

- 全称AbstractQueuedSynchronizer，队列同步器

##### AQS原理

- 维护一个标记位state，标记锁的状态，为0表示锁为被线程占有，>0表示锁被重入的次数
- 和一个同步队列CHL（等待队列），线程获取锁失败后会被插入到队尾，然后阻塞，等待前置节点的唤醒

##### AQS重要的成员变量

```java
private volatile int state;		//状态
private transient volatile Node head;	//队首
private transient volatile Node tail;	//队尾
```

AQS依赖对以上三个成员的维护，实现锁

##### AQS重要的方法

- 关于状态位：

```java
private fianl int getState(){...}
private fianl void setState(int newState){...}
//CAS操作修改标志位
private fianl boolean compareAndSetState(int expect, int update){...}
```

- 关于CHL队列

```java
//除了初始化外，唯一对队首修改的方法
private void setHead(Node node) {...}
//将节点插入等待队列队尾，失败则自旋，必要时进行初始化
private NOde enq(fianl Node node){...}
```

- 关于节点

```java
//为当前线程以给定模式创建节点，并插入到队尾
private Node addWaiter(Node mode) {
    ...
    enq(node);
    ...
}
//唤醒节点的后置节点
private void unparkSuccessor(Node node) {
     ...
     //最终通过LockSupport.unpark()方法唤醒
     if(s != null){
         LockSupport.unpark(s.thread);
     }
 }



public final void acquire(int arg) {
   if (!tryAcquire(arg) &&
         acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
      selfInterrupt();
}
```

- 在阻塞线程前，需要使用shouldParkAfterFailedAcquire方法对当前线程的前一个节点的等待状态进行更行，以防死锁

```
//此方法的返回值决定了parkAndCheckInterrupt会不会被调用，即线程会不会进入阻塞状态
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;
    //前一个节点的等待状态为SIGNAL,返回true
    if (ws == Node.SIGNAL)
        return true;
    //前一个节点等待状态为CANCELLED，向前遍历，直到找到不为CANCELLED状态的节点
    if (ws > 0) {
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
    //1、前一个节点的等待状态为0，说明是新节点，使用CAS操作更新节点状态为SIGNAL
    //2、前一个节点的等待状态为CONDITION或PROPAGATE...
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    //再次尝试去获取锁
    return false;
}
```

- 阻塞线程，在AQS中，真正对线程进行阻塞的方法是parkAndCheckInterrupt方法

```
//返回是否发生过中断
private final boolean parkAndCheckInterrupt() {
	//阻塞当前线程
    LockSupport.park(this);
    //线程被释放，通过interrupt方法判断是上一个节点通过unpark方法释放的还是线程被中断
    return Thread.interrupted();
}
```

**为什么查找队列中未被取消的节点需要从尾部开始？**

这个问题有两个原因可以解释，分别是 Node 入队和清理取消状态的节点

1. 先从 addWaiter 入队时说起，compareAndSetTail(pred, node)、pred.next = node 并非原子操作，如果在执行 pred.next = node 前进行 unparkSuccessor，就没有办法通过 next 指针向后遍历，所以才会从后向前找寻非取消的节点
2. cancelAcquire 方法也有导致使用 head 无法遍历全部 Node 的因素，因为先断开的是 next 指针，prev 指针并未断开

