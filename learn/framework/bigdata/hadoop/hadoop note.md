# hadoop
## block size
*   Block size is just an indication to HDFS how to split up and distribute the files across the cluster - there is not a physically reserved number of blocks in HDFS (you can change the block size for each individual file if you wish)
*   A final note - having lots to small files means that your name node will require more memory to track them (blocks sizes, locations etc), and its generally less efficient to process 128x1MB files than single 128MB file (although that depends on how you're processing it)

### http://blog.csdn.net/clerk0324/article/details/50888016
### https://www.cnblogs.com/Dhouse/p/6901028.html?utm_source=itdadao&utm_medium=referral

##  Hbase vs hive
*   Hbase主要解决实时数据查询问题，Hive主要解决数据处理和计算问题，一般是配合使用。
*   Hbase： Hadoop database 的简称，
    *   也就是基于Hadoop数据库，是一种NoSQL数据库，主要适用于海量明细数据（十亿、百亿）的随机实时查询，如日志明细、交易清单、轨迹行为等。
    *   数据应用从HBase查询数据；
*   Hive:Hive是Hadoop数据仓库 不是数据库
    *   主要是让开发人员能够通过SQL来计算和处理HDFS上的结构化数据，适用于离线的批量数据计算。
    *   HIve清洗处理后的结果，如果是面向海量数据随机查询场景的可存入Hbase   


## HA
### ZK Failover Controller 设计  故障自动转移
*   https://www.ibm.com/developerworks/cn/opensource/os-cn-hadoop-name-node/index.html
*   功能
    -   故障检测，自动发信active namenode 不可用
    -   触发将standby 节点切换为active
*   基于zooKeeper方式实现，使用zk的基础功能
    -   少量的任意信息强一致性存储 ==》 znodes
    -   当creator 出错时，创建的znode 自动删除 ==》 ephemeral node
    -   状态监控，异步通知 ==》 watchers
*   具体实现
    -   Failure detector 
        +   active NameNode 在zk中创建 ephemeral node，当active NameNode出错时， ephemeral node自动删除
    -   Active node locator 
        +   通过zk定位当前active  node
    -   Mutual exclusion of active state
        +   通过zk 确保只有一个active node 在同一个时刻
*   主要组件
    -   HealthMonitor，监控NameNode ,查看是否处理不可用或者进入不健康状态
        +   是一个线程，负责监控本地Namenode
        +   初始化完成之后会启动内部的线程来定时调用对应 NameNode 的 HAServiceProtocol RPC 接口的方法，对 NameNode 的健康状态进行检测。
        +   如果检测到 NameNode 的健康状态发生变化，会回调 ZKFailoverController 注册的相应方法进行处理。
    -   ActiveStandbyElector，管理和监控zookeeper中的状态
        +   主要两个方法：joinElection 加入选举，quitElection离开选举、
        +   ActiveStandbyElector 与 Zookeeper 进行交互完成自动的主备选举。
        +   ActiveStandbyElector 在主备选举完成后，会回调 ZKFailoverController 的相应方法来通知当前的 NameNode 成为主 NameNode 或备 NameNode。
    -   ZKFailoverController，订阅HealthMonitor 与 ActiveStandbyElector 事件
        +   管理NameNode状态
            *   启动时，初始化HealthMonitor，ActiveStandbyElector
            *   当HealthMonitor监控到状态改变时，处理相应的操作
            *   当ActiveStandbyElector发生改变时，调用本地NameNode执行相应操作
        +   如果 ZKFailoverController 判断需要进行主备切换，会首先使用 ActiveStandbyElector 来进行自动的主备选举。
*   三个组件运行在相同JVM,与NameNode在同一主机，不同JVM, ZKFC（DFSZKFailoverController） 进程
    -   一个HA集群2台NameNOde,每个NameNode运行自己的ZKFC进程

https://www.w3cschool.cn/hadoop/xvmi1hd6.html
