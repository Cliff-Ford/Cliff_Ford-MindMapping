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

##### 03 @RequestMapping

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

  





























