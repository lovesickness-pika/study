前置概念：

- 弱引用

ThreadLocal的作用：

提供线程内的局部变量，不同线程之间不会相互干扰，这种变量在线程生命周期内起作用，减少同一个线程内多个函数或组件之间一些公共变量传递的难度

应用场景：

- spring的@TransAction

- mybatis关于分页的处理	

ThreadLocal实现的方式：

每个线程内都有维护一个这样的map集合：

```java
ThreadLocal.ThreadLocalMap threadLocals = null;
```

在这个map集合内：维护了一个Entry数组，这个entry类对象继承了WeakReference类，且这个entry对象只有一个成员变量为要存储的对象，且在实例entry时将虚引用指向ThreadLocal

为什么要使用虚引用？

防止内存泄漏，当外部对于该ThreadLocal对象的强引用消失，此时如果该线程的ThreadLocalMap对象还存在对ThreadLocal的强引用，会导致该ThreadLocal对象无法回收，直到线程销毁

ThreadLocalMap的内存泄漏完全解决了吗？

- 并没有，由于ThreadLocalMap内的entry对于ThreadLocal是虚引用，在GC时会无视引用回收ThreadLocalMap对象，这导致了此entry对象不会再被访问（可以理解为key为null）

- 补救措施：ThreadLocalMap的set方法中会在遍历entry通过弱引用指向的对象，在找到该指向的对象为当前的ThreadLocal对象时，会通过replaceStaleEntry方法释放所有指向对象为null的entry的value值指向的对象
- 以上补救措施一定程度的解决value值内存泄漏的问题，但当此线程长时间不往这个map集合中插入值时，内存泄漏依旧无法解决，所以编程人员应该养成良好的习惯，使用remove消除此内存泄漏



ThreadLocalMap的构造与扩容：

- ThreadLocal类有一个用于寻址的值叫threadLocalHashCode，这个值是在ThreadLocal的一个静态原子类变量nextHashCode的基础上，每新建一个ThreadLocal对象，成员变量用32位整数的斐波那契乘数决定下一个值，以保持threadLocalHashCode的散列均匀

- ThreadLocal中使用的寻址方法为开发寻址法，从散列到的值开始向下遍历entry弱引用的值，如果为null，



##### expungeStaleEntry（int staleSlot）

1. 清除tab[staleSlot]
2. 从staleSlot下标开始，向后面遍历，直到entry为null
   1. 弱引用指向的值为null，清理该entry
   2. 弱引用的值不为null，通过*threadLocalHashCode & (len - 1)*计算是否应该是这个位置
      1. 是这个位置不做处理
      2. 不应该为这个下标，将这个entry向后找一个entry为null的放下，然后清理当前entry
3. 返回找到的第一个entry为null的下标，被清空的不算

##### cleanSomeSlots(int i, int n)

- 循环的去寻找脏Entry，即key=null的Entry，然后进行删除。
- n >>>= 1 说明要循环log2N次。在没有发现脏Entry时，会一直往后找下个位置的entry是否是脏的，如果是的话，就会使 n = 数组的长度。然后继续循环log2新N 次。

