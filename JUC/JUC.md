## JUC

### 线程

##### 线程安全类



#### 线程状态

![未命名文件](D:\Java\未命名文件.jpg)



#### 多线程中常用的方法

##### Thread.interrupt, Thread.interrupt, Thread.isInterrupt

- t.interrupt()并不会中断正在执行的线程，只是将线程的中断标志位设置为true，当线程处于调用了wait()，sleep()或join()时被中断，会抛出InterruptedException，然后中断标志位恢复为false
- private native boolean isInterrupted(boolean ClearInterrupted);这个方法是取得线程中断状态的方法，其中传入的参数代表是否要将线程的中断状态重置
- Thread.interrupted()和t.isInterrupt都会调用isInterrupted(boolean ClearInterrupted)这个本地方法，返回的都是当前线程的中断状态，不同的是前者传入的参数为true，后者传入的参数为false，即前者无论线程中断状态如何，都重置为false，后者不影响线程的中断状态

- 可以用t.interrupt()解决死锁问题吗？不可以，因为线程在获Synchronized锁和调用Lock.lock的过程中是不能被中断的

##### Object.wait

- Object.wait()方法让当前线程放弃锁资源（对象的Monitor对象），进入WAITING状态，所以只能在synchronized代码块中使用
- 如果当前对象锁处于偏向锁状态，调用wait会进行偏向锁撤销

##### Object.notify，Object.notifyAll

- Object.notify()，随机唤醒一个在该对象上等待的线程
- Object.notifyAll()，唤醒所有在该对象上等待的线程

##### LockSupport.park & LockSupport.unpark

- LockSupport类通过维护与线程相关的Parker实例中的**_counter**属性（称之为许可）实现了更加灵活，更具有解耦性的线程中断机制
- park()方法判断许可是否可用，可用直接返回，否则进入等待
- unpark()方法发放许可（许可只有一个），然后判断当前线程是否

##### Thread.sleep

- 此方法的目的在于使线程在指定时间内不参与cpu的抢占，不会放弃锁资源

##### Thread.join()

- 使当前线程等待调用join方法的线程结束后再继续运行，即如果在A线程中调用B.join()，则A会进入WAITING状态，直到B执行完毕(线程结束时会调用自身的notifyAll方法		实现原理：

```java
while(isAlive()){
    wait(0);	//无限期等待，直到等待的线程结束时调用notifyAll
}
```





##### 阻塞态与等待态

- 阻塞态是线程在进入synchronized修饰的代码块时由于未取得对象锁进入的状态，而等待态是线程显示的调用了ThreadSupport.lock方法或在Synchronized代码块中显示的调用了wait方法进入的状态；前者会进入阻塞队列，但并不会放弃当前持有的锁资源，会放弃cpu资源；后者会放弃锁资源和cpu资源

##### wait，notify，park，unpark方法的区别

##### 虚假唤醒问题

- 当一个条件满足时，很多线程都被唤醒了，但是只有其中部分是有用的唤醒，其它的唤醒都是无用的，甚至会导致程序的异常

解决方法：使用while进行判断条件的检测而不是if

### HashMap

##### 版本JDK8

- hashmap 在1.8中引入了红黑树来加快访问速度
- hashmap使用的懒加载机制，在第一次访问时才会进行初始化，否则只是记录了数组大小







### ConcurrentHashMap

##### 版本JDK8

#### 一些属性

```java
static final int HASH_BITS = 0x7fffffff; //这个值换算为而进制其实就是最大的正数，在hash值计算中用来防止hash值为负数


//0为默认值
//-1表示正在初始化
//-n表示有n-1个线程在扩容
//大于零表示扩容阈值
private transient volatile int sizeCtl;	

//标志数组的下标正在扩容
static final int MOVED     = -1;
//标志数组的下标存放的是树结构
static final int TREEBIN   = -2;
```





##### initTable过程







