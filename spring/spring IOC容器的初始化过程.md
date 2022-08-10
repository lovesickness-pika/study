### spring 容器的初始化过程

- 使用XML配置文件初始化的过程



- 在web应用中，通常我们不会手动的去初始化一个容器，而是通过配置ContextLoaderListener来监听web容器的初始化，当web容器初始化的时候，就会调用createWebApplicationContext方法对spring容器进行初始化，并且设置web容器为spring容器的父容器，这样就可以从web容器中取得spring容器了



1. 调用ClassPathXmlApplicationContext的构造方法，super()方法并传入一个ApplicationContext对象来进行ApplicationContext的父容器的设置以及一些初始参数的设置，setConfigurations方法传入资源路径，这个方法会在进行非空判断后将传入的参数放到spring容器的configurations数组中保存
2. 调用refresh方法进行BeanFactory的创建以及初始化，refresh方法使用了模板方法设计模式，共有13个方法



#### refresh方法

1. prepareRefresh方法进行了刷新前的预处理，比如initPropertySource初始化一些属性设置，可以作为扩展进行一些属性设置；还有getEnvironment().validateRequiredProperties()对属性进行合法性检验，并且使用earlyApplicationEvents来保存一些早期的事件
2. 使用obtainFreshBeanFactory刷新BeanFactory，就是销毁原有的BeanFactory，并创建一个新的DefaultListableBeanFactory，并设置一些初始化参数，比如允许BeanDefinition的覆盖（allBeanDefinitionOverriding），允许循环依赖（allowCircularReferences）
3. prepareBeanFactory(beanFactory)方法为BeanFactory进行准备工作，就是设置很多的参数
   1. 设置类加载器
   2. 设置表达式解析器
   3. 设置部分后置处理器
   4. 设置忽略的自动装配接口
   5. 注册可以解析的自动装配
   6. 注册一些组件比如environment
4. postProcessBeanFactory(beanfactory)方法进行准备工作的后置处理工作，默认为空实现，可以作为子类的扩展
5. invokeBeanFactoryPostProcessors执行beanfactory的后置处理器
   1. 两个接口：BeanFactoryPostProcessor、BeanDefinitionRegistryPostProcessor
   2. 执行BeanFactoryPostProcessor的方法；
              先执行BeanDefinitionRegistryPostProcessor
              1）、获取所有的BeanDefinitionRegistryPostProcessor；
              2）、看先执行实现了PriorityOrdered优先级接口的BeanDefinitionRegistryPostProcessor、
                  postProcessor.postProcessBeanDefinitionRegistry(registry)
              3）、在执行实现了Ordered顺序接口的BeanDefinitionRegistryPostProcessor；
                  postProcessor.postProcessBeanDefinitionRegistry(registry)
              4）、最后执行没有实现任何优先级或者是顺序接口的BeanDefinitionRegistryPostProcessors；
                  postProcessor.postProcessBeanDefinitionRegistry(registry)
   3. 再执行BeanFactoryPostProcessor的方法
          1）、获取所有的BeanFactoryPostProcessor
          2）、看先执行实现了PriorityOrdered优先级接口的BeanFactoryPostProcessor、
              postProcessor.postProcessBeanFactory()
          3）、在执行实现了Ordered顺序接口的BeanFactoryPostProcessor；
              postProcessor.postProcessBeanFactory()
          4）、最后执行没有实现任何优先级或者是顺序接口的BeanFactoryPostProcessor；
              postProcessor.postProcessBeanFactory()
6. registerBeanPostProcessors注册Bean的后置处理器，不同类型的BeanPostProcessor的执行时机是不同的
7. initMessageSource();初始化MessageSource组件，做国际化；消息绑定，消息解析用
8. initApplicationEventMulticaster();初始化事件派发器
9. onFresh方法默认空实现，可作为扩展
10. **finishBeanFactoryInitialization**，初始化所有非懒加载的bean对象，并触发BeanPostProcessor
    1. 使用preInstantiateSingletons方法初始化单例bean