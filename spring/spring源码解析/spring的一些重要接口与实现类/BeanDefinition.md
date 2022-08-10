### BeanDefinition

- spring通过BeanDefinition来封装bean的描述信息，用于beanFactory创建bean的描述信息



#### BeanDefinition实现类的一些重要成员变量

```java
public abstract class AbstractBeanDefinition extends BeanMetadataAttributeAccessor
		implements BeanDefinition, Cloneable {
    
    @Nullable
	private volatile Object beanClass;		//这个BeanDefinition代表的类型，用于反射调用生成对应的实例

	@Nullable
	private String scope = SCOPE_DEFAULT;		//这个bean的作用域，例如singleton、prototype
    
    @Nullable
	private Boolean lazyInit;		//是否懒加载
    
    @Nullable
	private String[] dependsOn;		//这个bean所依赖的其它bean成员
    
    //这个bean是否是主bean，在spring容器中一个类型可以产生多个实例，在进行依赖注入时，如果根据类型找到了多个bean，此时会根据这些bean中是否存在一个主bean，如果存在，则直接将这个bean注入到属性
    private boolean primary = false;		
    
}
```

