[官网](https://docs.docker.com/)
#docker
*   一般当人们说 “Docker”时， 他们一般指的是Docker Engine， 一个client-server 结构的应用， 包含Docker daemon，一个 用来和daemon 交互的REST API， 一个命令行应用CLI。 Docker Engine 在命令行中接收并解析、执行docker  命令， 例如: docker run <image>,  docker ps等。
*   Docker Machine 是一个创建和管理Docker Hosts 的工具。
    -   需要登录主机，按照主机及操作系统操作
    -   简化部署，在本机或者云服务，只要一个命令搭建好主机
    -   多平台docker管理
    -   本地迁移到云端，修改一下环境变量
*   docker结构 CS 结构：
    -   client,负责与Docker daemo交互
    -   Docker Host,docker 主机，可以是本地，也可以远程
        +   docker daemon,与client交互
        +   docker images
            *   只读模板，可以是一个已经安装了web应用的ubuntu操作系统
            *   images用来创建docker containers
        +   docker registries,images库，
        +   docker containers  
            *   containers 类似一个目录，持有运行程序所需要的全部资源
            *   每一个container从一个image创建
            *   可以启动，停止等，每个container都是资源隔离安全的应用平台
*   docker images如何工作
    -   每个image都有一系列的层级，通过UnionFs结合成一个image
    -   unionFs使得文件和目录独立于文件系统
    -   docker轻量级的一个原因就是因为这些层级
        +   更新一个应用只需要built一个新的层，而不需要像虚拟机那样替换整个image或者重新编译
    -   所有image都从一个基础image开始，如：ubuntu image ，apache image（可以作为所有web 应用的基础）
*   containers 如何工作：
    -   每个container 包含一些列系统资源，用户添加文件，元数据。
    -   images是只读的，但是运行在一个container中，就在image最上添加了一个读写层，

#环境:
1.  daocloud 加速
2.  环境准备 
    -   拉取ubuntu镜像
        -   sudo docker pull daocloud.io/library/ubuntu:xenial-20160317
        -   sudo docker run -it daocloud.io/library/ubuntu
    -   做自己的镜像
        -   docker commit c312ca2a51b5 ubuntu:fxiUbuntu
    -   导出
        -   docker save -o myubuntu.tar.gz ubuntu:fxiUbuntu
        -   sudo docker run -it ubuntu:fxiUbuntu sh -c '/etc/rc.local;/bin/bash'  
        -   docker load -i myubuntu.tar.gz
## windows/mac宿主访问方式:
1.  知道虚拟机的ip
    -   docker-machine ssh default
    -   ifconfig
2.  启动配置端口映射
    -   sudo docker run -it -p 50001:22 ubuntu:fxiUbuntu sh -c '/etc/rc.local;/bin/bash'
3.  访问的时候
    -   ssh -p 51000 test@虚拟机IP
    -   xifei 111111


docker run -p 9090:9090 prom/prometheus



##集群启动:
*   sudo docker run -it -h="s1" --name s1 -d -p 51001:22 -p 51002:7077  -p 51003:8080 -p 51004:4040  -p 51005:2181 -p 51006:9092 -p 51007:9042  -p 51008:8081 ubuntu:bigData sh -c '/etc/rc.local;/bin/bash'
*   sudo docker run -it -h="s2" --name s2 -d -p 52001:22 -p 52002:7077  -p 52003:8080 -p 52004:4040  -p 52005:2181 -p 52006:9092 -p 52007:9042  -p 52008:8081 ubuntu:bigData sh -c '/etc/rc.local;/bin/bash'
*   sudo docker run -it -h="master" --name master -d -p 50001:22 -p 50002:7077  -p 50003:8080 -p 50004:4040  -p 50005:2181 -p 50006:9092 -p 50007:9042 --link s1 --link s2  ubuntu:bigData sh -c '/etc/rc.local;/bin/bash'
*   sudo docker run -it -h="S4" --name S4 --link S1 --link S2  --link S3 -d -p 50004:22 ubuntu:bigDateTest sh -c '/etc/rc.local;/bin/bash'



##spark 集群：
1.  下载
2.  解压
3.  conf
    -   conf／slaves
    -   conf/spark-env.sh
    -   export JAVA_HOME='/usr/local/share/jdk1_8'
4.  /sbin/start-master.sh
    -   ./sbin/start-slave.sh spark://master:7077
    -   ./bin/spark-shell --master spark://master:7077
    -   ./start-master.sh
    -   ./start-slave.sh spark://172.17.0.8:7077
##zookeeper安装
```
 cp zoo_sample.cfg zoo.cfg
--------------
server.1=master:2888:3888
server.2=s1:2888:3888
server.3=s2:2888:3888
----------
vim /etc/hosts
172.17.0.18     master
172.17.0.11     s1 s1
172.17.0.16     s2 s2
-------
在data目录下创建myid
vim /tmp/zookeeper/myid
-----------
bin/zkServer.sh start
```


##kafka安装:
1. config/server.properties:
   broker.id=1
2. config/zookeeper.properites
3. bin/kafka-server-start.sh config/server.properties &
----
4.  test
    -   bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
    -   bin/kafka-topics.sh --list --zookeeper localhost:2181

##win10安装，toolbox
*   替换源
*   sudo passwd root,xifei
*   sudo adduser xifei

#Sentry Error
*   用户id : {map[id]}
*   应用类型 : {map[client]}
*   项目: {map[project]}
