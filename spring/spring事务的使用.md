### spring事务的使用



#### 配置spring事务

- spring配置文件中关于事务配置的有三个组成部分，分别是DataSource、TransactionManager和代理机制这三部分，无论哪种配置方式，一般变化的只是代理机制。



#### spring事务+mybatis

- DataSource	数据源

```xml
<!--配置德鲁伊数据库连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <!-- 连接 -->
        <property name="driverClassName" value="${jdbc.driverClass}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.user}"/>
        <property name="password" value="${jdbc.password}"/>

        <!-- 私有属性 -->
    </bean>
```

- TransactionManager	事务管理器

```xml
<!--配置MyBatis的平台事务管理器，声明式事务-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
```





#### spring事务+redis

