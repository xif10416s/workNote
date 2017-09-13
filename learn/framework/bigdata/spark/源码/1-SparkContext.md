#   SparkContext -- v 2.2
##  初始化过程
*   前置条件SparkConf设置：
    -   spark.master 必须设置 
        -   本地模式为 local[*]
        -   集群模式为 master地址，如：spark://192.168.0.147:7077
*   初始化或者从缓存获取SparkContext ：
    -   SparkContext##getOrCreate(config: SparkConf)
*   初始化context 
    -   new SparkContext(config)
*   重要组件初始化并启动
    -   SparkUI
        +   客户端webui,默认端口4040
    -   schedulerBackend [后台线程]:   SchedulerBackend，管理系统，决定如何获取资源，配合TaskSchedulerImpl运行task
        +   意图：不同集群资源获取方式不一样，相对于不同的集群提供不同的策略实现
        +   接口方法
            *   reviveOffers,找到合适work的合适的executor资源给task运行   
        +   在standalone模式下，StandaloneSchedulerBackend负责集群资源的获取和调度。继承自CoarseGrainedSchedulerBackend。
            *   **客户端通信线程**:StandaloneAppClient
                -   `client = new StandaloneAppClient(sc.env.rpcEnv, masters, appDesc, this, conf)`
                -   将driver app信息注册到spark 的master，包含driver端的通信地址
                -   监听并处理来自master过来的消息
        +   在本地模式下，LocalSchedulerBackend
    -   TaskScheduler :
        +   接收 DAGScheduler 划分好stage之后包含task列表的set
        +   负责发送task给集群运行，已经失败重试等操作
        +   接口方法
            *   submitTasks：提交任务执行
    -   persistentRdds :一个线程安全的map,跟踪所有缓存rdd, gc是自动释放缓存
        +   `val map: ConcurrentMap[Int, RDD[_]] = new MapMaker().weakValues().makeMap[Int, RDD[_]]()
    map.asScala`
        +   MapMaker:
            *   Google Collections中的MapMaker融合了Weak Reference，线程安全，高并发性能，异步超时清理，自定义构建元素等强大功能于一身。
    -   DAGScheduler ： DAG计算
        +   为每一个job计算DAG图，把划分的stage作为taskset的形式提交给TaskScheduler执行task
    -   SparkEnv
        +   blockManager
            *   管理每个节点（driver和executors)上的消息块
            *   提供了一套管理接口：存放或者接收来自本地和远程的各种消息块
                -   getBlockData
                -   putBlockData
            *   消息快存储在内存，或者磁盘
                -   memoryStore
                -   diskStore
