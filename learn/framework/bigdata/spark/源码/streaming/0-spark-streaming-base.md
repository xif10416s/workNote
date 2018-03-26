#  spark streaming  -- socketTextStream 基本处理
##   主要功能及组件
-   DStreams
    -   相同类型的一组RDD,代表连续的数据流
    +   可以由实时数据流创建,TCP sockets , kafka ,flume ,现有DStreams转换操作,
    +   该类提供了基础的map,fliter,widow,PairDStreamFunction ==>join,groupByKeyAndWindow
    +   batchInterval＝＞　batchDuration，执行任务的周期，包含一个或者多个ｂｌｏｃｋ
    +   blockInterval　＝＞　receiver接收到数据后创建ｂｌｏｃｋ的周期
    +   一个DStream关联一个ｒｅｃｅｉｖｅｒ，需要从多个Receiver并行读取需要创建多个DStream,一个Receiver运行载一个Ｅｘｅｃｕｔｏｒ，占用１个ｃｐｕ
    +   来自数据源的数据接收到以后，ｒｅｃｅｉｖｅｒ会每隔blockInterval时间生成ｂｌｏｃｋ
    +   batchInterval　间隔的block被创建成RDD,block数就是ＲＤＤ的分区数，分区数对应task数， blockInterval== batchinterval 　意味着只有一个分区，本地执行
-   DStreamGraph
    +   generateJobs：根据分配的block,生成jobs(周期性的产生一个job)
-   InputDStream
    +   继承自DStream，所有输入流的基类
    +   spark stream 系统会调用 start 和stop来开始或者停止接受数据
    -   一些不需要在worker 节点运行接受输入数据的操作，可以直接继承找个类，只运行在driver 节点.
        -   例如:FileInputDStream => 监控 HDFS目录生成的新文件
    -   需要在ｗｏｒｋｅｒ节点上接收数据的，需要继承ReceiverInputDStream
*   Receiver -- 数据接收抽象基类
    *   在ｗｏｒｋｅｒ节点获取数据的对象
    -   ReceiverSupervisorImpl
        +   监督并处理运行在work节点的receiver接受到的数据
        -   pushSingle　将接收到的数据添加数据到BlockGenerator的ｂｕｆｆｅｒ
            +   socket receiver 通过这个方法记录数据,测试用数据量小，直接在内存保存
        -   pushArrayBuffer 
            -   通过blockManager存储ｂｌｏｃｋ //TODO 
            -   通知driver 有新的blockInfo
            -   flume && kafka receiver 使用这个方法记录数据
        -   BlockGenerator
            -   waitToPush -- 单条数据推送
                -   限制每秒中能处理多少个消息
                -   可以通过spark.streaming.receiver.maxRate 参数配置，默认Long.MaxValue，不限速
                -   实现方案：
                    +   com.google.common.util.concurrent.RateLimiter
            -   启动线程，
                -   blockIntervalTimer线程
                    -   一个线程负责把ｒｅｃｅｉｖｅｒ中获取的数据分批block，缓存到推送队列blocksForPushing
                    -  分批时间间隔 "spark.streaming.blockInterval", "200ms" 
                -   blockPushingThread线程
                    -   通知receiver调用pushArrayBuffer把这些分批好的块push到block manager
*   ReceiverTracker
    -   管理ReceiverInputDStreams的执行
        +   负责发起启动receivers 的事件，启动receiver
    -   ReceivedBlockTracker
        +   跟踪接受到的数据block,划分批次
*   StreamingContext
    -   spark流处理上下文，用来创建DStream
*   JobScheduler
    -   通过JobGenerator生成job ,在spark上运行
    -   RecurringTimer
        +   ssc.graph.batchDuration.milliseconds 为周期生成GenerateJobs事件
    -   jobExecutor 执行任务的线程池，默认只有1个线程 -- 运行在driver上
        +   numConcurrentJobs = ssc.conf.getInt("spark.streaming.concurrentJobs", 1)
        +   执行的job 也是stream包的job
            *   org.apache.spark.streaming.scheduler.job
*   JobGenerator
    -   根据Receiver接受到的数据划分批次，在指定的间隔放入合适的block
    -   启动2个线程
        +   一个线程周期性开始一个新的批次，并把之前的批次划分成block
        +   一个线程推送这些block到block manager
*   ExecutorAllocationManager
    -   isDynamicAllocationEnabled(conf),根据配置是否打开
        +   key = spark.streaming.dynamicAllocation.enabled , 默认不打开
    -   管理executor的分配，和销毁
    -   根据每个批次的处理时间，而不是executor的空闲时间创建executor
        +   通过监听接口获取每个批次的处理时间
        +   计算平均处理时间 / 批次间隔时间 当大于一定上限比例时，请求更多的executor，小于一定下限比例时，销毁一些executor
*   组成关系
```
StreamingContext
    -   sparkContext
    -   DStreamGraph
    -   JobScheduler
        +   ExecutorAllocationManager
        -   ReceiverTracker 
        +   JobGenerator
            *   graph =ref ssc.graph
            *   RecurringTimer

SocketInputDStream
    -   getReceiver => SocketReceiver extends Receiver
        +   ReceiverSupervisorImpl extends ReceiverSupervisor
            *   BlockManagerBasedBlockHandler
            *   registeredBlockGenerators = new ConcurrentLinkedQueue[BlockGenerator]()
                -   BlockGenerator
            
```


##   receiver方式基本步骤 
-   利用  sparkConf创建一个 SparkContext
        +    StreamingContext.createNewSparkContext(conf)
-   初始化：SocketInputDStream  -- 实现如何从socket获取输入流
        -   val lines = ssc.socketTextStream(args(0), args(1).toInt, StorageLevel.MEMORY_AND_DISK_SER)
        -   提供了SocketReceiver创建的接口，在StreamingContext#start的时候创建并启动，开始接受数据
        -   注册到graph的inputStreams
-   转换操作 --  val words = lines.flatMap(_.split(" "))
    +   和RDD转换操作相似，封装成对应新的DStream(FlatMappedDStream,ShuffledDStream)
*   action操作 -- wordCounts.print()
    -   包装成ForEachDStream，并注册到graph的outputStreams
    -   父亲对象为转换操作的DStream，最终父亲对象为SocketInputDStream
        +   计算的时候先从最终父类SocketInputDStream.compute开始计算
*   StreamingContext启动 -- ssc.start()
    -   在worker节点创建receiver，启动线程接受socket数据
    -   定时周期线程，发送批次事件
        +   划分批次
        +   创建job并处理

##  receiver方式基本流程图
![](../../images/spark_stream_base.jpg)


## kafka stream -- 两种方式
### Receiver 方式 -- KafkaUtils.createStream(ssc, zkQuorum, group, topicMap).map(_._2)
*   KafkaInputDStream
*   Receiver
    -   KafkaReceiver -- 不可靠方式
        +   创建ConsumerConnector = Consumer.create(consumerConfig)
        +   根据配置的topic信息创建消息流topicMessageStreams = consumerConnector.createMessageStreams
        +   为每个topic 启动一个线程处理
            *   将接收到的消息通过blockManager存入内存
    -   ReliableKafkaReceiver -- 可靠方式 
        +   多创建一个对象blockGenerator = supervisor.createBlockGenerator(new GeneratedBlockHandler)
            *   数据首先进入blockGenerator的currentBuffer缓存，
            *   GeneratedBlockHandler监听各种事件，并处理
                -   新接受到的数据blockGenerator.addDataWithCallback加入缓存时
                    +   回调updateOffset，更新offset，
                    +   记录对应关系，topicPartitionOffsetMap.put(topicAndPartition, offset)
                -   划分新的一个批次时onGenerateBlock（blockId)
                    +   将blockid和之前的topicPartitionOffsetMap映射关系对应起来
                -   将新的批次的block通过blockManager存入内存时
                    +   提交offset

### Direct方式 -- KafkaUtils.createDirectStream
*   直接从kakfa blocks拉取消息，没有receiver,可以保证exactly once
*   不再通过zookeeper 保存offset,被消费的offset 由stream 本身维持处理
*   容错处理需要打开checkpointing，之前被消费的offset信息可以从checkpoint恢复
*   端到端语义，stream可以保证每个消息被高效的接收和转换exactly once ,但是不能保证 转换后的数据能不能输出的处理exactly once
*   DirectKafkaInputDStream
*   基本处理逻辑
    -   没有了receiver流程先写入memory，再创建rdd,直接计算时创建RDD，定时周期执行compute方法
    -   计算起始offset,第一次处理从哪里开始处理数据fromOffsets = getFromOffsets(kc, kafkaParams, topics)   
        -   auto.offset.reset =  "largest" or "smallest" （2.2.1 版本，2.0.1名称不同），决定从哪个位置开始处理消息
        +  "smallest"= kc.getEarliestLeaderOffsets(topicPartitions)，最早位置
        +  "largest" = kc.getLatestLeaderOffsets(topicPartitions)，最新位置
    -   定时周期执行compute方法
        +   构造RDD -- 从当前位置（第一次为fromOffset),到结束offset
            *   val rdd = KafkaRDD[K, V, U, T, R](
      context.sparkContext, kafkaParams, currentOffsets, untilOffsets, messageHandler)
            -   实际拉取数据通过低级kafka consumer api 拉取数据
        +   currentOffsets更新untilOffsets，并返回rdd

## window操作
### reduceByWindow
*   参数
    -   windowDuration-- 窗口时间，统计的区间范围，必须是batchDuration的整数倍
    -   slideDuration -- 窗口滑动时间，必须是batchDuration的整数倍
*   WindowedDStream
    -   compute
        -    切分窗口 val rddsInWindow = parent.slice(currentWindow)
        -    合并批次 Some(ssc.sc.union(rddsInWindow))
   

