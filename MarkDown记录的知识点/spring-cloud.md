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

###### Ribbon有哪些负载均衡算法

| RoundRobinRule            | 轮询                                                         |
| ------------------------- | ------------------------------------------------------------ |
| RandomRule                | 随机                                                         |
| AvailabilityFilteringRule | 会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，还有并发的连接数量超过阈值的服务，然后对剩余的服务列表按照轮询策略进行访问 |
| WeightedResponseTimeRule  | 根据平均响应时间计算所有服务的权重，响应时间越快服务权重越大越容易被选中，刚启动时如果统计信息不足，则使用RoundRobinRule策略，等统计信息足够了会自动切换为WeightedResponseTimeRule |
| RetryRule                 | 先按照RoundRobinRule策略获取服务，如果获取服务失败则在指定时间内进行重试，获取可用的服务 |
| BestAvailableRule         | 会先过滤掉由于多次访问故障处于断路器跳闸的服务，然后选择一个并发量最小的服务 |
| ZoneAvoidanceRule         | 默认规则，复合判断server所在区域的性能和server的可用性选择服务 |



###### 如何自定义负载均衡算法？

Ribbon的负载均衡算法，均继承于AbstractLoadBalancerRule抽象类，该抽象类实现了IRule, IClientConfigAware接口，简单来说，你可以编写一个类继承于AbstractLoadBalancerRule抽象类，然后重写部分方法实现自己的负载均衡算法，再利用配置类显示配置，同时在主启动类上面添加如下注解信息

@RibbonClient(name="service-name",  configuration=MyRobbinRule.class)，并且MyRobbinRule类不能放在@ComponentScan所能扫描的包及其子包下，而@SpringBoootApplication是一个混合注解（含有@ComponentScan），所以你不能将MyRobbinRule放在程序入口类所在的包及其子包，你需要在程序入口类的最上层包建立一个同级包，放在这个同级包里面

```java
@Configuration
public class MyRobbinRule{
    @Bean
    public IRule myRandomRule(){
        return new MyRandomRule();
    }
}
```

当然，Ribbon提供的7种负载均衡算法已经够用了

##### Feign知识点

###### Feign是什么？

Feign是一个声明式的Web服务客户端，使得编写Web服务客户端变得非常容易，只需要创建一个接口，然后再上面添加注解即可使用

Feign集成了Ribbon,利用Ribbon维护了Microservicecloud-dept的服务列表，并且通过轮询实现了客户端的负载均衡，而与Ribbon不同的是，通过feign只需要定义服务绑定接口且以声明式的方法，优雅而简单的实现了服务调用

##### Hystrix知识点

###### Hystix是什么？

Hystrix是一个用于处理分布式系统的延迟和容错的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时，异常等，Hystrix能够保证在一个依赖出问题的情况下，不会导致整体服务失败，便面级联故障，以提供分布式系统的弹性

”断路器“本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝），向调用方返回一个符合预期的，可处理的备选响应（FallBack），而不是长时间的等待或者抛出调用方无法处理的异常，这样就保证了服务调用方的线程不会被长时间，不必要地占用，从而避免了故障在分布式系统种的蔓延，乃至雪崩

###### 服务雪崩

多个微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其他的微服务，这就是所谓的”扇出“。如果扇出链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓”雪崩效应“。

对于高流量的应用来说，单一的后端依赖可能会导致所有服务器上的所有资源都在几秒钟内饱和，比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加，备份队列，线程和其他系统的资源紧张，导致整个系统发生发生更多的级联故障，这些都表示需要对故障和延迟进行隔离和管理，以便单个依赖关系的失败，能不能取消整个应用程序或系统

###### 服务熔断

熔断机制是应对服务雪崩的一种微服务链路保护机制

当扇出链路的某个微服务不可用或者响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，快速返回”错误“的响应信息。当检测到该节点微服务调用i昂应正常后恢复调用链路。

在SpringCloud框架里熔断机制通过Hystrix实现，Hystrix会见你控微服务间调用的状况，当失败的调用到达一定阈值，缺省时5秒内20次调用失败就会启动熔断机制。

熔断机制的注解时@HystrixCommand

###### 服务降级

客户端针对服务异常不可用时的一种缺省机制，需要配合@FeignClient(value="service-name",fallback=xxxFallBackFactory.class)来处理

##### Zuul知识点

###### Zuul是什么？

Zuul包含了对请求的路由和过滤两个最主要的功能：

* 路由功能负责将外部请求转发到具体的微服务实例上，是实现外部访问统一入口的基础
* 过滤功能负责对请求的处理过程进行干预，是实现请求校验，服务聚合等功能的基础

Zuul和Eureka进行整合，将Zuul自身注册为Eureka服务治理下的应用，同时从Eureka中获得其他微服务的消息，也即以后的访问微服务都是通过Zuul跳转后获得

##### SpringCloud Config知识点

###### SpringCloud Config是什么？

SpringCloud Config为微服务架构中的微服务提供及集中化的外部配置支持，配置服务器为各个不同微服务应用的所有环境提供了一个中心化的外部配置

###### SpringCloud Config是C-S架构

* 服务端也称为分布式配置中心，它是一个独立的微服务应用，用来连接配置服务器并为客户端提供获取配置信息，加密/解密信息等访问接口
* 客户端则是通过指定的配置中心来管理应用资源，以及与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息，配置服务器默认采用git来存储配置信息，这样就有助于对环境配置进行版本管理，并且可以通过git客户端工具来方便的管理和访问配置内容

###### bootstrap.yml文件

application.yml是用户级的资源配置项，bootstrap.yml文件是一个系统级的资源配置项，优先级更高

SpringCloud 会创建一个Bootstrap Context作为Spring应用的Application Context的父上下文，初始化的时候，Bootstrap Context负责从外部资源加载配置属性并解析配置，这两个上下文共享一i个从外部获取的Evironment。Bootstrap属性有更高的优先级，他们不会被本地配置项覆盖，Bootstrap Context和Application Context有着不同的约定，所以新增了一个bootstrap.yml文件，保证Bootstrap Context和Application Context配置的分离

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

##### 15. 创建microservicecloud-provider-dept-8002和microservicecloud-provider-dept-8003搭建服务集群

注意这两个模块都继承于父模块，pom文件相同，程序入口类名更改一下

###### 修改applcation.yml文件

```yml
# 端口号需要改
server:
  port: 8002

# 对外的服务名字不能改，Ribbon是根据服务名负载均衡的，Eureka也是根据服务名划分服务的
spring:
  application:
    name: microservicecloud-dept
  # 数据库连接改成2号库
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource      # 当前数据源操作类型
    driver-class-name: org.gjt.mm.mysql.Driver        # mysql驱动包
    url: jdbc:mysql://localhost:3306/cloudDB02        # 数据库url
    username: root
    password: 1234
    
eureka:
  client:  # eureka client注册进eureka服务列表内
    service-url:
      defaultZone: http://localhost1:7001/eureka,http://localhost2:7002/eureka,http://localhost3:7003/eureka
  instance:
  	# instance-id也需要改
    instance-id: microservicecloud-dept8002   # 给自己在eureka注册中心起别名
    prefer-ip-address: true                   # 显示完整ip
```

###### 测试

1. 启动7001 7002 7003
2. 启动8001 8002
3. 启动80
4. 测试localhost1:7001 localhost2:7002 localhost3:7003
5. 测试localhost:8001/dept/list localhost:8002/dept/list
6. 测试localhost/consumer/dept/list
7. 测试的目的是为了证明Ribbon的负载均衡生不生效
8. 如果你的内存不足，你可以只启动一个7001，也可以验证

##### 16. 创建microservicecloud-consumer-dept-feign-80模块

该模块继承于父模块，用于演示Feign功能

###### 修改microservicecloud-api模块

1. 修改pom文件

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-feign</artifactId>
       <version>RELEASE</version>
   </dependency>
   ```

2. 添加service包，添加DeptClientService类

   ```java
   @FeignClient(value = "MICROSERVICECLOUD-DEPT")
   public interface DeptClientService {
       @RequestMapping(value = "/dept/add", method = RequestMethod.POST)
       boolean add(@RequestBody Dept dept);
   
   
       @GetMapping(value = "/dept/get/{id}")
       Dept get(@PathVariable("id") Long id);
   
       @GetMapping(value = "/dept/list")
       List<Dept> list();
   
   }
   ```

3. 重新maven clean maven install该api模块

###### 编写microservicecloud-consumer-dept-feign-80模块pom文件

```xml
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

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-feign</artifactId>
            <version>RELEASE</version>
        </dependency>
    </dependencies>
```

###### 编写microservicecloud-consumer-dept-feign-80模块application.yml文件

```yml
server:
  port: 80

eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://localhost1:7001/eureka,http://localhost2:7002/eureka,http://localhost3:7003/eureka

```

###### 编写microservicecloud-consumer-dept-feign-80模块控制器

```java
//该类路径是cliff.ford.controller.DeptController_Consumer
@RestController
public class DeptController_Consumer {


    @Autowired
    private DeptClientService service;

    @RequestMapping(value = "/consumer/dept/add")
    public boolean add(Dept dept){
        return service.add(dept);
    }

    @RequestMapping(value = "/consumer/dept/get/{id}")
    public Dept get(@PathVariable("id") Long id){
        return service.get(id);
    }

    @RequestMapping(value = "/consumer/dept/list")
    public List<Dept> list(){
        System.out.println("方法进来了");
        return service.list();
    }
}
```

###### 编写启动类

```java
//该类路径是cliff.ford.DeptConsumer_Feign_App
@SpringBootApplication
@EnableEurekaClient
//这里的service是api模块里面的包
@EnableFeignClients(basePackages = {"service"})
//之前因为没有写cliff.ford，导致它去另一个模块扫描service包之后，本模块下面的controller包没有生效，连接访问一直404
@ComponentScan(basePackages = {"service", "cliff.ford"})
public class DeptConsumer_Feign_App {
    public static void main(String[] args) {
        System.out.println("SpringBoot生效");
        SpringApplication.run(DeptConsumer_Feign_App.class, args);
    }
}
```

###### 测试

启动7001 8001 feign-80 访问localhost/consumer/dept/list成功

##### 17. 创建microservicecloud-provider-dept-hystrix-8001模块

该模块继承于父模块，<font color=red>用于演示服务提供异常时的服务熔断机制</font>

###### 编写pom文件

```xml
<dependencies>
        <dependency>
            <groupId>cliff.ford</groupId>
            <artifactId>microservicecloud-api</artifactId>
            <version>${project.version}</version>
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
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
            <version>2.1.1.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <version>2.1.1.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
		# 重点
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
            <version>2.1.1.RELEASE</version>
        </dependency>
    </dependencies>
```

###### 编写application.yml文件

```yml
# 其他信息照抄microservicecloud-provider-dept-8001模块
eureka:
  client:
    service-url:
      defaultZone: http://localhost1:7001/eureka  # 太卡了，改单机
  instance:
    instance-id: microservicecloud-dept8001-hystrix   # 这里也改了
    prefer-ip-address: true                   
```

src/main/java下面的代码照抄microservicecloud-provider-dept-8001模块，程序入口类改名

###### 注意变更controller

```java
@GetMapping(value = "/dept/get/{id}")
//如果该方法出错，用wrongId方法去处理
@HystrixCommand(fallbackMethod = "wrongId")
public Dept get(@PathVariable("id") Long id){
    Dept dept = deptService.get(id);
    if(dept == null){
        throw new RuntimeException();
    }
    return dept;
}
//返回一个可处理的信息给上游
private Dept wrongId(@PathVariable("id") Long id){
    return new Dept().setDeptno(id).setDname("该Id:"+id+"没有对应信息--Hystrix")
        .setDb_source("no database");
}
```

###### 调整主启动类

```java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
@EnableCircuitBreaker//开启服务熔断功能
public class DeptProvider8001_Hystrix_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptProvider8001_Hystrix_App.class, args);
    }
}
```

###### 测试

启动7001 hystrix-8001 feign-80,访问localhost/consumer/dept/get/1和localhost/consumer/dept/get/99

##### 18. 修改microservicecloud-api模块和microservicecloud-consumer-dept-feign-80模块

<font color=red>用于演示服务降级</font>

###### 概念辨析

* 服务提供者利用@HystrixCommand(fallbackMethod = "wrongId")注解提供的异常处理我们叫服务熔断
* 服务消费者利用@FeignClient(value = "MICROSERVICECLOUD-DEPT", fallbackFactory = DeptClientServiceFallbackFactory.class)注解提供的异常处理我们叫服务降级

###### 服务熔断/降级的场景引入

每一个微服务下面的controller下的方法执行都有可能报错，所以需要给每一个方法都编写一个异常处理方法，这样会让controller异常臃肿，并且业务处理和异常处理耦合了，利用aop的编程思想，同时更深层次思考，如果某一个服务提供者挂掉了，服务熔断就不生效了，这时需要从客户端出发，做一个服务降级处理

###### 修改api模块下的service包

```java
//添加DeptClientServiceFallbackFactory类，该类实现FallbackFactory<T>接口，T是DeptClientService接口
@Component
public class DeptClientServiceFallbackFactory implements FallbackFactory<DeptClientService> {
    @Override
    public DeptClientService create(Throwable throwable) {
        return new DeptClientService() {
            @Override
            public boolean add(Dept dept) {
                return false;
            }

            @Override
            public Dept get(Long id) {
                return new Dept().setDeptno(id).setDname("id:"+id+" can not find any message")
                        .setDb_source("no database message");
            }

            @Override
            public List<Dept> list() {
                return null;
            }
        };
    }
}
```

重新maven clean maven install

###### 修改feign-80模块的application.yml文件

```yml
# 开启服务降级功能
feign:
  hystrix:
    enabled: true
```

###### 测试

1. 启动7001 并访问localhost1:7001
2. 启动8001 并访问localhost:8001/dept/get/1和localhost:8001/dept/get/99，这里说明服务熔断生效
3. 启动feign-80 并访问localhost/consumer/dept/get/1和localhost/consumer/dept/get/99，这里说明客户端能使服务熔断生效，观察99接口返回的内容
4. 手动停止8001 并访问localhost/consumer/dept/get/1和localhost/consumer/dept/get/99，这里说明客户端的服务降级生效，注意观察localhost/consumer/dept/get/99返回的内容与上一次的差别，说明服务熔断会先于服务降级之前被处理，这个异常的抛出顺序是相对应的

##### 19. 创建microservicecloud-consumer-hystrix-dashboard-9001模块

该模块继承于父模块，用于监控服务的监控情况

###### 编写pom文件

```xml
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
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        <version>2.1.1.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
        <version>2.1.1.RELEASE</version>
    </dependency>
</dependencies>
```

###### 编写application.yml文件

```yml
server:
  port: 9001
```

###### 编写启动类

```java
@SpringBootApplication
@EnableHystrixDashboard
public class DeptConsumer_DashBoard_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer_DashBoard_App.class, args);
    }
}
```

###### 更改microservicecloud-provider-dept-dept-hystrix-8001模块

确保pom文件添加了如下依赖

```yml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

修改application.yml文件

```yml
# 添加如下内容
management:
  endpoints:
    web:
      exposure:
        include: ["hystrix.stream"]
```

###### 启动测试

* 启动dashboard-9001并访问http://localhost:9001/hystrix
* 启动hystrix-8001并依次访问localhost:8001/actuator/hystrix.stream和localhost:8001/dept/get/1和localhost:8001/actuator/hystrix.stream，观察localhost:8001/actuator/hystrix.stream的变化
* 在http://localhost:9001/hystrix页面填入监控的url：localhost:8001/actuator/hystrix.stream，延迟时间：2000ms，监控名字title: provider-dept-8001，点击monitor stream监控
* 不断刷新localhost:8001/dept/get/1连接，继续观察监控页面

##### 创建microservicecloud-zuul-9527模块

该模块继承于父模块，用于演示路由网关功能

###### 编写pom文件

```xml
<dependencies>
    <dependency>
        <groupId>cliff.ford</groupId>
        <artifactId>microservicecloud-api</artifactId>
        <version>${project.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        <version>2.1.1.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-zuul</artifactId>
        <version>RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        <version>2.1.1.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

###### 编写applcation.yml文件

```yml
server:
  port: 9527

spring:
  application:
    name: microservice-zuul-gateway

eureka:
  client:
    service-url:
      defaultZone: http://localhost1:7001/eureka
  instance:
    instance-id: zuul-gateway
    prefer-ip-address: true

info:
  app.name: cliff.ford
  company.name: cliff.ford
  build.artifactId: ${project.artifactId}
  build.version: ${project.version}
```

###### 编写启动类

```java
@SpringBootApplication
@EnableZuulProxy
public class Zuul_9527_App {
    public static void main(String[] args) {
        SpringApplication.run(Zuul_9527_App.class, args);
    }
}
```

###### 测试

启动eureka-7001, dept_hystrix_8001, zuul_9527，并访问localhost:8001/dept/get/1和<font color=red>localhost:9527</font>/<font color=blue>microservicecloud-dept</font>/dept/get/2，后面一个连接红色部分是网关，蓝色部分是服务名，黑色部分是接口地址

###### 变更需求

令真正的服务接口对外不可用，只保留zuul提供的对外访问地址，并且对外访问的地址要屏蔽真正的服务名，此时需要修改zuul_9527模块的application.yml文件

```yml
zuul:
  prefix: cliff.ford   # 服务名前的前缀
  ignored-services: microservicecloud-dept  # 让所有服务名对外不可用，可以用"*"
  routes:
    mydept.serviceId: microservicecloud-dept
    mydept.path: /mydept/**
```

现在<font color=red>localhost:9527</font>/<font color=blue>microservicecloud-dept</font>/dept/get/2已经不可用了，要这样访问<font color=red>localhost:9527</font>/cliff.ford/<font color=blue>mydept</font>/dept/get/2才行

##### 20. 创建microservicecloud-config-server-3344模块

该模块继承于父模块，用于演示config server从github上面读取配置资源信息

###### 编写pom文件

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
        <version>2.1.1.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        <version>2.1.1.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

###### 编写application.yml文件

```yml
server:
  port: 3344

spring:
  application:
    name: microservicecloud-config-server
  cloud:
    config:
      server:
        git:
          uri: https://github.com/Cliff-Ford/microservicecloud-config.git //配置文件所在的仓库
```

###### 编写启动类

```java
@SpringBootApplication
@EnableConfigServer
public class Config_3344_App {
    public static void main(String[] args) {
        SpringApplication.run(Config_3344_App.class, args);
    }
}
```

###### 测试

注意application.yml文件要自己编写和推送上去

启动并访问localhost:3344/application-dev.yml和localhost:3344/application-test.yml和localhost:3344/application.yml

##### 21. 创建microservicecloud-config-client-3355模块

该模块继承于父模块，用于演示配置中心客户端远程读取资源的案例

###### 编写pom文件

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-client</artifactId>
        <version>RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-netflix-eureka-client</artifactId>
        <version>2.1.1.RELEASE</version>
    </dependency>
</dependencies>
```

编写bootstrap.yml文件

```yml
spring:
  cloud:
    config:
      name: microservicecloud-config-client # 配置从githut上读取的资源名称，注意没有yml后缀
      profile: dev  # 本次的配置项
      label: master
      uri: http://localhost:3344  # 本次服务启动后先去找3344号服务，通过SpringCloudConfig获取GitHub的服务地址
```

###### 编写访问控制器

```java
@RestController
public class ConfigClientRest {
    @Value("${spring.application.name}")
    private String applicationName;

    @Value("${eureka.client.service-url.defaultZone}")
    private String eurekaServer;

    @Value("${server.port}")
    private String port;

    @GetMapping(value = "/config")
    public String getConfig(){
        return toString();
    }

    @Override
    public String toString() {
        return "ConfigClientRest{" +
                "applicationName='" + applicationName + '\'' +
                ", eurekaServer='" + eurekaServer + '\'' +
                ", port='" + port + '\'' +
                '}';
    }
}
```

###### 编写启动类

```java
@SpringBootApplication
public class Config_3355_App {
    public static void main(String[] args) {
        SpringApplication.run(Config_3355_App.class, args);
    }
}
```

###### 测试

启动3344和3355，因为3355说明了从哪个分支下读取哪个环境下的资源文件

访问localhost:8201/config和localhost:8202/config，对比思考

##### 22. 整合Config Server/Client进行案例讲解

这里应该就是启动3344，然后给8001添加Config Client的依赖，将application.yml的内容清空或者注释，配置bootstrap.yml去远端拉取

最终所有的配置都在远端（GitHub），eureka集群启动，服务提供者集群开启（集成了服务熔断），服务消费者集群开启（集成了服务降级），网关zuul开启，监控dashboard开启