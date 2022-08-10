##### synchronized的特性

- 有序性
  - as-if-serial
  - happens-before
- 可见性
  - 内存强制刷新
- 原子性
  - 单一线程持有
- 可重入性
  - 计数器

##### 对象内存布局

- markword （标记字段）8字节
  - 根据对象的状态复用此空间，运行期间mark word内的数据随锁标志位的变化而变化
  - 锁信息
  - gc标记信息
  - hashcode
- klass pointer（类型指针），指向类元数据的指针
- data （实例数据）
- padding （对齐填充），对象起始地址为8字节的整数倍

##### 锁膨胀

- 无锁

  - 锁标志位：001
  - markword 存放对象的Identity hash code（懒加载，通过未被覆写java.lang.Object.hashCode() 或者java.lang.System.identityHashCode(Object) 计算）

- 偏向锁
  - 锁标志位：101
  - Mark word记录线程ID
  - 1.6以后默认开启，参数：-xx:-UseBiasedLocking=true
  
- 轻量级锁
  - 锁标志位：00
  - 基于CAS自旋
  
- 重量级锁
  - 锁标志位：10
  - Mark Word指向该对象的monitor对象
  
- 锁膨胀的过程：

  - 偏向锁->轻量级锁->重量级锁，锁膨胀只能顺序，无法逆序

    ![锁膨胀](C:\Users\皮卡丘\Pictures\Saved Pictures\锁膨胀.jpg)

##### 底层实现原理

- **monitor**监视器对象

##### 指令重排序

- 指令重排序的意义：使指令更加符合CPU的执行特性，最大限度的发挥机器的性能，提高程序的执行效率。

##### as-if-serial

- 遵循 as-if-serial，编译器和处理器不会对存在数据依赖关系的操作重排序，因为这种重排序会改变执行结果，即单线程执行结果不会改变

##### happens-before

指定一个操作发生在另一个操作之前，保证多线程中一个线程的操作对其它线程可见，防止多线程中发生的指令重排序

- 定义
  - 如果一个操作happens-before(之前发生)另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。
  - 两个操作之间存在happens-before关系，并不意味着一定要按照happens-before原则制定的顺序来执行。如果重排序之后的执行结果与按照happens-before关系来执行的结果一致，那么这种重排序并不非法。

- 具体规则
  - 程序顺序规则：一个线程中的每个操作，happens-before于该线程中的任意后续操作。
  - 监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的加锁。
  - volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。
    传递性：如果A happens-before B，且B happens-before C，那么A happens-before C。
  - start()规则：如果线程A执行操作ThreadB.start()（启动线程B），那么A线程的ThreadB.start()操作happens-before于线程B中的任意操作。
  - join()规则：如果线程A执行操作ThreadB.join()并成功返回，那么线程B中的任意操作happens-before于线程A从ThreadB.join()操作成功返回。
  - 程序中断规则：对线程interrupted()方法的调用先行于被中断线程的代码检测到中断时间的发生。
    对象finalize规则：一个对象的初始化完成（构造函数执行结束）先行于发生它的finalize()方法的开始。