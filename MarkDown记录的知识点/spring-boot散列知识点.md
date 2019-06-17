<center><h1>Spring-Boot散列知识点</h1></center>

##### 01. 控制器

通常我们会将一系列的带有<font color=red>`@Controller`</font>或者<font color=red>`@RestController`</font>注解的类放在同一个包中，该包通常叫<font color=red>`controller`</font>，但是有时候也会在一些其他的包零散的类上面标注<font color=red>`@Controller`</font>或者<font color=red>`@RestController`</font>，用以识别一个控制器类，配合<font color=red>`@RequestMapping`</font>注解实现URI控制

##### 02. 自定义父项目

通常我们使用spring boot插件帮助我们创建spring boot应用，创建好的应用骨架里面默认该项目的父项目继承于某个版本的spring-boot-starter-parent项目，如下所示

```xml
<parent>
	<groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.2.RELEASE</version>
    <relativePath/>
</parent>
```

上面的pom.xml文件通过<font color=red>`<parent/>`</font>标签明确规定了该spring-boot项目的父项目，父项目的作用是规定该版本（2.1.2.RELEASE）其他依赖的版本号，这也是为什么我们在编写<font color=red>`<dependency/>`</font>依赖时不需要指定版本号的原因

假设你需要为新建的项目指定父项目，这时你只需要正常的编写<font color=red>`<parent/>`</font>标签里面的内容，同时，利用<font color=red>`<dependencyManagement/>`</font>标签指定版本号，如下所示

```xml
<dependencyManagement>
	<dependencies>
    	<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.1.2.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

##### 03. 监控启动器Actuator

添加完Actuator启动器之后，通过localhost:8080/actuator/beans可以查看到整个应用有效的bean

##### 04. 单数据源

你需要在application.properties或者application.yml文件中配置你的数据源信息，如下所示

```properties
# 针对h2内存数据库，即使你不配置下面的这些信息，spring-boot都会给你配置，k-v值和下面的完全一样
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.username=sa
spring.datasource.password=
# 即使你不写下面这一句，spring-boot也可以根据你的url加载正确的驱动
spring.datasource.driver-class-name=org.h2.driver
```

```yaml
spring:
	datasource:
		url: jdbc:h2:mem:testdb
		username: sa
		password: 
		driver-class-name: org.h2.driver
```

##### 05. 多数据源

如何在spring-boot中配置多数据源，有两种方式

1. 完全排除spring-boot关于数据源的依赖，全部手工配置
2. 与spring-boot协同配置
   1. 如果你的数据源有主次之分，在你的主数据源上面配置@Primary类型的Bean，之后spring-boot的自动配置都会围绕主数据源进行配置
   2. 如果你的数据源没有主次之分，你需要排除关于DataSourceAutoConfiguration、DataSourceTransactionManagerAutoConfiguration、JdbcTemplateAutoConfiguration这几个自动配置项，然后分别对你的数据源进行事物管理，jdbc模板的配置

下面举一个栗子

###### 05.1 排除依赖

```java
@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class,
        DataSourceTransactionManagerAutoConfiguration.class,
        JdbcTemplateAutoConfiguration.class})
```

###### 05.2 编写application.properties

配置两个数据源，分别时foo和bar，这两个数据源没有主次之分，所以我得手动配置他们的事务管理器

```properties
foo.datasource.url=jdbc:h2:mem:foo
foo.datasource.username=sa
foo.datasource.password=

bar.datasource.url=jdbc:h2:mem:bar
bar.datasource.username=sa
bar.datasource.password=
```

###### 05.3 手动配置数据源四大属性和事务管理器

在某个配置类中添加如下信息，注意@SpringBootApplication标注的类也是一个配置类

```java
# @Bean标注在函数上面，表示将该函数返回的对象注册到spring容器中，方便在应用中利用@Autowried @Resource的方式引入
@Bean
# @ConfigurationProperties用于读取application.properties文件中指定开头的所有property项
@ConfigurationProperties("foo.datasource")
public DataSourceProperties fooDataSourceProperties() {
    return new DataSourceProperties();
}

@Bean
public DataSource fooDataSource() {
    DataSourceProperties dataSourceProperties = fooDataSourceProperties();
    log.info("foo datasource: {}", dataSourceProperties.getUrl());
    return dataSourceProperties.initializeDataSourceBuilder().build();
}

@Bean
# @Resource表示依赖注入，下面的这个函数的参数fooDataSource由spring容器通过name的方式查找并注入
@Resource
public PlatformTransactionManager fooTxManager(DataSource fooDataSource) {
    return new DataSourceTransactionManager(fooDataSource);
}

@Bean
@ConfigurationProperties("bar.datasource")
public DataSourceProperties barDataSourceProperties() {
    return new DataSourceProperties();
}

@Bean
public DataSource barDataSource() {
    DataSourceProperties dataSourceProperties = barDataSourceProperties();
    log.info("bar datasource: {}", dataSourceProperties.getUrl());
    return dataSourceProperties.initializeDataSourceBuilder().build();
}

@Bean
@Resource
public PlatformTransactionManager barTxManager(DataSource barDataSource) {
    return new DataSourceTransactionManager(barDataSource);
}
```































































