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

###### erueka和zookeeper都可以提供服务注册与发现的功能，有什么区别吗？

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







































