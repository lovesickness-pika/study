##### volatile的作用

- 保持线程可见性
  - 使其它工作内存缓存的该数据失效
- 禁止指令重排序
  - happens-before
  - as-if-serial
  - 内存屏障

