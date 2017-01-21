[官网](https://docs.docker.com/)
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
