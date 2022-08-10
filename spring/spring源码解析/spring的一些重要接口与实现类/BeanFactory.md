### BeanFactory



- bean工厂，可以用于创建bean实例和获取bean实例



#### DefaultListableBeanFactory

- 这个实现类是spring容器默认使用的beanFactory实现类，原因是DefaultListableBeanFactory是BeanFactory实现类中实现功能最多的实现类，DefaultListableBeanFactory支持的功能有
  - 单例bean
  - bean别名
  - 父子容器BeanFactory
  - bean类型转化
  - bean后置处理器
  - FactoryBean
  - 自动转配
  - .

![DefaultListableBeanFactory](D:\Java\IdeaProjects\spring-source5.2.18\DefaultListableBeanFactory.png)



- 从接口BeanFactory到HierarchicalBeanFactory，再到ConfigurableBeanFactory，是一条主要的BeanFactory设计路径。
  - BeanFactory定义了BeanFactory作为容器的最基本的功能，如getBean从容器中获取bean对象
  - HierarchicalBeanFactory在BeanFactory的基础上添加了双亲容器的管理功能
  - ConfigurableBeanFactory主要定义了一些对BeanFactory的管理功能，比如通过setParentBeanFactory()设置双亲容器，通过addBeanFactoryPostProcessor()配置Bean后置处理器