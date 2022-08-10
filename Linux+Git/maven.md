### maven

一款强大的自动化构建工具

##### maven的作用

1. 管理jar	

- 增加三方jar
- 管理jar包之间的依赖，自动下载jar包所依赖的jar

2. 将项目拆分为若干个模块



##### pom.xml

项目对象模型

- 使用gav唯一在本地仓库唯一定位一个模块
  - <groupId>域名翻转.总项目名</groupId>
  - <artifactId>子项目名</artifactId>
  - <version>版本号</version>

```
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-parent</artifactId>
<version>2.5.0</version>
```



依赖的四个级别：

Compile，Test，Runtime，Provided
