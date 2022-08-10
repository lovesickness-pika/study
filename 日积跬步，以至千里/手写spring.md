

2022.07.26 周二

- 从最基础的bean容器做起，定义最基础的对象存储接口BeanFactory与实现类AbstractBeanFactory
- 将bean拆分为单例和非单例，单例bean交由SingletonBeanRegistry管理，非单例bean则应该在获取的时候创建