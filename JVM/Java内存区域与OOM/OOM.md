### OutOfMemeoryError

- java运行时数据区除了程序计数器以外，都可能发生OOM



#### Java堆溢出

- 当堆的当前空间小于堆最大值时，堆满会进行自动扩容

##### 怎样解决堆溢出

- 先通过内存映像分析工具对Dump出来的堆转储快照进行分析，判断是发生了内存泄漏还是内存溢出
- 如果是内存泄漏，可通过工具查看泄漏对象到GC Roots的引用链，定位到这些对象创建的位置
- 如果是内存溢出则看堆空间是否有向上调整的空间；再从代码上检查是否存在某些对象生命周期过长、存储结构不合理等情况



#### 虚拟机栈和本地方法栈溢出

- 如果线程请求的栈深度大于虚拟机所允许的最大深度，将抛出StackOverflowError异常
- 如果虚拟机的栈内存允许动态扩展，当扩展栈容量无法申请到足够的内存时，将抛出OOM异常



#### 方法区和运行时常量池溢出

- String：：intern是一个本地方法，它的作用是如果字符串常量池中已经包含一个等于此String对象的字符串，则直接返回这个引用
- 