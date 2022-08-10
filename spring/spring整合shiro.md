#### Spring整合shiro



##### 导入依赖

```xml
<!-- shiro 包-->
	<dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-core</artifactId>
        <version>1.2.2</version>
    </dependency>

    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-web</artifactId>
        <version>1.2.2</version>
    </dependency>

    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-ehcache</artifactId>
        <version>1.2.2</version>
    </dependency>

    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-quartz</artifactId>
        <version>1.2.2</version>
    </dependency>

    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-spring</artifactId>
        <version>1.2.2</version>
    </dependency>
```



##### 在web.xml中配置shiro的filter

```xml
	<filter>
	  <filter-name>shiroFilter</filter-name>
	  <filter-class>
	      org.springframework.web.filter.DelegatingFilterProxy
	   </filter-class>
	  <init-param>
	    <param-name>targetFilterLifecycle</param-name>
	    <param-value>true</param-value>
	  </init-param>
	</filter>	 
	<filter-mapping>
	  <filter-name>shiroFilter</filter-name>
	  <url-pattern>/*</url-pattern>
	</filter-mapping>
```



##### 在spring核心配置文件中整合shiro的拦截器

```xml
```

