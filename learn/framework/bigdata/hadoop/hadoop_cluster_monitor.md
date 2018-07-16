# HADOOP 集群监控管理(CDH 5.8.0)
## hadoop 集群
### master1
####  org.apache.hadoop.hdfs.server.namenode.NameNode 
*    功能
    -    主要负责管理hdfs文件系统
        +    整个文件系统的文件目录树
        -    文件/目录的元信息和每个文件对应的数据块列表
        -    接收用户的操作请求。
    -   fsimage:元数据镜像文件。存储某一时段NameNode内存元数据信息   
        +   dfs.name.dir:/data/dfs/nn  <==187M
    -   edits:NameNode启动后,操作日志文件，append log 性能好
        +   由secondaryNamenode合并 fsImage + edits.log
    -   fstime:保存最近一次checkpoint的时间
*   NameNode / secondaryNamenode 状态检查
    -   curl 'http://uhadoop-adarbt-master1:50070/jmx?qry=Hadoop:service=NameNode,name=NameNodeStatus'
*   NameNode 高可用整体架构 -- https://www.ibm.com/developerworks/cn/opensource/os-cn-hadoop-name-node/
    -   主备切换控制器 ZKFailoverController：ZKFailoverController 作为独立的进程运行，对 NameNode 的主备切换进行总体控制。ZKFailoverController 能及时检测到 NameNode 的健康状况，在主 NameNode 故障时借助 Zookeeper 实现自动的主备选举和切换，当然 NameNode 目前也支持不依赖于 Zookeeper 的手动主备切换。
*   -Xmx3072m
*   停止原因
    -   Active NameNode 提交 EditLog 到 JournalNode 集群的过程实际上是同步阻塞的，但是并不需要所有的 JournalNode 都调用成功，只要大多数 JournalNode 调用成功就可以了。如果无法形成大多数，那么就认为提交 EditLog 失败，NameNode 停止服务退出进程
*   logs :/var/log/hadoop-hdfs/
    -   hadoop-hadoop-namenode-uhadoop-adarbt-master1.log
*   -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/path/heap/dump

#### JournalNode -- 基于 QJM 的共享存储系统的总体架构
*   基于 QJM 的共享存储系统主要用于保存 EditLog，并不保存 FSImage 文件。
*   QJM 共享存储的基本思想来自于 Paxos 算法 (参见参考文献 [3])，采用多个称为 JournalNode 的节点组成的 JournalNode 集群来存储 EditLog。
*   每个 JournalNode 保存同样的 EditLog 副本。每次 NameNode 写 EditLog 的时候，除了向本地磁盘写入 EditLog 之外，也会并行地向 JournalNode 集群之中的每一个 JournalNode 发送写请求，只要大多数 (majority) 的 JournalNode 节点返回成功就认为向 JournalNode 集群写入 EditLog 成功。
*   如果有 2N+1 台 JournalNode，那么根据大多数的原则，最多可以容忍有 N 台 JournalNode 节点挂掉。

#### org.apache.hadoop.hdfs.tools.DFSZKFailoverController -- 它负责整体的故障转移控制
*   日志
    -   hadoop-hadoop-zkfc-uhadoop-adarbt-master1.log

#### org.apache.hadoop.yarn.server.resourcemanager.ResourceManager
*   日志：/var/log/hadoop-yarn
*   ResourceManager HA
    -   Automatic failover
        +   Zookeeper-based ActiveStandbyElector决定RM是否active，rm自带
        +   不需要单独的 ZKFC 
    -   yarn ha的共享内存系统被抽象成RMStatStore
        +    底层采用不同的共享文件系统，比如zookeeper，NFS，HDFS等，不同的实现都要继承这个类，然后实现具体的save/load等功能。
*   命令
    -   yarn rmadmin -getServiceState rm1
    -   yarn rmadmin -transitionToStandby rm1 、 transitionToActive

#### org.apache.hadoop.hdfs.qjournal.server.JournalNode 共享存储系统
*   Active Namenode往里写editlog数据,StandBy再从里面读取数据进行同步


### master2
#### org.apache.hadoop.hdfs.server.namenode.NameNode  standBy
#### org.apache.hadoop.yarn.server.resourcemanager.ResourceManager standby
#### org.apache.hadoop.hdfs.qjournal.server.JournalNode
#### org.apache.hadoop.mapreduce.v2.hs.JobHistoryServer
*   jobhistory记录下已运行完的MapReduce作业信息并存放在指定的HDFS目录下

### nodeN
#### org.apache.hadoop.hdfs.server.datanode.DataNode
*   根据客户端或者是namenode的调度存储和检索数据，并且定期向namenode发送他们所存储的块(block)的列表。

#### org.apache.hadoop.yarn.server.nodemanager.NodeManager
*   是ResourceManager在每台机器上的代理，负责容器管理，并监控它们的资源使用情况，以及向ResourceManager/Scheduler提供资源使用报告

## hbase 集群
### master1
#### org.apache.hadoop.hbase.master.HMaster
*   log ： /var/log/hbase
*   功能
    -   负责分配region到regionserver,检测新增或失败的regionserver,与regionserver交互,regionserver间的负载均衡等;
    -   处理shcema的变更;
    -   实现ZooKeeper的Watcher接口,与zookeeper集群交互

#### org.apache.hadoop.hbase.thrift.ThriftServer
*   you use Java API - you access Region Server directly

### nodeN
####  org.apache.hadoop.hbase.regionserver.HRegionServer
*   log:/var/log/hbase
    