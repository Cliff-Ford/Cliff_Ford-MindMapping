<center><h3>ActiveMQ</h3></center>

##### MQ

MQ (Message Queue)

##### MQ产品分类

* ActiveMQ
* RabbitMQ
* RocketMQ
* ActiveMQ
  * api发送和接收
  * MQ的高可用性
  * MQ的集群和容错配置
  * MQ的持久化
  * 延时发送/定时投递
  * 签收机制
  * spring整合

##### 为什么使用MQ

在学生请教老师问题的场景下，每个学生请教老师问题大概要5分钟，在没有MQ框架之前，大家排队提问，老师依次回答，越后面的学生需要等待的时间越久。（学生系统和老师系统紧耦合，在高并发场景下，老师系统超负荷运转）

学生系统和老师系统的紧耦合理解为学生系统中远程调用老师系统中解决问题的方法。

在使用MQ时，学生系统将按照要求发一条记录了问题的消息给中间件（比如是班长），老师系统去班长那里消费消息，给学生系统做出应答，这样，<font color=red>同步调用变成了异步调用，实现了系统解耦，和流量削峰</font>

##### 秒杀业务逻辑

读取订单->库存检查->库存冻结->余额检查->余额冻结->订单生成->余额扣减->库存扣减->生成流水->余额解冻->库存解冻

##### Linux上面查看某个端口是否被占用

netstat -anp|grep 61616

##### ActiveMQ重要命令

启动1：先进入/activeMQ安装目录/bin，然后activemq start

启动2：先进入/activeMQ安装目录/bin，然后activemq start > /activemq.log 这是带日志的启动方式

##### docker安装和启动activemq

* docker search activemq
* docker pull activemq
* docker run -d --name activemq -p 61616:61616 -p 8161:8161 activemq
* 浏览器打开localhost:8161即可看到activemq的管理页面，账号密码默认admin/admin

##### MQ中的重要概念

* 队列：在点对点的消息传递域中，目的地被称为队列（queue）
* 主题：在发布订阅消息传递域中，目的地被称为主题（topic）

| Number Of Pending Message | 待消费的消息           |
| ------------------------- | ---------------------- |
| Number Of Consumers       | 消费者数量             |
| Message Enqueue           | 进入队列的历史消息总数 |
| Message Dequeued          | 已出队的消息           |

##### MQ生产消息

```java
public class JmsProduce {
    public static final String ACTIVEMQ_URL = "tcp://203.195.177.110:61616";
    public static final String QUEUE_NAME = "queue_01";

    public static void main(String[] args) throws JMSException {
        // 1 创建连接工厂，按照给定的url地址，采用默认用户名和密码
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        // 2 通过连接工厂，获得连接connection并启动访问
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        // 3 创建会话session,两个参数，第一个叫事务，第二个叫签收
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 4 创建目的地（队列或者主题）
        Queue queue = session.createQueue(QUEUE_NAME);
        // 5 创建消息的生产者
        MessageProducer messageProducer = session.createProducer(queue);
        // 6 通过messageProducer生产3条消息到MQ队列中
        for (int i = 1; i <= 3; i++){
            // 7 创建消息
            TextMessage textMessage = session.createTextMessage("msg---" + i);
            // 8 发送消息
            messageProducer.send(textMessage);
        }
        // 9 关闭连接
        messageProducer.close();
        session.close();
        connection.close();
        System.out.println("消息发布到MQ完成");
    }
}
```

##### MQ消费消息(receive方法)

```java
public class JmsConsumer {
    public static final String ACTIVEMQ_URL = "tcp://203.195.177.110:61616";
    public static final String QUEUE_NAME = "queue_01";

    public static void main(String[] args) throws JMSException {
        // 1 创建连接工厂，按照给定的url地址，采用默认用户名和密码
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        // 2 通过连接工厂，获得连接connection并启动访问
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        // 3 创建会话session,两个参数，第一个叫事务，第二个叫签收
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 4 创建目的地（队列或者主题）
        Queue queue = session.createQueue(QUEUE_NAME);
        // 5 创建消息的消费者
        MessageConsumer consumer = session.createConsumer(queue);
        // 6 循环消费所有消息
        while (true){
            //这种写法的消费者会一直等待
            //TextMessage message = (TextMessage)consumer.receive();
            //4s之后没有获取到消息，消费者退出
            TextMessage message = (TextMessage)consumer.receive(4000L);
            if(message != null){
                System.out.println("消费消息--"+message.getText());
            }else{
                break;
            }
        }
        consumer.close();
        session.close();
        connection.close();
        System.out.println("消息全部消费完毕");
    }
}
```

<font color=red>consumer.receive()方法是同步阻塞方式，在能够接收到消息之前将一直阻塞</font>

##### MQ消费消息(监听方式)

```java
public class JmsConsumer2 {
    public static final String ACTIVEMQ_URL = "tcp://203.195.177.110:61616";
    public static final String QUEUE_NAME = "queue_01";

    public static void main(String[] args) throws JMSException, IOException {
        // 1 创建连接工厂，按照给定的url地址，采用默认用户名和密码
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        // 2 通过连接工厂，获得连接connection并启动访问
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        // 3 创建会话session,两个参数，第一个叫事务，第二个叫签收
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 4 创建目的地（队列或者主题）
        Queue queue = session.createQueue(QUEUE_NAME);
        // 5 创建消息的消费者
        MessageConsumer consumer = session.createConsumer(queue);
        // 6 消费者设置监听器
        consumer.setMessageListener(message -> {
            if(message != null && message instanceof TextMessage){
                TextMessage textMessage = (TextMessage)message;
                try {
                    System.out.println("消费者监听并消费消息--"+textMessage.getText());
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        });
        //一定要写
        System.in.read();
        consumer.close();
        session.close();
        connection.close();
        System.out.println("消息全部消费完毕");
    }
}
```

<font color=red>监听方式时异步非阻塞方式</font>

##### MQ<font color=red>队列</font>中多个消费者时的消费情况

* 先启动生产者，再启动一号消费者，消息将全部由一号消费之消费
* 先启动生产者，再启动一号消费者，然后再启动二号消费者，如果启动二号消费者完成时，生产的消息尚未被一号消费者消费完毕，后面将会被一号和二号消费者同时轮询消费（负载均衡）
* 先启动两个消费者，再启动生产者，生产出来的消息将会被一号和二号消费者同时轮询消费（负载均衡）

##### MQ<font color=red>主题</font>中多个订阅者时的订阅情况

* 在主题中，订阅者要先出现，再出现发布者，否则发布者发布的消息是废消息
* 多个订阅者可以接收自订阅后发布者发布的所有消息

##### JMS开发步骤

1. 创建一个connection factory
2. 通过connection  factory来创建JMS connection
3. 启动JMS connection
4. 通过connection创建session
5. 通过session创建destination(queue,topic)
6. 通过session创建JMS producer或者创建JMS message并设置destination
7. 通过session创建JMS consumer或者注册一个JMS message listener
8. 发送或者接收JMS message
9. 关闭连接

##### Topic和Queue的对比

| 比较项目   | Topic模式队列                                                | Queue模式队列                                                |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 工作模式   | “订阅-发布”模式，如果当前没有订阅者，消息将会被丢弃。如果由多个订阅者，这些订阅者都会收到消息 | “负载均衡”模式，如果当前没有消费者，消息不会丢弃，如果有多个消费者，那么一条消费只会发送给其中一个消费者，并且要求消费者ack消息 |
| 有无状态   | 无状态                                                       | Queue数据默认会再mq服务器上以文件形式保存，比如Active MQ一般保存再￥AMQ_HOME\data\kr-store\data下面，也可以配置成DB存储 |
| 传递完整性 | 如果没有订阅者，消息会被丢弃                                 | 消息不会丢弃                                                 |
| 处理效率   | 由于消息要按照订阅者的数量进行复制，所以处理性能会随着订阅者的增加而明显降低，并且还要结合不同消息协议自身的性能差异 | 由于一条消息只发送给一个消费者，所以就算消费者再多，性能也不会有明显降低。当然不同消息协议的具体性能也有差异 |

##### JMS四大组件

* provider:实现JMS接口和规范的消息中间件，也就是MQ服务

* producer:消息生产者，创建和发送JMS消息的客户端应用

* consumer:消息消费者，接收和处理JMS消息的客户端应用

* message:消息

  * 消息头
    * JMSDestination(消息发送的目的地，主要是Queue和Topic)
    * JMSDeliveryMode(持久和非持久模式)
    * JMSExpiration(消息的过期时间)
    * JMSPriority(消息优先级)
    * JMSMessageID(消息ID)
  * 消息体
    * TextMessage(普通字符串消息，包含一个string)
    * MapMessage(一个Map类型的消息，key为string类型，而值为java的基本类型)
    * BytesMessage(二进制数组消息，包含一个byte[])
    * StreamMessage(Java数据流消息，用标准流操作来顺序的填充和读取)
    * ObjectMessage(对象消息，包含一个可序列化的Java对象)
  * 消息属性(消息头和消息体的扩展)

  复杂的消息案例如下

  ```java
  // 7 创建一个text消息体
  TextMessage textMessage = session.createTextMessage("msg---" + i);
  //设置消息头
  textMessage.setJMSPriority(9);
  //设置消息属性
  textMessage.setStringProperty("c01","vip");
  // 8 发送消息
  messageProducer.send(textMessage);
  ```

  ```java
  //获取消息内容
  System.out.println("消费者监听并消费消息--"+textMessage.getText());
  System.out.println(textMessage.getStringProperty("c01"));//vip
  System.out.println(textMessage.getJMSPriority());//9
  ```

##### JMS可靠性消息

###### JMS的持久（PERSISTENT）

消息头可以设置消息的持久化模式，队列的消息默认是持久化的

```java
//该模式下activemq宕机重启时消息消失
textMessage.setJMSDeliveryMode(DeliveryMode.NON_PERSISTENT);
//该模式下activemq宕机重启时消息恢复
textMessage.setJMSDeliveryMode(DeliveryMode.PERSISTENT);
```

Topic添加订阅者之后，等于向MQ注册，之后的持久化主题无论订阅者是否在线，订阅者都可以接收，离线重新上线时会接收到之前还没有接受到的信息

topic持久化案例如下

首先设置一个主题，并让该主题生产的消息是持久化的，注意下面几句话的顺序

```java
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        Connection connection = activeMQConnectionFactory.createConnection();
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        Topic topic = session.createTopic(TOPIC_NAME);
        MessageProducer messageProducer = session.createProducer(topic);
        // 5.1 设置生成的主题消息是持久化的
        messageProducer.setDeliveryMode(DeliveryMode.PERSISTENT);
//这时候才start
        connection.start();
```

接着创建该主题的持久化订阅者

```java
		ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        Connection connection = activeMQConnectionFactory.createConnection();
//向MQ注册
        connection.setClientID("3");
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        Topic topic = session.createTopic(TOPIC_NAME);
        //创建订阅者
        TopicSubscriber topicSubscriber = session.createDurableSubscriber(topic, "remark....");
//这时候才start
        connection.start();
```

###### JMS事务

再connection创建session的时候，通过boolean值决定是否开启事务

```java
connection.createSession(false/true, Session.AUTO_ACKNOWLEDGE);
```

<font color=red>在没有开启事务的时候，生产者生产的消息将直接进入队列</font>

<font color=red>在开启事务之后，生产者生产的消息在session.commit()方法之后才进入队列，同样的，消费者也需要session.commit()，消息才会从队列中出去，避免消息被重复消费</font>

###### JMS签收（Acknowledge）

- 自动签收（Session.AUTO_ACKNOWLEDGE）
- 手动签收（Session.CLIENT_ACKNOWLEDGE）
- 允许重复消息（Session.DUPS_OK_ACKNOWLEDGE）(通常不同)

###### <font color=red>非事务情况下的签收</font>

* 自动签收，正常跑就行
* 手动签收，需要给消费的消息手动签收，,textMessage.acknowledge()方法手动签收，不然该消息会被重复消费

###### <font color=red>事务情况下的签收</font>

* 自动签收，需要session.commit()
* 手动签收，有session.commit()的情况下，message.acknowledge()不重要，没有session.commit()的情况下，即使message.acknowledge()也不生效

###### <font color=red>结论</font>

* 在事务性会话中，当一个事务被成功提交则消息被自动签收
* 非事务性会话中，消息何时被确认取决于创建会话时的应答模式（Session.AUTO_ACKNOWLEDGE|SESSION_CLIENT_ACKNOWLEDGE)等

##### Spring整合ActiveMQ

###### pom.xml文件整合依赖

```xml
<dependencies>
    <!-- Spring对JMS的支持 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jms</artifactId>
        <version>4.0.0.RELEASE</version>
    </dependency>
    <!-- 添加ActiveMQ的pool包 -->
    <dependency>
        <groupId>org.apache.activemq</groupId>
        <artifactId>activemq-pool</artifactId>
        <version>5.9.0</version>
    </dependency>
</dependencies>
```

###### applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/aop
            http://www.springframework.org/schema/aop/spring-aop.xsd">
    <!--  开启包的自动扫描  -->
    <context:component-scan base-package="com.atguigu.activemq"/>
    <!--    配置连接工厂池，之后会自动创建一个session放在jmsTemplate里面-->
    <bean id="jmsFactory" class="org.apache.activemq.pool.PooledConnectionFactory" destroy-method="stop">
        <!--连接工厂-->
        <property name="connectionFactory">
            <bean class="org.apache.activemq.ActiveMQConnectionFactory">
                <!--目标地址-->
                <property name="brokerURL" value="tcp://203.195.177.110:61616"/>
            </bean>
        </property>
    </bean>

    <!--创建队列-->
    <bean id="destinationQueue" class="org.apache.activemq.command.ActiveMQQueue">
        <constructor-arg index="0" value="spring-active-queue"/>
    </bean>

    <!--创建主题-->
    <bean id="destinationTopic" class="org.apache.activemq.command.ActiveMQTopic">
        <constructor-arg index="0" value="spring-active-topic"/>
    </bean>

    <!--    spring提供的JMS工具类，它可以进行消息发送，接收等-->
    <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
        <property name="connectionFactory" ref="jmsFactory"/>
        <!--该ref指向谁，jmsTemplate就针对哪个目的地操作 -->
        <property name="defaultDestination" ref="destinationTopic"/>
        <property name="messageConverter">
            <bean class="org.springframework.jms.support.converter.SimpleMessageConverter"/>
        </property>
    </bean>
</beans>
```

###### 编写生产者生产消息

```java
@Service
public class SpringMQ_Produce {
    @Autowired
    private JmsTemplate jmsTemplate;

    public static void main(String[] args) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        SpringMQ_Produce produce = (SpringMQ_Produce) ctx.getBean("springMQ_Produce");
        //通过传入MessageCreator匿名内部类来生产消息
        produce.jmsTemplate.send(new MessageCreator() {
            public Message createMessage(Session session) throws JMSException {
                return session.createTextMessage("spring activemq message 3333");
            }
        });
    }
}
```

###### 编写消费者消费消息

```java
@Service
public class SpringMQ_Consumer {

    @Autowired
    private JmsTemplate jmsTemplate;

    public static void main(String[] args) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        SpringMQ_Consumer consumer = (SpringMQ_Consumer) ctx.getBean("springMQ_Consumer");
        //接收并转型
        String text = (String)consumer.jmsTemplate.receiveAndConvert();
        System.out.println(text);
    }
}
```

###### 设置消息监听者，生产者生产的消息将会被自动消费

1. 编写监听类

   ```java
   public class MyMessageListener implements MessageListener {
       public void onMessage(Message message) {
           if(message != null && message instanceof TextMessage){
               TextMessage textMessage = (TextMessage)message;
               try {
                   System.out.println(textMessage.getText());
               } catch (JMSException e) {
                   e.printStackTrace();
               }
           }
       }
   }
   ```

2. 将监听类注册到bean容器中

   ```xml
   <bean id="myMessageListener" class="com.atguigu.activemq.support.MyMessageListener"/>
   ```

3. 开启自动监听容器

   ```xml
   <bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
           <property name="connectionFactory" ref="jmsFactory"/>
           <property name="destination" ref="destinationTopic"/>
           <property name="messageListener" ref="myMessageListener"/>
       </bean>
   ```

4. 编写生产者生产消息，此时消息会被自动消费

   ```java
   @Service
   public class SpringMQ_Produce {
       @Autowired
       private JmsTemplate jmsTemplate;
   
       public static void main(String[] args) {
           ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
           SpringMQ_Produce produce = (SpringMQ_Produce) ctx.getBean("springMQ_Produce");
           produce.jmsTemplate.send(new MessageCreator() {
               public Message createMessage(Session session) throws JMSException {
                   return session.createTextMessage("spring activemq message 3333");
               }
           });
       }
   }
   ```


##### SpringBoot整合ActiveMQ

###### pom.xml文件整合依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-activemq</artifactId>
</dependency>
```

###### 配置application.yml文件

```yml
spring:
  activemq:
    broker-url: tcp://203.195.177.110:61616 # MQ地址
    user: admin # 用户名
    password: admin # 密码
  jms:
    pub-sub-domain: true    # false = Queue true = Topic 注意测试队列时改成false,同时注意将主题相关的生产者和消费者组件从容器中排除（通过注释@Conmponent的方法）

# 自定义了对列名
myqueue: spring-boot-activemq-queue
# 自定义了主题名
mytopic: spring-boot-activemq-topic
```

###### 编写配置类，配置一个队列和一个主题

```java
@Component
public class ConfigBean {
    //将application.yml里面的myqueue属性值注入该field
    @Value("${myqueue}")
    private String myQueue;

    @Value("${mytopic}")
    private String myTopic;

    //创建队列
    @Bean
    public Queue queue(){
        return new ActiveMQQueue(myQueue);
    }

    //创建主题
    @Bean
    public Topic topic(){
        return new ActiveMQTopic(myTopic);
    }
}

```

###### 编写队列生产者和主题生产者

```java
@Component
@EnableJms
public class Queue_Produce {
    @Autowired
    private JmsMessagingTemplate jmsMessagingTemplate;

    @Autowired
    private Queue queue;

    public void produceMsg(){
        jmsMessagingTemplate.convertAndSend(queue, UUID.randomUUID().toString());
    }

    //间隔三秒定投
    @Scheduled(fixedDelay = 3000)
    public void produceMsgScheduled(){
        jmsMessagingTemplate.convertAndSend(queue, "Scheduled : "+UUID.randomUUID().toString());

    }
}
```

```java
@Component
@EnableJms
public class Topic_Produce {
    @Autowired
    private JmsMessagingTemplate jmsMessagingTemplate;

    @Autowired
    private Topic topic;

    @Scheduled(fixedDelay = 3000)
    public void produceMsg(){
        jmsMessagingTemplate.convertAndSend(topic, UUID.randomUUID().toString());
    }

}
```

###### 编写队列消费者和主题消费者

```java
//@Component
public class Queue_Consumer {

    @JmsListener(destination = "${myqueue}")
    public void receive(TextMessage textMessage){
        System.out.println(textMessage);
    }
}
```

```java
@Component
public class Topic_Consumer {

    @JmsListener(destination = "${mytopic}")
    public void receive(TextMessage textMessage){
        System.out.println(textMessage);
    }
}
```

###### 测试

```java
@SpringBootApplication
//注意需要开启定时开关，定时任务才可以执行
@EnableScheduling
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}

}
```

##### ActiveMQ传输协议

ActiveMQ支持的client-broker通讯协议有<font color=red>TCP</font>、<font color=red>NIO</font>、UDP、SSL、HTTP(S)、VM

如果你要更改传输协议，需要到安装目录下的conf/activemq.xml中的<font color=red>transportConnectors</font>标签中改

TCP传输的优点

* TCP协议传输可靠性高，稳定性强
* 高效性：字节流方式传递，效率很高
* 有效性、可用性：应用广泛，支持任何平台

NIO的应用场景和优点

* 可能有大量的client去连接到Broker上，一般情况下，大量的Client去连接Broker是被操作系统的线程所限制的，因此 ，NIO的实现比TCP需要更少的线程去运行，所以建议使用NIO协议
* 可能对于Broker有一个很迟钝的网络传输，NIO比TCP提供更好的性能

| 协议    | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| TCP     | 默认，性能相对可以                                           |
| NIO     | 基于TCP协议之上的，进行了扩展和优化，具有更好的扩展性        |
| UDP     | 性能比TCP更好，但是不具备可靠性                              |
| SSL     | 安全连接                                                     |
| HTTP(S) | 基于HTTP或者HTTPS                                            |
| VM      | VM本身不是协议，当客户端和代理在同一个Java虚拟机（VM）中运行时，他们之间需要通信，但不想占用网络通道，而是直接通信，可以使用该方式 |

##### 如何将ActiveMQ默认的tcp协议更改为NIO协议？

如果你要更改传输协议，需要到安装目录下的conf/activemq.xml中的<font color=red>transportConnectors</font>标签中改

添加以下一句话

```xml
<transportConnector name="nio" uri="nio://0.0.0.0:61618?trace=true" />
```

<font color=red>tcp协议下的代码转换为nio协议时，代码基本不需要变动，只需要说明协议类型和端口，但如果改为其他协议，代码会有不同程度的变化</font>

如何让nio支持更多的协议呢？试一下auto+nio://0.0.0.0:61608?xxxxx

##### ActiveMQ消息的存储与持久化

KahaDB是目前默认的存储方式，消息存储使用一个事务日志和仅仅用一个索引文件来存储它所有的地址

KahaDB只有四类文件和一个lock

* db-number.log日志记录文件
* db.data该文件包含了持久化的BTree索引，索引了消息数据记录中的消息，他是消息的索引文件，本质上是B-Tree(B树)，使用B-Tree作为索引指向db-number.log里面存储的消息
* db.free当前db.data文件里哪些页面是空闲的，文件具体内容是所有空闲页的ID
* db.redo用来进行消息回复，如果KahaDB消息存储在强制退出后启动，用于恢复BTree索引
* lock文件锁，表示当前获得KahaDB读写权限的broker

##### 利用JDBC实现消息的持久化

1. 将JDBC驱动jar包和数据连接池jar包（默认放了dbcp2连接池）放进ActiveMQ安装目录下的lib文件中

2. 打开ActiveMQ安装目录下conf/activemq.xml配置文件，做出如下修改

   ```hxml
   <!--
       <persistenceAdapter>
           <kahaDB directory="${activemq.data}/kahadb"/>
       </persistenceAdapter>
   -->
   
   <persistenceAdapter>
   	<jdbcPersistenceAdapter dataSource="#mysql-ds" createTablesOnStartup="true"/>
   </persistenceAdapter>
   ```

3. 在activemq.xml配置文件broker结束标签后面，import标签之前添加#mysql-ds的bean

   ```xml
   <bean id="mysql-ds" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close"> 
       <property name="driverClassName" value="com.mysql.jdbc.Driver"/> 
       <property name="url" value="jdbc:mysql://localhost/activemq?relaxAutoCommit=true"/> 
       <property name="username" value="activemq"/> 
       <property name="password" value="activemq"/> 
       <property name="poolPreparedStatements" value="true"/> 
     </bean> 
   ```

4. 在数据库目标主机建立数据库

   ```sql
   create database activemq;
   ```

5. 重启activemq服务，目标数据库将自动创建三张表

   * ACTIVEMQ_MSGS（存储消息）
     * ID：自增数据库主键
     * CONTAINER：消息的Destination
     * MSGID_PROD：消息发送者的主键
     * MSG_SEQ：发送消息的顺序，MSGID_PROD+MSG_SEQ可以组成JMS的MessageID
     * EXPIRATION：消息的过期时间，存储的是从1970-1-1到现在的毫秒数
     * MSG：消息本体的Java序列化对象的二进制数据
     * PRIORITY：优先级，0-9
   * ACTIVEMQ_ACKS（存储订阅关系）
     * CONTAINER：消息的Destination
     * SUB_DEST：如果使用Static集群，这个字段会有集群其他系统的消息
     * ClIENT_ID：每个订阅者必须有一个唯一的客户端ID
     * SUB_NAME：订阅者名
     * SELECTOR：选择器，可以选择只消费满足条件的消息。条件可以自定义属性实现，可支持多属性AND和OR操作
     * LAST_ACKED_ID：记录消费过的消息的ID
   * ACTIVEMQ_LOCK（记录拥有读写权限的broker，用于集群的场景）
     * ID
     * BROKER_NAME

6. 代码验证，注意设置消息为持久模式，然后观察生产消息后数据库表的变化，消费消息后数据库的变化

##### 增强JDBC实现消息的持久化

这是一个告诉缓冲层，挡在数据库前面，一段时间后尚未被消费的消息才会进入数据库

```xml
<persistenceFactory>
	<journalPersistenceAdapterFactory>
		....
	</journalPersistenceAdapterFactory>
</persistenceFactory>
```

##### ActiveMQ的高可用（集群）

* 基于sharedFileSystem共享文件系统（KahaDB默认）
* 基于JDBC
* 基于可复制的LevelDB

































































