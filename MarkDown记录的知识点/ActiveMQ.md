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

##### JMS可靠性

###### JMS的持久性（PERSISTENT）

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





















































































