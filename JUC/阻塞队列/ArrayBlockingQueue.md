### ArrayBlockingQueue



- ArrayBlockingQueue是一个有界的阻塞队列，其底层使用的是Onject[]数组与可重入锁



#### ArrayBlockingQueue怎么实现阻塞的

- 首先，ArrayBlockingQueue是使用的可重入锁保证线程安全，



#### 初始化

- 底层使用Object数组存放值，使用ReentrantLock进行阻塞，默认为非公平







#### 插入值

- offer：获取lock锁，直接插入值，队列满则插入失败返回false；可以带一个时间参数，队列满会等待，超时返回false
- add：调用offer方法，若offer返回的false则抛出异常
- put：队列满会进入可中断的无限等待



#### 移除值



