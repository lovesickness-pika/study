### springboot使用流程

springboot版本：

#### 创建springboot项目

##### IDEA2020.2.3：

1. 新建项目，选择Spring Initializr
2. 配置项目相关：组名、java版本、打包类型等等
3. 选择场景（maven会导入相关依赖）



##### eclipse&sts:

1. 新建项目，选择 Spring Starter Project
2. 配置项目相关：组名、java版本、打包类型等等
3. 选择场景（maven会导入相关依赖）



#### 项目开发

在开发具体功能时，应遵循开发顺序：*持久层>业务层>控制器>前端页面*

springboot采用约定的方式省去了很多配置，因此很多地方要按照约定的方式开发，例如配置文件的位置，静态资源的位置等等，如果需要手动修改，应在springboot配置文件中做出修改

##### 数据库

- 规划实体类索引关系，在数据库中创建相应表
- 在springboot配置文件中添加数据库连接信息
- 测试数据库的连接

##### springboot启动类

- 即带main方法的类，项目的入口
- 该类应该放在项目的根目录或者其他包的父包中，以**保证其它组件位于此包的子包**中
- 给启动类添加**@SpringBootApplication**注解

##### 根据ssm流程开发项目

1. 数据实体类：domain（entity），并给每个实体类添加**@Component**注解
2. 数据接口访问层：repostory（mapper）， 并添加**@Repostory**注解
3. 业务接口访问层：service，并添加**@Service**注解
4. 控制器层：Controller, 并添加**@Controller**注解

以上注解用于将对应类添加到ioc容器中，前提是springboot启动类在这些组件的父包中，否则应该在启动类中使用@**Scan将整个包纳入扫描范围

##### 静态资源文件

- 导入静态资源文件（jquery.js）等，访问webjars网站，将对应静态资源的依赖加入pom文件，maven会将对应静态资源的jar包导入项目，访问路径：从/webjars/开始

- 自己的静态资源，根据约定，放在这些路径下：

  ```
  "classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/"
  ```

##### 动态资源

- springboot默认不支持jsp
- springboot推荐使用 模板引擎 thymeleaf

##### 配置文件

- springboot默认读取application.properties/application.yml文件
- 若同时存在以上两种配置文件且属性冲突，优先读取application.properties

- 根据约定，放在这些路径下(配置冲突则优先级从高到低，不冲突则互补)：

```bash
"file:项目根目录/config/","file:项目根目录/",
"classpath:项目类路径/config/","classpath项目类路径/"
```

- 项目外添加配置文件，配置优先级：命令行参数>外部配置文件>内部配置文件，这些只是部分方法，了解更多可以查看官方文档
  - run项目时添加参数arguments指定外部配置文件：**--spring.config.location=指定url**
  - run项目时添加参数指定某个参数： **--prefix.属性名=value**

##### 常用配置

```properties
#端口号
server.port=8888
#编码
server.servlet.encoding.charset=GBK
#访问时的项目根路径
server.servlet.context-path=test
```

##### 日志

- 日志框架：UCL，JUL，jboss-logging，logback，log4j，log4j2，slf4j...

- springboot默认配置了slf4j，lockback，log4j这三种日志框架，可直接使用，需要使用其它日志框架可以查看官方文档进行使用

- 日志级别（优先级由低到高）：TRACE, DEBUG, INFO, WARN, ERROR, FATAL, OFF

- springboot默认打印INFO及以后级别的日志

- 自定义打印日志级别：logging.level.启动类所在包=级别

- 日志打印方法

  - 通过日志类Logger直接打印
  - 配置日志输出到指定文件：logging.file=url
  - 配置日志输出到指定文件夹：logging.path=url/

- 指定日志输出格式(console表示设置控制台输出，file表示设置文件输出)：

  - ```properties
    logging.pattern.console=%d{yyyy-MM-dd}[%thread] %-5lever %logger{50} - %msg%n
    logging.pattern.file=...
    ```

    - %d：日期
    - %thread：线程名
    - %-5level：日志级别，显示5个字符宽度
    - %logger{50}：这条日志的长度
    - %msg：日志显示的消息
    - %n：回车

- 可以在jar包中看到日志的默认配置

#### 常见错误

- ```bash
  异常：创建项目时连接spring.io超时
  原因：网速不行
  解决方法：访问start.spring.io线上打包springboot项目
  ```

- ```bash
  异常：java.lang.IllegalStateException: Found multiple @SpringBootConfiguration annotated classes
  原因：出现了多个 @SpringBootApplication 注解
  解决方法：注释掉多余的注解
  ```

- 

### springboot源码

#### @SpringBootApplication

作用：

- 

@SpringBootApplication的重要源码：

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(...)
public @interface SpringBootApplication{...}
```

由上可知，此注解是对以上三个注解进行了集成，即标注一个注解相当于标注了三个注解，简化了程序的开发

#### @SpringBootConfiguration

##### 作用：

- 标注此类为配置类
- 添加此类中用**@bean**标注的方法返回实例添加到spring容器，且实例名为方法名

##### 重要源码：

```java
@Configuration
public @interface SpringBootApplication{...}
```

*@ SpringBootConfiguration*只是Spring标准中*@Configuration*批注的替代方法。 **两者之间的**唯一**区别是*@SpringBootConfiguration*允许自动找到配置。**在该注解中也包含了*@Configuration*

#### @EnableAutoConfiguration

##### 作用：

- 使springboot根据约定自动配置
- 

##### 重要源码：

```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoconfoguration{...}
```

##### @AutoConfigurationPackage

作用：

- 将启动类所在包及其子包纳入spring容器，相当于spring中为启动类的所在包及其子包的每一个包都手写scan扫描器

重要源码：

```java
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage {...}
```

- 在AutoConfigurationPackages.Registrar的registerBeanDefinitions方法中将标注类（启动类）所在包及其子包添加到spring容器中

##### Import(AutoConfigurationImportSelector.class)

作用：

- 引入三方jar包，将三方jar引入项目

如何引入三方jar？

调用栈：

```
AutoConfigurationImportSelector.selectImports
AutoConfigurationImportSelector.getAutoConfigurationEntry
AutoConfigurationImportSelector.getCandidateConfigurations
SpringFactoriesLoader.loadFactoryNames
SpringFactoriesLoader.loadSpringFactories
classLoader.getResources(FACTORIES_RESOURCE_LOCATION)
```

其中

```
FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories"
```

可知，springboot在启动时，会根据META-INF/spring.factories找到相应的三方依赖，并将这些依赖引入本项目，将一个个XxxAutoConfiguration加载

##### 自动装配

研究org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration
		通过观察该源码 发现：

- @Configuration：标识此类是一个配置类  、将此类纳入spring容器
- @EnableConfigurationProperties(HttpEncodingProperties.class)： 通HttpEncodingProperties将编码设置为了UTF_8 (即自动装配为UTF_8）


		@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true)
		当属性满足要求时，此条件成立：要求 如果没有配置spring.http.encoding.enabled=xxx, 则成立。

- springboot**不会**将所有*META-INF/spring.factories*中标注了自动装配的配置类都进行自动装配，而是通过每个配置类上的**@CondictionalOnXxx**条件满足时才进行自动装配

```properties
server.servlet.encoding.charset=GBK
```

- 修改配置：在springboot的配置文件中，根据配置类提供的**prefix.属性名=value**进行修改

- 查看开启了与禁用了哪些自动装配
  - 配置配置文件：**dubug=true**
  - Positive matches列表 表示 spring boot自动开启的装配
  - Negative matches列表 表示spring boot在此时 并没有启用的自动装配
