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

###### zkCli客户端命令介绍

在启动zkServer和zkCli正常连接之后，在zkCli客户端的操作

1. help查看所有可用的命令
2. ls path---> ls / 注意zookeeper根目录是/，该命令是列举zookeeper下的可用节点，现在是一个
3. ls2 path ---> ls2 / 更详细的列举zookeeper下的信息
4. create path data --> create /sanguo "liubei" 在/目录下创建节点sanguo并在该节点下写入数据“liubei"，如果不写入数据，节点不会被创建，注意不可级联创建节点，比如create /xiaoshuo/xiyouji ”sunwukong"，因为西游记上一级节点没有数据
5. get path ---> get /sanguo 获取指定路径下的数据
6. create -e /sanguo/wuguo "sunquan"创建短暂类型节点，注意/sanguo节点前面已经创建了，此时quit命令退出，重新连接，查看节点状态发现/wuguo不存在
7. create -s /sanguo/weiguo "caozao"创建持久型顺序节点weiguo，ls /sanguo查看节点名称变化
8. set /sanguo/shuguo "liushan" 修改shuguo节点的信息
9. 另外启动一个zkCli客户端get /sanguo watch 监听/san'guo这个节点的数据变化，注意只有一次有效，用另外一个zkCli set /sanguo "sanguo2"，返回查看信息的变化
10. delete /sanguo/wuguo 删除指定节点

###### stat结构体

1. czxid-创建节点的事务zxid

   每次修改zookeeper状态都会收到一个zxid形式的时间戳，也就是zookeeper事务id。事务id是zookeeper中所有修改总的次序，每个修改都有唯一的zxid，如果zxid1小于zxid2，那么zxid1发生在zxid2之前

2. ctime-znode被创建的毫秒数（从1970年开始）

3. mzxid-znode最后更新的事务zxid

4. mtime-znode最后修改的毫秒数

5. pZxid-znode最后更新的子节点zxid

6. cversion-znode子节点变化号，znode子节点修改次数

7. dataversion-znode数据变化号

8. aclVersion-znode访问控制列表的变化号

9. ephemeralOwner如果是临时节点，这个是znode拥有者的sessionid，如果不是临时节点，则为0

10. <font color=red>dataLength-znode的数据长度</font>

11. <font color=red>numChildren-znode</font>

###### 监听器原理

1. 首先要有一个main()线程
2. 在main线程中创建Zookeeper客户端，这时就会创建两个线程，一个负责网络连接通信connect，一个负责监听listener
3. 通过connect线程将注册的监听时间发送给zookeeper
4. 在zookeeper的注册监听器列表中将注册的监听事件添加到列表中
5. zookeeper监听到有数据路径变化，就会将这个消息发送给listener线程
6. <font color=red>listener线程内部调用了process方法</font>

一般会监听节点数据的变化或者节点增减的变化

###### 写数据流程

1. client项zookeeper的server1上写数据，发送一个写请求，如果server1不是leader，那么server1会把接收到的请求进一步转发给leader，leader广播给下面的follower，下面的follower会去写，写完之后会告诉leader
2. leader收到一半以上的follower的确认回复之后就会认为数据已经写好了，然后通知第一次的server1
3. server1收到来自leader的全局数据保存成功之后，会去通知client数据已经写好了

##### Zookeeper实战（动态监听服务器的上下线）

1. 启动zkServer，启动zkCli，创建节点create /servers "servers"

2. 创建maven工程并编写pom文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cliff.ford</groupId>
    <artifactId>zookeeper实践</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.11.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.5.3-beta</version>
        </dependency>
    </dependencies>
</project>
```

3. 编写监听客户端zkClient

   ```java
   public class DistributeClient {
       private ZooKeeper zkCli;
       public static void main(String[] args) throws IOException, KeeperException, InterruptedException {
           DistributeClient client = new DistributeClient();
           client.getConnect();
           client.getChildren();
           client.buiness();
       }
   
       //令程序一直运行
       private void buiness() throws InterruptedException {
           Thread.sleep(Long.MAX_VALUE);
       }
   
       private void getChildren() throws KeeperException, InterruptedException {
           //获取/servers下的节点，并设置一次该路径的监听事件
           List<String> children = zkCli.getChildren("/servers", true);
           ArrayList<String> hosts = new ArrayList<String>();
           for(String child : children){
               byte[] data = zkCli.getData("/servers/" + child, false, null);
               hosts.add(new String(data)+child);
           }
           System.out.println(hosts);
       }
   
       private void getConnect() throws IOException {
           //给该zkCli设置监听事件
           zkCli = new ZooKeeper("localhost:2181", 2000, new Watcher() {
               public void process(WatchedEvent watchedEvent) {
                   try {
                       //监听事件里面设置了监听事件，所以如果程序一直运行，则一直监听
                       getChildren();
                   } catch (KeeperException e) {
                       e.printStackTrace();
                   } catch (InterruptedException e) {
                       e.printStackTrace();
                   }
               }
           });
       }
   
   
   }
   ```

4. 在cmd界面的zkCli创建删除/servers/xxxx，查看监听程序的反应

5. 编写DistrubuteServer，并执行，查看监听程序的反应，停止运行该程序，查看监听程序的变化

   ```java
   public class DistributeServer {
       private String connectString = "localhost:2181";
       private int sessionTimeout = 2000;
       private ZooKeeper zkClient;
   
       public static void main(String[] args) throws IOException, KeeperException, InterruptedException {
           DistributeServer server = new DistributeServer();
           server.getConnect();
           for(int i = 0; i < 3; i++){
               server.regist(args[0]);
           }
           server.business();
       }
   
       private void business() throws InterruptedException {
           Thread.sleep(Long.MAX_VALUE);
       }
   
       private void regist(String hostname) throws KeeperException, InterruptedException {
           String s = zkClient.create("/servers/server", hostname.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
           System.out.println(s);
       }
   
       private void getConnect() throws IOException {
           zkClient = new ZooKeeper(connectString,sessionTimeout,new Watcher(){
   
               public void process(WatchedEvent watchedEvent) {
   
               }
           });
       }
   }
   ```

##### Zookerper面试题

* 监听原理
* 部署方式（单机和集群）
* Leader和Follower的关系和转变
* 集群最少几台（3）（半数原则）
* 常用命令

