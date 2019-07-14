<center><h3>Kafka知识</h3></center>

##### Kafka概述

###### 消息队列内部实现原理

* 利用一个队列结构实现消息的先进先出
* 利用监听的方式实现点对点消息传输
* 利用发布/订阅模式（观察者模式）实现一对多消息传输

###### 为什么要用消息队列

* 解耦（两个客户端的通讯由同步变成了异步，不再相互依赖）
* 冗余（数据备份）
* 扩展性（集群）
* 灵活性&峰值处理能力
* 可恢复性（数据备份）
* 顺序保证（队列）
* 缓冲（通讯两方的速度不一致）
* 异步通讯（通讯中的乙方挂掉也没事）

###### 什么是Kafka

在流式计算中，Kafka一般用来缓存数据，Storm通过消费Kafka的数据进行计算

1. Apache Kafka是一个开源消息系统，由Scala写成
2. Kafka是一个分布式消息队列。Kafka对消息保存时根据Topic进行归类，发送消息者称为Producer，消息接收者称为Consumer，此外Kafka集群由多个kafka实例组成，每个实例（server)称为broker
3. 无论kafka集群，还是consumer都依赖于zookeeper集群保存一些meta信息来保证系统的可用性

##### Kafka单机模式部署

1. 正常下载解压kafka，新版的kafka内嵌了zookeeper

2. 修改config/zookeeper.properties文件

   ```properties
   dataDir=D:/kafka_2.11-1.1.0/kafka_2.11-1.1.0-zookeeper-data
   ```

3. 在安装目录下编写创建zkServer-start.cmd脚本

   ```cmd
   cd D:\kafka_2.11-1.1.0
   start /b .\bin\windows\zookeeper-server-start.bat .\config\zookeeper.properties
   ```

   启动zkServer

4. 修改config/server.properties文件

   ```properties
   log.dirs=D:/kafka_2.11-1.1.0/kafka_2.11-1.1.0-logs
   listeners=PLAINTEXT://localhost:9092
   ```

5. 编写kafka-server-start.cmd启动脚本

   ```cmd
   cd D:\kafka_2.11-1.1.0
   start .\bin\windows\kafka-server-start.bat .\config\server.properties
   ```

   启动kafka server

6. 编写kafka-topic.cmd脚本创建test队列

   ```cmd
   cd D:\kafka_2.11-1.1.0\bin\windows
   start /b kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
   ```

7. 编写kafka-producer-start.cmd脚本创建test队列生产者

   ```cmd
   cd D:\kafka_2.11-1.1.0\bin\windows
   start /b kafka-console-producer.bat --broker-list localhost:9092 --topic test 
   ```

8. 编写kafka-consumer-start.cmd脚本创建test队列消费者

   ```cmd
   cd D:\kafka_2.11-1.1.0\bin\windows
   start /b kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test --from-beginning
   ```

9. 生产者端发送消息，消费者端接收消息，中文消息会乱码

##### <font color=red>Kafka集群部署</font>

在单机模式配置成功的情况下，继续修改config/server.properties文件如下

```properties
# 注意逗号后面没有空格
zookeeper.connect=localhost:2181,localhost:2182,localhost:2183
```





##### Kafka工作流程分析

![kafka集群工作流程](D:\02_个人文档\照片\kafka工作流程.png)

1. 首先启动zookeeper或者zookeeper集群

2. 接着启动kafka集群

3. 创建topic A 命令kafka-topic.bat --create --zookeeper localhost:2181 --replication-factor 2 --partition 2 --topic A 

   --replication-factor 2 是副本数，副本数不能大于broker数，一般是等于

   --partition 2 是分区数，分区会采用均衡分区的算法

4. 生产者A kafka-console-producer.bat --broker-list localhost:9092 --topic A

5. 消费者A kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic A [--from-beginning]

###### 分区

消息发送时都被发送到一个topic，本质就是一个目录，而topic是由一些Partition Logs(分区日志)组成，其组织结构如下

![img](https://ask.qcloudimg.com/http-save/1082016/znp42zwm5x.png?imageView2/2/w/1620)

###### 分区的原因

1. 方便在集群中扩展，每个Partition可以通过调整以适应它所在的机器，而一个topic又可以有多个Partition组成，因此整个集群就可以适应任意大小的数据了
2. 可以提高并发，因为可以以Partition为单位读写了

###### 分区原则

1. 指定了partition，直接使用
2. 未指定partition但指定了key，通过key的value进行hash出一个partition
3. partition和key都未指定，使用轮询选出一个partition

###### 副本（Replication）

同一个partition可能会有多个replication（对应server.properties配置中的default.replication.factor=N）。没有replication的情况下，一旦broker宕机，其上所有partition的数据都不可被消费，同时producer也不能再将数据存于其它的patition，引入replication之后，同一个partition可能会有多个replication,而这时需要再这些replication之间选出一个leader，producer和consumer只与这个leader交互，其他replication作为follower从leader中复制数据

###### 写入流程

![写入流程](D:\02_个人文档\照片\kafka生产者写入数据流程.png)

第一步的意思是生产者先从zookeeper那里获取目标分区的leader地址，然后第二步直接去给目标分区的leader发消息，发消息之后有三个确认级别

* 0级，只发，不管其他
* 1级，发给leader后，确认leader写入完毕，给生产者ack反馈，不管其他
* 2级，发给leader后，等leader和follower都写入完毕，再给生产者ack反馈

##### <font color=red>Kafka API实战</font>





##### Kafka producer拦截器





##### Kafka Streams







##### 扩展