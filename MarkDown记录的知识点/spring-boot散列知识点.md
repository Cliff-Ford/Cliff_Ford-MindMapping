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
   2. 如果你的数据源没有主次之分，你需要排除关于DataSourceAutoConfiguration、DataSourceTransactionManagerAutoConfiguration、JdbcTemplateAutoConfiguration这几个自动配置项，然后分别对你的数据源进行事务管理，jdbc模板的配置

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

##### 06. HiKariCp连接池

目前最快的连接池，是spring-boot 2.x版本默认的数据库连接池，[官网传送门](<https://github.com/brettwooldridge/HikariCP>)

* 字节码级别优化（很多方法通过JavaAssist生成）
* 大量小改进
  * 用FastStatementList代替ArrayList
  * 无锁集合ConcurrentBag
  * 代理类的优化（比如，用invokestatic代替了invokevirtual）

```java
/**
* Hikari DataSource configuration
*/
@ConditionalOnClass(HikariDataSource.class)
@ConditionalOnMissingBean(DataSource.class)
@ConditionalOnProperty(name = "spring.datasource.type", havingValue="com.zaxxer.hikari.HikariDataSource")
static class Hikari{
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.hikari")
    public HikariDataSource dataSource(DataSourceProperties properties){
        HikariDataSource dataSource = createDataSource(properties, HikariDataSource.class);
        if (StringUtils.hasText(properties.getName())) {
            dataSource.setPoolName(properties.getName());
        }
        return dataSource;
    }
}
```

上面案例先通过<font color=red>@ConditionalOnClass(HikariDataSource.class)</font>判断classpath有没有HikariDataSource这个类，存在的时候该类的时候，再通过<font color=red>@ConditionalOnMissingBean(DataSource.class)</font>判断spring容器目前没有DataSource类型的组件，最后还对application.properties文件中的指定k-v对做了校验，这些都验证通过时，才执行static class Hikari{}这一个代码片段，该片段主要通过读取hikari数据源的配置信息，创建hikari数据源并注册到spring容器中

###### HiKariCp常用配置项

```xml
# maximumPoolSize值代表连接池中允许的最大连接数
spring.datasource.hikari.maximumPoolSize=10
# minimumldle值代表连接池中允许的空闲连接数，建议minimumIdle和maximumIdle一致
spring.datasource.hikari.minimumldle=10
# idleTimeout表示空闲连接允许存活的时长，600000毫秒，即10分钟
spring.datasource.hikari.idleTimeout=600000
# 获取连接超过30秒时认为超时
spring.datasource.hikari.connectionTimeout=30000
# maxLifetime用来设置一个connection在连接池中的存活时间，默认30分钟
spring.datasource.hikari.maxLifetime=1800000
```

##### 07. Druid数据源

经过阿里巴巴双11摧残的数据源，值得信赖，[使用手册]([https://github.com/alibaba/druid/wiki/%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98](https://github.com/alibaba/druid/wiki/常见问题))

实用的功能

* 详细的监控（非常详细）
* ExceptionSorter，针对主流的数据库的返回码都有支持
* SQL防注入
* 内置加密配置
* 众多扩展点，方便进行定制

通过spring-boot启动器或者maven下载，注意因为Hikari是spring-boot 2.x版本默认的数据库连接池，所以在引用druid连接池时，需要排除Hikari依赖，如下配置所示

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
    <exclusions>
        <exclusion>
            <artifactId>HikariCP</artifactId>
            <groupId>com.zaxxer</groupId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.10</version>
</dependency>
```

###### Druid常用配置项

* Filter配置

  ```xml
  # 全部使用默认值
  spring.datasource.druid.filters=stat,config,wall,log4j
  ```

* 密码加密

  ```xml
  spring.datasource.password=加密密码
  spring.datasource.druid.filter.config.enabled=true
  spring.datasource.druid.connection-properties=config.decrypt=true;config.decrypt.key=key
  ```

* SQL防注入

  ```xml
  spring.datasource.druid.filter.wall.enabled=true
  spring.datasource.druid.filter.wall.db-type=h2
  spring.datasource.druid.filter.wall.config.delete-allow=false
  spring.datasource.druid.filter.wall.config.drop-table-allow=false
  ```

##### 08. JdbcTemplate

JdbcTemplate时spring封装的简化jdbc操作的类，该类提供很多方法

* 万能（只能执行一些没有动态传值的语句）

  jdbcTemplate.execute(sql);

* 插入、更新、删除数据的操作
  * int update(String sql) 该方法是最简单的update方法重载形式，它直接执行传入的sql语句，并返回受影响的行数
  * int update(PreparedStatementCreator psc) 该方法执行从PreparedStatementCreator返回的语句，并返回受影响的行数
  * int update(String sql，PreparedStatementSetter pss) 该方法通过PrepareStatementSetter设置sql语句中的参数，并返回受影响的行数
  * int update(String sql，Object... args) 该方法使用Object... 设置SQL语句中的参数，要求参数不能为null，并返回受影响的行数
* 查询操作
  * List query(String sql，RowMapper rowMapper) 执行String类型参数提供的sql语句，并通过RowMapper返回一个List类型的结果
  * List query(String sql，PreparedStatementSetter pss，RowMapper rowMapper) 根据String类型参数提供的SQL语句创建PreparedStatement对象，通过RowMapper将结果返回到List中
  * List query(String sql，Object[] args，RowMapper rowMapper) 使用Object[]的值来设置sql语句中的参数值，采用RowMapper回调方法可以直接返回List类型的数据
  * Object queryForObject(String sql，RowMapper rowMapper，Object... args) 将args参数绑定到sql语句中，并通过RowMapper返回一个Object类型的单行记录
  * T[] queryForList(String sql，Object[] args，class<T> elementType) 该方法可以返回多行数据的结果，但必须是返回列表，elementType参数返回的是List元素类型

##### 09. 事务传播特性

| 传播性                    | 值   | 描述                                 |
| ------------------------- | ---- | ------------------------------------ |
| PROPAGATION_REQUIRED      | 0    | 当前有实物就用当前的，没有就用新的   |
| PROPAGATION_SUPPORTS      | 1    | 事务可有可无，不是必须的             |
| PROPAGATION_MANDATORY     | 2    | 当前一定要有事务，不然就抛异常       |
| PROPAGATION_REQUIRES_NEW  | 3    | 无论是否有事务，都起个新的事务       |
| PROPAGATION_NOT_SUPPORTED | 4    | 不支持事务，按非事务方式运行         |
| PROPAGATION_NEVER         | 5    | 不支持事务，如有事务则抛异常         |
| PROPAGATION_NESTED        | 6    | 当前有事务就在当前事务里再起一个事务 |

##### 10. 事务隔离性

|           隔离性           |  值  | 脏读 | 不可重复读 | 幻读 |
| :------------------------: | :--: | :--: | :--------: | :--: |
| ISOLATION_READ_UNCOMMITTED |  1   |  √   |     √      |  √   |
|  ISOLATION_READ_COMMITTED  |  2   |  ×   |     √      |  √   |
| ISOLATION_REPEATABLE_READ  |  3   |  ×   |     ×      |  √   |
|   ISOLATION_SERIALIZABLE   |  4   |  ×   |     ×      |  ×   |

##### 11. 编程式事务

```xml
# 注入transactionTemplate事务模板
@Autowired
private TransactionTemplate transactionTemplate;

@Override
public void run(String... args) throws Exception {
	log.info("COUNT BEFORE TRANSACTION: {}", getCount());
	transactionTemplate.execute(new TransactionCallbackWithoutResult() {
        @Override
        protected void doInTransactionWithoutResult(TransactionStatus transactionStatus) 		{
            jdbcTemplate.execute("INSERT INTO FOO (ID, BAR) VALUES (1, 'aaa')");
            log.info("COUNT IN TRANSACTION: {}", getCount());
			# 事务回滚
            transactionStatus.setRollbackOnly();
        }
	});
	log.info("COUNT AFTER TRANSACTION: {}", getCount());
}
```

上面案例中，通过jdbcTemplate.execute(sql)执行插入了一条数据，后面通过transactionStatus.setRollbackOnly()事务回滚

##### 12. 声明式事务

###### 12.1 注解形式的声明式事务

```xml
	@Override
    @Transactional
    public void insertRecord() {
        jdbcTemplate.execute("INSERT INTO FOO (BAR) VALUES ('AAA')");
    }

    @Override
    @Transactional(rollbackFor = RollbackException.class)
    public void insertThenRollback() throws RollbackException {
        jdbcTemplate.execute("INSERT INTO FOO (BAR) VALUES ('BBB')");
        throw new RollbackException();
    }
```

上面案例，insertRecord()方法添加了@Transactional注解表示该方法的执行再事务管控内执行

insertThenRollback()方法添加一条数据，主动抛出指定异常用于回滚事务

###### 12.2 xml形式的声明式事务

##### 13. Actuator Endpoint

| URL               | 作用                     |
| :---------------- | ------------------------ |
| /actuator/health  | 健康检查                 |
| /actuator/beans   | 查看容器中的所有Bean     |
| /actuator/mapping | 查看Web的URL映射         |
| /actuator/env     | 查看环境信息             |
| /actuator/info    | 展示了关于应用的一般信息 |

##### 14. ApplicationRunner接口

spring-boot程序入口是SpringApplication.run(Application.class, args);如果你想再运行该方法之前执行一段预定义的程序代码，你可以令Application这个类实现ApplicationRunner接口，然后重写run方法

```java
@SpringBootApplication
@EnableJpaRepositories
@Slf4j
public class JpaDemoApplication implements ApplicationRunner {
	
	public static void main(String[] args) {
		SpringApplication.run(JpaDemoApplication.class, args);
	}

	@Override
	public void run(ApplicationArguments args) throws Exception {
		initOrders();
	}
}
```

##### 15. JPA中的Repository是如何从Interface变成Bean的?

###### 15.1 JpaRepositoriesRegistrar

* 激活了@EnableJpaRepositories
* 返回JpaRepositoryConfigExtension

###### 15.2 RepositoryBeanDefinitionRegistrarSupport.registerBeanDefinitions

* 注册Repository Bean（类型是 JpaRepositoryFactoryBean）

###### 15.3 RepositoryConfigurationExtensionSupport.getRepositoryConfigurations

* 取得Repository配置

###### 15.4 JpaRepositoryFactory.getTargetRepository

* 创建Repository

##### 16. MyBatis Generator(MBG)

MBG可以为我们指定的数据库生成pojo类，pojoExample面向对象查询类，mapper接口，mapper接口对应的xml文件。

核心配置文件generatorConfig.xml必须解决下面五个点的问题

* jdbcConnection 如何连接数据库
* javaModelGenerator 生成的pojo类放在哪里
* sqlMapGenerator 生成的包含sql语句的xml文件放在哪里
* javaClientGenerator 生成的mapper接口放在哪里
* table 为数据库的哪些表自动生成代码

##### 17. PageHelper

PageHelper是国人开发的一个分页功能插件，适用于很多类型的数据库，它的使用非常灵活，有很多种方式，详情可以通过[传送门](<https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/en/HowToUse.md>)查看，重点在第三部分How to use it in your code

我们推荐使用以下几种种方式

```java
@Mapper
public interface CoffeeMapper {
    @Select("select * from t_coffee order by id")
    List<Coffee> findAllWithRowBounds(RowBounds rowBounds);

    @Select("select * from t_coffee order by id")
    List<Coffee> findAllWithParam(@Param("pageNum") int pageNum,
                                  @Param("pageSize") int pageSize);
}
```



```java
# 第一种方式
coffeeMapper.findAllWithRowBounds(new RowBounds(1, 3))
				.forEach(c -> log.info("Page(1) Coffee {}", c));
# 第二种方式
coffeeMapper.findAllWithParam(1, 3)
				.forEach(c -> log.info("Page(1) Coffee {}", c));
```











































