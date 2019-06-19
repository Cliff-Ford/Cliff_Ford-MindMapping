<center><h3>Docker学习记录</h3></center>

#### 发展背景

​		Docker为开发阶段和运维阶段之间的交付过程提供了一个标准的解决方案。以往的交付是程序打包成jar文件交付，但却缺少相应的运行环境，配置文件以及操作系统的差异性。现在是整体打包交付，包括程序，配置环境，运行环境，运行依赖，操作系统发行版，内核等信息一起打包交付。

​		传统上认为，软件编码开发/测试结束后，所产出的成果即是程序或是能够编译执行的二进制字节码等（Java为例）。而为了让这些程序可以顺利执行，开发团队得准备完整的部署文件，让运维团队得以部署应用程序。不过，即使如此还是经常会出错。Docker镜像的出现，使得Docker得以打破过去 ***程序即应用***  的 概念，透过镜像***images***将作业系统核心除外，运作应用程序所需要的系统环境，有下而上打包，达到应用程序跨平台间的无缝接轨运作。

​		比喻，以前是搬家，现在是搬楼。

​		一句话，解决了运行环境和配置问题软件容器，方便做持续集成并有助于整体发布的容器虚拟化技术。

#### 概念

* 虚拟机：虚拟机可以在一种操作系统里面运行另一种操作系统，比如在Windows系统里面运行Linux系统。应用程序对此毫无感知，因为虚拟机看上去跟真实系统一模一样，而对于本机底层系统来说，虚拟机就是一个普通文件，不需要了就删除，对其他部分毫无影响。这类虚拟机完美的运行了另一套系统，能够使应用程序，操作系统和硬件三者之间的逻辑不变。
* 镜像（image）：镜像就是模板
* 容器（container）：镜像的一个实例叫容器，可以把容器看作是一个简易版的Linux环境（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。容器的定义和镜像几乎一模一样，也是一堆层的统一视角，唯一区别在于容器的最上面那一层是可读可写的。
* 仓库（repository）：仓库是集中存放镜像文件的场所，有公开仓库和私有仓库两种类型，Docker Hub是最大的公开仓库（科学上网），阿里云，网易云等也属于公开仓库

#### 常用命令

##### <font color=red>docker version</font>

查看docker的版本信息

```
Client: Docker Engine - Community
 Version:           18.09.2
 API version:       1.39
 Go version:        go1.10.8
 Git commit:        6247962
 Built:             Sun Feb 10 04:12:31 2019
 OS/Arch:           windows/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          18.09.2
  API version:      1.39 (minimum version 1.12)
  Go version:       go1.10.6
  Git commit:       6247962
  Built:            Sun Feb 10 04:13:06 2019
  OS/Arch:          linux/amd64
  Experimental:     false
```



##### <font color=red>docker info</font>

查看docker现在的运行信息，比version更详细，包含docker正在运行的容器，安装了哪些镜像等

```
Containers: 2
 Running: 0
 Paused: 0
 Stopped: 2
Images: 1
Server Version: 18.09.2
Storage Driver: overlay2
 Backing Filesystem: extfs
 Supports d_type: true
 Native Overlay Diff: true
Logging Driver: json-file
Cgroup Driver: cgroupfs
Plugins:
 Volume: local
 Network: bridge host macvlan null overlay
 Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
Swarm: inactive
Runtimes: runc
Default Runtime: runc
Init Binary: docker-init
containerd version: 9754871865f7fe2f4e74d43e2fc7ccd237edcbce
runc version: 09c8266bf2fcf9519a651b04ae54c967b9ab86ec
init version: fec3683
Security Options:
 seccomp
  Profile: default
Kernel Version: 4.9.125-linuxkit
Operating System: Docker for Windows
OSType: linux
Architecture: x86_64
CPUs: 2
Total Memory: 1.196GiB
Name: linuxkit-00155d976006
ID: OQFJ:MOE3:LBIN:OJGD:SO7U:NUGG:WEFE:LSNO:PAOZ:6LG4:SWVJ:D4DJ
Docker Root Dir: /var/lib/docker
Debug Mode (client): false
Debug Mode (server): true
 File Descriptors: 22
 Goroutines: 47
 System Time: 2019-06-10T05:30:14.6230217Z
 EventsListeners: 1
Registry: https://index.docker.io/v1/
Labels:
Experimental: false
Insecure Registries:
 127.0.0.0/8
Registry Mirrors:
 https://22nfujc0.mirror.aliyuncs.com/
Live Restore Enabled: false
Product License: Community Engine
```

##### <font color=red>docker --help</font>

如果忘记docker的命令了，可以这样查看

```
Usage:  docker [OPTIONS] COMMAND

A self-sufficient runtime for containers

Options:
      --config string      Location of client config files (default
                           "C:\\Users\\Cliff_Ford\\.docker")
  -D, --debug              Enable debug mode
  -H, --host list          Daemon socket(s) to connect to
  -l, --log-level string   Set the logging level
                           ("debug"|"info"|"warn"|"error"|"fatal")
                           (default "info")
      --tls                Use TLS; implied by --tlsverify
      --tlscacert string   Trust certs signed only by this CA (default
                           "C:\\Users\\Cliff_Ford\\.docker\\ca.pem")
      --tlscert string     Path to TLS certificate file (default
                           "C:\\Users\\Cliff_Ford\\.docker\\cert.pem")
      --tlskey string      Path to TLS key file (default
                           "C:\\Users\\Cliff_Ford\\.docker\\key.pem")
      --tlsverify          Use TLS and verify the remote
  -v, --version            Print version information and quit

Management Commands:
  builder     Manage builds
  config      Manage Docker configs
  container   Manage containers
  image       Manage images
  network     Manage networks
  node        Manage Swarm nodes
  plugin      Manage plugins
  secret      Manage Docker secrets
  service     Manage services
  stack       Manage Docker stacks
  swarm       Manage Swarm
  system      Manage Docker
  trust       Manage trust on Docker images
  volume      Manage volumes

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  build       Build an image from a Dockerfile
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  images      List images
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes

Run 'docker COMMAND --help' for more information on a command.
```

<font color=red>docker [OPTIONS] COMMAND</font>  怎么看？怎么用？

1. docker 是命令前缀，必须且不可更改

2. [OPTIONS] 可选项，***-v*** 表示版本，***-D*** 表示debug，可选项可以连着用 ***-DH***，也可以全称 ***--version***
3. COMMAND 表示命令

<font color=red>Run 'docker COMMAND --help' for more information on a command.</font>表示用docker run --help查找run命令的帮助信息

##### <font color=red>docker images [options]</font>

查看本地的镜像

```
C:\Users\Cliff_Ford>docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              fce289e99eb9        5 months ago        1.84kB
```

返回结果参数讲解

| repository                 | tag                | image id | created      | size     |
| -------------------------- | ------------------ | -------- | ------------ | -------- |
| 镜像的仓库源，相当于镜像名 | 标签，相当于版本号 | 镜像id   | 镜像创建时间 | 镜像大小 |

1. docker images -a 表示列出本地所有镜像的所有信息
2. docker images -p 表示列出本地所有镜像的所有id
3. docker images --help 查看查看images命令的更多信息

##### <font color=red>docker search target-image-name</font>

docker search tomcat 在docker查找tomcat的镜像

##### <font color=red>docker pull target-image-name</font>

docker pull tomcat 下载tomcat镜像，默认最新版本

##### <font color=red>docker run target-image-name</font>

docker run hello-world

docker window版安装完并启动之后，会生成一个daemon后台线程来处理命令，cmd窗口相当于一个client，输入docker run hello-world并回车，告诉后台线程执行hello-world这个容器，docker就会在本机寻找该容器，如果容器没有就会去找相应镜像，然后创建该容器，如果镜像也没有，就会去仓库拉取该镜像，再利用拉到的镜像创建容器，再执行。

##### <font color=red>docker ps [options]</font>

列出所有正在运行的容器，-it交互形式运行容器，-d后台运行容器

##### <font color=red>docker退出容器</font>

1. exit 容器会被关闭，重新进入需要start
2. ctrl+P+Q 容器会被挂起，重新进去不需要start，但要用指定命令进去，***docker attach target-container-name|target-container-id*** ,当然也有另一种不进去容器，即可操作容器的方法 ***docker exec -t target-container-name|target-container-id command*** 表示进入到指定的容器，执行什么命令，结果返回到宿主机，通常该命令形式如下

<center>docker exec -t container-id ls</center>

##### <font color=red>docker start target-container-name|target-container-id</font>

根据指定容器名或者id启动容器

##### <font color=red>docker restart target-container-name|target-container-id</font>

根据指定容器名或者id重启容器

##### <font color=red>docker stop target-container-name|target-container-id</font>

正常停止指定容器

##### <font color=red>docker kill target-container-name|target-container-id</font>

强制停止指定容器

##### <font color=red>docker logs -f -t --tail</font>

-t是加入时间戳

-f是跟随最新的日志打印

--tail数字，显示最后多少条

##### <font color=red>docker inspect target-container-id</font>

查看指定容器内部细节

##### <font color=red>docker cp target-container-name|target-container-id:source-contents target-contents</font>

将容器内指定目录文件拷贝到目标路径

##### <font color=red>docker commit -a="author" -m="describe" target-id|target-name namespace/image-name:tag</font>

以哪个镜像为目标重新生成一个新的镜像发布到docker供其他人下载使用

#### Docker是怎么工作的

Docker是一个Client-Server结构的系统，Docker守护进程运行在主机上，然后通过Socket链接从客户端访问，守护进程从客户端接收命令并管理运行在主机上的容器。容器，是一个运行时环境，就是理解上的集装箱

#### Docker比VM快的原因

1. docker有着比虚拟机更少的抽象层。由于docker不需要Hypervisor实现硬件资源虚拟化，运行在docker容器上的程序直接使用的都是实际物理机的硬件资源。因此在Cpu、内存利用率上docker将会比VM有明显优势
2. docker利用的是宿主机的内核，而不需要Guest OS。因此，当新建一个容器时，docker不需要和虚拟机一样重新加载一个操作系统内核，从而避免引寻、加载操作系统内核等比较费时费资源的过程，当新建一个虚拟机软件需要加载Guest OS，新建过程是分钟级别的，而docker是秒级的

|            | Docker容器              | 虚拟机（VM）                |
| ---------- | ----------------------- | --------------------------- |
| 操作系统   | 与宿主机共享OS          | 宿主机OS上面运行虚拟机OS    |
| 存储大小   | 镜像小，便于存储与传输  | 镜像庞大                    |
| 运行性能   | 几乎无额外性能损失      | 操作系统额外的CPU、内存消耗 |
| 移植性     | 轻便、灵活、适应于Linux | 笨重，与虚拟化技术耦合度高  |
| 硬件亲和性 | 面向软件开发者          | 面向硬件运维者              |

#### Docker镜像

docker镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS。

bootfs(boot file system)主要包含bootloader和kernel，bootloader主要是一i难道加载kernel，Linux刚启动时会加载bootfs文件系统，在docker镜像的最底层时bootfs。这一层与我们典型的Linux/Unix系统时一样的，包含boot加载器和内核，当boot加载完之后整个内核就都在内存中了，此时内存的使用权由bootfs转交给内核，此时系统也会卸载bootfs

rootfs(root file system)，在bootfs之上，包含的就是典型Linux系统中的/dev，/proc，/bin，/etc等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如ubuntu,centos等等

![镜像打包示意图](D:\02_个人文档\照片\16[00_13_33][20190611-110545].png)

<center>tomcat镜像打包示意图</center>

docker pull tomcat时发现有400多M，为什么这么大呢？因为镜像时在搬楼，它把运行tomcat需要的所有环境自下而上的打包了出来，从操作系统内核kernel->centos->jdk8->tomcat全部打包，这也是为什么镜像换环境也不容易出错的原因

#### DockerFile

DockerFile是用来构建Docker镜像的构建文件，是由一系列命令和参数构成的脚本

以centos镜像为例

```
# FROM 代表继承，scratch相当于Java中的Object类，scratch是所有镜像的基础镜像
FROM scratch
# ADD 添加一个压缩文件
ADD centos-7-x86_64-docker.tar.xz /
# 该镜像的一些标签
LABEL org.label-schema.schema-version="1.0" \
    org.label-schema.name="CentOS Base Image" \
    org.label-schema.vendor="CentOS" \
    org.label-schema.license="GPLv2" \
    org.label-schema.build-date="20190305"
# docker run image 后默认执行的一条命令
CMD ["/bin/bash"]
```

#### DockerFile构建过程解析

1. DockerFile内容基础知识

   * 每条保留字指令都必须为大写字母且后面要跟至少一个参数
   * 指令按照从上到下，顺序执行
   * #表示注释
   * 每条指令都会创建一个新的镜像层，并对镜像层进行提交

2. Docker执行DockerFile的大致流程

   1. docker从基础镜像运行一个容器
   2. 执行一条指令并对容器做出修改
   3. 执行类似docker commit的操作提交一个新的镜像层
   4. docker再给予刚提交的镜像运行一个新容器
   5. 执行dockerfile中下一条指令直到所有指令都执行完成

3. 小总结

   从应用软件的角度来看，DockerFile、Docker镜像与Docker容器分别代表软件的三个不同阶段

   * DockerFile是软件的原材料
   * Docker镜像是软件的交付品
   * Docker容器则可以认为是软件的运行态

   DockerFile面向开发，Docker镜像成为交付标准，Docker容器则设计部署与运维，三者缺一不可，合力充当Docker体系的基石

#### DockerFile保留字指令

* FROM 基础镜像，当前编写的镜像给予哪个镜像
* MAINTAINER 镜像维护者的姓名和邮箱地址
* RUN 容器构建时需要运行的命令
* EXPOSE 当前容器对外暴露出的端口号
* WORKDIR 指定在创建容器后，终端默认登陆进来的工作目录，一个落脚点
* ENV 用来在构建镜像过程中设置环境变量
* ADD 将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包
* COPY 类似ADD，拷贝文件和目录到镜像中。将从构建上下文目录中<源路径>的文件/目录复制到新的一层的镜像内的<目标路径>位置
  * COPY　src dest
  * COPY ["src", "dest"]
* VOLUME 容器数据卷，用于数据保存和持久化工作
* CMD 指定一个容器启动时要运行的命令，DockerFile中可以有多个CMD命令，但只有最后一个生效，CMD会被docker run之后的参数替换
* ENTRYPOINT 指定一个容器启动时要运行的命令，ENTERPOINT和CMD一样，都是在指定容器启动程序及参数，命令不会覆盖，起追加命令作用
* ONBUILD 当构建一个被继承的DockerFile运行命令，父镜像在被子继承父镜像的ONBUILD被触发

```
# 继承centos镜像
FROM centos
# 作者信息
MAINTAINER cliff_ford<473375811@qq.com>
# 设置环境变量
ENV mypath /tmp
# 设置工作目录，即落脚点
WORKDIR $mypath
# 安装支持vim ifconfig命令的工具包
RUN yum -y install vim
RUN yum -y install net-tools
# 暴露容器80端口
EXPOSE 80
# shell编程
CMD echo $mypath
CMD echo "success--------ok"
# 执行命令
CMD /bin/bash
```

#### docker操作实例

##### docker运行mongo实例

* docker pull mongo（下载）
* docker images（查看下载是否成功）
* docker run -d -p 27017:27017 --name mongo -v /home/docker_data/mongo:/data/db -e MONGO_INITDB_ROOT_USERNAME=root -e MONGO_INITDB_ROOT_PASSWORD=1234 mongo（后台运行，端口映射，别名，数据卷映射，账号密码）
* docker ps（查看是否运行成功）
* docker exec -it mongo bash（重新进入指定容器）
* mongo -u root -p 1234（登陆）
* help（查看帮助命令）

##### docker运行redis实例

* docker pull redis
* docker images
* docker run -d -p 6379:6379 --name redis redis
* docker ps
* docker exec -it redis redis-cli

























