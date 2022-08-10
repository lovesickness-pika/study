### spring IOC容器

Spring ioc 容器是一个IOC Service Provider，提供了两种容器类型：

BeanFactory，ApplicationContext

##### BeanFactory

- BeanFactory是基础类型的IOC容器，提供了完整的IOC服务支持，此容器默认采用延迟初始化的策略，即在容器内的某个对象第一次被访问时，才进行初始化和依赖注入操作

##### ApplicationContext

- ApplicationContext在BeanFactory的基础上构建，继承了BeanFactory的全部功能，且提供了更多的额外功能：
  - 国际化



#### 什么是spring ioc 容器，为什么把它当作spring生态的核心

- spring ioc容器实际上就是一个负责管理java bean对象的生命周期的容器，它通过反射机制去创建对象，通过依赖注入的方式给对象设置属性，通过aop的方式给对象添加功能

##### bean的生命周期

##### bean的作用域

##### @Autowired与@Resoure的区别

### AOP

- AOP是对OOP的一种补充

#### 如何配置AOP



### Spring 数据库开发

#### JDBC配置

#### spring aop和AspectJ AOP的区别与使用场景

### 设计模式

### 循环依赖

### 事务

### 生命周期

### 传播特性
