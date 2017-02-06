#概述
##基本流程
![](images/1.jpg)
![](images/2.png)
##driver worker 通信关系
![](images/3.jpg)

##spark 基础
###RDD
*   resillient distributed dataset 弹性分布式数据集
    -    分布式内存的抽象
    -    操作本地集合的方式来操作分布式数据集的抽象实现
*   分布式只读且已分区集合对象,加载到内存处理
*   容错实现
    -    记录数据的更新
        -    spark记录RDD转换关系Lineage，重新计算
    -    数据检查点
*   特性
    -    数据存储结构不可变
    -    支持跨集群的分布式数据操作
    -    可对数据记录按key进行分区
    -    提供了粗粒度的转换操作
    -    数据存储在内存中，保证了低延迟性
*   RDD依赖--看父RDD被多少个子RDD依赖
    -    宽依赖，子RDD对父RDD中的所有data partition都有依赖
            -    一个父RDD的一个分区对应一个子RDD的多个分区
        -   窄依赖，子RDD依赖于父RDD中固定的data partition
            -   一个父RDD的一个分区最多被一个子RDD引用
*   RDD数据存储管理
    -    RDD理解为分布在集群上的大的数组
    -    RDD的一个分区为一个partion--罗辑分区
        -    Partitions
            -    一个逻辑数据块，对应相应的物理块Block
            -    变换前后的新旧分区在物理上可能是同一块内存存储
        -   RDD之间通过Lineage产生依赖关系
    -    数据的一个元数据结构
        -    存储着数据分区及其逻辑结构映射关系(block,node等）
        -    存储着RDD之前的依赖转换关系
        -   每个Block中存储着RDD所有数据项的一个子集，暴露给用户的可以是一个Block的迭代器
        -    数据通过Spark默认的或者用户自定义的分区器决定数据块分布在哪些节点
*   计算一组数据的逻辑计划


##SparkContext
*   Spark 应用程序的入口，负责调度各个运算资源，协调各个 Worker
Node 上的 Executor

##Job 
*    一个action生成一个job
*    一个或者多个RDD的一组转换操作
*   根据RDD依赖划分一个或者多个Stage
    -    Stage
        -    ResultStage
            -    最终的stage, 没有输出, 而是直接产生结果或存储
        -    ShuffleMapStage
            -    非最终stage, 后面还有其他的stage, 所以它的输出一定是需要shuffle并作为后续的输入
            Tasks
            -    一个partion == 一个 Task
            -    Task 执行RDD中对应stage的func
            -    Task被封装好后放入Executor的线程池中执行

##BlockManager
*   运行在所有节点（driver 和 executors）
*   提供在各种存储（memory，disk，offheap）中存取本地和远程block接口
*   管理RDD的物理分区,每个Block就是节点上对应的一个数据块

##DAGScheduler
*   DAG --有向无环图，RDD之间的依赖关系
*   根据Job构建基于Stage的DAG，并提交TaskScheduler
*   计算每个任务的最佳位置


##TaskScheduler
*   将Taskset提交给Worker node集群运行并返回结果

##Application
*   Spark 的应用程序，用户提交后，Spark为App分配资源，将程序转换并执行，其中Application包含一个Driver program和若干Executor

##Driver Program
*   运行Application的main()函数并且创建SparkContext
    -   集群运行方式
        -   client mode
            -   client start
                -   启动Driver进程
                    -   启动和实例化DAGScheduler等组件
                    -   向Master注册
                    -   启动SchedulerBackend
                -   work向Master注册
                    -   Master控制Worker启动Executor
                    -   启动ExecutorRunner线程
                        -   启动ExecutorBackend进程
        -   cluster mode
            -   client start
                -   提交Master
                    -   Master分配app到Worker并启动Driver

##Executor
*   是为Application运行在Worker node上的一个进程，该进程负责运行Task，并且负责将数据存在内存或者磁盘上。每个Application都会申请各自的Executor来处理任务


##Worker Node
*   集群中任何可以运行Application代码的节点，运行一个或多个Executor进程

##Shuffle
*   把一组无规则的数据尽量转换成一组具有一定规则的数据
*   包裹在各种需要重分区的算子之下的一个对数据进行重新组合的过程


