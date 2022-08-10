##### @Target

​	元注解，标记注解可作用的对象

##### @Retention

​	元注解，标记注解的生命周期

##### @Document

​	元注解，表示该注解会被javadoc工具记录，即生成的文档中会有这个注解

##### @Component

​	将普通的pojo实例化到ioc容器，在类上面添加@Component等价于在spring配置文件中添加bean：

```java
@Component
public class Person{}
```

等价于：

```xml
<bean class="..Person"
```

##### @Repository、@Service、@Controller

​	分别对应 数据访问层、业务层、控制层

​	这三个注解是对@Component的扩充、每个注解都包含@Component、在目前版本使用这三个注解的效果与使用@Component一样



#### Springboot较为重要的注解



##### @SpringBootApplication

@SpringBootApplication的重要源码：

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(...)
public @interface SpringBootApplication{...}
```

由上可知，此注解是对以上三个注解进行了集成，即标注一个注解相当于标注了三个注解，简化了程序的开发

##### @SpringBootConfiguration

 *@ SpringBootConfiguration*只是Spring标准中*@Configuration*批注的替代方法。 **两者之间的**唯一**区别是*@SpringBootConfiguration*允许自动找到配置。**在该注解中也包含了*@Configuration*

##### @Configuration

1. 表示此类是一个配置类
2. 将此类自动纳入spring容器

##### @EnableAutoConfiguration









