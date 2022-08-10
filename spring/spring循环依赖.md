### spring循环依赖



##### 什么是spring的循环依赖

```java
@Component
class A{
    @Autowired
    B b;
}

@Component
class B{
    @Autowired
    A a;
}
```

- 当由spring管理的两个单例bean互相依赖时，在spring容器初始化的时候就会产生循环依赖问题



##### spring容器中非懒加载的单例bean生命周期的大概流程

1. 由BeanDefintionReader实现类对资源进行解析，并封装为BeanDefinition对象，然后将BeanDefinition注册到spring容器中
2. 在doCreateBean中调用createBeanInstance()方法创建实例
3. 将当前bean实例和bean定义封装为ObjectFactory放入singletonFactories（三级缓存）中
4. 调用populateBean方法进行依赖注入
5. 调用initializeBean方法对
6. 