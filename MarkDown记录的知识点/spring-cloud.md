<center><h3>Spring Cloud</h3></center>

##### 定义

SpringCloud是分布式微服务架构下的一站式解决方案，是各个微服务架构落地技术的集合体，俗称微服务全家桶

##### 技术圈

Spring Cloud的技术点有20多个，但主要学习其中的五大神兽类。

1. Eureka服务注册与发现
2. Ribbon/Feign负载均衡
3. Hystrix断路器
4. zuul路由网关
5. SpringCloud Config分布式配置中心

##### 基础问题

###### 什么是微服务

强调服务的大小，它关注的是某一个点，是具体解决某一个问题/提供落地对应服务的一个服务应用，侠义的看，可以看作Eclipse里面的一个个微服务工程/或者Module

###### 什么是为服务架构

微服务架构是一种架构模式或者说是一种架构风格，它提倡将单一应用程序划分成一组小的服务，每个服务运行在其独立的自己的进程中，服务之间互相协调、互相配合，为用户提供最终价值。服务之间采用轻量级的通信机制互相沟通（通常是基于HTTP的Restful API）。每个服务都围绕着具体业务进行构建，并且能偶被独立部署到生产环境。另外，应尽量避免统一的、集中式的服务管理机制，对具体的uige服务而言，应根据业务上下文，选择合适的语言，工具对其进行构建，可以有一个非常轻量级的集中式管理来协调这些服务，可以使用不同的语言来编写服务，也可以使用不同的数据存储。

###### 微服务之间是如何独立通讯的

###### SpringBoot和SpringCloud，请你谈谈对他们的理解

SpringBoot是微观上用于实现一个个分布式微服务的，SpringCloud是宏观上用于把控分布式微服务的，用一家医院来举例，一个个Boot应用是一个个科室，Cloud是对外的一家医院

SpringBoot专注于快速方便的开发单个个体微服务

SpringCloud是专注于全局的微服务协调整理治理框架，它将SpringBoot开发的一个个单体微服务整合并管理起来，为各个微服务之间提供、配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选、分布式会话等等集成服务

SpringBoot可以独立使用，SpringCloud依赖于SpringBoot

###### 什么是服务熔断？什么是服务降级

###### 微服务的优缺点分别是什么？说下你在项目开发中碰到的坑

* 优点

  1. 每个服务足够内聚，足够小，代码容易理解这样能聚焦一个指定的业务功能或业务需求

  2. 开发简单，开发效率提高，一个服务可能就是专一的只干一件事

  3. 微服务能够被小团队单独开发，这个小团队是2-5人
  4. 微服务是松耦合的，是有功能意义的服务，无论是在开发阶段或者是部署阶段都是独立的
  5. 微服务可以用不同的语言开发
  6. 易于集成第三方，微服务允许容易且灵活的方式集成自动部署，通过持续集成工具，如Jenkins，Hudson，bamboo
  7. 微服务允许利用融合最新技术
  8. 微服务只是业务逻辑的代码，不会和HTML,CSS等界面组件混合
  9. 每个微服务都有自己的存储能力，数据库可以独立

* 缺点

  1. 开发人员要处理分布式系统的复杂性
  2. 多服务运维难度，随着服务的增加，运维的压力也在增大
  3. 系统部署依赖
  4. 服务间通信成本
  5. 数据一致性
  6. 系统继承测试
  7. 性能监控

###### 你所知道的微服务技术栈有哪些？

| 微服务条目                               | 落地技术                      |
| ---------------------------------------- | ----------------------------- |
| 服务开发                                 | SpringBoot、Spring、SpringMVC |
| 服务配置与管理                           | Archaius、Diamond             |
| 服务注册与发现                           | Eureka、Consul、Zookeeper     |
| 服务调用                                 | Rest、RPC、gRPC               |
| 服务熔断器                               | Hystrix、Envoy                |
| 负载均衡                                 | Ribbon、Nginx                 |
| 服务接口调用（客户端调用服务的简化工具） | Feign                         |
| 消息队列                                 | Kafka、ActiveMQ               |
| 服务配置中心管理                         | SpringCloudConfig、Chef       |

###### ACID和CAP是什么？

传统的ACID，A(Atomiclty 原子性)，C(Consistency 一致性)，I(Isolation 独立性)，D(Durability 持久性)

CAP，C(Consistency 强一致性)，A(Availability 可用性)，P(Partition tolerance 分区容错性)

###### Eureka和Zookeeper都可以提供服务注册与发现的功能，有什么区别吗？

Zookeeper保证CP（强一致性和分区容错性）：当向注册中心查询服务列表时，我们可以容忍注册中心返回的时几分钟以前的注册信息，但不能接收服务直接down掉不可用。也就是说，服务注册功能对可用性的要求高于一致性，但是zk会出现这样的一种情况，当master节点因为网络故障于其他节点时区联系时，剩余节点会重新进行leader选举。问题在于，选举leader的时间太长，30-120秒，且选举期间整个zk集群时不可用的，这就导致了在选举期间服务注册功能瘫痪。在云部署的环境下，因网络问题使得zk集群失去master节点时较大概率的事情，虽然服务最终能够恢复，但是漫长的选举时间导致的注册长期不可用时不能容忍的

Eureka保证AP（可用性和分区容错性）：Eureka各个节点都是平等的，几个节点挂掉不会影响正常节点的工作，剩余的节点依然可以提供注册和发现服务。而Eureka 客户端在向某个Eureka Server注册或者如果发现连接失败，则会自动切换至其他节点，只要有一台Eureka Server还在，就能保证注册服务的可用性（保证可用性），只不过查到的信息可能不是最新的（不保证强一致性）。除此之外，Eureka还有一种自我保护机制，如果15分钟内超过85%的节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障，此时会出现以下几种情况：

1. Eureka不再从注册列表中一处因为长时间没收到心跳而应该过期的服务
2. Eureka仍然能够接收新服务的注册和查询请求，但是不会被同步到其他节点上（即保证当前节点依然可用）

###### Dubbo和Spring Cloud有什么区别

|              | Dubbo         | Spring Cloud                 |
| ------------ | ------------- | ---------------------------- |
| 服务注册中心 | Zookeeper     | Spring Cloud Eureka Netflix  |
| 服务调用方式 | RPC           | REST API                     |
| 服务监控     | Dubbo-monitor | Spring Boot Admin            |
| 断路器       | 不完善        | Spring Cloud Netflix Hystrix |
| 服务网关     | 无            | Spring Cloud Netflix Zuul    |
| 分布式配置   | 无            | Spring Cloud Config          |
| 服务跟踪     | 无            | Spring Cloud Sleuth          |
| 消息总线     | 无            | Spring Cloud Bus             |
| 数据流       | 无            | Spring Cloud Stream          |
| 批量任务     | 无            | Spring Cloud Task            |



##### Eureka知识点

###### Eureka是什么？

Eureka是Netflix的一个子模块，也是核心模块之一。Eureka是一个基于REST的服务，用于定位服务，以实现云端中间层服务发现和故障迁移。服务注册与发现对于微服务架构来说非常重要，有了服务注册和发现，只需要使用服务的标识符，就可以访问到服务，而不需要修改服务调用的配置文件，功能类似于dubbo的注册中心，比如Zookeeper

###### Eureka架构

Eureka采用了C-S的设计架构。

Eureka Server作为服务注册功能的服务器，他是服务注册中心，各个节点启动后，会在Eureka Server进行注册，如果某个Client在90秒内（3次心跳）均没有响应，将被移出注册中心

Eureka Client是一个Java客户端，用于简化Eureka Server的交互，系统中的其他微服务通过Client连接到Eureka Server并维持心跳连接（默认30秒一次）。

这样系统的维护人员就可以通过Eureka Server来监控系统中各个微服务是否正常运行。SpringCloud的一些其他模块（比如Zuul）就可以通过Eureka Server来发现系统中的其他微服务，并执行相关的逻辑

![Eureka架构图](D:\02_个人文档\照片\20[00_10_04][20190704-161605].png)

###### Eureka自我保护机制

默认情况下，如果Eureka Server在一定时间内没有接收到某个微服务的心跳，Eureka Server将会注销该实例（默认90秒）。但是当网络分区故障发生时，微服务于Eureka Server之间无法正常通信，以上行为就变得非常危险——因为微服务本身还是健康的，此时不应该注销微服务。Eureka通过自我保护模式来解决这个问题，当Eureka Server节点在短时间内丢失过多的客户端（可能发生了网络故障），一旦进入该模式，Eureka Server就会保护服务注册表中的信息，不再删除服务注册表中的数据，也即是不注销任何微服务。当网络故障恢复后，该Eureka Server节点将自动退出保护模式

一句话概括：某个时刻某个微服务不可用了，eureka不会立刻清理，依旧会对该微服务的信息进行保存

可通过eureka.server.enable-self-preservation = false来禁用自我保护模式

##### Ribbon知识点

###### Ribbon是什么？

Spring Cloud Ribbon是基于Netflix Ribbon实现的一套<font color=red>客户端</font>   <font color=purple>负载均衡</font>工具，主要功能是提供客户端的软件负载均衡算法，将Netflix的中间层服务连接在一起。Ribbon客户端组件提供一系列完善的配置项如连接超时，重试等。简单的说，就是在配置文件中列出Load Balancer(简称LB)后面所有的机器，Ribbon会自动的帮助你基于某种规则（如简单轮询，随机连接等）去连接这些机器，我们也可以自定义负载均衡算法。

Ribbon负载均衡算法属于进程内LB算法，将选择权交给客户端，案例：小明去KFC点餐，有两个队列，小明经过判断决定去人少的那一队排队。

<center><h3>分布式项目演示案例</h3></center>

##### 1. 创建一个空的Project

create new project -> empty project -> next+ -> finish

##### 2. 创建一个父的Maven Module

该父的Maven Module叫microservicecloud，用于规定子模块的一些版本信息，该模块相当于一个抽象类，被子模块继承扩展

在pom.xml文件里面添加如下信息

```xml
# 父项目一定要写这个
<packaging>pom</packaging>

<properties>
    # 定义一些property用于子项目的版本控制
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <junit.version>4.12</junit.version>
    <log4j.version>1.2.17</log4j.version>
    <lombok.version>1.16.18</lombok.version>
</properties>

<dependencyManagement>
    # 引入一些通用的jar
    <dependencies>
        # SpringCloud的版本控制
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Greenwich.SR1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        # SpringBoot的版本控制
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.1.2.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.27</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.10</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
            <version>1.2.3</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>${log4j.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

##### 3. 创建一个通用的microservicecloud-api模块

注意创建模块导航时，让该模块的parent属性继承microservicecloud，之后microservicecloud模块的pom.xml文件会自动生成一个modules标签属性

```xml
<modules>
    <module>microservicecloud-api</module>
</modules>
```

同时，microservicecloud-api的pom文件会有如下标签属性指明了它的父项目信息

```xml
<parent><!--子类里面显示声明才能有明确的继承表现，无意外就是父类默认版本，否则自己定义-->
    <artifactId>microservicecloud</artifactId>
    <groupId>cliff.ford</groupId>
    <version>1.0-SNAPSHOT</version>
</parent>
```

继续在该pom文件中引入lombok依赖

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

创建Entity包，编写部门Dept类

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Accessors(chain = true)
public class Dept implements Serializable {
    //部门编号
    private Long deptno;
    //部门名称
    private String dname;
    //来自哪个数据库，因为微服务架构可以一个服务对应一个数据库，同一个信息被存储到不同数据库
    private String db_source;
}
```

##### 5. maven打包该项目的类供其它模块使用

选择microservicecloud-api模块，右键run maven -> clean成功之后，run maven -> install，其他模块通过坐标引入即可使用该模块编写的类了

##### 6. 创建microservicecloud-provider-dept-8001模块

该模块用于提供部门的服务，注意该模块仍然继承于父模块microservicecloud

###### 编写pom文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>microservicecloud</artifactId>
        <groupId>cliff.ford</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>microservicecloud-provider-dept-8001</artifactId>

    <dependencies>
        <dependency>
            <groupId>cliff.ford</groupId>
            <artifactId>microservicecloud-api</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.0.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
            <version>RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
    </dependencies>
</project>
```

###### 编写application.yml文件

```yml
server:
  port: 8001

mybatis:
  config-location: classpath:mybatis/mybatis.cfg.xml  # mybatis配置文件所在路径
  type-aliases-package: entity                        # 所有Entity别名类所在包
  # 这里之前少写了一个s,找bug找了很久
  mapper-locations: classpath:mybatis/mapper/**/*.xml  # mapper映射文件

spring:
  application:
    name: microservicecloud-dept
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource      # 当前数据源操作类型
    driver-class-name: org.gjt.mm.mysql.Driver        # mysql驱动包
    url: jdbc:mysql://localhost:3306/cloudDB01        # 数据库url
    username: root
    password: 1234
    dbcp2:
      min-idle: 5                                     # 数据库连接池的最小维持数
      initial-size: 5                                 # 初始化连接数
      max-total: 5                                    # 最大连接数
      max-wait-millis: 200                            # 最大等待毫秒数
```

###### 编写sql脚本

```sql
create database cloudDB01 character set utf8;

create table dept(
    deptno bigint not null primary key auto_increment,
    dname varchar(60),
    db_source varchar(60)
);

insert into dept(dname, db_source) value('开发部', DATABASE()),('人事部', DATABASE()),('财务部', DATABASE()),('市场部', DATABASE()),('运维部', DATABASE());
```

###### 依次编写dao、service、controller的信息

```java
@Mapper
public interface DeptDao {
    boolean addDept(Dept dept);
    Dept findById(Long id);
    List<Dept> findAll();
}

public interface DeptService {
    boolean add(Dept dept);
    Dept get(Long id);
    List<Dept> list();
}

@Service
public class DeptServiceImpl implements DeptService {

    @Autowired
    private DeptDao deptDao;

    @Override
    public boolean add(Dept dept) {
        return deptDao.addDept(dept);
    }

    @Override
    public Dept get(Long id) {
        return deptDao.findById(id);
    }

    @Override
    public List<Dept> list() {
        System.out.println(deptDao);
        return deptDao.findAll();
    }
}


@RestController
public class DeptController {

    @Autowired
    private DeptService deptService;

    @RequestMapping(value = "/dept/add", method = RequestMethod.POST)
    public boolean add(@RequestBody Dept dept){
        return deptService.add(dept);
    }


    @GetMapping(value = "/dept/get/{id}")
    public Dept get(@PathVariable("id") Long id){
        return deptService.get(id);
    }

    @GetMapping(value = "/dept/list")
    public List<Dept> list(){
        return deptService.list();
    }
}
```

###### 根据之前yml文件的配置，创建相应文件夹及编写对应mapper.xml文件

```xml
<mapper namespace="cliff.ford.dao.DeptDao">
    <select id="findById" resultType="dept" parameterType="long">
        select * from dept where deptno = #{deptno}
    </select>

    <insert id="addDept" parameterType="dept">
        insert into dept(dname, db_source) values(#{dname}, DATABASE())
    </insert>

    <select id="findAll" resultType="dept">
        select * from dept
    </select>

</mapper>
```

###### 编写启动类并启动测试

```java
@SpringBootApplication
public class DeptProvider8001_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptProvider8001_App.class, args);
    }
}
```

##### 7. 创建microservicecloud-consumer-dept-80模块

该模块用于消费microservicecloud-privider-dept-8001提供的服务，这么做是为了隐藏真正的服务接口，注意该模块还是继承于父模块

###### pom文件编写

```xml
<parent>
        <artifactId>microservicecloud</artifactId>
        <groupId>cliff.ford</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>microservicecloud-consumer-dept-80</artifactId>
    <description>部门微服务消费者</description>

    <dependencies>
        <dependency>
            <groupId>cliff.ford</groupId>
            <artifactId>microservicecloud-api</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
            <version>1.2.8.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
    </dependencies>
```

###### 编写application.properties文件

```properties
server.port=80
```

###### 编写配置类

```java
//RestTemplate是REST风格服务的客户端调用工具类
@Configuration
public class ConfigBean {
    @Bean
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

###### 编写客户端调用代码

按照以前的开发经验，controller去调用service，service去调用dao，但这种模式是服务端的开发思维，客户端则是在controller里面通过restTemplate对象直接访问获取资源

```java
@RestController
public class DeptController_Consumer {

    private static final String REST_URL_PREFIX = "http://localhost:8001";

    @Autowired
    private RestTemplate restTemplate;

    @RequestMapping(value = "/consumer/dept/add")
    public boolean add(Dept dept){
        return restTemplate.postForObject(REST_URL_PREFIX+"/dept/add", dept, Boolean.class);
    }

    @RequestMapping(value = "/consumer/dept/get/{id}")
    public Dept get(@PathVariable("id") Long id){
        return restTemplate.getForObject(REST_URL_PREFIX+"/dept/get/"+id, Dept.class);
    }

    @SuppressWarnings("unchecked")
    @RequestMapping(value = "/consumer/dept/list")
    public List<Dept> list(){
        return restTemplate.getForObject(REST_URL_PREFIX+"/dept/list", List.class);
    }

}
```

###### 程序启动类

```java
@SpringBootApplication
public class DeptConsumer80_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer80_App.class, args);
    }
}
```

##### 8. 创建microservicecloud-eureka-7001模块

注意该模块仍然继承于父模块

###### 编写pom文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>microservicecloud</artifactId>
        <groupId>cliff.ford</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>microservicecloud-eureka-7001</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
            <version>2.1.1.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
            <version>1.2.8.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
    </dependencies>

</project>
```

###### 编写application.yml文件

```yml
server:
  port: 7001

eureka:
  instance:
    hostname: localhost
  client:
    register-with-eureka: false       # false 表示不向注册中心注册自己
    fetch-registry: false             # false 表示自己就是注册中心，我的职责是维护服务实例，并不需要去检索服务
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/   # 设置与Eureka Server交互的地址查询服务和注册服务都需要依赖该地址,注意最后面有个/
```

###### 编写启动类

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServer7001_App {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServer7001_App.class, args);
    }
}
```

###### 访问测试

http://localhost:70001

##### 9. 改造父模块microservicecloud

###### 改造pom文件

```xml
# 添加如下标签
<build>
    <finalName>microservicecloud</finalName>
    <resources>
        <resource>
            <directory>src/main/resource</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-resources-plugin</artifactId>
            <configuration>
                <delimiters>
                    <delimit>$</delimit>
                </delimiters>
            </configuration>
        </plugin>
    </plugins>
</build>
```

##### 10. 改造microservicecloud-provider-dept-8001模块

###### 改造pom.xml文件

```xml
# cloud配置
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
    <version>2.1.1.RELEASE</version>
</dependency>
# eureka client
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    <version>2.1.1.RELEASE</version>
</dependency>

# 添加actuator用于项目监控
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

###### 改造application.yml文件

```yml
eureka:
  client:  # eureka client注册进eureka服务列表内
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    instance-id: microservicecloud-dept8001   # 给自己在eureka注册中心起别名
    prefer-ip-address: true                   # 显示完整ip


info:
  app.name: cliffford-microservicecloud
  company.name: cliff.ford
  build.artifactId: $project.artfactId$
  build.version: $project.version$
```

###### 改造启动类

```java
# 添加注解
@EnableEurekaClient
```

此时再次启动eureka和provider，访问http://localhost:7001，将鼠标停留在Application右侧的超链接上面，可以有如下优化

![优化效果1](D:\02_个人文档\照片\微信截图_20190704215907.png)

点击超链接，将跳转到新页面

![优化效果2](D:\02_个人文档\照片\微信截图_20190704215633.png)

##### 11. 给microservicecloud-provider-dept-8001添加服务发现功能

###### 修改DeptController

```java
//添加代码
@Autowired
private DiscoveryClient discoveryClient;

@GetMapping(value = "/dept/discovery")
    public Object discovery(){
        List<String> list = discoveryClient.getServices();
        System.out.println(list);

        List<ServiceInstance> instances = discoveryClient.getInstances("MICROSERVICECLOUD-DEPT");
        for(ServiceInstance instance : instances){
            System.out.println(instance.getServiceId()+"\t"+
                    instance.getHost()+"\t"+
                    instance.getPort()+"\t"+
                    instance.getUri());
        }
        return this.discoveryClient;
    }
```

同时给启动类添加@EnableDiscoveryClient注解

##### 12. 给microservicecloud-consumer-dept-80添加服务发现功能

###### 修改DeptController_Consumer

```java
//添加代码
@GetMapping(value = "/consumer/dept/discovery")
    public Object discovery(){
        return restTemplate.getForObject(REST_URL_PREFIX+"/dept/discovery", Object.class);
    }
```

##### 13. 配置Eureka集群环境

###### 修改host文件

修改C:\Windows\System32\drivers\etc\host文件，添加如下代码，让浏览器识别更多的localhost

```txt
	127.0.0.1       localhost1
	127.0.0.1       localhost2
	127.0.0.1       localhost3
```

###### 新建microservicecloud-eureka-7002和microservicecloud-eureka-7003模块

这两个模块均继承于父模块，pom文件和7001模块一样，程序入口类和7001模块一样

###### 修改application.yml文件

以7003为例，7001和7002同理

```yml
server:
  port: 7003|7002|7001

eureka:
  instance:
    hostname: localhost3|localhost2|localhost1
  client:
    register-with-eureka: false       # false 表示不向注册中心注册自己
    fetch-registry: false             # false 表示自己就是注册中心，我的职责是维护服务实例，并不需要去检索服务
    service-url:
      # defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/   # 设置与Eureka Server交互的地址查询服务和注册服务都需要依赖该地址
      # 7003要告诉自己7001和7002的信息，另外两个同理
      defaultZone: http://localhost1:7001/eureka/,http://localhost2:7002/eureka/
```

###### 修改microservicecloud-provider-dept-8001模块的application.yml文件

上面配置了eureka的集群环境，但是服务提供者尚未注册到eureka集群中

```yml
eureka:
  client:  # eureka client注册进eureka服务列表内
    service-url:
      defaultZone: http://localhost1:7001/eureka,http://localhost2:7002/eureka,http://localhost3:7003/eureka
  instance:
    instance-id: microservicecloud-dept8001   # 给自己在eureka注册中心起别名
    prefer-ip-address: true                   # 显示完整ip

```

修改完之后，启动eureka集群和启动部门微服务服务端，分别打开localhost1:7001、localhost2:7002、localhost3:7003查看微服务信息

##### 14. 修改microservicecloud-consumer-dept-80模块，整合Ribbon

###### 修改pom文件

```xml
# 添加依赖
<!--Ribbon-->
<!--        <dependency>-->
<!--            <groupId>org.springframework.cloud</groupId>-->
<!--            <artifactId>spring-cloud-netflix-eureka-client</artifactId>-->
<!--            <version>2.1.1.RELEASE</version>-->
<!--        </dependency>-->
# 注意这个和上面那个的区别，是不一样的，改了一天的bug
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <version>2.1.1.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
            <version>2.1.1.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
            <version>2.1.1.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

###### 将之前的application.properties改为application.yml文件

```yml
server:
  port: 80

eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://localhost1:7001/eureka,http://localhost2:7002/eureka,http://localhost3:7003/eureka

```

###### 修改配置类

```java
@Configuration
public class ConfigBean {
    @Bean
    @LoadBalanced  // Ribbon 让restTemplate对象有了负载均衡的功能，注意是客户端
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

###### 修改对外访问控制器

```java
@RestController
public class DeptController_Consumer {

//    private static final String REST_URL_PREFIX = "http://localhost:8001";
    // 通过服务名屏蔽了ip和端口
    private static final String REST_URL_PREFIX = "http://MICROSERVICECLOUD-DEPT";
    ...
}
```

###### 修改程序入口类

```java
@SpringBootApplication
@EnableEurekaClient
public class DeptConsumer80_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer80_App.class, args);
    }
}
```

###### 测试及效果描述

1. 启动7001 7002 7003 模块
2. 启动8001模块
3. 启动80模块
4. localhost1:7001 localhost2:7002 localhost3:7003 正常，说明eureka集群配置完成
5. localhost:8001/dept/list通过，说明8001正常对外提供服务
6. localhost/consumer/dept/list通过，说明80可供客户端正常使用



























