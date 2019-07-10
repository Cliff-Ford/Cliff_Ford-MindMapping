<center><h3>ZooKeeper</h3></center>

##### Zookeeper入门

###### 概述

Zookeeper是一个开源为分布式应用提供协调服务的Apache项目

###### 工作机制

Zookeeper从设计模式角度来理解：是一个基于观察者模式设计的分布式服务管理框架，它负责存储和管理大家都关心的数据，然后接受观察者的注册，一旦这些数据的状态发生变化，Zookeeper将负责同志已经在Zookeeper上注册的哪些观察者做出相应的反应

Zookeeper=文件系统+通知机制

###### Zookeeper特点

![Zookeeper特点](D:\02_个人文档\照片\Zookeeper特点.PNG)

* Zookeeper：一个领导者（Leader），多个跟随者（Follower）组成的集群
* 集群中只要有半数以上节点存活，Zookeeper集群就能正常服务
* 全局数据一致：每个Server保存一份相同的数据副本，Client无论连接到哪个Server，数据都是一致的
* 更新请求顺序进行，来自同一个Client的更新请求按期发送顺序依次执行
* 数据更新原子性，一次数据更新要么成，要么失败
* 实时性，在一定时间范围内，Client能都读到最新数据

###### Zookeeper数据结构

![Zookeeper数据结构](D:\02_个人文档\照片\Zookeeper数据结构.PNG)

Zookeeper数据模型的结构与Unix文件系统很类似，整体上可以看作是一棵树，每个节点称作一个ZNode。每个ZNode默认能够存储1MB的数据，每个ZNode都可以通过其路径唯一标识

###### Zookeeper应用场景

* 统一命名服务

  在分布式环境下，经常需要对应用/服务进行统一命名，便于识别，例如IP不容易记住，而域名容易记住

* 统一配置管理

  * 分布式环境下，配置文件同步非常常见
    * 一般要求一个集群中，所有节点的配置信息是一致的，比如Kafka集群
    * 对配置文件修改后，希望能够快速同步到各个节点上
  * 配置管理可交由ZooKeeper实现
    * 可将配置信息写入Zookeeper上的一个Znode
    * 各个客户端服务器监听这个Znode
    * 一旦Znode中的数据被修改，ZooKeeper将通知各个客户端服务器

* 统一集群管理

  * 分布式环境中，实时掌握每个节点的状态是必要的，可根据节点实时状态做出一些调整
  * Zookeeper可以实现实时监控节点状态变化
    * 可将节点信息写入一个Zookeeper上的一个ZNode
    * 监听这个ZNode可获取它的实时状态变化

* 服务器节点动态上下线

  * 突然关掉的服务器，客户端能马上知道，之后不访问
  * 重新上线的服务器，客户端能马上知道，按照规则访问

* 软负载均衡

  在Zookeeper中记录每台服务器的访问数，让访问数最少的服务器去处理最新的客户端请求

##### Zookeeper安装及参数简介

* window
  1. https://archive.apache.org/dist/zookeeper/下载zookeeper-3.4.5
  2. 解压到D盘
  3. 将D:\zookeeper-3.4.5\conf\zoo_sample.cfg在当前目录下复制一份并改名为zoo.cfg
  4. 打开zoo.cfg修改dataDir=D:\\zookeeper-3.4.5-data,该文件用于存放zookeeper快照
  5. 进入D:\zookeeper-3.4.5\bin启动zkServer.cmd启动zookeeper server
  6. 进入D:\zookeeper-3.4.5\bin启动zkCli.cmd启动zookeeper client
  7. 观察D:\zookeeper-3.4.5-data的变化
* zoo.cfg配置文件参数讲解
  1. tickTime=2000--->心跳间隔为2s
  2. initLimit=10--->Leader和Follower之间的启动<font color=red>前</font>最大间隔通讯帧，一帧指一个心跳
  3. SyncLimit=5--->Leader和Follower之间的启动<font color=red>后</font>最大间隔通讯帧，一帧指一个心跳
  4. dataDir=D:\\zookeeper-3.4.5-data--->快照存放目录
  5. ClientPort=2181--->客户端访问端口

##### Zookeeper内部原理

###### 选举机制

1. 半数机制：集群中半数以上机器存活，集群可用。所以Zookeeper适合安装奇数台服务器
2. Zookeeper虽然在配置文件中并没有指定Master和Slave。但是Zookeeper工作时是有一个节点作为Leader，其他则为Follower，Leader是通过内部的选举机制临时产生的
3. 选举细节
   1. 每台机器首先会给自己投一票尝试让自己成为老大，如果不成功就将票投给id比自己大的机器
   2. 老大选举出来之后，如果一切正常，后面加进入的机器默认为小弟
   3. 老大挂掉了，小弟们才可以重新选举

###### 节点类型

* 持久（Persistent）：客户端和服务器端断开连接后，创建的节点不删除
  * 持久化目录节点
  * 持久化顺序编号目录节点
* 短暂（Ephemeral）：客户端和服务器端断开连接后，创建的节点删除
  * 临时目录节点
  * 临时顺序编号目录节点

##### Zookeeper实战





##### Zookerper面试题

