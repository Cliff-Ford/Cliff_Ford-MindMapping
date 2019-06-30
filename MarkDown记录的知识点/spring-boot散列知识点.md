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

##### 18. redis

spring-boot使用redis非常简单，首先在pom.xml文件中引入依赖

```xml
# orm框架
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
# redis相关
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
# 数据库驱动
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>

```

接着在application.properties里面配置redis的连接属性

```properties
# 这是最简单的配置，如果你没有其他要求
# redis.host=110.110.110.110
redis.host=localhost


# 补充其他的配置
#端口号
redis.port=6379  
#如果有密码
redis.password=123456  
#客户端超时时间单位是毫秒 默认是2000
redis.timeout=10000
#最大空闲数
redis.maxIdle=300  
#连接池的最大数据库连接数。设为0表示无限制,如果是jedis 2.4以后用redis.maxTotal
#redis.maxActive=600
#控制一个pool可分配多少个jedis实例,用来替换上面的redis.maxActive,如果是jedis 2.4以后用该属性
redis.maxTotal=1000  
#最大建立连接等待时间。如果超过此时间将接到异常。设为-1表示无限制。
redis.maxWaitMillis=1000  
#连接的最小空闲时间 默认1800000毫秒(30分钟)
redis.minEvictableIdleTimeMillis=300000  
#每次释放连接的最大数目,默认3
redis.numTestsPerEvictionRun=1024  
#逐出扫描的时间间隔(毫秒) 如果为负数,则不运行逐出线程, 默认-1
redis.timeBetweenEvictionRunsMillis=30000  
#是否在从池中取出连接前进行检验,如果检验失败,则从池中去除连接并尝试取出另一个
redis.testOnBorrow=true  
#在空闲时检查有效性, 默认false
redis.testWhileIdle=true
```

然后在你的某个配置类中配置jedisPool作为容器中的一个Bean

```java
	@Bean
	@ConfigurationProperties("redis")
	public JedisPoolConfig jedisPoolConfig() {
		return new JedisPoolConfig();
	}

	@Bean(destroyMethod = "close")
	public JedisPool jedisPool(@Value("${redis.host}") String host) {
		return new JedisPool(jedisPoolConfig(), host);
	}
```

最后，记住jedis连接并不是线程安全的，所以将它放在一个try块里面执行

```java
try (Jedis jedis = jedisPool.getResource()) {
    coffeeService.findAllCoffee().forEach(c -> {
        jedis.hset("springbucks-menu",
                   c.getName(),
                   Long.toString(c.getPrice().getAmountMinorLong()));
    });

    Map<String, String> menu = jedis.hgetAll("springbucks-menu");
    log.info("Menu: {}", menu);

    String price = jedis.hget("springbucks-menu", "espresso");
    log.info("espresso - {}",
             Money.ofMinor(CurrencyUnit.of("CNY"), Long.parseLong(price)));
}
```

##### 19. spring缓存抽象

<font color=red>spring为不同的缓存提供一层抽象</font>

* 为Java方法增加缓存，缓存执行结果
* 支持ConcurrentMap、EhCache、Caffeine、JCache(JSR-07)
* 接口
  * org.springframework.cache.Cache
  * org.springframework.cache.CacheManager

<font color=red>使用什么类型的缓存</font>

* 一些长久不会变的，或者一天之内不会变的，或者能接受变化存在一定的延时，这些数据应该缓存在界面内部，也称之为本地缓存，同时需要设置一个过期时间
* 在分布式缓存集群中，需要保持集群内机器访问的一致性时，就需要将数据缓存在中心处，比如用redis控制
* 一些数据的读写比很低的情况下，不需要使用缓存

<font color=red>@EnableCaching，@Cacheable，@CacheEvict，@CachePut，@Caching，@CacheConfig</font>

- 含义：spring缓存抽象中一系列的缓存注解
- 标注在哪里：class/method
- 重要注解参数：
- 额外说明
  - @EnableCaching一般标注在应用入口处，开启全局缓存配置
  - @Cacheable标注在方法上面，缓存命中直接返回，没有命中的情况下，将执行结果写入缓存中，常和get方法一起用
  - @CacheEvict清楚缓存，常和delete方法一起用
  - @CachePut直接执行，将执行结果写入缓存中
  - @Caching可以对上面的@Cacheable、@CacheEvict、@CachePut打包
  - @CachingCofig可以对缓存做一个设置，比如设置名字
- 用例：

<font color=red>本地缓存用例</font>

* spring缓存抽象要用到的注解不需要在pom文件刻意配置，直接用就可以了

* 在程序入口处开启全局缓存

  ```java
  @EnableCaching(proxyTargetClass = true)
  public class SpringBucksApplication implements ApplicationRunner {
  }
  ```

* 在Service类上面配置缓存仓库名，标注一些方法的缓存类型

  ```java
  @Slf4j
  @Service
  @CacheConfig(cacheNames = "coffee")
  public class CoffeeService {
      @Autowired
      private CoffeeRepository coffeeRepository;
  
      @Cacheable
      public List<Coffee> findAllCoffee() {
          return coffeeRepository.findAll();
      }
  
      @CacheEvict
      public void reloadCoffee() {
      }
  
      public Optional<Coffee> findOneCoffee(String name) {
          ExampleMatcher matcher = ExampleMatcher.matching()
                  .withMatcher("name", exact().ignoreCase());
          Optional<Coffee> coffee = coffeeRepository.findOne(
                  Example.of(Coffee.builder().name(name).build(), matcher));
          log.info("Coffee Found: {}", coffee);
          return coffee;
      }
  }
  ```

<font color=red>redis缓存用例</font>

* pom文件引入cache启动器和redis启动器

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-cache</artifactId>
  </dependency>
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-redis</artifactId>
  </dependency>
  ```

* 配置application.properties属性

  ```properties
  # 配置缓存类型是redis
  spring.cache.type=redis
  # 配置缓存仓库名是coffee
  spring.cache.cache-names=coffee
  # 配置缓存有效时间为5秒
  spring.cache.redis.time-to-live=5000
  # 不缓存空值
  spring.cache.redis.cache-null-values=false
  
  spring.redis.host=localhost
  ```

##### 20. redisTemplate

redisTemplate和jdbcTemplate相似，是spring简化redis开发写的模板类

案例：利用redisTemplate查询redis中缓存的数据

* pom文件引入依赖

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-redis</artifactId>
  </dependency>
  ```

* 在业务代码中注入redisTemplate

  ```java
  @Autowired
  private RedisTemplate<String, Coffee> redisTemplate;
  ```

* 查询指定数据

  ```java
  public Optional<Coffee> findOneCoffee(String name) {
          HashOperations<String, String, Coffee> hashOperations = redisTemplate.opsForHash();
      # 缓存是否命中
          if (redisTemplate.hasKey(CACHE) && hashOperations.hasKey(CACHE, name)) {
              log.info("Get coffee {} from Redis.", name);
              # 命中返回
              return Optional.of(hashOperations.get(CACHE, name));
          }
      # 没有命中时，到数据库查询
          ExampleMatcher matcher = ExampleMatcher.matching()
                  .withMatcher("name", exact().ignoreCase());
          Optional<Coffee> coffee = coffeeRepository.findOne(
                  Example.of(Coffee.builder().name(name).build(), matcher));
          log.info("Coffee Found: {}", coffee);
          if (coffee.isPresent()) {
              log.info("Put coffee {} to Redis.", name);
              hashOperations.put(CACHE, name, coffee.get());
              redisTemplate.expire(CACHE, 1, TimeUnit.MINUTES);
          }
          return coffee;
      }
  ```

##### 21. redisRepository

mysql或者h2都是数据库，spring-boot通过jdbcTemplate或者jpa都可以操作数据库

redis也是数据库，spring-boot通过redisTemplate和jpa也可以操作数据库

但是通过jpa操作redis时略有不同，案例如下

* 通过pom引入关键依赖

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
  </dependency>
  
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-redis</artifactId>
  </dependency>
  ```

* 编写model

  ```java
  # @RedisHash相当于@Entity,value的值相当于表名，timeToLive是存活时间，一分钟
  @RedisHash(value = "springbucks-coffee", timeToLive = 60)
  @Data
  @NoArgsConstructor
  @AllArgsConstructor
  @Builder
  public class CoffeeCache {
      # 一级索引
      @Id
      private Long id;
      # 二级索引
      @Indexed
      private String name;
      private Money price;
  }
  ```

* 编写repository接口

  ```xml
  public interface CoffeeCacheRepository extends CrudRepository<CoffeeCache, Long> {
      Optional<CoffeeCache> findOneByName(String name);
  }
  ```

* 在你的业务代码中使用它

  ```java
  public Optional<Coffee> findSimpleCoffeeFromCache(String name) {
      Optional<CoffeeCache> cached = cacheRepository.findOneByName(name);
      if (cached.isPresent()) {
          CoffeeCache coffeeCache = cached.get();
          Coffee coffee = Coffee.builder()
              .name(coffeeCache.getName())
              .price(coffeeCache.getPrice())
              .build();
          log.info("Coffee {} found in cache.", coffeeCache);
          return Optional.of(coffee);
      } else {
          Optional<Coffee> raw = findOneCoffee(name);
          raw.ifPresent(c -> {
              CoffeeCache coffeeCache = CoffeeCache.builder()
                  .id(c.getId())
                  .name(c.getName())
                  .price(c.getPrice())
                  .build();
              log.info("Save Coffee {} to cache.", coffeeCache);
              cacheRepository.save(coffeeCache);
          });
          return raw;
      }
  }
  ```

##### 22. project reactor

应该看看这篇文章[使用教程](<https://blog.51cto.com/liukang/2090191>)，[原理教程](<http://springcloud.cn/view/366>)

```java
userService.getFavorites(userId) ➊
           .flatMap(favoriteService::getDetails)  ➋
           .switchIfEmpty(suggestionService.getSuggestions())  ➌
           .take(5)  ➍
           .publishOn(UiUtils.uiThreadScheduler())  ➎
           .subscribe(uiList::show, UiUtils::errorPopup);  ➏
```

➊ 根据用户ID获得喜欢的信息（打开一个 `Publisher`）
➋ 使用 `flatMap` 操作获得详情信息
➌ 使用 `switchIfEmpty` 操作，在没有喜欢数据的情况下，采用系统推荐的方式获得
➍ 取前五个
➎ 在 `uiThread` 上进行发布
➏ 最终的消费行为

➊➋➌➍ 的行为就看起来很直观，这个和我们使用 [`Java 8 中的 Streams 编程`](http://www.runoob.com/java/java8-streams.html) 极为相近，但是这里实现和 Strem 是不同的，在后续的分析会展开。
➎ 和 ➏ 在我们之前的编码（注：传统后台服务）中没有遇见过类似的，这里的行为，我们可以在后续的 [`Reference#schedulers`](http://projectreactor.io/docs/core/release/reference/#schedulers) 中可以得知，`publishOn` 将影响后续的行为操作所在的线程，那我们就明白了，之前的操作会在某个线程中执行，而最后一个 `subscribe()` 函数将在 `uiThread` 中执行。

##### 23. ReactiveStringRedisTemplate

reactiveStringRedisTemplate是一个和stringRedisTemplate相类似的简化string类型操作redis的类，该类的使用需要引入依赖

* pom文件引入依赖

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
  </dependency>
  ```

* 在你的配置类中注册reactiveStringRedisTemplate

  ```java
  @Bean
      ReactiveStringRedisTemplate reactiveRedisTemplate(ReactiveRedisConnectionFactory factory) {
          return new ReactiveStringRedisTemplate(factory);
      }
  ```

* 将查询到的coffee以name作为key，price作为价格put到redis中

  ```java
  CountDownLatch cdl = new CountDownLatch(1);
  Flux.fromIterable(list) //list是所有的coffee
      .publishOn(Schedulers.single()) //后面的操作将发布到一个新的线程上面去处理
      .doOnComplete(() -> log.info("list ok")) //完成后输出list ok
      .flatMap(c -> {     // 让流里面的元素经过一层映射成为新的包装好的流
          log.info("try to put {},{}", c.getName(), c.getPrice());
          return hashOps.put(KEY, c.getName(), c.getPrice().toString());
      })
      .doOnComplete(() -> log.info("set ok"))
      .concatWith(redisTemplate.expire(KEY, Duration.ofMinutes(1)))//设置缓存有效时间
      .doOnComplete(() -> log.info("expire ok"))
      .onErrorResume(e -> {  //错误处理
          log.error("exception {}", e.getMessage());
          return Mono.just(false);
      })  // 订阅，订阅之后才开始工作
      .subscribe(b -> log.info("Boolean: {}", b), 
                 e -> log.error("Exception {}", e.getMessage()),
                 () -> cdl.countDown());
  ```

##### 24. ReactiveMongoTemplate

和23类似的，reactiveMongoTemplate是一个操作mongo的类，在你安装好mongo之后，你可能需要如下操作

* show dbs(查看目前拥有的数据库)

* use springbucks(使用springbucks库,没有会自动创建)

* mongo每个数据库的用户和密码都可以不一样，所以你需要给这个库建立用户和密码

  ```shell
  db.createUser({user: "root", pwd: "1234", roles: [{ role: "root", db: "admin" }]})
  ```

* pom文件引入依赖

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-mongodb-reactive</artifactId>
  </dependency>
  ```

* appliaction.properties编写数据库连接

  ```properties
  # spring.data.mongodb.uri=mongodb://root_name:password@ip:port/database_name
  spring.data.mongodb.uri=mongodb://root:1234@203.195.177.110:27017/springbucks
  ```

* 依赖注入reactiveMongoTemplate

  ```java
  @Autowired
  private ReactiveMongoTemplate mongoTemplate;
  ```

* 核心代码将传入的coffee存到mongo中

  ```java
  private void startFromInsertion(Runnable runnable) {
  		mongoTemplate.insertAll(initCoffee())
  				.publishOn(Schedulers.elastic())
  				.doOnNext(c -> log.info("Next: {}", c))
  				.doOnComplete(runnable)
  				.doFinally(s -> {
  					cdl.countDown();
  					log.info("Finnally 1, {}", s);
  				})
  				.count()
  				.subscribe(c -> log.info("Insert {} records", c));
  	}
  ```

* mongo数据库可以通过如下代码查看

  ```shell
  db.coffee.find().pretty()
  ```

##### 25. AOP HiKari SPY

* Spring AOP的核心概念

  |     概念      |                            含义                             |
  | :-----------: | :---------------------------------------------------------: |
  |    Aspect     |                            切面                             |
  |  Join Point   |          连接点，Spring AOP里总是代表一次方法执行           |
  |    Advice     |                  通知，在连接点执行的动作                   |
  |   Pointcut    |                 切入点，说明如何匹配连接点                  |
  | Introduction  |            引入，为现有类型声明额外的方法和属性             |
  | Target object |                          目标对象                           |
  |   AOP proxy   | AOP代理对象，可以是JDK动态代理（有接口），也可以是CGLIB代理 |
  |    Weaving    |        织入，连接切面与目标对象或类型创建代理的过程         |

* HiKari数据库连接池是没有sql监控功能的，现在通过spring aop 对sql的执行进行拦截，利用spy进行sql日志记录

  * 通过pom文件引入依赖

    ```xml
    <dependency>
        <groupId>p6spy</groupId>
        <artifactId>p6spy</artifactId>
        <version>3.8.1</version>
    </dependency>
    ```

  * 在application.properties文件中对datasource信息做出调整

    ```properties
    spring.datasource.driver-class-name=com.p6spy.engine.spy.P6SpyDriver
    spring.datasource.url=jdbc:p6spy:h2:mem:testdb
    spring.datasource.username=sa
    spring.datasource.password=
    ```

  * 通过spy.properties文件定义sql记录方式

    ```properties
    # 单行日志
    logMessageFormat=com.p6spy.engine.spy.appender.SingleLineFormat
    # 使用Slf4J记录sql
    appender=com.p6spy.engine.spy.appender.Slf4JLogger
    # 是否开启慢SQL记录
    outagedetection=true
    # 慢SQL记录标准，单位秒
    outagedetectioninterval=2
    ```

  * 在应用程序入口处开启切面增强功能

    ```java
    @EnableAspectJAutoProxy
    public class SpringBucksApplication implements ApplicationRunner {
    	
    }
    ```

  * 编写切面类

    ```java
    @Aspect
    @Component
    @Slf4j
    public class PerformanceAspect {
    //    @Around("execution(* geektime.spring.springbucks.repository..*(..))")
        @Around("repositoryOps()")
        public Object logPerformance(ProceedingJoinPoint pjp) throws Throwable {
            long startTime = System.currentTimeMillis();
            String name = "-";
            String result = "Y";
            try {
                name = pjp.getSignature().toShortString();
                // pjp.proceed()方法通知目标方法执行
                return pjp.proceed();
            } catch (Throwable t) {
                result = "N";
                throw t;
            } finally {
                long endTime = System.currentTimeMillis();
                log.info("{};{};{}ms", name, result, endTime - startTime);
            }
        }
    
        //计算连接点
        @Pointcut("execution(* geektime.spring.springbucks.repository..*(..))")
        private void repositoryOps() {
        }
    }
    ```

##### 26. Spring MVC

* DispatcherServlet 核心入口
  * Controller 控制器，每一个url该怎么处理
  * xxxResolver 各种解析器
    * ViewResolver 视图解析器
    * HandlerExceptionResolver 异常解析器
    * MultipartResolver 文件解析器
  * HandlerMapping 处理映射（url是怎么映射对应控制器的）

##### 27. spring应用程序上下文

spring的应用程序上下文允许有多个，每个应用程序上下文之间还允许有父子关系，但这个父子关系并不等同Java的继承关系

案例：通过aop对不同应用程序上下文中的同名bean做增强处理，看看哪一些有效，哪一些失效

* 首先定义TestBean

  ```java
  @AllArgsConstructor
  @Slf4j
  public class TestBean {
      private String context;
  
      public void hello() {
          log.info("hello " + context);
      }
  }
  ```

* 定义一个切面，该切面<font color=red>预计</font>会对所有的以testBean开头的bean做后处理增强

  ```java
  @Aspect
  @Slf4j
  public class FooAspect {
      @AfterReturning("bean(testBean*)")
      public void printAfter() {
          log.info("after hello()");
      }
  }
  ```

* 定义注解形式的配置类，该配置类会被<font color=red>注解形式的应用程序上下文</font>识别

  ```java
  @Configuration
  @EnableAspectJAutoProxy
  public class FooConfig {
      @Bean
      public TestBean testBeanX() {
          return new TestBean("foo");
      }
  
      @Bean
      public TestBean testBeanY() {
          return new TestBean("foo");
      }
  
      @Bean
      public FooAspect fooAspect() {
          return new FooAspect();
      }
  }
  ```

* 注解形式的切面会对testBean做出后处理增强

  ```java
  ApplicationContext fooContext = new     	AnnotationConfigApplicationContext(FooConfig.class);		
  
  TestBean bean = fooContext.getBean("testBeanX", TestBean.class);
  bean.hello();
  ```

* 现在定义applicationContext.xml文件,里面只有如下一个bean

  ```xml
  <bean id="testBeanX" class="geektime.spring.web.context.TestBean">
      <constructor-arg name="context" value="Bar" />
  </bean>
  ```

* xml形式的配置文件会被xml形式的应用程序上下文识别

  ```java
  ApplicationContext fooContext = new AnnotationConfigApplicationContext(FooConfig.class);
  
  		TestBean foo = fooContext.getBean("testBeanX", TestBean.class);
  		foo.hello();
  
  ClassPathXmlApplicationContext barContext = new ClassPathXmlApplicationContext(
  				new String[] {"applicationContext.xml"});
  
  		TestBean bar = barContext.getBean("testBeanX", TestBean.class);
  		bar.hello();
  ```

  这里的foo是被增强的

  这里的bar是没有被增强的，因为application.xml文件里面没有配置切面类，也没有开启aop功能

  注意：这里拿到的bar是application.xml里面的testBeanX

* 现在让注解形式的应用程序上下文作为xml形式的应用程序上下文的父上下文

  ```java
  ApplicationContext fooContext = new AnnotationConfigApplicationContext(FooConfig.class);
  
  		TestBean foo = fooContext.getBean("testBeanX", TestBean.class);
  		foo.hello();
  
  		log.info("=============");
  		ClassPathXmlApplicationContext barContext = new ClassPathXmlApplicationContext(
  				new String[] {"applicationContext.xml"}, fooContext);
  
  		TestBean bar = barContext.getBean("testBeanX", TestBean.class);
  		bar.hello();
  		# hello方法会被增强
  		bar = barContext.getBean("testBeanY", TestBean.class);
  		bar.hello();
  ```

  这时再去取testBeanX，会是哪一个应用程序上下文的bean呢？

  和java继承类似的，这里的bean会发生覆盖，所以取到的是xml里面的bean,由于该配置文件里面没有配置切面类，也没有开启aop功能，所以该bean不会被增强

  这时再通过barContext去取testBeanY，能不能取到呢？是可以的，和java继承类似，找不到的bean,递归去父文本中找，一直找不到就抛异常

* 现在给applicationContext.xml文件配置切面类，和开启aop功能

  ```xml
  	<aop:aspectj-autoproxy/>
  
      <bean id="testBeanX" class="geektime.spring.web.context.TestBean">
          <constructor-arg name="context" value="Bar" />
      </bean>
  
      <bean id="fooAspect" class="geektime.spring.web.foo.FooAspect" />
  ```

  现在xml里面的testBeanX也具备了后增强功能了

* 想想切面类在注解形式的配置里面已经配过一次了，在xml里面再配置一次，是不是重复了呢？试着将xml里面的切面类去掉，也是可以增强的

  这说明，应用程序在做aop是，切面是允许公用的，但如果你这个this.Context不开启aop功能的话，那你这个bean它也是不能被增强的，你可以试着将xml里面的aop功能关掉，再看看bar是否被增强了

##### 28. spring mvc 的请求处理流程

浏览器请求进来时首先会去到前置处理器Frontcontroller，该处理器根据url传递给指定的controller类，控制器类处理完之后返回Model和视图名给前置处理器，前置处理器拿到model和视图名后调用View Template动态渲染html页面，然后将html传递给前置处理器，前置处理器将拿到的html资源返回个浏览器

##### 29. 复杂的URL设计

```java
# url参数不能有name
@GetMapping(path = "/hello", params = "!name")
    
    
# url参数要有name
@GetMapping(path = "/hello", params = "name")
    
    
# 生产一个MediaType.APPLICATION_JSON_UTF8_VALUE类型数据
# /{id}表示将url中的斜杠后面的值作为id这个参数的值
@RequestMapping(path = "/{id}", method = RequestMethod.GET,
            produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
    
# /haha这个请求必须符合consumes定义的数据类型，同时处理后指定了返回码和数据格式
@PostMapping(path = "/haha", consumes = MediaType.APPLICATION_JSON_VALUE,
            produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
@ResponseStatus(HttpStatus.CREATED)
```

##### 30. controller层面的数据校验

```java
// 每一个NewCoffeeRequest对象都会对name和price属性进行条件设定，后面配合@Valid进行验证
public class NewCoffeeRequest {
    @NotEmpty
    private String name;
    @NotNull
    private Money price;
}
```

```java
// @Valid会对newCoffee对象进行验证，验证结果放在BindingResult对象里面
@PostMapping(path = "/", consumes = MediaType.APPLICATION_FORM_URLENCODED_VALUE)
@ResponseBody
@ResponseStatus(HttpStatus.CREATED)
public Coffee addCoffee(@Valid NewCoffeeRequest newCoffee,
                        BindingResult result) {
    if (result.hasErrors()) {
        // 这里先简单处理一下，后续讲到异常处理时会改
        log.warn("Binding Errors: {}", result);
        return null;
    }
    return coffeeService.saveCoffee(newCoffee.getName(), newCoffee.getPrice());
}
```

##### 31. 文件作为url的一部分

```java
	@PostMapping(path = "/", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    @ResponseBody
    @ResponseStatus(HttpStatus.CREATED)
    public List<Coffee> batchAddCoffee(@RequestParam("file") MultipartFile file) {
        List<Coffee> coffees = new ArrayList<>();
        if (!file.isEmpty()) {
            BufferedReader reader = null;
            try {
                reader = new BufferedReader(
                        new InputStreamReader(file.getInputStream()));
                String str;
                while ((str = reader.readLine()) != null) {
                    String[] arr = StringUtils.split(str, " ");
                    if (arr != null && arr.length == 2) {
                        coffees.add(coffeeService.saveCoffee(arr[0],
                                Money.of(CurrencyUnit.of("CNY"),
                                        NumberUtils.createBigDecimal(arr[1]))));
                    }
                }
            } catch (IOException e) {
                log.error("exception", e);
            } finally {
                IOUtils.closeQuietly(reader);
            }
        }
        return coffees;
    }
```

##### 32. Spring MVC的异常解析

###### 核心接口

* HandlerExceptionResolver

###### 实现类

* SimpleMappingExceptionResolver
* DefaultHandlerExceptionResolver
* ResponseStatusExceptionResolver
* ExceptionHandlerExceptionResolver

```java
@RestControllerAdvice
public class GlobalControllerAdvice {
    # 处理指定类型的异常
    @ExceptionHandler(ValidationException.class)
    # 返回指定异常码
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public Map<String, String> validationExceptionHandler(ValidationException exception) {
        Map<String, String> map = new HashMap<>();
        map.put("message", exception.getMessage());
        return map;
    }
}
```

##### 33. Spring MVC拦截器

###### 核心接口

* HandlerIntercetor
  * boolean preHandle() 目标方法执行前进行处理，比如权限检查，如果返回false将终止目标方法执行，返回true将继续执行
  * void postHandler() 目标方法执行完，视图未渲染之前执行
  * void afterCompletion() 目标方法执行完，视图渲染之后执行

###### 案例

* 编写自定义的拦截器

  ```java
  @Slf4j
  public class PerformanceInterceptor implements HandlerInterceptor {
      private ThreadLocal<StopWatch> stopWatch = new ThreadLocal<>();
  
      @Override
      public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
          StopWatch sw = new StopWatch();
          stopWatch.set(sw);
          sw.start();
          return true;
      }
  
      @Override
      public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
          stopWatch.get().stop();
          stopWatch.get().start();
      }
  
      @Override
      public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
          StopWatch sw = stopWatch.get();
          sw.stop();
          String method = handler.getClass().getSimpleName();
          if (handler instanceof HandlerMethod) {
              String beanType = ((HandlerMethod) handler).getBeanType().getName();
              String methodName = ((HandlerMethod) handler).getMethod().getName();
              method = beanType + "." + methodName;
          }
          log.info("{};{};{};{};{}ms;{}ms;{}ms", request.getRequestURI(), method,
                  response.getStatus(), ex == null ? "-" : ex.getClass().getSimpleName(),
                  sw.getTotalTimeMillis(), sw.getTotalTimeMillis() - sw.getLastTaskTimeMillis(),
                  sw.getLastTaskTimeMillis());
          stopWatch.remove();
      }
  }
  ```

* 在程序入口类上面实现WebMvcConfigurer接口，该接口是用来自定义webMvc功能的

  ```java
  //重写addInterceptors方法，并配置需要被拦截的uri
  @Override
  public void addInterceptors(InterceptorRegistry registry) {
      registry.addInterceptor(new PerformanceInterceptor())
          .addPathPatterns("/coffee/**").addPathPatterns("/order/**");
  }
  ```

  

























