<center><h1>spring-boot注解</h1></center>

##### 00 @

* 含义：
* 标注在哪里：class/method
* 重要注解参数：value
* 用例：

##### 01 @Controller

+ 含义：表示一个控制器，返回某个静态页面（html，jsp等）或者动态页面（ModelAndView）
+ 标注在哪里：class
+ 重要注解参数：value
+ 用例：@Controller("/hello")

##### 02 @RestController

- 含义：表示一个控制器，返回字符串
- 标注在哪里：class
- 重要注解参数：value
- 用例：@RestController("/hello")

##### 03 @RequestMapping，@GetMapping, @PostMapping, @PutMapping, @DeleteMapping

- 含义：表示匹配一个url，该注解通常嵌套在@Controller或者@RestController里面使用
- 标注在哪里：method
- 重要注解参数：path/value
- 用例：@RequestMapping("/hi")

##### 04 @Configuration

- 含义：表示一个配置类，相当于spring里面的一张xml文件

- 标注在哪里：class

- 重要注解参数：

- 用例：

  ```java
  @Configuration
  public class xyzConfig{
      
  }
  ```

##### 05 @Bean

- 含义：表示一个对象，被@Bean标注的方法所返回的对象实例将作为spring容器中的一个组件，方便在整个应用的其他地方使用

- 标注在哪里：method

- 重要注解参数：

- 用例：

  ```java
  @Bean
  public Hello hello() {
      return new Hello();
  }
  ```

  上面代码表示将一个Hello对象注册为spring容器的一个组件

##### 06 @Autowried

- 含义：该注解通过<font color=red>类型匹配</font>的方式装配bean

- 标注在哪里：field/method

- 重要注解参数：

- 用例：

  标注在字段上

  ```java
  @Autowried
  private UserDao userDao;
  ```

  标注在方法上

  ```java
  @Autowried
  public void hello(UserDao userDao){
      
  }
  ```

##### 07 @Resource

- 含义：该注解通过<font color=red>名字匹配</font>的方式装配bean

- 标注在哪里：field/method

- 重要注解参数：

- 用例：

  标注在字段上

  ```java
  @Resource
  private UserDao userDao;
  ```

  标注在方法上

  ```java
  @Autowried
  public void hello(UserDao userDao){
      
  }
  ```

##### 08 @ConfigurationProperties

- 含义：该注解用于从application.properties文件中读取指定properties属性

- 标注在哪里：method

- 重要注解参数：prefix/value

- 用例：

  ```java
  @Bean
  @ConfigurationProperties("bar.datasource")
  public DataSourceProperties barDataSourceProperties() {
      return new DataSourceProperties();
  }
  ```

  该案例通过读取bar.datasource前缀下的所有properties属性，然后传递给DataSourceProperties类的无参构造方法，通过该方法构造出数据源需要的必要参数，用于后面的创建数据源对象

##### 09 @ConditionalOnClass，@CoditionalOnMissingBean，@ConditionalOnProperty

- 含义：执行某个方法前的判定条件

- 标注在哪里：method

- 重要注解参数：value/name

- 用例：

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

##### 10 @Component，@Repository，@Service

- 含义：表示spring容器的一个组件
- 标注在哪里：class
- 重要注解参数：name
- 用例：@Service("userService") @Repository("userDao")等等

##### 11 @Data

- 含义：Lombok中的一个注解，将会自动生成getter和setter方法

- 标注在哪里：class

- 重要注解参数：

- 用例：

  ```java
  @Data
  public class User{
      private Long id;
      private String bar;
  }
  ```

##### 12 @Builer

- 含义：Lombok中的一个注解，将会给目标类设计一个建造者模式的建造方法

- 标注在哪里：class

- 重要注解参数：

- 用例：

  ```java
  @Data
  @Builder
  public class User{
      private Long id;
      private String bar;
  }
  ```

##### 13 @Self4j

- 含义：Lombok中的一个注解，将会给目标类注入一个log日志记录对象

- 标注在哪里：class

- 重要注解参数：

- 用例：

  ```java
  @Self4j
  public class UserDao{
      public void insert(){
          log.trace("insert xxxx");
          log.info("insert xxxx");
          log.debug("insert xxxx");
          log.warn("insert xxxx");
          log.error("insert xxxx");
      }
  }
  ```

##### 14 @EnableTransactionManagement

- 含义：spring开启事务管理功能

- 标注在哪里：class

- 重要注解参数：mode

- 用例：

  ```java
  # 事务管理采用代理模式
  @EnableTransactionManagement(mode = AdviceMode.PROXY)
  ```

##### 15 @ComponentScan

- 含义：组件扫描
- 标注在哪里：class
- 重要注解参数：value
- 用例：@ComponentScan("packname")扫描指定报名下的组件

##### 16 @Entity，@Table，@Id，@MappedSuperclass，@GeneratedValue，@SequenceGenerator，@CreationTimestamp，@UpdateTimestamp，@Type

- 含义：JPA中的注解，@Entity定义实体类，@Table定义关联的表

- 标注在哪里：class

- 重要注解参数：

- 用例：

  ```java
  # @Entity注解默认数据库表名与类名相同，如果不相同，需要用@Table关联映射
  @Entity
  # @Table将User实体类与数据库表中的user1表关联
  @Table("user1")
  public class User{
      # 标明该字段是表中的主键
      @Id
      # 主键生成策略
      @GenerateValue
      # 默认类属性与表字段相同，如果不相同，可以通过@Column注解关联
      @Column("id1")
      private Long id;
      
      @Type(type = "package name", parameters={})
      private Money price;
      
      @Column(updateable = false)
      @CreationTimestamp
      private Date createTime;
      
      @UpdateTimestamp
      private Date updateTime;
  }
  ```

- 额外说明：@Column注解用updateable说明可不可以更新，@Type用于指定自定义类型

##### 17 @Getter，@Setter，@ToString，@NoArgsConstructor，@RequiredConstructor，@AllArgsConstructor，@CommonsLog，@Log4j2

- 含义：Lombok中的注解，见名思意
- 标注在哪里：class/filed
- 重要注解参数：
- 用例：
- 额外说明：@Data相当于同时使用@Getter和@Setter和@ToString，@RequiredArgsConstructor为所有 final 和 @NonNull 修饰的字段生成一个构造方法，通常使用@Self4j日志记录

##### 18 @ManyToMany，@JoinTable

- 含义：JPA中的注解，用以说明表之间多对多的关系

- 标注在哪里：filed

- 重要注解参数：name

- 用例：

  ```java
  public class Order implements Serializable{
      ...
      @ManyToMany
      @JoinTable(name = "T_ORDER_COFFEE")
      private List<Coffee> items;
      ...
  }
  ```

  上面的订单是通过<font color=red>T_ORDER_COFFEE</font>关联查询到coffee的，多对多关系需要三张表才能描述清楚

##### 19 @EnableTransactionManagement，@Transactional

- 含义：开启事务

- 标注在哪里：class/method

- 重要注解参数：

- 用例：

  ```java
  # 该注解通常标注再xxxApplication类上面，开启全局的注解形式的事务管理
  @EnableTransactionManagement
  public class JpaDemoApplication{
  	# 该注解通常标注在service类的某个方法上面
      @Transactional
      public void insert(){
  
      }
  }
  ```

##### 20 @EntityScan，@EnableJpaRepository

- 含义：实体扫描，开启Repository类扫描，一般当实体类或者repository接口不在xxxApplication启动类所在的包或者子包时才配置
- 标注在哪里：class
- 重要注解参数：
- 用例：@EntityScan("package name") @EnableJpaRepository("package name")

##### 21 @Mapper，@Insert，@Select，@Delete，@Update，@Results，@Result

- 含义：Mybatis相关的注解

- 标注在哪里：class/method

- 重要注解参数：

- 用例：

  ```java
  @Mapper
  public interface CoffeeMapper {
      @Insert("insert into t_coffee (name, price, create_time, update_time)"
              + "values (#{name}, #{price}, now(), now())")
      @Options(useGeneratedKeys = true)
      int save(Coffee coffee);
  
      @Select("select * from t_coffee where id = #{id}")
      @Results({
              @Result(id = true, column = "id", property = "id"),
              @Result(column = "create_time", property = "createTime"),
              // map-underscore-to-camel-case = true 可以实现一样的效果
              // @Result(column = "update_time", property = "updateTime"),
      })
      Coffee findById(@Param("id") Long id);
  }
  ```

- 额外说明：@Mapper通常标注在xxxMapper接口上面，说明该接口将以注解的形式关联sql语句，@Results和@Result一起完成结果映射

##### 22 @MapperScan

- 含义：映射器扫描
- 标注在哪里：class
- 重要注解参数：packageName
- 用例：通常标注在应用入口类，告诉spring去哪里查找mapper@MapperScan("package name")

##### 23 @MappedSuperclass

- 含义：这个注解常用在pojo类体系设计上，被该注解标注的类A一般作为pojo的父类，并且A并不会映射到数据库里面，A内的属性与其子类的属性一起映射到数据库表上，该注解不可以与@Entity或者@Table一起使用

- 标注在哪里：class

- 重要注解参数：

- 用例：

  ```java
  @MappedSuperclass
  @Data
  @NoArgsConstructor
  @AllArgsConstructor
  public class BaseEntity implements Serializable {
      @Id
      @GeneratedValue(strategy = GenerationType.IDENTITY)
      private Long id;
      @Column(updatable = false)
      @CreationTimestamp
      private Date createTime;
      @UpdateTimestamp
      private Date updateTime;
  }
  
  @Entity
  @Table(name = "T_COFFEE")
  @Builder
  @Data
  @ToString(callSuper = true)
  @NoArgsConstructor
  @AllArgsConstructor
  public class Coffee extends BaseEntity implements Serializable {
      private String name;
      @Type(type = "org.jadira.usertype.moneyandcurrency.joda.PersistentMoneyMinorAmount",
              parameters = {@org.hibernate.annotations.Parameter(name = "currencyCode", value = "CNY")})
      private Money price;
  }
  ```


##### 24 @EnableCaching，@Cacheable，@CacheEvict，@CachePut，@Caching，@CacheConfig

- 含义：spring缓存抽象中一系列的缓存注解

- 标注在哪里：class/method

- 重要注解参数：

- 额外说明

  * @EnableCaching一般标注在应用入口处，开启全局缓存配置
  * @Cacheable标注在方法上面，缓存命中直接返回，没有命中的情况下，将执行结果写入缓存中
  * @CacheEvict清楚缓存
  * @CachePut直接执行，将执行结果写入缓存中
  * @Caching可以对上面的@Cacheable、@CacheEvict、@CachePut打包
  * @CachingCofig可以对缓存做一个设置，比如设置名字

- 用例：

  - spring缓存抽象要用到的注解不需要在pom文件刻意配置，直接用就可以了

  - 在程序入口处开启全局缓存

    ```java
    @EnableCaching(proxyTargetClass = true)
    public class SpringBucksApplication implements ApplicationRunner {
    }
    ```

  - 在Service类上面配置缓存仓库名，标注一些方法的缓存类型

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

##### 25. @EnableRedisRepositories

- 含义：开启全局redisRepositories支持

- 标注在哪里：class

- 重要注解参数：

- 用例：

  1. 编写redis中的model

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

  2. 编写redis的jpa接口

  ```java
  public interface CoffeeCacheRepository extends CrudRepository<CoffeeCache, Long> {
      Optional<CoffeeCache> findOneByName(String name);
  }
  ```
  

##### 26. @EnableAspectJAutoProxy, @Aspect, @Pointcut, @Before, @After, @AfterReturing, @AfterThrowing, @Around, @Order

- 含义：spring aop相关的注解
- 标注在哪里：class/method
- 重要注解参数：
- 用例：HiKari数据库连接池是没有sql监控功能的，现在通过spring aop 对sql的执行进行拦截，利用spy进行sql日志记录

- 通过pom文件引入依赖

  ```xml
  <dependency>
      <groupId>p6spy</groupId>
      <artifactId>p6spy</artifactId>
      <version>3.8.1</version>
  </dependency>
  ```

- 在application.properties文件中对datasource信息做出调整

  ```properties
  spring.datasource.driver-class-name=com.p6spy.engine.spy.P6SpyDriver
  spring.datasource.url=jdbc:p6spy:h2:mem:testdb
  spring.datasource.username=sa
  spring.datasource.password=
  ```

- 通过spy.properties文件定义sql记录方式

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

- 在应用程序入口处开启切面增强功能

  ```java
  @EnableAspectJAutoProxy
  public class SpringBucksApplication implements ApplicationRunner {
  	
  }
  ```

- 编写切面类

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

##### 27. @JsonSerialize，@JsonDeserialize

- 含义：Json序列化相关类

- 标注在哪里：field

- 重要注解参数：using

- 用例：

  ```java
  public class MoneySerializer extends StdSerializer<Money> {
      protected MoneySerializer() {
          super(Money.class);
      }
  
      @Override
      public void serialize(Money money, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
          jsonGenerator.writeNumber(money.getAmountMinorLong());
      }
  }
  
  
  public class MoneyDeserializer extends StdDeserializer<Money> {
      protected MoneyDeserializer() {
          super(Money.class);
      }
  
      @Override
      public Money deserialize(JsonParser p, DeserializationContext ctxt) throws IOException, JsonProcessingException {
          return Money.ofMinor(CurrencyUnit.of("CNY"), p.getLongValue());
      }
  }
  ```

##### 28. @RequestBody, @ResponseBody, @ResponseStatus

- 含义：http请求相关的注解

- 标注在哪里：field

- 重要注解参数：using

- 用例：

  @RequestBody标注的形参将接收请求体中的参数，没有@RequestBody标注的形参将接受URL中的参数

  @ResponseBody设置应答报文体内容，@ResponseStatus应答响应码

##### 29. @NotEmpty, @NotNull, @Valid

- 含义：数据验证相关的注解

- 标注在哪里：field

- 重要注解参数：

- 用例：

  ```java
  public class NewCoffeeRequest {
      @NotEmpty
      private String name;
      @NotNull
      private Money price;
  }
  ```

  ```java
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


##### 30. @RestControllerAdvice, @ControllerAdvice

- 含义：控制器增强处理相关的注解

- 标注在哪里：class

- 重要注解参数：

- 用例：

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


##### 31. @Schedule，@EnableScheduling

- 含义：定时任务相关注解

- 标注在哪里：method/class

- 重要注解参数：fixedDelay

- 用例：

  ```java
  //注意在程序入口类上面开启定时任务开关@EnableScheduling
  //间隔三秒定投
      @Scheduled(fixedDelay = 3000)
      public void produceMsgScheduled(){
          jmsMessagingTemplate.convertAndSend(queue, "Scheduled : "+UUID.randomUUID().toString());
  
      }
  ```

##### 32. @JmsListener，@EnableJms

- 含义：JMS消费者相关注解

- 标注在哪里：method/class

- 重要注解参数：destination

- 用例：

  ```java
  //注意在程序入口类开启JMS开关@EnableJms
  @Component
  public class Queue_Consumer {
  
      @JmsListener(destination = "${myqueue}")
      public void receive(TextMessage textMessage){
          System.out.println(textMessage);
      }
  }
  ```

  













